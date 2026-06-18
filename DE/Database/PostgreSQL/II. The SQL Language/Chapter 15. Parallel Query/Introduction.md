PostgreSQL có thể thiết kế các kế hoạch truy vấn tận dụng nhiều CPU để trả lời truy vấn nhanh hơn. Tính năng này được gọi là truy vấn song song. Nhiều truy vấn không thể hưởng lợi từ truy vấn song song, hoặc do những hạn chế của việc triển khai hiện tại hoặc vì không có kế hoạch truy vấn nào nhanh hơn kế hoạch truy vấn tuần tự. Tuy nhiên, đối với các truy vấn có thể hưởng lợi, tốc độ tăng lên từ truy vấn song song thường rất đáng kể. Nhiều truy vấn có thể chạy nhanh hơn gấp đôi khi sử dụng truy vấn song song, và một số truy vấn có thể chạy nhanh hơn gấp bốn lần hoặc thậm chí hơn nữa. Các truy vấn xử lý lượng lớn dữ liệu nhưng chỉ trả về một vài hàng cho người dùng thường sẽ được hưởng lợi nhiều nhất. Chương này giải thích một số chi tiết về cách thức hoạt động của truy vấn song song và trong những trường hợp nào nó có thể được sử dụng để người dùng muốn sử dụng nó có thể hiểu được những gì cần mong đợi.

Nguồn: https://www.postgresql.org/docs/18/parallel-query.html
- [[15.1. How Parallel Query Works]]
- [[15.2. When Can Parallel Query Be Used?]]
- [[15.3. Parallel Plans]]
- [[15.4. Parallel Safety]]
