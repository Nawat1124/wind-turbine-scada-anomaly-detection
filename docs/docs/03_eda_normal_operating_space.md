┌──────────────────────────────────────────────────────────────┐
│ Start: Mount Drive + Set Paths                               │
│ 开始：挂载 Google Drive，设置路径                             │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Load outputs from 02_data_preprocessing                      │
│ 读取 02_data_preprocessing 的输出                             │
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
│ 定义安全正常样本 mask                                         │
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
│ 定义物理特征家族                                              │
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
│ 同时定义去冗余后的家族版本                                    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Basic dataset overview                                       │
│ 数据集基础概览                                                │
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
│ 时间断点诊断                                                  │
│                                                              │
│ For each asset:                                              │
│ - sort by physical_order_index                               │
│ - compute time difference                                    │
│ - expected step = 10 minutes                                 │
│ - classify gaps as small or big                              │
│                                                              │
│ Output: gap_diag + summaries                                 │
│ 结果：gap_diag 和按 asset / overall 的 gap summary            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Asset-level density plots                                    │
│ 按风机画密度分布                                              │
│                                                              │
│ Variables:                                                   │
│ - sensor_47_avg                                              │
│ - sensor_40_avg                                              │
│ - sensor_38_avg                                              │
│ - sensor_51_avg                                              │
│                                                              │
│ Purpose: show turbine-level normal baseline differences      │
│ 目的：展示不同风机的正常分布基线差异                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ IQR outlier diagnostics                                      │
│ IQR 异常值诊断                                                │
│                                                              │
│ Fit rows: normal_safe_mask                                   │
│ Fitting rows: 443248                                         │
│ Feature columns: 214                                         │
│ K_IQR = 3.0                                                  │
│ Per asset thresholds                                         │
│                                                              │
│ Only flag, no clipping                                       │
│ 只打 outlier flag，不裁剪原始值                               │
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
│ 对重点变量画直方图                                            │
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
│ 目的：检查偏态、长尾、多峰结构                                │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Within-family correlation analysis                           │
│ 家族内部相关性分析                                            │
│                                                              │
│ Method: Pearson correlation                                  │
│ Threshold: abs_corr >= 0.95                                  │
│                                                              │
│ Output:                                                      │
│ - all within-family pairs                                    │
│ - high correlation pairs                                     │
│ - family-level correlation summary                           │
│                                                              │
│ Purpose: justify redundancy pruning                          │
│ 目的：解释为什么要去除高度冗余特征                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Correlation heatmaps after pruning                           │
│ 去冗余后的相关性热力图                                        │
│                                                              │
│ 1. within-family heatmaps                                    │
│    家族内部 heatmap                                           │
│                                                              │
│ 2. between-family correlation                                │
│    家族之间 correlation                                       │
│                                                              │
│ Purpose: show remaining subsystem coupling                   │
│ 目的：展示去冗余后仍存在的子系统耦合关系                      │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Temporal diagnostics                                         │
│ 时间序列诊断                                                  │
│                                                              │
│ ACF of power_62_avg on Asset 2                               │
│ Asset 2 上 power_62_avg 的自相关                              │
│                                                              │
│ Max lag = 50 steps = 500 minutes                             │
│ Purpose: support 4-hour window length                        │
│ 目的：支持 4 小时窗口长度                                     │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Granger causality diagnostics                                │
│ Granger 因果诊断                                              │
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
│ 目的：检查短滞后预测关系，支持短窗口设计                      │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ KMeans operating-state diagnostics                           │
│ KMeans 正常运行状态诊断                                       │
│                                                              │
│ Use train normal rows                                        │
│ 使用 train normal rows                                       │
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
│ 目的：支持 KMeans-state threshold 的合理性                    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ KMeans diagnostic plots                                      │
│ KMeans 诊断图                                                 │
│                                                              │
│ - number of normal rows per state                            │
│ - q99 center distance by state                               │
│ - compare with global q99 distance                           │
│                                                              │
│ Purpose: show unequal variability across normal states       │
│ 目的：展示不同正常状态内部波动程度不同                        │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ UMAP normal operating-space visualization                    │
│ UMAP 正常运行空间可视化                                       │
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
│ 目的：观察正常运行空间是否有结构性分区                        │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA explained variance                                       │
│ PCA 解释方差                                                  │
│                                                              │
│ Display pca_explained table                                  │
│ Plot explained variance ratio + cumulative variance          │
│                                                              │
│ Purpose: summarize global low-dimensional structure          │
│ 目的：总结正常空间的低维主结构                                │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA scatter plot                                             │
│ PCA 散点图                                                    │
│                                                              │
│ Plot PC1 vs PC2                                              │
│ Colored by KMeans state                                      │
│                                                              │
│ Purpose: check whether KMeans states align with PCA space    │
│ 目的：检查 KMeans 状态是否对应 PCA 空间中的结构差异            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA loading interpretation                                   │
│ PCA loading 解释                                              │
│                                                              │
│ Map each feature to physical family                          │
│ 将每个特征映射到物理家族                                      │
│                                                              │
│ For PC1 ~ PC5:                                               │
│ - list top loading features                                  │
│ - plot loading bars                                          │
│                                                              │
│ Purpose: interpret PCs physically                            │
│ 目的：用物理子系统解释主成分                                  │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ PCA family contribution                                      │
│ PCA 家族贡献分析                                              │
│                                                              │
│ For each PC:                                                 │
│ - square each loading                                        │
│ - sum squared loadings by family                             │
│ - plot family contribution for PC1 ~ PC10                    │
│                                                              │
│ Purpose: show which subsystem drives each PC                 │
│ 目的：说明每个主成分主要由哪些子系统贡献                      │
└──────────────────────────────────────────────────────────────┘
