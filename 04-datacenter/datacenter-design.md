# 04｜机房设计

**先确定 IT load / rack density / cooling architecture，再租机房。** 128 台高密度 B300 节点属于 MW 级项目，传统低密度企业机房不能直接套用。

IT Load = GPU Compute + Compute Network + Storage + Storage Network + Management/Control + Security/Services。

## 必查
- 电力：utility/generator、UPS、A/B feed、busway/PDU、breaker、rack kW、grounding、扩容余量。
- 冷却：OEM thermal requirement、air vs DLC、CDU、facility water loop、温度/流量/压力、N+1、leak detection。
- 物理：rack U/depth/width、重量、楼板承重、运输路径、冷热通道、维护空间、线槽和光纤弯曲半径。
- 安全：消防、漏水、门禁/CCTV、环境传感器、EPO policy。

早期只做数量级估算；施工参数必须来自所选 DGX/HGX OEM 最新 site planning 文档。
