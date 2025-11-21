
- [[#Create a table|Create a table]]
- [[#Load a table|Load a table]]
	- [[#Load a table#Catalog table|Catalog table]]
	- [[#Load a table#Static table|Static table]]
- [[#Check if a table exists|Check if a table exists]]
- [[#Write to a table|Write to a table]]
	- [[#Write to a table#Partial overwrites|Partial overwrites]]
	- [[#Write to a table#Upsert|Upsert]]
- [[#Inspecting tables|Inspecting tables]]
	- [[#Inspecting tables#Snapshots|Snapshots]]
	- [[#Inspecting tables#Partitions|Partitions]]
	- [[#Inspecting tables#Entries|Entries]]
	- [[#Inspecting tables#References|References]]
	- [[#Inspecting tables#Manifests|Manifests]]
	- [[#Inspecting tables#Metadata Log Entries|Metadata Log Entries]]
	- [[#Inspecting tables#History|History]]
	- [[#Inspecting tables#Files|Files]]
- [[#Add Files|Add Files]]
	- [[#Add Files#Usage|Usage]]
	- [[#Add Files#Example|Example]]
- [[#Schema evolution|Schema evolution]]
	- [[#Schema evolution#Union by Name|Union by Name]]
	- [[#Schema evolution#Add column|Add column]]
	- [[#Schema evolution#Rename column|Rename column]]
	- [[#Schema evolution#Move column|Move column]]
	- [[#Schema evolution#Update column|Update column]]
	- [[#Schema evolution#Delete column|Delete column]]
- [[#Partition evolution|Partition evolution]]
	- [[#Partition evolution#Add fields|Add fields]]
	- [[#Partition evolution#Remove fields|Remove fields]]
	- [[#Partition evolution#Rename fields|Rename fields]]
- [[#Sort order updates|Sort order updates]]
	- [[#Sort order updates#Updating a sort order on a table|Updating a sort order on a table]]
- [[#Table properties|Table properties]]
- [[#Snapshot properties|Snapshot properties]]
	- [[#Snapshot properties#Tags|Tags]]
	- [[#Snapshot properties#Branching|Branching]]
- [[#Table Maintenance|Table Maintenance]]
	- [[#Table Maintenance#Snapshot Expiration|Snapshot Expiration]]
		- [[#Snapshot Expiration#Real-world Example|Real-world Example]]
- [[#Views (skip)|Views (skip)]]

(Py)Iceberg tập trung vào [catalog](https://iceberg.apache.org/terms/#catalog). Nghĩa là việc đọc/ghi dữ liệu sẽ thông qua một catalog. Bước đầu tiên là khởi tạo một catalog để tải một bảng. Hãy sử dụng cấu hình sau trong `.pyiceberg.yaml` để định nghĩa một REST catalog có tên là `prod`:

```python
catalog:
  prod:
    uri: http://rest-catalog/ws/
    credential: t-1234:secret
```

Lưu ý rằng nhiều catalog có thể được định nghĩa trong cùng một `.pyiceberg.yaml`, ví dụ, trong trường hợp Hive và REST catalog:

```python
catalog:
  hive:
    uri: thrift://127.0.0.1:9083
    s3.endpoint: http://127.0.0.1:9000
    s3.access-key-id: admin
    s3.secret-access-key: password
  rest:
    uri: https://rest-server:8181/
    warehouse: my-warehouse
```

Các catalogs khác nhau có thể được tải trong PyIceberg theo tên của chúng: `load_catalog(name="hive")` và `load_catalog(name="rest")` . Tổng quan về các tùy chọn cấu hình có thể được tìm thấy trên [configuration page](https://py.iceberg.apache.org/configuration/).

Thông tin này phải được đặt bên trong tệp có tên `.pyiceberg.yaml` nằm trong thư mục `$HOME` hoặc `%USERPROFILE%` (tùy thuộc vào hệ điều hành là Unix hay Windows), trong thư mục làm việc hiện tại hoặc trong thư mục `$PYICEBERG_HOME` (nếu biến môi trường tương ứng được đặt).

Bạn cũng có thể tải catalog mà không cần sử dụng `.pyiceberg.yaml` bằng cách truyền trực tiếp các thuộc tính:

```python
from pyiceberg.catalog import load_catalog

catalog = load_catalog(
    "docs",
    **{
        "uri": "http://127.0.0.1:8181",
        "s3.endpoint": "http://127.0.0.1:9000",
        "py-io-impl": "pyiceberg.io.pyarrow.PyArrowFileIO",
        "s3.access-key-id": "admin",
        "s3.secret-access-key": "password",
    }
)
```

Tiếp theo, tạo một namespace:

```python
ns = catalog.list_namespaces()

assert ns == [("docs_example",)]
```

## Create a table

Để tạo bảng từ catalog:

```python
from pyiceberg.schema import Schema
from pyiceberg.types import (
    TimestampType,
    FloatType,
    DoubleType,
    StringType,
    NestedField,
    StructType,
)

schema = Schema(
    NestedField(field_id=1, name="datetime", field_type=TimestampType(), required=True),
    NestedField(field_id=2, name="symbol", field_type=StringType(), required=True),
    NestedField(field_id=3, name="bid", field_type=FloatType(), required=False),
    NestedField(field_id=4, name="ask", field_type=DoubleType(), required=False),
    NestedField(
        field_id=5,
        name="details",
        field_type=StructType(
            NestedField(
                field_id=4, name="created_by", field_type=StringType(), required=False
            ),
        ),
        required=False,
    ),
)

from pyiceberg.partitioning import PartitionSpec, PartitionField

partition_spec = PartitionSpec(
    PartitionField(
        source_id=1, field_id=1000, transform="day", name="datetime_day"
    )
)

from pyiceberg.table.sorting import SortOrder, SortField

# Sort on the symbol
sort_order = SortOrder(SortField(source_id=2, transform='identity'))

catalog.create_table(
    identifier="docs_example.bids",
    schema=schema,
    partition_spec=partition_spec,
    sort_order=sort_order,
)
```

Khi bảng được tạo, tất cả ID trong schema sẽ được gán lại để đảm bảo tính duy nhất.

Để tạo bảng bằng pyarrow schema:

```python
import pyarrow as pa

schema = pa.schema([
        pa.field("foo", pa.string(), nullable=True),
        pa.field("bar", pa.int32(), nullable=False),
        pa.field("baz", pa.bool_(), nullable=True),
])

catalog.create_table(
    identifier="docs_example.bids",
    schema=schema,
)
```

Một API khác để tạo bảng là sử dụng `create_table_transaction`. API này tương tự như các API khi cập nhật bảng. Đây là một API thân thiện cho cả việc thiết lập thông số phân vùng và thứ tự sắp xếp, vì bạn không cần phải xử lý ID trường.

```python
with catalog.create_table_transaction(identifier="docs_example.bids", schema=schema) as txn:
    with txn.update_schema() as update_schema:
        update_schema.add_column(path="new_column", field_type='string')

    with txn.update_spec() as update_spec:
        update_spec.add_identity("symbol")

    txn.set_properties(test_a="test_aa", test_b="test_b", test_c="test_c")
```

## Load a table

Có hai cách để đọc bảng Iceberg: thông qua catalog và bằng cách trỏ trực tiếp vào Iceberg metadata. Đọc qua catalog được ưu tiên hơn, còn trỏ trực tiếp vào metadata thì chỉ đọc được (read-only).

### Catalog table

Đang tải bảng `bids` :

```python
table = catalog.load_table("docs_example.bids")
# Equivalent to:
table = catalog.load_table(("docs_example", "bids"))
# The tuple syntax can be used if the namespace or table contains a dot.
```

Câu lệnh này trả về một `Table` biểu diễn bảng Iceberg có thể được truy vấn và thay đổi.

### Static table

Để tải bảng trực tiếp từ tệp `metadata.json` (tức là không sử dụng catalog), bạn có thể sử dụng `StaticTable` như sau:

```python
from pyiceberg.table import StaticTable

static_table = StaticTable.from_metadata(
    "s3://warehouse/wh/nyc.db/taxis/metadata/00002-6ea51ce3-62aa-4197-9cf8-43d07c3440ca.metadata.json"
)
```

Bảng tĩnh không cho phép các thao tác ghi. Nếu thư mục metadata bảng của bạn chứa tệp `version-hint.text`, bạn chỉ cần chỉ định đường dẫn gốc của bảng, và tệp `metadata.json` mới nhất sẽ tự động được giải quyết:

```python
from pyiceberg.table import StaticTable

static_table = StaticTable.from_metadata(
    "s3://warehouse/wh/nyc.db/taxis"
)
```

## Check if a table exists

Để kiểm tra xem bảng `bids` có tồn tại hay không:

```python
catalog.table_exists("docs_example.bids")
```

Trả về True nếu bảng đã tồn tại.

## Write to a table

Việc đọc và ghi được thực hiện bằng [Apache Arrow](https://arrow.apache.org/). Arrow là định dạng cột trong bộ nhớ giúp trao đổi dữ liệu nhanh chóng và phân tích trong bộ nhớ. Hãy xem xét Bảng Arrow sau:

```python
import pyarrow as pa

df = pa.Table.from_pylist(
    [
        {"city": "Amsterdam", "lat": 52.371807, "long": 4.896029},
        {"city": "San Francisco", "lat": 37.773972, "long": -122.431297},
        {"city": "Drachten", "lat": 53.11254, "long": 6.0989},
        {"city": "Paris", "lat": 48.864716, "long": 2.349014},
    ],
)
```

Tiếp theo, tạo một bảng bằng cách sử dụng Arrow schema:

```python
from pyiceberg.catalog import load_catalog

catalog = load_catalog("default")

tbl = catalog.create_table("default.cities", schema=df.schema)
```

Tiếp theo, ghi dữ liệu vào bảng. Cả lệnh `append` và `overwrite` đều cho kết quả như nhau, vì bảng trống khi tạo:

```python
tbl.append(df)

# or

tbl.overwrite(df)
```

>[!note]
>PyIceberg mặc định sử dụng lệnh [fast append](https://iceberg.apache.org/spec/#snapshots) để giảm thiểu lượng dữ liệu được ghi. Điều này cho phép các thao tác fast commit, giảm khả năng xung đột. Nhược điểm của lệnh fast append là nó tạo ra nhiều metadata hơn so với lệnh merge commit. Quá trình nén được lên kế hoạch ([Compaction is planned](https://github.com/apache/iceberg-python/issues/270)) và sẽ tự động ghi lại tất cả metadata khi đạt đến ngưỡng, nhằm duy trì hiệu suất đọc.

Bây giờ, dữ liệu được ghi vào bảng và bảng có thể được đọc bằng cách sử dụng tbl.scan().to_arrow():

```python
pyarrow.Table
city: string
lat: double
long: double
----
city: [["Amsterdam","San Francisco","Drachten","Paris"]]
lat: [[52.371807,37.773972,53.11254,48.864716]]
long: [[4.896029,-122.431297,6.0989,2.349014]]
```

Nếu chúng ta muốn thêm dữ liệu, chúng ta có thể sử dụng `.append()` một lần nữa:

```python
tbl.append(pa.Table.from_pylist(
    [{"city": "Groningen", "lat": 53.21917, "long": 6.56667}],
))
```

Khi đọc bảng `tbl.scan().to_arrow()` bạn có thể thấy `Groningen` hiện cũng là một phần của bảng:

```python
pyarrow.Table
city: string
lat: double
long: double
----
city: [["Amsterdam","San Francisco","Drachten","Paris"],["Groningen"]]
lat: [[52.371807,37.773972,53.11254,48.864716],[53.21917]]
long: [[4.896029,-122.431297,6.0989,2.349014],[6.56667]]
```

Các nested lists biểu thị các Arrow buffers khác nhau. Mỗi lần ghi tạo ra một [Parquet file](https://parquet.apache.org/), trong đó mỗi [row group](https://parquet.apache.org/docs/concepts/) được dịch thành một Arrow buffers. Trong trường hợp bảng lớn, PyIceberg cũng cho phép tùy chọn truyền phát bộ đệm bằng Arrow [RecordBatchReader](https://arrow.apache.org/docs/python/generated/pyarrow.RecordBatchReader.html), tránh việc phải kéo tất cả dữ liệu vào bộ nhớ ngay lập tức:

```python
for buf in tbl.scan().to_arrow_batch_reader():
    print(f"Buffer contains {len(buf)} rows")
```

Để tránh bất kỳ sự không nhất quán về kiểu nào trong khi viết, bạn có thể chuyển đổi Iceberg table schema thành Arrow:

```python
df = pa.Table.from_pylist(
    [{"city": "Groningen", "lat": 53.21917, "long": 6.56667}], schema=table.schema().as_arrow()
)

tbl.append(df)
```

Bạn có thể xóa một số dữ liệu khỏi bảng bằng cách gọi` tbl.delete()` với `delete_filter `mong muốn. Thao tác này sẽ sử dụng Iceberg metadata để chỉ mở các tệp Parquet chứa thông tin liên quan.

```
tbl.delete(delete_filter="city == 'Paris'")
```

Trong ví dụ trên, bất kỳ bản ghi nào có giá trị trường thành phố bằng `Paris `sẽ bị xóa. Chạy `tbl.scan().to_arrow() `sẽ cho kết quả:

```python
pyarrow.Table
city: string
lat: double
long: double
----
city: [["Amsterdam","San Francisco","Drachten"],["Groningen"]]
lat: [[52.371807,37.773972,53.11254],[53.21917]]
long: [[4.896029,-122.431297,6.0989],[6.56667]]
```

Trong trường hợp `tbl.delete(delete_filter="city == 'Groningen'")`, toàn bộ tệp Parquet sẽ bị xóa mà không kiểm tra nội dung của nó, vì từ Iceberg metadata, PyIceberg có thể suy ra rằng tất cả nội dung trong tệp đều thỏa mãn điều kiện.

### Partial overwrites

Khi sử dụng `overwrite` API , bạn có thể sử dụng `overwrite_filter` để xóa dữ liệu khớp với bộ lọc trước khi thêm dữ liệu mới vào bảng. Ví dụ, hãy xem xét bảng Iceberg sau:

```python
import pyarrow as pa
df = pa.Table.from_pylist(
    [
        {"city": "Amsterdam", "lat": 52.371807, "long": 4.896029},
        {"city": "San Francisco", "lat": 37.773972, "long": -122.431297},
        {"city": "Drachten", "lat": 53.11254, "long": 6.0989},
        {"city": "Paris", "lat": 48.864716, "long": 2.349014},
    ],
)

from pyiceberg.catalog import load_catalog
catalog = load_catalog("default")

tbl = catalog.create_table("default.cities", schema=df.schema)

tbl.append(df)
```

Bạn có thể ghi đè lên bản ghi của `Paris` bằng bản ghi của `New York` :

```python
from pyiceberg.expressions import EqualTo
df = pa.Table.from_pylist(
    [
        {"city": "New York", "lat": 40.7128, "long": 74.0060},
    ]
)
tbl.overwrite(df, overwrite_filter=EqualTo('city', "Paris"))
```

Điều này tạo ra kết quả sau với `tbl.scan().to_arrow()`:

```python
pyarrow.Table
city: large_string
lat: double
long: double
----
city: [["New York"],["Amsterdam","San Francisco","Drachten"]]
lat: [[40.7128],[52.371807,37.773972,53.11254]]
long: [[74.006],[4.896029,-122.431297,6.0989]]
```

Nếu bảng PyIceberg được phân vùng, bạn có thể sử dụng `tbl.dynamic_partition_overwrite(df)` để thay thế các phân vùng hiện có bằng các phân vùng mới được cung cấp trong dataframe. Các phân vùng cần thay thế sẽ được tự động phát hiện từ bảng mũi tên được cung cấp. Ví dụ: với bảng Iceberg có phân vùng được chỉ định trên trường "city":

``` python
from pyiceberg.schema import Schema
from pyiceberg.types import DoubleType, NestedField, StringType

schema = Schema(
    NestedField(1, "city", StringType(), required=False),
    NestedField(2, "lat", DoubleType(), required=False),
    NestedField(3, "long", DoubleType(), required=False),
)

tbl = catalog.create_table(
    "default.cities",
    schema=schema,
    partition_spec=PartitionSpec(PartitionField(source_id=1, field_id=1001, transform=IdentityTransform(), name="city_identity"))
)
```

Và chúng ta muốn ghi đè dữ liệu cho phân vùng `"Paris"`:

```python
import pyarrow as pa

df = pa.Table.from_pylist(
    [
        {"city": "Amsterdam", "lat": 52.371807, "long": 4.896029},
        {"city": "San Francisco", "lat": 37.773972, "long": -122.431297},
        {"city": "Drachten", "lat": 53.11254, "long": 6.0989},
        {"city": "Paris", "lat": -48.864716, "long": -2.349014},
    ],
)
tbl.append(df)
```

Sau đó chúng ta có thể gọi `dynamic_partition_overwrite` bằng bảng arrow này:

```python
df_corrected = pa.Table.from_pylist([
    {"city": "Paris", "lat": 48.864716, "long": 2.349014}
])
tbl.dynamic_partition_overwrite(df_corrected)
```

Điều này tạo ra kết quả sau với `tbl.scan().to_arrow()`:

```python
pyarrow.Table
city: large_string
lat: double
long: double
----
city: [["Paris"],["Amsterdam"],["Drachten"],["San Francisco"]]
lat: [[48.864716],[52.371807],[53.11254],[37.773972]]
long: [[2.349014],[4.896029],[6.0989],[-122.431297]]
```

### Upsert

PyIceberg hỗ trợ các thao tác upsert, nghĩa là nó có thể hợp nhất bảng Arrow vào bảng Iceberg. Các hàng được coi là giống nhau dựa trên trường định danh ([identifier field](https://iceberg.apache.org/spec/?column-projection#identifier-field-ids)). Nếu một hàng đã có trong bảng, nó sẽ cập nhật hàng đó. Nếu không tìm thấy hàng, nó sẽ chèn hàng mới.

Hãy xem xét bảng sau đây, có một số dữ liệu:

```python
from pyiceberg.schema import Schema
from pyiceberg.types import IntegerType, NestedField, StringType

import pyarrow as pa

schema = Schema(
    NestedField(1, "city", StringType(), required=True),
    NestedField(2, "inhabitants", IntegerType(), required=True),
    # Mark City as the identifier field, also known as the primary-key
    identifier_field_ids=[1]
)

tbl = catalog.create_table("default.cities", schema=schema)

arrow_schema = pa.schema(
    [
        pa.field("city", pa.string(), nullable=False),
        pa.field("inhabitants", pa.int32(), nullable=False),
    ]
)

# Write some data
df = pa.Table.from_pylist(
    [
        {"city": "Amsterdam", "inhabitants": 921402},
        {"city": "San Francisco", "inhabitants": 808988},
        {"city": "Drachten", "inhabitants": 45019},
        {"city": "Paris", "inhabitants": 2103000},
    ],
    schema=arrow_schema
)
tbl.append(df)
```

Tiếp theo, chúng ta sẽ upsert một bảng vào bảng Iceberg:

```python
df = pa.Table.from_pylist(
    [
        # Will be updated, the inhabitants has been updated
        {"city": "Drachten", "inhabitants": 45505},

        # New row, will be inserted
        {"city": "Berlin", "inhabitants": 3432000},

        # Ignored, already exists in the table
        {"city": "Paris", "inhabitants": 2103000},
    ],
    schema=arrow_schema
)
upd = tbl.upsert(df)

assert upd.rows_updated == 1
assert upd.rows_inserted == 1
```

PyIceberg sẽ tự động phát hiện những hàng nào cần được cập nhật, chèn vào hoặc có thể bỏ qua.

## Inspecting tables

Để khám phá metadata của bảng, có thể kiểm tra các bảng.

> [!TIP] Time Travel  
>Để kiểm tra siêu dữ liệu của bảng bằng tính năng time travel feature, hãy gọi phương thức inspect table với đối số `snapshot_id`. Du hành thời gian được hỗ trợ trên tất cả các bảng siêu dữ liệu, ngoại trừ `snapshot` và `ref`.
>
> ```python
> table.inspect.entries(snapshot_id=805611270568163028)
> ```

### Snapshots

Kiểm tra snapshot của bảng:

```python
table.inspect.snapshots()
```

```python
pyarrow.Table
committed_at: timestamp[ms] not null
snapshot_id: int64 not null
parent_id: int64
operation: string
manifest_list: string not null
summary: map<string, string>
  child 0, entries: struct<key: string not null, value: string> not null
      child 0, key: string not null
      child 1, value: string
----
committed_at: [[2024-03-15 15:01:25.682,2024-03-15 15:01:25.730,2024-03-15 15:01:25.772]]
snapshot_id: [[805611270568163028,3679426539959220963,5588071473139865870]]
parent_id: [[null,805611270568163028,3679426539959220963]]
operation: [["append","overwrite","append"]]
manifest_list: [["s3://warehouse/default/table_metadata_snapshots/metadata/snap-805611270568163028-0-43637daf-ea4b-4ceb-b096-a60c25481eb5.avro","s3://warehouse/default/table_metadata_snapshots/metadata/snap-3679426539959220963-0-8be81019-adf1-4bb6-a127-e15217bd50b3.avro","s3://warehouse/default/table_metadata_snapshots/metadata/snap-5588071473139865870-0-1382dd7e-5fbc-4c51-9776-a832d7d0984e.avro"]]
summary: [[keys:["added-files-size","added-data-files","added-records","total-data-files","total-delete-files","total-records","total-files-size","total-position-deletes","total-equality-deletes"]values:["5459","1","3","1","0","3","5459","0","0"],keys:["added-files-size","added-data-files","added-records","total-data-files","total-records",...,"total-equality-deletes","total-files-size","deleted-data-files","deleted-records","removed-files-size"]values:["5459","1","3","1","3",...,"0","5459","1","3","5459"],keys:["added-files-size","added-data-files","added-records","total-data-files","total-delete-files","total-records","total-files-size","total-position-deletes","total-equality-deletes"]values:["5459","1","3","2","0","6","10918","0","0"]]]
```

### Partitions

Kiểm tra partitions của bảng:

```python
table.inspect.partitions()
```

```python
pyarrow.Table
partition: struct<dt_month: int32, dt_day: date32[day]> not null
  child 0, dt_month: int32
  child 1, dt_day: date32[day]
spec_id: int32 not null
record_count: int64 not null
file_count: int32 not null
total_data_file_size_in_bytes: int64 not null
position_delete_record_count: int64 not null
position_delete_file_count: int32 not null
equality_delete_record_count: int64 not null
equality_delete_file_count: int32 not null
last_updated_at: timestamp[ms]
last_updated_snapshot_id: int64
----
partition: [
  -- is_valid: all not null
  -- child 0 type: int32
[null,null,612]
  -- child 1 type: date32[day]
[null,2021-02-01,null]]
spec_id: [[2,1,0]]
record_count: [[1,1,2]]
file_count: [[1,1,2]]
total_data_file_size_in_bytes: [[641,641,1260]]
position_delete_record_count: [[0,0,0]]
position_delete_file_count: [[0,0,0]]
equality_delete_record_count: [[0,0,0]]
equality_delete_file_count: [[0,0,0]]
last_updated_at: [[2024-04-13 18:59:35.981,2024-04-13 18:59:35.465,2024-04-13 18:59:35.003]]
```

### Entries

Để hiển thị tất cả các mục nhập manifest hiện tại của bảng cho cả tệp dữ liệu và tệp xóa.

```python
table.inspect.entries()
```

```python
pyarrow.Table
status: int8 not null
snapshot_id: int64 not null
sequence_number: int64 not null
file_sequence_number: int64 not null
data_file: struct<content: int8 not null, file_path: string not null, file_format: string not null, partition: struct<> not null, record_count: int64 not null, file_size_in_bytes: int64 not null, column_sizes: map<int32, int64>, value_counts: map<int32, int64>, null_value_counts: map<int32, int64>, nan_value_counts: map<int32, int64>, lower_bounds: map<int32, binary>, upper_bounds: map<int32, binary>, key_metadata: binary, split_offsets: list<item: int64>, equality_ids: list<item: int32>, sort_order_id: int32> not null
  child 0, content: int8 not null
  child 1, file_path: string not null
  child 2, file_format: string not null
  child 3, partition: struct<> not null
  child 4, record_count: int64 not null
  child 5, file_size_in_bytes: int64 not null
  child 6, column_sizes: map<int32, int64>
      child 0, entries: struct<key: int32 not null, value: int64> not null
          child 0, key: int32 not null
          child 1, value: int64
  child 7, value_counts: map<int32, int64>
      child 0, entries: struct<key: int32 not null, value: int64> not null
          child 0, key: int32 not null
          child 1, value: int64
  child 8, null_value_counts: map<int32, int64>
      child 0, entries: struct<key: int32 not null, value: int64> not null
          child 0, key: int32 not null
          child 1, value: int64
  child 9, nan_value_counts: map<int32, int64>
      child 0, entries: struct<key: int32 not null, value: int64> not null
          child 0, key: int32 not null
          child 1, value: int64
  child 10, lower_bounds: map<int32, binary>
      child 0, entries: struct<key: int32 not null, value: binary> not null
          child 0, key: int32 not null
          child 1, value: binary
  child 11, upper_bounds: map<int32, binary>
      child 0, entries: struct<key: int32 not null, value: binary> not null
          child 0, key: int32 not null
          child 1, value: binary
  child 12, key_metadata: binary
  child 13, split_offsets: list<item: int64>
      child 0, item: int64
  child 14, equality_ids: list<item: int32>
      child 0, item: int32
  child 15, sort_order_id: int32
readable_metrics: struct<city: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: string, upper_bound: string> not null, lat: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null, long: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null>
  child 0, city: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: string, upper_bound: string> not null
      child 0, column_size: int64
      child 1, value_count: int64
      child 2, null_value_count: int64
      child 3, nan_value_count: int64
      child 4, lower_bound: string
      child 5, upper_bound: string
  child 1, lat: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null
      child 0, column_size: int64
      child 1, value_count: int64
      child 2, null_value_count: int64
      child 3, nan_value_count: int64
      child 4, lower_bound: double
      child 5, upper_bound: double
  child 2, long: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null
      child 0, column_size: int64
      child 1, value_count: int64
      child 2, null_value_count: int64
      child 3, nan_value_count: int64
      child 4, lower_bound: double
      child 5, upper_bound: double
----
status: [[1]]
snapshot_id: [[6245626162224016531]]
sequence_number: [[1]]
file_sequence_number: [[1]]
data_file: [
  -- is_valid: all not null
  -- child 0 type: int8
[0]
  -- child 1 type: string
["s3://warehouse/default/cities/data/00000-0-80766b66-e558-4150-a5cf-85e4c609b9fe.parquet"]
  -- child 2 type: string
["PARQUET"]
  -- child 3 type: struct<>
    -- is_valid: all not null
  -- child 4 type: int64
[4]
  -- child 5 type: int64
[1656]
  -- child 6 type: map<int32, int64>
[keys:[1,2,3]values:[140,135,135]]
  -- child 7 type: map<int32, int64>
[keys:[1,2,3]values:[4,4,4]]
  -- child 8 type: map<int32, int64>
[keys:[1,2,3]values:[0,0,0]]
  -- child 9 type: map<int32, int64>
[keys:[]values:[]]
  -- child 10 type: map<int32, binary>
[keys:[1,2,3]values:[416D7374657264616D,8602B68311E34240,3A77BB5E9A9B5EC0]]
  -- child 11 type: map<int32, binary>
[keys:[1,2,3]values:[53616E204672616E636973636F,F5BEF1B5678E4A40,304CA60A46651840]]
  -- child 12 type: binary
[null]
  -- child 13 type: list<item: int64>
[[4]]
  -- child 14 type: list<item: int32>
[null]
  -- child 15 type: int32
[null]]
readable_metrics: [
  -- is_valid: all not null
  -- child 0 type: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: string, upper_bound: string>
    -- is_valid: all not null
    -- child 0 type: int64
[140]
    -- child 1 type: int64
[4]
    -- child 2 type: int64
[0]
    -- child 3 type: int64
[null]
    -- child 4 type: string
["Amsterdam"]
    -- child 5 type: string
["San Francisco"]
  -- child 1 type: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double>
    -- is_valid: all not null
    -- child 0 type: int64
[135]
    -- child 1 type: int64
[4]
    -- child 2 type: int64
[0]
    -- child 3 type: int64
[null]
    -- child 4 type: double
[37.773972]
    -- child 5 type: double
[53.11254]
  -- child 2 type: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double>
    -- is_valid: all not null
    -- child 0 type: int64
[135]
    -- child 1 type: int64
[4]
    -- child 2 type: int64
[0]
    -- child 3 type: int64
[null]
    -- child 4 type: double
[-122.431297]
    -- child 5 type: double
[6.0989]]
```

### References

Để hiển thị các tham chiếu snapshot đã biết của một bảng:

```python
table.inspect.refs()
```

```python
pyarrow.Table
name: string not null
type: string not null
snapshot_id: int64 not null
max_reference_age_in_ms: int64
min_snapshots_to_keep: int32
max_snapshot_age_in_ms: int64
----
name: [["main","testTag"]]
type: [["BRANCH","TAG"]]
snapshot_id: [[2278002651076891950,2278002651076891950]]
max_reference_age_in_ms: [[null,604800000]]
min_snapshots_to_keep: [[null,10]]
max_snapshot_age_in_ms: [[null,604800000]]
```

### Manifests

Để hiển thị tệp manifests hiện tại của bảng:

```python
table.inspect.manifests()
```

```python
pyarrow.Table
content: int8 not null
path: string not null
length: int64 not null
partition_spec_id: int32 not null
added_snapshot_id: int64 not null
added_data_files_count: int32 not null
existing_data_files_count: int32 not null
deleted_data_files_count: int32 not null
added_delete_files_count: int32 not null
existing_delete_files_count: int32 not null
deleted_delete_files_count: int32 not null
partition_summaries: list<item: struct<contains_null: bool not null, contains_nan: bool, lower_bound: string, upper_bound: string>> not null
  child 0, item: struct<contains_null: bool not null, contains_nan: bool, lower_bound: string, upper_bound: string>
      child 0, contains_null: bool not null
      child 1, contains_nan: bool
      child 2, lower_bound: string
      child 3, upper_bound: string
----
content: [[0]]
path: [["s3://warehouse/default/table_metadata_manifests/metadata/3bf5b4c6-a7a4-4b43-a6ce-ca2b4887945a-m0.avro"]]
length: [[6886]]
partition_spec_id: [[0]]
added_snapshot_id: [[3815834705531553721]]
added_data_files_count: [[1]]
existing_data_files_count: [[0]]
deleted_data_files_count: [[0]]
added_delete_files_count: [[0]]
existing_delete_files_count: [[0]]
deleted_delete_files_count: [[0]]
partition_summaries: [[    -- is_valid: all not null
    -- child 0 type: bool
[false]
    -- child 1 type: bool
[false]
    -- child 2 type: string
["test"]
    -- child 3 type: string
["test"]]]
```

### Metadata Log Entries

Để hiển thị các table metadata log:

```python
table.inspect.metadata_log_entries()
```

```python
pyarrow.Table
timestamp: timestamp[ms] not null
file: string not null
latest_snapshot_id: int64
latest_schema_id: int32
latest_sequence_number: int64
----
timestamp: [[2024-04-28 17:03:00.214,2024-04-28 17:03:00.352,2024-04-28 17:03:00.445,2024-04-28 17:03:00.498]]
file: [["s3://warehouse/default/table_metadata_log_entries/metadata/00000-0b3b643b-0f3a-4787-83ad-601ba57b7319.metadata.json","s3://warehouse/default/table_metadata_log_entries/metadata/00001-f74e4b2c-0f89-4f55-822d-23d099fd7d54.metadata.json","s3://warehouse/default/table_metadata_log_entries/metadata/00002-97e31507-e4d9-4438-aff1-3c0c5304d271.metadata.json","s3://warehouse/default/table_metadata_log_entries/metadata/00003-6c8b7033-6ad8-4fe4-b64d-d70381aeaddc.metadata.json"]]
latest_snapshot_id: [[null,3958871664825505738,1289234307021405706,7640277914614648349]]
latest_schema_id: [[null,0,0,0]]
latest_sequence_number: [[null,0,0,0]]
```

### History

Để hiển thị lịch sử của một bảng:

```python
table.inspect.history()
```

```python
pyarrow.Table
made_current_at: timestamp[ms] not null
snapshot_id: int64 not null
parent_id: int64
is_current_ancestor: bool not null
----
made_current_at: [[2024-06-18 16:17:48.768,2024-06-18 16:17:49.240,2024-06-18 16:17:49.343,2024-06-18 16:17:49.511]]
snapshot_id: [[4358109269873137077,3380769165026943338,4358109269873137077,3089420140651211776]]
parent_id: [[null,4358109269873137077,null,4358109269873137077]]
is_current_ancestor: [[true,false,true,true]]
```

### Files

Kiểm tra các tệp dữ liệu trong snapshot hiện tại của bảng:

```python
table.inspect.files()
```

```python
pyarrow.Table
content: int8 not null
file_path: string not null
file_format: dictionary<values=string, indices=int32, ordered=0> not null
spec_id: int32 not null
record_count: int64 not null
file_size_in_bytes: int64 not null
column_sizes: map<int32, int64>
  child 0, entries: struct<key: int32 not null, value: int64> not null
      child 0, key: int32 not null
      child 1, value: int64
value_counts: map<int32, int64>
  child 0, entries: struct<key: int32 not null, value: int64> not null
      child 0, key: int32 not null
      child 1, value: int64
null_value_counts: map<int32, int64>
  child 0, entries: struct<key: int32 not null, value: int64> not null
      child 0, key: int32 not null
      child 1, value: int64
nan_value_counts: map<int32, int64>
  child 0, entries: struct<key: int32 not null, value: int64> not null
      child 0, key: int32 not null
      child 1, value: int64
lower_bounds: map<int32, binary>
  child 0, entries: struct<key: int32 not null, value: binary> not null
      child 0, key: int32 not null
      child 1, value: binary
upper_bounds: map<int32, binary>
  child 0, entries: struct<key: int32 not null, value: binary> not null
      child 0, key: int32 not null
      child 1, value: binary
key_metadata: binary
split_offsets: list<item: int64>
  child 0, item: int64
equality_ids: list<item: int32>
  child 0, item: int32
sort_order_id: int32
readable_metrics: struct<city: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: large_string, upper_bound: large_string> not null, lat: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null, long: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null>
  child 0, city: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: string, upper_bound: string> not null
      child 0, column_size: int64
      child 1, value_count: int64
      child 2, null_value_count: int64
      child 3, nan_value_count: int64
      child 4, lower_bound: large_string
      child 5, upper_bound: large_string
  child 1, lat: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null
      child 0, column_size: int64
      child 1, value_count: int64
      child 2, null_value_count: int64
      child 3, nan_value_count: int64
      child 4, lower_bound: double
      child 5, upper_bound: double
  child 2, long: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double> not null
      child 0, column_size: int64
      child 1, value_count: int64
      child 2, null_value_count: int64
      child 3, nan_value_count: int64
      child 4, lower_bound: double
      child 5, upper_bound: double
----
content: [[0,0]]
file_path: [["s3://warehouse/default/table_metadata_files/data/00000-0-9ea7d222-6457-467f-bad5-6fb125c9aa5f.parquet","s3://warehouse/default/table_metadata_files/data/00000-0-afa8893c-de71-4710-97c9-6b01590d0c44.parquet"]]
file_format: [["PARQUET","PARQUET"]]
spec_id: [[0,0]]
record_count: [[3,3]]
file_size_in_bytes: [[5459,5459]]
column_sizes: [[keys:[1,2,3,4,5,...,8,9,10,11,12]values:[49,78,128,94,118,...,118,118,94,78,109],keys:[1,2,3,4,5,...,8,9,10,11,12]values:[49,78,128,94,118,...,118,118,94,78,109]]]
value_counts: [[keys:[1,2,3,4,5,...,8,9,10,11,12]values:[3,3,3,3,3,...,3,3,3,3,3],keys:[1,2,3,4,5,...,8,9,10,11,12]values:[3,3,3,3,3,...,3,3,3,3,3]]]
null_value_counts: [[keys:[1,2,3,4,5,...,8,9,10,11,12]values:[1,1,1,1,1,...,1,1,1,1,1],keys:[1,2,3,4,5,...,8,9,10,11,12]values:[1,1,1,1,1,...,1,1,1,1,1]]]
nan_value_counts: [[keys:[]values:[],keys:[]values:[]]]
lower_bounds: [[keys:[1,2,3,4,5,...,8,9,10,11,12]values:[00,61,61616161616161616161616161616161,01000000,0100000000000000,...,009B6ACA38F10500,009B6ACA38F10500,9E4B0000,01,00000000000000000000000000000000],keys:[1,2,3,4,5,...,8,9,10,11,12]values:[00,61,61616161616161616161616161616161,01000000,0100000000000000,...,009B6ACA38F10500,009B6ACA38F10500,9E4B0000,01,00000000000000000000000000000000]]]
upper_bounds:[[keys:[1,2,3,4,5,...,8,9,10,11,12]values:[00,61,61616161616161616161616161616161,01000000,0100000000000000,...,009B6ACA38F10500,009B6ACA38F10500,9E4B0000,01,00000000000000000000000000000000],keys:[1,2,3,4,5,...,8,9,10,11,12]values:[00,61,61616161616161616161616161616161,01000000,0100000000000000,...,009B6ACA38F10500,009B6ACA38F10500,9E4B0000,01,00000000000000000000000000000000]]]
key_metadata: [[0100,0100]]
split_offsets:[[[],[]]]
equality_ids:[[[],[]]]
sort_order_id:[[[],[]]]
readable_metrics: [
  -- is_valid: all not null
  -- child 0 type: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: large_string, upper_bound: large_string>
    -- is_valid: all not null
    -- child 0 type: int64
[140]
    -- child 1 type: int64
[4]
    -- child 2 type: int64
[0]
    -- child 3 type: int64
[null]
    -- child 4 type: large_string
["Amsterdam"]
    -- child 5 type: large_string
["San Francisco"]
  -- child 1 type: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double>
    -- is_valid: all not null
    -- child 0 type: int64
[135]
    -- child 1 type: int64
[4]
    -- child 2 type: int64
[0]
    -- child 3 type: int64
[null]
    -- child 4 type: double
[37.773972]
    -- child 5 type: double
[53.11254]
  -- child 2 type: struct<column_size: int64, value_count: int64, null_value_count: int64, nan_value_count: int64, lower_bound: double, upper_bound: double>
    -- is_valid: all not null
    -- child 0 type: int64
[135]
    -- child 1 type: int64
[4]
    -- child 2 type: int64
[0]
    -- child 3 type: int64
[null]
    -- child 4 type: double
[-122.431297]
    -- child 5 type: double
[6.0989]]
```

>[!info]
>Nội dung đề cập đến loại nội dung được lưu trữ bởi tệp dữ liệu: `0` - `Data`, `1` - `Position Deletes`, `2` - `Equality Deletes`

Để chỉ hiển thị các tệp dữ liệu hoặc xóa các tệp trong snapshot hiện tại, hãy sử dụng `table.inspect.data_files()` và `table.inspect.delete_files()` tương ứng.

## Add Files

Người dùng Iceberg chuyên nghiệp có thể commit existing parquet files vào bảng Iceberg dưới dạng tệp dữ liệu mà không cần ghi lại chúng.

>[!note]
>Vì `add_files` sử dụng các tệp hiện có mà không cần ghi các tệp parquet mới nhận biết Iceberg schema, nên nó yêu cầu bảng Iceberg phải có [Name Mapping](https://iceberg.apache.org/spec/?h=name+mapping#name-mapping-serialization) (The Name Mapping sẽ ánh xạ tên trường trong các tệp parquet với ID trường Iceberg). Do đó, `add_files` yêu cầu không có ID trường nào trong metadata của tệp parquet và tạo một Name Mapping mới dựa trên schema hiện tại của bảng nếu bảng chưa có.

>[!note]
>`add_files` chỉ yêu cầu client đọc phần metadata footer của các tệp parquet hiện có để suy ra giá trị phân vùng của mỗi tệp. Việc triển khai này cũng hỗ trợ việc thêm tệp vào bảng Iceberg bằng các phép biến đổi phân vùng như `MonthTransform` và `TruncateTransform`, giúp bảo toàn thứ tự các giá trị sau khi biến đổi (Bất kỳ phép biến đổi nào có thuộc tính `preserves_order` được đặt thành True đều được hỗ trợ). Xin lưu ý rằng nếu số liệu thống kê cột của cột nguồn của `PartitionField` không có trong parquet metadata, giá trị phân vùng sẽ được suy ra là `None`.

>[!warning]
>Vì `add_files` xác nhận các tệp parquet hiện có vào Bảng Iceberg như bất kỳ tệp dữ liệu nào khác, nên các hoạt động bảo trì mang tính hủy diệt như snapshot hết hạn sẽ xóa chúng.

>[!Warning]
>Tham số `check_duplicate_files` xác định liệu phương thức có xác thực rằng các `file_paths` được chỉ định chưa tồn tại trong bảng Iceberg hay không. Khi được đặt thành True (mặc định), phương thức sẽ thực hiện xác thực với các tệp dữ liệu hiện tại của bảng để tránh trùng lặp ngẫu nhiên, giúp duy trì tính nhất quán của dữ liệu bằng cách đảm bảo cùng một tệp không được thêm nhiều lần. Mặc dù kiểm tra này rất quan trọng đối với tính toàn vẹn dữ liệu, nhưng nó có thể gây ra tình trạng quá tải hiệu suất cho các bảng có số lượng tệp lớn. Việc đặt check_duplicate_files=False có thể cải thiện hiệu suất nhưng làm tăng nguy cơ tệp trùng lặp, có thể dẫn đến dữ liệu không nhất quán hoặc hỏng bảng. Khuyến nghị mạnh mẽ nên bật tham số này trừ khi việc xử lý tệp trùng lặp được thực thi nghiêm ngặt ở nơi khác.

### Usage

| Parameter               | Required? | Type           | Description                                                             |
| ----------------------- | --------- | -------------- | ----------------------------------------------------------------------- |
| `file_paths`            | ✔️        | List[str]      | The list of full file paths to be added as data files to the table      |
| `snapshot_properties`   |           | Dict[str, str] | Properties to set for the new snapshot. Defaults to an empty dictionary |
| `check_duplicate_files` |           | bool           | Whether to check for duplicate files. Defaults to `True`                |
### Example

Thêm tệp vào bảng Iceberg:

```python
# Given that these parquet files have schema consistent with the Iceberg table

file_paths = [
    "s3a://warehouse/default/existing-1.parquet",
    "s3a://warehouse/default/existing-2.parquet",
]

# They can be added to the table without rewriting them

tbl.add_files(file_paths=file_paths)

# A new snapshot is committed to the table with manifests pointing to the existing parquet files
```

Thêm tệp vào bảng Iceberg với các thuộc tính snapshot tùy chỉnh:

```python
# Assume an existing Iceberg table object `tbl`

file_paths = [
    "s3a://warehouse/default/existing-1.parquet",
    "s3a://warehouse/default/existing-2.parquet",
]

# Custom snapshot properties
snapshot_properties = {"abc": "def"}

# Enable duplicate file checking
check_duplicate_files = True

# Add the Parquet files to the Iceberg table without rewriting
tbl.add_files(
    file_paths=file_paths,
    snapshot_properties=snapshot_properties,
    check_duplicate_files=check_duplicate_files
)

# NameMapping must have been set to enable reads
assert tbl.name_mapping() is not None

# Verify that the snapshot property was set correctly
assert tbl.metadata.snapshots[-1].summary["abc"] == "def"
```

## Schema evolution

PyIceberg hỗ trợ quá trình schema evolution đầy đủ thông qua API Python. Nó đảm nhiệm việc thiết lập ID trường và đảm bảo chỉ thực hiện các thay đổi không gây gián đoạn (có thể ghi đè).

Trong các ví dụ dưới đây, `.update_schema()` được gọi từ chính bảng đó.

```python
with table.update_schema() as update:
    update.add_column("some_field", IntegerType(), "doc")
```

Bạn cũng có thể khởi tạo transaction nếu muốn thực hiện nhiều thay đổi hơn là chỉ  evolving the schema:

```python
with table.transaction() as transaction:
    with transaction.update_schema() as update_schema:
        update.add_column("some_other_field", IntegerType(), "doc")
    # ... Update properties etc
```

### Union by Name

Khi sử dụng `.union_by_name()`  bạn có thể merge một schema khác vào schema hiện có mà không cần phải lo lắng về ID trường:

```python
from pyiceberg.catalog import load_catalog
from pyiceberg.schema import Schema
from pyiceberg.types import NestedField, StringType, DoubleType, LongType

catalog = load_catalog()

schema = Schema(
    NestedField(1, "city", StringType(), required=False),
    NestedField(2, "lat", DoubleType(), required=False),
    NestedField(3, "long", DoubleType(), required=False),
)

table = catalog.create_table("default.locations", schema)

new_schema = Schema(
    NestedField(1, "city", StringType(), required=False),
    NestedField(2, "lat", DoubleType(), required=False),
    NestedField(3, "long", DoubleType(), required=False),
    NestedField(10, "population", LongType(), required=False),
)

with table.update_schema() as update:
    update.union_by_name(new_schema)
```

Bây giờ bảng có sự hợp nhất của hai schemas `print(table.schema())`:

```python
table {
  1: city: optional string
  2: lat: optional double
  3: long: optional double
  4: population: optional long
}
```

### Add column

Sử dụng `add_column` bạn có thể thêm một cột, không cần phải lo lắng về field-id:

```
with table.update_schema() as update:
    update.add_column("retries", IntegerType(), "Number of retries to place the bid")
    # In a struct
    update.add_column("details", StructType())

with table.update_schema() as update:
    update.add_column(("details", "confirmed_by"), StringType(), "Name of the exchange")
```

Một kiểu dữ liệu phức tạp phải tồn tại trước khi có thể thêm cột vào. Các trường trong kiểu dữ liệu phức tạp được thêm vào theo một tuple.

### Rename column

Đổi tên một trường trong bảng Iceberg rất đơn giản:

```python
with table.update_schema() as update:
    update.rename_column("retries", "num_retries")
    # This will rename `confirmed_by` to `processed_by` in the `details` struct
    update.rename_column(("details", "confirmed_by"), "processed_by")
```

### Move column

Di chuyển thứ tự các trường:

```python
with table.update_schema() as update:
    update.move_first("symbol")
    # This will move `bid` after `ask`
    update.move_after("bid", "ask")
    # This will move `confirmed_by` before `exchange` in the `details` struct
    update.move_before(("details", "confirmed_by"), ("details", "exchange"))
```

### Update column

Cập nhật loại trường, mô tả hoặc yêu cầu.

```python
with table.update_schema() as update:
    # Promote a float to a double
    update.update_column("bid", field_type=DoubleType())
    # Make a field optional
    update.update_column("symbol", required=False)
    # Update the documentation
    update.update_column("symbol", doc="Name of the share on the exchange")
```

Hãy cẩn thận, một số thao tác không tương thích nhưng bạn vẫn có thể thực hiện theo rủi ro của riêng mình bằng cách thiết lập `allow_incompatible_changes`:

```python
with table.update_schema(allow_incompatible_changes=True) as update:
    # Incompatible change, cannot require an optional field
    update.update_column("symbol", required=True)
```

### Delete column

Xóa một trường, hãy cẩn thận vì đây là một thay đổi không tương thích (người đọc/người viết có thể mong đợi trường này):

```python
with table.update_schema(allow_incompatible_changes=True) as update:
    update.delete_column("some_field")
    # In a struct
    update.delete_column(("details", "confirmed_by"))
```

## Partition evolution

PyIceberg hỗ trợ tính năng partition evolution. Xem phần [partition evolution](https://iceberg.apache.org/spec/#partition-evolution) để biết thêm chi tiết.

API được sử dụng khi evolving partitions là API `update_spec` trên bảng.

```python
with table.update_spec() as update:
    update.add_field("id", BucketTransform(16), "bucketed_id")
    update.add_field("event_ts", DayTransform(), "day_ts")
```

Việc cập nhật thông số phân vùng cũng có thể được thực hiện như một phần của giao dịch với các hoạt động khác.

```python
with table.transaction() as transaction:
    with transaction.update_spec() as update_spec:
        update_spec.add_field("id", BucketTransform(16), "bucketed_id")
        update_spec.add_field("event_ts", DayTransform(), "day_ts")
    # ... Update properties etc
```

### Add fields

Có thể thêm trường phân vùng mới thông qua API `add_field`, API này sẽ lấy tên trường để phân vùng, phép biến đổi phân vùng và tên phân vùng tùy chọn. Nếu tên phân vùng chưa được chỉ định, một tên phân vùng sẽ được tạo.

```python
with table.update_spec() as update:
    update.add_field("id", BucketTransform(16), "bucketed_id")
    update.add_field("event_ts", DayTransform(), "day_ts")
    # identity is a shortcut API for adding an IdentityTransform
    update.identity("some_field")
```

### Remove fields

Các trường phân vùng cũng có thể được xóa thông qua API `remove_field` nếu việc phân vùng trên các trường đó không còn ý nghĩa nữa.

```python
with table.update_spec() as update:
    # Remove the partition field with the name
    update.remove_field("some_partition_name")
```

### Rename fields

Các trường phân vùng cũng có thể được đổi tên thông qua API `rename_field`.

```python
with table.update_spec() as update:
    # Rename the partition field with the name bucketed_id to sharded_id
    update.rename_field("bucketed_id", "sharded_id")
```

## Sort order updates

Người dùng có thể cập nhật thứ tự sắp xếp trên các bảng hiện có cho dữ liệu mới. Xem mục [sorting](https://iceberg.apache.org/spec/#sorting)  để biết thêm chi tiết.

API được sử dụng khi cập nhật thứ tự sắp xếp là API `update_sort_order` trên bảng.

### Updating a sort order on a table

Để tạo thứ tự sắp xếp mới, bạn có thể sử dụng API `asc` hoặc `desc` tùy thuộc vào việc bạn muốn dữ liệu được sắp xếp theo thứ tự tăng dần hay giảm dần. Cả hai đều sử dụng tên trường, biến đổi thứ tự sắp xếp và một thứ tự null mô tả thứ tự của các giá trị null khi được sắp xếp.

```python
with table.update_sort_order() as update:
    update.desc("event_ts", DayTransform(), NullOrder.NULLS_FIRST)
    update.asc("some_field", IdentityTransform(), NullOrder.NULLS_LAST)
```

## Table properties

Đặt và xóa thuộc tính thông qua `Transaction` API:

```python
with table.transaction() as transaction:
    transaction.set_properties(abc="def")

assert table.properties == {"abc": "def"}

with table.transaction() as transaction:
    transaction.remove_properties("abc")

assert table.properties == {}
```

Hoặc không cần trình quản lý context:

```python
table = table.transaction().set_properties(abc="def").commit_transaction()

assert table.properties == {"abc": "def"}

table = table.transaction().remove_properties("abc").commit_transaction()

assert table.properties == {}
```

## Snapshot properties

Tùy chọn, thuộc tính Snapshot có thể được thiết lập khi ghi vào bảng bằng cách sử dụng API `append` hoặc `overwrite`:

```python
tbl.append(df, snapshot_properties={"abc": "def"})

# or

tbl.overwrite(df, snapshot_properties={"abc": "def"})

assert tbl.metadata.snapshots[-1].summary["abc"] == "def"
```

Bạn cũng có thể sử dụng trình quản lý context để thực hiện thêm nhiều thay đổi:

```python
with table.manage_snapshots() as ms:
    ms.create_branch(snapshot_id1, "Branch_A").create_tag(snapshot_id2, "tag789")
```

### Tags

Tags (thẻ) là các tham chiếu được đặt tên đến các snapshot không thể thay đổi. Chúng có thể được sử dụng để đánh dấu các snapshot quan trọng cần lưu trữ lâu dài hoặc để tham chiếu đến các phiên bản bảng cụ thể.

Tạo thẻ trỏ đến snapshot cụ thể:

```python
# Create a tag with default retention
table.manage_snapshots().create_tag(
    snapshot_id=snapshot_id,
    tag_name="v1.0.0"
).commit()

# Create a tag with custom max reference age
table.manage_snapshots().create_tag(
    snapshot_id=snapshot_id,
    tag_name="v1.0.0",
    max_ref_age_ms=604800000  # 7 days
).commit()
```

Xóa thẻ hiện có:

```python
table.manage_snapshots().remove_tag("v1.0.0").commit()
```

### Branching

Nhánh là các tham chiếu có tên có thể thay đổi đến các snapshot và có thể được cập nhật theo thời gian. Chúng cho phép các dòng dõi độc lập của các thay đổi bảng, cho phép các trường hợp sử dụng như nhánh phát triển, môi trường thử nghiệm hoặc quy trình làm việc song song.

Tạo một nhánh trỏ tới một snapshot cụ thể:

```python
# Create a branch with default settings
table.manage_snapshots().create_branch(
    snapshot_id=snapshot_id,
    branch_name="dev"
).commit()

# Create a branch with retention policies
table.manage_snapshots().create_branch(
    snapshot_id=snapshot_id,
    branch_name="dev",
    max_ref_age_ms=604800000,        # Max age of the branch reference (7 days)
    max_snapshot_age_ms=259200000,   # Max age of snapshots to keep (3 days)
    min_snapshots_to_keep=10         # Minimum number of snapshots to retain
).commit()
```

Xóa một nhánh hiện có:

```python
table.manage_snapshots().remove_branch("dev").commit()
```

## Table Maintenance

PyIceberg cung cấp các hoạt động bảo trì bảng thông qua API `table.maintenance`. API này cung cấp một giao diện rõ ràng để thực hiện các tác vụ bảo trì như expire snapshot.

### Snapshot Expiration

Expire các snapshot cũ để dọn dẹp metadata bảng và giảm chi phí lưu trữ:

```python
# Expire snapshots older than three days
from datetime import datetime, timedelta
table.maintenance.expire_snapshots().older_than(
    datetime.now() - timedelta(days=3)
).commit()

# Expire a specific snapshot by ID
table.maintenance.expire_snapshots().by_id(12345).commit()

# Context manager usage (recommended for multiple operations)
with table.maintenance.expire_snapshots() as expire:
    expire.by_id(12345)
    expire.by_id(67890)
    # Automatically commits when exiting the context
```

#### Real-world Example

```python
def cleanup_old_snapshots(table_name: str, snapshot_ids: list[int]):
    """Remove specific snapshots from a table."""
    catalog = load_catalog("production")
    table = catalog.load_table(table_name)

    # Use context manager for safe transaction handling
    with table.maintenance.expire_snapshots() as expire:
        for snapshot_id in snapshot_ids:
            expire.by_id(snapshot_id)

    print(f"Expired {len(snapshot_ids)} snapshots from {table_name}")

# Usage
cleanup_old_snapshots("analytics.user_events", [12345, 67890, 11111])
```

## Views (skip)

PyIceberg hỗ trợ các thao tác xem.






