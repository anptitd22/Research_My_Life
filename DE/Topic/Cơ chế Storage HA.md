Đa số cơ chế các storage trong hệ thống HA sẽ scale n node và có m rep (m <= n) phân bổ trong các node
- MinIO: các node sẽ active-active, cho phép Read, Write song song phân tán giữa các node
- Hadoop: các node sẽ activate-standby