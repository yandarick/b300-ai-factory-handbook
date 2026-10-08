# 2026-10-08｜驱动、CUDA 与容器兼容性核对

核查日期：2026-10-08（UTC）。补齐软件平台页的兼容性核对方法，不宣称当日发布新版本。基线保持 128 台服务器 × 8 张 B300 GPU/台 = 1,024 张 GPU。

## 实际读取的 NVIDIA 官方原文

| 官方页面 | 版本、发布日期或更新日期 | 本次用途与适用范围 |
|---|---|---|
| [DGX B300 Introduction：DGX OS Software](https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html) | HTML 未标独立版号或初始发布日期；页尾更新 2026-10-05 | 核对 DGX B300 预装软件类别；该节未给完整软件版本组合，不外推至 OEM HGX B300。 |
| [nvidia-smi](https://docs.nvidia.com/deploy/nvidia-smi/index.html) | 滚动命令文档，未标独立文档版号、发布日期或更新日期；变更日志的 v595 相对 v590 条目记载字段更名 | GPU Attributes 中 KMD/CUDA UMD 的含义及旧字段名弃用；不据此推荐安装某一驱动版本。 |
| [Why CUDA Compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/latest/why-cuda-compatibility.html) | `latest`；未标独立文档版号或初始发布日期；页尾更新 2026-09-09 | CUDA Toolkit、Runtime、驱动的关系及兼容机制；属于 CUDA 通用说明。 |
| [Minor Version Compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/latest/minor-version-compatibility.html) | `latest`；未标独立文档版号或初始发布日期；页尾更新 2026-09-09 | 小版本兼容的最低驱动、功能、PTX 和目标架构条件；没有把表中通用驱动下限当作 B300 支持清单。 |
| [Installing the NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) | `latest`；当日安装示例使用 `1.20.1-1`，页面未标独立文档版号、发布日期或更新日期 | 主机 GPU 驱动前提及容器引擎配置关系；只读原文，未执行其中的安装或配置命令，也不将示例版本作为项目选型。 |

以上链接为可变页面；核查日期和页尾更新日期不等于软件发布日期。

## 内容变化与验证边界

- 在[软件平台](../05-software/software-stack.md)补充三层软件的职责、nvidia-smi 字段解释和从整机基线到应用验证的核对顺序；同步首页入口与来源索引。
- 区分官方兼容机制与项目建议，明确 CUDA Version/CUDA UMD Version 不能作为 Toolkit 或容器依赖清单，通用兼容规则也不能代替 B300 整机支持说明。
- 本次未运行查询命令、容器、应用或硬件测试，未安装、升级或部署；没有 B300 实测数据，未确认任何现场驱动/镜像组合。未新增图解。
