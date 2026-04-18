
- [[#What is partitioning?|What is partitioning?]]
- [[#What does Iceberg do differently?|What does Iceberg do differently?]]
- [[#Partitioning in Hive (skip)|Partitioning in Hive (skip)]]
- [[#Iceberg's hidden partitioning|Iceberg's hidden partitioning]]

### What is partitioning?

Phân vùng là một cách để thực hiện truy vấn nhanh hơn bằng cách nhóm các hàng tương tự lại với nhau khi viết. 

Ví dụ, các truy vấn cho mục nhật ký từ bảng nhật ký thường bao gồm phạm vi thời gian, như truy vấn này cho nhật ký từ 10 đến 12 giờ sáng:

```sql
SELECT level, message FROM logs
WHERE event_time BETWEEN '2018-12-01 10:00:00' AND '2018-12-01 12:00:00';
```

Cấu hình bảng logs để phân vùng theo ngày event_time sẽ nhóm các sự kiện nhật ký vào các tệp có cùng ngày sự kiện. Iceberg sẽ theo dõi ngày đó và sẽ sử dụng nó để bỏ qua các tệp cho các ngày khác không có dữ liệu hữu ích.

Iceberg có thể phân vùng dấu thời gian theo mức độ chi tiết năm, tháng, ngày và giờ. Nó cũng có thể sử dụng cột phân loại, như level trong ví dụ nhật ký này, để lưu trữ các hàng lại với nhau và tăng tốc truy vấn.

### What does Iceberg do differently?

Các định dạng bảng khác như Hive hỗ trợ phân vùng, nhưng Iceberg hỗ trợ phân vùng ẩn (hidden partition). 

- Iceberg xử lý nhiệm vụ tẻ nhạt và dễ xảy ra lỗi là tạo giá trị phân vùng cho các hàng trong bảng. 
    
- Iceberg tránh tự động đọc các phân vùng không cần thiết. Người dùng không cần biết bảng được phân vùng như thế nào và không cần thêm bộ lọc bổ sung vào truy vấn của họ. 
    
- Iceberg partition layouts có thể phát triển tùy theo nhu cầu.
    
### Partitioning in Hive

Để minh họa sự khác biệt, hãy xem xét cách Hive xử lý bảng nhật ký.

Trong Hive, các phân vùng được chỉ định rõ ràng và xuất hiện dưới dạng một cột, vì vậy bảng nhật ký sẽ có một cột tên là `event_date`. Khi ghi dữ liệu, thao tác insert cần cung cấp dữ liệu cho cột `event_date`.

```sql
INSERT INTO logs PARTITION (event_date)
  SELECT level, message, event_time, format_time(event_time, 'YYYY-MM-dd')
  FROM unstructured_log_source;
```

Tương tự, các truy vấn tìm kiếm trong bảng nhật ký phải có bộ lọc `event_date` ngoài bộ lọc `event_time`.

```sql
SELECT level, count(1) as count FROM logs
WHERE event_time BETWEEN '2018-12-01 10:00:00' AND '2018-12-01 12:00:00'
  AND event_date = '2018-12-01';
```

Nếu bộ lọc `event_date` bị thiếu, Hive sẽ quét qua mọi tập tin trong bảng vì nó không biết rằng cột `event_time` có liên quan đến cột `event_date`.

### Problems with Hive partitioning

Hive cần được cung cấp các giá trị phân vùng. Trong ví dụ về nhật ký, nó không biết mối quan hệ giữa `event_time` và `event_date`.

Điều này dẫn đến một số vấn đề.

Hive không thể xác thực giá trị phân vùng; việc tạo ra giá trị chính xác là trách nhiệm của người ghi.

- Việc sử dụng định dạng sai, ví dụ `2018-12-01` thay vì `20181201`, sẽ dẫn đến kết quả không chính xác mà không báo lỗi truy vấn.
- Việc sử dụng sai cột nguồn, chẳng hạn như `processing_time` hoặc múi giờ, cũng gây ra kết quả không chính xác, chứ không phải lỗi.

Người dùng phải tự viết các truy vấn một cách chính xác.

- Việc sử dụng sai định dạng cũng dẫn đến kết quả không chính xác mà không hề báo trước.
- Người dùng không hiểu bố cục vật lý của bảng sẽ gặp phải tình trạng truy vấn chậm không cần thiết. Hive không thể tự động dịch các bộ lọc.

Các truy vấn hoạt động được liên kết với partitioning scheme của bảng, do đó cấu hình phân vùng không thể thay đổi mà không làm hỏng các truy vấn.

[https://iceberg.apache.org/docs/latest/partitioning/#problems-with-hive-partitioning](https://iceberg.apache.org/docs/latest/partitioning/#problems-with-hive-partitioning)

### Iceberg's hidden partitioning

Iceberg tạo ra các giá trị phân vùng bằng cách lấy giá trị cột và tùy ý chuyển đổi nó. Iceberg chịu trách nhiệm chuyển đổi `event_time` thành `event_date` và theo dõi mối quan hệ. 

Phân vùng bảng được cấu hình bằng các mối quan hệ này. Bảng logs sẽ được phân vùng theo ngày (`event_time`) và level. 

Vì Iceberg không yêu cầu các cột phân vùng do người dùng quản lý, nên nó có thể ẩn phân vùng. Các giá trị phân vùng luôn được tạo chính xác và luôn được sử dụng để tăng tốc truy vấn, nếu có thể. Các producer và consumer thậm chí sẽ không nhìn thấy event_date. 

Quan trọng nhất, các truy vấn không còn phụ thuộc vào bố cục vật lý của bảng. Với sự tách biệt giữa vật lý và logic, các bảng Iceberg có thể phát triển các schema phân vùng theo thời gian khi khối lượng dữ liệu thay đổi. Các bảng được cấu hình sai có thể được sửa mà không cần di chuyển tốn kém. 

Để biết chi tiết về tất cả các chuyển đổi phân vùng ẩn được hỗ trợ, hãy xem phần [Partition Transforms](https://iceberg.apache.org/spec/#partition-transforms).

Để biết thông tin chi tiết về việc cập nhật thông số phân vùng của bảng, hãy xem phần [Partition evolution](https://iceberg.apache.org/docs/latest/evolution/#partition-evolution).