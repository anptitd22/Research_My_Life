# OS

**Journaling**, hoặc **write-ahead logging**, là một giải pháp tinh vi cho vấn đề không nhất quán hệ thống tệp ([file system inconsistency](https://www.geeksforgeeks.org/operating-systems/file-system-inconsistency/)) trong hệ điều hành. Lấy cảm hứng từ các hệ thống quản lý cơ sở dữ liệu, phương pháp này trước tiên ghi lại tóm tắt các hành động sẽ được thực hiện vào một "log" trước khi thực sự ghi chúng vào đĩa. Do đó có tên là "write-ahead logging". Trong trường hợp xảy ra sự cố, hệ điều hành chỉ cần kiểm tra log này và tiếp tục từ nơi nó đã dừng lại. Điều này giúp tiết kiệm nhiều lần quét đĩa để khắc phục sự không nhất quán, như trường hợp thường gặp phải.

# Postgres

Tham khảo: [[28.3. Write-Ahead Logging (WAL)]]

MySQL

Tham khảo: https://dev.mysql.com/blog-archive/mysql-8-0-new-lock-free-scalable-wal-design/
