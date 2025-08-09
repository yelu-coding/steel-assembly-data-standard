# Standardized Assembly Data Framework for Steel Structure Automation

This project presents a **standardized assembly description framework** designed to enable **efficient and integrated automation** in steel structure construction.  
Developed as part of my MSc thesis at **RWTH Aachen University**, under the supervision of **Dipl.-Ing. Heinrich Knitt** and reviewed by **Prof. Dr. Sigrid Brell-Cokcan**, it bridges **IFC building models** and **DSTV-compliant XML assembly data** through a geometry-aware, modular matching system.

**Key capabilities include**:
- IFC component extraction (geometry, identifiers, custom properties)
- DSTV XML parsing and NC file name mapping
- Hybrid matching strategy: NC name + geometry-based alignment
- Rotation matrix error computation with manual check flag
- JSON/CSV export for downstream integration
- Grasshopper Hops interface for live control and visualization

The framework is built with **Python Flask + Grasshopper Hops**, and is ready for integration into **digital fabrication workflows** in the AEC industry.
