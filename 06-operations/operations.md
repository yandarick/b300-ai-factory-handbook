# 06｜运维体系：128 台 DGX B300 Day-2 Operations

> 更新：2026-10-03

目标不是“服务器能 ping”，而是持续维持 GPU useful throughput、训练稳定性、网络无拥塞、存储持续供数和机房环境稳定。

## 运维分层

128 台 DGX B300 建议拆成五层管理：

1. 单机硬件：BMC/Redfish、GPU/HBM、NVLink/NVSwitch、CPU/RAM/NVMe、PSU、ConnectX-8、BlueField-3。
2. 集群基础设施：Compute Fabric、Storage Fabric、In-band、OOB、UFM、DNS/NTP。
3. 平台：NVIDIA Mission Control / Base Command Manager、Slurm、Kubernetes、Run:ai。
4. 可观测性：DCGM、Prometheus、Grafana、集中日志、交换机和 UFM telemetry。
5. 设施：rack PDU、母线、UPS、CDU/RDHx、供回水、漏液、环境传感器和 BMS。

## 当前 B300 软件运维基线

Mission Control 2.3.1 当前支持 DGX B300，官方 release notes 列出 BCM 11.33.1、Run:ai 2.25、Autonomous Hardware Recovery、Autonomous Job Recovery、Observability Stack、Air-gapped deployment 等能力。

2.3.x feature matrix 对 B300 给出的控制平面规模是 10 台 x86 管理节点：

- 2 × BCM Head
- 3 × User Kubernetes
- 3 × Admin Kubernetes
- 2 × Slurm

注意：Mission Control 的部分旧安装说明仍出现 B300 “32 nodes/SU”的文字，而当前 B300 SuperPOD 硬件参考架构和较新的系统管理说明已经是 InfiniBand XDR 72 nodes/SU、Spectrum-X Ethernet 64 nodes/SU。物理网络、机柜、光纤和扩容容量应以当前 B300 Reference Architecture 为准。

## 每天必须观察的指标

GPU/HBM：温度、功耗、ECC、HBM UCE、Xid、reset/recovery、利用率。

NVLink/NVSwitch：link state、CRC/replay/errors、degraded link、NVSwitch health。

Compute Fabric：ConnectX-8 link、800G error、RDMA error、拥塞、NCCL all-reduce bandwidth 与 tail latency。

Storage：读写吞吐、metadata latency、checkpoint flush time、data stall、容量和 inode。

Scheduler：GPU allocation efficiency、queued/failed jobs、node drain/down、job retry、useful GPU hours。

机房：rack kW、PDU/busbar current、CDU/RDHx、供回水温度/压力/流量、漏液、rack inlet temperature。

## 标准故障路径

发现 → 隔离 → 采证 → 恢复 → 基线测试 → 重新入池。

节点异常时先从 Slurm/Run:ai drain，保存 DCGM、BMC/Redfish、kernel、NCCL、NIC、UFM/交换机日志，再判断 GPU/HBM、NVLink、PCIe、CX-8、光模块/光纤、交换机或机房环境。修复后必须做单节点 health check 和多节点 NCCL/RDMA 基线测试，再 return-to-service。

## Mission Control 自动化基线测试

NVIDIA 当前 B300 autonomous hardware recovery 文档已经提供 automated baseline testing，覆盖 CPU/GPU、Memory/Storage、Network、installed software、firmware version，并包含 HPL、NCCL、Nemotron LLM 等测试。

正式项目应维护受控 Golden Baseline：

- DGX OS / kernel
- NVIDIA Driver / CUDA / NCCL
- NIC/DPU firmware
- BMC / BIOS / GPU tray firmware
- switch NOS / firmware / UFM
- firmware Source of Truth
- NCCL / RDMA / storage benchmark baseline

## 维护批次

128 台生产集群不要一次性全升级。建议：

Canary 1–2 台 → 1 个机柜 → 部分 SU → 剩余集群滚动升级。

每批之后执行 firmware inventory、DCGM、NVLink/NVSwitch health、NIC/RDMA、NCCL、storage throughput、scheduler test job，并与升级前 baseline 对比。

## 官方资料

- https://docs.nvidia.com/mission-control/docs/systems-quick-start-guide/2.3.1/nmc-release-notes.html
- https://docs.nvidia.com/mission-control/docs/nmc-software-installation-guide/2.3.0/mission-control-feature-support-matrix.html
- https://docs.nvidia.com/mission-control/docs/systems-administration-guide/2.3.0/autonomous-hardware-recovery.html
- https://docs.nvidia.com/mission-control/docs/nmc-software-installation-guide/2.3.1/airgapped-installation/observability-install.html
- https://docs.nvidia.com/dgx/dgxb300-user-guide/
- https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/
