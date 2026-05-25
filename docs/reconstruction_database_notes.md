用于写：数据为什么重构、duplicated SCADA sequences、asset-level physical timeline、event\_mapping、df\_reconstructed 的逻辑。



总图：从原始行到重构后的定位

┌──────────────────────────────────────────────┐

│                原始 CSV 数据层                                                           │

└──────────────────────────────────────────────┘

&#x20;                 │

&#x20;                 ▼

&#x20;       每一行原始数据的“原始身份”

&#x20;       (asset\_id, session\_id, source\_pos)

&#x20;                 │

&#x20;                 │  这是原始行最稳定的身份证

&#x20;                 │

&#x20;                 ▼

┌──────────────────────────────────────────────┐

│              原始行辅助信息层                                                            │

├──────────────────────────────────────────────┤

│ source\_row\_uid   = asset\_id|session\_id|source\_pos

│ source\_time\_stamp = 原始匿名时间

│ scada\_hash        = 原始SCADA数值的hash

└──────────────────────────────────────────────┘

&#x20;                 │

&#x20;                 │

&#x20;                 │ scada\_hash 只用于

&#x20;                 │ “找重复块 / 找 overlap”

&#x20;                 ▼

┌──────────────────────────────────────────────┐

│                pair\_diag 诊断层          						    │

├──────────────────────────────────────────────┤

│ session\_a, session\_b                     						    │

│ a\_start, a\_end                               						    │

│ b\_start, b\_end                               						    │

│ stitching\_status                            					            │

│ inferred\_order                             						    │

└──────────────────────────────────────────────┘

&#x20;                 │

&#x20;                 │

&#x20;                 │ 决定哪些原始行保留、哪些被剪掉、

&#x20;                 │ 哪些只作为 contained / overlap fallback

&#x20;                 ▼

┌──────────────────────────────────────────────┐

│          reconstructed timeline 重构层            				     │

├──────────────────────────────────────────────┤

│ df\_reconstructed                        						     │

│ reconstructed\_by\_asset\[aid]                  					     │

│                                            						             │

│ 关键定位变量：                                                			     │

│ physical\_order\_index  ← 重构后的主顺序索引  				     │

│ reconstructed\_time   ← 重构后的时间戳       	   			     │

│ source\_session\_id    ← 这行来自哪个原session  			     │

│ source\_pos           ← 这行在原session第几行 				     │

│ source\_time\_stamp    ← 原始时间戳            				     │

└──────────────────────────────────────────────┘

&#x20;                 │

&#x20;                 │

&#x20;                 │ 这里的每一行已经是“重构后的物理行”

&#x20;                 ▼

┌──────────────────────────────────────────────┐

│              event\_mapping 映射层          					    │

├──────────────────────────────────────────────┤

│ 原始 event/session 的 train/prediction 区间  				    │

│ 映射到 reconstructed timeline 上的位置     				    │

│                                            							    │

│ train\_start\_idx / train\_end\_idx              					    │

│ prediction\_start\_idx / prediction\_end\_idx    				    │

│ event\_order\_rank                            					    │

└──────────────────────────────────────────────┘

