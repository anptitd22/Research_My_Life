
- [[#General configuration|General configuration]]
	- [[#General configuration#Fault-tolerant execution support|Fault-tolerant execution support]]
- [[#File system access configuration|File system access configuration]]
- [[#Type mapping|Type mapping]]
	- [[#Type mapping#Iceberg to Trino type mapping|Iceberg to Trino type mapping]]
	- [[#Type mapping#Trino to Iceberg type mapping|Trino to Iceberg type mapping]]
- [[#Security|Security]]
	- [[#Security#Kerberos authentication|Kerberos authentication]]
	- [[#Security#Authorization|Authorization]]
- [[#SQL support|SQL support]]
	- [[#SQL support#Basic usage examples|Basic usage examples]]
	- [[#SQL support#Procedures|Procedures]]
		- [[#Procedures#Register table|Register table]]
		- [[#Procedures#Unregister table|Unregister table]]
		- [[#Procedures#Migrate table|Migrate table]]
		- [[#Procedures#Add files|Add files]]
	- [[#SQL support#Functions|Functions]]
		- [[#Functions#bucket|bucket]]
	- [[#SQL support#Data management|Data management]]
		- [[#Data management#Deletion by partition|Deletion by partition]]
		- [[#Data management#Row level deletion|Row level deletion]]
	- [[#SQL support#Schema and table management|Schema and table management]]
		- [[#Schema and table management#Schema evolution|Schema evolution]]
		- [[#Schema and table management#ALTER TABLE EXECUTE|ALTER TABLE EXECUTE]]
			- [[#ALTER TABLE EXECUTE#optimize|optimize]]
			- [[#ALTER TABLE EXECUTE#optimize_manifests|optimize_manifests]]
			- [[#ALTER TABLE EXECUTE#expire_snapshots|expire_snapshots]]
			- [[#ALTER TABLE EXECUTE#remove_orphan_files|remove_orphan_files]]
			- [[#ALTER TABLE EXECUTE#drop_extended_stats|drop_extended_stats]]
		- [[#Schema and table management#ALTER TABLE SET PROPERTIES|ALTER TABLE SET PROPERTIES]]
			- [[#ALTER TABLE SET PROPERTIES#Table properties|Table properties]]
		- [[#Schema and table management#Metadata tables|Metadata tables]]


Apache Iceberg là một định dạng bảng mở (open format table) dành cho các tập dữ liệu phân tích khổng lồ. Trình kết nối Iceberg cho phép truy vấn dữ liệu được lưu trữ trong các tệp được viết theo định dạng Iceberg, như được định nghĩa trong [Iceberg Table Spec](https://iceberg.apache.org/spec/). Trình kết nối này hỗ trợ đặc tả bảng Apache Iceberg phiên bản 1 và 2.

Trạng thái bảng được duy trì trong các tệp metadata. Mọi thay đổi đối với trạng thái bảng sẽ tạo một tệp metadata mới và thay thế metadata cũ bằng một hoán đổi nguyên tử. Tệp metadata bảng theo dõi schema bảng, cấu hình phân vùng, thuộc tính tùy chỉnh và snapshot nội dung bảng.

Các tệp dữ liệu Iceberg được lưu trữ theo định dạng Parquet, ORC hoặc Avro, tùy thuộc vào thuộc tính định dạng trong định nghĩa bảng.

Iceberg được thiết kế để cải thiện những hạn chế về khả năng mở rộng đã biết của Hive, vốn lưu trữ metadata bảng trong một metastore được hỗ trợ bởi một cơ sở dữ liệu quan hệ như MySQL. Nó theo dõi vị trí phân vùng trong metastore, nhưng không theo dõi các tệp dữ liệu riêng lẻ. Các truy vấn Trino sử dụng trình kết nối Hive trước tiên phải gọi metastore để lấy vị trí phân vùng, sau đó gọi hệ thống tệp cơ sở để liệt kê tất cả các tệp dữ liệu bên trong mỗi phân vùng, rồi đọc metadata từ mỗi tệp dữ liệu.

Vì Iceberg lưu trữ đường dẫn đến các tệp dữ liệu trong các tệp metadata nên nó chỉ tham khảo hệ thống tệp cơ bản để tìm các tệp cần phải đọc.

## General configuration

Để cấu hình trình kết nối Iceberg, hãy tạo tệp thuộc tính catalogs etc/catalog/example.properties tham chiếu đến trình kết nối Iceberg.

[Hive metastore catalog](https://trino.io/docs/current/object-storage/metastores.html#hive-thrift-metastore) là triển khai mặc định

Bạn phải chọn và cấu hình một trong các [supported file systems](https://trino.io/docs/current/connector/iceberg.html#iceberg-file-system-configuration).

```config
connector.name=iceberg
hive.metastore.uri=thrift://example.net:9083
fs.x.enabled=true
```

Thay thế thuộc tính cấu hình `fs.x.enabled` bằng hệ thống tệp mong muốn.

Các loại metadata catalog khác được liệt kê trong phần yêu cầu của chủ đề này cũng khả dụng. Mỗi loại metadata có các thuộc tính cấu hình cụ thể cùng với các thuộc tính cấu hình metadata chung.

Các thuộc tính cấu hình sau đây không phụ thuộc vào việc sử dụng triển khai catalog nào:

<p align="center">Thuộc tính cấu hình chung của Iceberg </p>

|   |   |   |
|---|---|---|
|Property name|Description|Default|
|iceberg.catalog.type|Define the [metastore type](https://trino.io/docs/current/object-storage/metastores.html#general-metastore-properties) to use. Possible values are:<br><br>- hive_metastore<br>    <br>- glue<br>    <br>- jdbc<br>    <br>- rest<br>    <br>- nessie<br>    <br>- snowflake|hive_metastore|
|iceberg.file-format|Define the data storage file format for Iceberg tables. Possible values are:<br><br>- PARQUET<br>    <br>- ORC<br>    <br>- AVRO|PARQUET|
|iceberg.compression-codec|The compression codec used when writing files. Possible values are:<br><br>- NONE<br>    <br>- SNAPPY<br>    <br>- LZ4<br>    <br>- ZSTD<br>    <br>- GZIP|ZSTD|
|iceberg.use-file-size-from-metadata|Read file sizes from metadata instead of file system. This property must only be used as a workaround for [this issue](https://github.com/apache/iceberg/issues/1980). The problem was fixed in Iceberg version 0.11.0.|true|
|iceberg.max-partitions-per-writer|Maximum number of partitions handled per writer. The equivalent catalog session property is max_partitions_per_writer.|100|
|iceberg.target-max-file-size|Target maximum size of written files; the actual size may be larger.|1GB|
|iceberg.unique-table-location|Use randomized, unique table locations.|true|
|iceberg.dynamic-filtering.wait-timeout|Maximum duration to wait for completion of dynamic filters during split generation.|1s|
|iceberg.delete-schema-locations-fallback|Whether schema locations are deleted when Trino can’t determine whether they contain external files.|false|
|iceberg.minimum-assigned-split-weight|A decimal value in the range (0, 1] used as a minimum for weights assigned to each split. A low value may improve performance on tables with small files. A higher value may improve performance for queries with highly skewed aggregations or joins.|0.05|
|iceberg.table-statistics-enabled|Enable [Table statistics](https://trino.io/docs/current/optimizer/statistics.html). The equivalent [catalog session property](https://trino.io/docs/current/sql/set-session.html) is statistics_enabled for session specific use. Set to false to prevent statistics usage by the [Cost-based optimizations](https://trino.io/docs/current/optimizer/cost-based-optimizations.html) to make better decisions about the query plan and therefore improve query processing performance. Setting to false is not recommended and does not disable statistics gathering.|true|
|iceberg.extended-statistics.enabled|Enable statistics collection with [ANALYZE](https://trino.io/docs/current/sql/analyze.html) and use of extended statistics. The equivalent catalog session property is extended_statistics_enabled.|true|
|iceberg.extended-statistics.collect-on-write|Enable collection of extended statistics for write operations. The equivalent catalog session property is collect_extended_statistics_on_write.|true|
|iceberg.projection-pushdown-enabled|Enable [projection pushdown](https://trino.io/docs/current/optimizer/pushdown.html)|true|
|iceberg.hive-catalog-name|Catalog to redirect to when a Hive table is referenced.||
|iceberg.register-table-procedure.enabled|Enable to allow user to call [register_table procedure](https://trino.io/docs/current/connector/iceberg.html#iceberg-register-table).|false|
|iceberg.add-files-procedure.enabled|Enable to allow user to call [add_files procedure](https://trino.io/docs/current/connector/iceberg.html#iceberg-add-files).|false|
|iceberg.query-partition-filter-required|Set to true to force a query to use a partition filter for schemas specified with iceberg.query-partition-filter-required-schemas. Equivalent catalog session property is query_partition_filter_required.|false|
|iceberg.query-partition-filter-required-schemas|Specify the list of schemas for which Trino can enforce that queries use a filter on partition keys for source tables. Equivalent session property is query_partition_filter_required_schemas. The list is used if the iceberg.query-partition-filter-required configuration property or the query_partition_filter_required catalog session property is set to true.|[]|
|iceberg.incremental-refresh-enabled|Set to false to force the materialized view refresh operation to always perform a full refresh. You can use the incremental_refresh_enabled catalog session property for temporary, catalog specific use. In the majority of cases, using incremental refresh, as compared to a full refresh, is beneficial since a much smaller subset of the source tables needs to be scanned. While incremental refresh may scan less data, it may result in the creation of more data files, since it uses the append operation to insert the new records.|true|
|iceberg.metadata-cache.enabled|Set to false to disable in-memory caching of metadata files on the coordinator. This cache is not used when fs.cache.enabled is set to true.|true|
|iceberg.object-store-layout.enabled|Set to true to enable Iceberg’s [object store file layout](https://iceberg.apache.org/docs/latest/aws/#object-store-file-layout). Enabling the object store file layout appends a deterministic hash directly after the data write path.|false|
|iceberg.expire-snapshots.min-retention|Minimal retention period for the [expire_snapshot command](https://trino.io/docs/current/connector/iceberg.html#iceberg-expire-snapshots). Equivalent session property is expire_snapshots_min_retention.|7d|
|iceberg.remove-orphan-files.min-retention|Minimal retention period for the [remove_orphan_files command](https://trino.io/docs/current/connector/iceberg.html#iceberg-remove-orphan-files). Equivalent session property is remove_orphan_files_min_retention.|7d|
|iceberg.idle-writer-min-file-size|Minimum data written by a single partition writer before it can be considered as idle and can be closed by the engine. Equivalent session property is idle_writer_min_file_size.|16MB|
|iceberg.sorted-writing-enabled|Enable [sorted writing](https://trino.io/docs/current/connector/iceberg.html#iceberg-sorted-files) to tables with a specified sort order. Equivalent session property is sorted_writing_enabled.|true|
|iceberg.sorted-writing.local-staging-path|A local directory that Trino can use for staging writes to sorted tables. The ${USER} placeholder can be used to use a different location for each user. When this property is not configured, the target storage will be used for staging while writing to sorted tables which can be inefficient when writing to object stores like S3. When fs.hadoop.enabled is not enabled, using this feature requires setup of [local file system](https://trino.io/docs/current/object-storage/file-system-local.html)||
|iceberg.allowed-extra-properties|List of extra properties that are allowed to be set on Iceberg tables. Use * to allow all properties.|[]|
|iceberg.split-manager-threads|Number of threads to use for generating splits.|Double the number of processors on the coordinator node.|
|iceberg.metadata.parallelism|Number of threads used for retrieving metadata. Currently, only table loading is parallelized.|8|
|iceberg.file-delete-threads|Number of threads to use for deleting files when running expire_snapshots procedure.|Double the number of processors on the coordinator node.|
|iceberg.bucket-execution|Enable bucket-aware execution. This allows the engine to use physical bucketing information to optimize queries by reducing data exchanges.|true|

### Fault-tolerant execution support

The connector hỗ trợ [Fault-tolerant execution](https://trino.io/docs/current/admin/fault-tolerant-execution.html) trong thực thi xử lý truy vấn. Cả thao tác đọc và ghi đều được hỗ trợ với any retry policy.

## File system access configuration

Trình kết nối hỗ trợ truy cập các hệ thống tệp sau:

- [Azure Storage file system support](https://trino.io/docs/current/object-storage/file-system-azure.html)
    
- [Google Cloud Storage file system support](https://trino.io/docs/current/object-storage/file-system-gcs.html)
    
- [S3 file system support](https://trino.io/docs/current/object-storage/file-system-s3.html)
    
- [HDFS file system support](https://trino.io/docs/current/object-storage/file-system-hdfs.html)
    
Bạn phải bật và cấu hình quyền truy cập hệ thống tệp cụ thể. Không khuyến khích hỗ trợ phiên bản cũ ([Legacy support](https://trino.io/docs/current/object-storage.html#file-system-legacy)) và sẽ bị xóa.

## Type mapping

Bộ kết nối đọc và ghi dữ liệu vào các định dạng tệp dữ liệu được hỗ trợ Avro, ORC và Parquet, theo thông số kỹ thuật của Iceberg.

Vì Trino và Iceberg đều hỗ trợ các kiểu dữ liệu mà cái kia không hỗ trợ, nên trình kết nối này sẽ sửa đổi một số kiểu dữ liệu khi đọc hoặc ghi dữ liệu. Các kiểu dữ liệu có thể không được ánh xạ theo cùng một cách theo cả hai hướng giữa Trino và nguồn dữ liệu. Tham khảo các phần sau để biết cách ánh xạ kiểu dữ liệu theo từng hướng.

Đặc tả Iceberg bao gồm các kiểu dữ liệu được hỗ trợ và ánh xạ tới định dạng trong các tệp Avro, ORC hoặc Parquet:

- [Iceberg to Avro](https://iceberg.apache.org/spec/#avro)
    
- [Iceberg to ORC](https://iceberg.apache.org/spec/#orc)
    
- [Iceberg to Parquet](https://iceberg.apache.org/spec/#parquet)

### Iceberg to Trino type mapping

Bộ kết nối ánh xạ các loại Iceberg với các loại Trino tương ứng theo bảng sau:

|   |   |
|---|---|
|Iceberg type|Trino type|
|BOOLEAN|BOOLEAN|
|INT|INTEGER|
|LONG|BIGINT|
|FLOAT|REAL|
|DOUBLE|DOUBLE|
|DECIMAL(p,s)|DECIMAL(p,s)|
|DATE|DATE|
|TIME|TIME(6)|
|TIMESTAMP|TIMESTAMP(6)|
|TIMESTAMPTZ|TIMESTAMP(6) WITH TIME ZONE|
|STRING|VARCHAR|
|UUID|UUID|
|BINARY|VARBINARY|
|FIXED (L)|VARBINARY|
|STRUCT(...)|ROW(...)|
|LIST(e)|ARRAY(e)|
|MAP(k,v)|MAP(k,v)|

Không hỗ trợ bất kỳ loại nào khác.

### Trino to Iceberg type mapping

Bộ kết nối ánh xạ các loại Trino với các loại Iceberg tương ứng theo bảng sau:

|   |   |
|---|---|
|Trino type|Iceberg type|
|BOOLEAN|BOOLEAN|
|INTEGER|INT|
|BIGINT|LONG|
|REAL|FLOAT|
|DOUBLE|DOUBLE|
|DECIMAL(p,s)|DECIMAL(p,s)|
|DATE|DATE|
|TIME(6)|TIME|
|TIMESTAMP(6)|TIMESTAMP|
|TIMESTAMP(6) WITH TIME ZONE|TIMESTAMPTZ|
|VARCHAR|STRING|
|UUID|UUID|
|VARBINARY|BINARY|
|ROW(...)|STRUCT(...)|
|ARRAY(e)|LIST(e)|
|MAP(k,v)|MAP(k,v)|

Không hỗ trợ bất kỳ loại nào khác.

## Security

### Kerberos authentication

Trình kết nối Iceberg hỗ trợ xác thực Kerberos cho Hive metastore và HDFS, đồng thời được cấu hình bằng các tham số tương tự như trình kết nối Hive. Tìm hiểu thêm thông tin trong phần hỗ trợ hệ thống tệp HDFS ([HDFS file system support](https://trino.io/docs/current/object-storage/file-system-hdfs.html)).

### Authorization

Trình kết nối Iceberg cho phép bạn chọn một trong nhiều phương tiện cung cấp quyền cấp phép ở catalog level.

Bạn có thể bật kiểm tra ủy quyền cho trình kết nối bằng cách thiết lập thuộc tính iceberg.security trong tệp thuộc tính catalog. Thuộc tính này phải là một trong các giá trị sau:

<div align="center"> 
	Iceberg security values 
<div>

|   |   |
|---|---|
|Property value|Description|
|ALLOW_ALL|No authorization checks are enforced.|
|SYSTEM|The connector relies on system-level access control.|
|READ_ONLY|Operations that read data or metadata, such as [SELECT](https://trino.io/docs/current/sql/select.html) are permitted. No operations that write data or metadata, such as [CREATE TABLE](https://trino.io/docs/current/sql/create-table.html), [INSERT](https://trino.io/docs/current/sql/insert.html), or [DELETE](https://trino.io/docs/current/sql/delete.html) are allowed.|
|FILE|Authorization checks are enforced using a catalog-level access control configuration file whose path is specified in the security.config-file catalog configuration property. See [Catalog-level access control files](https://trino.io/docs/current/security/file-system-access-control.html#catalog-file-based-access-control) for information on the authorization configuration file.|

  
## SQL support

Trình kết nối này cung cấp quyền truy cập đọc và ghi vào dữ liệu và metadata trong Iceberg. Ngoài các câu lệnh thao tác đọc ([read operation](https://trino.io/docs/current/language/sql-support.html#sql-read-operations)) và khả dụng toàn cục ([globally available](https://trino.io/docs/current/language/sql-support.html#sql-globally-available)), trình kết nối này còn hỗ trợ các tính năng sau:

- [Write operations](https://trino.io/docs/current/language/sql-support.html#sql-write-operations):  
      
- [Schema and table management](https://trino.io/docs/current/connector/iceberg.html#iceberg-schema-table-management) and [Partitioned tables](https://trino.io/docs/current/connector/iceberg.html#iceberg-tables)
    
- [Data management](https://trino.io/docs/current/connector/iceberg.html#iceberg-data-management)
    
- [View management](https://trino.io/docs/current/language/sql-support.html#sql-view-management)
    
- [Materialized view management](https://trino.io/docs/current/language/sql-support.html#sql-materialized-view-management), see also [Materialized views](https://trino.io/docs/current/connector/iceberg.html#iceberg-materialized-views)
    

### Basic usage examples

Bonus: Các vị trí schema (không phải vị trí table) được nhắc ở đây có nghĩa:

<vị trí schema>.<schema>.<table> 

Trình kết nối hỗ trợ tạo schema. Bạn có thể tạo schema có hoặc không có vị trí cụ thể.

Bạn có thể tạo một schema bằng câu lệnh CREATE SCHEMA và thuộc tính location schema. Các bảng trong schema này, không có vị trí được thiết lập rõ ràng trong câu lệnh CREATE TABLE, sẽ nằm trong một thư mục con thuộc thư mục tương ứng với location schema.

Tạo schema trên S3:

```sql
CREATE SCHEMA example.example_s3_schema
WITH (location = 's3://my-bucket/a/path/');
```

Tạo schema trên bộ lưu trữ object tương thích với S3 như MinIO:

```sql
CREATE SCHEMA example.example_s3a_schema
WITH (location = 's3a://my-bucket/a/path/');
```

Tạo schema trên HDFS:

```sql
CREATE SCHEMA example.example_hdfs_schema
WITH (location='hdfs://hadoop-master:9000/user/hive/warehouse/a/path/');
```

Tùy chọn, trên HDFS, vị trí có thể được bỏ qua:

```sql
CREATE SCHEMA example.example_hdfs_schema;
```

Trình kết nối Iceberg hỗ trợ tạo bảng bằng cú pháp CREATE TABLE. Bạn có thể tùy chọn chỉ định các thuộc tính bảng được trình kết nối này hỗ trợ:


Trình kết nối Iceberg hỗ trợ tạo bảng bằng cú pháp [CREATE TABLE](https://trino.io/docs/current/sql/create-table.html) . Bạn có thể tùy chọn chỉ định các thuộc tính bảng ([table properties](https://trino.io/docs/current/connector/iceberg.html#iceberg-table-properties)) được trình kết nối này hỗ trợ:

```sql
CREATE TABLE example_table (
    c1 INTEGER,
    c2 DATE,
    c3 DOUBLE
)
WITH (
    format = 'PARQUET',
    partitioning = ARRAY['c1', 'c2'],
    sorted_by = ARRAY['c3'],
    location = 's3://my-bucket/a/path/'
);
```
  
Khi thuộc tính bảng vị trí bị bỏ qua, nội dung của bảng sẽ được lưu trữ trong một thư mục con trong thư mục tương ứng với vị trí schema.

Một cách khác để tạo bảng bằng [CREATE TABLE AS](https://trino.io/docs/current/sql/create-table-as.html) là sử dụng cú pháp [VALUES](https://trino.io/docs/current/sql/values.html):

```sql
CREATE TABLE yearly_clicks (
    year,
    clicks
)
WITH (
    partitioning = ARRAY['year']
)
AS VALUES
    (2021, 10000),
    (2022, 20000);
```

### Procedures

Sử dụng câu lệnh CALL để thực hiện thao tác dữ liệu hoặc các tác vụ quản trị. Các thủ tục có sẵn trong schema hệ thống của mỗi catalog. Đoạn mã sau đây hiển thị cách gọi example_procedure trong examplecatalog catalog:

CALL examplecatalog.system.example_procedure();

#### Register table

Bộ kết nối có thể đăng ký các bảng Iceberg hiện có vào metastore liệu nếu iceberg.register-table-procedure.enabled được đặt thành true cho catalog.

Quy trình system.register_table cho phép người gọi đăng ký bảng Iceberg hiện có trong metastore, bằng cách sử dụng metadata và tệp dữ liệu hiện có của bảng đó:

```sql
CALL example.system.register_table(
  schema_name => 'testdb', 
  table_name => 'customer_orders', 
  table_location => 'hdfs://hadoop-master:9000/user/hive/warehouse/customer_orders-581fad8517934af6be1857a903559d44');
```

Để ngăn người dùng trái phép truy cập dữ liệu, quy trình này bị tắt theo mặc định. Quy trình này chỉ được bật khi iceberg.register-table-procedure.enabled được đặt thành true.

#### Unregister table

Trình kết nối có thể xóa các bảng Iceberg hiện có khỏi metastore. Sau khi hủy đăng ký, bạn sẽ không thể truy vấn bảng từ Trino nữa.

Quy trình system.unregister_table cho phép người gọi hủy đăng ký bảng Iceberg hiện có khỏi metastore mà không xóa dữ liệu:

```sql
CALL example.system.unregister_table(
  schema_name => 'testdb', 
  table_name => 'customer_orders');
```

#### Migrate table

Bộ kết nối có thể đọc hoặc ghi vào các bảng Hive đã được migrated sang Iceberg.

Sử dụng thủ tục system.migrate để di chuyển một bảng từ định dạng Hive sang định dạng Iceberg, được tải bằng các tệp dữ liệu nguồn. Table schema, phân vùng, thuộc tính và vị trí được sao chép từ bảng nguồn. Một bảng Hive được phân vùng sẽ được di chuyển dưới dạng một bảng Iceberg không được phân vùng. Các tệp dữ liệu trong bảng Hive phải sử dụng định dạng tệp Parquet, ORC hoặc Avro.

Quy trình này phải được gọi cho một ví dụ catalog cụ thể với schema và tên bảng có liên quan được cung cấp cùng với các tham số bắt buộc schema_name và table_name:

```sql
CALL example.system.migrate(
    schema_name => 'testdb',
    table_name => 'customer_orders');
```

Di chuyển sẽ không thành công nếu bất kỳ phân vùng bảng nào sử dụng định dạng tệp không được hỗ trợ.

Ngoài ra, bạn có thể cung cấp đối số recursive_directory để di chuyển bảng Hive có chứa các thư mục con:

```sql
CALL example.system.migrate(
    schema_name => 'testdb',
    table_name => 'customer_orders',
    recursive_directory => 'true');
```

Giá trị mặc định là fail, khiến quy trình migrate sẽ đưa ra ngoại lệ nếu tìm thấy thư mục con. Đặt giá trị thành true để di chuyển các thư mục lồng nhau, hoặc thành false để bỏ qua chúng.

#### Add files

Trình kết nối có thể thêm tệp từ bảng hoặc vị trí vào bảng Iceberg hiện có nếu iceberg.add-files-procedure.enabled được đặt thành true cho catalog.

Sử dụng quy trình add_files_from_table để thêm các tệp hiện có từ bảng Hive vào catalog hiện tại hoặc add_files để thêm các tệp hiện có từ vị trí đã chỉ định vào bảng Iceberg hiện có.

Các tệp dữ liệu phải ở định dạng tệp Parquet, ORC hoặc Avro.

Quy trình này thêm các tệp vào bảng đích, được chỉ định sau lệnh ALTER TABLE, và tải chúng từ bảng nguồn được chỉ định với các tham số bắt buộc schema_name và table_name. Bảng nguồn phải có thể truy cập được trong cùng catalog với bảng đích và sử dụng định dạng Hive. Bảng đích phải sử dụng định dạng Iceberg. Catalog phải sử dụng trình kết nối Iceberg.

Các ví dụ sau đây sao chép dữ liệu từ bảng Hive hive_customer_orders trong schema cũ của example catalog vào bảng Iceberg iceberg_customer_orders trong schema lakehouse của example catalog:

```sql
ALTER TABLE example.lakehouse.iceberg_customer_orders 
EXECUTE add_files_from_table(
    schema_name => 'legacy',
    table_name => 'customer_orders');
```

Ngoài ra, bạn có thể thiết lập catalog và schema hiện tại bằng câu lệnh USE và bỏ qua thông tin catalog và schema:

```sql
USE example.lakehouse;
ALTER TABLE iceberg_customer_orders 
EXECUTE add_files_from_table(
    schema_name => 'legacy',
    table_name => 'customer_orders');
```

Sử dụng đối số partition_filter để thêm tệp từ các phân vùng được chỉ định. Ví dụ sau đây thêm tệp từ một phân vùng có khu vực là ASIA và quốc gia là JAPAN:

```sql
ALTER TABLE example.lakehouse.iceberg_customer_orders 
EXECUTE add_files_from_table(
    schema_name => 'legacy',
    table_name => 'customer_orders',
    partition_filter => map(ARRAY['region', 'country'], ARRAY['ASIA', 'JAPAN']));
```

Ngoài ra, bạn có thể cung cấp đối số recursive_directory để di chuyển bảng Hive có chứa các thư mục con:

```sql
ALTER TABLE example.lakehouse.iceberg_customer_orders 
EXECUTE add_files_from_table(
    schema_name => 'legacy',
    table_name => 'customer_orders',
    recursive_directory => 'true');
```

Giá trị mặc định của recursive_directory là fail, khiến quy trình ném ra ngoại lệ nếu tìm thấy thư mục con. Đặt giá trị thành true để thêm tệp từ các thư mục lồng nhau, hoặc false để bỏ qua chúng.

Thủ tục add_files hỗ trợ việc thêm tệp, và do đó là dữ liệu chứa trong đó, vào một bảng đích, được chỉ định sau lệnh ALTER TABLE. Thủ tục này tải các tệp từ một đường dẫn lưu trữ đối tượng được chỉ định với tham số location bắt buộc. Các tệp phải sử dụng định dạng được chỉ định, với ORC và PARQUET là các giá trị hợp lệ. Bảng Iceberg đích phải sử dụng cùng định dạng với các tệp đã thêm. Thủ tục này không xác thực schema tệp về khả năng tương thích với bảng Iceberg đích. Thuộc tính location được hỗ trợ cho các bảng phân vùng.

Các ví dụ sau đây sao chép các tệp định dạng ORC từ vị trí s3://my-bucket/a/path vào bảng Iceberg iceberg_customer_orders trong lược đồ lakehouse của example catalog:

```sql
ALTER TABLE example.lakehouse.iceberg_customer_orders 
EXECUTE add_files(
    location => 's3://my-bucket/a/path',
    format => 'ORC');
```

### Functions

Các hàm có sẵn trong schema hệ thống của mỗi catalog. Các hàm có thể được gọi trong một câu lệnh SQL. Ví dụ: đoạn mã sau đây hiển thị cách thực thi hàm system.bucket trong  Iceberg catalog:

```sql
SELECT system.bucket('trino', 16);
```

#### bucket

Hàm này hiển thị [Iceberg bucket transform](https://iceberg.apache.org/spec/#bucket-transform-details) để người dùng có thể xác định một giá trị cụ thể thuộc bucket nào. Hàm này sử dụng hai đối số: giá trị phân vùng và số lượng bucket.

Các kiểu được hỗ trợ cho đối số thứ nhất của hàm này là:

- TINYINT

- SMALLINT

- INTEGER

- BIGINT

- VARCHAR
    
- VARBINARY
    
- DATE
    
- TIMESTAMP
    
- TIMESTAMP WITH TIME ZONE
    

Hàm này có thể được sử dụng trong mệnh đề WHERE để chỉ hoạt động trên một nhóm cụ thể:

```sql
SELECT count(*)
FROM customer
WHERE system.bucket(custkey, 16) = 2;
```

### Data management

Chức năng [Data management](https://trino.io/docs/current/language/sql-support.html#sql-data-management) bao gồm hỗ trợ cho các câu lệnh INSERT, UPDATE, DELETE, TRUNCATE và MERGE.

#### Deletion by partition

Đối với các bảng phân vùng, trình kết nối Iceberg hỗ trợ xóa toàn bộ phân vùng nếu mệnh đề WHERE chỉ định bộ lọc trên các cột phân vùng đã được chuyển đổi danh tính, có thể khớp với toàn bộ phân vùng. Dựa trên định nghĩa bảng từ phần [Partitioned Tables](https://trino.io/docs/current/connector/iceberg.html#iceberg-tables), câu lệnh SQL sau sẽ xóa tất cả các phân vùng có quốc gia là US:

```sql
DELETE FROM example.testdb.customer_orders
WHERE country = 'US';
```

Việc xóa phân vùng được thực hiện nếu mệnh đề WHERE đáp ứng các điều kiện này.

#### Row level deletion

Các bảng sử dụng v2 của đặc tả Iceberg hỗ trợ xóa từng hàng bằng cách ghi các tệp xóa vị trí.

### Schema and table management

Chức năng [Schema and table management](https://trino.io/docs/current/language/sql-support.html#sql-schema-table-management) bao gồm hỗ trợ cho:

- [CREATE SCHEMA](https://trino.io/docs/current/sql/create-schema.html)
    
- [DROP SCHEMA](https://trino.io/docs/current/sql/drop-schema.html)
    
- [ALTER SCHEMA](https://trino.io/docs/current/sql/alter-schema.html)
    
- [CREATE TABLE](https://trino.io/docs/current/sql/create-table.html)
    
- [CREATE TABLE AS](https://trino.io/docs/current/sql/create-table-as.html)
    
- [DROP TABLE](https://trino.io/docs/current/sql/drop-table.html)
    
- [ALTER TABLE](https://trino.io/docs/current/sql/alter-table.html)
    
- [COMMENT](https://trino.io/docs/current/sql/comment.html)
    

#### Schema evolution

Iceberg hỗ trợ schema evolution, với các thao tác thêm, xóa và đổi tên cột an toàn, bao gồm cả trong các cấu trúc lồng nhau.

Iceberg chỉ hỗ trợ cập nhật các loại cột cho các hoạt động mở rộng:

- INTEGER to BIGINT
    
- REAL to DOUBLE
    
- DECIMAL(p,s) to DECIMAL(p2,s) when p2 > p (scale cannot change)
    
Phân vùng cũng có thể được thay đổi và trình kết nối vẫn có thể truy vấn dữ liệu được tạo trước khi phân vùng thay đổi.

#### ALTER TABLE EXECUTE

Trình kết nối hỗ trợ các lệnh sau để sử dụng với [ALTER TABLE EXECUTE](https://trino.io/docs/current/sql/alter-table.html#alter-table-execute).

##### optimize

Lệnh optimize được sử dụng để ghi lại nội dung của bảng đã chỉ định sao cho nó được hợp nhất thành ít tệp hơn nhưng lớn hơn. Nếu bảng được phân vùng, quá trình nén dữ liệu sẽ hoạt động riêng biệt trên từng phân vùng được chọn để tối ưu hóa. Thao tác này cải thiện hiệu suất đọc.

Tất cả các tệp có kích thước nhỏ hơn tham số file_size_threshold tùy chọn (giá trị mặc định cho ngưỡng là 100MB) sẽ được hợp nhất trong trường hợp bất kỳ điều kiện nào sau đây được đáp ứng trên mỗi phân vùng:

- có nhiều hơn một tệp dữ liệu để hợp nhất
    
- có ít nhất một tệp dữ liệu, có đính kèm các tệp xóa
    
```sql
ALTER TABLE test_table EXECUTE optimize
```

Câu lệnh sau đây sẽ hợp nhất các tệp trong một bảng có kích thước dưới 128 megabyte:

```sql
ALTER TABLE test_table EXECUTE optimize(file_size_threshold => '128MB')
```

Bạn có thể sử dụng mệnh đề WHERE với các cột được sử dụng để phân vùng bảng nhằm lọc các phân vùng được tối ưu hóa:

```sql
ALTER TABLE test_partitioned_table EXECUTE optimize
WHERE partition_key = 1
```

Bạn có thể sử dụng mệnh đề WHERE phức tạp hơn để thu hẹp phạm vi của quy trình tối ưu hóa. Ví dụ sau đây chuyển đổi giá trị dấu thời gian sang ngày tháng và sử dụng phép so sánh để chỉ tối ưu hóa các phân vùng có dữ liệu từ năm 2022 trở về sau:

```sql
ALTER TABLE test_table EXECUTE optimize
WHERE CAST(timestamp_tz AS DATE) > DATE '2021-12-31'
```

Sử dụng mệnh đề WHERE với các [metadata columns](https://trino.io/docs/current/connector/iceberg.html#iceberg-metadata-columns) để lọc các tệp được tối ưu hóa.

```sql
ALTER TABLE test_table EXECUTE optimize
WHERE "$file_modified_time" > date_trunc('day', CURRENT_TIMESTAMP);
```

##### optimize_manifests

Viết lại các tệp manifest để nhóm chúng lại bằng cách phân vùng các cột. Điều này có thể được sử dụng để tối ưu hóa kế hoạch quét khi có nhiều tệp manifest nhỏ hoặc khi có bộ lọc phân vùng trong các truy vấn đọc nhưng các tệp manifest không được nhóm theo phân vùng. Thuộc tính bảng iceberg commit.manifest.target-size-bytes kiểm soát kích thước tối đa của các tệp manifest được tạo ra bởi quy trình này.

optimize_manifests có thể được chạy như sau:

```sql
ALTER TABLE test_table EXECUTE optimize_manifests;
```

##### expire_snapshots

Lệnh expire_snapshots sẽ xóa tất cả các snapshot và tất cả metadata và tệp dữ liệu liên quan. Việc thường xuyên xóa các snapshot hết hạn được khuyến nghị để xóa các tệp dữ liệu không còn cần thiết và giữ cho kích thước metadata của bảng nhỏ. Quy trình này áp dụng cho tất cả các snapshot cũ hơn khoảng thời gian được cấu hình với tham số retention_threshold.

expire_snapshots có thể được chạy như sau:

```sql
ALTER TABLE test_table EXECUTE expire_snapshots(retention_threshold => '7d');
```

Giá trị của retention_threshold phải cao hơn hoặc bằng iceberg.expire-snapshots.min-retention trong catalog, nếu không, quy trình sẽ thất bại với thông báo tương tự: Thời gian lưu trữ được chỉ định (1.00d) ngắn hơn thời gian lưu trữ tối thiểu được cấu hình trong hệ thống (7.00d). Giá trị mặc định cho thuộc tính này là 7d.

##### remove_orphan_files

Lệnh remove_orphan_files xóa tất cả các tệp khỏi thư mục dữ liệu của bảng nếu chúng không được liên kết với các metadata và có tuổi đời lớn hơn giá trị của tham số retention_threshold. Việc xóa các tệp mồ côi theo định kỳ được khuyến nghị để kiểm soát kích thước thư mục dữ liệu của bảng.

remove_orphan_files có thể được chạy như sau:

```sql
ALTER TABLE test_table EXECUTE remove_orphan_files(retention_threshold => '7d');
```


| metric_name               | metric_value |
|---------------------------|--------------|
| processed_manifests_count | 2            |
| active_files_count        | 98           |
| scanned_files_count       | 97           |
| deleted_files_count       | 0            |

  
Giá trị của retention_threshold phải cao hơn hoặc bằng iceberg.remove-orphan-files.min-retention trong catakig, nếu không, quy trình sẽ thất bại với thông báo tương tự: Thời gian lưu giữ được chỉ định (1.00d) ngắn hơn thời gian lưu giữ tối thiểu được cấu hình trong hệ thống (7.00d). Giá trị mặc định cho thuộc tính này là 7d.

Đầu ra của truy vấn có các số liệu sau:


|   |   |
|---|---|
|Property name|Description|
|processed_manifests_count|The count of manifest files read by remove_orphan_files.|
|active_files_count|The count of files belonging to snapshots that have not been expired.|
|scanned_files_count|The count of files scanned from the file system.|
|deleted_files_count|The count of files deleted by remove_orphan_files.|

##### drop_extended_stats

Lệnh drop_extended_stats xóa mọi thông tin thống kê mở rộng khỏi bảng.

drop_extended_stats có thể được chạy như sau:

```sql
ALTER TABLE test_table EXECUTE drop_extended_stats;
```

#### ALTER TABLE SET PROPERTIES

Trình kết nối hỗ trợ sửa đổi các thuộc tính trên các bảng hiện có bằng cách sử dụng [ALTER TABLE SET PROPERTIES](https://trino.io/docs/current/sql/alter-table.html#alter-table-set-properties).

Các thuộc tính bảng sau đây có thể được cập nhật sau khi bảng được tạo:

- format
    
- format_version
    
- partitioning
    
- sorted_by
    
- max_commit_retry
    
- object_store_layout_enabled
    
- data_location
    

Ví dụ, để cập nhật bảng từ v1 của đặc tả Iceberg lên v2:

```sql
ALTER TABLE table_name SET PROPERTIES format_version = 2;
```

Hoặc để đặt cột my_new_partition_column làm cột phân vùng trên một bảng:

```sql
ALTER TABLE table_name SET PROPERTIES partitioning = ARRAY[<existing partition columns>, 'my_new_partition_column'];
```

Giá trị hiện tại của các thuộc tính trong bảng có thể được hiển thị bằng lệnh [SHOW CREATE TABLE](https://trino.io/docs/current/sql/show-create-table.html)..

##### Table properties

Thuộc tính bảng cung cấp hoặc thiết lập metadata cho các bảng cơ sở. Đây là chìa khóa cho các câu lệnh [CREATE TABLE AS](https://trino.io/docs/current/sql/create-table-as.html). Thuộc tính bảng được truyền đến trình kết nối bằng mệnh đề [WITH](https://trino.io/docs/current/sql/create-table-as.html).

Thuộc tính iceberg

|Property name|Description|
|---|---|
|format|Optionally specifies the format of table data files; either PARQUET, ORC, or AVRO. Defaults to the value of the iceberg.file-format catalog configuration property, which defaults to PARQUET.|
|compression_codec|Optionally specifies the compression-codec used for writing the table; either NONE, ZSTD, SNAPPY, LZ4, or GZIP. Defaults to the value of the iceberg.compression-codec catalog configuration property, which defaults to ZSTD.|
|partitioning|Optionally specifies table partitioning. If a table is partitioned by columns c1 and c2, the partitioning property is partitioning = ARRAY['c1', 'c2'].|
|sorted_by|The sort order to be applied during writes to the content of each file written to the table. If the table files are sorted by columns c1 and c2, the sort order property is sorted_by = ARRAY['c1', 'c2']. The sort order applies to the contents written within each output file independently and not the entire dataset.|
|location|Optionally specifies the file system location URI for the table.|
|format_version|Optionally specifies the format version of the Iceberg specification to use for new tables; either 1 or 2. Defaults to 2. Version 2 is required for row level deletes.|
|max_commit_retry|Number of times to retry a commit before failing. Defaults to the value of the iceberg.max-commit-retry catalog configuration property, which defaults to 4.|
|orc_bloom_filter_columns|Comma-separated list of columns to use for ORC bloom filter. It improves the performance of queries using Equality and IN predicates when reading ORC files. Requires ORC format. Defaults to [].|
|orc_bloom_filter_fpp|The ORC bloom filters false positive probability. Requires ORC format. Defaults to 0.05.|
|parquet_bloom_filter_columns|Comma-separated list of columns to use for Parquet bloom filter. It improves the performance of queries using Equality and IN predicates when reading Parquet files. Requires Parquet format. Defaults to [].|
|object_store_layout_enabled|Whether Iceberg’s [object store file layout](https://iceberg.apache.org/docs/latest/aws/#object-store-file-layout) is enabled. Defaults to false.|
|data_location|Optionally specifies the file system location URI for the table’s data files|
|extra_properties|Additional properties added to an Iceberg table. The properties are not used by Trino, and are available in the $properties metadata table. The properties are not included in the output of SHOW CREATE TABLE statements.|

Định nghĩa bảng bên dưới chỉ định sử dụng tệp Parquet, phân vùng theo cột c1 và c2 và vị trí hệ thống tệp là /var/example_tables/test_table:

```sql
CREATE TABLE test_table (
    c1 INTEGER,
    c2 DATE,
    c3 DOUBLE)
WITH (
    format = 'PARQUET',
    partitioning = ARRAY['c1', 'c2'],
    location = '/var/example_tables/test_table');
```

Định nghĩa bảng bên dưới chỉ định sử dụng các tệp ORC với compression_codec SNAPPY, chỉ mục bộ lọc bloom theo cột c1 và c2, fpp là 0,05 và vị trí hệ thống tệp là /var/example_tables/test_table:

```sql
CREATE TABLE test_table (
    c1 INTEGER,
    c2 DATE,
    c3 DOUBLE)
WITH (
    format = 'ORC',
    compression_codec = 'SNAPPY',
    location = '/var/example_tables/test_table',
    orc_bloom_filter_columns = ARRAY['c1', 'c2'],
    orc_bloom_filter_fpp = 0.05);
```

Định nghĩa bảng bên dưới chỉ định sử dụng tệp Avro, phân vùng theo trường child1 trong cột cha:

```sql
CREATE TABLE test_table (
    data INTEGER,
    parent ROW(child1 DOUBLE, child2 INTEGER))
WITH (
    format = 'AVRO',
    partitioning = ARRAY['"parent.child1"']);
```

#### Metadata tables

