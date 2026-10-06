# Sources / 资料来源

优先级：NVIDIA 官方 reference architecture / deployment guide / product documentation → OEM 正式设计指南 → 网络与存储厂商 validated design → 行业标准。

每次关键架构参数更新时记录：来源、文档版本、发布日期/访问日期、适用产品、是否为估算。

## 重点跟踪
- NVIDIA DGX B300 User Guide
- NVIDIA DGX BasePOD / B300 Deployment Guides
- NVIDIA Enterprise Reference Architectures: HGX B300 + Spectrum-X
- NVIDIA Spectrum-X Solution Stack
- NVIDIA Mission Control / Base Command Manager
- NVIDIA DCGM
- NVIDIA UFM / InfiniBand documentation

## 变更记录

- 2026-10-06：[DGX B300 计算网络的计数口径](2026-10-06-network-counting.md)；补充端点、链路两端端口、两 SU 预留及单向/双向带宽换算，正文见[800G 端口与链路计数](../02-network/128-node-link-count.md)。

- 2026-10-04：在[首页](../README.md#核对网站阅读版本)补充网站来源 SHA、主线提交和原始笔记链接的核对方法，避免将已提交或已合并误认为已发布；依据为网站同步实现及线上来源清单。

- 2026-10-04：[DCGM 在线健康监控与维护窗口诊断](2026-10-04-dcgm.md)；补充运行时机、排空作业顺序和结果判读，正文见[运维体系](../06-operations/operations.md)。

最后更新：2026-10-06
