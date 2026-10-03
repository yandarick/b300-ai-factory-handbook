# 00｜小白入门

先建立五层模型：GPU → 单机 Scale-Up → 集群 Scale-Out → 高性能存储 → AI Factory 基础设施。

## 核心术语
- HGX：GPU 平台，由 OEM 集成为服务器。
- DGX：NVIDIA 整机系统。
- NVLink/NVSwitch：节点内 GPU 高速互联。
- NCCL：多 GPU collective communication library。
- RDMA：高性能远程内存访问。
- InfiniBand：HPC/AI 高性能网络。
- RoCE：Ethernet 上的 RDMA。
- Spectrum-X：NVIDIA AI Ethernet 平台。
- DCGM：GPU telemetry/diagnostics/health。
- UFM：InfiniBand fabric 管理。
- Slurm：HPC/AI 作业调度。

推荐顺序：8-GPU 单机 → NVLink/NVSwitch → NCCL → RDMA → 32 节点 SU → 128 节点 fabric → 存储 → 电力/冷却 → 调度/监控 → 验收。
