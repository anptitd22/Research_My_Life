Khôi phục tại một thời điểm cụ thể đề cập đến việc khôi phục các thay đổi dữ liệu cho đến một thời điểm nhất định. Thông thường, loại khôi phục này được thực hiện sau khi khôi phục bản sao lưu đầy đủ đưa máy chủ về trạng thái tại thời điểm sao lưu được thực hiện. (Bản sao lưu đầy đủ có thể được thực hiện theo nhiều cách, chẳng hạn như các cách được liệt kê trong [Section 9.2, “Database Backup Methods”](https://dev.mysql.com/doc/refman/9.7/en/backup-methods.html "9.2 Database Backup Methods").) Sau đó, khôi phục tại một thời điểm cụ thể sẽ cập nhật máy chủ một cách tăng dần từ thời điể sao lưu đầy đủ đến một thời điểm gần đây hơn (khôi phục từng bước thời điểm tăng dần).

[9.5.1 Point-in-Time Recovery Using Binary Log](https://dev.mysql.com/doc/refman/9.7/en/point-in-time-recovery-binlog.html)

- Recovery binlog theo từng mốc time

[9.5.2 Point-in-Time Recovery Using Event Positions](https://dev.mysql.com/doc/refman/9.7/en/point-in-time-recovery-positions.html)

- Recovery binlog theo từng vị trí

Nên dùng `mysqlbinlog` 