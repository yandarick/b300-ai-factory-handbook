# 2026-10-10｜本地 NVMe 缓存与持久检查点

核查日期：2026-10-10（UTC）。补齐存储正文的数据存放与恢复边界，不宣称当日发布新产品或软件。基线保持 128 台服务器 × 8 张 B300 GPU/台 = 1,024 张 GPU。

## 本次采用的 NVIDIA 官方原文

| 官方页面 | 版本、发布日期或更新日期 | 本次用途与适用范围 |
|---|---|---|
| [DGX B300 Introduction](https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html) | HTML 未标独立版号或初始发布日期；页尾更新 2026-10-05 | 区分 DGX B300 启动存储与缓存存储；不外推至 OEM HGX B300。 |
| [DGX OS 7：System Configurations](https://docs.nvidia.com/dgx/dgx-os-7-user-guide/system_configurations.html) | DGX OS 7 用户指南；本章未标独立版号或初始发布日期；页尾更新 2026-09-30 | Data Storage Configuration 与 Changing the RAID Configuration for Data Drives：默认缓存用途、RAID 0 无冗余及单盘故障后果；不认定现场仍为默认配置。 |
| [XDR/AC 参考架构首页](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/index.html) | RA-11339-001 V01；发布日期 2025-07-23；页尾更新 2026-09-02 | 核对下列章节适用于 DGX B300 + Quantum-X800 InfiniBand + AC 供电架构。 |
| [High Performance Storage Architecture](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300-xdr/latest/storage-architecture.html) | 同上；本章页尾更新 2026-09-02 | 本地 NVMe 缓存/暂存、共享数据访问和检查点 I/O 对训练的影响；未新增吞吐承诺。 |

以上均为可变页面；核查日期和页尾更新日期不等于硬件或软件发布日期。

## 内容变化与验证边界

- 在[存储架构](../03-storage/storage-architecture.md)补充本地缓存与持久检查点的区别、数据存放原则、落点核对、完成判据和跨节点恢复验收顺序；同步首页与来源索引。
- 官方默认配置与项目建议分别标明；明确本地容量相加不自动构成共享存储，共享访问也不证明具备备份。保留既有吞吐指导页，未改变其数字或 SU 口径。
- 本次仅阅读官方原文并检查文档，未执行存储或训练命令、安装、部署、RAID 变更或硬件操作；没有 B300 实测数据，也未验证现场文件系统、框架完成语义、数据保护策略和恢复能力。未新增图解或测试文件。
