# 06｜运维体系

目标不是“服务器能 ping”，而是持续保持 GPU useful throughput。

监控 GPU temperature/power/ECC/Xid、NVLink/NVSwitch、NIC/RDMA、交换机拥塞/errors、NCCL、CPU/RAM/NVMe、存储、BMC/PSU、rack PDU、CDU/liquid loop、环境与 scheduler utilization。

工具方向：DCGM、Prometheus/Grafana、UFM（IB）、switch telemetry、central logs、alert/ticketing、配置与 firmware inventory。
