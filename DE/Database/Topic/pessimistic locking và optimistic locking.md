# Pessimistic

Hoạt động dựa trên niềm tin rằng xung đột giữa các transaction có thể xảy ra, và do đó nó sử dụng khóa để ngăn chặn xung đột bản ghi trong cơ sở dữ liệu. Một số hậu quả là các loại giao dịch khác chỉ có thể truy cập bản ghi ở chế độ read-only hoặc phải đợi cho đến khi bản ghi này được mở khóa.

Quy trình các giai đoạn giao dịch:

- Validate -> Read -> Compute -> Write

Ưu điểm:

- Ngăn ngừa xung đột: Do đó, các bản ghi không thể bị thay đổi bởi các giao dịch khác do cơ chế khóa, vì vậy sẽ có ít xung đột khi giao dịch chưa hoàn tất.
- Độ tin cậy cao: Phương pháp này hữu ích trong các tình huống có mức độ tranh chấp giao dịch cao nhờ các khóa đóng vai trò quan trọng trong việc duy trì tính nhất quán (consistency) của dữ liệu.

Nhược điểm:

- Rủi ro tắc nghẽn: Vì vậy, khi làm việc với các khóa, cần phải hiểu rằng chúng có thể dẫn đến tắc nghẽn, cụ thể là deadlock và do đó làm cho việc lập trình trở nên phức tạp hơn.
- Giảm khả năng xử lý đồng thời: Khóa làm giảm khả năng xử lý đồng thời của hệ thống vì các giao dịch có thể phải chờ để có được quyền sử dụng các khóa cần thiết.
- Chi phí lưu trữ cao hơn: Hầu hết các trường hợp, nó đòi hỏi nhiều không gian lưu trữ hơn vì cần phải duy trì các khóa hoặc các cơ chế [concurrency control](https://www.geeksforgeeks.org/dbms/concurrency-control-in-dbms/) khác.

Ứng dụng: update for vs share for

# Optimistic

Dựa trên giả định rằng xung đột giao dịch rất hiếm khi xảy ra. Thay vì sử dụng khóa trong quá trình thực thi thông thường, nó cho phép nhiều giao dịch diễn ra đồng thời và chỉ kiểm tra xung đột tại thời điểm cam kết. Nếu phát hiện xung đột trong quá trình xác thực, giao dịch sẽ bị hủy bỏ.

Trích: https://www.geeksforgeeks.org/dbms/difference-between-pessimistic-approach-and-optimistic-approach-in-dbms/
