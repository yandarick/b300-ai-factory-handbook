# 03｜存储架构

GPU 最怕等待数据。必须同时考虑 Dataset、Checkpoint、Model/Artifact、Container/Image、Home/Project、Logs/Metrics。

分层可采用：容量/Object tier → Parallel Shared Storage → 高速 Storage Fabric → Compute → Local NVMe/cache/scratch。

设计不仅是 PB：还要定义 usable capacity、aggregate read/write、checkpoint burst、metadata IOPS、client 数、failure domain、rebuild 时间、backup/archive 和数据导入速度。
