# Standardized Assembly Data Framework for Steel Structure Automation

This project presents a **standardized assembly description framework** designed to enable **efficient and integrated automation** in steel structure construction.  
Developed as part of my MSc thesis at **RWTH Aachen University**, under the supervision of **Dipl.-Ing. Heinrich Knitt** and reviewed by **Prof. Dr. Sigrid Brell-Cokcan**, it bridges **IFC building models** and **DSTV-compliant XML assembly data** through a geometry-aware, modular matching system.

---

## 📚 Background

In the steel construction industry, data for automated assembly often comes from heterogeneous sources such as **IFC building information models** and **DSTV-compliant XML assembly instructions**.  
The absence of a standardized description method leads to inconsistencies, integration challenges, and reduced automation efficiency.

This research proposes a **Standardized Assembly Description Framework** that unifies geometry, identifiers, and metadata into a consistent, automation-ready format, enabling seamless integration into digital fabrication workflows.

---

## 🧩 Key Capabilities

- **IFC Component Extraction**: Geometry, GlobalId, position, direction, and custom properties.
- **DSTV XML Parsing**: Direct NC file reference extraction.
- **NC Filename Mapping**: Link IFC component properties to existing NC file directories.
- **Hybrid Matching**: NC file name-based + geometry-based alignment.
- **Error Computation**: Rotation matrix deviation with manual check flag.
- **Data Export**: JSON and CSV formats for downstream use.
- **Grasshopper Integration**: Modular Hops endpoints for live input & control.

---

## 📁 Repository Structure
```text
steel-assembly-data-standard/
├── docs/                  # Thesis abstract, diagrams, GIF demos
├── src/                   # Core Python + Grasshopper Hops scripts
├── data/                  # Sample IFC/XML input files
├── output/                # Example output files (.json, .csv, .xlsx)
├── LICENSE                # License information
└── README.md              # You are here
、、、

## 🚀 Quick Start
### 1. Clone the repository
、、、bash
git clone https://github.com/yelu-coding/steel-assembly-data-standard.git
cd steel-assembly-data-standard
、、、
