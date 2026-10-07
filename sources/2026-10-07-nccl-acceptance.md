# 2026-10-07｜NCCL AllReduce 验收结果判读

核查日期：2026-10-07（UTC）。补齐部署验收页的指标解释，不宣称当日发布新版本。基线保持 128 台 × 8 GPU/台 = 1,024 GPU。

## 实际读取的 NVIDIA 官方原文

| 官方页面 | 版本、发布日期或更新日期 | 本次用途与适用范围 |
|---|---|---|
| [NCCL Collective Operations](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/collectives.html) | 页面标题为 NCCL 2.32.3；正文与页尾未提供发布日期或更新日期 | AllReduce 与 rank 的含义；属于 NCCL 通用语义，不是 B300 兼容矩阵。 |
| [nccl-tests README](https://github.com/NVIDIA/nccl-tests/blob/master/README.md) | 当日读取的 `master`；正文未标独立版号或发布日期，读取页面未显示文件提交日期 | rank 数、分组结果范围、MPI 前提及校验参数；不据此指定生产安装版本。另读取了页面链接的 Raw 原文。 |
| [nccl-tests Performance](https://github.com/NVIDIA/nccl-tests/blob/master/doc/PERFORMANCE.md) | 当日读取的 `master`；正文未标独立版号或发布日期，读取页面未显示文件提交日期 | algbw、AllReduce busbw 的公式及大小消息的观察重点；另读取了页面链接的 Raw 原文。 |

以上均为可变路径；核查日期不是软件发布日期。实际验收须留存所用 NCCL 版本与 nccl-tests 提交号，并以实际输出表头确认时间单位。

## 内容变化与验证边界

- 在[部署与验收](../07-deployment/deployment-and-acceptance.md)补充 AllReduce 指标、参与 GPU 范围和验收阅读顺序；同步首页入口与来源索引。
- 明确每 rank 数据量、十进制 GB/s、GB/GiB 差异，以及 busbw 与物理端口、双向速率的区别；8-rank 数字仅为假设算例，不是 B300 性能预测或实测。
- 校验结果、同条件基线比较、分层扩大测试范围与维护资源安排分别注明官方依据或项目建议。没有执行 NCCL、安装、部署或硬件操作；未验证 B300 软件兼容性和性能门槛，本次未新增图解。
