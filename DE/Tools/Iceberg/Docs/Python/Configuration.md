
- [[#Setting Configuration Values|Setting Configuration Values]]
- [[#Tables|Tables]]
	- [[#Tables#Write options|Write options]]
	- [[#Tables#Table behavior options|Table behavior options]]
- [[#FileIO|FileIO]]
	- [[#FileIO#S3|S3]]
	- [[#FileIO#HDFS|HDFS]]
	- [[#FileIO#Azure Data lake|Azure Data lake]]
	- [[#FileIO#Google Cloud Storage|Google Cloud Storage]]
	- [[#FileIO#Alibaba Cloud Object Storage Service (OSS) (skip)|Alibaba Cloud Object Storage Service (OSS) (skip)]]
	- [[#FileIO#Hugging Face (skip)|Hugging Face (skip)]]
	- [[#FileIO#PyArrow|PyArrow]]
- [[#Location Providers|Location Providers]]
	- [[#Location Providers#Simple Location Provider|Simple Location Provider]]
	- [[#Location Providers#Object Store Location Provider|Object Store Location Provider]]
		- [[#Object Store Location Provider#Partition Exclusion|Partition Exclusion]]
	- [[#Location Providers#Loading a Custom Location Provider|Loading a Custom Location Provider]]
- [[#Catalogs|Catalogs]]
	- [[#Catalogs#REST Catalog|REST Catalog]]
		- [[#REST Catalog#Headers in REST Catalog|Headers in REST Catalog]]
		- [[#REST Catalog#Authentication Options|Authentication Options]]
			- [[#Authentication Options#Legacy OAuth2|Legacy OAuth2]]
			- [[#Authentication Options#SigV4|SigV4]]
			- [[#Authentication Options#Pluggable Authentication via AuthManager|Pluggable Authentication via AuthManager]]
				- [[#Pluggable Authentication via AuthManager#Supported Authentication Types|Supported Authentication Types]]
				- [[#Pluggable Authentication via AuthManager#Configuration Properties|Configuration Properties]]
				- [[#Pluggable Authentication via AuthManager#Property Reference|Property Reference]]
				- [[#Pluggable Authentication via AuthManager#Examples|Examples]]
				- [[#Pluggable Authentication via AuthManager#Notes|Notes]]
		- [[#REST Catalog#Common Integrations & Examples (skip)|Common Integrations & Examples (skip)]]
	- [[#Catalogs#SQL Catalog|SQL Catalog]]
	- [[#Catalogs#In Memory Catalog|In Memory Catalog]]
	- [[#Catalogs#Hive Catalog|Hive Catalog]]
	- [[#Catalogs#Glue Catalog (skip)|Glue Catalog (skip)]]
	- [[#Catalogs#DynamoDB Catalog (skip)|DynamoDB Catalog (skip)]]
	- [[#Catalogs#Custom Catalog Implementations|Custom Catalog Implementations]]
- [[#Unified AWS Credentials|Unified AWS Credentials]]
- [[#Concurrency|Concurrency]]
- [[#Backward Compatibility|Backward Compatibility]]
- [[#Nanoseconds Support|Nanoseconds Support]]

## Setting Configuration Values

Có ba cách để cấu hình:
- Sử dụng tệp cấu hình `.pyiceberg.yaml` (Khuyến nghị)
- Thông qua các biến môi trường (env)
- Bằng cách truyền thông tin xác thực qua CLI hoặc API Python

Tệp cấu hình có thể được lưu trữ trong thư mục được chỉ định bởi biến môi trường `PYICEBERG_HOME` thư mục gốc hoặc thư mục làm việc hiện tại (theo thứ tự này).

Để thay đổi đường dẫn tìm kiếm cho `.pyiceberg.yaml`, bạn có thể ghi đè lên biến môi trường `PYICEBERG_HOME`.

Một lựa chọn khác là thông qua các biến môi trường:

```env
export PYICEBERG_CATALOG__DEFAULT__URI=thrift://localhost:9083
export PYICEBERG_CATALOG__DEFAULT__S3__ACCESS_KEY_ID=username
export PYICEBERG_CATALOG__DEFAULT__S3__SECRET_ACCESS_KEY=password
```

Biến môi trường được Iceberg chọn bắt đầu bằng `PYICEBERG_` và sau đó theo cấu trúc yaml bên dưới, trong đó dấu gạch dưới kép `__` biểu thị một trường lồng nhau và dấu gạch dưới `_` được chuyển đổi thành dấu gạch ngang `-` .

Ví dụ: `PYICEBERG_CATALOG__DEFAULT__S3__ACCESS_KEY_ID`, đặt `s3.access-key-id` trên `catalog` mặc định.

## Tables

Bảng Iceberg hỗ trợ các thuộc tính bảng để cấu hình hành vi của bảng.

### Write options

|Key|Options|Default|Description|
|---|---|---|---|
|`write.parquet.compression-codec`|`{uncompressed,zstd,gzip,snappy}`|zstd|Sets the Parquet compression coddec.|
|`write.parquet.compression-level`|Integer|null|Parquet compression level for the codec. If not set, it is up to PyIceberg|
|`write.parquet.row-group-limit`|Number of rows|1048576|The upper bound of the number of entries within a single row group|
|`write.parquet.page-size-bytes`|Size in bytes|1MB|Set a target threshold for the approximate encoded size of data pages within a column chunk|
|`write.parquet.page-row-limit`|Number of rows|20000|Set a target threshold for the maximum number of rows within a column chunk|
|`write.parquet.dict-size-bytes`|Size in bytes|2MB|Set the dictionary page size limit per row group|
|`write.metadata.previous-versions-max`|Integer|100|The max number of previous version metadata files to keep before deleting after commit.|
|`write.metadata.delete-after-commit.enabled`|Boolean|False|Whether to automatically delete old _tracked_ metadata files after each table commit. It will retain a number of the most recent metadata files, which can be set using property `write.metadata.previous-versions-max`.|
|`write.object-storage.enabled`|Boolean|False|Enables the [`ObjectStoreLocationProvider`](https://py.iceberg.apache.org/configuration/#object-store-location-provider) that adds a hash component to file paths.|
|`write.object-storage.partitioned-paths`|Boolean|True|Controls whether [partition values are included in file paths](https://py.iceberg.apache.org/configuration/#partition-exclusion) when object storage is enabled|
|`write.py-location-provider.impl`|String of form `module.ClassName`|null|Optional, [custom `LocationProvider`](https://py.iceberg.apache.org/configuration/#loading-a-custom-location-provider) implementation|
|`write.data.path`|String pointing to location|`{metadata.location}/data`|Sets the location under which data is written.|
|`write.metadata.path`|String pointing to location|`{metadata.location}/metadata`|Sets the location under which metadata is written.|

### Table behavior options

|Key|Options|Default|Description|
|---|---|---|---|
|`commit.manifest.target-size-bytes`|Size in bytes|8388608 (8MB)|Target size when merging manifest files|
|`commit.manifest.min-count-to-merge`|Number of manifests|100|Minimum number of manifests to accumulate before merging|
|`commit.manifest-merge.enabled`|Boolean|False|Controls whether to automatically merge manifests on writes|

>[!note] Fast append
>Không giống như triển khai Java, PyIceberg mặc định sử dụng lệnh [fast append](https://py.iceberg.apache.org/api/#write-support) và do đó `commit.manifest-merge.enabled` được đặt thành `False` theo mặc định.

## FileIO

Iceberg hoạt động dựa trên khái niệm FileIO, một mô-đun có thể cắm thêm để đọc, ghi và xóa tệp. Theo mặc định, PyIceberg sẽ thử khởi tạo FileIO phù hợp với schema (`s3://`, `gs://`, v.v.) và sẽ sử dụng FileIO đầu tiên được cài đặt.

- **s3**, **s3a**, **s3n**: `PyArrowFileIO`, `FsspecFileIO`
- **gs**: `PyArrowFileIO`
- **file**: `PyArrowFileIO`
- **hdfs**: `PyArrowFileIO`
- **abfs**, **abfss**: `FsspecFileIO`
- **oss**: `PyArrowFileIO`
- **hf**: `FsspecFileIO`

Bạn cũng có thể thiết lập FileIO một cách rõ ràng:

|Key|Example|Description|
|---|---|---|
|py-io-impl|pyiceberg.io.fsspec.FsspecFileIO|Sets the FileIO explicitly to an implementation, and will fail explicitly if it can't be loaded|

Đối với FileIO, có một số tùy chọn cấu hình có sẵn:

### S3

| Key                         | Example                                              | Description                                                                                                                                                                                                                                                             |
| --------------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| s3.endpoint                 | [https://10.0.19.25/](https://10.0.19.25/)           | Configure an alternative endpoint of the S3 service for the FileIO to access. This could be used to use S3FileIO with any s3-compatible object storage service that has a different endpoint, or access a private S3 endpoint in a virtual private cloud.               |
| s3.access-key-id            | admin                                                | Configure the static access key id used to access the FileIO.                                                                                                                                                                                                           |
| s3.secret-access-key        | password                                             | Configure the static secret access key used to access the FileIO.                                                                                                                                                                                                       |
| s3.session-token            | AQoDYXdzEJr...                                       | Configure the static session token used to access the FileIO.                                                                                                                                                                                                           |
| s3.role-session-name        | session                                              | An optional identifier for the assumed role session.                                                                                                                                                                                                                    |
| s3.role-arn                 | arn:aws:...                                          | AWS Role ARN. If provided instead of access_key and secret_key, temporary credentials will be fetched by assuming this role.                                                                                                                                            |
| s3.signer                   | bearer                                               | Configure the signature version of the FileIO.                                                                                                                                                                                                                          |
| s3.signer.uri               | [http://my.signer:8080/s3](http://my.signer:8080/s3) | Configure the remote signing uri if it differs from the catalog uri. Remote signing is only implemented for `FsspecFileIO`. The final request is sent to `<s3.signer.uri>/<s3.signer.endpoint>`.                                                                        |
| s3.signer.endpoint          | v1/main/s3-sign                                      | Configure the remote signing endpoint. Remote signing is only implemented for `FsspecFileIO`. The final request is sent to `<s3.signer.uri>/<s3.signer.endpoint>`. (default : v1/aws/s3/sign).                                                                          |
| s3.region                   | us-west-2                                            | Configure the default region used to initialize an `S3FileSystem`. `PyArrowFileIO` attempts to automatically tries to resolve the region if this isn't set (only supported for AWS S3 Buckets).                                                                         |
| s3.resolve-region           | False                                                | Only supported for `PyArrowFileIO`, when enabled, it will always try to resolve the location of the bucket (only supported for AWS S3 Buckets).                                                                                                                         |
| s3.proxy-uri                | [http://my.proxy.com:8080](http://my.proxy.com:8080) | Configure the proxy server to be used by the FileIO.                                                                                                                                                                                                                    |
| s3.connect-timeout          | 60.0                                                 | Configure socket connection timeout, in seconds.                                                                                                                                                                                                                        |
| s3.request-timeout          | 60.0                                                 | Configure socket read timeouts on Windows and macOS, in seconds.                                                                                                                                                                                                        |
| s3.force-virtual-addressing | False                                                | Whether to use virtual addressing of buckets. If true, then virtual addressing is always enabled. If false, then virtual addressing is only enabled if endpoint_override is empty. This can be used for non-AWS backends that only support virtual hosted-style access. |
| s3.retry-strategy-impl      | None                                                 | Ability to set a custom S3 retry strategy. A full path to a class needs to be given that extends the [S3RetryStrategy](https://github.com/apache/arrow/blob/639201bfa412db26ce45e73851432018af6c945e/python/pyarrow/_s3fs.pyx#L110) base class.                         |
| s3.anonymous                | True                                                 | Configure whether to use anonymous connection. If False (default), uses key/secret if configured or boto's credential resolver.                                                                                                                                         |

### HDFS

|Key|Example|Description|
|---|---|---|
|hdfs.host|[https://10.0.19.25/](https://10.0.19.25/)|Configure the HDFS host to connect to|
|hdfs.port|9000|Configure the HDFS port to connect to.|
|hdfs.user|user|Configure the HDFS username used for connection.|
|hdfs.kerberos_ticket|kerberos_ticket|Configure the path to the Kerberos ticket cache.|
### Azure Data lake

|Key|Example|Description|
|---|---|---|
|adls.connection-string|AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqF...;BlobEndpoint=[http://localhost/](http://localhost/)|A [connection string](https://learn.microsoft.com/en-us/azure/storage/common/storage-configure-connection-string). This could be used to use FileIO with any adls-compatible object storage service that has a different endpoint (like [azurite](https://github.com/azure/azurite)).|
|adls.account-name|devstoreaccount1|The account that you want to connect to|
|adls.account-key|Eby8vdM02xNOcqF...|The key to authentication against the account.|
|adls.sas-token|NuHOuuzdQN7VRM%2FOpOeqBlawRCA845IY05h9eu1Yte4%3D|The shared access signature|
|adls.tenant-id|ad667be4-b811-11ed-afa1-0242ac120002|The tenant-id|
|adls.client-id|ad667be4-b811-11ed-afa1-0242ac120002|The client-id|
|adls.client-secret|oCA3R6P*ka#oa1Sms2J74z...|The client-secret|
|adls.account-host|accountname1.blob.core.windows.net|The storage account host. See [AzureBlobFileSystem](https://github.com/fsspec/adlfs/blob/adb9c53b74a0d420625b86dd00fbe615b43201d2/adlfs/spec.py#L125) for reference|
|adls.blob-storage-authority|.blob.core.windows.net|The hostname[:port] of the Blob Service. Defaults to `.blob.core.windows.net`. Useful for connecting to a local emulator, like [azurite](https://github.com/azure/azurite). See [AzureFileSystem](https://arrow.apache.org/docs/python/filesystems.html#azure-storage-file-system) for reference|
|adls.dfs-storage-authority|.dfs.core.windows.net|The hostname[:port] of the Data Lake Gen 2 Service. Defaults to `.dfs.core.windows.net`. Useful for connecting to a local emulator, like [azurite](https://github.com/azure/azurite). See [AzureFileSystem](https://arrow.apache.org/docs/python/filesystems.html#azure-storage-file-system) for reference|
|adls.blob-storage-scheme|https|Either `http` or `https`. Defaults to `https`. Useful for connecting to a local emulator, like [azurite](https://github.com/azure/azurite). See [AzureFileSystem](https://arrow.apache.org/docs/python/filesystems.html#azure-storage-file-system) for reference|
|adls.dfs-storage-scheme|https|Either `http` or `https`. Defaults to `https`. Useful for connecting to a local emulator, like [azurite](https://github.com/azure/azurite). See [AzureFileSystem](https://arrow.apache.org/docs/python/filesystems.html#azure-storage-file-system) for reference|
|adls.token|eyJ0eXAiOiJKV1QiLCJhbGci...|Static access token for authenticating with ADLS. Used for OAuth2 flows.|

### Google Cloud Storage

|Key|Example|Description|
|---|---|---|
|gcs.project-id|my-gcp-project|Configure Google Cloud Project for GCS FileIO.|
|gcs.oauth2.token|ya29.dr.AfM...|String representation of the access token used for temporary access.|
|gcs.oauth2.token-expires-at|1690971805918|Configure expiration for credential generated with an access token. Milliseconds since epoch|
|gcs.access|read_only|Configure client to have specific access. Must be one of 'read_only', 'read_write', or 'full_control'|
|gcs.consistency|md5|Configure the check method when writing files. Must be one of 'none', 'size', or 'md5'|
|gcs.cache-timeout|60|Configure the cache expiration time in seconds for object metadata cache|
|gcs.requester-pays|False|Configure whether to use requester-pays requests|
|gcs.session-kwargs|{}|Configure a dict of parameters to pass on to aiohttp.ClientSession; can contain, for example, proxy settings.|
|gcs.service.host|[http://0.0.0.0:4443](http://0.0.0.0:4443)|Configure an alternative endpoint for the GCS FileIO to access (format protocol://host:port) If not given, defaults to the value of environment variable "STORAGE_EMULATOR_HOST"; if that is not set either, will use the standard Google endpoint.|
|gcs.default-location|US|Configure the default location where buckets are created, like 'US' or 'EUROPE-WEST3'.|
|gcs.version-aware|False|Configure whether to support object versioning on the GCS bucket.|

### Alibaba Cloud Object Storage Service (OSS) (skip)

### Hugging Face (skip)

### PyArrow

|Key|Example|Description|
|---|---|---|
|pyarrow.use-large-types-on-read|True|Use large PyArrow types i.e. [large_string](https://arrow.apache.org/docs/python/generated/pyarrow.large_string.html), [large_binary](https://arrow.apache.org/docs/python/generated/pyarrow.large_binary.html) and [large_list](https://arrow.apache.org/docs/python/generated/pyarrow.large_list.html) field types on table scans. The default value is True.|
## Location Providers

Apache Iceberg sử dụng khái niệm `LocationProvider` để quản lý đường dẫn tệp cho các tệp dữ liệu của bảng. Trong PyIceberg, mô-đun `LocationProvider` được thiết kế để có thể cắm được, cho phép tùy chỉnh cho các trường hợp sử dụng cụ thể và xác định thêm vị trí tệp metadata. `LocationProvider` cho một bảng có thể được chỉ định thông qua thuộc tính bảng.

Cả vị trí tệp dữ liệu và tệp metadata đều có thể được tùy chỉnh bằng cách cấu hình các thuộc tính bảng `write.data.path` và `write.metadata.path` tương ứng.

Để kiểm soát chi tiết hơn, bạn có thể ghi đè các phương thức `new_data_location` và `new_metadata_location` của `LocationProvider` để xác định logic tùy chỉnh cho việc tạo đường dẫn tệp. Xem [`Loading a Custom Location Provider`](https://py.iceberg.apache.org/configuration/#loading-a-custom-location-provider).

PyIceberg mặc định sử dụng `SimpleLocationProvider` để quản lý đường dẫn tệp.

### Simple Location Provider

`SimpleLocationProvider` cung cấp các đường dẫn có tiền tố là `{location}/data/`, trong đó `location` lấy từ [table metadata](https://iceberg.apache.org/spec/#table-metadata-fields). Điều này có thể được ghi đè bằng cách thiết lập [`write.data.path` table configuration](https://py.iceberg.apache.org/configuration/#write-options) . 

Ví dụ, một bảng không phân vùng có thể có tệp dữ liệu với vị trí:

```bash
s3://bucket/ns/table/data/0000-0-5affc076-96a4-48f2-9cd2-d5efbc9f0c94-00001.parquet
```

Khi bảng được phân vùng, các tệp trong một phân vùng nhất định sẽ được nhóm vào một thư mục con, với khóa và giá trị phân vùng đó là tên thư mục - đây được gọi là định dạng đường dẫn phân vùng kiểu Hive. Ví dụ: một bảng được phân vùng theo `category` cột chuỗi có thể có tệp dữ liệu với vị trí:

```bash
s3://bucket/ns/table/data/category=orders/0000-0-5affc076-96a4-48f2-9cd2-d5efbc9f0c94-00001.parquet
```

### Object Store Location Provider

PyIceberg cung cấp `ObjectStoreLocationProvider` và tùy chọn tối ưu hóa [partition-exclusion](https://py.iceberg.apache.org/configuration/#partition-exclusion), được thiết kế cho các bảng được lưu trữ trong bộ nhớ đối tượng. Để biết thêm bối cảnh và động lực liên quan đến các cấu hình này, hãy xem tài liệu của họ về triển khai Java của Iceberg ([documentation for Iceberg's Java implementation](https://iceberg.apache.org/docs/latest/aws/#object-store-file-layout)).

Khi nhiều tệp được lưu trữ dưới cùng một tiền tố (prefix), các kho lưu trữ đối tượng đám mây như S3 thường hạn chế các yêu cầu trên tiền tố ([throttle requests on prefixes](https://repost.aws/knowledge-center/http-5xx-errors-s3)), dẫn đến chậm tải. `ObjectStoreLocationProvider` khắc phục điều này bằng cách chèn các hàm băm xác định, dưới dạng thư mục nhị phân, vào đường dẫn tệp, để phân phối tệp trên nhiều tiền tố kho lưu trữ đối tượng hơn.

Đường dẫn được thêm tiền tố `{location}/data/` trong đó `location` lấy từ [table metadata](https://iceberg.apache.org/spec/#table-metadata-fields), tương tự như [`SimpleLocationProvider`](https://py.iceberg.apache.org/configuration/#simple-location-provider). Điều này có thể được ghi đè bằng cách thiết lập [`write.data.path` table configuration](https://py.iceberg.apache.org/configuration/#write-options).

Ví dụ, một bảng được phân vùng theo `category` cột chuỗi có thể có tệp dữ liệu với vị trí: (lưu ý các thư mục nhị phân bổ sung)

```bash
s3://bucket/ns/table/data/0101/0110/1001/10110010/category=orders/0000-0-5affc076-96a4-48f2-9cd2-d5efbc9f0c94-00001.parquet
```

`ObjectStoreLocationProvider` được bật cho một bảng bằng cách thiết lập rõ ràng thuộc tính bảng `write.object-storage.enabled` thành `True`.

#### Partition Exclusion

Khi sử dụng `ObjectStoreLocationProvider`, thuộc tính bảng `write.object-storage.partitioned-paths`, mặc định là `True`, có thể được đặt thành `False` như một tối ưu hóa bổ sung cho kho lưu trữ đối tượng. Thao tác này loại bỏ hoàn toàn các khóa và giá trị phân vùng khỏi đường dẫn tệp dữ liệu để giảm thêm kích thước khóa. Khi tắt thuộc tính này, cùng tệp dữ liệu trên sẽ được ghi vào: (lưu ý không có thuộc tính `category=orders`)

```bash
s3://bucket/ns/table/data/1101/0100/1011/00111010-00000-0-5affc076-96a4-48f2-9cd2-d5efbc9f0c94-00001.parquet
```

### Loading a Custom Location Provider

Tương tự như FileIO, `LocationProvider` tùy chỉnh có thể được cung cấp cho một bảng bằng cách phân lớp cụ thể lớp cơ sở trừu tượng [`LocationProvider`](https://py.iceberg.apache.org/reference/pyiceberg/table/locations/#pyiceberg.table.locations.LocationProvider).

Thuộc tính bảng `write.py-location-provider.impl` phải được đặt thành tên đầy đủ của `LocationProvider` tùy chỉnh (tức là `mymodule.MyLocationProvider`). Lưu ý rằng `LocationProvider` được cấu hình cho từng bảng, cho phép cung cấp vị trí khác nhau cho các bảng khác nhau. Cũng cần lưu ý rằng triển khai Java của Iceberg sử dụng một thuộc tính bảng khác, `write.location-provider.impl`, cho các triển khai Java tùy chỉnh.

Ví dụ về triển khai `LocationProvider` tùy chỉnh được hiển thị bên dưới.

```python
import uuid

class UUIDLocationProvider(LocationProvider):
    def __init__(self, table_location: str, table_properties: Properties):
        super().__init__(table_location, table_properties)

    def new_data_location(self, data_file_name: str, partition_key: Optional[PartitionKey] = None) -> str:
        # Can use any custom method to generate a file path given the partitioning information and file name
        prefix = f"{self.table_location}/{uuid.uuid4()}"
        return f"{prefix}/{partition_key.to_path()}/{data_file_name}" if partition_key else f"{prefix}/{data_file_name}"
```

## Catalogs

PyIceberg hiện hỗ trợ kiểu catalog gốc cho REST, SQL, Hive, Glue và DynamoDB. Ngoài ra, bạn cũng có thể thiết lập trực tiếp việc triển khai catalog:

|Key|Example|Description|
|---|---|---|
|type|rest|Type of catalog, one of `rest`, `sql`, `hive`, `glue`, `dymamodb`. Default to `rest`|
|py-catalog-impl|mypackage.mymodule.MyCatalog|Sets the catalog explicitly to an implementation, and will fail explicitly if it can't be loaded|

### REST Catalog

```yaml
catalog:
  default:
    uri: http://rest-catalog/ws/
    credential: t-1234:secret

  default-mtls-secured-catalog:
    uri: https://rest-catalog/ws/
    ssl:
      client:
        cert: /absolute/path/to/client.crt
        key: /absolute/path/to/client.key
      cabundle: /absolute/path/to/cabundle.pem
```

|Key|Example|Description|
|---|---|---|
|uri|[https://rest-catalog/ws](https://rest-catalog/ws)|URI identifying the REST Server|
|warehouse|myWarehouse|Warehouse location or identifier to request from the catalog service. May be used to determine server-side overrides, such as the warehouse location.|
|snapshot-loading-mode|refs|The snapshots to return in the body of the metadata. Setting the value to `all` would return the full set of snapshots currently valid for the table. Setting the value to `refs` would load all snapshots referenced by branches or tags.|
|`header.X-Iceberg-Access-Delegation`|`vended-credentials`|Signal to the server that the client supports delegated access via a comma-separated list of access mechanisms. The server may choose to supply access via any or none of the requested mechanisms. When using `vended-credentials`, the server provides temporary credentials to the client. When using `remote-signing`, the server signs requests on behalf of the client. (default: `vended-credentials`)|

#### Headers in REST Catalog

Để cấu hình tiêu đề tùy chỉnh trong REST Catalog, hãy thêm chúng vào thuộc tính catalog bằng `header.<Header-Name>`. Điều này đảm bảo rằng tất cả các yêu cầu HTTP đến dịch vụ REST đều bao gồm các tiêu đề đã chỉ định.

```yaml
catalog:
  default:
    uri: http://rest-catalog/ws/
    credential: t-1234:secret
    header.content-type: application/vnd.api+json
```

#### Authentication Options

##### Legacy OAuth2

Thuộc tính OAuth2 cũ sẽ bị xóa trong PyIceberg 1.0 thay thế cho các thuộc tính AuthManager có thể cài được bên dưới

|Key|Example|Description|
|---|---|---|
|oauth2-server-uri|[https://auth-service/cc](https://auth-service/cc)|Authentication URL to use for client credentials authentication (default: uri + 'v1/oauth/tokens')|
|token|FEW23.DFSDF.FSDF|Bearer token value to use for `Authorization` header|
|credential|client_id:client_secret|Credential to use for OAuth2 credential flow when initializing the catalog|
|scope|openid offline corpds:ds:profile|Desired scope of the requested security token (default : catalog)|
|resource|rest_catalog.iceberg.com|URI for the target resource or service|
|audience|rest_catalog|Logical name of target resource or service|
##### SigV4

|Key|Example|Description|
|---|---|---|
|rest.sigv4-enabled|true|Sign requests to the REST Server using AWS SigV4 protocol|
|rest.signing-region|us-east-1|The region to use when SigV4 signing a request|
|rest.signing-name|execute-api|The service signing name to use when SigV4 signing a request|

##### Pluggable Authentication via AuthManager

RESTCatalog hỗ trợ xác thực dạng pluggable thông qua khối cấu hình `auth`. Điều này cho phép bạn chỉ định cách mã thông báo truy cập sẽ được lấy và quản lý để sử dụng với các yêu cầu HTTP đến máy chủ RESTCatalog. Phương thức xác thực được chọn bằng cách thiết lập thuộc tính `auth.type` và cấu hình bổ sung có thể được cung cấp khi cần thiết cho từng phương thức.

###### Supported Authentication Types

- `noop`: No authentication (no Authorization header sent).
- `basic`: HTTP Basic authentication.
- `oauth2`: OAuth2 client credentials flow.
- `custom`: Custom authentication manager (requires `auth.impl`).
- `google`: Google Authentication support

###### Configuration Properties

`Auth` được cấu trúc như sau:

```yaml
catalog:
  default:
    type: rest
    uri: http://rest-catalog/ws/
    auth:
      type: <auth_type>
      <auth_type>:
        # Type-specific configuration
      impl: <custom_class_path>  # Only for custom auth
```

###### Property Reference

|Property|Required|Description|
|---|---|---|
|`auth.type`|Yes|The authentication type to use (`noop`, `basic`, `oauth2`, or `custom`).|
|`auth.impl`|Conditionally|The fully qualified class path for a custom AuthManager. Required if `auth.type` is `custom`.|
|`auth.basic`|If type is `basic`|Block containing `username` and `password` for HTTP Basic authentication.|
|`auth.oauth2`|If type is `oauth2`|Block containing OAuth2 configuration (see below).|
|`auth.custom`|If type is `custom`|Block containing configuration for the custom AuthManager.|
|`auth.google`|If type is `google`|Block containing `credentials_path` to a service account file (if using). Will default to using Application Default Credentials.|
###### Examples

No Authentication:

```yaml
auth:
  type: noop
```

Basic Authentication:

```yaml
auth:
  type: basic
  basic:
    username: myuser
    password: mypass
```

OAuth2 Authentication:

```yaml
auth:
  type: oauth2
  oauth2:
    client_id: my-client-id
    client_secret: my-client-secret
    token_url: https://auth.example.com/oauth/token
    scope: read
    refresh_margin: 60         # (optional) seconds before expiry to refresh
    expires_in: 3600           # (optional) fallback if server does not provide
```

Custom Authentication:

```yaml
auth:
  type: custom
  impl: mypackage.module.MyAuthManager
  custom:
    property1: value1
    property2: value2
```

###### Notes

- Nếu `auth.type` là `custom`, bạn phải chỉ định auth.impl với đường dẫn lớp đầy đủ tới AuthManager tùy chỉnh của bạn. 
- Nếu `auth.type` không phải là `custom` thì không được phép chỉ định `auth.impl`. 
- Khối cấu hình trong mỗi loại (ví dụ: `basic`, `oauth2`, `custom`) được truyền dưới dạng đối số từ khóa tới AuthManager tương ứng.

#### Common Integrations & Examples (skip)

### SQL Catalog

SQL catalog yêu cầu một cơ sở dữ liệu cho phần backend của nó. PyIceberg hỗ trợ PostgreSQL và SQLite thông qua psycopg2. Kết nối cơ sở dữ liệu phải được cấu hình bằng thuộc tính `uri`. Thuộc tính init_catalog_tables là tùy chọn và mặc định là True. Nếu được đặt thành False, các table catalog sẽ không được tạo khi SQLCatalog được khởi tạo. Xem tài liệu hướng dẫn của SQLAlchemy để biết định dạng URL([documentation for URL format](https://docs.sqlalchemy.org/en/20/core/engines.html#backend-specific-urls)):

Đối với PostgreSQL:

```yaml
catalog:
  default:
    type: sql
    uri: postgresql+psycopg2://username:password@localhost/mydatabase
    init_catalog_tables: false
```

Trong trường hợp của SQLite:

```yaml
catalog:
  default:
    type: sql
    uri: sqlite:////tmp/pyiceberg.db
    init_catalog_tables: false
```

|Key|Example|Default|Description|
|---|---|---|---|
|uri|postgresql+psycopg2://username:password@localhost/mydatabase||SQLAlchemy backend URL for the catalog database (see [documentation for URL format](https://docs.sqlalchemy.org/en/20/core/engines.html#backend-specific-urls))|
|echo|true|false|SQLAlchemy engine [echo param](https://docs.sqlalchemy.org/en/20/core/engines.html#sqlalchemy.create_engine.params.echo) to log all statements to the default log handler|
|pool_pre_ping|true|false|SQLAlchemy engine [pool_pre_ping param](https://docs.sqlalchemy.org/en/20/core/engines.html#sqlalchemy.create_engine.params.pool_pre_ping) to test connections for liveness upon each checkout|

>[!warning] Development only
>SQLite không được xây dựng để xử lý đồng thời, bạn nên sử dụng catalog này cho mục đích khám phá hoặc phát triển.

### In Memory Catalog

in-memory catalog được xây dựng dựa trên SqlCatalog và sử dụng cơ sở dữ liệu trong bộ nhớ SQLite làm phần backend.

Nó hữu ích cho thử nghiệm, demo và sân chơi nhưng không hữu ích trong sản xuất vì không hỗ trợ truy cập đồng thời.

```yaml
catalog:
  default:
    type: in-memory
    warehouse: /tmp/pyiceberg/warehouse
```

|Key|Example|Default|Description|
|---|---|---|---|
|warehouse|/tmp/pyiceberg/warehouse|file:///tmp/iceberg/warehouse|The directory where the in-memory catalog will store its data files.|
### Hive Catalog

```yaml
catalog:
  default:
    uri: thrift://localhost:9083
    s3.endpoint: http://localhost:9000
    s3.access-key-id: admin
    s3.secret-access-key: password
```

|Key|Example|Description|
|---|---|---|
|hive.hive2-compatible|true|Using Hive 2.x compatibility mode|
|hive.kerberos-authentication|true|Using authentication via Kerberos|
|hive.kerberos-service-name|hive|Kerberos service name (default hive)|
|ugi|t-1234:secret|Hadoop UGI for Hive client.|

Khi sử dụng Hive 2.x, hãy đảm bảo đặt cờ (flag) tương thích:

```yaml
catalog:
  default:
...
    hive.hive2-compatible: true
```

### Glue Catalog (skip)

### DynamoDB Catalog (skip)

### Custom Catalog Implementations 

Nếu bạn muốn tải bất kỳ triển khai catalog tùy chỉnh nào, bạn có thể thiết lập cấu hình catalog như sau:

```yaml
catalog:
  default:
    py-catalog-impl: mypackage.mymodule.MyCatalog
    custom-key1: value1
    custom-key2: value2
```

## Unified AWS Credentials

Bạn có thể thiết lập rõ ràng thông tin xác thực AWS cho cả Glue/DynamoDB Catalog và S3 FileIO bằng cách cấu hình thuộc tính `client.*`. Ví dụ:

```yaml
catalog:
  default:
    type: glue
    client.access-key-id: <ACCESS_KEY_ID>
    client.secret-access-key: <SECRET_ACCESS_KEY>
    client.region: <REGION_NAME>
```

cấu hình thông tin xác thực AWS cho cả Glue Catalog và S3 FileIO.

| Key                      | Example        | Description                                                                                                                  |
| ------------------------ | -------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| client.region            | us-east-1      | Set the region of both the Glue/DynamoDB Catalog and the S3 FileIO                                                           |
| client.access-key-id     | admin          | Configure the static access key id used to access both the Glue/DynamoDB Catalog and the S3 FileIO                           |
| client.secret-access-key | password       | Configure the static secret access key used to access both the Glue/DynamoDB Catalog and the S3 FileIO                       |
| client.session-token     | AQoDYXdzEJr... | Configure the static session token used to access both the Glue/DynamoDB Catalog and the S3 FileIO                           |
| client.role-session-name | session        | An optional identifier for the assumed role session.                                                                         |
| client.role-arn          | arn:aws:...    | AWS Role ARN. If provided instead of access_key and secret_key, temporary credentials will be fetched by assuming this role. |
>[!note] Properties Priority
>Các thuộc tính `client.*` sẽ bị ghi đè bởi các thuộc tính dành riêng cho dịch vụ nếu chúng được thiết lập. Ví dụ: nếu `client.region` được đặt thành `us-west-1` và `s3.region` được đặt thành `us-east-1`, S3 FileIO sẽ sử dụng `us-east-1` làm vùng.

## Concurrency

PyIceberg sử dụng nhiều luồng để song song hóa (parallelize) các hoạt động. Số lượng worker có thể được cấu hình bằng cách cung cấp mục `max-workers` trong tệp cấu hình hoặc bằng cách thiết lập biến môi trường `PYICEBERG_MAX_WORKERS`. Giá trị mặc định phụ thuộc vào phần cứng hệ thống và phiên bản Python. Xem [the Python documentation](https://docs.python.org/3/library/concurrent.futures.html#threadpoolexecutor) để biết thêm chi tiết.

## Backward Compatibility

Các phiên bản Java trước (<1.4.0) triển khai nhầm lẫn thuộc tính tùy chọn `current-snapshot-id` là thuộc tính bắt buộc trong TableMetadata. Điều này có nghĩa là nếu `current-snapshot-id` bị thiếu trong tệp metadata (ví dụ: khi tạo bảng), ứng dụng sẽ ném ra ngoại lệ mà không thể tải bảng. Giả định này đã được sửa trong các phiên bản Iceberg gần đây hơn. Tuy nhiên, có thể buộc PyIceberg tạo một bảng với tệp metadata tương thích với các phiên bản trước. Điều này có thể được cấu hình bằng cách đặt thuộc tính `legacy-current-snapshot-id` thành "True" trong tệp cấu hình, hoặc bằng cách đặt biến môi trường `PYICEBERG_LEGACY_CURRENT_SNAPSHOT_ID`. Tham khảo [PR discussion](https://github.com/apache/iceberg-python/pull/473) để biết thêm chi tiết về vấn đề này.

## Nanoseconds Support

PyIceberg hiện chỉ hỗ trợ độ chính xác lên đến micro giây trong TimestampType. Các kiểu dấu thời gian PyArrow trong 's' và 'ms' sẽ tự động được upcast thành dấu thời gian có độ chính xác 'us' khi ghi. Dấu thời gian có độ chính xác 'ns' cũng có thể được downcast tự động khi ghi nếu muốn. Điều này có thể được cấu hình bằng cách đặt thuộc tính `downcast-ns-timestamp-to-us-on-write` thành "True" trong tệp cấu hình, hoặc bằng cách đặt biến môi trường `PYICEBERG_DOWNCAST_NS_TIMESTAMP_TO_US_ON_WRITE`. Tham khảo tài liệu [nanoseconds timestamp proposal document](https://docs.google.com/document/d/1bE1DcEGNzZAMiVJSZ0X1wElKLNkT9kRkk0hDlfkXzvU/edit#heading=h.ibflcctc9i1d) để biết thêm chi tiết về lộ trình dài hạn cho việc hỗ trợ nano giây.











