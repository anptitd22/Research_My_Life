# Bridge network driver

A Docker bridge network có một IPv4 subnet và, tùy chọn, một mạng IPv6 subnet. Mỗi container được kết nối với bridge network đều có một giao diện mạng với các địa chỉ trong các mạng con của mạng đó. Theo mặc định:
- Cho phép truy cập mạng không hạn chế vào các container trong mạng từ máy chủ và từ các container khác được kết nối với cùng một bridge network.
- Chặn truy cập từ các container trong mạng khác và từ bên ngoài máy chủ Docker.
- Sử dụng kỹ thuật che giấu địa chỉ IP để cấp quyền truy cập mạng bên ngoài cho các container. Các thiết bị trên mạng bên ngoài của máy chủ chỉ thấy địa chỉ IP của máy chủ Docker.
- Hỗ trợ công bố cổng, trong đó lưu lượng mạng được chuyển tiếp giữa các cổng của container và các cổng trên địa chỉ IP của máy chủ. Các cổng đã công bố có thể được truy cập từ bên ngoài máy chủ Docker, thông qua địa chỉ IP của nó.

[Differences between user-defined bridges and the default bridge](https://docs.docker.com/engine/network/drivers/bridge/#differences-between-user-defined-bridges-and-the-default-bridge)

**User-defined bridges provide automatic DNS resolution between containers**.


