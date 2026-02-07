Multi-GAM Model Comparison for Biomass Gasification
(With and Without Monotonic Constraints)
This project implements a comprehensive comparison of 10 variants of Generalized Additive Models (GAMs) (5 base GAM models + their monotonic constrained versions) for predictive modeling of biomass gasification processes. The core focus is on balancing predictive accuracy, physical conformity (alignment with chemical engineering physical laws), and model interpretability for key gasification output variables.
Project Overview
The study aims to evaluate the performance of state-of-the-art GAM-based models in predicting critical biomass gasification outputs (H₂, CO, CO₂, CH₄, and gas yield GY). Monotonic constraints are integrated into each model to enforce compliance with the physical trends of the gasification process, addressing the issue of physically inconsistent predictions from pure data-driven models. The project includes full-automated pipelines for data preprocessing, 5-fold cross-validation, model training, performance evaluation, publication-quality visualization, and optimal model selection.
Models Compared
5 base GAM models with two versions each (standard + monotonic constrained) for a total of 10 model variants:
表格
Base Model	Monotonic Version	Abbreviation	Dependencies/Libraries	Key Characteristics
Explainable Boosting Machine	EBM with Monotonic Constraints	EBM/EBM_Mono	interpret (InterpretML)	Glassbox model, additive structure, high interpretability
Classical GAM with Splines	pyGAM with Monotonic Constraints	pyGAM/pyGAM_Mono	pygam	Traditional spline-based GAM, flexible non-linear fitting
GA²M (GAMs with Pairwise Interactions)	GA2M with Monotonic Constraints	GA2M/GA2M_Mono	interpret (InterpretML)	EBM extension with pairwise feature interactions
Neural Additive Models	NAM with Monotonic Constraints	NAM/NAM_Mono	torch (PyTorch)	Neural network-based additive model, combines deep learning with interpretability
Generalized Additive Mixed Models	GAMM with Monotonic Constraints	GAMM/GAMM_Mono	pygam	GAM extension with random intercepts for categorical features
Environment Setup
Dependencies
Install all required Python packages via pip:
bash
运行
pip install numpy pandas scikit-learn matplotlib pygam interpret torch scipy openpyxl joblib seaborn
Python Version: 3.8+ recommended
GPU Support: Optional (for NAM model acceleration, requires CUDA-enabled PyTorch)
Data Preparation
Input Data File
The project uses a single Excel file: Data of biomass gasification.xlsx (configured in the CFG.DATA_PATH parameter) with two sheets:
data of %: Contains molar concentration data for H₂, CO, CO₂, CH₄ (N₂ free, vol%)
data of GY: Contains gas yield (GY) data (Nm³/kg daf)
Features & Target Variables
Predictor Features
8 Continuous Features: T [K], ER [-], Steam/Biomass (S/B) [-], C [wt%], H [wt%], O [wt%], Ash [wt%], Moisture [wt%]
1 Categorical Feature: Bed material (4 categories, one-hot encoded to Bed_1/Bed_2/Bed_3/Bed_4)
Target Variables
H₂, CO, CO₂, CH₄ (all vol% N₂ free), and GY (Nm³/kg daf) — the core output metrics of biomass gasification.
Data Preprocessing Pipeline
The project automatically executes the following preprocessing steps:
Sheet merging and column renaming
Unit conversion (temperature from °C to K)
One-hot encoding for categorical bed material
Z-score outlier filtering (threshold = 5.0)
Standardization (Z-score) for continuous features
Target-wise data splitting (removal of NaN/infinite values)
How to Run
Place the biomass gasification Excel data file in the project root directory (matching CFG.DATA_PATH)
Run the main script directly:
python
运行
python gam_comparison_biomass_gasification.py
All experimental parameters are configurable via the CFG class:
N_FOLDS: 5-fold cross-validation (default)
RANDOM_STATE: 42 (for reproducibility)
OUTPUT_DIR: results_gam_comparison (default output directory)
TARGETS_MAP: Maps target abbreviations to full names/units
Key Features
1. Multi-Metric Performance Evaluation
Models are evaluated on both train and test sets with the following metrics:
Regression accuracy: RMSE, MAE, R²
Relative error: MAPE (%)
Physical conformity: PhysRate (%) — the percentage of samples complying with pre-defined physical monotonic constraints
2. Physical Conformity Check
A custom physical consistency validation module (check_physical_conformity) verifies if model predictions follow the physical trends of biomass gasification (e.g., temperature positively correlates with H₂ yield). Monotonic constraints are defined per target variable via get_constraints_vector.
3. Publication-Quality Visualization
The project generates both PNG and PDF formats for all visualizations with a standardized publication-style plot configuration (Times New Roman font, consistent color scheme, professional axis styling). Key visualizations include:
Feature importance bar plots
Train/test prediction scatter plots (with perfect prediction line and ±10% error bands)
Residual analysis (residual vs. predicted + normal distribution fit)
Model comparison boxplots (RMSE, R², PhysRate)
Heatmaps for cross-model/target metric comparison
Feature shape functions (marginal effect of each feature on the target)
ICE/PDP plots (Individual Conditional Expectation / Partial Dependence Plots)
3D bivariate dependency surface plots (all 28 pairs of continuous features)
4. Model Deployment Utilities
Trained models and feature scalers are saved via joblib in the saved_models subdirectory
Auto-generated prediction scripts (predict_<model>_<target>.py) for standalone model inference
Inference supports direct input of raw feature values (automatic scaling/one-hot encoding)
5. Optimal Model Auto-Analysis
The project automatically identifies the best model for each target variable (highest PhysRate ≥99.9% + highest R²) and generates advanced interpretability visualizations (shape functions, ICE/PDP, 3D bivariate plots) for the optimal model.
6. Reproducible Results
Fixed random seed for all stochastic operations
5-fold cross-validation with stratified shuffling
Comprehensive raw/aggregated result tables (Excel format)
Full logging of model training/fitting time and error messages
Output Structure
All results are saved to the results_gam_comparison directory (configurable) with the following structure:
plaintext
results_gam_comparison/
├── cv_results_raw.xlsx          # Raw 5-fold cross-validation results
├── cv_results_aggregated.xlsx   # Aggregated (mean/std) model performance metrics
├── comprehensive_fold_summary.xlsx # Full train/test metric summary (per fold/target/model)
├── heatmap_*.png/pdf            # Cross-model/target heatmaps (RMSE, R², PhysRate)
├── *.comparison_boxplot.png/pdf # Target-wise model comparison boxplots
├── [H2/CO/CO2/CH4/GY]/          # Target-specific subdirectories
│   ├── saved_models/            # Trained models and scalers (joblib)
│   ├── prediction_scripts/      # Standalone prediction scripts
│   ├── *.feature_importance.png/pdf/xlsx
│   ├── *.scatter.png/pdf/xlsx
│   ├── *.residuals.png/pdf
│   ├── *.shape_functions.png/pdf/xlsx
│   ├── *.ice_pdp.png/pdf/xlsx
│   └── best_model_analysis/     # Advanced analysis for the optimal model
│       └── bivariate_3d/        # 3D bivariate dependency plots
└── ...
References
The models and methodologies are based on the following key publications:
Lou, Y., Caruana, R., & Gehrke, J. (2012). Intelligible Models for Classification and Regression. KDD 2012.
Lou, Y., Caruana, R., & Hooker, G. (2013). Accurate Intelligible Models with Pairwise Interactions. KDD 2013.
Hastie, T., & Tibshirani, R. (1990). Generalized Additive Models. Chapman & Hall.
Agarwal, R., et al. (2021). Neural Additive Models: Interpretable Machine Learning with Neural Nets. NeurIPS 2021.
Notes
Monotonic Constraints for GY: No physical constraints are applied to the GY target variable (no authoritative physical trend reports available).
Model Compatibility Check: The script automatically checks for installed dependencies and skips models with missing libraries (with clear logging).
GPU Acceleration: The NAM model uses PyTorch and automatically leverages CUDA if available (falls back to CPU otherwise).
Visualization Customization: Plot styles (font, color, size, DPI) are fully configurable via the mpl.rcParams section in the script.
Scalability: The pipeline supports easy extension to additional target variables/features by modifying the CFG class and TARGETS/CONTINUOUS_COLS parameters.
