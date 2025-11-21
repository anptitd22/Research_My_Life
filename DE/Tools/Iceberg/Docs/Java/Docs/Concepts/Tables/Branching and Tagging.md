
- [[#Overview|Overview]]
- [[#Use Cases|Use Cases]]
- [[#Historical Tags|Historical Tags]]
- [[#Audit Branch|Audit Branch]]
- [[#Usage|Usage]]
- [[#Schema selection with branches and tags|Schema selection with branches and tags]]

### Overview

Metadata bảng Iceberg duy trì snapshot log, thể hiện những thay đổi được áp dụng cho một bảng. Snapshot rất quan trọng trong Iceberg vì chúng là cơ sở cho việc cô lập trình đọc và truy vấn time travel. Để kiểm soát kích thước metadata và chi phí lưu trữ, Iceberg cung cấp các quy trình quản lý vòng đời snapshot như expire_snapshots để xóa các snapshot không sử dụng và các tệp dữ liệu không còn cần thiết dựa trên các thuộc tính lưu giữ snapshot của bảng.

Để quản lý vòng đời snapshot tinh vi hơn, Iceberg hỗ trợ các nhánh và thẻ, được đặt tên là tham chiếu đến các snapshot với vòng đời độc lập riêng. Vòng đời này được kiểm soát bởi các chính sách lưu giữ ở cấp nhánh và thẻ. Các nhánh là các dòng snapshot độc lập và trỏ đến đầu dòng. Các nhánh và thẻ có thuộc tính tuổi tham chiếu tối đa, kiểm soát thời điểm tham chiếu đến chính snapshot nên hết hạn. Các nhánh có các thuộc tính lưu giữ, xác định số lượng snapshot tối thiểu cần lưu giữ trên một nhánh cũng như tuổi tối đa của từng snapshot riêng lẻ được lưu giữ trên nhánh. Các thuộc tính này được sử dụng khi chạy thủ tục expireSnapshots. Để biết chi tiết về thuật toán cho expireSnapshots, hãy tham khảo đặc tả kỹ thuật.

### Use Cases

Phân nhánh và gắn thẻ có thể được sử dụng để xử lý các yêu cầu của GDPR và lưu giữ các snapshot lịch sử quan trọng để kiểm tra. Phân nhánh cũng có thể được sử dụng như một phần của quy trình kỹ thuật dữ liệu, cho phép các nhánh thử nghiệm kiểm tra và xác thực các công việc mới. Xem bên dưới để biết một số ví dụ về cách phân nhánh và gắn thẻ có thể tạo điều kiện thuận lợi cho các trường hợp sử dụng này.

### Historical Tags

Thẻ có thể được sử dụng để lưu giữ các lịch sử snapshot quan trọng cho mục đích kiểm tra.

![alt text](images/Pasted_image_20251121130256.png)

Sơ đồ trên minh họa việc lưu giữ lịch sử snapshot quan trọng bằng chính sách lưu giữ sau, được xác định thông qua Spark SQL.

1. Lưu giữ 1 snapshot mỗi tuần trong 1 tháng. Điều này có thể thực hiện bằng cách gắn thẻ snapshot hàng tuần và đặt thời gian lưu giữ tag là 1 tháng. Các snapshot sẽ được lưu giữ, và tham chiếu nhánh sẽ được lưu giữ trong 1 tuần.
    
```sql
-- Create a tag for the first end of week snapshot. Retain the snapshot for a week
ALTER TABLE prod.db.table CREATE TAG `EOW-01` AS OF VERSION 7 RETAIN 7 DAYS;
```

2. Lưu giữ 1 snapshot mỗi tháng trong 6 tháng. Có thể thực hiện điều này bằng cách gắn thẻ snapshot hàng tháng và đặt thời gian lưu giữ thẻ là 6 tháng.
    
```sql
-- Create a tag for the first end of month snapshot. Retain the snapshot for 6 months
ALTER TABLE prod.db.table CREATE TAG `EOM-01` AS OF VERSION 30 RETAIN 180 DAYS;
```


3. Lưu giữ vĩnh viễn 1 bản sao lưu mỗi năm. Có thể thực hiện điều này bằng cách gắn thẻ cho bản sao lưu hàng năm. Mặc định, các nhánh và thẻ sẽ được lưu giữ vĩnh viễn.
    
```sql
-- Create a tag for the end of the year and retain it forever.
ALTER TABLE prod.db.table CREATE TAG `EOY-2023` AS OF VERSION 365;
```

4. Tạo một "test-branch" tạm thời được lưu giữ trong 7 ngày và 2 snapshot mới nhất trên nhánh sẽ được lưu giữ.
    
```sql
-- Create a branch "test-branch" which will be retained for 7 days along with the  latest 2 snapshots
ALTER TABLE prod.db.table CREATE BRANCH `test-branch` RETAIN 7 DAYS WITH SNAPSHOT RETENTION 2 SNAPSHOTS;
```

### Audit Branch

![alt text](images/Pasted_image_20251121130433.png)

Sơ đồ trên cho thấy ví dụ về việc sử dụng nhánh kiểm tra để xác thực quy trình ghi.

1. Đầu tiên hãy đảm bảo `write.wap.enabled` được thiết lập.
    
```sql
ALTER TABLE db.table SET TBLPROPERTIES (
    'write.wap.enabled'='true'
);
```

2. Create` audit-branch`bắt đầu từ snapshot 3, nhánh này sẽ được ghi vào và lưu giữ trong 1 tuần. 
    
```sql
ALTER TABLE db.table CREATE BRANCH `audit-branch` AS OF VERSION 3 RETAIN 7 DAYS;
```

3. Việc ghi được thực hiện trên một nhánh kiểm tra riêng biệt, độc lập với lịch sử bảng chính.
    
```sql
-- WAP Branch write
SET spark.wap.branch = audit-branch
INSERT INTO prod.db.table VALUES (3, 'c');
```

4. Quy trình xác thực có thể xác thực (ví dụ: chất lượng dữ liệu) trạng thái của nhánh `audit-branch`.
    

5. Sau khi xác thực, nhánh chính có thể `fastForward` đến đầu nhánh `audit-branch` để cập nhật trạng thái bảng chính.
    
```sql
CALL catalog_name.system.fast_forward('prod.db.table', 'main', 'audit-branch');
```

6. Tham chiếu nhánh sẽ bị xóa khi `expireSnapshots` được chạy sau 1 tuần.
    
### Usage

Thư viện Iceberg Java và tích hợp công cụ Spark và Flink hỗ trợ việc tạo, truy vấn và ghi vào các nhánh và thẻ.

- [Iceberg Java Library](https://iceberg.apache.org/docs/latest/java-api-quickstart/#branching-and-tagging)
    
- [Spark DDLs](https://iceberg.apache.org/docs/latest/spark-ddl/#branching-and-tagging-ddl)
    
- [Spark Reads](https://iceberg.apache.org/docs/latest/spark-queries/#time-travel)
    
- [Spark Branch Writes](https://iceberg.apache.org/docs/latest/spark-writes/#writing-to-branches)
    
- [Flink Reads](https://iceberg.apache.org/docs/latest/flink-queries/#reading-branches-and-tags-with-SQL)
    
- [Flink Branch Writes](https://iceberg.apache.org/docs/latest/flink-writes/#branch-writes)
    
- [Trino](https://trino-io.translate.goog/docs/current/connector/iceberg.html?_x_tr_sl=en&_x_tr_tl=vi&_x_tr_hl=vi&_x_tr_pto=tc%C6%B0) (mặc định 20 version gần nhất)
    
### Schema selection with branches and tags

Điều quan trọng là phải hiểu rằng schema được theo dõi cho một bảng có hiệu lực trên tất cả các nhánh. Khi làm việc với các nhánh, schema của bảng được sử dụng vì đó là schema được xác thực khi ghi dữ liệu vào một nhánh. Mặt khác, việc truy vấn thẻ sẽ sử dụng schema của snapshot, tức là schema ID mà snapshot trỏ đến khi snapshot được tạo.

Các ví dụ dưới đây cho thấy schema nào đang được sử dụng khi làm việc với các nhánh.

Tạo một bảng và chèn một số dữ liệu:

```sql
CREATE TABLE db.table (id bigint, data string, col float);
INSERT INTO db.table VALUES (1, 'a', 1.0), (2, 'b', 2.0), (3, 'c', 3.0);
SELECT * FROM db.table;
1   a   1.0

2   b   2.0

3   c   3.0
```
  
Tạo nhánh `test_branch` trỏ đến snapshot hiện tại và đọc dữ liệu từ nhánh:

```sql
ALTER TABLE db.table CREATE BRANCH test_branch;
SELECT * FROM db.table.branch_test_branch;
1   a   1.0
2   b   2.0
3   c   3.0
```

Sửa đổi schema của bảng bằng cách xóa cột `col` và thêm một cột mới có tên là `new_col`:

```sql
ALTER TABLE db.table DROP COLUMN col;

ALTER TABLE db.table ADD COLUMN new_col date;

INSERT INTO db.table VALUES (4, 'd', date('2024-04-04')), (5, 'e', date('2024-05-05'));

SELECT * FROM db.table;
1   a   NULL
2   b   NULL
3   c   NULL
4   d   2024-04-04
5   e   2024-05-05
```
  
Truy vấn đầu nhánh bằng một trong các câu lệnh dưới đây sẽ trả về dữ liệu bằng cách sử dụng schema của bảng:

```sql
SELECT * FROM db.table.branch_test_branch;
1   a   NULL
2   b   NULL
3   c   NULL

SELECT * FROM db.table VERSION AS OF 'test_branch';
1   a   NULL
2   b   NULL
3   c   NULL
```

Thực hiện truy vấn travel time bằng cách sử dụng ID snapshot sẽ sử dụng schema của snapshot:

```sql
SELECT * FROM db.table.refs;
test_branch BRANCH  8109744798576441359 NULL    NULL    NULL
main        BRANCH  6910357365743665710 NULL    NULL    NULL

SELECT * FROM db.table VERSION AS OF 8109744798576441359;
1   a   1.0
2   b   2.0
3   c   3.0
```

Khi ghi vào nhánh, schema của bảng được sử dụng để xác thực:

```sql
INSERT INTO db.table.branch_test_branch VALUES (6, 'e', date('2024-06-06')), (7, 'g', date('2024-07-07'));

SELECT * FROM db.table.branch_test_branch;
6   e   2024-06-06
7   g   2024-07-07
1   a   NULL
2   b   NULL
3   c   NULL
```