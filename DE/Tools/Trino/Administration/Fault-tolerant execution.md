- [[#Configuration|Configuration]]
- [[#Retry policy|Retry policy]]
	- [[#Retry policy#QUERY|QUERY]]
	- [[#Retry policy#TASK|TASK]]
- [[#Encryption|Encryption]]
- [[#Advanced configuration|Advanced configuration]]
	- [[#Advanced configuration#Retry limits|Retry limits]]
	- [[#Advanced configuration#Task sizing|Task sizing]]
	- [[#Advanced configuration#Node allocation|Node allocation]]
	- [[#Advanced configuration#Other tuning|Other tuning]]
- [[#Exchange manager|Exchange manager]]
	- [[#Exchange manager#Configuration|Configuration]]
		- [[#Configuration#AWS S3|AWS S3]]
		- [[#Configuration#Azure Blob Storage|Azure Blob Storage]]
		- [[#Configuration#Google Cloud Storage|Google Cloud Storage]]
		- [[#Configuration#HDFS|HDFS]]
		- [[#Configuration#Local filesystem storage|Local filesystem storage]]
- [[#Adaptive plan optimizations|Adaptive plan optimizations]]

Theo mặc định, nếu một nút Trino thiếu tài nguyên để thực hiện một tác vụ hoặc gặp lỗi trong quá trình thực thi truy vấn, truy vấn sẽ thất bại và phải được chạy lại thủ công. Thời gian chạy của truy vấn càng dài, khả năng xảy ra lỗi như vậy càng cao.

Chế độ thực thi chịu lỗi là một cơ chế trong Trino cho phép cụm máy chủ giảm thiểu lỗi truy vấn bằng cách thử lại các query hoặc các thành phần task của chúng trong trường hợp xảy ra lỗi. Khi chế độ thực thi chịu lỗi được bật, dữ liệu trao đổi trung gian được lưu trữ tạm thời và có thể được sử dụng lại bởi một máy chủ khác trong trường hợp máy chủ đó ngừng hoạt động hoặc xảy ra lỗi khác trong quá trình thực thi truy vấn.

>[!note]
>Khả năng chịu lỗi không áp dụng cho các truy vấn bị lỗi hoặc các lỗi khác do người dùng gây ra. Ví dụ, Trino không tốn tài nguyên để thử lại một truy vấn bị lỗi vì không thể phân tích cú pháp câu lệnh SQL của nó.
>Để có hướng dẫn từng bước giải thích cách cấu hình cụm Trino với khả năng thực thi chịu lỗi nhằm cải thiện khả năng phục hồi xử lý truy vấn, hãy đọc bài viết [Improve query processing resilience](https://trino.io/docs/current/installation/query-resiliency.html).

## Configuration

Chế độ thực thi chịu lỗi được tắt theo mặc định. Để bật tính năng này, hãy đặt thuộc tính cấu hình `retry-policy` thành `QUERY` hoặc `TASK` tùy thuộc vào chính sách thử lại mong muốn.

```
retry-policy=QUERY
```

>[!warning]
>Việc thiết lập `retry-policy` có thể khiến các truy vấn thất bại với các connector không hỗ trợ rõ ràng việc thực thi chịu lỗi, dẫn đến thông báo lỗi “This connector does not support query retries”.
>Khả năng hỗ trợ thực thi câu lệnh SQL chịu lỗi khác nhau tùy thuộc vào từng connector, với thông tin chi tiết hơn trong tài liệu của mỗi connector. Các connector sau hỗ trợ thực thi chịu lỗi

Các thuộc tính cấu hình sau đây kiểm soát hành vi thực thi chịu lỗi trên cụm Trino:

            Fault tolerance retry limit configuration properties

| Property name                                          | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Default value           |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| `retry-policy`                                         | Configures what is retried in the event of failure, either `QUERY` to retry the whole query, or `TASK` to retry tasks individually if they fail. See [retry policy](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-retry-policy) for more information. Use the equivalent session property `retry_policy` only on clusters configured for fault-tolerant execution and typically only to deactivate with `NONE`, since switching between modes on a cluster is not tested.                                                                                                          | `NONE`                  |
| `retry-policy.allowed`                                 | List of retry policies that are allowed to be configured for a cluster. This property is used to prevent a user from configuring a retry policy that is not meant to be used on the given cluster.                                                                                                                                                                                                                                                                                                                                                                                                         | `NONE`, `QUERY`, `TASK` |
| `exchange.deduplication-buffer-size`                   | [Data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) of the coordinator’s in-memory buffer used by fault-tolerant execution to store output of query [stages](https://trino.io/docs/current/overview/concepts.html#trino-concept-stage). If this buffer is filled(đầy) during query execution, the query fails with a “Exchange manager must be configured for the failure recovery capabilities to be fully functional” error message unless an [exchange manager](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-exchange-manager) is configured. | `32MB`                  |
| `fault-tolerant-execution.exchange-encryption-enabled` | Enable encryption of spooling data, see [Encryption](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-encryption) for details. Setting this property to false is not recommended if Trino processes sensitive data.                                                                                                                                                                                                                                                                                                                                                                   | `true`                  |
Bạn có thể tìm thêm các thuộc tính liên quan trong phần [Properties reference](https://trino.io/docs/current/admin/properties.html), cụ thể là trong các thuộc tính [Resource management properties](https://trino.io/docs/current/admin/properties-resource-management.html) và [Exchange properties](https://trino.io/docs/current/admin/properties-exchange.html).

## Retry policy

Thuộc tính cấu hình `retry-policy`, hay thuộc tính phiên `retry_policy`, chỉ định xem Trino có thử lại toàn bộ truy vấn hay chỉ thử lại từng task riêng lẻ của truy vấn trong trường hợp thất bại hay không.

### QUERY

`QUERY` retry policy hướng dẫn Trino tự động thử lại truy vấn trong trường hợp xảy ra lỗi trên một node worker. `QUERY` retry policy được khuyến nghị khi phần lớn khối lượng công việc của cụm Trino bao gồm nhiều truy vấn nhỏ.

Theo mặc định, Trino không triển khai khả năng chịu lỗi cho các truy vấn có tập kết quả vượt quá `32MB`, chẳng hạn như các câu lệnh `SELECT` trả về một tập dữ liệu rất lớn cho người dùng. Giới hạn này có thể được tăng lên bằng cách sửa đổi thuộc tính cấu hình `exchange.deduplication-buffer-size` thành giá trị lớn hơn giá trị mặc định là `32MB`, nhưng điều này sẽ dẫn đến việc sử dụng bộ nhớ cao hơn trên coordinator.

Để đảm bảo khả năng thực thi chịu lỗi đối với các truy vấn có tập kết quả lớn hơn, rất nên cấu hình [exchange manager](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-exchange-manager)  sử dụng bộ nhớ ngoài để lưu trữ dữ liệu tạm thời và do đó cho phép lưu trữ dữ liệu bị tràn vượt quá kích thước in-memory buffer size.

### TASK

 `TASK` retry policy hướng dẫn Trino thử lại từng tasks truy vấn riêng lẻ trong trường hợp xảy ra lỗi. Bạn phải cấu hình [exchange manager](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-exchange-manager) để sử dụng task retry policy này. Chính sách này được khuyến nghị khi thực hiện các truy vấn theo batch lớn, vì cụm máy chủ có thể thử lại các task nhỏ hơn trong truy vấn hiệu quả hơn so với việc thử lại toàn bộ truy vấn.

Khi một cụm được cấu hình với `TASK` retry policy, một số thuộc tính cấu hình liên quan sẽ được thay đổi giá trị mặc định để tuân theo các thực tiễn tốt nhất cho một cụm chịu lỗi. Tuy nhiên, thay đổi tự động này không ảnh hưởng đến các cụm đã cấu hình thủ công các thuộc tính này. Nếu bạn đã cấu hình bất kỳ thuộc tính nào sau đây trong tệp `config.properties` trên một cụm có `TASK` retry policy, bạn nên đặt thuộc tính quản lý truy vấn ([query management property](https://trino.io/docs/current/admin/properties-query-management.html)) `task.low-memory-killer.policy` thành `total-reservation-on-blocked-nodes`, nếu không các truy vấn có thể cần phải được hủy thủ công nếu cụm hết memory.

>[!note]
>`TASK` retry policy phù hợp nhất cho các truy vấn theo lô lớn, nhưng chính sách này có thể dẫn đến độ trễ cao hơn đối với các truy vấn ngắn được thực thi với khối lượng lớn. Theo kinh nghiệm tốt nhất, nên chạy một cụm máy chủ chuyên dụng với chính sách thử lại `TASK` cho các truy vấn theo batch lớn, tách biệt với một cụm máy chủ khác xử lý các truy vấn ngắn.

## Encryption

Trino mã hóa dữ liệu trước khi chuyển vào bộ nhớ lưu trữ. Điều này ngăn chặn việc truy cập dữ liệu truy vấn bởi bất kỳ ai ngoài cụm máy chủ Trino đã ghi dữ liệu đó, kể cả quản trị viên hệ thống lưu trữ. Một khóa mã hóa mới được tạo ngẫu nhiên cho mỗi truy vấn và khóa này sẽ bị loại bỏ sau khi truy vấn hoàn tất.

## Advanced configuration

Bạn có thể cấu hình thêm khả năng thực thi chịu lỗi bằng các thuộc tính cấu hình sau. Các giá trị mặc định cho các thuộc tính này sẽ hoạt động tốt trong hầu hết các trường hợp triển khai, nhưng bạn có thể thay đổi các giá trị này để thử nghiệm hoặc khắc phục sự cố.

### Retry limits

Các thuộc tính cấu hình sau đây kiểm soát ngưỡng mà tại đó các queries/tasks sẽ không được thử lại trong trường hợp xảy ra lỗi lặp đi lặp lại:

             Fault tolerance retry limit configuration properties

|Property name|Description|Default value|Retry policy|
|---|---|---|---|
|`query-retry-attempts`|Maximum number of times Trino may attempt to retry a query before declaring the query as failed.|`4`|Only `QUERY`|
|`task-retry-attempts-per-task`|Maximum number of times Trino may attempt to retry a single task before declaring the query as failed.|`4`|Only `TASK`|
|`retry-initial-delay`|Minimum [time](https://trino.io/docs/current/admin/properties.html#prop-type-duration) that a failed query or task must wait before it is retried. May be overridden with the `retry_initial_delay` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).|`10s`|`QUERY` and `TASK`|
|`retry-max-delay`|Maximum [time](https://trino.io/docs/current/admin/properties.html#prop-type-duration) that a failed query or task must wait before it is retried. Wait time is increased on each subsequent failure. May be overridden with the `retry_max_delay` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).|`1m`|`QUERY` and `TASK`|
|`retry-delay-scale-factor`|Factor by which retry delay is increased on each query or task failure. May be overridden with the `retry_delay_scale_factor` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).|`2.0`|`QUERY` and `TASK`|
### Task sizing

Với `TASK` retry policy, việc quản lý lượng dữ liệu được xử lý trong mỗi task là rất quan trọng. Nếu các task quá nhỏ, việc quản lý điều phối task có thể tốn nhiều thời gian xử lý và tài nguyên hơn cả việc thực thi task đó. Nếu các task quá lớn, thì một task duy nhất có thể yêu cầu nhiều tài nguyên hơn mức có sẵn trên bất kỳ node nào và do đó ngăn cản việc hoàn thành truy vấn.

Trino hỗ trợ tính năng tự động điều chỉnh kích thước task ở mức độ hạn chế. Nếu xảy ra sự cố trong quá trình thực thi fault-tolerant task, bạn có thể cấu hình các thuộc tính sau để điều khiển kích thước task theo cách thủ công. Các thuộc tính cấu hình này chỉ áp dụng cho `TASK` retry policy.

                Task sizing configuration properties

| Property name                                                                            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | Default value |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------- |
| `fault-tolerant-execution-standard-split-size`                                           | Standard [split](https://trino.io/docs/current/overview/concepts.html#trino-concept-splits) [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) processed by tasks that read data from source tables. Value is interpreted with split weight taken into account. If the weight of splits produced by a catalog denotes that they are lighter or heavier than “standard” split, then the number of splits processed by a single task is adjusted accordingly.<br><br>May be overridden for the current session with the `fault_tolerant_execution_standard_split_size` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition). | `64MB`        |
| `fault-tolerant-execution-max-task-split-count`                                          | Maximum number of [splits](https://trino.io/docs/current/overview/concepts.html#trino-concept-splits) processed by a single task. This value is not split weight-adjusted and serves as protection against situations where catalogs report an incorrect split weight.<br><br>May be overridden for the current session with the `fault_tolerant_execution_max_task_split_count` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).                                                                                                                                                                                                                       | `2048`        |
| `fault-tolerant-execution-arbitrary-distribution-compute-task-target-size-growth-period` | The number of tasks created for any given non-writer stage of arbitrary distribution before task size is increased.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | `64`          |
| `fault-tolerant-execution-arbitrary-distribution-compute-task-target-size-growth-factor` | Growth factor for adaptive sizing of non-writer tasks of arbitrary distribution for fault-tolerant execution. Lower bound is 1.0. For every task size increase, new task target size is old task target size multiplied by this growth factor.                                                                                                                                                                                                                                                                                                                                                                                                                                                               | `1.26`        |
| `fault-tolerant-execution-arbitrary-distribution-compute-task-target-size-min`           | Initial/minimum target input [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for non-writer tasks of arbitrary distribution of fault-tolerant execution.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | `512MB`       |
| `fault-tolerant-execution-arbitrary-distribution-compute-task-target-size-max`           | Maximum target input [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for each non-writer task of arbitrary distribution of fault-tolerant execution.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `50GB`        |
| `fault-tolerant-execution-arbitrary-distribution-write-task-target-size-growth-period`   | The number of tasks created for any given writer stage of arbitrary distribution before task size is increased.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | `64`          |
| `fault-tolerant-execution-arbitrary-distribution-write-task-target-size-growth-factor`   | Growth factor for adaptive sizing of writer tasks of arbitrary distribution for fault-tolerant execution. Lower bound is 1.0. For every task size increase, new task target size is old task target size multiplied by this growth factor.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | `1.26`        |
| `fault-tolerant-execution-arbitrary-distribution-write-task-target-size-min`             | Initial/minimum target input [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for writer tasks of arbitrary distribution of fault-tolerant execution.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `4GB`         |
| `fault-tolerant-execution-arbitrary-distribution-write-task-target-size-max`             | Maximum target input [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for writer tasks of arbitrary distribution of fault-tolerant execution.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `50GB`        |
| `fault-tolerant-execution-hash-distribution-compute-task-target-size`                    | Target input [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for non-writer tasks of hash distribution of fault-tolerant execution.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | `512MB`       |
| `fault-tolerant-execution-hash-distribution-write-task-target-size`                      | Target input [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) of writer tasks of hash distribution of fault-tolerant execution.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | `4GB`         |
| `fault-tolerant-execution-hash-distribution-write-task-target-max-count`                 | Soft upper bound on number of writer tasks in a stage of hash distribution of fault-tolerant execution.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `2000`        |
|                                                                                          |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |               |
### Node allocation

Với `TASK` retry policy các node được phân bổ cho các task dựa trên memory khả dụng và mức sử dụng memory ước tính. Nếu task thất bại do vượt quá memory khả dụng trên một node, task sẽ được khởi động lại với yêu cầu phân bổ toàn bộ node để thực thi.

Việc ước tính yêu cầu memory ban đầu của task là tĩnh và được cấu hình `fault-tolerant-execution-task-memory`. Thuộc tính này chỉ áp dụng cho `TASK` retry policy.

                Node allocation configuration properties

|Property name|Description|Default value|
|---|---|---|
|`fault-tolerant-execution-task-memory`|Initial task memory [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) estimation used for bin-packing when allocating nodes for tasks. May be overridden for the current session with the `fault_tolerant_execution_task_memory` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).|`5GB`|
### Other tuning

Thuộc tính cấu hình bổ sung sau đây có thể được sử dụng để quản lý việc thực thi chịu lỗi:

              Other fault-tolerant execution configuration properties

|Property name|Description|Default value|Retry policy|
|---|---|---|---|
|`fault-tolerant-execution-task-descriptor-storage-max-memory`|Maximum [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) of memory to be used to store task descriptors for fault-tolerant queries on coordinator. Extra memory is needed to be able to reschedule tasks in case of a failure.|(JVM heap size * 0.15)|Only `TASK`|
|`fault-tolerant-execution-max-partition-count`|Maximum number of partitions to use for distributed joins and aggregations, similar in function to the `query.max-hash-partition-count` [query management property](https://trino.io/docs/current/admin/properties-query-management.html). It is not recommended to increase this property value higher than the default of `50`, which may result in instability and poor performance. May be overridden for the current session with the `fault_tolerant_execution_max_partition_count` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).|`50`|Only `TASK`|
|`fault-tolerant-execution-min-partition-count`|Minimum number of partitions to use for distributed joins and aggregations, similar in function to the `query.min-hash-partition-count` [query management property](https://trino.io/docs/current/admin/properties-query-management.html). May be overridden for the current session with the `fault_tolerant_execution_min_partition_count` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).|`4`|Only `TASK`|
|`fault-tolerant-execution-min-partition-count-for-write`|Minimum number of partitions to use for distributed joins and aggregations in write queries, similar in function to the `query.min-hash-partition-count-for-write` [query management property](https://trino.io/docs/current/admin/properties-query-management.html). May be overridden for the current session with the `fault_tolerant_execution_min_partition_count_for_write` [session property](https://trino.io/docs/current/sql/set-session.html#session-properties-definition).|`50`|Only `TASK`|
|`max-tasks-waiting-for-node-per-query`|Allow for up to configured number of tasks to wait for node allocation per query, before pausing scheduling for other tasks from this query.|`50`|Only `TASK`|
## Exchange manager

Cơ chế lưu trữ tạm thời (spooling) của Exchange chịu trách nhiệm lưu trữ và quản lý dữ liệu tạm thời để đảm bảo khả năng thực thi chịu lỗi. Bạn có thể cấu hình trình quản lý Exchange dựa trên hệ thống tệp để lưu trữ dữ liệu tạm thời tại một vị trí cụ thể, chẳng hạn như AWS S3 và các hệ thống tương thích với [AWS S3](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-exchange-aws-s3) and S3-compatible systems, [Azure Blob Storage](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-exchange-azure-blob), [Google Cloud Storage](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-exchange-gcs), or [HDFS](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-exchange-hdfs).

### Configuration

Để cấu hình exchange manager, hãy tạo một tệp cấu hình mới `etc/exchange-manager.properties` trên coordinator và tất cả các worker nodes. Trong tệp này, hãy đặt thuộc tính cấu hình `exchange-manager.name` thành `filesystem` hoặc `hdfs`, và đặt thêm các thuộc tính cấu hình khác nếu cần cho giải pháp lưu trữ của bạn.

Bạn cũng có thể chỉ định vị trí của tệp cấu hình exchange manager trong `config.properties` bằng thuộc tính `exchange-manager.config-file`. Khi thuộc tính này được thiết lập, Trino sẽ tải cấu hình exchange manager từ đường dẫn được chỉ định thay vì đường dẫn mặc định `etc/exchange-manager.properties`.

Bảng sau liệt kê các thuộc tính cấu hình khả dụng cho `exchange-manager.properties`, giá trị mặc định của chúng và các hệ thống tệp mà thuộc tính đó có thể được cấu hình:

                Exchange manager configuration properties

|Property name|Description|Default value|Supported filesystem|
|---|---|---|---|
|`exchange.base-directories`|Comma-separated list of URI locations that the exchange manager uses to store spooling data.||Any|
|`exchange.max-page-storage-size`|Max storage size of a page written to a sink, including the page itself and its size.|`16MB`|Any|
|`exchange.sink-buffer-pool-min-size`|The minimum buffer pool size for an exchange sink. The larger the buffer pool size, the larger the write parallelism and memory usage.|`10`|Any|
|`exchange.sink-buffers-per-partition`|The number of buffers per partition in the buffer pool. The larger the buffer pool size, the larger the write parallelism and memory usage.|`2`|Any|
|`exchange.sink-max-file-size`|Max [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) of files written by exchange sinks.|`1GB`|Any|
|`exchange.source-concurrent-readers`|Number of concurrent readers to read from spooling storage. The larger the number of concurrent readers, the larger the read parallelism and memory usage.|`4`|Any|
|`exchange.s3.aws-access-key`|AWS access key to use. Required for a connection to AWS S3 and GCS, can be ignored for other S3 storage systems.||AWS S3, GCS|
|`exchange.s3.aws-secret-key`|AWS secret key to use. Required for a connection to AWS S3 and GCS, can be ignored for other S3 storage systems.||AWS S3, GCS|
|`exchange.s3.iam-role`|IAM role to assume.||AWS S3, GCS|
|`exchange.s3.external-id`|External ID for the IAM role trust policy.||AWS S3, GCS|
|`exchange.s3.region`|Region of the S3 bucket.||AWS S3, GCS|
|`exchange.s3.endpoint`|S3 storage endpoint server if using an S3-compatible storage system that is not AWS. If using AWS S3, this can be ignored unless HTTPS is required by an AWS bucket policy. If TLS is required, then this property can be set to an https endpoint such as `https://s3.us-east-1.amazonaws.com`. Note that TLS is redundant due to [automatic encryption](https://trino.io/docs/current/admin/fault-tolerant-execution.html#fte-encryption). If using GCS, set it to `https://storage.googleapis.com`.||Any S3-compatible storage|
|`exchange.s3.max-error-retries`|Maximum number of times the exchange manager’s S3 client should retry a request.|`10`|Any S3-compatible storage|
|`exchange.s3.path-style-access`|Enables using [path-style access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/VirtualHosting.html#path-style-access) for all requests to S3.|`false`|Any S3-compatible storage|
|`exchange.s3.upload.part-size`|Part [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for S3 multi-part upload.|`5MB`|Any S3-compatible storage|
|`exchange.gcs.json-key-file-path`|Path to the JSON file that contains your Google Cloud Platform service account key. Not to be set together with `exchange.gcs.json-key`||GCS|
|`exchange.gcs.json-key`|Your Google Cloud Platform service account key in JSON format. Not to be set together with `exchange.gcs.json-key-file-path`||GCS|
|`exchange.azure.endpoint`|Azure blob endpoint used to access the spooling container. Not to be set together with `exchange.azure.connection-string`||Azure Blob Storage|
|`exchange.azure.connection-string`|Connection string used to access the spooling container. Not to be set together with `exchange.azure.endpoint`||Azure Blob Storage|
|`exchange.azure.block-size`|Block [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for Azure block blob parallel upload.|`4MB`|Azure Blob Storage|
|`exchange.azure.max-error-retries`|Maximum number of times the exchange manager’s Azure client should retry a request.|`10`|Azure Blob Storage|
|`exchange.hdfs.block-size`|Block [data size](https://trino.io/docs/current/admin/properties.html#prop-type-data-size) for HDFS storage.|`4MB`|HDFS|
|`exchange.hdfs.skip-directory-scheme-validation`|Skip directory scheme validation to support Hadoop-compatible file system.|false|HDFS|
|`hdfs.config.resources`|Comma-separated list of paths to HDFS configuration files, for example `/etc/hdfs-site.xml`. The files must exist on all nodes in the Trino cluster.||HDFS|
Để giảm tải I/O tổng thể của exchange manager, thuộc tính cấu hình `exchange.compression-codec` mặc định là `LZ4`. Ngoài ra, quá trình nén và giải nén tập tin được thực hiện tự động và một số chi tiết có thể được cấu hình.

Ngoài ra, nên cấu hình quy tắc vòng đời của bucket để tự động xóa các đối tượng bị bỏ quên trong trường hợp node gặp sự cố.
#### AWS S3

```
exchange-manager.name=filesystem
exchange.base-directories=s3://exchange-spooling-bucket
exchange.s3.region=us-west-1
exchange.s3.aws-access-key=example-access-key
exchange.s3.aws-secret-key=example-secret-key
```

```
exchange.base-directories=s3://exchange-spooling-bucket-1,s3://exchange-spooling-bucket-2
```

#### Azure Blob Storage

```
exchange-manager.name=filesystem
exchange.base-directories=abfs://container_name@account_name.dfs.core.windows.net
exchange.azure.connection-string=connection-string
```

#### Google Cloud Storage

```
exchange-manager.name=filesystem
exchange.base-directories=gs://exchange-spooling-bucket
exchange.s3.region=us-west-1
exchange.s3.aws-access-key=example-access-key
exchange.s3.aws-secret-key=example-secret-key
exchange.s3.endpoint=https://storage.googleapis.com
exchange.gcs.json-key-file-path=/path/to/gcs_keyfile.json
```

#### HDFS

```
exchange-manager.name=hdfs
exchange.base-directories=hadoop-master:9000/exchange-spooling-directory
hdfs.config.resources=/usr/lib/hadoop/etc/hadoop/core-site.xml
```

#### Local filesystem storage

Ví dụ cấu hình `exchange-manager.properties` sau đây chỉ định một thư mục cục bộ, `/tmp/trino-exchange-manager`, làm đích lưu trữ tạm thời.

>[!note]
>Chỉ nên sử dụng hệ thống tệp cục bộ để trao đổi dữ liệu trong các cụm máy chủ độc lập, không dùng cho mục đích sản xuất. Thư mục cục bộ chỉ có thể được sử dụng để trao đổi dữ liệu trong một cụm máy chủ phân tán nếu thư mục trao đổi được chia sẻ và có thể truy cập được từ tất cả các nút.

```
exchange-manager.name=filesystem
exchange.base-directories=/tmp/trino-exchange-manager
```

## Adaptive plan optimizations

Chế độ thực thi chịu lỗi cung cấp một số tối ưu hóa kế hoạch thích ứng, điều chỉnh kế hoạch thực thi truy vấn một cách linh hoạt dựa trên số liệu thống kê thời gian chạy. Để biết thêm thông tin, hãy xem [[Adaptive plan optimizations]].

Nguồn: https://trino.io/docs/current/admin/fault-tolerant-execution.html
