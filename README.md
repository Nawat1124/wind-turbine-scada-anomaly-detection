# Wind Turbine SCADA Anomaly Detection

## Project overview
Normal-only anomaly detection for early fault warning in wind turbine SCADA data.

This project uses CARE Wind Farm B and treats early fault warning as an event-level alarm problem. The goal is not to classify specific fault types, but to learn normal turbine operating patterns and identify deviations during prediction periods.

## Data availability
The raw CARE Wind Farm B dataset is not included in this repository. Users should download the CARE benchmark data from the original source and place the Wind Farm B files in the expected local data directory before running the notebooks.

Large intermediate files such as `.parquet`, `.pkl`, model outputs, figures, and result folders are excluded from version control.
## Key challenge
The raw CARE Wind Farm B event files contain duplicated physical SCADA sequences across sessions. Directly merging the files can cause train-prediction leakage. This project reconstructs an asset-level physical timeline and keeps row-level lineage mappings to preserve event correspondence.

## Workflow
1. `notebooks/01_pipeline_reconstruction_v3.ipynb`
2. `notebooks/02_data_preprocessing.ipynb`
3. `notebooks/03_eda_normal_operating_space.ipynb`
4. `notebooks/04a_final_if_vs_lstm.ipynb`
5. `notebooks/04b_lstm_threshold_comparison.ipynb`

## Methods
- Leakage-aware data reconstruction
- Safe normal training-data definition
- Feature transformation and physical-family grouping
- KMeans-state metadata and PCA diagnostics
- LSTM autoencoder anomaly scoring
- Isolation Forest baseline
- Global / per-asset / KMeans-state threshold comparison
- CARE event-level evaluation

## Main results
| Experiment | Main finding |
|---|---|
| IF vs LSTM-AE | IF detected more anomaly events but produced more false positives. |
| Threshold calibration | KMeans-state q90 gave the best false-alarm control and CARE balance. |
| Event-level evaluation | CARE metrics were used instead of only point-level or window-level accuracy. |

## Repository structure
```text
.
├── README.md
├── .gitignore
├── requirements.txt
├── notebooks/
│   ├── 01_pipeline_reconstruction_v3.ipynb
│   ├── 02_data_preprocessing.ipynb
│   ├── 03_eda_normal_operating_space.ipynb
│   ├── 04a_final_if_vs_lstm.ipynb
│   └── 04b_lstm_threshold_comparison.ipynb
└── docs/
    ├── 01_reconstruction_pipeline_overview.md
    ├── 02_data_preprocessing_overview.md
    └── 03_eda_normal_operating_space_overview.md
```
