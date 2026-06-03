- [[#Concurrent write operations|Concurrent write operations]]
	- [[#Concurrent write operations#Cost of retries|Cost of retries]]
- [[#Compatibility|Compatibility]]

Iceberg được thiết kế để giải quyết các vấn đề về tính chính xác ảnh hưởng đến các bảng Hive chạy trên S3.

Bảng Hive theo dõi các data files bằng cả central metastore cho các phân vùng và hệ thống tệp cho từng tệp riêng lẻ. Điều này khiến cho việc atomic changes nội dung của bảng trở nên bất khả thi, và các kho lưu trữ nhất quán cuối cùng như S3 có thể trả về kết quả không chính xác do việc sử dụng các listing files để tái tạo trạng thái của bảng. Nó cũng yêu cầu lập kế hoạch công việc để thực hiện nhiều lệnh gọi liệt kê chậm: O(n) với số lượng phân vùng.

Iceberg theo dõi toàn bộ danh sách các data files trong mỗi [snapshot](https://iceberg.apache.org/terms/#snapshot) bằng cách sử dụng cấu trúc cây bền vững. Mỗi thao tác ghi hoặc xóa sẽ tạo ra một snapshot mới, tái sử dụng tối đa cấu trúc cây metadata của snapshot trước đó để tránh khối lượng ghi lớn.

Các snapshots hợp lệ trong bảng Iceberg được lưu trữ trong tệp metadata của bảng, cùng với tham chiếu đến snapshot hiện tại. Các thao tác commit thay thế đường dẫn của tệp metadata bảng hiện tại bằng một thao tác nguyên tử (atomic). Điều này đảm bảo rằng tất cả các cập nhật đối với dữ liệu và metadata của bảng đều là atomic, và là cơ sở cho [serializable isolation](https://en.wikipedia.org/wiki/Isolation_\(database_systems\)#Serializable).

Điều này giúp cải thiện độ tin cậy.

- **Serializable isolation**: All table changes occur in a linear history of atomic table updates
- **Reliable reads**: Readers always use a consistent snapshot of the table without holding a lock
- **Version history and rollback**: Table snapshots are kept as history and tables can roll back if a job produces bad data
- **Safe file-level operations**. By supporting atomic changes, Iceberg enables new use cases, like safely compacting small files and safely appending late data to tables

Thiết kế này cũng có những lợi ích về hiệu suất.

- **O(1) RPCs to plan**: Instead of listing O(n) directories in a table to plan a job, reading a snapshot requires O(1) RPC calls
- **Distributed planning**: File pruning and predicate push-down is distributed to jobs, removing the metastore as a bottleneck
- **Finer granularity partitioning**: Distributed planning and O(1) RPC calls remove the current barriers to finer-grained partitioning

## Concurrent write operations

Iceberg hỗ trợ nhiều thao tác ghi đồng thời bằng cách sử dụng cơ chế đồng thời lạc quan (optimistic concurrency).

Mỗi tác vụ ghi giả định rằng không có tác vụ ghi nào khác đang hoạt động và ghi metadata bảng mới cho một thao tác. Sau đó, tác vụ ghi cố gắng commit bằng cách atomically swapping tệp new table metadata file với existing metadata file.

Nếu thao tác atomically swapping thất bại do một tác nhân ghi khác đã thực hiện việc ghi, tác nhân ghi bị lỗi sẽ thử lại bằng cách ghi một cây metadata mới dựa trên trạng thái bảng hiện tại mới.

### Cost of retries

Writers tránh các thao tác thử lại tốn kém bằng cách cấu trúc các thay đổi sao cho công việc có thể được tái sử dụng trong các lần thử lại.

Ví dụ, thao tác thêm dữ liệu thường tạo một tệp manifest mới cho các tệp dữ liệu được thêm vào, tệp này có thể được thêm vào bảng mà không cần phải ghi đè tệp manifest mỗi lần thực hiện.

Retry validation

Các commit được cấu trúc dưới dạng các giả định và hành động. Sau khi xảy ra xung đột, người ghi commit sẽ kiểm tra xem các giả định có được đáp ứng bởi trạng thái hiện tại của bảng hay không. Nếu các giả định được đáp ứng, thì việc áp dụng lại các hành động và thực hiện commit là an toàn.

Ví dụ, thao tác nén có thể ghi đè `file_a.avro` và `file_b.avro` thành `merged.parquet`. Thao tác này an toàn để thực hiện commit miễn là bảng vẫn chứa cả `file_a.avro` và `file_b.avro`. Nếu một trong hai tệp bị xóa do xung đột trong thao tác commit, thì thao tác đó phải thất bại. Ngược lại, việc xóa các tệp nguồn và thêm tệp đã hợp nhất là an toàn.

## Compatibility

Bằng cách tránh các thao tác liệt kê và đổi tên tệp, bảng Iceberg tương thích với bất kỳ kho lưu trữ đối tượng nào. Không cần phải liệt kê nhất quán.