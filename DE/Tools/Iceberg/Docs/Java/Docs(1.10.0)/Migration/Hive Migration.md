- [[#Enabling Migration from Hive to Iceberg|Enabling Migration from Hive to Iceberg]]
- [[#Snapshot Hive Table to Iceberg|Snapshot Hive Table to Iceberg]]
- [[#Migrate Hive Table To Iceberg|Migrate Hive Table To Iceberg]]
- [[#Add Files From Hive Table to Iceberg|Add Files From Hive Table to Iceberg]]

Apache Hive hỗ trợ các định dạng tệp ORC, Parquet và Avro, có thể được chuyển đổi sang Iceberg. Khi chuyển dữ liệu sang bảng Iceberg, vốn cung cấp tính năng quản lý phiên bản và cập nhật giao dịch, chỉ cần chuyển các tệp dữ liệu mới nhất.

Iceberg hỗ trợ cả ba thao tác di chuyển: Snapshot Table, Migrate Table, and Add Files để di chuyển dữ liệu từ bảng Hive sang bảng Iceberg. Vì bảng Hive không duy trì snapshot, quá trình di chuyển về cơ bản bao gồm việc tạo một bảng Iceberg mới với schema hiện có và cam kết tất cả các tệp dữ liệu trên tất cả các phân vùng vào bảng Iceberg mới. Sau khi di chuyển ban đầu, bất kỳ tệp dữ liệu mới nào sẽ được thêm vào bảng Iceberg mới bằng thao tác Add Files.

## Enabling Migration from Hive to Iceberg

Các thao tác di chuyển bảng Hive được hỗ trợ bởi mô-đun Spark Integration thông qua các Spark Procedures. Các procedures này được đóng gói trong Spark runtime jar, có sẵn trong phần [Iceberg Release Downloads](https://iceberg.apache.org/releases/#downloads).

## Snapshot Hive Table to Iceberg

Để tạo snapshot của bảng Hive, người dùng có thể chạy câu lệnh Spark SQL sau.

```sql
CALL catalog_name.system.snapshot('db.source', 'db.dest')
```

Xem [Spark Procedure: snapshot](https://iceberg.apache.org/docs/1.10.0/spark-procedures/#snapshot) để biết thêm chi tiết.

## Migrate Hive Table To Iceberg

Để chuyển đổi bảng Hive sang Iceberg, người dùng có thể chạy câu lệnh Spark SQL sau:

```sql
CALL catalog_name.system.migrate('db.sample')
```

Xem [Spark Procedure: migrate](https://iceberg.apache.org/docs/1.10.0/spark-procedures/#migrate) để biết thêm chi tiết

## Add Files From Hive Table to Iceberg

Để thêm các tập dữ liệu từ bảng Hive vào bảng Iceberg đã cho, người dùng có thể chạy câu lệnh Spark SQL sau.

```sql
CALL spark_catalog.system.add_files(
table => 'db.tbl',
source_table => 'db.src_tbl'
)
```

Xem [Spark Procedure: add_files](https://iceberg.apache.org/docs/1.10.0/spark-procedures/#add_files) để biết thêm chi tiết.

Nguồn: https://iceberg.apache.org/docs/1.10.0/hive-migration/#snapshot-hive-table-to-iceberg

---

Ngoài ra ta có thể convert bảng hive sang iceberg thông qua trino

Xem https://trino.io/docs/current/connector/iceberg.html#procedures để biết chi tiết