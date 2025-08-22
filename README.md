Here’s the **updated README** (simplified ToC, with the corrected **DSTV-XML** section and Step 1 covering **both IFC + XML** flattening). You can paste this straight into `README.md`.

---

# Advancing Steel Structure Assembly Automation

**A standardized assembly description & NC-pivot matching framework**
*M.Sc. Thesis: “Advancing Steel Structure Assembly Automation: A Standardized Assembly Description for Enhanced Efficiency and Integration.”*

> Make **IFC**, **DSTV-XML**, and **DSTV-NC** work together by using the **NC file** as the **pivot**.
> Three stages: **Flatten & normalize (IFC+XML)** → **Group, seed & learn (unique NC → R\_ref)** → **Propagate & check (duplicate-NC with optimal assignment; ties flagged)**.

## Table of Contents (Simplified)

* [Overview](#overview)
* [Data Model](#data-model)
* [Pipeline](#pipeline)
* [Usage (CLI & Python)](#usage-cli--python)
* [Results & Survey](#results--survey)
* [Limits & Roadmap / License / Contact](#limits--roadmap--license--contact)

---

## Overview

Design and fabrication live in **different software stacks**. **IFC** carries model meaning; **DSTV-XML/NC** carry fabrication & machine details. Links are weak, so teams still fix IDs/axes **by hand**.
In practice, **IFC and XML both reference the same NC filenames**—but IFC has **no standard field** for that link, so projects stash it in **different properties**. We treat the **NC file as a pivot**: start from easy, unique cases, then solve the hard duplicates.
**Outputs** are portable **JSON/CSV** → plug into Grasshopper, planning, QA, or digital twins **without reformat**. Cutting rework also reduces **waste & extra transport**.

---

## Data Model

**We flatten both IFC and XML** into compact per-part records, keeping only what matching and audits need.

### IFC → `flattened_ifc`

**Source fields**

* **GlobalId** — globally unique identifier (cross-dataset key).
* **Location** — global position via `IfcLocalPlacement` / `IfcCartesianPoint`.
* **RefDirection (X)** & **Axis (Z)** — from `IfcAxis2Placement3D`; compute **Y = Z × X** (right-hand rule).

**Normalized record**

```jsonc
{
  "ifc_id": "GUID-or-ElementID",
  "location": [x, y, z],
  "axis": [[x1,y1,z1],[x2,y2,z2],[x3,y3,z3]],  // B_ifc = [X Y Z], Y = Z × X
  "reflection": false,                           // det(B_ifc) < 0 → true
  "properties": { "Pset_*": { "Key": "Value" } },
  "nc_hints": [
    { "value": "PROJ_045.nc", "source": "Pset_Custom.NCFile" },
    { "value": "045.nc",      "source": "Name" }
  ]
}
```

> `nc_hints` is a **flattening-time helper list** that surfaces NC-like strings found anywhere in IFC properties, with **source path** for auditability. If your project already uses a fixed field (e.g., `Pset_NC_Linkage.NC_FileName`), `nc_hints` will simply hold that authoritative value.

### DSTV-XML → `flattened_xml`

**Source fields** (exporters vary; we accept multiple representations)

* **ID** — unique per-part identifier (e.g., `<Id>`, `<PosNo>`, `<Name>`, attribute `id`).
* **Origin/Base** — global coordinates (e.g., `<Base>`, `<Origin>`, `<Transformation><Origin>`, `<KOORD>`).
* **Orientation** — one of:

  * **A. Vector form**: two unit vectors (e.g., `<Rx>`, `<Ry>` or `<X>`, `<Y>`).
  * **B. Matrix form**: a 3×3 rotation matrix (e.g., `<Matrix>` or `m11..m33`).
  * **C. Euler/axis-angle form**: e.g., `rx/ry/rz` or `<Rotation>` with angles.
* **NC filename** — e.g., `<NCFile>`, `<Reference>`, `<DSTV><FileName>`, attribute `filename`, etc.

**Normalized record**

```jsonc
{
  "xml_id": "P045",
  "location": [x, y, z],
  "axis": [[x1,y1,z1],[x2,y2,z2],[x3,y3,z3]],  // B_xml = [X Y Z]
  "reflection": false,                           // det(B_xml) < 0 → true
  "properties": { "Profile": "HEA200", "Grade": "S355" },
  "nc_filename": "PROJ_045.nc"
}
```

**How `axis` is built**

* **A. Vector form**
  `X = normalize(Rx or X)`; `Y = normalize(Ry or Y)`; `Z = normalize(X × Y)` → **B\_xml = \[X Y Z]**.
* **B. Matrix form**
  `B_xml = orthonormalize(Matrix)`; use columns as **X, Y, Z**.
* **C. Euler/axis-angle**
  Build `B_xml` from the documented rotation order (e.g., `Rz(γ)·Ry(β)·Rz(α)`), then orthonormalize.

> Regardless of source form, we **orthonormalize** to a right-handed basis `B_xml = [X Y Z]` and set `reflection = (det(B_xml) < 0)`.

---

## Pipeline

**Step 1 — Flatten & normalize (IFC + XML).**
Parse IFC & XML → build unified records `{id, axis, location, reflection, properties}`.

* IFC: compute **B\_ifc** from `IfcAxis2Placement3D` (**Y = Z × X**); collect **`nc_hints[]`** from properties (since IFC has no standard NC field).
* XML: compute **B\_xml** from vectors/matrix/angles; keep the **authoritative** `nc_filename` from XML.
  Write both to **JSON/CSV**. This step **exposes** IFC’s hidden NC links and preserves XML’s **authoritative** NC, making grouping and seeding straightforward.

**Step 2 — Group, seed & learn.**
Use XML’s **`nc_filename`** as the anchor; for each NC name, find IFC parts whose **`nc_hints[].value` equals that name**.

* If the NC group is **unique 1↔1**, match it immediately → **seed pair**.
* From seeds, **learn** the reference transform: rotation **`R_ref`** (and optional translation **`t_ref`**).

**Step 3 — Propagate & check.**
For **duplicate-NC** groups (m\:n), build a **cost matrix** comparing candidates against **`R_ref`** (rotation primary; position optional) and solve **one-to-one** with **Hungarian** (optimal assignment).

* **Tie handling**: if two assignments have **equal cost** within tolerance, mark as **ambiguous** and request a quick confirmation.
* **Outputs**: `match.json` (groups, `R_ref`, scores, flags) and `validation.csv` (auditable per-pair table).

> Fail **safely**, not silently.

---

## Usage (CLI & Python)

> **CLI** = Command-Line Interface. Use it for reproducibility and batch runs; or import functions directly in Python.

**Install**

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt   # numpy pandas scipy ifcopenshell lxml/xmltodict rich click
```

**CLI — run a case**

```bash
python -m src.cli match \
  --ifc data/case1_ring_beam/ring.ifc \
  --xml data/case1_ring_beam/parts.xml \
  --out outputs/case1 \
  --rot-metric frobenius --tau-tie 1e-3
# Inspect: outputs/case1/match.json, outputs/case1/validation.csv
```

**Python — programmatic**

```python
from src.flatten import flatten_ifc, flatten_xml
from src.grouping import group_by_nc
from src.learn_transform import learn_reference
from src.assign import assign_hungarian

ifc = flatten_ifc("ring.ifc")      # returns flattened_ifc list/dict
xml = flatten_xml("parts.xml")     # returns flattened_xml list/dict
groups = group_by_nc(ifc, xml)     # XML nc_filename ↔ IFC nc_hints[].value
R_ref, t_ref, seeds = learn_reference(groups)
assignments = assign_hungarian(groups, R_ref, t_ref)  # includes tie flags
```

---

## Results & Survey

**Case 1 — Ring beam (9 parts).** Clean baseline with several unique NC groups: **100%** correct; a duplicate name resolved by geometry (not rules).
**Case 2 — Industrial (100+ parts).** Many duplicate-NC groups. Flow: **lock unique seeds → learn `R_ref` → propagate**. On the checked subset, **43 pairs** confirmed; **overall accuracy > 90%**; flagged pairs remained functionally correct after visual check.
**Case 3 — IFC2x3 vs IFC4 (subset).** IFC4 from production (not a curated twin), trimmed to a **common subset** and matched to the **same XML**. Because the two exports **list components in different orders**, **two pairs got the same match score**. When scores **tie**, the solver may return **either** pairing—**order-neutral** yet equally valid. This reveals the **real cause of ambiguity**, so the pipeline **labels all ties as ambiguous** for a brief check.

**Industry survey → actions.**
Teams reported **manual/semi-manual linking** and **inconsistent NC storage** in IFC; demand for automated IFC–NC is strong.
We propose: IFC **`Pset_NC_Linkage.NC_FileName`** + unique NC naming **`PROJECTCODE_MARKNO.nc`**.

---

## Limits & Roadmap / License / Contact

**Limits.** Ties are **reported** (not forced); the method benefits from basic NC naming hygiene.
**Roadmap.** Add secondary cues (pos offsets, feature signatures, adjacency) or **AI** to **break ties**; formalize **assembly semantics** in IFC; broaden cross-tool validation; parallel per-NC group.
**License.** MIT (or your choice).
**Contact.** **Ye Lu** · RWTH Aachen — Construction & Robotics (2025). Issues → GitHub Issues; email on your poster/card.

---

Want me to drop this into a file and add a minimal `requirements.txt` + `src/cli.py` skeleton that matches the README commands?
