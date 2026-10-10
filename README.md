# B300 AI Factory Handbook

[在线阅读](https://learn.rickcn.cn/b300/)

面向从零学习、设计、建设和运维 NVIDIA B300 大规模 GPU 集群的中文知识库。

**当前设计基线：128 台 8-GPU B300 节点 = 1,024 GPU，并预留继续横向扩展能力。**

> “128 台 B300”与“128 张 B300 GPU”完全不同。本项目默认前者；实际采购口径变化时必须重新计算网络、供电、制冷、机柜和存储规模。

## 学习地图

1. 小白入门：`00-beginner/`
2. 128 节点总体架构：`01-architecture/`
3. 网络架构：[网络总览](02-network/network-overview.md) · [800G 端口与链路计数](02-network/128-node-link-count.md)
4. 存储架构：[本地 NVMe 缓存与持久检查点](03-storage/storage-architecture.md) · [按参考架构区分的吞吐指导值](03-storage/b300-storage-targets-note.md)
5. 机房设计：`04-datacenter/`
6. 软件平台：[驱动、CUDA 与容器兼容性核对](05-software/software-stack.md)
7. 运维体系：[Day-2 运维与 DCGM 检查边界](06-operations/operations.md)
8. 部署与验收：[部署流程与 NCCL AllReduce 结果判读](07-deployment/deployment-and-acceptance.md)
9. 128 节点实例：`08-design-example/`
10. 官方资料与变更记录：`sources/`

## 核心问题

- 128 节点如何分组、形成故障域和扩容单元？
- Scale-up（节点内 NVLink/NVSwitch）与 Scale-out（节点间 RDMA）分别解决什么？
- InfiniBand XDR 与 Spectrum-X Ethernet 怎么选？
- 1,024 GPU 需要多少 800G 端口、交换机和光连接？
- Storage / Compute / In-band / OOB 网络为什么分开？
- 机房需要多少 MW、每柜多少 kW、怎样供电和冷却？
- Slurm、Kubernetes、BCM/Mission Control、DCGM、UFM 各负责什么？
- 怎样做 burn-in、NCCL、RDMA、存储和故障切换验收？
- 从 128 扩到 256/512 节点时，哪些基础设施第一天就必须预留？

## 原则

**先做架构和容量模型，再选机房。** AI Factory 的限制往往不是 U 位，而是单柜功率、冷却能力、供电路径、承重、800G 布线距离和网络扩展边界。

本仓库持续更新。采购和施工前，参数必须以当时 NVIDIA、服务器 OEM、网络、存储和机房厂商正式设计文件为准。

## 核对网站阅读版本

网站从本仓库的 `main` 构建。打开[网站来源清单](https://learn.rickcn.cn/build-info.json)，找到 `repo` 为 `yandarick/b300-ai-factory-handbook` 的记录，将其 `sha` 与 [main 的最近提交](https://github.com/yandarick/b300-ai-factory-handbook/commits/main/)对照；文章顶部“查看原始笔记”链接也应指向该来源提交。

PR 已推送、合并到主线和网站成功发布是不同状态。若来源 SHA 尚未更新，网站可能仍在展示上一次成功构建；构建时间也不等于文章技术核验日期。这一核对方法依据[网站同步实现](https://github.com/yandarick/rick-ai-learning-site/blob/f52bcdf8462edddac93ac4f9fbd06505bcf501f9/scripts/sync-content.mjs)及线上来源清单，核查日期：2026-10-04。
