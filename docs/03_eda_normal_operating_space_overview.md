```text
┌──────────────────────────────────────────────────────────────┐
│ Start: Set Paths                                             │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Load outputs from 02_data_preprocessing                      │
│                                                              │
│ - eda_base.parquet                                           │
│ - df_model.parquet                                           │
│ - model_ready_feature_list.json                              │
│ - pca_explained_variance.csv                                 │
│ - pca_loadings.csv                                           │
│                                                              │
│ eda:      560060 × 256                                       │
│ df_model: 560060 × 71                                        │
│ model features: 49                                           │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Define normal safe masks                                     │
│                                                              │
│ normal_safe_mask =                                           │
│   never_prediction_physical_row                              │
│   AND status_type_id in {0, 2}                               │
│                                                              │
│ train_normal_mask = normal_safe_mask AND split_code == 0     │
│                                                              │
│ normal safe rows: 443248                                     │
│ train normal rows: 397198                                    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Define physical feature families                             │
│                                                              │
│ 8 families:                                                  │
│ - power / energy output                                      │
│ - electrical grid current voltage                            │
│ - wind / orientation / pitch control                         │
│ - drivetrain speed torque                                    │
│ - ambient / generator / motor thermal                        │
│ - gearbox / bearing / oil thermal                            │
│ - transformer thermal                                        │
│ - vibration / structural                                     │
│                                                              │
│ Also define pruned families after redundant feature removal  │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Basic dataset overview                                       │
│                                                              │
│ - number of rows                                             │
│ - asset IDs                                                  │
│ - status_type_id counts                                      │
│ - never_prediction_physical_row counts                       │
│ - split_code counts                                          │
│ - final model feature list                                   │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Gap diagnostics                                              │
│                                                              │
│ For each asset:                                              │
│ - sort by physical_order_index                               │
│ - compute time difference                                    │
│ - expected step = 10 minutes                                 │
│ - classify gaps as small or big                              │
│                                                              │
│ Output: gap_diag + summaries                                 │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Asset-level density plots                                    │
│                                                              │
│ Variables:                                                   │
│ - sensor_47_avg                                              │
│ - sensor_40_avg                                              │
│ - sensor_38_avg                                              │
│ - sensor_51_avg                                              │
│                                                              │
│ Purpose: show turbine-level normal baseline differences      │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ IQR outlier diagnostics                                      │
│                                                              │
│ Fit rows: normal_safe_mask                                   │
│ Fitting rows: 443248                                         │
│ Feature columns: 214                                         │
│ K_IQR = 3.0                                                  │
│ Per asset thresholds                                         │
│                                                              │
│ Only flag, no clipping                                       │
│                                                              │
│ Output:                                                      │
│ - feature__iqr_outlier flags                                 │
│ - row_iqr_outlier_n                                          │
│ - row_iqr_outlier_any                                        │
│ - iqr_outlier_summary                                        │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Distribution histograms for selected variables               │
│                                                              │
│ Examples:                                                    │
│ - sensor_35_avg                                              │
│ - sensor_34_avg                                              │
│ - sensor_39_avg                                              │
│ - sensor_36_avg                                              │
│ - sensor_37_avg                                              │
│ - sensor_47_avg                                              │
│ - sensor_10_avg                                              │
│ - sensor_32_avg                                              │
│ - sensor_9_avg                                               │
│                                                              │
│ Purpose: inspect skewness, tails, multimodality              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Within-family correlation analysis                           ││                                                              │
│ Method: Pearson correlation                                  │
│ Threshold: abs_corr >= 0.95                                  │
│                                                              │
│ Output:                                                      │
│ - all within-family pairs                                    │
│ - high correlation pairs                                     │
│ - family-level correlation summary                           │
│                                                              │
│ Purpose: justify redundancy pruning                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Correlation heatmaps after pruning                           │
│                                                              │
│ 1. within-family heatmaps                                    │
│                                                              │
│ 2. between-family correlation                                │
│                                                              │
│ Purpose: show remaining subsystem coupling                   │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Temporal diagnostics                                         │
│                                                              │
│ ACF of power_62_avg on Asset 2                               │
│                                                              │
│ Max lag = 50 steps = 500 minutes                             │
│ Purpose: support 4-hour window length                        │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Granger causality diagnostics                                │
│                                                              │
│ Target: power_62_avg                                         │
│ Asset: 2                                                     │
│ Max lag: 24 steps                                            │
│ Candidate variables:                                         │
│ - wind_speed_59_avg                                          │
│ - sensor_16_avg                                              │
│ - sensor_12_avg                                              │
│ - sensor_34_avg                                              │
│ - sensor_40_avg                                              │
│ - sensor_54_avg                                              │
│                                                              │
│ Purpose: check short-lag predictive relationships            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ KMeans operating-state diagnostics                           │
│                                                              │
│ Use train normal rows                                        │
│                                                              │
│ Output:                                                      │
│ - number of rows per KMeans state                            │
│ - median / q95 / q99 / max center distance                   │
│ - effective number of states                                 │
│                                                              │
│ Occupied states: 6                                           │
│ Effective number of states: about 5.83                       │
│                                                              │
│ Purpose: justify operating-state-aware threshold             │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ KMeans diagnostic plots                                      │
│                                                              │
│ - number of normal rows per state                            │
│ - q99 center distance by state                               │
│ - compare with global q99 distance                           │
│                                                              │
│ Purpose: show unequal variability across normal states       │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ UMAP normal operating-space visualization                    │
│                                                              │
│ Input: 30,000 sampled train-normal rows                      │
│ Features: 49 model features                                  │
│ Scaling: StandardScaler                                      │
│ UMAP: n_neighbors = 30, min_dist = 0.1                       │
│                                                              │
│ Colored by:                                                  │
│ - asset_id                                                   │
│ - power_62_avg                                               │
│ - wind_speed_59_avg                                          │
│ - sensor_10_avg                                              │
│ - sensor_36_avg                                              │
│ - sensor_57_avg                                              │
│ - kmeans_state                                               │
│                                                              │
│ Purpose: visualize normal-state structure                    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA explained variance                                       │
│                                                              │
│ Display pca_explained table                                  │
│ Plot explained variance ratio + cumulative variance          │
│                                                              │
│ Purpose: summarize global low-dimensional structure          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA scatter plot                                             │
│                                                              │
│ Plot PC1 vs PC2                                              │
│ Colored by KMeans state                                      │
│                                                              │
│ Purpose: check whether KMeans states align with PCA space    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA loading interpretation                                   │
│                                                              │
│ Map each feature to physical family                          │
│                                                              │
│ For PC1 ~ PC5:                                               │
│ - list top loading features                                  │
│ - plot loading bars                                          │
│                                                              │
│ Purpose: interpret PCs physically                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA family contribution                                      │
│                                                              │
│ For each PC:                                                 │
│ - square each loading                                        │
│ - sum squared loadings by family                             │
│ - plot family contribution for PC1 ~ PC10                    │
│                                                              │
│ Purpose: show which subsystem drives each PC                 │
└──────────────────────────────────────────────────────────────┘
```
