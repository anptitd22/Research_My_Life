- [[#Apache Iceberg Sink Connector|Apache Iceberg Sink Connector]]
- [[#Features|Features]]
- [[#Installation|Installation]]
- [[#Requirements|Requirements]]
- [[#Configuration|Configuration]]
	- [[#Configuration#Kafka configuration|Kafka configuration]]
		- [[#Kafka configuration#Message format|Message format]]
	- [[#Configuration#Catalog configuration|Catalog configuration]]
		- [[#Catalog configuration#REST example|REST example]]
		- [[#Catalog configuration#Hive example|Hive example]]
		- [[#Catalog configuration#Glue example|Glue example]]
		- [[#Catalog configuration#Nessie example|Nessie example]]
		- [[#Catalog configuration#BigQuery Metastore example|BigQuery Metastore example]]
		- [[#Catalog configuration#Notes|Notes]]
	- [[#Configuration#Azure ADLS configuration example|Azure ADLS configuration example]]
	- [[#Configuration#Google GCS configuration example|Google GCS configuration example]]
	- [[#Configuration#Hadoop configuration|Hadoop configuration]]
- [[#Examples|Examples]]
	- [[#Examples#Initial setup|Initial setup]]
		- [[#Initial setup#Source topic|Source topic]]
		- [[#Initial setup#Control topic|Control topic]]
		- [[#Initial setup#Iceberg catalog configuration|Iceberg catalog configuration]]
	- [[#Examples#Single destination table|Single destination table]]
		- [[#Single destination table#Create the destination table|Create the destination table]]
		- [[#Single destination table#Connector config|Connector config]]
	- [[#Examples#Multi-table fan-out, static routing|Multi-table fan-out, static routing]]
		- [[#Multi-table fan-out, static routing#Create two destination tables|Create two destination tables]]
		- [[#Multi-table fan-out, static routing#Connector config|Connector config]]
	- [[#Examples#Multi-table fan-out, dynamic routing|Multi-table fan-out, dynamic routing]]
		- [[#Multi-table fan-out, dynamic routing#Create two destination tables|Create two destination tables]]
		- [[#Multi-table fan-out, dynamic routing#Connector config|Connector config]]
- [[#SMTs for the Apache Iceberg Sink Connector|SMTs for the Apache Iceberg Sink Connector]]
	- [[#SMTs for the Apache Iceberg Sink Connector#CopyValue|CopyValue]]
		- [[#CopyValue#Configuration|Configuration]]
		- [[#CopyValue#Example|Example]]
	- [[#SMTs for the Apache Iceberg Sink Connector#DmsTransform|DmsTransform]]
	- [[#SMTs for the Apache Iceberg Sink Connector#DebeziumTransform|DebeziumTransform]]
	- [[#SMTs for the Apache Iceberg Sink Connector#JsonToMapTransform|JsonToMapTransform]]
	- [[#SMTs for the Apache Iceberg Sink Connector#KafkaMetadataTransform|KafkaMetadataTransform]]
	- [[#SMTs for the Apache Iceberg Sink Connector#MongoDebeziumTransform|MongoDebeziumTransform]]

## Apache Iceberg Sink Connector

Apache Iceberg Sink Connector cho Kafka Connect là một bộ kết nối sink để ghi dữ liệu từ Kafka vào các bảng Iceberg.

## Features

- Commit coordination for centralized Iceberg commits
    
- Exactly-once delivery semantics
    
- Multi-table fan-out
    
- Automatic table creation and schema evolution
    
- Field name mapping via Iceberg’s column mapping functionality
    
## Installation

[https://github.com/apache/iceberg](https://github.com/apache/iceberg)

The connector zip archive is created as part of the Iceberg build. You can run the build via: 

```bash
cd kafka-connect

./gradlew -x test -x integrationTest clean build
```

Tệp lưu trữ zip sẽ được tìm thấy trong thư mục `./kafka-connect/kafka-connect-runtime/build/distributions`. Có một bản phân phối bao gồm máy khách Hive Metastore và các dependencies liên quan, và một bản không bao gồm. Sao chép distribution archive vào thư mục plugins của Kafka Connect trên tất cả các nodes.

## Requirements

Cơ chế xử lý dữ liệu này dựa vào KIP-447 để đảm bảo tính chính xác một lần duy nhất. Điều này yêu cầu Kafka 2.5 trở lên.

## Configuration

|Property|Description|
|---|---|
|iceberg.tables|Comma-separated list of destination tables|
|iceberg.tables.dynamic-enabled|Set to `true` to route to a table specified in `routeField` instead of using `routeRegex`, default is `false`|
|iceberg.tables.route-field|For multi-table fan-out, the name of the field used to route records to tables|
|iceberg.tables.default-commit-branch|Default branch for commits, main is used if not specified|
|iceberg.tables.default-id-columns|Default comma-separated list of columns that identify a row in tables (primary key)|
|iceberg.tables.default-partition-by|Default comma-separated list of partition field names to use when creating tables|
|iceberg.tables.auto-create-enabled|Set to `true` to automatically create destination tables, default is `false`|
|iceberg.tables.evolve-schema-enabled|Set to `true` to add any missing record fields to the table schema, default is `false`|
|iceberg.tables.schema-force-optional|Set to `true` to set columns as optional during table create and evolution, default is `false` to respect schema|
|iceberg.tables.schema-case-insensitive|Set to `true` to look up table columns by case-insensitive name, default is `false` for case-sensitive|
|iceberg.tables.auto-create-props.*|Properties set on new tables during auto-create|
|iceberg.tables.write-props.*|Properties passed through to Iceberg writer initialization, these take precedence|
|iceberg.table.<_table-name_>.commit-branch|Table-specific branch for commits, use `iceberg.tables.default-commit-branch` if not specified|
|iceberg.table.<_table-name_>.id-columns|Comma-separated list of columns that identify a row in the table (primary key)|
|iceberg.table.<_table-name_>.partition-by|Comma-separated list of partition fields to use when creating the table|
|iceberg.table.<_table-name_>.route-regex|The regex used to match a record's `routeField` to a table|
|iceberg.control.topic|Name of the control topic, default is `control-iceberg`|
|iceberg.control.group-id-prefix|Prefix for the control consumer group, default is `cg-control`|
|iceberg.control.commit.interval-ms|Commit interval in msec, default is 300,000 (5 min)|
|iceberg.control.commit.timeout-ms|Commit timeout interval in msec, default is 30,000 (30 sec)|
|iceberg.control.commit.threads|Number of threads to use for commits, default is (`cores * 2`)|
|iceberg.coordinator.transactional.prefix|Prefix for the transactional id to use for the coordinator producer, default is to use no/empty prefix|
|iceberg.catalog|Name of the catalog, default is `iceberg`|
|iceberg.catalog.*|Properties passed through to Iceberg catalog initialization|
|iceberg.hadoop-conf-dir|If specified, Hadoop config files in this directory will be loaded|
|iceberg.hadoop.*|Properties passed through to the Hadoop configuration|
|iceberg.kafka.*|Properties passed through to control topic Kafka client initialization|
Nếu `iceberg.tables.dynamic-enabled` là false (mặc định), bạn phải chỉ định `iceberg.tables`. Nếu `iceberg.tables.dynamic-enabled` là true, bạn phải chỉ định `iceberg.tables.route-field`, trường này sẽ chứa tên của bảng.

### Kafka configuration

Theo mặc định, trình kết nối sẽ cố gắng sử dụng cấu hình máy Kafka client từ thuộc tính của worker để kết nối với topic điều khiển. Nếu vì lý do nào đó không thể đọc được cấu hình đó, cài đặt máy Kafka client có thể được thiết lập rõ ràng bằng cách sử dụng các thuộc tính `iceberg.kafka.*`

#### Message format

Các messages cần được chuyển đổi thành struct hoặc map bằng cách sử dụng bộ chuyển đổi Kafka Connect thích hợp.

### Catalog configuration

Các thuộc tính `iceberg.catalog.*` là bắt buộc để kết nối với Iceberg catalog. Các loại catalog cốt lõi được bao gồm trong bản phân phối mặc định, bao gồm REST, Glue, DynamoDB, Hadoop, Nessie, JDBC, Hive và BigQuery Metastore. Trình điều khiển JDBC không được bao gồm trong bản phân phối mặc định, vì vậy bạn sẽ cần phải thêm chúng nếu cần. Khi sử dụng Hive catalog, bạn có thể sử dụng bản phân phối bao gồm máy Hive metastore client, nếu không bạn sẽ cần phải tự thêm nó.

Để thiết lập loại catalog, bạn có thể đặt `iceberg.catalog.type` thành rest, hive hoặc hadoop. Đối với các loại catalog khác, bạn cần đặt `iceberg.catalog.catalog-impl` thành tên của lớp catalog.

#### REST example

```
"iceberg.catalog.type": "rest",
"iceberg.catalog.uri": "https://catalog-service",
"iceberg.catalog.credential": "<credential>",
"iceberg.catalog.warehouse": "<warehouse>",
```

#### Hive example

LƯU Ý: Hãy sử dụng bản phân phối có bao gồm HMS client (hoặc tự cài đặt HMS client). Sử dụng `S3FileIO` khi dùng S3 để lưu trữ và `GCSFileIO` khi dùng GCS (mặc định là `HadoopFileIO` với `HiveCatalog`).

```
"iceberg.catalog.type": "hive",
"iceberg.catalog.uri": "thrift://hive:9083",
"iceberg.catalog.io-impl": "org.apache.iceberg.aws.s3.S3FileIO",
"iceberg.catalog.warehouse": "s3a://bucket/warehouse",
"iceberg.catalog.client.region": "us-east-1",
"iceberg.catalog.s3.access-key-id": "<AWS access>",
"iceberg.catalog.s3.secret-access-key": "<AWS secret>",
```

#### Glue example

```
"iceberg.catalog.catalog-impl": "org.apache.iceberg.aws.glue.GlueCatalog",
"iceberg.catalog.warehouse": "s3a://bucket/warehouse",
"iceberg.catalog.io-impl": "org.apache.iceberg.aws.s3.S3FileIO",
```

#### Nessie example

```
"iceberg.catalog.catalog-impl": "org.apache.iceberg.nessie.NessieCatalog",
"iceberg.catalog.uri": "http://localhost:19120/api/v2",
"iceberg.catalog.ref": "main",
"iceberg.catalog.warehouse": "s3a://bucket/warehouse",
"iceberg.catalog.io-impl": "org.apache.iceberg.aws.s3.S3FileIO",
```

#### BigQuery Metastore example

```
"iceberg.catalog.catalog-impl": "org.apache.iceberg.gcp.bigquery.BigQueryMetastoreCatalog",
"iceberg.catalog.gcp.bigquery.project-id": "my-project",
"iceberg.catalog.gcp.bigquery.location": "us-east1",
"iceberg.catalog.warehouse": "gs://bucket/warehouse",
"iceberg.catalog.io-impl": "org.apache.iceberg.gcp.gcs.GCSFileIO",
"iceberg.tables.auto-create-props.bq_connection": "projects/my-project/locations/us-east1/connections/my-connection",
```

#### Notes

Tùy thuộc vào cấu hình của bạn, bạn cũng có thể cần thiết lập `iceberg.catalog.s3.endpoint`, `iceberg.catalog.s3.staging-dir` hoặc `iceberg.catalog.s3.path-style-access`. Xem [Iceberg docs](https://iceberg.apache.org/docs/latest/) để biết chi tiết đầy đủ về cách cấu hình catalog.

### Azure ADLS configuration example

### Google GCS configuration example

### Hadoop configuration

## Examples

### Initial setup

#### Source topic

Điều này giả định rằng topic nguồn đã tồn tại và được đặt tên là `events`.

#### Control topic

Nếu cụm Kafka của bạn đã thiết lập `auto.create.topics.enable` thành `true` (mặc định), thì control topic sẽ được tạo tự động. Nếu không, bạn cần tạo topic trước. Tên topic mặc định là `control-iceberg`:

```
bin/kafka-topics.sh  \
  --command-config command-config.props \
  --bootstrap-server ${CONNECT_BOOTSTRAP_SERVERS} \
  --create \
  --topic control-iceberg \
  --partitions 1
```

_NOTE: Clusters running on Confluent Cloud have `auto.create.topics.enable` set to `false` by default._

#### Iceberg catalog configuration

Các thuộc tính cấu hình có tiền tố `iceberg.catalog.` sẽ được chuyển đến quá trình khởi tạo Iceberg catalog. Xem [Iceberg docs](https://iceberg.apache.org/docs/latest/) để biết chi tiết về cách cấu hình một catalog cụ thể.

### Single destination table

Ví dụ này ghi tất cả các bản ghi đến vào một bảng duy nhất.

#### Create the destination table

```
CREATE TABLE default.events (
    id STRING,
    type STRING,
    ts TIMESTAMP,
    payload STRING)
PARTITIONED BY (hours(ts))
```

#### Connector config

Cấu hình ví dụ này kết nối với REST catalog của Iceberg.

```
{
  "name": "events-sink",
  "config": {
    "connector.class": "org.apache.iceberg.connect.IcebergSinkConnector",
    "tasks.max": "2",
    "topics": "events",
    "iceberg.tables": "default.events",
    "iceberg.catalog.type": "rest",
    "iceberg.catalog.uri": "https://localhost",
    "iceberg.catalog.credential": "<credential>",
    "iceberg.catalog.warehouse": "<warehouse name>"
  }
}
```

### Multi-table fan-out, static routing

Ví dụ này ghi các bản ghi có kiểu được đặt là `list` vào bảng `default.events_list`, và ghi các bản ghi có kiểu được đặt là `create` vào bảng `default.events_create`. Các bản ghi khác sẽ bị bỏ qua.

#### Create two destination tables

```
CREATE TABLE default.events_list (
    id STRING,
    type STRING,
    ts TIMESTAMP,
    payload STRING)
PARTITIONED BY (hours(ts));

CREATE TABLE default.events_create (
    id STRING,
    type STRING,
    ts TIMESTAMP,
    payload STRING)
PARTITIONED BY (hours(ts));
```

#### Connector config

```
{
  "name": "events-sink",
  "config": {
    "connector.class": "org.apache.iceberg.connect.IcebergSinkConnector",
    "tasks.max": "2",
    "topics": "events",
    "iceberg.tables": "default.events_list,default.events_create",
    "iceberg.tables.route-field": "type",
    "iceberg.table.default.events_list.route-regex": "list",
    "iceberg.table.default.events_create.route-regex": "create",
    "iceberg.catalog.type": "rest",
    "iceberg.catalog.uri": "https://localhost",
    "iceberg.catalog.credential": "<credential>",
    "iceberg.catalog.warehouse": "<warehouse name>"
  }
}
```

### Multi-table fan-out, dynamic routing

ònVí dụ này ghi dữ liệu vào các bảng có tên được lấy từ giá trị trong trường `db_table`. Nếu bảng có tên đó không tồn tại, bản ghi sẽ bị bỏ qua. Ví dụ, nếu trường `db_table` của bản ghi được đặt thành `default.events_list`, thì bản ghi sẽ được ghi vào bảng `default.events_list`.

#### Create two destination tables

Xem hướng dẫn ở trên để tạo hai bảng.

#### Connector config

```
{
  "name": "events-sink",
  "config": {
    "connector.class": "org.apache.iceberg.connect.IcebergSinkConnector",
    "tasks.max": "2",
    "topics": "events",
    "iceberg.tables.dynamic-enabled": "true",
    "iceberg.tables.route-field": "db_table",
    "iceberg.catalog.type": "rest",
    "iceberg.catalog.uri": "https://localhost",
    "iceberg.catalog.credential": "<credential>",
    "iceberg.catalog.warehouse": "<warehouse name>"
  }
}
```

## SMTs for the Apache Iceberg Sink Connector

Dự án này chứa một số SMTs có thể hữu ích khi chuyển đổi dữ liệu Kafka để sử dụng bởi trình kết nối Iceberg sink.

### CopyValue

Chức năng `CopyValue` SMT sao chép giá trị từ một trường sang một trường mới.

#### Configuration

|Property|Description|
|---|---|
|source.field|Source field name|
|target.field|Target field name|
#### Example

```
"transforms": "copyId",
"transforms.copyId.type": "org.apache.iceberg.connect.transforms.CopyValue",
"transforms.copyId.source.field": "id",
"transforms.copyId.target.field": "id_copy",
```

### DmsTransform

### DebeziumTransform

### JsonToMapTransform

### KafkaMetadataTransform

### MongoDebeziumTransform

Nguồn: https://iceberg.apache.org/docs/nightly/kafka-connect/#multi-table-fan-out-dynamic-routing