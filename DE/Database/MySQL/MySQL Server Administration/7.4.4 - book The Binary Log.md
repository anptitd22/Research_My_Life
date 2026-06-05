**Khái niệm cơ bản về Binary Log** Binary log (nhật ký nhị phân) là một thành phần chứa các "sự kiện" (events) mô tả chi tiết các thay đổi đối với cơ sở dữ liệu, chẳng hạn như thao tác tạo bảng hoặc cập nhật dữ liệu. Nó cũng lưu lại thông tin về thời gian thực thi của các câu lệnh cập nhật. **Binary log không ghi lại các câu lệnh không sửa đổi dữ liệu** như `SELECT` hay `SHOW` (nếu cần ghi các lệnh này, bạn phải dùng general query log). Mặc dù việc bật binary log có thể làm hiệu suất máy chủ chậm đi một chút, nhưng các lợi ích nó mang lại vượt trội hơn nhiều so với phần suy giảm hiệu suất nhỏ này.

**Hai mục đích (vấn đề) quan trọng nhất của Binary Log:**

1. **Sao chép dữ liệu (Replication):** Trên một máy chủ nguồn (source server), binary log đóng vai trò là bản ghi các thay đổi dữ liệu để gửi tới các máy chủ bản sao (replicas). Các replicas sẽ nhận thông tin này và tái tạo lại các giao dịch tương tự để cập nhật dữ liệu đồng nhất với máy chủ nguồn.
2. **Khôi phục dữ liệu (Data Recovery):** Khi thực hiện khôi phục hệ thống từ một bản sao lưu (backup), các sự kiện trong binary log diễn ra sau thời điểm sao lưu sẽ được thực thi lại, giúp cơ sở dữ liệu lấy lại trạng thái mới nhất.

**Các định dạng và cơ chế lưu trữ:**

- **Định dạng ghi (Logging Formats):** MySQL hỗ trợ ba loại định dạng để ghi log: dựa trên hàng (row-based), dựa trên câu lệnh (statement-based) và định dạng hỗn hợp (mixed-base logging).
- **Quản lý Tệp:** Binary log thực chất bao gồm một tập hợp các tệp được đánh số tuần tự và một tệp chỉ mục (index file) có đuôi `.index` dùng để theo dõi các tệp log đã sử dụng. Hệ thống tự động tạo tệp log mới mỗi khi máy chủ khởi động/khởi động lại, khi xả log (flush), hoặc khi kích thước tệp đạt đến giới hạn `max_binlog_size`. Tuy nhiên, một tệp log có thể lớn hơn `max_binlog_size` nếu nó chứa một giao dịch lớn, vì một giao dịch luôn được ghi nguyên vẹn trong một tệp chứ không bị chia cắt.

**Xử lý Giao dịch (Transactions):**

- Quá trình ghi binary log diễn ra ngay sau khi một câu lệnh hoặc giao dịch hoàn tất, nhưng **phải trước khi các khóa (locks) được giải phóng hoặc thao tác commit được thực hiện** để đảm bảo dữ liệu được ghi đúng thứ tự.
- Trong một giao dịch chưa commit, các thay đổi đối với các bảng hỗ trợ giao dịch (như InnoDB) được lưu tạm trong một bộ đệm có kích thước `binlog_cache_size`. Nếu dữ liệu vượt mức này, nó sẽ được lưu vào một tệp tạm thời trên đĩa. Trong trường hợp tổng dung lượng giao dịch vượt mức giới hạn tuyệt đối `max_binlog_cache_size`, giao dịch đó sẽ bị lỗi và hoàn tác (roll back).
- Với các bảng không hỗ trợ giao dịch (nontransactional tables), những thay đổi không thể bị hoàn tác. Do đó, nếu một giao dịch chứa bảng loại này bị roll back, toàn bộ giao dịch vẫn sẽ được ghi vào binary log kèm theo lệnh `ROLLBACK` ở cuối cùng để đảm bảo các máy chủ replica cũng thực hiện sự hoàn tác tương đương.

**Đồng bộ hóa và Xử lý sự cố (Crash Handling):**

- Binary log có khả năng chống chịu trước các sự cố dừng đột ngột của máy chủ; chỉ các sự kiện hoặc giao dịch hoàn chỉnh mới được ghi lại và đọc lại.
- **Biến** **sync_binlog**: Mặc định, `sync_binlog = 1` giúp đồng bộ hóa log nhị phân xuống ổ đĩa vật lý sau mỗi lần ghi. Đây là giá trị an toàn nhất, dù tốc độ xử lý là chậm nhất, giúp giảm thiểu rủi ro mất mát giao dịch khi sự cố hệ thống xảy ra.
- **Đồng bộ với dữ liệu InnoDB**: Trong MySQL 9.7, tính năng commit hai giai đoạn (two-phase commit) trong các giao dịch XA của InnoDB luôn được bật. Khi khởi động lại sau một sự cố rớt mạng/sập máy chủ, MySQL sẽ quét tệp binary log mới nhất để thu thập các ID giao dịch hợp lệ, ra lệnh cho InnoDB hoàn tất các giao dịch đã ghi nhận thành công, đồng thời cắt đi đoạn log lỗi. Tuy nhiên, nếu ổ đĩa không thực sự đồng bộ lúc được yêu cầu, log nhị phân có thể bị ngắn hơn kỳ vọng. Khi đó, máy chủ sẽ báo lỗi, và bản sao (replica) sẽ phải được khởi động lại từ một bản chụp dữ liệu (snapshot) hoàn toàn mới.
- **Lỗi thao tác ghi:** Nếu máy chủ không thể ghi vào binary log (ví dụ: đầy ổ đĩa), biến `binlog_error_action` sẽ điều khiển phản ứng của hệ thống. Chế độ mặc định là `ABORT_SERVER`, sẽ buộc máy chủ dừng hoạt động để bạn kiểm tra nguyên nhân. Tùy chọn `IGNORE_ERROR` (chủ yếu dùng để tương thích ngược) sẽ bỏ qua việc ghi log nhưng vẫn cho phép cập nhật cơ sở dữ liệu.

**Bảo mật và Vận hành:**

- **Bảo mật:** Mật khẩu trong các câu lệnh được ghi vào log sẽ được hệ thống tự động ẩn đi và không hiển thị dưới dạng văn bản thuần túy (plain text). Hơn nữa, để chống lại các cuộc tấn công bên ngoài, bạn có thể mã hóa các tệp binary log và relay log bằng cách bật biến `binlog_encryption` lên `ON`.
- **Bảo trì, Xóa file:** Bạn không nên xóa bỏ thủ công các tệp log cũ trên máy chủ nguồn khi chưa chắc chắn rằng các replicas đã sử dụng xong. Hãy dùng lệnh `PURGE BINARY LOGS` để xóa, bởi lệnh này giúp loại bỏ log một cách an toàn và tự động cập nhật lại tệp chỉ mục. Để kiểm tra nội dung log, bạn có thể dùng công cụ `mysqlbinlog`.