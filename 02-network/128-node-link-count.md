# 128 节点：800G 端口、链路与线缆怎样计数

> 核查日期：2026-10-06（UTC）。项目基线仍为 **128 台服务器 × 8 GPU/台 = 1,024 GPU**。本页以 DGX B300、每个计算接口使用单条 800 Gb/s XDR InfiniBand 链路为前提；不直接套用到 OEM HGX B300、拆分端口或 Spectrum-X 配置。

## 先确定数的是什么

**官方事实：** DGX B300 用户指南列出每台有 8 个 OSFP 计算网络接口，连接 8 个 ConnectX-8，标称为 8 × 800 Gb/s；存储与管理网络另列。OSFP 是接口/模块外形规格，GPU 数、接口数和线缆数需要分别记账。[DGX B300 用户指南](https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html)（核查：2026-10-06）。

**本页计数约定：** endpoint 指服务器侧计算端口；一条物理链路连接服务器端口与 Leaf（接入交换机）端口，两端各占一个端口。Spine（骨干交换机）与 Leaf 之间的链路单独计算；收发两个方向不算成两条物理链路。

以下为上述接口规格的**条件算术推导**，假设所有计算接口均接通，且没有端口拆分：

| 对象与算式 | 128 台实际规模 | 144 台满配规模 |
|---|---:|---:|
| GPU：服务器数 × 8 GPU/台 | 1,024 GPU | 1,152 GPU |
| 服务器侧计算端口：服务器数 × 8 端口/台 | 1,024 个 | 1,152 个 |
| Node–Leaf 物理链路：每个服务器端口接 1 条 | 1,024 条 | 1,152 条 |
| Leaf 侧被这些链路占用的 800G 端口：每条占 1 个 | 1,024 个 | 1,152 个 |
| 上述链路两端端口合计：链路数 × 2 端口/条 | 2,048 个 | 2,304 个 |

这里 GPU 数与服务器计算端口数相同，是因为本例两者恰好都是每台 8 个，不能用 GPU 数替代接口规格。两端端口合计也不是链路数或光模块采购数量。

## 两 SU 的容量为何还要另列

**官方事实：** DGX B300 + Quantum-X800 XDR + AC 供电参考架构的 Table 3 列出两 SU（Scalable Unit，可扩展单元）为 144 台服务器、16 台 Leaf、8 台 Spine，`Cable Count` 中 Node–Leaf 与 Leaf–Spine 各为 1,152。该表是满配示例，不能把 Node–Leaf 一列当作整个计算网络的线缆总数。[RA：DGX SuperPOD Architecture](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/dgx-superpod-architecture.html)（核查：2026-10-06）。

**官方原则与本项目应用：** 官方要求非满 SU 部署仍按完整 SU 设计 Leaf/Spine 交换机及其互连线缆，缺少节点的位置留空。因此，若本项目选用这份 XDR 架构，128 台对应两 SU 容量，预留 144 − 128 = 16 台位置，即 16 台 × 8 端口/台 = 128 个服务器计算端口的未来接入容量；这些不是已接通的链路。Leaf–Spine 部分不应按 128/144 的比例自行缩减。[RA：Network Fabrics](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/network-fabrics.html)（核查：2026-10-06）。

## 800G 是哪个方向、什么单位

**官方事实：** XDR 的标称峰值为每方向 800 Gb/s。[RA：InfiniBand Technology](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/dgx-superpod-components.html)（核查：2026-10-06）。据此作十进制理论换算，`b` 是 bit，`B` 是 byte，1 B = 8 bit：

- 单链路单向：800 Gb/s ÷ 8 = 100 GB/s。
- 单节点所有计算端口的单向标称合计：8 × 800 Gb/s = 6,400 Gb/s = 800 GB/s。
- 单节点若将收、发两方向相加：2 × 800 GB/s = 1,600 GB/s，必须标为“双向合计”，不能当作单向发送能力。

这些只是端口标称速率之和，不是 NCCL 或应用实测吞吐量；协议开销、通信模式、拥塞及主机数据路径都会影响可用性能，也不能据此直接断言集群任意两组节点之间的带宽。

## 从计数表走到物料清单

**本项目建议顺序：** 先确认服务器型号和端口模式，再按 Node–Leaf、Leaf–Spine 分别制作端口对照表（设备、端口、对端、速率、已用/预留），最后根据模块形态、线缆组件、长度和备件要求形成 BOM（物料清单）。链路、端口、模块、线缆组件各列一栏；不能仅凭“1,024 个端点”确定采购件数。存储、管理与 OOB 网络另列。

用户指南要求按 ConnectX 型号及其实际固件版本查询兼容线缆/交换机列表；本次没有取得现场固件版本、模块型号和布线表，因此不确认任何具体线缆兼容性或最终 BOM。[DGX B300：Supported Network Cables and Adapters](https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html)（核查：2026-10-06）。

本页仅作资料核查与容量计算，没有执行设备命令、布线或 B300 性能测试。文档版本、发布日期与核查边界见[本次来源记录](../sources/2026-10-06-network-counting.md)。
