# Iceberg Table Spec

Đây là bản specification (đặc tả) cho định dạng bảng Iceberg, được thiết kế để quản lý một tập hợp lớn các tệp tin thay đổi chậm trong distributed file system hoặc kho lưu trữ key-value dưới dạng bảng.

## Format Versioning

Các phiên bản 1, 2 và 3 của Iceberg spec đã hoàn thiện và được cộng đồng chấp nhận.

Phiên bản 4 hiện đang được phát triển tích cực và chưa được chính thức thông qua.

Số phiên bản định dạng được tăng lên khi các tính năng mới được thêm vào có thể phá vỡ khả năng tương thích ngược — nghĩa là, khi các trình đọc cũ hơn sẽ không đọc đúng các tính năng bảng mới hơn. Các bảng vẫn có thể được ghi bằng phiên bản cũ hơn của đặc tả để đảm bảo khả năng tương thích bằng cách không sử dụng các tính năng chưa được các công cụ xử lý triển khai.

### Version 1: Analytic Data Tables

### Version 2: Row-level Deletes

### Version 3: Extended Types and Capabilities

### Version 4: Metadata Structure and Representation

## Goals

- **Serializable isolation**: Các thao tác đọc (read) sẽ được tách biệt khỏi các thao tác ghi (write) đồng thời và luôn sử dụng commited snapshot của dữ liệu bảng. Các thao tác ghi sẽ hỗ trợ xóa và thêm tệp trong một lần thao tác duy nhất và không bao giờ hiển thị một phần. Người đọc sẽ không chiếm giữ khóa.
- Speed: Các thao tác sẽ sử dụng các lệnh gọi từ xa O(1) để lập kế hoạch các tệp cho quá trình quét chứ không phải O(n) trong đó n tăng theo kích thước của bảng, chẳng hạn như số lượng phân vùng hoặc tệp.
- **Scale**: Việc lập kế hoạch công việc sẽ chủ yếu do clients đảm nhiệm và không bị tắc nghẽn ở central metadata store. Metadata sẽ bao gồm thông tin cần thiết cho việc tối ưu hóa dựa trên chi phí.
- **Evolution**: Bảng sẽ hỗ trợ đầy đủ quá trình schema và partition spec evolution. Schema evolution hỗ trợ thêm, xóa, sắp xếp lại và đổi tên cột một cách an toàn, kể cả trong cấu trúc lồng nhau.
- **Storage separation**: Việc phân vùng sẽ được cấu hình trong bảng. Việc đọc dữ liệu sẽ được lập kế hoạch bằng cách sử dụng các điều kiện lọc trên giá trị dữ liệu, chứ không phải giá trị phân vùng. Các bảng sẽ hỗ trợ các partition schemes có thể thay đổi (evolving).
- **Formats**: Các định dạng tệp dữ liệu cơ bản sẽ hỗ trợ các quy tắc và kiểu schema evolution giống hệt nhau. Cả định dạng tối ưu hóa đọc (read-optimized) và tối ưu hóa ghi (write-optimized) đều sẽ có sẵn.

## Overview

![[iceberg_spec_overview.png]]

Định dạng bảng này theo dõi các tệp dữ liệu riêng lẻ trong một bảng thay vì các thư mục. Điều này cho phép người ghi tạo các tệp dữ liệu trực tiếp và chỉ thêm các tệp vào bảng khi có lệnh commit rõ ràng.

Trạng thái của bảng được lưu trữ trong các metadata file. Mọi thay đổi đối với trạng thái bảng đều tạo ra một tệp metadata mới và thay thế metadata cũ bằng một thao tác hoán đổi nguyên tử (atomic swap). The table metadata file theo dõi table schema, partitioning config, custom properties, and snapshots of the table contents. A snapshot thể hiện trạng thái của bảng tại một thời điểm nào đó và được sử dụng để truy cập toàn bộ tập hợp các tệp dữ liệu trong bảng.

Các tập tin dữ liệu trong snapshot được theo dõi bởi một hoặc nhiều tập tin kê khai (manifest files), mỗi tập tin chứa một hàng cho mỗi tập tin dữ liệu trong bảng, dữ liệu phân vùng của tập tin và các chỉ số của nó (metrics). Dữ liệu trong snapshot là sự kết hợp của tất cả các tập tin trong các tập tin kê khai (manifests) của nó. Các tập tin kê khai (Manifest files) được sử dụng lại trên nhiều snapshot để tránh ghi đè metadata ,thứ gây thay đổi chậm. Các tập tin kê khai (manifest files) có thể theo dõi các tập tin dữ liệu với bất kỳ tập hợp con nào của bảng và không được liên kết với các phân vùng.

Các tệp kê khai (manifests) tạo nên snapshot được lưu trữ trong một tệp danh sách tệp kê khai (manifest list file). Mỗi tệp danh sách tệp kê (manifest list) khai lưu trữ metadata về các tệp kê khai (manifests), bao gồm số liệu thống kê phân vùng và số lượng tệp dữ liệu. Các số liệu thống kê này được sử dụng để tránh đọc các tệp kê khai (manifests) không cần thiết cho một thao tác.

Xem thêm:

---
1. Bản chất thay đổi: Quản lý theo "File" thay vì "Thư mục"

Trong kiến trúc cũ (Hive), khi bạn thực hiện câu lệnh ghi dữ liệu, kết quả sẽ được ghi trực tiếp vào một thư mục (directory). Nếu một câu lệnh đang ghi dở mà bị sập, thư mục đó sẽ chứa dữ liệu "rác" (partial data), dẫn đến việc người khác vào đọc sẽ bị sai.
Iceberg thay đổi cuộc chơi bằng cách:
- **Ghi in-place (tại chỗ):** Các file dữ liệu mới (`.parquet`, `.orc`, `.avro`) cứ việc ghi thoải mái xuống Storage (MinIO/S3). Nhưng lúc này, hệ thống **chưa ai nhìn thấy** các file này cả.
- **Atomic Commit (Cam kết nguyên tử):** Chỉ khi toàn bộ các file được ghi xong xuôi, một hành động "Commit" được kích hoạt để cập nhật trạng thái bảng. Nếu commit thành công, dữ liệu mới chính thức xuất hiện. Nếu commit lỗi, các file vừa ghi sẽ bị bỏ qua (đảm bảo tính ACID).

2. Chi tiết 3 Tầng Metadata của Apache Iceberg
Để quản lý được việc "commit" và trạng thái bảng, Iceberg chia Metadata thành 3 tầng file được lưu trữ dạng cây (Tree Structure):

### Tầng 1: Table Metadata File (`.metadata.json`)

Đây là "đầu não" quản lý trạng thái của bảng. Mỗi khi bảng có thay đổi (thêm dữ liệu, thay đổi schema, thay đổi cấu hình phân vùng), một file `.metadata.json` mới sẽ được tạo ra và thay thế file cũ bằng một cú **Atomic Swap** (đổi con trỏ ở tầng Catalog như Nessie/Hive Metastore).

File này chứa:

- **Schema:** Cấu trúc các cột (định danh bằng Field ID như đã nói ở câu hỏi trước).
    
- **Partition Specs:** Cấu hình phân vùng (ví dụ: phân vùng theo ngày, theo tháng).
    
- **Snapshots:** Lịch sử các phiên bản của bảng. Mỗi Snapshot đại diện cho trạng thái toàn bộ dữ liệu tại một thời điểm cụ thể (đây chính là chìa khóa để làm **Time Travel** - truy vấn dữ liệu quá khứ).
    
### Tầng 2: Manifest List File (`snap-*.avro`)

Mỗi một **Snapshot** ở Tầng 1 sẽ trỏ tương ứng đến một file **Manifest List**.

- Nhiệm vụ của Manifest List là quản lý danh sách các file **Manifest** (Tầng 3) cấu thành nên Snapshot đó.
    
- **Tối ưu hóa (Pruning):** File này lưu trữ các thông tin thống kê cấp cao (như dải giá trị của partition trong từng manifest, số lượng data file). Nhờ vậy, khi bạn query, Engine (Trino/Spark) chỉ cần đọc Manifest List là biết ngay cần phải xuống đọc tiếp những Manifest nào, bỏ qua (skip) những Manifest nào không cần thiết mà không cần quét toàn bộ bảng.
    
### Tầng 3: Manifest File (`*.avro`)

Đây là tầng metadata cuối cùng trước khi chạm đến dữ liệu thực tế. Một Manifest File quản lý một tập hợp các **Data Files**.

- Nó chứa một hàng (row) cho **mỗi data file** trong bảng.
    
- Lưu trữ chi tiết: Đường dẫn file (URI), dữ liệu phân vùng của file đó, và đặc biệt là **Metrics** (giá trị Min/Max của các cột, số lượng dòng, số lượng giá trị Null).
    
- **Tính năng tái sử dụng (Reuse):** Như đoạn văn của bạn có nêu: _""Manifest files are reused across snapshots...""_. Khi bạn append thêm dữ liệu mới, Iceberg chỉ tạo ra 1 manifest mới cho file dữ liệu mới đó, rồi tạo một Manifest List mới trỏ đến cả Manifest cũ lẫn Manifest mới. Hành động này giúp Iceberg chạy cực nhanh vì không phải viết lại metadata của các dữ liệu cũ (vốn ít thay đổi).

---

### Optimistic Concurrency

Việc atomic swap một table metadata file này với một table metadata file khác tạo cơ sở cho serializable isolation. Người đọc sử dụng snapshot hiện tại khi họ tải table metadata và không bị ảnh hưởng bởi các thay đổi cho đến khi họ làm mới và chọn vị trí metadata mới.

### Sequence Numbers

### Row-level Deletes

### File System Operations

### File Locations in Metadata

## Specification