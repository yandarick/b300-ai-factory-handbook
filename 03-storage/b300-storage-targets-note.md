# B300 Storage Targets：按 Reference Architecture 区分 SU

> 核对日期：2026-10-04。以下为 NVIDIA 官方 Reference Architecture 的容量指导，不等于本项目最终采购指标。

NVIDIA 当前 DGX B300 SuperPOD 资料给出的单 SU 高性能存储吞吐指导值为：

| Workload | 单 SU 聚合读取 | 单 SU 聚合写入 |
|---|---:|---:|
| Standard | 80 GB/s | 40 GB/s |
| Enhanced | 250 GB/s | 124 GB/s |

## 为什么不能把“单 SU”固定理解成 72 台

B300 的 SU 不是跨产品线统一数字：

- DGX B300 SuperPOD + Quantum-X800 XDR InfiniBand / AC Power：1 SU = 72 台 DGX B300。
- DGX B300 SuperPOD + Spectrum-X Ethernet / DC Busbar：1 SU = 64 台 DGX B300。
- OEM HGX B300 Enterprise Reference Architecture 又采用 4 台服务器 / SU 的扩展单元。

因此，80/40 GB/s 或 250/124 GB/s 应理解为对应 DGX SuperPOD RA 的“单 SU 指导值”，不能脱离所选 RA 换算成固定的每节点性能，也不能在尚未确定 DGX/HGX、XDR/Spectrum-X 之前直接线性换算 128 台项目。

## 128 台项目怎么用这些数字

这些数字适合做早期 sizing 起点，不应直接变成采购指标。最终需要用真实 workload 验证：dataset streaming、metadata/small-file workload、checkpoint burst/restore、model weight loading、多租户并发读写、故障重建与降级状态，以及训练期间的尾延迟和 data stall。

XDR/AC Reference Architecture 的 Network Fabrics 页面还指出，DGX SuperPOD 的节点 I/O 需求需要超过 80 GB/s。设计存储网络时应同时核对节点侧接口、Storage Fabric 拓扑、并发模型和文件系统能力，而不是只看后端阵列峰值带宽。

## 官方资料

- https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/
- https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/network-fabrics.html
- https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300/latest/
- https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300/latest/storage-architecture.html
