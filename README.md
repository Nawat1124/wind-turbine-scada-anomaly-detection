# Wind Turbine SCADA Anomaly Detection

## Project overview
Normal-only anomaly detection for early fault warning in wind turbine SCADA data.

## Key challenge
The raw CARE Wind Farm B event files contain duplicated physical SCADA sequences across sessions.
This project reconstructs an asset-level physical timeline to reduce train-prediction leakage.

## Workflow
1. pipeline_reconstruction_v3.ipynb
2. 02_data_preprocessing.ipynb
3. 03_eda_normal_operating_space.ipynb
4. 04b_lstm_threshold_comparison.ipynb

## Methods
- Leakage-aware data reconstruction
- Safe normal training-data definition
- Feature transformation and physical-family grouping
- KMeans-state metadata and PCA diagnostics
- LSTM autoencoder anomaly scoring
- Global / per-asset / KMeans-state threshold comparison
- CARE event-level evaluation

## Main result
The small LSTM-AE with KMeans-state q90 threshold achieved the best balance between anomaly coverage and false-alarm control in the current experiment.

## Repository structure
Explain what each notebook and docs file does.
