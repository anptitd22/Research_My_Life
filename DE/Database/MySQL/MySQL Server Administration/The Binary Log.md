[7.4.4.1 Binary Logging Formats](https://dev.mysql.com/doc/refman/9.7/en/binary-log-formats.html)

[7.4.4.2 Setting The Binary Log Format](https://dev.mysql.com/doc/refman/9.7/en/binary-log-setting.html)

[7.4.4.3 Mixed Binary Logging Format](https://dev.mysql.com/doc/refman/9.7/en/binary-log-mixed.html)

[7.4.4.4 Logging Format for Changes to mysql Database Tables](https://dev.mysql.com/doc/refman/9.7/en/binary-log-mysql-database.html)

[7.4.4.5 Binary Log Transaction Compression](https://dev.mysql.com/doc/refman/9.7/en/binary-log-transaction-compression.html)

The binary log chứa các "sự kiện" mô tả các thay đổi trong cơ sở dữ liệu, chẳng hạn như các thao tác tạo bảng hoặc thay đổi dữ liệu bảng. Nó cũng chứa các sự kiện cho các câu lệnh có khả năng gây ra thay đổi (ví dụ: lệnh DELETE không khớp với bất kỳ hàng nào), trừ khi sử dụng ghi nhật ký theo hàng. The binary log cũng chứa thông tin về thời gian mỗi câu lệnh thực hiện cập nhật dữ liệu. The binary log có hai mục đích quan trọng.

- Đối với replication, the binary log trên máy chủ nguồn sao chép cung cấp bản ghi về các thay đổi dữ liệu cần gửi đến các bản sao. Máy chủ nguồn gửi thông tin có trong nhật ký nhị phân của nó đến các bản sao, các bản sao này sẽ tái tạo lại các giao dịch đó để thực hiện các thay đổi dữ liệu giống như đã được thực hiện trên máy chủ nguồn. Xem [Section 19.2, “Replication Implementation”](https://dev.mysql.com/doc/refman/9.7/en/replication-implementation.html "19.2 Replication Implementation").
- Một số thao tác khôi phục dữ liệu yêu cầu sử dụng nhật ký nhị phân. Sau khi bản sao lưu được khôi phục, các sự kiện trong nhật ký nhị phân được ghi lại sau khi sao lưu được thực thi lại. Các sự kiện này cập nhật cơ sở dữ liệu từ thời điểm sao lưu. Xem [Section 9.5, “Point-in-Time (Incremental) Recovery”](https://dev.mysql.com/doc/refman/9.7/en/point-in-time-recovery.html "9.5 Point-in-Time (Incremental) Recovery").

Việc chạy máy chủ với tính năng ghi nhật ký nhị phân được bật sẽ làm giảm hiệu suất một chút. Tuy nhiên, lợi ích của việc ghi nhật ký nhị phân trong việc cho phép bạn thiết lập sao chép và thực hiện các thao tác khôi phục thường lớn hơn sự suy giảm hiệu suất nhỏ này.

Nhật ký nhị phân có khả năng chống chịu được các sự cố dừng đột ngột. Chỉ những sự kiện hoặc giao dịch hoàn chỉnh mới được ghi vào nhật ký hoặc đọc lại.

Mật khẩu trong các câu lệnh được ghi vào nhật ký nhị phân sẽ được máy chủ viết lại để không xuất hiện nguyên văn dưới dạng văn bản thuần. Xem thêm [Section 8.1.2.3, “Passwords and Logging”](https://dev.mysql.com/doc/refman/9.7/en/password-logging.html "8.1.2.3 Passwords and Logging").

Các tệp nhật ký nhị phân và tệp nhật ký chuyển tiếp của MySQL có thể được mã hóa, giúp bảo vệ các tệp này và dữ liệu nhạy cảm tiềm ẩn trong đó khỏi bị kẻ tấn công bên ngoài lạm dụng, cũng như khỏi việc người dùng hệ điều hành nơi chúng được lưu trữ xem trái phép. Bạn bật tính năng mã hóa trên máy chủ MySQL bằng cách đặt biến hệ thống binlog_encryption thành ON. Để biết thêm thông tin, hãy xem [Section 19.3.2, “Encrypting Binary Log Files and Relay Log Files”](https://dev.mysql.com/doc/refman/9.7/en/replication-binlog-encryption.html "19.3.2 Encrypting Binary Log Files and Relay Log Files").

Phần thảo luận sau đây mô tả một số tùy chọn và biến máy chủ ảnh hưởng đến hoạt động của việc ghi nhật ký nhị phân. Để có danh sách đầy đủ, xem [Section 19.1.6.4, “Binary Logging Options and Variables”](https://dev.mysql.com/doc/refman/9.7/en/replication-options-binary-log.html "19.1.6.4 Binary Logging Options and Variables").

Chế độ ghi nhật ký nhị phân được bật theo mặc định (biến hệ thống log_bin được đặt thành ON). Ngoại lệ là nếu bạn sử dụng mysqld để khởi tạo thư mục dữ liệu theo cách thủ công bằng cách gọi nó với tùy chọn initialize hoặc initialize-insecure, khi đó chế độ ghi nhật ký nhị phân bị tắt theo mặc định, nhưng có thể được bật bằng cách chỉ định tùy chọn log-bin.

Để tắt tính năng ghi nhật ký nhị phân, bạn có thể chỉ định tùy chọn skip-log-bin hoặc disable-log-bin khi khởi động. Nếu một trong hai tùy chọn này được chỉ định và log-bin cũng được chỉ định, tùy chọn được chỉ định sau sẽ được ưu tiên.

Các tùy chọn log-replica-updates và replica-preserve-commit-order yêu cầu ghi nhật ký nhị phân. Nếu bạn tắt ghi nhật ký nhị phân, hãy bỏ qua các tùy chọn này hoặc chỉ định log-replica-updates=OFF và skip-replica-preserve-commit-order. MySQL sẽ tắt các tùy chọn này theo mặc định khi skip-log-bin hoặc disable-log-bin được chỉ định. Nếu bạn chỉ định log-replica-updates hoặc replica-preserve-commit-order cùng với skip-log-bin hoặc disable-log-bin, một thông báo cảnh báo hoặc lỗi sẽ được đưa ra.

Tùy chọn log-bin[=base_name] được sử dụng để chỉ định tên cơ sở cho các tệp nhật ký nhị phân. Nếu bạn không cung cấp tùy chọn log-bin, MySQL sẽ sử dụng binlog làm tên cơ sở mặc định cho các tệp nhật ký nhị phân. Để tương thích với các phiên bản trước đó, nếu bạn cung cấp tùy chọn log-bin mà không có chuỗi hoặc với một chuỗi rỗng, tên cơ sở mặc định sẽ là host_name-bin, sử dụng tên của máy chủ. Bạn nên chỉ định một tên cơ sở, để nếu tên máy chủ thay đổi, bạn vẫn có thể dễ dàng sử dụng cùng tên tệp nhật ký nhị phân (xem [Section B.3.7, “Known Issues in MySQL”](https://dev.mysql.com/doc/refman/9.7/en/known-issues.html "B.3.7 Known Issues in MySQL")). Nếu bạn cung cấp phần mở rộng trong tên nhật ký (ví dụ: log-bin=base_name.extension), phần mở rộng sẽ bị xóa và bỏ qua một cách âm thầm.

mysqld thêm một phần mở rộng số vào tên cơ sở của nhật ký nhị phân để tạo tên tệp nhật ký nhị phân. Số này tăng lên mỗi khi máy chủ tạo một tệp nhật ký mới, do đó tạo ra một chuỗi tệp được sắp xếp theo thứ tự. Máy chủ tạo một tệp mới trong chuỗi mỗi khi bất kỳ sự kiện nào sau đây xảy ra.

...

Nếu bạn đang sử dụng nhật ký nhị phân và ghi nhật ký theo hàng, các thao tác chèn đồng thời sẽ được chuyển đổi thành các thao tác chèn thông thường cho các câu lệnh CREATE ... SELECT hoặc INSERT ... SELECT. Điều này được thực hiện để đảm bảo bạn có thể tạo lại một bản sao chính xác của các bảng bằng cách áp dụng nhật ký trong quá trình sao lưu. Nếu bạn đang sử dụng ghi nhật ký dựa trên câu lệnh, câu lệnh gốc sẽ được ghi vào nhật ký.

Định dạng nhật ký nhị phân có một số hạn chế đã biết có thể ảnh hưởng đến việc khôi phục từ bản sao lưu. Xem [Section 19.5.1, “Replication Features and Issues”](https://dev.mysql.com/doc/refman/9.7/en/replication-features.html "19.5.1 Replication Features and Issues").

Việc ghi nhật ký nhị phân cho các chương trình được lưu trữ được thực hiện như mô tả trong  [Section 27.9, “Stored Program Binary Logging”](https://dev.mysql.com/doc/refman/9.7/en/stored-programs-logging.html "27.9 Stored Program Binary Logging").



