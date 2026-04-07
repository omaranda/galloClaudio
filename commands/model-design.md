---
name: model-design
description: Design a machine learning model architecture for biodiversity or ecological data — from problem framing and feature selection through model choice, validation strategy, and deployment plan.
---

You are acting as the data-scientist agent. Design a machine learning model.

**Step 1 — Frame the problem:**
- What is the prediction target? (species presence, abundance, habitat class, anomaly detection)
- What data is available? (camera trap detections, EO indices, audio detections, survey data)
- Is this a classification, regression, clustering, or density estimation problem?
- What is the required output format? (probability, count, class label, grid map)
- What is an acceptable level of false positives vs. false negatives for the use case?

**Step 2 — Audit the available data:**
- What is the sample size? Are classes balanced?
- Is there spatial or temporal structure that will cause data leakage if ignored?
- Are there missing values? What is the missingness mechanism?
- What are the geographic and temporal extents of the data?

**Step 3 — Choose modeling approach:**

**Decision guide:**
- Ecological process with known structure → Bayesian hierarchical model (PyMC)
- Species distribution from survey + env. covariates → MaxEnt, BRT, or Bayesian SDM
- Image classification (species ID from camera trap crops) → Fine-tuned CNN (EfficientNet/ResNet via timm)
- Tabular prediction with structured features → XGBoost / LightGBM
- Temporal forecasting → LSTM, temporal fusion transformer, or ARIMA for simpler cases
- Anomaly detection in time-series → Isolation Forest, or Bayesian changepoint detection

**Step 4 — Design the feature engineering pipeline:**
- Spatial features: distance to water, elevation, habitat class from EO data
- Temporal features: day of year, hour of day (for camera traps / audio)
- Derived EO features: NDVI, NDWI, land cover class
- Normalize continuous features; encode categorical features appropriately

**Step 5 — Design the validation strategy:**
- Spatial data → Spatial block cross-validation (not random k-fold — that leaks)
- Temporal data → Forward-chaining (train on past, test on future)
- Balanced classes? If not, document class weights or resampling strategy

**Step 6 — Define success metrics:**
- Classification: AUC-ROC, precision/recall at relevant threshold, F1
- Regression: RMSE, MAE, R² with spatial breakdown
- Occupancy models: predictive performance on held-out sites

**Step 7 — Provide:**
- Model architecture diagram or description
- Feature engineering code skeleton
- Cross-validation strategy implementation
- Baseline model to beat first (simple rule or majority class)
- Deployment sketch: batch scoring script or FastAPI inference endpoint
