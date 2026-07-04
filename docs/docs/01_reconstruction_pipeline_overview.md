```text
┌──────────────────────────────────────────────────────────────┐
│ Start: Set paths and reconstruction parameters               │
│                                                              │
│ DATA_ROOT = raw CARE Wind Farm B datasets                    │
│ OUT_DIR   = reconstructed output directory                   │
│ MIN_BLOCK_LEN = 100                                          │
│ PLOT_ASSETS = all                                            │
│ SAVE_PLOTS = True                                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Load raw CARE event/session CSV files                        │
│                                                              │
│ For each CSV:                                                │
│ - read with separator ";"                                   │
│ - parse asset_id                                             │
│ - parse time_stamp                                           │
│ - sort by time_stamp                                         │
│ - add source_session_id                                      │
│ - add source_pos                                             │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build row-level SCADA hash                                   │
│                                                              │
│ Exclude metadata columns:                                    │
│ - id                                                         │
│ - asset_id                                                   │
│ - source_session_id                                          │
│ - time_stamp                                                 │
│ - train_test                                                 │
│ - status_type_id                                             │
│ - source_pos                                                 │
│                                                              │
│ Use common numeric SCADA columns only                        │
│                                                              │
│ Purpose: detect duplicated physical sensor records           │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Compare sessions within each asset                           │
│                                                              │
│ For every pair of sessions from the same asset:              │
│ - match rows by scada_hash                                   │
│ - find shared physical records                               │
│ - locate the longest aligned duplicated block                │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Diagnose pairwise overlap type                               │
│                                                              │
│ Boundary types:                                              │
│ - full                                                       │
│ - prefix                                                     │
│ - suffix                                                     │
│ - internal                                                   │
│ - none                                                       │
│                                                              │
│ Stitching status:                                            │
│ - INDEPENDENT                                                │
│ - CLEAN_STITCH                                               │
│ - CONTAINMENT                                                │
│ - AMBIGUOUS_SMALL_OVERLAP                                    │
│ - AMBIGUOUS_INTERNAL_OVERLAP                                 │
│                                                              │
│ Also count train-prediction conflict rows                    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build pair_diag                                              │
│                                                              │
│ pair_diag records:                                           │
│ - asset_id                                                   │
│ - session_a / session_b                                      │
│ - shared rows                                                │
│ - max duplicated block length                                │
│ - overlap boundaries                                         │
│ - stitching status                                           │
│ - inferred order                                             │
│ - train-prediction conflict count                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Reconstruct one physical timeline per asset                  │
└───────────────────────────────┬──────────────────────────────┘
                                │
               ┌────────────────┼────────────────┐
               │                │                │
               v                v                v
┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐
│ INDEPENDENT           │ │ CLEAN_STITCH          │ │ CONTAINMENT           │
│                      │ │                      │ │                      │
│ Keep sessions sorted │ │ Keep earlier session  │ │ Keep carrier session  │
│ by time              │ │ Add only unique tail  │ │ Map contained rows    │
└──────────┬───────────┘ └──────────┬───────────┘ └──────────┬───────────┘
           │                        │                        │
           └────────────────────────┼────────────────────────┘
                                    │
                                    v
┌──────────────────────────────────────────────────────────────┐
│ Align reconstructed_time                                     │
│                                                              │
│ For CLEAN_STITCH, shift timestamps so duplicated blocks      │
│ share one physical timeline.                                 │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Sort by reconstructed_time and assign physical_order_index   │
│                                                              │
│ physical_order_index is assigned per asset                   │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build df_reconstructed                                       │
│                                                              │
│ This is the unique reconstructed physical timeline           │
│                                                              │
│ Used by all downstream preprocessing and modelling notebooks │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build row_lineage_map                                        │
│                                                              │
│ Direct rows:                                                 │
│ original source row -> reconstructed physical row            │
│                                                              │
│ Omitted duplicated rows:                                     │
│ original source row -> counterpart physical row              │
│                                                              │
│ Purpose: preserve original event row correspondence          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Build event_mapping                                          │
│                                                              │
│ For each original session:                                   │
│ - map train interval to reconstructed physical rows          │
│ - map prediction interval to reconstructed physical rows     │
│ - record start/end index and start/end time                  │
│                                                              │
│ Purpose: preserve event structure                            │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Plot reconstructed timelines                                 │
│                                                              │
│ Each asset plot shows:                                       │
│ - reconstructed physical timeline                            │
│ - confirmed duplicated overlap                               │
│ - original train intervals                                   │
│ - original prediction intervals                              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Save minimal v3 artifacts                                    │
│                                                              │
│ Save:                                                        │
│ - df_reconstructed.parquet                                   │
│ - row_lineage_map.parquet                                    │
│ - event_mapping.parquet                                      │
│ - figures/asset_*_reconstructed_timeline.png                 │
│                                                              │
│ Do not save unnecessary debug artifacts                      │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                v
┌──────────────────────────────────────────────────────────────┐
│ Final sanity checks                                          │
│                                                              │
│ Check:                                                       │
│ - physical_order_index is not missing                        │
│ - reconstructed_time is not missing                          │
│ - row_lineage_map maps original rows                         │
└──────────────────────────────────────────────────────────────┘
```
