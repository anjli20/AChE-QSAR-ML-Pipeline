# AChE-QSAR: ML Pipeline for Acetylcholinesterase Bioactivity Prediction

A production-grade QSAR (Quantitative Structure-Activity Relationship) pipeline for
predicting the inhibitory activity (pIC50) of small molecules against Acetylcholinesterase
(AChE) — a key target in Alzheimer's disease drug discovery. The pipeline fetches public
ChEMBL bioactivity data, engineers molecular features using RDKit, benchmarks multiple
ML models under scaffold-aware validation, generates conformal prediction intervals,
assesses applicability domain, and supports nearest-neighbour analog search for
virtual screening.

## Background

Acetylcholinesterase inhibition is central to the treatment and prevention of Alzheimer's
disease. Predicting pIC50 from molecular structure allows early-stage virtual screening
of compound libraries before expensive wet-lab experiments. This pipeline is designed
not just for accuracy but for reliability — incorporating uncertainty quantification,
domain applicability checks, and interpretable analog comparison.

## Dataset

- Source: ChEMBL REST API (target CHEMBL220, standard type IC50)
- Raw records fetched: 1,200
- After cleaning and consensus filtering: 790 compounds
- pIC50 range: 3.0 to 9.52 (mean 6.02, std 1.37)
- Duplicate handling: per-compound median pIC50, compounds with std > 1.0 removed
- Outlier filtering: pIC50 outside [3.0, 12.5] removed

## Pipeline Overview

1. Fetch IC50 bioactivity data from ChEMBL API with pagination and rate limiting
2. Clean and curate data — filter by standard type, relation, and units
3. Convert IC50 (nM) to pIC50 = 9 - log10(IC50)
4. Duplicate consensus filtering — median aggregation, std-based exclusion
5. Compute sample weights based on measurement count and consistency
6. Featurise each molecule using RDKit — 26 physicochemical descriptors, 256-bit Morgan fingerprints (radius 2), 167-bit MACCS keys (449 features total)
7. Lipinski rule-of-five analysis per compound
8. Murcko scaffold extraction for scaffold-aware splitting
9. Variance threshold and correlation filtering (threshold 0.98) — 449 → 404 features
10. Repeated scaffold-aware benchmark across 4 models (3 repeats)
11. Hyperparameter tuning via RandomizedSearchCV (12 iterations, 3-fold CV)
12. Conformal prediction calibration for 90% and 95% interval coverage
13. Weighted ensemble of top 3 models
14. Applicability domain assessment — Tanimoto similarity threshold + descriptor range profiling
15. Permutation feature importance
16. Nearest-neighbour analog search from training set
17. Full screening score per compound with penalty terms
18. Export model artifact, dataset CSV, benchmark summary, and experiment log JSON

## Molecular Features

| Feature Group | Description | Dimensions |
|---|---|---|
| RDKit Descriptors | MolWt, LogP, TPSA, H-donors/acceptors, rings, BertzCT, Kappa indices, etc. | 26 |
| Morgan Fingerprints | Circular fingerprints, radius=2, 256 bits | 256 |
| MACCS Keys | Structural key fingerprints | 167 |
| Total (pre-filter) | | 449 |
| Total (post-filter) | After variance + correlation filtering | 404 |

## Tech Stack

| Library | Purpose |
|---|---|
| RDKit | Molecular featurisation, fingerprints, scaffolds, Lipinski |
| scikit-learn | Models, feature selection, cross-validation, tuning |
| requests | ChEMBL REST API data fetching |
| joblib | Model artifact serialisation |
| pandas / numpy | Data processing |
| matplotlib | EDA and diagnostic plots |
