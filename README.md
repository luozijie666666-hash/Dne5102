# Multi-GAM Model Comparison for Biomass Gasification

This repository contains code for a systematic comparison of generalized additive models (GAMs) applied to biomass gasification product prediction. The script evaluates four GAM families—with and without monotonic constraints—via cross-validation, physical conformity assessment, interpretability analysis, and publication-ready visualization.

## Overview

The study predicts five target variables from biomass gasification experiments:

| Symbol | Description |
|--------|-------------|
| H2 | Hydrogen volume fraction (N₂-free) |
| CO | Carbon monoxide volume fraction |
| CO2 | Carbon dioxide volume fraction |
| CH4 | Methane volume fraction |
| GY | Gas yield [Nm³/kg daf] |

## Models Compared

**4 base models × 2 variants = 8 model configurations:**

| Model | Monotonic variant | Description |
|-------|-------------------|-------------|
| EBM | EBM_Mono | Explainable Boosting Machine (InterpretML) |
| pyGAM | pyGAM_Mono | Classical spline-based GAM |
| GA2M | GA2M_Mono | GA²M with pairwise interactions |
| NAM | NAM_Mono | Neural Additive Model (PyTorch) |

- Unsuffixed names: standard fitting
- `_Mono` suffix: monotonic constraints derived from physical trends in the literature

## Input Features

**Continuous (8 features, Z-score standardized before fitting):**

- `T` — Temperature [K] (converted automatically from °C)
- `ER` — Equivalence ratio
- `Steam/Biomass` — Steam-to-biomass ratio
- `C`, `H`, `O`, `Ash`, `Moisture` — Elemental composition, ash, and moisture [wt% db]

**Categorical (one-hot encoded):**

- `Bed material` — Bed material type, values 1–4

## Requirements

- Python ≥ 3.8
- Missing optional packages cause the corresponding model to be skipped automatically

| Package | Purpose | Models |
|---------|---------|--------|
| numpy, pandas, scikit-learn | Data handling and metrics | All |
| matplotlib, scipy | Plotting and statistics | All |
| openpyxl | Excel I/O | All |
| joblib | Model serialization | All |
| interpret | EBM / GA2M backend | EBM, GA2M |
| pygam | Spline GAM | pyGAM |
| torch | Neural networks | NAM |

Install example:

```bash
pip install numpy pandas scikit-learn matplotlib scipy openpyxl joblib interpret pygam torch
```

## Data File

Place the Excel workbook in the same directory as the script. Default filename:

```
Data of biomass gasification.xlsx
```

Required sheets:

| Sheet name | Content |
|------------|---------|
| `data of %` | H2, CO, CO2, CH4, and process/composition inputs |
| `data of GY` | Gas yield (GY) and corresponding inputs |

Column names must match the mappings in `merge_sheets()` (e.g. `(x1)T [ºC]`, `(x2)ER [-]`). Override the path via `CFG.DATA_PATH` if needed.

## Quick Start

1. Rename `投稿代码.txt` to `gam_comparison.py` (or run the `.txt` file directly with Python).
2. Ensure the data file is in the working directory.
3. Run:

```bash
python gam_comparison.py
```

All outputs are written to `results_gam_comparison/` (configurable via `CFG.OUTPUT_DIR`).

## Workflow

The script runs the following steps automatically:

1. **Load data** — Merge both sheets, encode bed material, apply Z-score outlier filtering (5σ threshold)
2. **Detect available models** — Load up to 8 variants depending on installed dependencies
3. **5-fold cross-validation** — Per target: RMSE, MAE, R², MAPE, and physical conformity (PhysRate)
4. **Aggregate results** — Export per-fold and summary Excel tables
5. **Train final models** — Save models; scatter and residual plots use each model's best CV fold
6. **Comparison plots** — Boxplots and heatmaps (RMSE, R², PhysRate)
7. **Best-model analysis** — Shape functions, ICE/PDP, and bivariate 3D surfaces (28 continuous-feature pairs)
8. **Fold-level summary** — Multi-sheet Excel report

## Output Structure

```
results_gam_comparison/
├── cv_results_raw.xlsx              # Per-fold raw results
├── cv_results_aggregated.xlsx       # Aggregated model × target statistics
├── comprehensive_fold_summary.xlsx  # Multi-sheet fold-level summary
├── heatmap_RMSE.png / .pdf / .xlsx
├── heatmap_R2.png / .pdf / .xlsx
├── heatmap_PhysRate.png / .pdf / .xlsx
├── {H2,CO,CO2,CH4,GY}_comparison_boxplot.png / .pdf
├── {target}/
│   ├── saved_models/                # joblib models and scalers
│   ├── prediction_scripts/          # Standalone prediction scripts
│   ├── {Model}_{target}_scatter.png / .pdf
│   ├── {Model}_{target}_residuals.png / .pdf
│   ├── {Model}_{target}_feature_importance.png / .pdf / .xlsx
│   └── best_model_analysis/         # Shape functions, ICE/PDP, 3D plots for best model
│       └── bivariate_3d/            # 28 bivariate 3D surface plots
```

## Monotonic Constraints

`_Mono` models enforce sign constraints on continuous features (`+1` increasing, `-1` decreasing, `0` unconstrained):

| Target | T | ER | Moisture |
|--------|---|----|----------|
| H2 | ↑ | ↓ | ↑ |
| CO | ↑ | ↓ | ↓ |
| CO2 | ↓ | ↑ | ↑ |
| CH4 | — | ↓ | ↑ |
| GY | — | — | — |

Steam/Biomass and elemental features (C, H, O, Ash) are unconstrained. No monotonic constraints are applied for GY.

**PhysRate** (physical conformity rate): percentage of test samples for which the numerical gradient of the model prediction aligns with the expected sign for constrained features.

## Using Prediction Scripts

A standalone script is generated for each model–target pair, e.g.:

```
results_gam_comparison/H2/prediction_scripts/predict_EBM_Mono_H2.py
```

Interactive example:

```bash
cd results_gam_comparison/H2/prediction_scripts
python predict_EBM_Mono_H2.py
```

Enter T [K], ER, Steam/Biomass, elemental composition, moisture, and bed material (1–4) when prompted.

## Configuration

Key settings in the `CFG` class:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `DATA_PATH` | `Data of biomass gasification.xlsx` | Input data path |
| `OUTPUT_DIR` | `results_gam_comparison` | Output directory |
| `N_FOLDS` | `5` | Cross-validation folds |
| `RANDOM_STATE` | `42` | Random seed |

## References

- Lou et al. (2012). *Intelligible Models for Classification and Regression.* KDD.
- Lou et al. (2013). *Accurate Intelligible Models with Pairwise Interactions.* KDD.
- Hastie & Tibshirani (1990). *Generalized Additive Models.* Chapman & Hall.
- Agarwal et al. (2021). *Neural Additive Models.* NeurIPS.

## Notes

- NAM training uses 200 epochs by default; a full run can take considerable time. CPU is used when no GPU is available.
- If a dependency is missing, the corresponding model is reported as `[SKIP]` at startup and excluded from the comparison.
- Re-running overwrites existing files in `results_gam_comparison/`; back up prior results if needed.
