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
```
---

## 🚀 Quick Start

The implementation of this framework, including all source code, sample data, and setup instructions, is available in a separate repository:  
[![View Code](https://img.shields.io/badge/View_Code-Repository-blue)](https://github.com/yelu-coding/gh_ifc-xml_matcher)

Follow the installation and usage steps provided there to:
1. Clone the repository and install dependencies
2. Start the Python Flask + Grasshopper Hops server
3. Connect Grasshopper components to the Hops endpoints
4. Run example IFC/XML matching workflows

---

## 📊 Sample Data Flow Diagram

This diagram illustrates the complete data processing and matching pipeline of the Standardized Assembly Description Framework,  
from IFC/DSTV parsing to geometric comparison, ambiguity detection, and final data export.

![Data Flow Diagram](data_flow_diagram.png)

The diagram shows how IFC and DSTV XML components are parsed, preprocessed, matched by NC identifiers or geometric comparison,
validated through ambiguity checks, and exported as structured data for automated steel structure assembly.

---

## 🎓 Academic Information
- **Institution**: RWTH Aachen University  
- **MSc Thesis**: *Advancing Steel Structure Assembly Automation: A Standardized Assembly Description for Enhanced Efficiency and Integration*  
- **First Reviewer**: Prof. Dr. Sigrid Brell-Cokcan  
- **Supervisor**: Dipl.-Ing. Heinrich Knitt  

---

## 📄 License
MIT License © 2025 Ye Lu

---

## 📬 Contact
- **Email**: luye_momo@foxmail.com
- **GitHub**: [yelu-coding](https://github.com/yelu-coding)
- **LinkedIn**: https://www.linkedin.com/in/ye-lu-a16261238/

---

## 🔗 Related Work
- [GH Dome Project](https://github.com/yelu-coding/gh-dome) — Previous Grasshopper scripting project
