# 02｜网络架构

传统企业网关注可达、冗余、安全；AI Fabric 还要关注 NCCL、east-west bandwidth、latency/jitter、RDMA、拥塞控制、oversubscription 和 topology awareness。

## 两条主路线
**InfiniBand**：重点学习 XDR、ConnectX、Quantum-X800、UFM、adaptive routing、SHARP。

**Spectrum-X Ethernet**：重点学习 Spectrum-X、ConnectX-8/SuperNIC、RoCE、ECN/PFC、adaptive routing、telemetry、single/dual/quad-plane。

## 128 节点待计算
- 服务器准确 SKU 与 NIC/port mode
- InfiniBand vs Spectrum-X
- 800G endpoint ports
- rail/plane 数量
- Leaf/Spine 或 fat-tree
- blocking ratio / switch radix
- cable matrix / optics / AOC / DAC
- OOB 与 storage fabric
- IP/VLAN/ASN/loopback
- NTP/DNS/AAA/log/config backup

> 不能简单用 128×8 直接当最终 800G BOM；最终数量取决于服务器 SKU、NIC 模式、plane 和参考架构。
