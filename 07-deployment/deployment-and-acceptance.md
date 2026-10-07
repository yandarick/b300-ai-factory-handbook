# 07｜部署与验收

阶段：Requirements → HLD → LLD → Site Survey → FAT → 机房 Ready → 上架 → 布线 → Firmware baseline → OS/Provisioning → Fabric → Storage → Burn-in → Performance Acceptance → Failure/Recovery → Handover。

验收分层：单 GPU、8-GPU 单节点、NVLink/NVSwitch、NIC/RDMA、单 SU、多 SU、NCCL、Storage、Scheduler、Monitoring、Power/Cooling failure scenario。

最终建立可重复的 Golden Baseline，扩容后用同一方法比较。

## NCCL AllReduce：怎样读验收结果

> 核查日期：2026-10-07（UTC）。本节解释测试指标，不给出 B300 性能达标值。设计基线保持 **128 台服务器 × 8 GPU/台 = 1,024 GPU**。

**官方事实：** NCCL 是 GPU 集体通信库。AllReduce（全归约）把各参与 GPU 的输入按指定运算合并，并让每个参与者都得到结果；例如求和时，各 GPU 得到同一份逐元素求和结果。rank 是通信组内的参与者编号，不能当作服务器编号。[NCCL Collective Operations，核查：2026-10-07][nccl-collectives]

### 先确认有多少 GPU 真正参与

nccl-tests 的总 rank 数按“进程数 × 每进程线程数 × 每线程 GPU 数”计算；若设置了分组测试，还必须分别记录每组规模。它支持将 GPU 分成多个并行通信组，报告带宽按组计。[nccl-tests README，核查：2026-10-07][nccl-tests-readme]

**本项目应用：** 只有全部 128 台、每台 8 张 GPU 都参加同一个通信组时，下式的 `n` 才是 1,024。单台 8-GPU 测试取 `n = 8`；不能把单机测试结果写成整个集群已通过验收。

### algbw 和 busbw 分别表示什么

以下为官方 AllReduce 换算定义，`S` 是**每个 rank 的输入数据量**（B），`t` 是每次操作耗时（先按实际输出表头换算为秒），`n` 是该通信组的 rank 数。[nccl-tests Performance，核查：2026-10-07][nccl-tests-performance]

| 指标 | 算式与单位 | 阅读方式 |
|---|---|---|
| `algbw`：算法带宽 | `S ÷ t ÷ 10^9`，GB/s | 用输入数据量除以耗时，衡量这次操作完成得多快。 |
| `busbw`：总线带宽 | `algbw × 2 × (n − 1) ÷ n`，GB/s | 对 AllReduce 通信量作归一化，帮助分析互联利用情况；其他集体操作的系数可能不同。 |

**假设算例，非实测或预测：** 8 个 rank 各有 `S = 1,000,000,000 B = 1 GB` 输入，假设一次操作耗时 `t = 0.02 s`，则 `algbw = 50 GB/s`，`busbw = 50 × 2 × 7 ÷ 8 = 87.5 GB/s`。这里的 1 GB 是十进制，不能替换成 `1 GiB = 1,073,741,824 B` 后仍沿用同一结果；实际耗时取决于硬件、软件、拓扑和消息大小。

**判读边界：** 由上述定义可知，busbw 是计算指标，不是网口计数器读数；系数中的 `2` 也不是把物理链路收发速率相加。比较硬件峰值前要确认通信实际经过的路径和瓶颈，不能直接用 busbw 对照某一条网口的标称速率。物理链路口径见[800G 端口与链路计数](../02-network/128-node-link-count.md)。

### 建议的验收阅读顺序

1. **留存条件。** 记录服务器/GPU 清单、通信组与 rank 映射、NCCL 版本、nccl-tests 提交号、CUDA/驱动、网络与固件基线，以及完整参数和日志。跨节点运行所用 nccl-tests 须具备 MPI 支持；按现场已验证的软件组合准备。[官方构建要求，核查：2026-10-07][nccl-tests-readme]
2. **先看正确性。** 确认实际启用了结果校验，再检查所有已测数据大小的校验结果及异常日志；官方以 `--check` 控制校验迭代次数。无错误只证明本次已测用例，不能仅凭出现带宽数字认定验收通过。[官方参数说明，核查：2026-10-07][nccl-tests-readme]
3. **再看性能。** 小消息侧重耗时，大消息侧重带宽。[官方指标说明，核查：2026-10-07][nccl-tests-performance] 本项目建议在相同规模、消息大小、数据类型、缓冲区模式、预热和迭代设置下重复比较 Golden Baseline（已验收配置的基准记录），保存波动及异常，按事先约定的门槛判定；不从网口标称值反推统一通过线。
4. **逐层扩大范围。** 从单节点到跨节点、单 SU、跨 SU 留下独立结果，不能用一层通过代替下一层。性能测试由获授权人员在调度器预留并排空的维护资源上进行，避免与生产作业争用 GPU 和网络；这是本项目建议。

**适用范围与未验证事项：** 所读用户指南标题为 NCCL 2.32.3；nccl-tests 文档取自当日读取的 `master`，两者不构成 DGX/OEM B300 软件兼容认证或安装建议。本次没有执行测试命令、安装或硬件操作，没有 B300 实测数据。资料日期与来源边界见[本次核查记录](../sources/2026-10-07-nccl-acceptance.md)。

[nccl-collectives]: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/collectives.html
[nccl-tests-readme]: https://github.com/NVIDIA/nccl-tests/blob/master/README.md
[nccl-tests-performance]: https://github.com/NVIDIA/nccl-tests/blob/master/doc/PERFORMANCE.md
