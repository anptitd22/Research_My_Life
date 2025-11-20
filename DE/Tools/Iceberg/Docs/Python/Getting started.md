
- [[#Installation|Installation]]
- [[#Connecting to a catalog|Connecting to a catalog]]
- [[#Write a PyArrow dataframe|Write a PyArrow dataframe]]
	- [[#Write a PyArrow dataframe#Explore Iceberg data and metadata files|Explore Iceberg data and metadata files]]
- [[#More details|More details]]

PyIceberg là một ứng dụng Python dùng để truy cập các bảng Iceberg mà không cần đến JVM.
## Installation

Trước khi cài đặt PyIceberg, hãy đảm bảo rằng bạn đang sử dụng phiên bản pip mới nhất:

```bash
pip install --upgrade pip
```

Bạn có thể cài đặt phiên bản phát hành mới nhất từ ​​pypi:

```bash
pip install "pyiceberg[s3fs,hive]"
```

Bạn có thể kết hợp các phụ thuộc tùy chọn tùy theo nhu cầu của mình:

|Key|Description:|
|---|---|
|hive|Support for the Hive metastore|
|hive-kerberos|Support for Hive metastore in Kerberos environment|
|glue|Support for AWS Glue|
|dynamodb|Support for AWS DynamoDB|
|bigquery|Support for Google Cloud BigQuery|
|sql-postgres|Support for SQL Catalog backed by Postgresql|
|sql-sqlite|Support for SQL Catalog backed by SQLite|
|pyarrow|PyArrow as a FileIO implementation to interact with the object store|
|pandas|Installs both PyArrow and Pandas|
|duckdb|Installs both PyArrow and DuckDB|
|ray|Installs PyArrow, Pandas, and Ray|
|bodo|Installs Bodo|
|daft|Installs Daft|
|polars|Installs Polars|
|s3fs|S3FS as a FileIO implementation to interact with the object store|
|adlfs|ADLFS as a FileIO implementation to interact with the object store|
|snappy|Support for snappy Avro compression|
|gcsfs|GCSFS as a FileIO implementation to interact with the object store|
|rest-sigv4|Support for generating AWS SIGv4 authentication headers for REST Catalogs|
|pyiceberg-core|Installs iceberg-rust powered core|
|datafusion|Installs both PyArrow and Apache DataFusion|
Bạn cần cài đặt s3fs, adlfs, gcsfs hoặc pyarrow để có thể lấy tệp từ kho lưu trữ đối tượng.

## Connecting to a catalog

Iceberg tận dụng catalog để có một nơi tập trung để sắp xếp các bảng ([catalog to have one centralized place to organize the tables](https://iceberg.apache.org/terms/#catalog)). Đây có thể là Hive catalog truyền thống để lưu trữ các bảng Iceberg của bạn bên cạnh các bảng còn lại, một giải pháp của nhà cung cấp như AWS Glue catalog, hoặc một triển khai [REST protocol](https://github.com/apache/iceberg/tree/main/open-api) riêng của Iceberg. Vui lòng xem trang [configuration](https://py.iceberg.apache.org/configuration/) để tìm tất cả chi tiết cấu hình.

Để minh họa, chúng ta sẽ cấu hình catalog để sử dụng triển khai SqlCatalog, triển khai này sẽ lưu trữ thông tin trong cơ sở dữ liệu sqlite cục bộ. Chúng ta cũng sẽ cấu hình catalog để lưu trữ các tệp dữ liệu trong hệ thống tệp cục bộ thay vì kho lưu trữ đối tượng. Không nên sử dụng triển khai này trong môi trường sản xuất do khả năng mở rộng hạn chế.

Tạo vị trí tạm thời cho Iceberg:

```bash
mkdir /tmp/warehouse
```

Mở Python 3 REPL để thiết lập catalog:

```python
from pyiceberg.catalog import load_catalog

warehouse_path = "/tmp/warehouse"
catalog = load_catalog(
    "default",
    **{
        'type': 'sql',
        "uri": f"sqlite:///{warehouse_path}/pyiceberg_catalog.db",
        "warehouse": f"file://{warehouse_path}",
    },
)
```

The SQL catalog hoạt động để thử nghiệm cục bộ mà không cần dịch vụ khác. Nếu bạn muốn dùng thử catalog khác, vui lòng kiểm tra [configuration](https://py.iceberg.apache.org/configuration/#catalogs).

## Write a PyArrow dataframe

Chúng ta hãy lấy tập dữ liệu Taxi và ghi vào bảng Iceberg.

Tải xuống dữ liệu của một tháng đầu tiên:

```bash
curl https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet -o /tmp/yellow_tripdata_2023-01.parquet
```

Tải nó vào khung dữ liệu PyArrow của bạn:

```python
import pyarrow.parquet as pq

df = pq.read_table("/tmp/yellow_tripdata_2023-01.parquet")
```

Tạo một bảng Iceberg mới:

```python
catalog.create_namespace("default")

table = catalog.create_table(
    "default.taxi_dataset",
    schema=df.schema,
)
```

Thêm khung dữ liệu vào bảng:

```python
table.append(df)
len(table.scan().to_arrow())
```

3066766 hàng đã được ghi vào bảng.

Bây giờ hãy tạo một tính năng tip-per-mile feature để đào tạo mô hình:

```python
import pyarrow.compute as pc

df = df.append_column("tip_per_mile", pc.divide(df["tip_amount"], df["trip_distance"]))
```

Evolve the schema của bảng với cột mới:

```bash
with table.update_schema() as update_schema:
    update_schema.union_by_name(df.schema)
```

Và bây giờ chúng ta có thể ghi khung dữ liệu mới vào bảng Iceberg:

```
table.overwrite(df)
print(table.scan().to_arrow())
```

Và cột mới ở đó:

```
taxi_dataset(
  1: VendorID: optional long,
  2: tpep_pickup_datetime: optional timestamp,
  3: tpep_dropoff_datetime: optional timestamp,
  4: passenger_count: optional double,
  5: trip_distance: optional double,
  6: RatecodeID: optional double,
  7: store_and_fwd_flag: optional string,
  8: PULocationID: optional long,
  9: DOLocationID: optional long,
  10: payment_type: optional long,
  11: fare_amount: optional double,
  12: extra: optional double,
  13: mta_tax: optional double,
  14: tip_amount: optional double,
  15: tolls_amount: optional double,
  16: improvement_surcharge: optional double,
  17: total_amount: optional double,
  18: congestion_surcharge: optional double,
  19: airport_fee: optional double,
  20: tip_per_mile: optional double
),
```

Và chúng ta có thể thấy rằng 2371784 hàng có một tip-per-mile:

```
df = table.scan(row_filter="tip_per_mile > 0").to_arrow()
len(df)
```

### Explore Iceberg data and metadata files

Vì catalog được cấu hình để sử dụng hệ thống tệp cục bộ nên chúng ta có thể khám phá cách Iceberg lưu dữ liệu và tệp metadata từ các hoạt động trên.

```
find /tmp/warehouse/
```

## More details

Để biết chi tiết, vui lòng kiểm tra trang [CLI](https://py.iceberg.apache.org/cli/) hoặc [Python API](https://py.iceberg.apache.org/api/).