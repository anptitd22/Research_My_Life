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
- Các phương pháp tạo bản sao lưu
- Các phương pháp phục hồi, bao gồm phục hồi tại một thời điểm cụ thể.
- Lập lịch sao lưu, nén và mã hóa
- Bảo trì bảng, để cho phép khôi phục các bảng bị hỏng.