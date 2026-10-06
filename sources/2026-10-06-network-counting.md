# 2026-10-06｜DGX B300 计算网络的计数口径

核查日期：2026-10-06（UTC）。本次补齐现有链路计数页的解释，不宣称当日发布新硬件或参考架构。基线保持 128 台 × 8 GPU/台 = 1,024 GPU。

## 实际读取的 NVIDIA 官方原文

| 官方页面 | 版本、发布日期或更新日期 | 本次用途与适用范围 |
|---|---|---|
| [DGX B300 Introduction](https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html) | HTML 未标出版号及初始发布日期；页尾更新 2026-10-05 | DGX B300 的 8 个 OSFP / ConnectX-8 计算接口、标称速率及按固件查线缆兼容性的要求；不代替 OEM HGX 整机规格。 |
| [DGX B300 User Guide 首页](https://docs.nvidia.com/dgx/dgxb300-user-guide/index.html) | HTML 未标出版号及初始发布日期；页尾更新 2026-10-05 | 核对指南入口与文档范围。 |
| [XDR / AC RA 首页](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/index.html) | RA-11339-001 V01；发布日期 2025-07-23；页尾更新 2026-09-02 | 确定下列三章属于 DGX B300 + Quantum-X800 InfiniBand + AC 供电参考架构。 |
| [DGX SuperPOD Architecture](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/dgx-superpod-architecture.html) | 同上；本章页尾更新 2026-09-02 | Table 3 的两 SU、节点、交换机及两段 Cable Count。 |
| [Network Fabrics](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/network-fabrics.html) | 同上；本章页尾更新 2026-09-02 | 非满 SU 仍按完整 SU 设计交换机和 Leaf–Spine 线缆的原则。 |
| [Key Components](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/dgx-superpod-components.html) | 同上；本章页尾更新 2026-09-02 | InfiniBand Technology 小节的 XDR 每方向 800 Gb/s 标称值。 |

页尾更新日期不代表各项规格的首次发布日期；`latest` 路径可能继续变化。

## 内容变化与验证边界

- 扩充[现有链路计数页](../02-network/128-node-link-count.md)，保留 128 台对应 1,024 个服务器计算端点、144 台对应 1,152 个端点的正确结论，补充单条 800G XDR 链路前提及两端端口计数。
- 区分实际接入与两 SU 预留容量，解释 Node–Leaf 与 Leaf–Spine 线缆分开列账，以及 Gb/s、GB/s 和单向/双向的理论换算；同步首页、网络总览和来源索引。
- 数量与带宽计算是明确假设下的算术推导；没有在 B300 上实测，没有执行安装、配置或硬件操作，也未核验现场端口模式、固件/模块组合和最终采购清单。本次未新增图解。
