# Physically Constrained GAMs for Interpretable Biomass Gasification Prediction

This repository contains the complete Python workflow used to compare generalized additive model (GAM) family methods for biomass gasification prediction, with and without monotonic constraints.

The workflow evaluates four additive model families:

- **GAM** implemented with `pyGAM`
- **EBM** implemented with `InterpretML`
- **GA2M** implemented with `InterpretML`
- **NAM** implemented with `PyTorch`

Each model is evaluated in unconstrained and monotonic forms. The prediction targets are H₂, CO, CO₂, CH₄, and gas yield (GY).

## Main functions

The code performs the following steps:

1. Loads and merges the gas composition and gas yield worksheets.
2. Converts gasification temperature from °C to K.
3. Removes extreme input outliers using a z-score threshold of ±5.
4. One-hot encodes the four bed material categories.
5. Standardizes continuous variables using training-set statistics only.
6. Evaluates eight model variants using five-fold cross-validation.
7. Calculates R², RMSE, MAE, MAPE, and physical trend conformity rate (PhysRate).
8. Selects the best split for each model–target combination using the highest test R².
9. Saves fitted models, scalers, predictions, tables, and publication-quality figures.
10. Refits GA2M-Mono on all available preprocessed samples for intrinsic interpretability analysis.
11. Generates term importance, main-effect, ICE/PDP, and pairwise interaction results.

## Repository structure

A recommended repository structure is:

```text
.
├── GAMs.py
├── Data of biomass gasification.xlsx
├── README.md
└── results_gam_comparison/          # Generated after execution
```

Rename the provided Python code to `GAMs.py`, or update the command below to match the actual script name.

## Requirements

Python 3.10 or later is recommended.

Install the required packages with:

```bash
pip install numpy pandas scipy matplotlib scikit-learn openpyxl joblib interpret pygam torch
```

A recent version of `InterpretML` is recommended because the interpretation workflow uses fitted-term metadata and term-level evaluation functions.

GPU acceleration is optional. NAM automatically uses CUDA when it is available and otherwise runs on the CPU.

## Input data

The default input file is:

```text
Data of biomass gasification.xlsx
```

It must be placed in the same directory as the Python script unless `CFG.DATA_PATH` is changed.

### Required worksheets

The workbook must contain:

```text
data of %
data of GY
```

The two worksheets are merged according to their row order. Their corresponding records should therefore remain aligned.

### Required input columns

```text
(x1)T [ºC]
(x2)ER [-]
(x3)Steam/Biomass
(x4)C [%wt db]
(x5)H [%wt db]
(x6)O [%wt db]
(x7)Ash [%wt db]
(x8)Moisture [%wt]
Bed material
```

`Bed material` should use the category labels `1`, `2`, `3`, and `4`.

### Required target columns

```text
H2 [%vol N2 free]
CO [%vol N2 free]
CO2 [%vol N2 free]
CH4 [%vol N2 free]
GY [Nm3/kg daf]
```

Rows with a missing value for a given target are excluded only from the model developed for that target.

## Configuration

The main settings are defined in the `CFG` class:

```python
class CFG:
    DATA_PATH = "Data of biomass gasification.xlsx"
    OUTPUT_DIR = "results_gam_comparison"
    SHEET_PERCENT = "data of %"
    SHEET_GY = "data of GY"
    N_FOLDS = 5
    RANDOM_STATE = 42
```

Change these values directly in the script when using a different file name, output directory, worksheet name, number of folds, or random seed.

The current script does not use command-line arguments.

## Running the workflow

Run the complete analysis with:

```bash
python GAMs.py
```

The script automatically checks whether the packages required by each model are available. Missing model packages cause the corresponding model variants to be skipped. Install all packages listed above to reproduce the full eight-model comparison.

The complete workflow includes repeated cross-validation, final model fitting, and high-resolution figure generation. Runtime depends on the processor, memory, and availability of GPU acceleration.

## Monotonic constraints

Directional constraints are imposed only on variable–target relationships supported by the predefined physical priors.

| Variable | H₂ | CO | CO₂ | CH₄ |
|---|---:|---:|---:|---:|
| T | Increasing | Increasing | Decreasing | Unconstrained |
| ER | Decreasing | Decreasing | Increasing | Decreasing |
| Moisture | Increasing | Decreasing | Increasing | Increasing |
| S/B | Unconstrained | Unconstrained | Unconstrained | Unconstrained |

No monotonic constraints are imposed on C, H, O, Ash, bed material, or GY.

For EBM-Mono and GA2M-Mono, constraints are applied directly to the corresponding main-effect terms. Pairwise interactions remain unconstrained. NAM-Mono uses post-training isotonic correction and is therefore not an end-to-end monotonic neural architecture.

## PhysRate

PhysRate evaluates whether the complete model follows all prescribed local response directions around each evaluated sample.

For every constrained feature, the code estimates local derivatives using central finite differences over ten perturbation scales. A sample is counted as conforming only when every constrained derivative satisfies its required direction at every evaluated scale.

A PhysRate of 100% means that all evaluated samples satisfy all prescribed local directional trends. It does not imply compliance with every thermodynamic, kinetic, or conservation constraint of biomass gasification.

## Cross-validation and model selection

The workflow uses shuffled five-fold cross-validation with a fixed random seed.

For every target and model:

- approximately 80% of the samples are used for training in each split;
- continuous variables are standardized using training-set statistics only;
- the same split is used for all compared models;
- mean cross-validation performance is reported;
- the split with the highest test R² is recorded as the best split.

The best split is used to report prediction potential and to generate the corresponding measured-versus-predicted results.

## Interpretation workflow

GA2M-Mono is refitted on all available preprocessed samples for H₂, CO, CO₂, and CH₄. This full-data refit is used only for interpretation.

The workflow exports:

- mean absolute importance of main-effect and pairwise terms;
- centered main-effect functions;
- ICE curves for 50 reproducibly selected observations;
- PDP curves averaged over all observations;
- isolated pairwise interaction contribution surfaces;
- rankings of retained continuous–continuous interaction terms.

The pairwise surfaces exclude the corresponding main effects and other model terms. They therefore represent the additional contribution associated with the selected variable pair.

## Output files

All results are written to:

```text
results_gam_comparison/
```

Important outputs include:

```text
results_gam_comparison/
├── cv_results_raw.xlsx
├── cv_results_aggregated.xlsx
├── comprehensive_fold_summary.xlsx
├── manuscript_tables/
│   ├── Table3_Predictive_Performance.xlsx
│   ├── Table3_Predictive_Performance.csv
│   └── README_Table3.txt
├── manuscript_figures/
│   ├── Fig1_feature_importance_GA2M_Mono_four_targets.*
│   ├── Fig2_shape_functions_biomass.*
│   ├── Fig3_shape_functions_operating.*
│   ├── Fig4_pairwise_interactions.*
│   ├── FigS1_predicted_vs_measured_best_split_GAMs_Mono.*
│   ├── FigS2_ICE_PDP_biomass.*
│   └── FigS3_ICE_PDP_operating.*
├── manuscript_excels/
│   └── Data corresponding to the manuscript figures
├── H2/
├── CO/
├── CO2/
├── CH4/
└── GY/
```

Each target directory may contain:

- saved models and scalers;
- interactive prediction scripts;
- best-split prediction data;
- prediction scatter plots;
- residual plots;
- model-specific interpretation files.

Most figures are saved in PNG, PDF, and SVG formats. Numerical data underlying the principal figures are also exported to Excel.

## Using a saved prediction model

The workflow generates model-specific scripts in:

```text
results_gam_comparison/<TARGET>/prediction_scripts/
```

For example:

```bash
python results_gam_comparison/H2/prediction_scripts/predict_GA2M_Mono_H2.py
```

The script requests T, ER, S/B, C, H, O, Ash, Moisture, and bed material, then loads the corresponding saved model and scaler to generate a prediction.

Temperature must be entered in K, and bed material must be an integer from 1 to 4.

## Reproducibility notes

- The global random seed is fixed at `42`.
- Cross-validation uses shuffled `KFold`.
- The same data splits are used for all models of a given target.
- Training-set statistics are used for scaling within each split.
- ICE samples are selected with a local random generator using the same fixed seed.
- Available model variants depend on the installed packages and their versions.

Small numerical differences may occur across operating systems, package versions, processor architectures, and CPU/GPU execution.

## Method references

- Hastie, T., and Tibshirani, R. *Generalized Additive Models*. Chapman & Hall, 1990.
- Lou, Y., Caruana, R., and Gehrke, J. “Intelligible Models for Classification and Regression.” KDD, 2012.
- Lou, Y., Caruana, R., Gehrke, J., and Hooker, G. “Accurate Intelligible Models with Pairwise Interactions.” KDD, 2013.
- Agarwal, R. et al. “Neural Additive Models: Interpretable Machine Learning with Neural Nets.” NeurIPS, 2021.

## Citation

When using this code, please cite the associated manuscript:

> **Physically constrained generalized additive models for interpretable biomass gasification prediction**

Full bibliographic information can be added here after publication.

## Contact

For questions about the implementation or reproduction of the results, please contact the corresponding authors listed in the associated manuscript.
