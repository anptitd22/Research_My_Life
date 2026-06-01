# Iceberg Nessie Integration

Iceberg tích hợp với Nessie thông qua mô-đun `iceberg-nessie`. Phần này mô tả cách sử dụng Iceberg với Nessie. Nessie cung cấp một số tính năng quan trọng bổ sung cho Iceberg.
- multi-table transactions
	- **Ví dụ:** Bạn có một pipeline dữ liệu cần cập nhật đồng thời cả bảng `Fact_Sales` (Doanh số) và bảng `Dim_Customers` (Khách hàng). Nếu quá trình ghi vào bảng `Dim_Customers` bị lỗi, Nessie sẽ rollback (hoàn tác) dữ liệu trên _cả hai bảng_, đảm bảo dữ liệu giữa các bảng luôn đồng bộ và nhất quán tuyệt đối.
- git-like operations (eg branches, tags, commits)
- hive-like metastore capabilities (Khả năng làm Metastore thay thế Hive)