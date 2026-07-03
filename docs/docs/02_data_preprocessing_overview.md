┌──────────────────────────────────────────────────────────────┐
│ Start: Mount Drive + Set Paths                               │
│ 开始：挂载 Google Drive，设置项目路径                         │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Load reconstructed v3 artifacts                              │
│ 读取重构后的 v3 数据产物                                      │
│                                                              │
│ - df_reconstructed.parquet                                   │
│ - row_lineage_map.parquet                                    │
│ - event_mapping.parquet                                      │
│ - feature_description.csv                                    │
│ - event_info_v6.csv                                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Basic sanity checks                                           │
│ 基础一致性检查                                                │
│                                                              │
│ - required columns exist                                     │
│ - no duplicated asset_id + physical_order_index              │
│ - lineage rows map back to reconstructed physical rows       │
│ - row_lineage_map should not contain status_type_id          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Remove confirmed data glitch                                 │
│ 删除人工确认的数据异常块                                      │
│                                                              │
│ Asset 2                                                       │
│ 2025-07-02 10:40:00 ~ 2025-07-02 12:10:00                    │
│ 10 rows removed                                               │
│ 不重新编号 physical_order_index                              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Angle encoding                                                │
│ 角度变量编码                                                  │
│                                                              │
│ sensor_4_avg  -> sin / cos                                   │
│ sensor_21_avg -> sin / cos                                   │
│ sensor_10_avg kept raw                                       │
│ sensor_10_avg 保留原值                                       │
│                                                              │
│ Drop raw circular angle columns                              │
│ 删除原始圆形角度列                                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Drop unusable variables                                      │
│ 删除不可用变量                                                │
│                                                              │
│ - all-zero columns                                           │
│ - constant columns                                           │
│ - angle std columns                                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Remove counter-like sensor_0 ~ sensor_3 family               │
│ 删除不稳定 counter-like 变量族                                │
│                                                              │
│ Reason: reset / jumps / not stable physical-state features   │
│ 原因：存在归零、跳变，不适合作为稳定物理状态变量              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Add never_prediction_physical_row                            │
│ 添加 never_prediction_physical_row 标记                      │
│                                                              │
│ safe normal row =                                            │
│   status_type_id in {0, 2}                                   │
│   AND never appeared in prediction segment                   │
│                                                              │
│ 安全正常训练行 = 正常状态 + 从未出现在 prediction 段           │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Define final model feature list                              │
│ 定义最终模型特征列表                                          │
│                                                              │
│ Keep:                                                        │
│ - avg variables                                              │
│ - angle sin/cos variables                                    │
│                                                              │
│ Remove:                                                      │
│ - highly redundant avg columns                               │
│ - counter-like variables                                     │
│                                                              │
│ Final model feature count: 49                                │
│ 最终模型输入特征数：49                                        │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Train / validation / embargo split                           │
│ 训练集 / 验证集 / embargo 切分                                │
│                                                              │
│ split_code:                                                  │
│ -1 = not eligible                                            │
│  0 = train                                                   │
│  1 = validation                                              │
│  2 = embargo                                                 │
│                                                              │
│ Per asset, split safe normal rows into 4 folds               │
│ 每台风机内部按 physical order 分成 4 段                       │
│ Middle 10% of each fold used as validation                   │
│ 每段中间 10% 作为 validation                                  │
│ 4-hour embargo around validation                             │
│ validation 前后 4 小时作为 embargo                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Fit metadata models on train safe normal rows only           │
│ 只在 train safe normal 上拟合 metadata models                 │
│                                                              │
│ KMeans: MiniBatchKMeans, K = 6                               │
│ PCA: 10 components                                           │
│ Sample size cap: 200,000 rows                                │
│                                                              │
│ 注意：KMeans / PCA 是 metadata，不是 LSTM 输入特征             │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Add KMeans + PCA metadata to all rows                        │
│ 给所有行添加 KMeans 和 PCA 元数据                             │
│                                                              │
│ KMeans metadata:                                             │
│ - kmeans_state                                               │
│ - kmeans_center_dist                                         │
│                                                              │
│ PCA metadata:                                                │
│ - pca_PC1 ... pca_PC10                                       │
│ - pca_residual_after_PC10                                    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build df_model                                                │
│ 构建模型可用主表 df_model                                     │
│                                                              │
│ Includes:                                                    │
│ - metadata columns                                           │
│ - 49 model input features                                    │
│ - 11 PCA metadata columns                                    │
│ - 2 KMeans metadata columns                                  │
│                                                              │
│ Sorted by asset_id + physical_order_index                    │
│ 按风机和物理顺序排序                                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Add prediction event IDs                                     │
│ 添加 prediction event ID                                     │
│                                                              │
│ source_session_id is treated as event_id                     │
│ source_session_id 被视为 event_id                            │
│                                                              │
│ Add:                                                         │
│ - pred_event_id_1                                            │
│ - pred_event_id_2                                            │
│                                                              │
│ Final df_model shape: 560060 rows × 71 columns               │
│ 最终 df_model：560060 行 × 71 列                              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build sliding windows                                        │
│ 构建滑动窗口                                                  │
│                                                              │
│ Window = 24 rows = 4 hours                                   │
│ Stride = 12 rows = 2 hours                                   │
│ Expected step = 10 minutes                                   │
│                                                              │
│ Reject windows crossing time gaps                            │
│ 不允许窗口跨越时间断点                                        │
│                                                              │
│ Output:                                                      │
│ - X_train: 30591 × 24 × 49                                   │
│ - X_val:   3378 × 24 × 49                                    │
│ - X_pred_by_event                                            │
│ - window metadata                                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Save model-ready outputs                                     │
│ 保存模型可用产物                                              │
│                                                              │
│ - df_model.parquet                                           │
│ - model_ready_feature_list.json                              │
│ - windows_4h_stride2h_bundle.pkl                             │
│ - pca_explained_variance.csv                                 │
│ - pca_loadings.csv                                           │
│ - model_ready_sample.csv                                     │
│ - preprocessing_summary.json                                 │
│ - eda_base.parquet                                           │
└──────────────────────────────────────────────────────────────┘
