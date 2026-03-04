# QSAR Model for CNS Drug Property Prediction

A machine learning pipeline for predicting **blood-brain barrier permeability (logBB)** of small molecules, built using public ChEMBL data, RDKit featurization, scikit-learn models, and ChemProp (D-MPNN).

---

## Motivation

CNS drug development requires compounds that can cross the blood-brain barrier. logBB — the log ratio of drug concentration in brain tissue vs. blood — is a key early-stage filter in CNS drug discovery programs. This project builds and benchmarks QSAR models that predict logBB from molecular structure alone, with a focus on **realistic validation**, **noise-aware evaluation**, and **applicability domain analysis**.

---

## Project Structure

```
qsar-logbb/
├── QSAR_CNS_Property_Prediction.ipynb   # Main notebook
├── model_results.csv                    # Final metrics table
├── logBB_distribution.png               # EDA figure
├── predicted_vs_actual.png              # Scatter plots per model
├── model_comparison.png                 # R² and RMSE bar charts
├── feature_importance.png               # RF top 20 features
├── applicability_domain.png             # In-domain vs. OOD scatter
└── README.md
```

---

## Pipeline Overview

```
ChEMBL API
    │
    ▼
Raw logBB assay data (~1,500 compounds)
    │
    ▼
RDKit Featurization
  ├── Physicochemical descriptors (MW, logP, TPSA, HBD/HBA, ...)
  └── Morgan fingerprints (ECFP4, radius=2, 1024 bits)
    │
    ▼
Scaffold-Based Train/Test Split (80/20)
    │
    ├──▶ Ridge Regression
    ├──▶ Random Forest
    ├──▶ Gradient Boosting        ──▶ Metrics vs. Noise Floor
    └──▶ ChemProp (D-MPNN)
    │
    ▼
Applicability Domain Analysis (k-NN)
    │
    ▼
Figures + model_results.csv
```

---

## Key Design Choices

### Scaffold-Based Splitting
Rather than a random 80/20 split, compounds are grouped by their **Murcko scaffold** (core ring system). This ensures the test set contains chemically distinct scaffolds never seen during training — a far more realistic evaluation scenario for drug discovery, where models are used to prioritize **new** chemical series.

### Noise Floor Estimation
logBB assays have a typical reproducibility of ~0.3 log units. Even a perfect model cannot beat measurement noise. We simulate this noise to estimate the **theoretical RMSE floor and R² ceiling**, then compare all models against this baseline. This answers the question: *is the model limited by chemistry or by data quality?*

### Applicability Domain
A model should only be trusted for compounds **similar** to its training set. We use **k-NN distance in Morgan fingerprint space** to flag out-of-domain test compounds (those beyond the 95th percentile of average neighbor distances). These predictions should be treated with extra caution.

### ChemProp (D-MPNN)
ChemProp uses a **Directed Message Passing Neural Network** that operates directly on molecular graphs, learning atom and bond representations without hand-crafted fingerprints. It is state-of-the-art for small-molecule property prediction and serves as the deep learning benchmark alongside the classical scikit-learn models.

---

## Installation

```bash
pip install chembl_webresource_client rdkit scikit-learn chemprop pandas numpy matplotlib seaborn
```

---

## Usage

Open `QSAR_CNS_Property_Prediction.ipynb` in Jupyter or Colab and run cells top to bottom (Shift+Enter).

| Cell | Description |
|------|-------------|
| 1 | Install dependencies |
| 2 | Imports |
| 3 | Fetch logBB data from ChEMBL |
| 4 | EDA — distribution and boxplot |
| 5 | Featurization functions |
| 6 | Scaffold split + feature matrix |
| 7 | Noise floor estimation |
| 8 | Train Ridge / RF / GBM |
| 9 | Train ChemProp (D-MPNN) |
| 10 | Applicability domain analysis |
| 11–14 | Generate figures |
| 15 | Final summary + export CSV |



---

## Results

| Model | R² | RMSE | MAE |
|---|---|---|---|
| Ridge Regression | -0.941 | 1.216 | 0.922 |
| Random Forest | -0.157 | 0.939 | 0.63 |
| Gradient Boosting | -0.328 | 1.006 | 0.757 |



---

## Outputs

| File | Description |
|------|-------------|
| `model_results.csv` | R², RMSE, MAE for all models |
| `logBB_distribution.png` | Histogram and boxplot of the dataset |
| `predicted_vs_actual.png` | Scatter plots with noise band overlay |
| `model_comparison.png` | Bar charts of R² and RMSE vs. noise floor |
| `feature_importance.png` | Top 20 Random Forest features |
| `applicability_domain.png` | In-domain vs. OOD test compounds |

---

## Dependencies

| Package | Purpose |
|---------|---------|
| `chembl_webresource_client` | ChEMBL data access |
| `rdkit` | Molecular featurization, scaffold analysis |
| `scikit-learn` | Ridge, RF, GBM models, k-NN AD |
| `chemprop` | Graph neural network (D-MPNN) |
| `pandas / numpy` | Data handling |
| `matplotlib` | Visualization |

---

## References

- Ertl, P. & Schuffenhauer, A. (2009). Estimation of synthetic accessibility score of drug-like molecules. *J Cheminform.*
- Yang, K. et al. (2019). Analyzing learned molecular representations for property prediction. *J Chem Inf Model.* (ChemProp)
- Sheridan, R.P. (2013). Time-split cross-validation as a method for estimating the goodness of prospective prediction. *J Chem Inf Model.*
- ChEMBL Database: https://www.ebi.ac.uk/chembl/

---

## Author

**Sarvesh**  
MS Bioinformatics, Northeastern University
