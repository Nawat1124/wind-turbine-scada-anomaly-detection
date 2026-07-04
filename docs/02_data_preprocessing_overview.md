```text
┌──────────────────────────────────────────────────────────────┐
│ Start: Set Paths                                             │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Load reconstructed v3 artifacts                              │
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
│ Basic sanity checks                                          │
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
│                                                              │
│ Asset 2                                                      │
│ 2025-07-02 10:40:00 ~ 2025-07-02 12:10:00                    │
│ 10 rows removed                                              │
│ Keep physical_order_index unchanged                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Angle encoding                                               │
│                                                              │
│ sensor_4_avg  -> sin / cos                                   │
│ sensor_21_avg -> sin / cos                                   │
│ sensor_10_avg kept raw                                       │
│                                                              │
│ Drop raw circular angle columns                              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Drop unusable variables                                      │
│                                                              │
│ - all-zero columns                                           │
│ - constant columns                                           │
│ - angle std columns                                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Remove counter-like sensor_0 ~ sensor_3 family               │
│                                                              │
│ Reason: reset / jumps / not stable physical-state features   │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Add never_prediction_physical_row                            │
│                                                              │
│ safe normal row =                                            │
│   status_type_id in {0, 2}                                   │
│   AND never appeared in prediction segment                   │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Define final model feature list                              │
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
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Train / validation / embargo split                           │
│                                                              │
│ split_code:                                                  │
│ -1 = not eligible                                            │
│  0 = train                                                   │
│  1 = validation                                              │
│  2 = embargo                                                 │
│                                                              │
│ Per asset, split safe normal rows into 4 seasonal folds      │
│ Middle 10% of each fold used as validation                   │
│ 4-hour embargo around validation                             │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Fit metadata models on train safe normal rows only           │
│                                                              │
│ KMeans: MiniBatchKMeans, K = 6                               │
│ PCA: 10 components                                           │
│ Sample size cap: 200,000 rows                                │
│                                                              │
│Note: KMeans and PCA variables are metadata,                  │
│not input features for the LSTM.                              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Add KMeans + PCA metadata to all rows                        │
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
│                                                              │
│ Includes:                                                    │
│ - metadata columns                                           │
│ - 49 model input features                                    │
│ - 11 PCA metadata columns                                    │
│ - 2 KMeans metadata columns                                  │
│                                                              │
│ Sorted by asset_id + physical_order_index                    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Add prediction event IDs                                     │
│                                                              │
│ source_session_id is treated as event_id                     │
│                                                              │
│ Add:                                                         │
│ - pred_event_id_1                                            │
│ - pred_event_id_2                                            │
│                                                              │
│ Final df_model shape: 560060 rows × 71 columns               │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build sliding windows                                        │
│                                                              │
│ Window = 24 rows = 4 hours                                   │
│ Stride = 12 rows = 2 hours                                   │
│ Expected step = 10 minutes                                   │
│                                                              │
│ Reject windows crossing time gaps                            │
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
```
