# 03｜存储架构

GPU 最怕等待数据。必须同时考虑 Dataset、Checkpoint、Model/Artifact、Container/Image、Home/Project、Logs/Metrics。

分层可采用：容量/Object tier → Parallel Shared Storage → 高速 Storage Fabric → Compute → Local NVMe/cache/scratch。

设计不仅是 PB：还要定义 usable capacity、aggregate read/write、checkpoint burst、metadata IOPS、client 数、failure domain、rebuild 时间、backup/archive 和数据导入速度。

## 本地 NVMe 缓存与持久检查点

> 核查日期：2026-10-10（UTC）。学习目标：区分可重新生成的本地数据与必须保留的训练进度。项目基线保持 **128 台服务器 × 8 张 B300 GPU/台 = 1,024 张 GPU**。

### 先分清数据放在哪里

**官方事实：** DGX B300 用户指南分别列出启动存储（Boot storage）与缓存存储（Cache storage）。这里的本地 NVMe 是服务器内的高速固态存储；SuperPOD 参考架构将其用于缓存或预先暂存数据，减少重复读取时的网络访问。[DGX B300 硬件说明][dgx-storage]、[XDR/AC 存储架构][ra-storage]（核查：2026-10-10）。

DGX OS 7 指南说明，默认数据盘阵列采用 RAID 0：数据分布在多块盘上，但没有冗余；其中一块 SSD 故障会导致阵列数据丢失。官方明确要求这些盘用于应用缓存，不应存放关键、持久或长期数据。[DGX OS 7 数据存储配置，核查：2026-10-10][dgx-os-storage] 这是官方默认配置说明，现场重装或定制后仍须核对实际布局；不能直接套到所有 OEM HGX B300 整机。

**本项目建议：** 先确定数据丢失后能否重新取得，再决定存放位置。

| 数据用途 | 建议存放方式 | 需要确认的边界 |
|---|---|---|
| 可重新下载的数据集副本、可重新计算的临时文件 | 可放本地缓存或 scratch（临时工作区） | 保留来源与版本；节点更换后应能重新准备。 |
| Checkpoint（用于继续训练的检查点）、唯一的数据集、模型成果 | 放入已约定持久性与故障保护的共享存储 | 确认所有恢复节点可访问，并明确写入完成、保留期限和备份规则。 |

由上述本地盘定位可知，**把各节点的缓存容量相加，不能得到一套已具备共享访问和数据保护能力的存储系统**。共享访问本身也不证明已有备份；数据保护能力须由实际存储方案和恢复验收确认。

### 检查点从保存到恢复的建议顺序

官方参考架构指出，检查点写入用于训练容错，读写检查点的 I/O 会影响端到端训练效率。[XDR/AC 存储架构，核查：2026-10-10][ra-storage] 以下是据此制定的项目验收建议，尚未执行：

1. **确认真实落点。** 记录应用内的保存路径、容器挂载关系、主机挂载点与后端存储。目录名称相同，不足以证明不同节点访问的是同一份数据。
2. **确认何时可恢复。** 与框架及存储负责人约定“检查点完成”的判据。若采用先写本地再转存的方案，应分别记录本地写完和共享存储确认完成的时刻；转存未完成时，不把该检查点计为可承受原节点丢失的恢复点。
3. **在其他节点试恢复。** 由获授权人员使用调度器预留的测试资源和测试数据，在不依赖原节点本地副本的条件下验证读取与继续训练，留存检查点标识、软件版本和结果。无需通过拔盘或断电来做这项检查；故障注入须另有维护方案。
4. **分别记录保存与恢复耗时。** 保存侧记录训练等待及共享存储写入完成时间；恢复侧记录从读取检查点到继续训练的时间。已有[按参考架构区分的吞吐指导值](b300-storage-targets-note.md)仅供容量规划起步，不能代替应用恢复验证。

**适用范围与验证边界：** 本节采用 DGX B300 用户指南、DGX OS 7 默认存储说明，以及 DGX B300 + Quantum-X800 InfiniBand + AC 供电参考架构。不指定 OEM 磁盘布局、文件系统、挂载参数或备份产品。本次未运行任何存储或训练命令，未安装、部署、改变 RAID 或访问 B300 硬件；没有实测容量、吞吐或恢复时间。版本和资料日期见[本次来源记录](../sources/2026-10-10-local-cache-checkpoints.md)。

[dgx-storage]: https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html
[dgx-os-storage]: https://docs.nvidia.com/dgx/dgx-os-7-user-guide/system_configurations.html#data-storage-configuration
[ra-storage]: https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/storage-architecture.html
