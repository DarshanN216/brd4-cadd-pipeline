# BRD4 Inhibitor Discovery Pipeline
### A Production-Grade Computational Drug Discovery Workflow

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![RDKit](https://img.shields.io/badge/RDKit-2023+-green)
![scikit--learn](https://img.shields.io/badge/scikit--learn-1.3+-orange)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Overview

This repository contains a complete, end-to-end computational pipeline for the identification and prioritization of BRD4 (Bromodomain-containing protein 4) inhibitors using cheminformatics, machine learning, and structure-based methods.

BRD4 is an epigenetic reader protein that drives transcription of oncogenes including MYC, making it a high-value target in oncology. This pipeline was built to demonstrate production-grade CADD (Computer-Aided Drug Design) workflows applicable to any small-molecule target.

**Target:** BRD4, Human — ChEMBL ID: CHEMBL4523

---

## Scientific Highlights

- **2,909 BRD4 inhibitors** curated from ChEMBL with rigorous data quality controls
- **QSAR model** achieving Test R² = 0.715 on held-out compounds
- **1,153 repurposing candidates** identified from FDA-approved drugs
- **Top hit: Sunitinib** (predicted pIC50 = 7.95) — consistent with published BRD4 polypharmacology literature
- Full **Applicability Domain** validation on all virtual screening predictions
- Complete **scaffold analysis** identifying privileged BRD4 chemical frameworks

---

## Pipeline Architecture

```
Phase 1 — Data Quality & Featurization
│
├── Module 1 │ Raw Ingestion & Chemical Cleanliness
├── Module 2 │ Physicochemical Property Space & Drug-likeness Filtering
├── Module 3 │ Molecular Fingerprints & Similarity Analysis
└── Module 4 │ Chemical Space Visualization & Applicability Domain

Phase 2 — Predictive Modeling & Virtual Screening
│
├── Module 5 │ QSAR Model Optimization & Validation
└── Module 6 │ Virtual Screening Engine & Hit Prioritization

Phase 3 — Structural Analysis & Production
│
├── Module 7 │ Scaffold Analysis & R-Group Decomposition
├── Module 8 │ ADMET Prediction & Multi-Parameter Optimization
├── Module 9 │ 3D Docking & Protein-Ligand Interaction Fingerprints
└── Module 10│ Production Deployment & Portfolio Documentation
```

---

## Repository Structure

```
brd4_pipeline/
│
├── data/
│   ├── raw/                        ← Raw ChEMBL API output (never modified)
│   │   └── brd4_IC50_raw.csv
│   │
│   ├── processed/                  ← Intermediate cleaned datasets
│   │   ├── brd4_IC50_cleaned.csv
│   │   ├── brd4_IC50_structures.csv
│   │   ├── brd4_morgan_fp.npy
│   │   ├── brd4_maccs_fp.npy
│   │   ├── brd4_rdkit_fp.npy
│   │   ├── brd4_ad_params.json
│   │   ├── brd4_best_model.pkl
│   │   ├── brd4_model_meta.json
│   │   ├── brd4_scaffold_analysis.csv
│   │   └── brd4_privileged_scaffolds.csv
│   │
│   └── filtered/                   ← Final analysis-ready datasets
│       ├── brd4_final_clean.csv
│       ├── brd4_druglike.csv
│       ├── brd4_PAINS_flagged.csv
│       ├── brd4_virtual_hits.csv
│       └── brd4_screened_library.csv
│
├── notebooks/
│   ├── module_01_ingestion.ipynb
│   ├── module_02_properties.ipynb
│   ├── module_03_fingerprints.ipynb
│   ├── module_04_chemical_space.ipynb
│   ├── module_05_qsar_modeling.ipynb
│   ├── module_06_virtual_screening.ipynb
│   ├── module_07_scaffold_analysis.ipynb
│   ├── module_08_admet.ipynb
│   ├── module_09_docking.ipynb
│   └── module_10_production.ipynb
│
├── outputs/                        ← All generated plots and figures
│   ├── module02_property_distributions.png
│   ├── module02_druglike_filter.png
│   ├── module03_fingerprint_analysis.png
│   ├── module04_pca.png
│   ├── module04_umap.png
│   ├── module04_applicability_domain.png
│   ├── module05_model_performance.png
│   ├── module06_virtual_screening.png
│   └── module07_scaffold_landscape.png
│
└── README.md
```

---

## Module Summary

### Module 1 — Raw Ingestion & Chemical Cleanliness
Pulls IC50 bioactivity data directly from the ChEMBL API for BRD4 (CHEMBL4523). Applies a rigorous cleaning pipeline including binding assay filtering, pIC50 log-transformation, RDKit salt stripping, canonical SMILES generation, and PAINS substructure removal.

**Key results:**
- Raw records: 3,931
- Final clean molecules: 2,909
- Retention rate: 74.0%
- pIC50 range: 3.59 – 10.92

---

### Module 2 — Physicochemical Property Space
Calculates six molecular descriptors (MW, LogP, HBD, HBA, TPSA, RotB) using RDKit. Applies Lipinski's Rule of Five and Veber rules to retain drug-like compounds suitable for oral administration.

**Key results:**
- Drug-like molecules retained: 2,208 (76.0%)
- Mean MW: 400.0 Da
- Mean LogP: 3.53 (consistent with BRD4 hydrophobic pocket)
- Mean TPSA: 90.3 Ų

---

### Module 3 — Molecular Fingerprints & Similarity
Generates three complementary fingerprint representations: Morgan/ECFP4 (2048-bit), MACCS Keys (167-bit), and RDKit Topological (2048-bit). Performs Tanimoto similarity analysis against the most potent compound (CHEMBL3940091, pIC50 = 10.92).

**Key results:**
- Fingerprint matrices: (2208×2048), (2208×167), (2208×2048)
- Dataset chemical diversity: 95.6% of molecules have Tanimoto < 0.40 vs best compound

---

### Module 4 — Chemical Space Visualization & Applicability Domain
Applies PCA and UMAP dimensionality reduction to visualize 2208 molecules in 2D chemical space colored by pIC50. Defines a PCA-based Applicability Domain (±3 SD bounding box) for regulatory-grade prediction confidence flagging.

**Key results:**
- UMAP reveals high-pIC50 molecules cluster in defined regions — confirms learnable SAR
- AD boundary saved for use in virtual screening (Module 6)

---

### Module 5 — QSAR Model Optimization & Validation
Trains and compares three regression architectures: Random Forest, XGBoost, and SVM. Uses 5-fold cross-validation and GridSearchCV hyperparameter tuning. Evaluates on a held-out 20% test set.

**Key results:**

| Model | CV R² | Test R² | Test RMSE |
|---|---|---|---|
| Random Forest (tuned) | 0.687 | **0.715** | **0.648** |
| XGBoost (tuned) | 0.699 | 0.710 | 0.654 |
| SVM | 0.601 | 0.655 | 0.713 |

Selected model: **Random Forest (tuned)**

---

### Module 6 — Virtual Screening & Hit Prioritization
Screens 3,475 FDA-approved small molecule drugs from ChEMBL against the trained BRD4 QSAR model. Filters hits by predicted pIC50 ≥ 6.0, Applicability Domain membership, and SA Score ≤ 6.0.

**Key results:**
- Final hits: 1,153 compounds
- Top hit: **Sunitinib** (pIC50 predicted = 7.95)
- Notable hits: Niraparib, Abrocitinib, Gemifloxacin

---

### Module 7 — Scaffold Analysis & R-Group Decomposition
Extracts Bemis-Murcko scaffolds for all 2,208 drug-like molecules. Identifies privileged scaffolds defined as those with count ≥ 5, mean pIC50 ≥ 7.0, and max pIC50 ≥ 8.0. Performs R-group decomposition on the top scaffold to map substituent effects on potency.

---

### Module 8 — ADMET Prediction *(In Progress)*
Builds predictive filters for hERG cardiac safety, blood-brain barrier permeability, and aqueous solubility. Applies multi-parameter optimization (MPO) scoring to rank compounds by combined potency and safety profile.

---

### Module 9 — 3D Docking *(In Progress)*
Retrieves BRD4 crystal structure from PDB, generates 3D ligand conformers, and runs AutoDock Vina docking simulations. Extracts Protein-Ligand Interaction Fingerprints (PLIF) to confirm hinge-region hydrogen bond contacts.

---

### Module 10 — Production Deployment *(In Progress)*
Refactors notebook code into production Python modules. Builds automated pipeline with YAML configuration. Finalizes GitHub repository with full technical documentation.

---

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/brd4_pipeline.git
cd brd4_pipeline

# Create conda environment
conda create -n brd4 python=3.10 -y
conda activate brd4

# Install dependencies
pip install pandas numpy matplotlib seaborn
pip install rdkit
pip install chembl_webresource_client
pip install scikit-learn xgboost
pip install umap-learn
pip install jupyterlab
```

---

## Quick Start

```python
# Run modules in order
# Each module saves checkpoints — safely resumable at any step

# Module 1: Pull and clean BRD4 data
# Output: data/filtered/brd4_final_clean.csv

# Module 2: Drug-likeness filtering
# Output: data/filtered/brd4_druglike.csv

# Module 3: Fingerprint generation
# Output: data/processed/brd4_morgan_fp.npy

# Module 4: Chemical space + Applicability Domain
# Output: data/processed/brd4_ad_params.json

# Module 5: QSAR modeling
# Output: data/processed/brd4_best_model.pkl

# Module 6: Virtual screening
# Output: data/filtered/brd4_virtual_hits.csv
```

---

## Key Dependencies

| Library | Version | Purpose |
|---|---|---|
| RDKit | ≥ 2023.03 | Cheminformatics engine |
| pandas | ≥ 2.0 | Data manipulation |
| scikit-learn | ≥ 1.3 | Machine learning |
| XGBoost | ≥ 1.7 | Gradient boosting |
| UMAP-learn | ≥ 0.5 | Dimensionality reduction |
| chembl_webresource_client | ≥ 0.10 | ChEMBL API access |
| matplotlib / seaborn | latest | Visualization |

---

## Data Sources

- **Bioactivity data:** ChEMBL Database v33 (EMBL-EBI)
  — Target: BRD4, Human (CHEMBL4523)
  — Activity type: IC50, Binding assays only

- **Screening library:** ChEMBL approved small molecule drugs
  — max_phase = 4, molecule_type = Small molecule

- **PAINS filters:** Baell & Holloway, J. Med. Chem. 2010, 53(7), 2719–2740

- **SA Score:** Ertl & Schuffenhauer, J. Cheminform. 2009, 1, 8

---

## Results Summary

```
Starting point      : 3,931 raw ChEMBL records
After cleaning      : 2,909 unique BRD4 inhibitors
After drug-likeness : 2,208 oral drug candidates
QSAR model R²       : 0.715 (Random Forest, tuned)
Virtual hits        : 1,153 repurposing candidates
Top predicted hit   : Sunitinib (pIC50 = 7.95)
```

---

## Author

**Darshi **
B.Pharm | Cheminformatics & AI Drug Discovery

Building production-grade CADD pipelines at the intersection of pharmacy, cheminformatics, and machine learning.

---

## License

This project is licensed under the MIT License.

---

## Acknowledgements

- EMBL-EBI ChEMBL team for maintaining the bioactivity database
- RDKit community for the open-source cheminformatics toolkit
- Bemis & Murcko (1996) for the scaffold framework methodology
- Baell & Holloway (2010) for the PAINS filter definitions
