# 05｜软件平台

分层：Firmware/BMC/NIC/Switch OS → Linux/Driver/CUDA/DOCA/OFED → NCCL → Container → Slurm/Kubernetes → AI Framework → Monitoring/Logging → Portal/Quota/Accounting。

重点回答：裸机自动安装、兼容矩阵、GPU 分配、多节点调度、故障定位、滚动升级、用户/项目/quota/accounting，以及 Slurm 与 Kubernetes 是否共存。

## 驱动、CUDA 与容器：怎样核对兼容性

> 核查日期：2026-10-08（UTC）。本节讲版本核对方法，不指定生产安装组合。项目基线仍为 **128 台服务器 × 8 张 B300 GPU/台 = 1,024 张 GPU**。

### 先分清三层软件

| 层次 | 作用与应记录的信息 |
|---|---|
| 主机 NVIDIA GPU 驱动 | 主机是实际运行容器的服务器；驱动包含内核组件和用户态 CUDA 驱动，负责支撑 GPU 应用运行。记录实际加载的驱动版本。[官方说明，核查：2026-10-08][cuda-why] |
| 应用使用的 CUDA | CUDA Toolkit 是开发工具包，包含编译工具、CUDA Runtime（运行库）及其他库和工具；应用还可能依赖动态链接库。记录应用或镜像所用的 CUDA 与库版本，不能只抄主机驱动版本。[官方说明，核查：2026-10-08][cuda-why] |
| 容器引擎与 NVIDIA Container Toolkit | 前者运行容器，后者配合容器引擎提供 GPU 访问。官方安装前提仍包含主机 GPU 驱动，因此换镜像不能省去主机驱动核对。分别记录容器引擎和 Container Toolkit 版本。[官方前提与配置说明，核查：2026-10-08][container-install] |

**DGX B300 的产品边界：** 官方用户指南列出的预装软件包括 GPU 驱动、Docker Engine 和 NVIDIA Container Toolkit，但该清单没有在此给出完整版本组合。它适用于 NVIDIA DGX B300，不能据此认定某台 OEM HGX B300 已安装相同软件；交付时应向对应整机厂商核对版本清单。[DGX B300 用户指南，核查：2026-10-08][dgx-software]

### 不把 CUDA Version 当成 Toolkit 安装清单

**官方事实：** 当日 `nvidia-smi` 文档已将 `Driver Version`、`CUDA Version` 标为弃用字段名，分别改用 `KMD Version`、`CUDA UMD Version`。KMD 指内核态驱动，CUDA UMD 指用户态 CUDA 驱动；后者表示驱动支持的最高 CUDA 版本，可能与已安装的 Toolkit 不同。字段名称以现场版本的帮助和实际输出为准。[nvidia-smi GPU Attributes 与变更日志，核查：2026-10-08][nvidia-smi]

由此可知，仅看到这个 CUDA 数字，不能证明主机安装了对应 Toolkit，也不能证明容器内框架使用了相同版本。版本核对要同时拿到主机驱动记录、应用镜像的组件清单和该镜像的 release notes（版本说明）。

**官方兼容规则：** 新驱动对旧 CUDA 应用有向后兼容机制；旧驱动运行较新的 CUDA 软件则要满足对应兼容条件。同一 CUDA 主版本内的小版本兼容也受最低驱动、新功能、PTX（需由驱动进一步编译的 GPU 中间代码）和目标 GPU 架构等条件限制，不能只比较两个版本号的大小。[兼容机制][cuda-why]、[小版本兼容限制][cuda-minor]（核查：2026-10-08）。这些是 CUDA 通用规则，不是 B300 整机或特定框架的兼容认证。

### 本项目建议的核对顺序

1. **锁定整机与主机基线。** 写清 DGX B300 或具体 OEM 型号、OS/内核和驱动版本，取得该整机对应的软件支持说明；不要把其他 GPU 的最低驱动要求当成 B300 的支持结论。
2. **锁定应用镜像。** 记录镜像标签与 digest（内容摘要）、框架、CUDA、NCCL 及相关库版本，并保留该版本的组件清单和 release notes，便于复现。
3. **逐项对照。** 分别核对 B300 硬件支持、OS/驱动组合、镜像的驱动要求和 CUDA 兼容限制。缺少其中一项证据就记为“待核对”；不能因容器启动成功或能列出 GPU 就判定应用兼容。
4. **再验证实际应用。** 获得匹配硬件后，在调度器预留并排空的维护资源上测试目标应用的正确性，再做多 GPU/跨节点验证；通信验收沿用[已有 NCCL 判读方法](../07-deployment/deployment-and-acceptance.md#nccl-allreduce怎样读验收结果)。留存完整条件和日志，结果只覆盖实际测过的组合。

**验证边界：** 本次只阅读官方资料，没有运行 `nvidia-smi`、容器或应用测试，也没有安装、升级或访问 B300 硬件；未核验任何现场软件组合。资料版本与日期见[本次来源记录](../sources/2026-10-08-cuda-container-compatibility.md)。

[cuda-why]: https://docs.nvidia.com/deploy/cuda-compatibility/latest/why-cuda-compatibility.html
[cuda-minor]: https://docs.nvidia.com/deploy/cuda-compatibility/latest/minor-version-compatibility.html
[container-install]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
[dgx-software]: https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html
[nvidia-smi]: https://docs.nvidia.com/deploy/nvidia-smi/index.html
