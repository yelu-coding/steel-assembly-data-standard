# Advancing Steel Structure Assembly Automation
**A standardized assembly description & NC-pivot matching framework**  
_M.Sc. Thesis: **“Advancing Steel Structure Assembly Automation: A Standardized Assembly Description for Enhanced Efficiency and Integration.”**_

<p align="center">
  <img src="assets/data_flow_diagram.png" alt="Pipeline overview" width="720"/>
</p>

## TL;DR
Make **IFC**, **DSTV-XML**, and **DSTV-NC** _work together_ by using the **NC file** as the **pivot**.  
**Three steps:** (1) **Flatten & normalize** IFC/XML → a small, readable schema; (2) **Group, seed & learn** a reliable IFC↔XML rotation from **unique NC** pairs; (3) **Propagate & check** to resolve **duplicate-NC** groups via optimal assignment, and **flag ties** (equal-cost matches) for a quick confirmation.  
Outputs: clean **JSON/CSV** that are **robot‑ready** and easy to plug into GH / planning / QA / digital twins.

---

## Why this matters
- Files come from **different software**, so links are weak: **IFC** carries model meaning; **DSTV-XML/NC** carry fabrication & machine details.  
- Teams still fix IDs/axes by hand → slow & error‑prone; automation stalls.  
- In practice, **IFC and XML both reference the same NC filenames**. We treat the NC file as a **pivot** to start simple, then solve the hard cases.

---

## Pipeline Overview
**Step 1 — Flatten & normalize.**  
Flatten IFC’s complex hierarchy and XML into a compact per‑part record:  
```jsonc
{
  "id": "IFC/element id or XML id",
  "axis": [[r11,r12,r13],[r21,r22,r23],[r31,r32,r33]],   // local basis or orientation
  "location": [x, y, z],                                 // reference point (e.g., part origin / centroid)
  "reflection": false,                                   // mirror flag if present
  "properties": {"...": "..."},                          // carries arbitrary Psets / attrs
  "nc_candidates": ["P123_045.nc"]                       // strings surfaced from properties
}
```
> There is **no standard place** in IFC to store NC links; projects hide them in different properties. Pre‑cleaning makes NC hints **visible** and matching **simpler**.

**Step 2 — Group, seed & learn.**  
- **Group by NC filename** (from `nc_candidates`).  
- Wherever a group is **unique 1↔1**, match it immediately and **learn** the IFC↔XML **reference rotation** `R_ref` (and optional translation `t_ref`) from these **seed pairs**.

**Step 3 — Propagate & check.**  
- For **duplicate‑NC** groups (m:n), build a **cost matrix** comparing each candidate’s orientation to `R_ref` (and optionally position).  
- Solve **one‑to‑one** with the **Hungarian** algorithm (optimal assignment).  
- **Tie handling:** if two assignments have **equal cost** within a tolerance, mark as **ambiguous** → require a quick confirmation. The pipeline **fails safely**, not silently.

---

## Mathematical Notes (concise)
- Let `B_ifc`, `B_xml` be local bases (3×3) for an IFC and an XML part. A candidate rotation is `R = B_xml · (B_ifc)^{-1}`.  
- The rotation cost can be **Frobenius** `ε_rot = ||R − R_ref||_F` or an **angle** distance derived from `R_ref^T R`.  
- Optional position term: `ε_pos = ||(p_xml − p_ifc) − t_ref||_2`.  
- **Total cost** (tunable): `ε = w_rot·ε_rot + w_pos·ε_pos`.  
- **Tie** if `|ε_i − ε_j| < τ_tie` (and optional `ε_pos` within tolerance).

---

## Repository Structure
```
.
├── assets/                         # diagrams, poster figures (add data_flow_diagram.png here)
├── data/
│   ├── case1_ring_beam/            # 9 parts; clean baseline
│   ├── case2_industrial_subset/    # 100+ parts; duplicate-NC groups
│   └── case3_ifc2x3_vs_ifc4/       # IFC4 subset aligned to IFC2x3; same XML
├── src/
│   ├── io_ifc.py                   # IFC parsing (IFCOpenShell or alternative)
│   ├── io_xml.py                   # DSTV-XML parsing utilities
│   ├── flatten.py                  # build {id, axis, location, reflection, properties, nc_candidates}
│   ├── grouping.py                 # NC-based grouping utilities
│   ├── learn_transform.py          # estimate R_ref (and t_ref) from unique-NC seeds
│   ├── assign.py                   # cost matrix + Hungarian + tie detection
│   ├── export_json.py              # structured JSON export
│   ├── export_csv.py               # validation CSV export
│   └── cli.py                      # tiny CLI entry points
├── viz/
│   ├── gh_reader.ghx               # (optional) Grasshopper file to visualize checks
│   └── rhino_notes.md
├── outputs/                        # results written here
└── README.md
```

---

## Quick Start
> Requires Python **3.10+**. Recommended on a fresh virtual environment.

```bash
# 1) Create env & install
python -m venv .venv && source .venv/bin/activate   # (Windows) .venv\Scripts\activate
pip install -r requirements.txt  # numpy, pandas, scipy, ifcopenshell, lxml/xmltodict, rich, click

# 2) Run a case (example: Case 1)
python -m src.cli match \
  --ifc data/case1_ring_beam/ring.ifc \
  --xml data/case1_ring_beam/parts.xml \
  --out outputs/case1 \
  --rot-metric frobenius --tau-tie 1e-3 --tau-pos 1e-3

# 3) Inspect outputs
# - outputs/case1/validation.csv   (ifc_id, xml_id, nc_name, rot_cost, pos_cost, total, tie, ambiguous, status)
# - outputs/case1/match.json       (groups, R_ref, t_ref, per-pair scores & flags)
```

**Reproduce all cases**
```bash
python -m src.cli run-all --out outputs \
  --case1 --case2 --case3 --rot-metric frobenius --tau-tie 1e-3
```

---

## Results (replicable)
- **Case 1 — Ring beam (9 parts):** clean baseline, **100%** matches; duplicate name resolved by geometry (not ad‑hoc rules).  
- **Case 2 — Industrial (100+ parts):** many duplicate‑NC groups. Flow: **lock unique seeds → learn R_ref → propagate**. On the checked subset, **43 pairs** confirmed with **overall accuracy > 90%**; flagged pairs were visually confirmed and functionally correct downstream.  
- **Case 3 — IFC2x3 vs IFC4 (subset):** IFC4 was from production (not a curated twin), so trimmed to a **common subset** and matched to the **same XML**. Because exports **list components in different orders**, **two pairs had the same match score**. With a **tie**, the solver may return **either** pairing—order‑dependent yet equally valid. This exposes the **real cause of ambiguity**, so the pipeline **labels all ties as ambiguous** for a quick confirmation.

---

## Outputs
- **`match.json`** — structured record per NC group: seeds, `R_ref`, candidate scores, assignments, tie/ambiguous flags.  
- **`validation.csv`** — per‑pair table for audits: `ifc_id, xml_id, nc_name, rot_cost, pos_cost, total_cost, tie, ambiguous, status`.  
- (Optional) **`sequence.csv`** — simple sequence suggestion (if you export one).

---

## Industry Survey → Actions
- **Findings:** linking is **manual/semi‑manual**; NC is stored **inconsistently** across IFC properties; demand for **automated IFC–NC** linking.  
- **Actions:** define **`Pset_NC_Linkage.NC_FileName`** in IFC; adopt **unique NC naming** e.g. `PROJECTCODE_MARKNO.nc`.

---

## Limitations & Roadmap
- **Ties:** equal-cost assignments are **reported as ambiguous** rather than forced.  
- **Naming hygiene:** method benefits from basic NC naming discipline.  
- **Next:** add secondary cues (pos offsets, feature signatures, adjacency) or **AI** to **break ties**; formalize **assembly semantics** in IFC; broaden cross‑tool validation; parallel pipelines per NC group.

---

## Sustainability & Impact
Less manual “glue work”, fewer errors, **robot‑ready** identities & axes. Portable **JSON/CSV** slots into GH, scheduling, QA, or digital twins with **no reformat**. Cutting rework/mis‑fabrication **reduces waste & transport**.

---

## Citation
If this repository helps your work, please cite:
```text
Ye Lu. Advancing Steel Structure Assembly Automation: A Standardized Assembly Description for Enhanced Efficiency and Integration, 2025.
```

## License
Choose a license (e.g., MIT). Replace this section with the actual text once decided.

## Contact
**Ye Lu** · RWTH Aachen — Construction & Robotics (2025)  
Issues & questions: please open a GitHub issue or reach me via the email on my poster/card.
