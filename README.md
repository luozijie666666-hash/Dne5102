# GAMs
gam_comparison_study.py# Multi-GAM Model Comparison for Biomass GasificationA comprehensive framework for comparing variants of Generalized Additive Models (GAMs) (with/without monotonic constraints) on biomass gasification data, focusing on regression performance, physical consistency, and interpretability.
Project Overview
This repository implements a systematic comparison of 10 variants of Generalized Additive Models (5 base GAM models × 2 versions: standard/monotonic constrained) for regression tasks on biomass gasification datasets. The core goals include:
Evaluate regression performance of different GAM models on key gasification targets (H₂, CO, CO₂, CH₄, gas yield GY).
Verify physical consistency of model predictions (aligning with domain knowledge, e.g., temperature's monotonic impact on H₂).
Compare model interpretability (feature importance) and training efficiency.
Provide publication-quality visualizations and structured result outputs.
Key Features
End-to-end data pipeline: Excel data loading, categorical feature encoding, outlier filtering, feature standardization.
Support for 5 base GAM models (EBM, pyGAM, GA2M, NAM, GAMM) and their monotonic-constrained variants.
Bayesian hyperparameter optimization (Optuna) for model tuning (optional).
Multi-dimensional evaluation: RMSE/MAE/R²/MAPE (regression), physical conformity rate, training time.
Publication-ready visualizations: feature importance, prediction scatter plots, residual analysis, model comparison boxplots/heatmaps.
Structured result saving (Excel/PNG/PDF) for reproducibility.
Environment Setup
Prerequisites
Python version: 3.8 ~ 3.10 (compatibility with older packages is recommended)
CUDA (optional): For accelerating NAM model training with GPU (Torch GPU version)
Step 1: Create Virtual Environment (Recommended)
bash
运行
# Using Conda (Anaconda/Miniconda required)
conda create -n gam_gasification python=3.9
conda activate gam_gasification

# Or using venv
python -m venv gam_gasification
# Windows activation: gam_gasification\Scripts\activate
# Linux/Mac activation: source gam_gasification/bin/activate
Step 2: Install Dependencies
Install core dependencies:
bash
运行
pip install pandas numpy matplotlib scipy scikit-learn joblib openpyxl
pip install interpret pygam torch
Install optional dependency (Bayesian hyperparameter optimization):
bash
运行
pip install optuna
Data Preparation
Prepare the biomass gasification dataset in Excel format (default filename: Data of biomass gasification.xlsx).
The Excel file must contain two sheets:
data of %: Features (temperature, ER, Steam/Biomass, C/H/O/Ash/Moisture, Bed material) and gas composition targets (H₂/CO/CO₂/CH₄, unit: %vol N₂ free).
data of GY: Same features as above + gas yield target (GY, unit: Nm³/kg daf).
Ensure the feature columns match the naming convention in CFG class (adjust CFG in code if your column names differ):
Temperature (T): Convert to Kelvin (code auto-converts °C to K).
Categorical feature: Bed material (values: 1/2/3/4, auto-converted to one-hot dummies).
How to Run
1. Configure Parameters (Optional)
Modify the CFG class in gam_comparison_study.py to adjust key settings:
python
运行
class CFG:
    DATA_PATH = "Data of biomass gasification.xlsx"  # Path to your Excel data
    OUTPUT_DIR = "results_gam_comparison"            # Result save directory
    N_FOLDS = 5                                      # K-fold cross-validation
    RANDOM_STATE = 42                                # Random seed for reproducibility
    OPTUNA_N_TRIALS = 30                             # Optuna trial number (if enabled)
    USE_BAYESIAN_OPT = False                         # Enable/disable hyperparameter optimization
2. Execute the Code
Run the main script directly:
bash
运行
python gam_comparison_study.py
Output Results
All results are saved to the directory specified by CFG.OUTPUT_DIR (default: results_gam_comparison), including:
1. Numerical Results (Excel)
Model evaluation metrics (RMSE/MAE/R²/MAPE/physical conformity rate) across folds/targets.
Feature importance scores for each model/target.
Bayesian optimization results (best hyperparameters + RMSE, if enabled).
Prediction vs. actual values for train/test sets.
2. Visualizations (PNG/PDF)
Visualization Type	Description
Feature Importance Bar Chart	Relative importance of each feature for a model/target.
Prediction Scatter Plot	Train/test predicted vs. measured values (with R²/RMSE annotations).
Residual Analysis	Residual vs. predicted values + residual distribution (normal fit).
Model Comparison Boxplot	Distribution of RMSE/R²/physical conformity across models for each target.
Performance Heatmap	Aggregated metric (mean RMSE/R²) across models/targets (publication-ready).
Model Descriptions
The framework compares 10 model variants (5 base models × standard/monotonic constrained):
Base Model	Full Name	Monotonic Variant	Key Notes
EBM	Explainable Boosting Machine	EBM_Mono	Glassbox model (InterpretML)
pyGAM	Classical GAM with splines	pyGAM_Mono	Spline-based additive model (pyGAM)
GA2M	GAM with Pairwise Interactions	GA2M_Mono	EBM variant with feature interactions
NAM	Neural Additive Models	NAM_Mono	Neural network-based additive model (Torch)
GAMM	Generalized Additive Mixed Models	GAMM_Mono	GAM + random intercept (pyGAM extension)
Monotonic Constraints
Monotonic variants apply domain-specific constraints (e.g., temperature ↑ → H₂ ↑, ER ↑ → CO ↓) to ensure predictions align with physical laws of biomass gasification. The constraint logic is defined in get_constraints_vector().
Evaluation Metrics
Metric	Description
RMSE	Root Mean Squared Error (lower = better)
MAE	Mean Absolute Error (lower = better)
R²	Coefficient of Determination (1 = perfect prediction)
MAPE	Mean Absolute Percentage Error (lower = better)
Physical Rate	Percentage of samples satisfying monotonic constraints (100% = fully conform)
Fit Time	Model training time (lower = more efficient)
References
Lou, Y., et al. (2012). Intelligible Models for Classification and Regression. KDD 2012.
Lou, Y., et al. (2013). Accurate Intelligible Models with Pairwise Interactions. KDD 2013.
Hastie, T., & Tibshirani, R. (1990). Generalized Additive Models. Chapman & Hall.
Agarwal, R., et al. (2021). Neural Additive Models: Interpretable Machine Learning with Neural Nets. NeurIPS 2021.
License
This project is intended for academic/research use only. For commercial use, please contact the authors.
Notes
Ensure all dependencies are installed correctly (missing packages will skip corresponding models with a [SKIP] log).
For large datasets, NAM model training may be slower on CPU—enable CUDA for GPU acceleration.
Adjust matplotlib rcParams in the code to customize visualization styles for publications.# Multi-GAM Model Comparison for Biomass Gasification
A comprehensive framework for comparing variants of Generalized Additive Models (GAMs) (with/without monotonic constraints) on biomass gasification data, focusing on regression performance, physical consistency, and interpretability.
Project Overview
This repository implements a systematic comparison of 10 variants of Generalized Additive Models (5 base GAM models × 2 versions: standard/monotonic constrained) for regression tasks on biomass gasification datasets. The core goals include:
Evaluate regression performance of different GAM models on key gasification targets (H₂, CO, CO₂, CH₄, gas yield GY).
Verify physical consistency of model predictions (aligning with domain knowledge, e.g., temperature's monotonic impact on H₂).
Compare model interpretability (feature importance) and training efficiency.
Provide publication-quality visualizations and structured result outputs.
Key Features
End-to-end data pipeline: Excel data loading, categorical feature encoding, outlier filtering, feature standardization.
Support for 5 base GAM models (EBM, pyGAM, GA2M, NAM, GAMM) and their monotonic-constrained variants.
Bayesian hyperparameter optimization (Optuna) for model tuning (optional).
Multi-dimensional evaluation: RMSE/MAE/R²/MAPE (regression), physical conformity rate, training time.
Publication-ready visualizations: feature importance, prediction scatter plots, residual analysis, model comparison boxplots/heatmaps.
Structured result saving (Excel/PNG/PDF) for reproducibility.
Environment Setup
Prerequisites
Python version: 3.8 ~ 3.10 (compatibility with older packages is recommended)
CUDA (optional): For accelerating NAM model training with GPU (Torch GPU version)
Step 1: Create Virtual Environment (Recommended)
bash
运行
# Using Conda (Anaconda/Miniconda required)
conda create -n gam_gasification python=3.9
conda activate gam_gasification

# Or using venv
python -m venv gam_gasification
# Windows activation: gam_gasification\Scripts\activate
# Linux/Mac activation: source gam_gasification/bin/activate
Step 2: Install Dependencies
Install core dependencies:
bash
运行
pip install pandas numpy matplotlib scipy scikit-learn joblib openpyxl
pip install interpret pygam torch
Install optional dependency (Bayesian hyperparameter optimization):
bash
运行
pip install optuna
Data Preparation
Prepare the biomass gasification dataset in Excel format (default filename: Data of biomass gasification.xlsx).
The Excel file must contain two sheets:
data of %: Features (temperature, ER, Steam/Biomass, C/H/O/Ash/Moisture, Bed material) and gas composition targets (H₂/CO/CO₂/CH₄, unit: %vol N₂ free).
data of GY: Same features as above + gas yield target (GY, unit: Nm³/kg daf).
Ensure the feature columns match the naming convention in CFG class (adjust CFG in code if your column names differ):
Temperature (T): Convert to Kelvin (code auto-converts °C to K).
Categorical feature: Bed material (values: 1/2/3/4, auto-converted to one-hot dummies).
How to Run
1. Configure Parameters (Optional)
Modify the CFG class in gam_comparison_study.py to adjust key settings:
python
运行
class CFG:
    DATA_PATH = "Data of biomass gasification.xlsx"  # Path to your Excel data
    OUTPUT_DIR = "results_gam_comparison"            # Result save directory
    N_FOLDS = 5                                      # K-fold cross-validation
    RANDOM_STATE = 42                                # Random seed for reproducibility
    OPTUNA_N_TRIALS = 30                             # Optuna trial number (if enabled)
    USE_BAYESIAN_OPT = False                         # Enable/disable hyperparameter optimization
2. Execute the Code
Run the main script directly:
bash
运行
python gam_comparison_study.py
Output Results
All results are saved to the directory specified by CFG.OUTPUT_DIR (default: results_gam_comparison), including:
1. Numerical Results (Excel)
Model evaluation metrics (RMSE/MAE/R²/MAPE/physical conformity rate) across folds/targets.
Feature importance scores for each model/target.
Bayesian optimization results (best hyperparameters + RMSE, if enabled).
Prediction vs. actual values for train/test sets.
2. Visualizations (PNG/PDF)
Visualization Type	Description
Feature Importance Bar Chart	Relative importance of each feature for a model/target.
Prediction Scatter Plot	Train/test predicted vs. measured values (with R²/RMSE annotations).
Residual Analysis	Residual vs. predicted values + residual distribution (normal fit).
Model Comparison Boxplot	Distribution of RMSE/R²/physical conformity across models for each target.
Performance Heatmap	Aggregated metric (mean RMSE/R²) across models/targets (publication-ready).
Model Descriptions
The framework compares 10 model variants (5 base models × standard/monotonic constrained):
Base Model	Full Name	Monotonic Variant	Key Notes
EBM	Explainable Boosting Machine	EBM_Mono	Glassbox model (InterpretML)
pyGAM	Classical GAM with splines	pyGAM_Mono	Spline-based additive model (pyGAM)
GA2M	GAM with Pairwise Interactions	GA2M_Mono	EBM variant with feature interactions
NAM	Neural Additive Models	NAM_Mono	Neural network-based additive model (Torch)
GAMM	Generalized Additive Mixed Models	GAMM_Mono	GAM + random intercept (pyGAM extension)
Monotonic Constraints
Monotonic variants apply domain-specific constraints (e.g., temperature ↑ → H₂ ↑, ER ↑ → CO ↓) to ensure predictions align with physical laws of biomass gasification. The constraint logic is defined in get_constraints_vector().
Evaluation Metrics
Metric	Description
RMSE	Root Mean Squared Error (lower = better)
MAE	Mean Absolute Error (lower = better)
R²	Coefficient of Determination (1 = perfect prediction)
MAPE	Mean Absolute Percentage Error (lower = better)
Physical Rate	Percentage of samples satisfying monotonic constraints (100% = fully conform)
Fit Time	Model training time (lower = more efficient)
References
Lou, Y., et al. (2012). Intelligible Models for Classification and Regression. KDD 2012.
Lou, Y., et al. (2013). Accurate Intelligible Models with Pairwise Interactions. KDD 2013.
Hastie, T., & Tibshirani, R. (1990). Generalized Additive Models. Chapman & Hall.
Agarwal, R., et al. (2021). Neural Additive Models: Interpretable Machine Learning with Neural Nets. NeurIPS 2021.
License
This project is intended for academic/research use only. For commercial use, please contact the authors.
Notes
Ensure all dependencies are installed correctly (missing packages will skip corresponding models with a [SKIP] log).
For large datasets, NAM model training may be slower on CPU—enable CUDA for GPU acceleration.
Adjust matplotlib rcParams in the code to customize visualization styles for publications.
