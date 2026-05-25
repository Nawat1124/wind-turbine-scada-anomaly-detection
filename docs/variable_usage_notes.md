以后变量怎么用，比如 df\_reconstructed、reconstructed\_by\_asset、event\_mapping、pair\_diag、SCADA\_HASH\_COLS。

原始定位变量：它们各自干什么？

asset\_id

└─ 这是哪台风机

session\_id

└─ 这是哪个原始 event CSV，例如 23.csv / 53.csv

source\_pos

└─ 这行在原始 session 里的第几行

└─ 原始行的“行号身份”

source\_row\_uid = asset\_id|session\_id|source\_pos

└─ 原始行的唯一ID

└─ 本质上是把前三个信息拼成一个字符串

source\_time\_stamp

└─ 原始 CSV 里这行自带的匿名时间

└─ 只能当 source metadata，看原始来源

└─ 不能直接当全局真实物理时间

scada\_hash

└─ 这行 SCADA 数值长什么样

└─ 用来找“重复块 / overlap”

└─ 不是原始行唯一身份

└─ 更不能直接拿来决定 reconstructed\_time

重构后定位变量：它们各自干什么？

physical\_order\_index

└─ 重构后真正的主顺序索引

└─ 谁先谁后，主要靠它

└─ 后续 window / chronological evaluation 最应该信它（？）

reconstructed\_time

└─ 重构后给这一行分配的时间戳

└─ 主要用于画图、reindex、gap labeling

└─ 是“服务于后续处理的时间轴”

└─ 不是原始行唯一身份

source\_session\_id

└─ 这条重构后的物理行，来自哪个原始 session

source\_pos

└─ 这条重构后的物理行，在原始 session 里原来是第几行

source\_time\_stamp

└─ 这条重构后物理行原来的匿名时间

all\_source\_sessions

└─ 同一 physical row 曾在哪些原始 session 里出现过

└─ 主要是 debug / overlap 来源追踪

