Mỗi định dạng ghi nhật ký nhị phân đều có ưu điểm và nhược điểm riêng. Đối với hầu hết người dùng, định dạng sao chép hỗn hợp sẽ cung cấp sự kết hợp tốt nhất giữa tính toàn vẹn dữ liệu và hiệu suất. Tuy nhiên, nếu bạn muốn tận dụng các tính năng đặc thù của định dạng sao chép dựa trên câu lệnh hoặc dựa trên hàng khi thực hiện một số tác vụ nhất định, bạn có thể sử dụng thông tin trong phần này, tóm tắt những ưu điểm và nhược điểm tương đối của chúng, để xác định định dạng nào phù hợp nhất với nhu cầu của bạn.

- [Advantages of statement-based replication](https://dev.mysql.com/doc/refman/9.7/en/replication-sbr-rbr.html#replication-sbr-rbr-sbr-advantages "Advantages of statement-based replication")
- [Disadvantages of statement-based replication](https://dev.mysql.com/doc/refman/9.7/en/replication-sbr-rbr.html#replication-sbr-rbr-sbr-disadvantages "Disadvantages of statement-based replication")
- [Advantages of row-based replication](https://dev.mysql.com/doc/refman/9.7/en/replication-sbr-rbr.html#replication-sbr-rbr-rbr-advantages "Advantages of row-based replication")
- [Disadvantages of row-based replication](https://dev.mysql.com/doc/refman/9.7/en/replication-sbr-rbr.html#replication-sbr-rbr-rbr-disadvantages "Disadvantages of row-based replication")

### Advantages of statement-based replication

- Lượng dữ liệu được ghi vào tập tin nhật ký ít hơn. Khi các thao tác cập nhật hoặc xóa ảnh hưởng đến nhiều hàng, điều này dẫn đến việc cần ít không gian lưu trữ hơn cho tập tin nhật ký. Điều này cũng có nghĩa là việc sao lưu và khôi phục từ bản sao lưu có thể được thực hiện nhanh hơn.
- Các tập tin nhật ký chứa tất cả các câu lệnh đã thực hiện bất kỳ thay đổi nào, vì vậy chúng có thể được sử dụng để kiểm tra cơ sở dữ liệu.

### Disadvantages of statement-based replication

- Các câu lệnh không an toàn cho SBR. Không phải tất cả các câu lệnh sửa đổi dữ liệu (như các câu lệnh INSERT, DELETE, UPDATE và REPLACE) đều có thể được sao chép bằng phương pháp sao chép dựa trên câu lệnh. Bất kỳ hành vi không xác định nào đều khó sao chép khi sử dụng phương pháp sao chép dựa trên câu lệnh. Ví dụ về các câu lệnh Ngôn ngữ sửa đổi dữ liệu (DML)

- Lệnh INSERT ... SELECT yêu cầu số lượng khóa cấp hàng nhiều hơn so với sao chép dựa trên hàng.

- Các câu lệnh UPDATE yêu cầu quét toàn bộ bảng (vì không sử dụng chỉ mục trong mệnh đề WHERE) phải khóa nhiều hàng hơn so với sao chép dựa trên hàng.

- Đối với InnoDB: Câu lệnh INSERT sử dụng AUTO_INCREMENT sẽ chặn các câu lệnh INSERT khác không xung đột.

### Advantages of row-based replication

- Tất cả các thay đổi đều có thể được sao chép. Đây là hình thức sao chép an toàn nhất.
- Số lượng khóa hàng cần thiết trên nguồn sẽ ít hơn, do đó đạt được khả năng xử lý đồng thời cao hơn đối với các loại câu lệnh sau:
	- - [`INSERT ... SELECT`](https://dev.mysql.com/doc/refman/9.7/en/insert-select.html "15.2.7.1 INSERT ... SELECT Statement")
	- [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement") statements with `AUTO_INCREMENT`
	- [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement") or [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement") statements with `WHERE` clauses that do not use keys or do not change most of the examined rows.
- Số lượng khóa hàng cần thiết trên bản sao sẽ ít hơn đối với bất kỳ câu lệnh INSERT, UPDATE hoặc DELETE nào.

### Disadvantages of row-based replication

- RBR có thể tạo ra nhiều dữ liệu hơn cần được ghi vào nhật ký. Để sao chép một câu lệnh DML (chẳng hạn như câu lệnh UPDATE hoặc DELETE), sao chép dựa trên câu lệnh chỉ ghi câu lệnh đó vào nhật ký nhị phân. Ngược lại, sao chép dựa trên hàng ghi từng hàng đã thay đổi vào nhật ký nhị phân. Nếu câu lệnh thay đổi nhiều hàng, sao chép dựa trên hàng có thể ghi nhiều dữ liệu hơn đáng kể vào nhật ký nhị phân; điều này đúng ngay cả đối với các câu lệnh đã được hoàn tác. Điều này cũng có nghĩa là việc tạo và khôi phục bản sao lưu có thể mất nhiều thời gian hơn. Ngoài ra, nhật ký nhị phân bị khóa trong thời gian dài hơn để ghi dữ liệu, điều này có thể gây ra các vấn đề về đồng thời. Sử dụng binlog_row_image=minimal để giảm thiểu đáng kể nhược điểm này.
- Các hàm có thể tải được xác định tạo ra các giá trị BLOB lớn cần nhiều thời gian sao chép hơn với phương pháp sao chép dựa trên hàng so với phương pháp sao chép dựa trên câu lệnh. Điều này là do giá trị cột BLOB được ghi lại, chứ không phải câu lệnh tạo ra dữ liệu.
- Bạn không thể xem trên bản sao những câu lệnh nào đã được nhận từ nguồn và đã được thực thi. Tuy nhiên, bạn có thể xem dữ liệu nào đã được thay đổi bằng cách sử dụng mysqlbinlog với các tùy chọn base64-output=DECODE-ROWS và verbose.
- Ngoài ra, bạn cũng có thể sử dụng biến binlog_rows_query_log_events, nếu được kích hoạt, biến này sẽ thêm sự kiện Rows_query cùng với câu lệnh vào đầu ra mysqlbinlog khi sử dụng tùy chọn vv.
- Đối với các bảng sử dụng công cụ lưu trữ MyISAM, cần có khóa mạnh hơn trên bản sao cho các câu lệnh INSERT khi áp dụng chúng dưới dạng sự kiện dựa trên hàng vào nhật ký nhị phân so với khi áp dụng chúng dưới dạng câu lệnh thông thường. Điều này có nghĩa là việc chèn đồng thời vào các bảng MyISAM không được hỗ trợ khi sử dụng sao chép dựa trên hàng.