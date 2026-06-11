# BRD4 Inhibitor Discovery Pipeline
### A end-to-end Computer-Aided Drug Design (CADD) workflow for BRD4 (CHEMBL4523) repurposing and hit identification

![Python](https://img.shields.io/badge/Python-3.9+-blue?logo=python&logoColor=white)
![RDKit](https://img.shields.io/badge/RDKit-Cheminformatics-green)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-ML-orange?logo=scikit-learn)
![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Overview

This repository implements a fully reproducible, modular CADD pipeline targeting **BRD4 (Bromodomain-containing protein 4)** — a well-validated epigenetic target in oncology. The pipeline spans the complete drug discovery workflow: from raw bioactivity data ingestion through machine learning-based potency prediction, virtual screening, molecular docking, ADMET filtering, and molecular dynamics validation.

BRD4 was selected for its clinical relevance (acute myeloid leukemia, NUT midline carcinoma) and the availability of high-quality public bioactivity data via ChEMBL.

> **Goal:** Identify drug-like BRD4 inhibitor candidates through computational methods, with a focus on repurposing approved drugs.

---

## Pipeline Architecture

```
ChEMBL Bioactivity Data
        │
        ▼
┌─────────────────────┐
│  Module 1           │  Data Ingestion & pIC50 Transformation
│  ChEMBL API         │  → 4,800+ compounds retrieved
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Module 2           │  Drug-likeness Filtering
│  Lipinski / Veber   │  → ~2,200 drug-like compounds retained
│  / PAINS            │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Module 3           │  Molecular Fingerprinting
│  Morgan ECFP4       │  → ECFP4 / MACCS / RDKit descriptor matrices
│  MACCS / RDKit      │  → Tanimoto similarity analysis
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Module 4           │  Dimensionality Reduction
│  PCA / UMAP         │  → Chemical space visualization
│                     │  → Applicability domain (brd4_ad_params.json)
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Module 5           │  ML Potency Prediction
│  Random Forest      │  → Best model: RF  Test R² = 0.715
│  XGBoost / SVM      │  → Cross-validated, hyperparameter-tuned
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Module 6           │  Virtual Screening
│  Approved Drugs     │  → ChEMBL approved drugs screened
│                     │  → Repurposing candidates ranked by pIC50
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Modules 7–10       │  Molecular Docking (AutoDock Vina / GNINA)
│  [In Progress]      │  ADMET Prediction · MD Simulation
└─────────────────────┘
```

---

## Modules

| # | Module | Key Output | Status |
|---|--------|------------|--------|
| 1 | Data Ingestion & Preprocessing | `brd4_raw.csv`, `brd4_pIC50.csv` | ✅ Complete |
| 2 | Drug-likeness Filtering | `brd4_filtered.csv` (~2,200 cpds) | ✅ Complete |
| 3 | Molecular Fingerprinting | ECFP4/MACCS/RDKit matrices, Tanimoto heatmap | ✅ Complete |
| 4 | PCA / UMAP Visualization | Chemical space plots, `brd4_ad_params.json` | ✅ Complete |
| 5 | ML Model Training | RF model (R²=0.715), XGBoost, SVM | ✅ Complete |
| 6 | Virtual Screening | Repurposing candidate list | ✅ Complete |
| 7 | Molecular Docking | Docking scores, binding poses | 🔄 In Progress |
| 8 | ADMET Prediction | ADMET-filtered candidate list | 🔄 In Progress |
| 9 | MD Simulation | Trajectory analysis, RMSD/RMSF | 🔄 Planned |
| 10 | Pipeline Automation | Snakemake/Nextflow workflow | 🔄 Planned |

---

## Key Results (Modules 1–6)

- **4,800+** BRD4 bioactivity records retrieved from ChEMBL 33
- **~2,200** compounds passed Lipinski, Veber, and PAINS filters
- **Random Forest** achieved the best predictive performance: **Test R² = 0.715**, outperforming XGBoost and SVM
- **Applicability domain** defined using PCA bounding box — stored in `brd4_ad_params.json` for use in prospective screening
- **Virtual screening** of ChEMBL approved drugs identified repurposing candidates ranked by predicted pIC50

---

## Repository Structure

```
brd4_pipeline/
│
├── data/
│   ├── raw/               # ChEMBL raw downloads (gitignored)
│   ├── processed/         # Cleaned & transformed data
│   ├── filtered/          # Drug-likeness filtered compounds
│   └── outputs/           # Final results, plots, models
│
├── notebooks/
│   ├── module_01_ingestion.ipynb
│   ├── module_02_filtering.ipynb
│   ├── module_03_fingerprints.ipynb
│   ├── module_04_dimensionality.ipynb
│   ├── module_05_ml_models.ipynb
│   └── module_06_virtual_screening.ipynb
│
├── models/                # Saved ML models (.pkl)
├── config.yaml            # Pipeline configuration
├── requirements.txt       # Python dependencies
├── .gitignore
└── README.md
```

---

## Installation & Setup

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/brd4-cadd-pipeline.git
cd brd4-cadd-pipeline

# Create a conda environment (recommended)
conda create -n brd4_env python=3.9
conda activate brd4_env

# Install RDKit via conda
conda install -c conda-forge rdkit

# Install remaining dependencies
pip install -r requirements.txt
```

### `requirements.txt`
```
chembl-webresource-client
pandas
numpy
scikit-learn
xgboost
umap-learn
matplotlib
seaborn
joblib
pyyaml
jupyter
```

---

## Usage

Run notebooks sequentially from `Module 1` through `Module 6`. Each notebook is self-contained and reads from / writes to the `data/` directory as defined in `config.yaml`.

```bash
jupyter notebook notebooks/module_01_ingestion.ipynb
```

Or run all modules end-to-end *(automation in progress — Module 10)*.

---

## Scientific Background

**BRD4** (Bromodomain-containing protein 4) is an epigenetic reader protein that recognizes acetylated lysine residues on histones, regulating transcription of oncogenes including **c-Myc** and **BCL2**. BRD4 inhibition has shown therapeutic promise in:

- Acute Myeloid Leukemia (AML)
- NUT Midline Carcinoma
- Solid tumors with MYC amplification

This pipeline uses **CHEMBL4523** as the target ID and retrieves IC₅₀ data transformed to **pIC50 = −log₁₀(IC₅₀)** for continuous regression modeling.

---

## Technologies Used

| Category | Tools |
|----------|-------|
| Cheminformatics | RDKit, ChEMBL Web Resource Client |
| ML / Data Science | scikit-learn, XGBoost, pandas, numpy |
| Visualization | matplotlib, seaborn, UMAP |
| Docking *(upcoming)* | AutoDock Vina / GNINA |
| ADMET *(upcoming)* | SwissADME, pkCSM, ADMETlab |
| MD Simulation *(upcoming)* | GROMACS / OpenMM |
| Workflow | Jupyter Notebooks → Snakemake *(planned)* |

---

## Author

**Darshi**
B.Pharm | Aspiring Cheminformatics & CADD Scientist

- Building toward roles in computational drug discovery at pharma/biotech
- Combining pharmacy domain knowledge with practical ML/cheminformatics skills

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

*This pipeline is under active development. Modules 7–10 covering molecular docking, ADMET prediction, MD simulation, and full automation are in progress.*
