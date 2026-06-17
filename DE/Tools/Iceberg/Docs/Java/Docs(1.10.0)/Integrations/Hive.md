Iceberg hỗ trợ đọc và ghi các bảng Iceberg thông qua [Hive](https://hive.apache.org/) bằng cách sử dụng [StorageHandler](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers).

Xem thêm: [[DE/Tools/Hive/Introduction]]

## Feature support

Hive hỗ trợ các tính năng sau đây với phiên bản Hive 4.0.0 trở lên.

- Creating an Iceberg table.
- Creating an Iceberg identity-partitioned table.
- Creating an Iceberg table with any partition spec, including the various transforms supported by Iceberg.
- Creating a table from an existing table (CTAS table).
- Dropping a table.
- Altering a table while keeping Iceberg and Hive schemas in sync.
- Altering the partition schema (updating columns).
- Altering the partition schema by specifying partition transforms.
- Truncating a table / partition, dropping a partition.
- Migrating tables in Avro, Parquet, or ORC (Non-ACID) format to Iceberg.
- Reading an Iceberg table.
- Reading the schema of a table.
- Querying Iceberg metadata tables.
- Time travel applications.
- Inserting into a table / partition (INSERT INTO).
- Inserting data overwriting existing data (INSERT OVERWRITE) in a table / partition.
- Copy-on-write support for delete, update and merge queries, CRUD support for Iceberg V1 tables.
- Altering a table with expiring snapshots.
- Create a table like an existing table (CTLT table).
- Support adding parquet compression type via Table properties [Compression types](https://spark.apache.org/docs/2.4.3/sql-data-sources-parquet.html#configuration).
- Altering a table metadata location.
- Supporting table rollback.
- Honors sort orders on existing tables when writing a table [Sort orders specification](https://iceberg.apache.org/spec/#sort-orders).
- Creating, writing to and dropping an Iceberg branch / tag.
- Allowing expire snapshots by Snapshot ID, by time range, by retention of last N snapshots and using table properties.
- Set current snapshot using snapshot ID for an Iceberg table.
- Support for renaming an Iceberg table.
- Altering a table to convert to an Iceberg table.
- Fast forwarding, cherry-picking commit to an Iceberg branch.
- Creating a branch from an Iceberg tag.
- Set current snapshot using branch/tag for an Iceberg table.
- Delete orphan files for an Iceberg table.
- Allow full table compaction of Iceberg tables.
- Support of showing partition information for Iceberg tables (SHOW PARTITIONS).

>[!warning]
>DML operations work only with Tez execution engine.

## Enabling Iceberg support in Hive

Từ phiên bản 1.8.0 trở đi, Iceberg không phát hành Hive runtime connector nữa. Để tích hợp công cụ truy vấn Hive (đặc biệt với Hive 2.x và 3.x), hãy sử dụng Hive runtime connector đi kèm với Iceberg 1.6.1, hoặc sử dụng Hive 4.0.0 trở lên, phiên bản này đã được tích hợp sẵn Iceberg.

### Hive 4.0.x

Hive 4.0.x đi kèm với Iceberg 1.4.3.

...

Nguồn: https://iceberg.apache.org/docs/1.10.0/hive/#create-table-overlaying-an-existing-iceberg-table

