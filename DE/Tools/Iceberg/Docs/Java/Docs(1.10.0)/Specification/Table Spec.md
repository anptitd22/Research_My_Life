- [[#Format Versioning|Format Versioning]]
	- [[#Format Versioning#Version 1: Analytic Data Tables|Version 1: Analytic Data Tables]]
	- [[#Format Versioning#Version 2: Row-level Deletes|Version 2: Row-level Deletes]]
	- [[#Format Versioning#Version 3: Extended Types and Capabilities|Version 3: Extended Types and Capabilities]]
	- [[#Format Versioning#Version 4: Metadata Structure and Representation|Version 4: Metadata Structure and Representation]]
- [[#Goals|Goals]]
- [[#Overview|Overview]]
	- [[#Overview#Optimistic Concurrency|Optimistic Concurrency]]
	- [[#Overview#Sequence Numbers|Sequence Numbers]]
	- [[#Overview#Row-level Deletes|Row-level Deletes]]
	- [[#Overview#File System Operations|File System Operations]]
	- [[#Overview#File Locations in Metadata|File Locations in Metadata]]
- [[#Specification|Specification]]
	- [[#Specification#Terms|Terms]]
	- [[#Specification#Writer requirements[🔗](https://iceberg.apache.org/spec/#writer-requirements "Permanent link")|Writer requirements[🔗](https://iceberg.apache.org/spec/#writer-requirements "Permanent link")]]
		- [[#Writer requirements[🔗](https://iceberg.apache.org/spec/#writer-requirements "Permanent link")#Writing data files[🔗](https://iceberg.apache.org/spec/#writing-data-files "Permanent link")|Writing data files[🔗](https://iceberg.apache.org/spec/#writing-data-files "Permanent link")]]
	- [[#Specification#Paths in Metadata[🔗](https://iceberg.apache.org/spec/#paths-in-metadata "Permanent link")|Paths in Metadata[🔗](https://iceberg.apache.org/spec/#paths-in-metadata "Permanent link")]]
		- [[#Paths in Metadata[🔗](https://iceberg.apache.org/spec/#paths-in-metadata "Permanent link")#Path Resolution[🔗](https://iceberg.apache.org/spec/#path-resolution "Permanent link")|Path Resolution[🔗](https://iceberg.apache.org/spec/#path-resolution "Permanent link")]]
		- [[#Paths in Metadata[🔗](https://iceberg.apache.org/spec/#paths-in-metadata "Permanent link")#Path Relativization[🔗](https://iceberg.apache.org/spec/#path-relativization "Permanent link")|Path Relativization[🔗](https://iceberg.apache.org/spec/#path-relativization "Permanent link")]]
		- [[#Paths in Metadata[🔗](https://iceberg.apache.org/spec/#paths-in-metadata "Permanent link")#Table Location Specification[🔗](https://iceberg.apache.org/spec/#table-location-specification "Permanent link")|Table Location Specification[🔗](https://iceberg.apache.org/spec/#table-location-specification "Permanent link")]]
	- [[#Specification#Schemas and Data Types|Schemas and Data Types]]
		- [[#Schemas and Data Types#Nested Types[🔗](https://iceberg.apache.org/spec/#nested-types "Permanent link")|Nested Types[🔗](https://iceberg.apache.org/spec/#nested-types "Permanent link")]]
		- [[#Schemas and Data Types#Semi-structured Types[🔗](https://iceberg.apache.org/spec/#semi-structured-types "Permanent link")|Semi-structured Types[🔗](https://iceberg.apache.org/spec/#semi-structured-types "Permanent link")]]
		- [[#Schemas and Data Types#Primitive Types[🔗](https://iceberg.apache.org/spec/#primitive-types "Permanent link")|Primitive Types[🔗](https://iceberg.apache.org/spec/#primitive-types "Permanent link")]]
		- [[#Schemas and Data Types#Default values[🔗](https://iceberg.apache.org/spec/#default-values "Permanent link")|Default values[🔗](https://iceberg.apache.org/spec/#default-values "Permanent link")]]
		- [[#Schemas and Data Types#Schema Evolution[🔗](https://iceberg.apache.org/spec/#schema-evolution "Permanent link")|Schema Evolution[🔗](https://iceberg.apache.org/spec/#schema-evolution "Permanent link")]]
		- [[#Schemas and Data Types#Identifier Field IDs[🔗](https://iceberg.apache.org/spec/#identifier-field-ids "Permanent link")|Identifier Field IDs[🔗](https://iceberg.apache.org/spec/#identifier-field-ids "Permanent link")]]
		- [[#Schemas and Data Types#Reserved Field IDs[🔗](https://iceberg.apache.org/spec/#reserved-field-ids "Permanent link")|Reserved Field IDs[🔗](https://iceberg.apache.org/spec/#reserved-field-ids "Permanent link")]]
		- [[#Schemas and Data Types#Row Lineage[🔗](https://iceberg.apache.org/spec/#row-lineage "Permanent link")|Row Lineage[🔗](https://iceberg.apache.org/spec/#row-lineage "Permanent link")]]
	- [[#Specification#Partitioning|Partitioning]]
		- [[#Partitioning#Partition Transforms|Partition Transforms]]
		- [[#Partitioning#Bucket Transform Details[🔗](https://iceberg.apache.org/spec/#bucket-transform-details "Permanent link")|Bucket Transform Details[🔗](https://iceberg.apache.org/spec/#bucket-transform-details "Permanent link")]]
		- [[#Partitioning#Truncate Transform Details[🔗](https://iceberg.apache.org/spec/#truncate-transform-details "Permanent link")|Truncate Transform Details[🔗](https://iceberg.apache.org/spec/#truncate-transform-details "Permanent link")]]
		- [[#Partitioning#Partition Evolution[🔗](https://iceberg.apache.org/spec/#partition-evolution "Permanent link")|Partition Evolution[🔗](https://iceberg.apache.org/spec/#partition-evolution "Permanent link")]]
	- [[#Specification#Sorting|Sorting]]
	- [[#Specification#Manifests|Manifests]]
		- [[#Manifests#Manifest Entry Fields|Manifest Entry Fields]]
	- [[#Specification#Snapshots|Snapshots]]
	- [[#Specification#Manifest Lists|Manifest Lists]]
		- [[#Manifest Lists#First Row ID Assignment|First Row ID Assignment]]
	- [[#Specification#Scan Planning|Scan Planning]]


# Iceberg Table Spec

Đây là bản specification (đặc tả) cho định dạng bảng Iceberg, được thiết kế để quản lý một tập hợp lớn các tệp tin thay đổi chậm trong distributed file system hoặc kho lưu trữ key-value dưới dạng bảng.

## Format Versioning

>[!note]
>Khác với software version (1.10.0, 1.11.0, ...), đây là version of iceberg spec

Các phiên bản 1, 2 và 3 của Iceberg spec đã hoàn thiện và được cộng đồng chấp nhận.

Phiên bản 4 hiện đang được phát triển tích cực và chưa được chính thức thông qua.

Số phiên bản định dạng được tăng lên khi các tính năng mới được thêm vào có thể phá vỡ khả năng tương thích ngược — nghĩa là, khi các trình đọc cũ hơn sẽ không đọc đúng các tính năng bảng mới hơn. Các bảng vẫn có thể được ghi bằng phiên bản cũ hơn của đặc tả để đảm bảo khả năng tương thích bằng cách không sử dụng các tính năng chưa được các công cụ xử lý triển khai.

### Version 1: Analytic Data Tables

Phiên bản 1 của đặc tả Iceberg định nghĩa cách quản lý các bảng phân tích lớn bằng cách sử dụng các định dạng tệp bất biến: Parquet, Avro và ORC.

Tất cả các tệp dữ liệu và siêu dữ liệu phiên bản 1 đều hợp lệ sau khi nâng cấp bảng lên phiên bản 2. [Appendix E](https://iceberg.apache.org/spec/#version-2) mô tả cách thiết lập giá trị mặc định cho các trường phiên bản 2 khi đọc siêu dữ liệu phiên bản 1.

### Version 2: Row-level Deletes

Phiên bản 2 của đặc tả Iceberg bổ sung chức năng cập nhật và xóa ở cấp độ hàng cho các bảng phân tích có tệp bất biến (analytic tables with immutable files).

Thay đổi chính trong phiên bản 2 là bổ sung chức năng xóa tệp để mã hóa các hàng bị xóa trong các tệp dữ liệu hiện có. Phiên bản này có thể được sử dụng để xóa hoặc thay thế các hàng riêng lẻ trong các tệp dữ liệu bất biến mà không cần ghi đè lên các tệp đó.

Ngoài việc xóa dữ liệu ở cấp độ hàng, phiên bản 2 còn đưa ra một số yêu cầu khắt khe hơn đối với người ghi dữ liệu. Toàn bộ các thay đổi được liệt kê trong [Appendix E](https://iceberg.apache.org/spec/#version-2).

### Version 3: Extended Types and Capabilities

Phiên bản 3 của đặc tả Iceberg mở rộng các kiểu dữ liệu và cấu trúc metadata hiện có để bổ sung các khả năng mới.

- Kiểu dữ liệu mới: nanosecond timestamp(tz), unknown, variant, geometry, geography
- Hỗ trợ giá trị mặc định cho các cột
- Các phép biến đổi đa tham số dùng cho phân vùng và sắp xếp
- Row Lineage tracking
- Binary deletion vectors
- Khóa mã hóa bảng (Table encryption keys)

Toàn bộ các thay đổi được liệt kê trong [Appendix E](https://iceberg.apache.org/spec/#version-3).

### Version 4: Metadata Structure and Representation

Phiên bản 4 của đặc tả Iceberg tái cấu trúc metadata để cải thiện hiệu suất và bổ sung các khả năng mới.

- Hỗ trợ [relative locations](https://iceberg.apache.org/spec/#file-locations-in-metadata) trong metadata fields

Toàn bộ các thay đổi được liệt kê trong [Appendix E](https://iceberg.apache.org/spec/#version-4).

## Goals

- **Serializable isolation**: Các thao tác đọc (read) sẽ được tách biệt khỏi các thao tác ghi (write) đồng thời và luôn sử dụng commited snapshot của dữ liệu bảng. Các thao tác ghi sẽ hỗ trợ xóa và thêm tệp trong một lần thao tác duy nhất và không bao giờ hiển thị một phần. Người đọc sẽ không chiếm giữ khóa.
- Speed: Các thao tác sẽ sử dụng các lệnh gọi từ xa (remote calls) O(1) để lập kế hoạch các tệp cho quá trình quét và không phải O(n) trong đó n tăng theo kích thước của bảng, chẳng hạn như số lượng phân vùng hoặc tệp.
- **Scale**: Việc lập kế hoạch công việc sẽ chủ yếu do clients đảm nhiệm và không bị tắc nghẽn ở central metadata store. Metadata sẽ bao gồm thông tin cần thiết cho việc tối ưu hóa dựa trên chi phí.
- **Evolution**: Bảng sẽ hỗ trợ đầy đủ quá trình schema và partition spec evolution. Schema evolution hỗ trợ thêm, xóa, sắp xếp lại và đổi tên cột một cách an toàn, kể cả trong cấu trúc lồng nhau.
- **Storage separation**: Việc phân vùng sẽ được cấu hình trong bảng. Việc đọc dữ liệu sẽ được lập kế hoạch bằng cách sử dụng các điều kiện lọc trên giá trị dữ liệu, chứ không phải giá trị phân vùng. Các bảng sẽ hỗ trợ các partition schemes có thể thay đổi (evolving).
- **Formats**: Các định dạng tệp dữ liệu cơ bản sẽ hỗ trợ các quy tắc và kiểu schema evolution giống hệt nhau. Cả định dạng tối ưu hóa đọc (read-optimized) và tối ưu hóa ghi (write-optimized) đều sẽ có sẵn.

## Overview

![[iceberg_spec_overview.png]]

Định dạng bảng này theo dõi các tệp dữ liệu riêng lẻ trong một bảng thay vì các thư mục. Điều này cho phép người ghi tạo các tệp dữ liệu trực tiếp và chỉ thêm các tệp vào bảng khi có lệnh commit rõ ràng.

Trạng thái của bảng được lưu trữ trong các metadata file. Mọi thay đổi đối với trạng thái bảng đều tạo ra một tệp metadata mới và thay thế metadata cũ bằng một thao tác hoán đổi nguyên tử (atomic swap). The table metadata file theo dõi table schema, partitioning config, custom properties, and snapshots of the table contents. A snapshot thể hiện trạng thái của bảng tại một thời điểm nào đó và được sử dụng để truy cập toàn bộ tập hợp các tệp dữ liệu trong bảng.

Các tập tin dữ liệu trong snapshot được theo dõi bởi một hoặc nhiều tập tin kê khai (manifest files), mỗi tập tin chứa một hàng cho mỗi tập tin dữ liệu trong bảng, dữ liệu phân vùng của tập tin và các chỉ số của nó (metrics). Dữ liệu trong snapshot là sự kết hợp của tất cả các tập tin trong các tập tin kê khai (manifests) của nó. Các tập tin kê khai (Manifest files) được sử dụng lại trên nhiều snapshot để tránh ghi đè metadata ,thứ gây thay đổi chậm. Các tập tin kê khai (manifest files) có thể theo dõi các tập tin dữ liệu với bất kỳ tập hợp con nào của bảng và không được liên kết với các phân vùng.

Các tệp kê khai (manifests) tạo nên snapshot được lưu trữ trong một tệp danh sách tệp kê khai (manifest list file). Mỗi tệp danh sách tệp kê khai (manifest list) lưu trữ metadata về các tệp kê khai (manifests), bao gồm số liệu thống kê phân vùng và số lượng tệp dữ liệu. Các số liệu thống kê này được sử dụng để tránh đọc các tệp kê khai (manifests) không cần thiết cho một thao tác.

Xem thêm: [[illuminating]]

### Optimistic Concurrency

Việc atomic swap một table metadata file này với một table metadata file khác tạo cơ sở cho serializable isolation. Người đọc sử dụng snapshot hiện tại khi họ tải table metadata và không bị ảnh hưởng bởi các thay đổi cho đến khi họ làm mới và chọn vị trí metadata mới.

Người viết tạo các table metadata files một cách lạc quan (optimistically), giả định rằng phiên bản hiện tại sẽ không bị thay đổi trước khi writer thực hiện commit. Sau khi writer tạo bản cập nhật, họ sẽ commit bằng cách hoán đổi con trỏ tệp metadata của bảng từ phiên bản cơ sở sang phiên bản mới.

Nếu snapshot mà bản cập nhật dựa trên đó không còn chính xác, người ghi phải thử lại quá trình cập nhật dựa trên phiên bản chính xác mới hiện tại. Một số thao tác hỗ trợ thử lại bằng cách áp dụng lại các thay đổi metadata và commiting, trong những điều kiện được xác định rõ. Ví dụ, một thay đổi ghi đè lên các tệp có thể được áp dụng cho snapshot bảng mới nếu tất cả các tệp được ghi đè vẫn còn trong bảng.

Các điều kiện cần thiết để thao tác ghi thành công sẽ xác định isolation level. Người ghi có thể chọn những gì cần xác thực và có thể đưa ra các đảm bảo isolation khác nhau.

### Sequence Numbers

Tuổi đời tương đối của dữ liệu và các tập tin đã xóa phụ thuộc vào một số thứ tự được gán cho mỗi lần commit thành công. Khi một snapshot được tạo cho một lần commit, nó được gán một cách lạc quan (optimistically) số thứ tự tiếp theo (next sequence number) và số này được ghi vào metadata của snapshot. Nếu lần commit thất bại và phải thử lại, số thứ tự sẽ được gán lại và ghi vào metadata của snapshot mới.

Tất cả các tệp kê khai (manifests), tệp dữ liệu (data files) và tệp xóa (delete files) được tạo cho một snapshot đều kế thừa số thứ tự của snapshot đó. Metadata của tệp kê khai (manifests) trong danh sách kê khai (manifest list) lưu trữ số thứ tự của tệp kê khai. Các mục nhập tệp dữ liệu và metadata mới được ghi với giá trị `null` thay cho số thứ tự, và số thứ tự này sẽ được thay thế bằng số thứ tự của tệp kê khai (manifest) tại thời điểm đọc. Khi một tệp dữ liệu hoặc tệp xóa được ghi vào một tệp kê khai mới (dưới dạng "existing"), số thứ tự thừa kế được ghi lại để đảm bảo nó không thay đổi sau khi được thừa kế lần đầu.

Việc kế thừa số thứ tự từ metadata của manifest cho phép ghi một manifest mới một lần và tái sử dụng nó trong các lần thử lại commit. Để thay đổi số thứ tự cho một lần thử lại, chỉ cần ghi lại danh sách manifest, điều này dù sao cũng sẽ được ghi lại với bộ manifest mới nhất.

### Row-level Deletes

Các thao tác xóa ở cấp độ hàng được lưu trữ trong các tệp xóa.

Có hai loại thao tác xóa ở cấp độ hàng:

- **Position deletes**: Đánh dấu hàng bị xóa bằng đường dẫn tệp dữ liệu và vị trí hàng trong tệp dữ liệu. Việc xóa theo vị trí được mã hóa trong tệp xóa theo vị trí ([_position delete file_](https://iceberg.apache.org/spec/#position-delete-files)) (V2) hoặc [_deletion vector_](https://iceberg.apache.org/spec/#deletion-vectors) (V3 trở lên).
- **Equality deletes**: Đánh dấu một hàng bị xóa dựa trên một hoặc nhiều giá trị cột, ví dụ như id = 5. Các thao tác xóa bằng nhau được mã hóa trong tệp xóa bằng nhau ([_equality delete file_](https://iceberg.apache.org/spec/#equality-delete-files)).

Giống như các tập tin dữ liệu, các tập tin xóa được theo dõi theo phân vùng. Nói chung, một tập tin xóa phải được áp dụng cho các tập tin dữ liệu cũ hơn có cùng phân vùng; xem [Scan Planning](https://iceberg.apache.org/spec/#scan-planning) để biết chi tiết. Các chỉ số cột có thể được sử dụng để xác định xem các hàng của tập tin xóa có trùng lặp với nội dung của tập tin dữ liệu hoặc phạm vi quét hay không.

### File System Operations

Iceberg chỉ yêu cầu hệ thống tệp (file systems) hỗ trợ các thao tác sau:

- **In-place write**: Các tập tin sẽ không được di chuyển hoặc sửa đổi sau khi đã được ghi.
- **Seekable reads**: Các định dạng tệp dữ liệu yêu cầu hỗ trợ tìm kiếm
- **Deletes**: Bảng sẽ xóa các tệp không còn được sử dụng nữa.

Các yêu cầu này tương thích với các kho lưu trữ đối tượng, chẳng hạn như S3.

Bảng dữ liệu không yêu cầu ghi truy cập ngẫu nhiên. Sau khi được ghi, các tệp dữ liệu và metadata sẽ không thể thay đổi cho đến khi chúng bị xóa.

Các bảng không cần phải đổi tên, ngoại trừ các bảng sử dụng chức năng đổi tên nguyên tử (atomic) để thực hiện thao tác commit cho các tệp metadata mới.

### File Locations in Metadata

Tất cả các trường vị trí trong định dạng phiên bản 3 trở xuống đều chứa đường dẫn đầy đủ.

Phiên bản 4 của đặc tả Iceberg bổ sung hỗ trợ cho các vị trí tương đối trong metadata, cho phép di chuyển các bảng mà không cần ghi đè lên các tệp metadata. Các vị trí tương đối được cho phép trong tất cả các trường vị trí được theo dõi trong metadata và được giải quyết dựa trên vị trí cơ sở của bảng. Vị trí của bảng có thể được cố định trong metadata của bảng hoặc được suy luận, nhưng mục đích là để được quản lý và cung cấp bởi một catalog. Các yêu cầu về việc tương đối hóa và giải quyết được nêu trong mục Đường dẫn trong [Paths in Metadata](https://iceberg.apache.org/spec/#paths-in-metadata).

## Specification

### Terms

- **Schema** -- Tên và kiểu dữ liệu của các trường trong bảng.
- **Partition spec** -- Định nghĩa về cách các giá trị phân vùng được tạo ra từ các trường dữ liệu
- **Snapshot** -- Trạng thái của một bảng tại một thời điểm nhất định, bao gồm toàn bộ tập hợp các tệp dữ liệu.
- **Manifest list** -- Một tập tin liệt kê các tập manifests files; một tập tin cho mỗi snapshot.
- **Manifest** -- Một tập tin liệt kê dữ liệu hoặc delete files; một tập hợp con của snapshot.
- **Data file** -- Một tập tin chứa các hàng của một bảng.
- **Delete file** -- Một tệp mã hóa các hàng của bảng được xóa theo vị trí hoặc giá trị dữ liệu.
- **Absolute path** -- Một chuỗi đường dẫn bao gồm [URI](https://datatracker.ietf.org/doc/html/rfc3986#section-3.1) scheme và có thể được sử dụng trực tiếp.
- **Relative path** -- Một chuỗi đường dẫn không có URI scheme cần được giải quyết ([resolved](https://iceberg.apache.org/spec/#path-resolution)) dựa trên vị trí bảng.

### Writer requirements[🔗](https://iceberg.apache.org/spec/#writer-requirements "Permanent link")

Một số bảng trong tài liệu đặc tả này có các cột chỉ định yêu cầu đối với bảng theo phiên bản. Các yêu cầu này dành cho người ghi dữ liệu khi thêm các tệp metadata (bao gồm tệp kê khai (manifests files) và danh sách kê khai (manifest lists)) vào bảng với phiên bản đã cho.

| Requirement | Write behavior                                        |
| ----------- | ----------------------------------------------------- |
| (blank)     | Trường này nên được bỏ qua.                           |
| _optional_  | Bạn có thể điền thông tin vào trường này hoặc bỏ qua. |
| _required_  | Trường này phải được viết                             |
Người đọc nên khoan dung hơn vì các tệp v1 metadata files được cho phép trong các bảng v2 (hoặc các phiên bản sau này) để các bảng có thể được nâng cấp mà không cần viết lại cây metadata. Đối với danh sách manifest và các tệp manifest, bảng này hiển thị hành vi đọc dự kiến ​​cho các phiên bản sau này.

|v1|v2|v2+ read behavior|
|---|---|---|
||_optional_|Read the field as _optional_|
||_required_|Read the field as _optional_; it may be missing in v1 files|
|_optional_||Ignore the field|
|_optional_|_optional_|Read the field as _optional_|
|_optional_|_required_|Read the field as _optional_; it may be missing in v1 files|
|_required_||Ignore the field|
|_required_|_optional_|Read the field as _optional_|
|_required_|_required_|Fill in a default or throw an exception if the field is missing|

...

#### Writing data files[🔗](https://iceberg.apache.org/spec/#writing-data-files "Permanent link")
...

### Paths in Metadata[🔗](https://iceberg.apache.org/spec/#paths-in-metadata "Permanent link")

...

#### Path Resolution[🔗](https://iceberg.apache.org/spec/#path-resolution "Permanent link")

...

#### Path Relativization[🔗](https://iceberg.apache.org/spec/#path-relativization "Permanent link")

...

#### Table Location Specification[🔗](https://iceberg.apache.org/spec/#table-location-specification "Permanent link")

...

### Schemas and Data Types

Schema của một bảng là một danh sách các cột được đặt tên. Tất cả các kiểu dữ liệu đều là kiểu dữ liệu nguyên thủy hoặc kiểu lồng nhau, bao gồm maps, lists hoặc structs. Schema của một bảng cũng là một kiểu cấu trúc.

Để biết cách biểu diễn các kiểu dữ liệu này trong các định dạng tệp Avro, ORC và Parquet, xem Appendix A.

#### Nested Types[🔗](https://iceberg.apache.org/spec/#nested-types "Permanent link")

#### Semi-structured Types[🔗](https://iceberg.apache.org/spec/#semi-structured-types "Permanent link")

#### Primitive Types[🔗](https://iceberg.apache.org/spec/#primitive-types "Permanent link")

#### Default values[🔗](https://iceberg.apache.org/spec/#default-values "Permanent link")

#### Schema Evolution[🔗](https://iceberg.apache.org/spec/#schema-evolution "Permanent link")

#### Identifier Field IDs[🔗](https://iceberg.apache.org/spec/#identifier-field-ids "Permanent link")

#### Reserved Field IDs[🔗](https://iceberg.apache.org/spec/#reserved-field-ids "Permanent link")

#### Row Lineage[🔗](https://iceberg.apache.org/spec/#row-lineage "Permanent link")

### Partitioning

Các tệp dữ liệu được lưu trữ trong các manifest với một bộ giá trị phân vùng được sử dụng trong các quá trình quét để lọc ra các tệp không chứa bản ghi phù hợp với điều kiện lọc của quá trình quét. Giá trị phân vùng cho một tệp dữ liệu phải giống nhau đối với tất cả các bản ghi được lưu trữ trong tệp dữ liệu đó. (Manifest lưu trữ các tệp dữ liệu từ bất kỳ phân vùng nào, miễn là thông số kỹ thuật phân vùng giống nhau đối với các tệp dữ liệu.)

Các bảng được cấu hình với một partition spec xác định cách tạo ra một bộ giá trị phân vùng từ một bản ghi. Partition spec có một danh sách các trường bao gồm:

- ID cột nguồn hoặc danh sách ID cột nguồn từ schema của bảng.
- **Partition field id** được sử dụng để xác định trường phân vùng và là duy nhất trong phạm vi một partition spec. Trong metadata bảng v2, mã này là duy nhất trên tất cả các partition spec.
- Một phép biến đổi được áp dụng cho cột nguồn để tạo ra giá trị phân vùng.
- Tên phân vùng

Các cột nguồn, được chọn bằng ID, phải là kiểu dữ liệu nguyên thủy và không được chứa trong một map hoặc list, nhưng có thể được lồng trong một struct. Để biết chi tiết về cách chuyển đổi partition spec thành JSON, xem Appendix C.

Thông số phân vùng ghi lại quá trình chuyển đổi từ dữ liệu bảng sang giá trị phân vùng. Điều này được sử dụng để chuyển đổi các điều kiện lọc thành điều kiện lọc phân vùng, ngoài việc chuyển đổi các giá trị dữ liệu. Việc suy ra các điều kiện lọc phân vùng từ các điều kiện lọc cột trên dữ liệu bảng được sử dụng để tách biệt các truy vấn logic khỏi bộ nhớ vật lý: việc phân vùng có thể thay đổi và các bộ lọc phân vùng chính xác luôn được suy ra từ các điều kiện lọc cột. Điều này giúp đơn giản hóa các truy vấn vì người dùng không cần phải cung cấp cả vị từ logic và vị từ phân vùng. Để biết thêm thông tin, hãy xem phần Lập kế hoạch quét bên dưới.

Các trường phân vùng sử dụng phép biến đổi không xác định có thể được đọc bằng cách bỏ qua trường phân vùng đó nhằm mục đích lọc các tệp dữ liệu trong quá trình lập kế hoạch quét. Trong phiên bản v1 và v2, trình đọc nên bỏ qua các trường có phép biến đổi không xác định trong khi đọc; hành vi này là bắt buộc trong phiên bản v3. Trình ghi không được phép commit dữ liệu bằng cách sử dụng thông số kỹ thuật phân vùng chứa trường có phép biến đổi không xác định.

Hai cấu hình phân vùng được coi là tương đương nếu chúng có cùng số lượng trường và đối với mỗi trường tương ứng, các trường đó có cùng ID cột nguồn, định nghĩa chuyển đổi và tên phân vùng. Người ghi không được tạo cấu hình phân vùng mới nếu đã tồn tại một cấu hình phân vùng tương thích được định nghĩa trong bảng.

ID trường phân vùng phải được sử dụng lại nếu thông số kỹ thuật phân vùng hiện có chứa trường tương đương.

#### Partition Transforms

| Transform name    | Description                                                  | Source types                                                                                                                                | Result type          |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- |
| **`identity`**    | Source value, unmodified                                     | Any except for `geometry`, `geography`, and `variant`                                                                                       | Source type          |
| **`bucket[N]`**   | Hash of value, mod `N` (see below)                           | `int`, `long`, `decimal`, `date`, `time`, `timestamp`, `timestamptz`, `timestamp_ns`, `timestamptz_ns`, `string`, `uuid`, `fixed`, `binary` | `int`                |
| **`truncate[W]`** | Value truncated to width `W` (see below)                     | `int`, `long`, `decimal`, `string`, `binary`                                                                                                | Source type          |
| **`year`**        | Extract a date or timestamp year, as years from 1970         | `date`, `timestamp`, `timestamptz`, `timestamp_ns`, `timestamptz_ns`                                                                        | `int`                |
| **`month`**       | Extract a date or timestamp month, as months from 1970-01-01 | `date`, `timestamp`, `timestamptz`, `timestamp_ns`, `timestamptz_ns`                                                                        | `int`                |
| **`day`**         | Extract a date or timestamp day, as days from 1970-01-01     | `date`, `timestamp`, `timestamptz`, `timestamp_ns`, `timestamptz_ns`                                                                        | `int`                |
| **`hour`**        | Extract a timestamp hour, as hours from 1970-01-01 00:00:00  | `timestamp`, `timestamptz`, `timestamp_ns`, `timestamptz_ns`                                                                                | `int`                |
| **`void`**        | Always produces `null`                                       | Any                                                                                                                                         | Source type or `int` |

Tất cả các phép biến đổi phải trả về null nếu giá trị đầu vào là null.

Phép biến đổi void có thể được sử dụng để thay thế phép biến đổi trong trường phân vùng hiện có, sao cho trường đó thực chất bị loại bỏ trong các bảng v1. Xem phần tiến hóa phân vùng bên dưới.

#### Bucket Transform Details[🔗](https://iceberg.apache.org/spec/#bucket-transform-details "Permanent link")

#### Truncate Transform Details[🔗](https://iceberg.apache.org/spec/#truncate-transform-details "Permanent link")

#### Partition Evolution[🔗](https://iceberg.apache.org/spec/#partition-evolution "Permanent link")

### Sorting

Người dùng có thể sắp xếp dữ liệu của họ trong các phân vùng theo cột để cải thiện hiệu suất. Thông tin về cách dữ liệu được sắp xếp có thể được khai báo cho từng tệp dữ liệu hoặc tệp xóa, theo thứ tự sắp xếp.

Thứ tự sắp xếp được xác định bởi một ID thứ tự sắp xếp và một danh sách các trường sắp xếp. Thứ tự của các trường sắp xếp trong danh sách xác định thứ tự áp dụng việc sắp xếp cho dữ liệu. Mỗi trường sắp xếp bao gồm:

- ID cột nguồn hoặc danh sách ID cột nguồn từ schema của bảng.
- Phép biến đổi được sử dụng để tạo ra các giá trị dùng để sắp xếp từ cột nguồn. Đây là phép biến đổi tương tự như được mô tả trong [partition transforms](https://iceberg.apache.org/spec/#partition-transforms).
- **sort direction** chỉ có thể là tăng dần hoặc giảm dần.
- Một **null order** mô tả thứ tự của các giá trị null khi được sắp xếp. Chỉ có thể là nulls-first hoặc nulls-last.

Để biết thêm chi tiết về cách chuyển đổi thứ tự sắp xếp thành JSON, vui lòng tham khảo JSON, xem Appendix C.

Sắp xếp id 0 được dành riêng cho row chưa được sắp xếp.

Việc sắp xếp các số thực sẽ tạo ra kết quả như sau: NaN < -Infinity < -value < -0 < 0 < value < Infinity < NaN. Điều này phù hợp với cách triển khai so sánh các kiểu dữ liệu số thực trong Java.

Tệp dữ liệu hoặc tệp xóa được liên kết với một thứ tự sắp xếp bằng ID của thứ tự sắp xếp đó trong [a manifest](https://iceberg.apache.org/spec/#manifests). Do đó, bảng phải khai báo tất cả các thứ tự sắp xếp để tra cứu. Bảng cũng có thể được cấu hình với ID thứ tự sắp xếp mặc định, cho biết dữ liệu mới nên được sắp xếp như thế nào theo mặc định. Người ghi nên sử dụng thứ tự sắp xếp mặc định này để sắp xếp dữ liệu khi ghi, nhưng không bắt buộc nếu việc sử dụng thứ tự mặc định quá tốn kém, như trường hợp ghi dữ liệu theo luồng.

Xem thêm: [[illuminating]]

### Manifests

Manifest là một tệp Avro bất biến liệt kê các lists data files hoặc delete files, cùng với bộ file’s partition data, metrics và thông tin theo dõi của mỗi tệp. Một hoặc nhiều manifest files được sử dụng để lưu trữ [snapshot](https://iceberg.apache.org/spec/#snapshots), theo dõi tất cả các tệp trong một bảng tại một thời điểm nhất định. Manifests được theo dõi bởi một [manifest list](https://iceberg.apache.org/spec/#manifest-lists) cho mỗi snapshot của bảng.

Manifest là một tệp dữ liệu Iceberg hợp lệ: các tệp phải sử dụng định dạng, schema và column projection hợp lệ của Iceberg.

Một manifest có thể lưu trữ các tệp dữ liệu hoặc các tệp xóa, nhưng không thể cả hai vì manifests chứa tệp xóa sẽ được quét trước trong quá trình lập kế hoạch công việc. Thông tin về việc một manifest là data manifest hay delete manifest được lưu trữ trong manifest metadata.

Manifest lưu trữ các tệp cho một thông số phân vùng duy nhất. Khi thông số phân vùng của bảng thay đổi, các tệp cũ vẫn được giữ lại trong manifest cũ và các tệp mới hơn được ghi vào manifest mới. Điều này là cần thiết vì schema của manifest dựa trên thông số phân vùng của nó (xem bên dưới). Thông số phân vùng của mỗi manifest cũng được sử dụng để chuyển đổi các điều kiện trên các hàng dữ liệu của bảng thành các điều kiện trên các giá trị phân vùng được sử dụng trong quá trình lập kế hoạch công việc để chọn các tệp từ manifest.

Manifest file phải lưu trữ thông số phân vùng và metadata khác dưới dạng thuộc tính trong Avro file's key-value metadata.

| v1         | v2 and v3  | Key                 | Value                                                                                                                                                                       |
| ---------- | ---------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _required_ | _required_ | `schema`            | JSON representation of the table schema at the time the manifest was written                                                                                                |
| _optional_ | _required_ | `schema-id`         | ID of the schema used to write the manifest as a string                                                                                                                     |
| _required_ | _required_ | `partition-spec`    | JSON representation of only the partition fields array of the partition spec used to write the manifest. See [Appendix C](https://iceberg.apache.org/spec/#partition-specs) |
| _optional_ | _required_ | `partition-spec-id` | ID of the partition spec used to write the manifest as a string                                                                                                             |
| _optional_ | _required_ | `format-version`    | Table format version number of the manifest as a string                                                                                                                     |
|            | _required_ | `content`           | Type of content files tracked by the manifest: "data" or "deletes"                                                                                                          |

Cấu trúc của manifest được định nghĩa bởi cấu trúc manifest_entry, được mô tả trong phần tiếp theo.

#### Manifest Entry Fields

...

Xem thêm: [[illuminating]]

### Snapshots

Snapshot bao gồm các trường sau:

|v1|v2|v3|Field|Description|
|---|---|---|---|---|
|_required_|_required_|_required_|**`snapshot-id`**|A unique long ID|
|_optional_|_optional_|_optional_|**`parent-snapshot-id`**|The snapshot ID of the snapshot's parent. Omitted for any snapshot with no parent|
||_required_|_required_|**`sequence-number`**|A monotonically increasing long that tracks the order of changes to a table|
|_required_|_required_|_required_|**`timestamp-ms`**|A timestamp when the snapshot was created, used for garbage collection and table inspection|
|_optional_|_required_|_required_|**`manifest-list`**|The location of a manifest list for this snapshot that tracks manifest files with additional metadata|
|_optional_|||**`manifests`**|A list of manifest file locations. Must be omitted if `manifest-list` is present|
|_optional_|_required_|_required_|**`summary`**|A string map that summarizes the snapshot changes, including `operation` as a _required_ field (see below)|
|_optional_|_optional_|_optional_|**`schema-id`**|ID of the table's current schema when the snapshot was created|
|||_required_|**`first-row-id`**|The first `_row_id` assigned to the first row in the first data file in the first manifest, see [Row Lineage](https://iceberg.apache.org/spec/#row-lineage)|
|||_required_|**`added-rows`**|The upper bound of the number of rows with assigned row IDs, see [Row Lineage](https://iceberg.apache.org/spec/#row-lineage)|
|||_optional_|**`key-id`**|ID of the encryption key that encrypts the manifest list key metadata|

### Manifest Lists

Snapshot được nhúng trong metadata của bảng, nhưng list of manifests cho một snapshot được lưu trữ trong một tệp manifest list riêng biệt.

Một manifest list mới được ghi lại cho mỗi lần cố gắng lưu snapshot vì list of manifest luôn thay đổi để tạo ra snapshot mới. Khi một manifest list được ghi, the (optimistic) sequence number của snapshot sẽ được ghi cho tất cả các tệp manifest mới được theo dõi bởi danh sách đó.

Manifest list bao gồm metadata tóm tắt có thể được sử dụng để tránh quét tất cả các manifest trong snapshot khi lập kế hoạch quét bảng. Điều này bao gồm số lượng tệp được thêm, hiện có và đã xóa, cũng như tóm tắt các giá trị cho từng trường của thông số kỹ thuật phân vùng được sử dụng để ghi manifest.

Manifest list là một tệp dữ liệu Iceberg hợp lệ: các tệp phải sử dụng định dạng, schema và column projection của Iceberg.

Manifest list files lưu trữ `manifest_file`, một cấu trúc với các trường sau:

|v1|v2|v3|Field id, name|Type|Description|
|---|---|---|---|---|---|
|_required_|_required_|_required_|**`500 manifest_path`**|`string`|Location of the manifest file|
|_required_|_required_|_required_|**`501 manifest_length`**|`long`|Length of the manifest file in bytes|
|_required_|_required_|_required_|**`502 partition_spec_id`**|`int`|ID of a partition spec used to write the manifest; must be listed in table metadata `partition-specs`|
||_required_|_required_|**`517 content`**|`int` with meaning: `0: data`, `1: deletes`|The type of files tracked by the manifest, either data or delete files; 0 for all v1 manifests|
||_required_|_required_|**`515 sequence_number`**|`long`|The sequence number when the manifest was added to the table; use 0 when reading v1 manifest lists|
||_required_|_required_|**`516 min_sequence_number`**|`long`|The minimum data sequence number of all live data or delete files in the manifest; use 0 when reading v1 manifest lists|
|_required_|_required_|_required_|**`503 added_snapshot_id`**|`long`|ID of the snapshot where the manifest file was added|
|_optional_|_required_|_required_|**`504 added_files_count`**|`int`|Number of entries in the manifest that have status `ADDED` (1), when `null` this is assumed to be non-zero|
|_optional_|_required_|_required_|**`505 existing_files_count`**|`int`|Number of entries in the manifest that have status `EXISTING` (0), when `null` this is assumed to be non-zero|
|_optional_|_required_|_required_|**`506 deleted_files_count`**|`int`|Number of entries in the manifest that have status `DELETED` (2), when `null` this is assumed to be non-zero|
|_optional_|_required_|_required_|**`512 added_rows_count`**|`long`|Number of rows in all of files in the manifest that have status `ADDED`, when `null` this is assumed to be non-zero|
|_optional_|_required_|_required_|**`513 existing_rows_count`**|`long`|Number of rows in all of files in the manifest that have status `EXISTING`, when `null` this is assumed to be non-zero|
|_optional_|_required_|_required_|**`514 deleted_rows_count`**|`long`|Number of rows in all of files in the manifest that have status `DELETED`, when `null` this is assumed to be non-zero|
|_optional_|_optional_|_optional_|**`507 partitions`**|`list<508: field_summary>` **(see below)**|A list of field summaries for each partition field in the spec. Each field in the list corresponds to a field in the manifest file’s partition spec.|
|_optional_|_optional_|_optional_|**`519 key_metadata`**|`binary`|Implementation-specific key metadata for encryption|
|||_optional_|**`520 first_row_id`**|`long`|The starting `_row_id` to assign to rows added by `ADDED` data files [First Row ID Assignment](https://iceberg.apache.org/spec/#first-row-id-assignment)|

`field_summary` là một cấu trúc với các trường sau:

|v1|v2 and v3|Field id, name|Type|Description|
|---|---|---|---|---|
|_required_|_required_|**`509 contains_null`**|`boolean`|Whether the manifest contains at least one partition with a null value for the field|
|_optional_|_optional_|**`518 contains_nan`**|`boolean`|Whether the manifest contains at least one partition with a NaN value for the field|
|_optional_|_optional_|**`510 lower_bound`**|`bytes` [1]|Lower bound for the non-null, non-NaN values in the partition field, or null if all values are null or NaN [2]|
|_optional_|_optional_|**`511 upper_bound`**|`bytes` [1]|Upper bound for the non-null, non-NaN values in the partition field, or null if all values are null or NaN [2]|

1. Giới hạn dưới và giới hạn trên được serialized thành byte bằng cách sử dụng phương pháp single-object serialization trong Appendix D. Kiểu dữ liệu được sử dụng để mã hóa giá trị là kiểu dữ liệu của trường phân vùng.
2. Nếu 0.0 là giá trị của trường phân vùng, thì lower_bound không được lớn hơn +0.0, và nếu +0.0 là giá trị của trường phân vùng, thì upper_bound không được lớn hơn 0.0.

#### First Row ID Assignment

...

Xem thêm: [[illuminating]]

### Scan Planning

Xem thêm: [[illuminating]]

Nguồn: https://iceberg.apache.org/spec/