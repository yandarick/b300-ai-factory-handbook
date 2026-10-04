# 06｜运维体系：128 台 DGX B300 Day-2 Operations

> 更新：2026-10-04（本次核查 DCGM 检查边界；其余条目沿用 2026-10-03）。

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

## DCGM：在线健康监控与维护窗口诊断

DCGM（Data Center GPU Manager）既能读取运行状态，也能主动给 GPU 施加测试负载。**生产作业运行时用被动健康监控；主动诊断安排在目标资源空闲且由调度器隔离的时段。** 官方所说的 online diagnostics 是“在已安装的系统环境内运行”，不等于“可以与训练同时运行”。[官方 Health Monitoring，核查：2026-10-04][dcgm-health]

### 先选检查，再安排作业

下表的测试范围依据当日官方命令参考；“运行安排”是本项目运维建议。实际插件支持取决于 DCGM 版本、硬件及依赖，不能把表格当成每台 B300 已通过兼容性验证的证明。[官方命令参考，核查：2026-10-04][dcgm-cli]

| 检查 | 目的与范围 | 运行安排 |
|---|---|---|
| `dcgmi health` | 被动分析已采集的指标和事件，不施加诊断负载。[官方说明，2026-10-04][dcgm-health] | 可随生产作业持续运行，用于日常巡检和告警。 |
| `dcgmi diag -r 1` | 快速软件部署检查，核对运行环境；不代表负载能力验收。[官方说明，2026-10-04][dcgm-cli] | 放在 prologue（作业启动前的准备阶段）；目标 GPU 必须空闲并被调度器预留，不要求整个集群停机。 |
| `dcgmi diag -r 2` | 在前一级基础上增加 GPU 内存和 PCIe/节点内互联测试。[官方说明，2026-10-04][dcgm-cli] | 在维护窗口，或失败作业的 epilogue（退出后的清理阶段）运行；先确认作业及其他使用者已退出，诊断结束前不接新作业。 |
| `dcgmi diag -r 3` / `-r 4` | 长测试包含计算、带宽、功耗等压力；更长级别还增加内存模式与脉冲功耗测试。[官方说明，2026-10-04][dcgm-cli] | 管理员在维护窗口内隔离目标资源后运行，并核对供电、冷却余量；不加入生产节点的每日在线巡检。 |

主动诊断会占用 GPU、显存和互联等资源，部分长测试可能重置状态或与运行进程冲突。`production_testing` 也是主动测试套件，它的名称不表示可与生产作业并发。[官方 Diagnostics][dcgm-diag]、[套件定义][dcgm-cli]（核查：2026-10-04）。

### 在线巡检：先启用采集，再查询健康

以下是**未实际执行的命令模板**。由运维人员在目标节点复用已核对成员的 GPU group（DCGM 设备分组），将 `<group-id>` 替换成真实编号；不假定组号等于节点号，也不把单节点分组当作整个集群。按实际安装版本复核命令帮助后使用。

```text
# 配置被动监控：PCIe、内存、InfoROM、温度/功耗、NVLink、驱动
dcgmi health --set pmitnd --group <group-id>
# 确认启用了哪些监控项；这一步不是健康判定
dcgmi health --fetch --group <group-id>
# 采集到足够样本后查询，可在作业运行期间使用
dcgmi health --check --group <group-id>
```

部分监控规则需要约 60 秒样本；刚启用就查询可能没有意义。`Healthy` 只表示保留的数据中未命中已启用规则，不代表所有子系统都被检查，更不代表通过压力测试。长期节点监控应保留监控项，并使数据保留时长覆盖轮询间隔。[官方示例及结果解释，核查：2026-10-04][dcgm-health]

### 主动诊断：从停止派发到重新入池

以下为本项目建议流程；依据是 NVIDIA 要求在侵入式测试前排空应用及对等使用者，并结合逐项结果判断。[官方准备与结果说明，核查：2026-10-04][dcgm-diag]

1. **停止派发并采证。** 在实际调度平台将目标节点设为不可接新作业的状态（常称 drain），保存作业编号、错误时间和原始日志。仅设置 drain 不等于现有训练进程已退出；按作业策略完成 checkpoint（保存训练进度）、退出或迁移，并确认跨节点作业的相关进程及对等访问已释放。
2. **确认独占范围。** 核对节点、GPU 标识、DCGM/驱动版本与实际测试目标。共享节点上若不能证明其他作业不受影响，建议整节点维护；禁止凭“GPU 利用率为零”判定已无人使用。
3. **选所需测试。** 按上表选择级别，在维护范围内明确指定设备或已核对成员的 group。例如 `dcgmi diag --run 2 --group <group-id> --json` 是未执行的维护窗口模板，不是定时巡检命令。供电、冷却异常尚未处理时，先排障，不立即追加压力测试。
4. **读完整结果。** 保存实际命令、目标、版本、退出码、逐项结果和相关日志。遇到失败、跳过或不支持的项目，先查明原因；不能仅凭命令结束、出现成功提示或退出码正常就认定所有验收项通过。
5. **满足恢复条件再入池。** 故障原因处理完成后，对比修复前后健康记录和主动测试结果，再按故障范围执行受控 NCCL/RDMA 与调度测试。GPU reset、重启和 NVIDIA 离线诊断另按维护方案执行；DCGM 诊断不自动修复故障，也不能替代离线硬件诊断或决定 RMA（返厂维修/换件）资格。[官方能力边界，核查：2026-10-04][dcgm-diag]

**版本与适用范围：** 本节参考当日 `latest` 文档；release notes 顶部版本为 4.7.0，其 4.4.0 条目列出新增 B300 主动诊断支持（devId `3182`、subsystem ID `20E610DE`）。这不是对所有 B300 整机软件组合的兼容承诺；上线前仍需按目标 GPU 标识、已安装 DCGM、驱动及依赖核对相应版本说明。[官方 Release Notes，核查：2026-10-04][dcgm-release] 本次没有访问 B300 硬件或实际集群，不提供实测耗时、通过率或已部署声明；维护时长须在匹配环境测定。来源页更新时间和本次变更见[核查记录](../sources/2026-10-04-dcgm.md)。

## 标准故障路径

发现 → 隔离 → 采证 → 恢复 → 基线测试 → 重新入池。

节点异常时先在实际调度平台停止新作业派发，保存 DCGM、BMC/Redfish、kernel、NCCL、NIC、UFM/交换机日志，再判断 GPU/HBM、NVLink、PCIe、CX-8、光模块/光纤、交换机或机房环境。主动诊断前必须按上节确认作业退出及资源隔离。修复后，在维护范围内完成单节点健康监控复核、所需主动诊断和多节点 NCCL/RDMA 基线测试，再 return-to-service（恢复接收作业）。

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

每批升级后、重新接收生产作业前，在该批节点的维护窗口内核对 firmware inventory、DCGM 被动健康与所需主动诊断、NVLink/NVSwitch health，并执行 NIC/RDMA、NCCL、storage throughput、scheduler test job，与升级前 baseline 对比。重新入池后持续保留被动监控。

## 官方资料

- https://docs.nvidia.com/mission-control/docs/systems-quick-start-guide/2.3.1/nmc-release-notes.html
- https://docs.nvidia.com/mission-control/docs/nmc-software-installation-guide/2.3.0/mission-control-feature-support-matrix.html
- https://docs.nvidia.com/mission-control/docs/systems-administration-guide/2.3.0/autonomous-hardware-recovery.html
- https://docs.nvidia.com/mission-control/docs/nmc-software-installation-guide/2.3.1/airgapped-installation/observability-install.html
- https://docs.nvidia.com/dgx/dgxb300-user-guide/
- https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/

[dcgm-health]: https://docs.nvidia.com/datacenter/dcgm/latest/learn/modules/health-monitoring.html
[dcgm-diag]: https://docs.nvidia.com/datacenter/dcgm/latest/learn/modules/dcgm-diagnostics.html
[dcgm-cli]: https://docs.nvidia.com/datacenter/dcgm/latest/reference/command-line-reference/dcgmi/dcgmi-diag.html
[dcgm-release]: https://docs.nvidia.com/datacenter/dcgm/latest/release-notes/changelog.html
