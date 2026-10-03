# DGX B300 128 节点｜供电、制冷与机房容量估算

> 更新日期：2026-10-03  
> 适用基线：128 台 DGX B300（每台 8 GPU，共 1,024 GPU）。  
> 本文用于租赁机房前的容量筛选与方案学习，不替代 NVIDIA / OEM / 机房厂商的施工设计。

## 1. 两种典型部署方式

NVIDIA 2026 年《Data Center Best Practices with DGX B300》给出的典型值：

| 项目 | 高密度方案 | 低密度方案 |
|---|---:|---:|
| 每柜 DGX B300 | 4 台 | 2 台 |
| 供电 | DC Busbar + Power Shelves | AC Rack PDU |
| 单柜平均功率 | 58 kW | 30 kW |
| 单柜峰值功率 | 76 kW | 39.4 kW |
| 机柜 | 48U | 48U |
| 冷却 | 风冷 + Active RDHx | 风冷 |
| 单柜风量 | 6200 CFM | 4200 CFM |

### 128 台的服务器侧数量级

高密度 4 台/柜：
- 32 个计算柜。
- 计算柜平均功率约 1.856 MW。
- 计算柜峰值功率约 2.432 MW。
- 计算柜总风量参考约 198,400 CFM。

低密度 2 台/柜：
- 64 个计算柜。
- 计算柜平均功率约 1.920 MW。
- 计算柜峰值功率约 2.522 MW。
- 计算柜总风量参考约 268,800 CFM。

以上仍未计入计算网络、存储网络、管理节点、存储阵列、UFM、交换机、CDU、RDHx 自身用电等。

## 2. 单台 DGX B300 物理参数

NVIDIA DGX B300 User Guide 当前列出的典型参数：

- 10U。
- 8 × B300 GPU。
- PSU 版系统重量约 168 kg。
- DC Busbar 版系统重量约 123 kg。
- User Guide 给出系统功耗 14.5 kW；2026 Data Center Best Practices 对 AC 版本给出约 15 kW 平均、19.7 kW 峰值，对 DC 版本给出约 14.5 kW 平均、19 kW 峰值。
- 最大热输出约 49,476 BTU/hr。
- 机身深度约 904 mm。
- BMC 独立 1GbE 管理口。
- 计算网络：8 × 800Gb/s ConnectX-8。
- 另有 BlueField-3 DPU 用于存储与管理网络。

## 3. 高密度方案的优点与代价

如果按 4 台/柜，高密度方案只需要约 32 个计算柜，可缩短 800G compute fabric 的物理跨度与平均线长。

代价是：
- 58 kW 平均、76 kW 峰值/柜不是普通企业机房的常规能力；
- NVIDIA 推荐 Active Rear Door Heat Exchanger（RDHx）；
- 需要 DC Busbar、Power Shelf、CDU/冷却水系统配合；
- 柜体和设备重量、RDHx 开门空间、管路、漏液检测、维护通道必须一起设计。

高密度不是简单把服务器堆得更紧，而是把供电、制冷、机柜、管路和网络一起升级为 AI Factory 级基础设施。

## 4. 租机房时先要的 12 个数据

1. 可立即交付的 IT 电力容量（MW）。
2. 可扩容电力容量，以及扩容需要多久。
3. 单柜持续可交付功率（kW/rack）。
4. 单柜峰值/短时能力。
5. A/B 双路是否来自真正独立的 UPS/配电路径。
6. 400/415V 三相供电是否可提供。
7. 是否支持 DC Busbar + Power Shelf 架构。
8. 冷却方式：传统 CRAC/CRAH、RDHx、CDU、设施水。
9. 可提供的供水温度、回水温度、流量、压力和水质标准。
10. 楼板承重、机柜重量限制和运输路径限制。
11. 机柜区与网络/存储区的距离，以及上走线/下走线能力。
12. 未来扩到 144 / 288 节点时，电力、冷却、空间是否还能连续扩展。

## 5. 供电设计要点

NVIDIA 高密度参考：
- 每柜 4 个 1U 33 kW DC Power Shelves。
- 4 台 B300 时至少 3 个 Power Shelves 需要保持 active，形成 N+1。
- 每个 Power Shelf 有冗余输入，形成 2N 输入路径。
- 高密度参考要求每个 utility source 提供 4 条 60A 三相回路，最低 400V。

低密度参考：
- 每柜 2 台 B300。
- Rack PDU 2N。
- 每个 utility source 1 条 60A 三相回路，最低 400V。

施工前必须由电气工程师结合当地规范、breaker derating、UPS/发电机策略和选择性保护重新计算。

## 6. 制冷设计不是只看“总冷量”

NVIDIA B300 数据中心实践文档指出：
- 高密度 4 台/柜推荐使用 Active RDHx。
- Passive RDHx 不推荐用于 DGX B300。
- 每台 B300 的送风量可达到约 2145 CFM（海平面条件下的上限量级）。
- 还要考虑过滤、洁净度、热/冷通道、RDHx 开门维护空间、CDU 布置、管路、阀门和漏液检测。

风冷服务器 + RDHx 仍需要把热量最终带出机房，因此设施水系统能力仍然决定高密度方案能不能真正落地。

## 7. 64 节点高密度示例的用途

NVIDIA 文档给了一个“典型 64 节点高密度 SU”设施级示例，包含：
- 16 个 DGX B300 计算柜；
- 8 个 Network / Management / Storage 机柜；
- 2 个 CDU；
- 24 个 RDHx；
- 总平均需求约 975 kVA；
- 总峰值约 1450 kVA。

它适合做机房容量级 benchmark，但不能直接当作 72-node XDR SU 的最终 BOM，因为当前 XDR SuperPOD 网络参考架构以 72 台为一个 SU，而 Spectrum-X 文档中又可见 64 节点 SU。

## 8. 128 台项目当前建议

如果走 XDR InfiniBand：
- Compute Fabric 按 2 × 72-node XDR SU（144 节点容量）规划。
- 首期安装 128 台，预留 16 台位置和端口。
- 物理上优先把计算柜、Leaf、Spine、线槽和光纤路径按完整 SU 规划。
- 机房容量目标不要只按 1.856 MW 服务器平均功耗；还要为网络、存储、管理、RDHx/CDU、冗余和扩容留足空间。

如果走 Spectrum-X：
- NVIDIA 当前 B300 数据中心实践资料把 Spectrum-X SU 标为最多 64 台。
- 128 台可以自然分成 2 × 64-node。
- 是否选择 Spectrum-X，要结合团队对 RoCE/ECN/PFC、Ethernet 运维能力、成本和生态要求决定。

## 9. 租机房前的一句话原则

**先把“单柜多少 kW、总 IT MW、怎么冷、怎么扩、800G 光纤怎么走”搞清楚，再谈机柜租金。**

## 官方资料

1. NVIDIA, Data Center Best Practices with DGX B300, Version 1.0, 2026-02-27  
   https://docs.nvidia.com/dgx-pdf/data-center-best-practices-with-dgx-b300-v1.pdf

2. NVIDIA DGX B300 User Guide  
   https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html

3. NVIDIA DGX SuperPOD with DGX B300 + Quantum-X800 XDR Reference Architecture  
   https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/
