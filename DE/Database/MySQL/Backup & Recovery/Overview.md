**Table of Contents**

[9.1 Backup and Recovery Types](https://dev.mysql.com/doc/refman/9.7/en/backup-types.html)

[9.2 Database Backup Methods](https://dev.mysql.com/doc/refman/9.7/en/backup-methods.html)

[9.3 Example Backup and Recovery Strategy](https://dev.mysql.com/doc/refman/9.7/en/backup-strategy-example.html)

[9.4 Using mysqldump for Backups](https://dev.mysql.com/doc/refman/9.7/en/using-mysqldump.html)

[9.5 Point-in-Time (Incremental) Recovery](https://dev.mysql.com/doc/refman/9.7/en/point-in-time-recovery.html)

[9.6 MyISAM Table Maintenance and Crash Recovery](https://dev.mysql.com/doc/refman/9.7/en/myisam-table-maintenance.html)

Việc sao lưu cơ sở dữ liệu rất quan trọng để bạn có thể khôi phục dữ liệu và tiếp tục hoạt động trong trường hợp xảy ra sự cố, chẳng hạn như lỗi hệ thống, hỏng phần cứng hoặc người dùng xóa dữ liệu nhầm. Sao lưu cũng rất cần thiết như một biện pháp bảo vệ trước khi nâng cấp cài đặt MySQL, và chúng có thể được sử dụng để chuyển cài đặt MySQL sang hệ thống khác hoặc để thiết lập máy chủ sao chép.

MySQL cung cấp nhiều chiến lược sao lưu khác nhau, bạn có thể chọn phương pháp phù hợp nhất với yêu cầu của hệ thống. Chương này thảo luận về một số chủ đề sao lưu và phục hồi mà bạn nên nắm rõ.

- Các loại sao lưu: Sao lưu logic so với sao lưu vật lý, sao lưu toàn bộ so với sao lưu tăng dần, v.v.
- Các phương pháp tạo bản sao lưu (backups)
- Các phương pháp phục hồi, bao gồm phục hồi tại một thời điểm cụ thể (point-in-time recovery).
- Lập lịch sao lưu, nén và mã hóa
- Bảo trì bảng, để cho phép khôi phục các bảng bị hỏng.

## Additional Resources

Các nguồn lực liên quan đến sao lưu hoặc duy trì tính khả dụng của dữ liệu bao gồm những điều sau:

- Khách hàng của MySQL Enterprise Edition có thể sử dụng sản phẩm MySQL Enterprise Backup để sao lưu dữ liệu. Để biết tổng quan về sản phẩm MySQL Enterprise Backup, xem [Section 32.1, “MySQL Enterprise Backup Overview”](https://dev.mysql.com/doc/refman/9.7/en/mysql-enterprise-backup.html "32.1 MySQL Enterprise Backup Overview").
- Bạn có thể tìm thấy diễn đàn dành riêng cho các vấn đề sao lưu tại địa chỉ https://forums.mysql.com/list.php?28.
- Thông tin chi tiết về mysqldump có thể được tìm thấy trong [Chapter 6, _MySQL Programs_](https://dev.mysql.com/doc/refman/9.7/en/programs.html "Chapter 6 MySQL Programs").
- Cú pháp của các câu lệnh SQL được mô tả ở đây được trình bày trong [Chapter 15, _SQL Statements_](https://dev.mysql.com/doc/refman/9.7/en/sql-statements.html "Chapter 15 SQL Statements").
- Để biết thêm thông tin về quy trình sao lưu InnoDB, xem [Section 17.18.1, “InnoDB Backup”](https://dev.mysql.com/doc/refman/9.7/en/innodb-backup.html "17.18.1 InnoDB Backup").
- Sao chép dữ liệu cho phép bạn duy trì dữ liệu giống hệt nhau trên nhiều máy chủ. Điều này mang lại một số lợi ích, chẳng hạn như cho phép phân bổ tải truy vấn của máy khách trên các máy chủ, đảm bảo tính khả dụng của dữ liệu ngay cả khi một máy chủ nhất định bị ngoại tuyến hoặc gặp sự cố, và khả năng tạo bản sao lưu mà không ảnh hưởng đến nguồn dữ liệu bằng cách sử dụng bản sao. Xem [Chapter 19, _Replication_](https://dev.mysql.com/doc/refman/9.7/en/replication.html "Chapter 19 Replication").
- MySQL InnoDB Cluster và NDB Cluster

Nguồn: https://dev.mysql.com/doc/refman/9.7/en/backup-and-recovery.html

