- [[#Versions|Versions]]
	- [[#Versions#Version 2:|Version 2:]]
- [[#1. Bối cảnh: "Immutable files" là gì và vấn đề ở V1?|1. Bối cảnh: "Immutable files" là gì và vấn đề ở V1?]]
- [[#2. Giải pháp của Version 2: "Delete Files" (Xóa ở cấp độ dòng)|2. Giải pháp của Version 2: "Delete Files" (Xóa ở cấp độ dòng)]]
- [[#3. Ý cuối: "Requirements stricter for writers" nghĩa là gì?|3. Ý cuối: "Requirements stricter for writers" nghĩa là gì?]]
- [[#Overview|Overview]]
	- [[#Overview#1. Bản chất thay đổi: Quản lý theo "File" thay vì "Thư mục"|1. Bản chất thay đổi: Quản lý theo "File" thay vì "Thư mục"]]
	- [[#Overview#2. Chi tiết 3 Tầng Metadata của Apache Iceberg|2. Chi tiết 3 Tầng Metadata của Apache Iceberg]]
	- [[#Overview#Tầng 1: Table Metadata File (`.metadata.json`)|Tầng 1: Table Metadata File (`.metadata.json`)]]
	- [[#Overview#Tầng 2: Manifest List File (`snap-*.avro`)|Tầng 2: Manifest List File (`snap-*.avro`)]]
	- [[#Overview#Tầng 3: Manifest File (`*.avro`)|Tầng 3: Manifest File (`*.avro`)]]
- [[#Sorting|Sorting]]
	- [[#Sorting#1. Định nghĩa một "Thứ tự sắp xếp" (Sort Order)|1. Định nghĩa một "Thứ tự sắp xếp" (Sort Order)]]
	- [[#Sorting#2. Quy tắc sắp xếp số thập phân (Floating-point)|2. Quy tắc sắp xếp số thập phân (Floating-point)]]
	- [[#Sorting#3. Cách Iceberg quản lý và áp dụng Sort Order|3. Cách Iceberg quản lý và áp dụng Sort Order]]
- [[#Manifest|Manifest]]
- [[#Manifest list|Manifest list]]
- [[#Scan Planning|Scan Planning]]
- [[#Table Metadata|Table Metadata]]


## Versions

### Version 2:

## 1. Bối cảnh: "Immutable files" là gì và vấn đề ở V1?

- **Immutable files (Các file không thể sửa đổi):** Trong các hệ thống lưu trữ dữ liệu lớn (như HDFS, S3, GCS), dữ liệu thường được lưu dưới dạng các file nén lớn (như Parquet hoặc ORC). Một khi file này đã được ghi xong, nó là **bất biến** (immutable) — bạn không thể mở nó ra để sửa một vài dòng rồi lưu lại được.
    
- **Vấn đề ở V1 (Copy-on-Write):** Ở Iceberg phiên bản 1, nếu bạn muốn XÓA hoặc CẬP NHẬT (Update) dù chỉ **1 dòng** dữ liệu nằm trong một file Parquet nặng 500MB, Iceberg sẽ phải đọc toàn bộ file 500MB đó lên, lọc bỏ dòng cần xóa (hoặc sửa dòng cần update), rồi **ghi lại thành một file 500MB hoàn toàn mới**. Quá trình này gọi là _Copy-on-Write_, nó cực kỳ tốn tài nguyên và làm chậm hệ thống khi cần xử lý dữ liệu thời gian thực.

## 2. Giải pháp của Version 2: "Delete Files" (Xóa ở cấp độ dòng)

Thay vì phải ghi đè lại cả một file dữ liệu lớn, Version 2 của Iceberg giới thiệu một cơ chế thông minh hơn gọi là **Merge-on-Read**, sử dụng các **Delete files (File đánh dấu xóa)**.

Thay vì sửa trực tiếp vào file dữ liệu cũ, Iceberg V2 làm như sau:

- **Khi có lệnh Xóa (Delete):** Iceberg **không đụng vào file dữ liệu cũ**. Nó chỉ ghi một file nhỏ mới gọi là **Delete File**. File này giống như một "sổ đen" ghi lại: _"Trong file dữ liệu A, dòng số 5 đã bị xóa nhé"_.
    
- **Khi có lệnh Cập nhật (Update):** Bản chất của Update là Xóa dòng cũ và Thêm dòng mới. Iceberg V2 sẽ ghi địa chỉ dòng cũ vào _Delete File_, và ghi dữ liệu mới cập nhật vào một _Data File_ mới.
    
- **Khi bạn Đọc dữ liệu (Read):** Trình truy vấn (như Spark, Trino, BigQuery) sẽ đọc song song cả file dữ liệu gốc lẫn file "sổ đen" (Delete file) kia, tự động loại bỏ các dòng đã bị đánh dấu xóa rồi trả ra kết quả cuối cùng cho bạn.

## 3. Ý cuối: "Requirements stricter for writers" nghĩa là gì?

"Nâng cao tiêu chuẩn/yêu cầu khắt khe hơn đối với các Writer (Trình ghi dữ liệu)": Vì kiến trúc V2 phức tạp hơn (vừa phải quản lý file dữ liệu, vừa phải quản lý file đánh dấu xóa, kiểm soát xem dòng nào khớp với dòng nào dựa trên ID), nên các công cụ dùng để ghi dữ liệu vào Iceberg (như Apache Spark, Flink) buộc phải tuân thủ các quy tắc nghiêm ngặt hơn rất nhiều so với V1.

Họ phải đảm bảo ghi đúng định dạng của các Delete File (theo dạng _Position Delete_ hoặc _Equality Delete_) và tuân thủ các quy định khắt khe được liệt kê cụ thể trong **Appendix E (Phụ lục E)** của tài liệu kỹ thuật Iceberg để tránh làm sai lệch dữ liệu khi người khác đọc.

## Overview

### 1. Bản chất thay đổi: Quản lý theo "File" thay vì "Thư mục"

Trong kiến trúc cũ (Hive), khi bạn thực hiện câu lệnh ghi dữ liệu, kết quả sẽ được ghi trực tiếp vào một thư mục (directory). Nếu một câu lệnh đang ghi dở mà bị sập, thư mục đó sẽ chứa dữ liệu "rác" (partial data), dẫn đến việc người khác vào đọc sẽ bị sai.
Iceberg thay đổi cuộc chơi bằng cách:

- **Ghi in-place (tại chỗ):** Các file dữ liệu mới (`.parquet`, `.orc`, `.avro`) cứ việc ghi thoải mái xuống Storage (MinIO/S3). Nhưng lúc này, hệ thống **chưa ai nhìn thấy** các file này cả.

- **Atomic Commit (Cam kết nguyên tử):** Chỉ khi toàn bộ các file được ghi xong xuôi, một hành động "Commit" được kích hoạt để cập nhật trạng thái bảng. Nếu commit thành công, dữ liệu mới chính thức xuất hiện. Nếu commit lỗi, các file vừa ghi sẽ bị bỏ qua (đảm bảo tính ACID).

### 2. Chi tiết 3 Tầng Metadata của Apache Iceberg

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

## Sorting

### 1. Định nghĩa một "Thứ tự sắp xếp" (Sort Order)

> _"A sort order is defined by a sort order id and a list of sort fields..."_

Để máy tính hiểu được bạn muốn sắp xếp dữ liệu như thế nào, Iceberg định nghĩa một khái niệm gọi là **Sort Order**. Mỗi một Sort Order sẽ có một mã định danh (**Sort Order ID**) và danh sách các cột cần sắp xếp theo thứ tự ưu tiên.

Mỗi cột tham gia vào việc sắp xếp (Sort Field) sẽ bao gồm 4 thành phần:

1. **Source column id:** ID của cột gốc trong bảng (ví dụ: cột `customer_id`).
    
2. **Transform (Hàm biến đổi):** Bạn có thể sắp xếp dựa trên dữ liệu đã qua biến đổi chứ không nhất thiết phải là dữ liệu thô. (Ví dụ: Biến đổi cột `timestamp` thành `date` rồi mới sắp xếp).
    
3. **Sort direction (Hướng sắp xếp):** Chỉ có thể là tăng dần (`asc` - Ascending) hoặc giảm dần (`desc` - Descending).
    
4. **Null order (Thứ tự của giá trị rỗng):** Quy định các dòng bị khuyết dữ liệu (`null`) sẽ nằm ở đâu. Chỉ có 2 lựa chọn: Đứng đầu bảng (`nulls-first`) hoặc đứng cuối bảng (`nulls-last`).
    

> [!note] 
> **Lưu ý đặc biệt:** Mã ID số `0` (`Order id 0`) được Iceberg giữ riêng để quy định cho dữ liệu **không sắp xếp** (unsorted).

### 2. Quy tắc sắp xếp số thập phân (Floating-point)

> _"Sorting floating-point numbers should produce the following behavior..."_

Đối với các số thập phân (Float/Double), đôi khi sẽ có các giá trị đặc biệt như Số âm, Số dương, Vô cực (`Infinity`), hoặc Lỗi không phải là số (`NaN` - Not a Number). Iceberg quy định thứ tự từ nhỏ đến lớn chuẩn theo ngôn ngữ lập trình Java như sau:

−NaN<−Infinity<−value<−0<0<value<Infinity<`NaN`

(Giá trị −NaN là nhỏ nhất, và giá trị NaN dương là lớn nhất).

### 3. Cách Iceberg quản lý và áp dụng Sort Order

> _"A data or delete file is associated with a sort order by the sort order's id within a manifest..."_

- **Lưu trữ ở đâu?:** Thông tin về việc một file dữ liệu đã được sắp xếp theo chuẩn nào sẽ **không** ghi vào bản thân file dữ liệu đó, mà được ghi ở tầng **Manifest File** thông qua cái mã `Sort Order ID`. Do đó, bảng Iceberg phải lưu lại danh sách tất cả các kiểu Sort Order từng tồn tại để đối chiếu.
    
- **Cấu hình mặc định (Default Sort Order):** Bạn có thể cài đặt một kiểu sắp xếp mặc định cho bảng. Khi có dữ liệu mới ghi vào, các tiến trình ghi (Writers) sẽ nhìn vào đây để tự động sắp xếp dữ liệu trước khi lưu xuống đĩa.

> [!note]
> Trường hợp ngoại lệ (Streaming ghi dữ liệu liên tục)
> _"...Writers should use this default sort order to sort the data on write, but are not required to if the default order is prohibitively expensive, as it would be for streaming writes."_

Việc sắp xếp dữ liệu khi ghi tốn rất nhiều RAM và CPU. Do đó, Iceberg **không bắt buộc** lúc nào cũng phải sắp xếp.

Đối với các hệ thống **Streaming** (dữ liệu đổ về liên tục từng giây theo thời gian thực), việc bắt hệ thống dừng lại để sắp xếp rồi mới ghi sẽ làm nghẽn cổ chai (quá đắt đỏ - _prohibitively expensive_). Trong trường hợp này, Writer được phép ghi thẳng dữ liệu xuống dạng "chưa sắp xếp" (`Order id 0`) để ưu tiên tốc độ, việc sắp xếp sẽ được xử lý sau (bằng các tiến trình tối ưu hóa định kỳ như Optimize/Compaction).

## Manifest

**Manifest** là một tệp bất biến (thường ở định dạng Avro) có chức năng liệt kê các tệp dữ liệu (data files) hoặc tệp xóa (delete files) thuộc về một Snapshot tại một thời điểm cụ thể. Dưới đây là các đặc điểm và khái niệm cốt lõi:

- **Tách biệt nội dung:** Một tệp manifest chỉ có thể lưu trữ danh sách các tệp dữ liệu hoặc tệp xóa, **không bao giờ chứa cả hai**. Việc này giúp hệ thống ưu tiên quét các tệp manifest chứa dữ liệu xóa trước tiên trong quá trình lập kế hoạch truy vấn (job planning).
- **Ràng buộc phân vùng (Partitioning):** Mỗi manifest chỉ lưu trữ các tệp thuộc về **một đặc tả phân vùng (partition spec) duy nhất**. Dữ liệu phân vùng của từng tệp cùng các số liệu thống kê (metrics) được lưu trực tiếp trong manifest.
- **Cấu trúc Manifest Entry:** Mỗi bản ghi trong manifest chứa các trường quản lý trạng thái của tệp như: trạng thái (`EXISTING`, `ADDED`, `DELETED`), `snapshot_id`, số thứ tự (`sequence_number`), và một cấu trúc `data_file`.
- **Cấu trúc Data File:** Nằm bên trong Manifest Entry, chứa thông tin chi tiết về tệp vật lý như: đường dẫn (`file_path`), định dạng (`file_format`), số lượng bản ghi (`record_count`), kích thước tệp, và các số liệu thống kê ở cấp độ cột (số giá trị null, giới hạn trên/dưới của dữ liệu).
- **Manifest List:** Các tệp manifest cấu thành nên một Snapshot được theo dõi và quản lý bởi một tệp cấp cao hơn gọi là **Manifest List**. Tệp này chứa các thống kê tóm tắt về từng manifest để tối ưu hóa quá trình đọc.

**Các Vấn đề và Giải pháp kỹ thuật liên quan đến Manifest**

Kiến trúc dựa trên manifest của Iceberg được thiết kế để giải quyết các vấn đề lớn liên quan đến hiệu suất, sự đồng thời và tiến hóa dữ liệu trong kho dữ liệu khổng lồ:

- **Tối ưu hóa lập kế hoạch truy vấn (Scan Planning):**
    - Thay vì phải dùng lệnh "list" toàn bộ thư mục vật lý (chi phí O(n)), Iceberg dùng manifest list và manifest để tìm dữ liệu.
    - Các tệp manifest chứa số liệu thống kê (bounds, counts) và thông tin phân vùng, cho phép quá trình truy vấn **bỏ qua các manifest hoặc tệp không chứa dữ liệu khớp với bộ lọc (predicates)** một cách nhanh chóng.
    - Các điều kiện lọc trên dữ liệu (scan predicates) được tự động chuyển đổi thành điều kiện lọc phân vùng (partition predicates) dựa trên metadata lưu trong manifest.
- **Tiến hóa Phân vùng (Partition Evolution):**
    - Khi bảng thay đổi cách phân vùng (ví dụ từ theo tháng sang theo ngày), dữ liệu cũ vẫn nằm yên trong các tệp manifest cũ gắn với partition spec cũ. Các tệp dữ liệu mới sẽ được ghi vào các tệp manifest mới áp dụng partition spec mới. Truy vấn sẽ dùng spec tương ứng của từng manifest để lọc, đảm bảo tính chính xác mà không cần viết lại toàn bộ dữ liệu lịch sử.
- **Cơ chế Kế thừa (Inheritance) và Tái sử dụng để giải quyết xung đột ghi:**
    - Trong môi trường đồng thời, nếu một commit thất bại do có giao dịch khác ghi chèn, người dùng không cần phải tạo lại toàn bộ tệp manifest.
    - Iceberg sử dụng **cơ chế kế thừa giá trị null**. Khi thêm một tệp mới, các trường `sequence_number`, `file_sequence_number` và `first_row_id` của tệp được đặt là `null` trong manifest.
    - Lúc đọc, các giá trị này tự động kế thừa từ metadata của manifest list. Điều này cho phép manifest được ghi một lần và tái sử dụng cho các lần thử lại giao dịch (commit retries) mà chỉ cần cập nhật lại tệp manifest list.
- **Hiệu suất và Khả năng mở rộng (Performance & Scalability):**
    - **Fast Append:** Việc chia dữ liệu thành nhiều manifest cho phép thêm dữ liệu mới cực nhanh bằng cách chỉ tạo ra một manifest mới chứa danh sách tệp vừa ghi, thay vì phải ghi đè hay nối vào manifest cũ.
    - **Phân chia tác vụ:** Đối với các bảng dữ liệu khổng lồ, metadata được phân chia qua nhiều manifest, giúp các engine xử lý (như Spark, Flink) có thể song song hóa (parallelize) quá trình lập kế hoạch truy vấn và giảm chi phí khi cần viết lại metadata

## Manifest list

**Manifest List** là một tệp đóng vai trò quản lý cấp cao hơn, nằm giữa Snapshot và các tệp Manifest trong cây metadata. Nó lưu trữ danh sách tất cả các tệp manifest cấu thành nên một Snapshot tại một thời điểm cụ thể.

Dưới đây là phân tích chi tiết về các khái niệm cốt lõi và các vấn đề kỹ thuật mà Manifest List giải quyết:

1. Các Khái niệm Cốt lõi của Manifest List

- **Tách biệt với Metadata bảng:** Mặc dù Snapshot được lưu trực tiếp bên trong tệp metadata của bảng, nhưng danh sách các manifest của snapshot đó lại được lưu trữ riêng biệt trong tệp Manifest List. Mỗi lần tạo snapshot mới (commit), một tệp manifest list mới sẽ được viết ra.
- **Cấu trúc dữ liệu hợp lệ:** Bản thân manifest list cũng là một tệp dữ liệu Iceberg hợp lệ, tuân thủ các định dạng, lược đồ và phép chiếu cột (column projection) của Iceberg. Nó lưu trữ danh sách các cấu trúc `manifest_file`.
- **Thống kê chi tiết cấp độ Manifest (Manifest Metadata):** Đối với mỗi tệp manifest, manifest list lưu trữ các số liệu tóm tắt rất quan trọng, bao gồm:
    - **Phân loại tệp và hàng:** Số lượng tệp hoặc hàng được thêm mới (`added_files_count`, `added_rows_count`), đang tồn tại (`existing_files_count`), hoặc đã xóa (`deleted_files_count`).
    - **Loại nội dung:** Định danh manifest này chứa tệp dữ liệu (`data`) hay tệp xóa (`deletes`).
    - **Tóm tắt phân vùng (Partition Summaries):** Chứa các giới hạn trên và dưới (`lower_bound`, `upper_bound`), cũng như thông tin về việc manifest có chứa giá trị null hay NaN cho từng trường phân vùng hay không.

2. Các Vấn đề và Giải pháp kỹ thuật do Manifest List đảm nhiệm

Manifest List được thiết kế nhằm giải quyết các bài toán về hiệu suất mở rộng, tính kế thừa và quản lý xung đột trong một Data Lake khổng lồ:

- **Tối ưu hóa Lập kế hoạch Quét (Scan Planning) cực nhanh:**
    - _Vấn đề:_ Đọc hàng ngàn tệp manifest để tìm xem tệp dữ liệu nào khớp với câu truy vấn là một quá trình rất chậm (chi phí O(n)).
    - _Giải pháp:_ Quá trình quét (scan) sẽ đọc manifest list trước tiên. Nhờ các thông kê phân vùng (như `lower_bound`, `upper_bound`) và số lượng tệp được lưu sẵn trong manifest list, engine truy vấn có thể **bỏ qua hoàn toàn các tệp manifest không chứa dữ liệu khớp với bộ lọc (predicates)**.
- **Kế thừa Số thứ tự (Sequence Number Inheritance) & Tái sử dụng Manifest:**
    - _Vấn đề:_ Trong môi trường đồng thời lạc quan (optimistic concurrency), nếu một commit thất bại, việc phải viết lại toàn bộ hàng loạt tệp manifest mới (chỉ để cập nhật sequence number mới) sẽ gây thắt cổ chai về hiệu suất.
    - _Giải pháp:_ Sequence number (thể hiện tuổi tương đối của dữ liệu) được hệ thống ghi vào tệp manifest list thay vì tệp manifest. Khi đọc, các tệp dữ liệu sẽ tự động "kế thừa" số thứ tự từ metadata của manifest list. Nhờ đó, tệp manifest có thể được **ghi một lần và tái sử dụng** cho các lần thử lại giao dịch (commit retries); hệ thống chỉ việc tạo ra một tệp manifest list mới.
- **Quản lý Nguồn gốc Hàng (Row Lineage / First Row ID Assignment - V3):**
    - _Vấn đề:_ Cần cấp phát một ID duy nhất (`_row_id`) cho từng hàng dữ liệu để phục vụ việc theo dõi nguồn gốc và thay đổi dữ liệu (như CDC) mà không làm chậm quá trình ghi.
    - _Giải pháp:_ Manifest List chịu trách nhiệm cấp phát `first_row_id` (ID của hàng đầu tiên) cho các manifest chứa tệp dữ liệu mới. Nó tự động tính toán ID này dựa trên tổng số lượng hàng từ các manifest được cấp phát trước đó. Điều này đảm bảo mỗi bản ghi có một không gian ID liên tục và duy nhất trên toàn bộ bảng.
- **Điều phối Tiến hóa Phân vùng (Partition Evolution):**
    - _Vấn đề:_ Khi thay đổi cách phân vùng, dữ liệu cũ và mới có cấu trúc phân vùng khác nhau.
    - _Giải pháp:_ Manifest list ghi nhận `partition_spec_id` cho từng manifest riêng biệt. Điều này cho phép một snapshot có thể chứa nhiều manifest áp dụng các lược đồ phân vùng khác nhau (ví dụ: dữ liệu năm ngoái phân vùng theo tháng, dữ liệu năm nay phân vùng theo ngày) mà engine truy vấn vẫn đọc và lọc chính xác mà không gặp lỗi

## Scan Planning

**Lập kế hoạch Quét (Scan Planning)** là quá trình xác định chính xác tập hợp các tệp dữ liệu và tệp xóa cần thiết để thực hiện một câu truy vấn. Các khái niệm cấu thành quá trình này bao gồm:

- **Quét qua Manifest (Manifest Scanning):** Quá trình lập kế hoạch được thực hiện bằng cách đọc các tệp manifest thuộc về snapshot hiện tại. Bất kỳ tệp dữ liệu hoặc tệp xóa nào bị đánh dấu trạng thái "DELETED" đều sẽ bị loại bỏ khỏi quá trình quét.
- **Loại bỏ tệp/manifest không phù hợp (Skipping):** Hệ thống sử dụng số lượng đếm tệp (file counts) và các bảng tóm tắt phân vùng (partition summaries) lưu ở cấp độ manifest list để bỏ qua hoàn toàn các manifest không chứa tệp khớp với câu truy vấn.
- **Phép chiếu bao hàm (Inclusive Projection):** Đây là kỹ thuật chuyển đổi các bộ lọc truy vấn trên dữ liệu (scan predicates) thành các bộ lọc trên phân vùng (partition predicates) dựa trên cấu hình phân vùng (partition spec) của manifest. Nó được gọi là "bao hàm" vì một tệp vẫn có thể được thêm vào tập kết quả quét nếu phân vùng của nó khớp, ngay cả khi nó chứa các hàng dữ liệu không thỏa mãn bộ lọc dữ liệu.
- **Phép chiếu nghiêm ngặt (Strict Projection):** Trái ngược với bao hàm, phép chiếu nghiêm ngặt tạo ra một bộ lọc chỉ khớp với một tệp nếu **tất cả** các hàng bên trong tệp đó bắt buộc phải khớp với điều kiện quét, qua đó giúp tạo ra các bộ lọc cặn (residual predicates) áp dụng sau cùng cho từng tệp.

**Phân tích các Vấn đề và Giải pháp kỹ thuật trong Scan Planning**

Quá trình Scan Planning giải quyết các vấn đề hiệu suất và sự phức tạp vật lý trong các Data Lake lớn:

- **Giải quyết tốc độ quét (Speed) qua Metadata O(1):** Iceberg giải quyết tình trạng "thắt cổ chai" khi phải liệt kê hàng triệu tệp. Lập kế hoạch quét sử dụng các truy vấn O(1) để quét metadata tree, không bị chậm theo thời gian độ lớn O(n) như số lượng tệp hay phân vùng vật lý trên hệ thống.
- **Tách biệt logic truy vấn khỏi cấu trúc vật lý (Storage separation):** Scan planning sử dụng **Inclusive projection** để giải phóng người dùng khỏi việc biết cấu trúc bảng. Ví dụ, nếu bạn có bảng theo dõi sự kiện truy vấn bằng khoảng thời gian `ts > X` nhưng bảng lại phân vùng theo ngày `ts_day=day(ts)`, phép chiếu sẽ tự động biến đổi bộ lọc thành `ts_day >= day(X)` để tìm chính xác các tệp. Người dùng không bao giờ cần cung cấp trực tiếp các điều kiện phân vùng vào câu lệnh.
- **Tối ưu lọc bằng Số liệu Thống kê Cột (Column Metrics Filter):** Các bộ lọc quét (scan predicates) không chỉ áp dụng cho dữ liệu phân vùng mà còn sử dụng các số liệu thống kê (giới hạn trên/dưới, số lượng null) được lưu ở cấp độ tệp (cả tệp dữ liệu và tệp xóa) trong manifest. Nếu số liệu thống kê chứng minh một tệp xóa không có hàng nào khớp với bộ lọc, hệ thống sẽ bỏ qua tệp xóa đó để tiết kiệm thời gian đọc.
- **Bài toán áp dụng Phạm vi Tệp xóa (Scope Rules cho Deletes):** Để đảm bảo chỉ đọc đúng dữ liệu, Scan Planning quy định nghiêm ngặt cách áp dụng tệp xóa vào tệp dữ liệu:
    - **Deletion Vectors và Position Delete:** Chỉ áp dụng cho một tệp dữ liệu nếu chúng có đường dẫn tệp (file_path) trùng khớp, dữ liệu phân vùng bằng nhau, và số thứ tự (sequence number) của tệp dữ liệu **nhỏ hơn hoặc bằng** số thứ tự của tệp xóa. Việc cho phép "bằng" giúp giải quyết vấn đề xóa ngay các hàng dữ liệu vừa được thêm vào trong cùng một giao dịch commit.
    - **Equality Delete:** Chỉ áp dụng khi số thứ tự của tệp dữ liệu **nhỏ hơn nghiêm ngặt** tệp xóa, và hai tệp phải chung phân vùng. Nếu một equality delete file không có cấu hình phân vùng (unpartitioned spec), nó giải quyết vấn đề xóa toàn cục (áp dụng cho mọi phân vùng).
- **Xử lý sai sót dữ liệu và thay đổi bất định:**
    - Nếu gặp một phép biến đổi phân vùng không xác định (unknown transform), bộ lọc sẽ luôn trả về `true` (không lọc), đảm bảo an toàn tuyệt đối không bỏ sót dữ liệu thay vì gây lỗi dừng đột ngột.
    - Hệ thống kiểm tra sự toàn vẹn của tệp: trong một snapshot, mỗi đường dẫn tệp (mang trạng thái ADDED hoặc EXISTING) chỉ được phép xuất hiện tối đa một lần trên toàn bộ các tệp manifest; nếu xuất hiện nhiều hơn, kết quả quét sẽ bất định (undefined) và hệ thống có thể bắn ra lỗi.

## Table Metadata

**Table Metadata (Siêu dữ liệu Bảng)** là trung tâm điều khiển (root) của toàn bộ hệ thống. Nó được lưu trữ dưới định dạng JSON và chịu trách nhiệm theo dõi toàn bộ trạng thái, cấu trúc, và lịch sử của bảng.

Dưới đây là phân tích chi tiết về các khái niệm cốt lõi cấu thành Table Metadata và những vấn đề hệ thống mà nó giải quyết.

1. Các Khái niệm Cấu trúc trong Table Metadata

Table Metadata bao gồm nhiều trường dữ liệu để duy trì trạng thái của bảng theo thời gian:

- **Quản lý Lược đồ (Schema Management):** Thay vì chỉ lưu một lược đồ hiện tại, metadata lưu trữ một danh sách toàn bộ các lược đồ từng tồn tại trong trường `schemas` và trỏ đến lược đồ đang kích hoạt bằng `current-schema-id`. Nó cũng duy trì `last-column-id` (ID cột cao nhất đã cấp phát) để đảm bảo không bao giờ có sự trùng lặp ID khi thêm cột mới.
- **Đặc tả Phân vùng (Partition Specs):** Tương tự như lược đồ, mọi cách phân vùng trong lịch sử được lưu trong `partition-specs`, và cách phân vùng đang được sử dụng cho dữ liệu ghi mới được trỏ bởi `default-spec-id`.
- **Theo dõi Snapshot (Snapshot Tracking):** Chứa danh sách các snapshot hợp lệ (`snapshots`) và con trỏ đến snapshot hiện tại `current-snapshot-id`.
- **Lịch sử và Nhật ký (Logs):**
    - `snapshot-log`: Lưu lại các mốc thời gian và ID của snapshot tương ứng khi bảng thay đổi, phục vụ cho việc du hành thời gian (time travel).
    - `metadata-log`: Ghi lại các tệp metadata cũ để theo dõi sự thay đổi của chính siêu dữ liệu.
- **Tham chiếu (Refs - Branches & Tags):** Trường `refs` cho phép định nghĩa các nhánh (branch) hoặc thẻ (tag) để theo dõi các chuỗi snapshot cụ thể (ví dụ: tạo nhánh thử nghiệm riêng biệt mà không ảnh hưởng nhánh chính).
- **ID Hàng và Thứ tự (Row Lineage & Sequence):** Metadata lưu trữ `last-sequence-number` (số thứ tự cao nhất đã cấp phát) để theo dõi độ tuổi tương đối của dữ liệu, và `next-row-id` để cấp phát ID duy nhất cho các bản ghi mới.

2. Quá trình Table Metadata giải quyết các vấn đề kỹ thuật lớn

Table Metadata được thiết kế để giải quyết những thách thức khổng lồ về sự đồng thời, độ ổn định và tính mở rộng của các kho dữ liệu (data lakes).

**Vấn đề 1: Đảm bảo Tính nhất quán và Xử lý Xung đột Đồng thời (Concurrency & Conflict Resolution)**

- _Vấn đề:_ Khi có nhiều quy trình (engine) cùng ghi dữ liệu vào bảng cùng một lúc, dữ liệu rất dễ bị hỏng hoặc mất tính nhất quán.
- _Giải pháp:_ Mọi thay đổi đối với trạng thái bảng đều tạo ra một tệp metadata JSON hoàn toàn mới. Tệp mới này thay thế tệp cũ thông qua một **phép hoán đổi nguyên tử (atomic swap)**, tạo ra một lịch sử tuyến tính.
    - Với hệ thống tệp cục bộ hoặc HDFS, Iceberg sử dụng lệnh đổi tên nguyên tử (atomic rename). Tuy nhiên cơ chế này bị phản đối trong V4 vì không an toàn trên các kho lưu trữ Object (như S3).
    - Với môi trường thực tế (có Metastore như Hive, AWS Glue), hệ thống dùng thao tác `check-and-put`: nó kiểm tra xem phiên bản metadata cũ có còn là mới nhất không, nếu đúng thì trỏ sang tệp metadata mới. Nếu thất bại (có người khác ghi chen vào), giao dịch sẽ tự động tải lại trạng thái và thử lại (retry).

**Vấn đề 2: Truy vấn Xuyên thời gian (Point-in-Time Reads / Time Travel)**

- _Vấn đề:_ Cần đọc chính xác trạng thái của dữ liệu vào đúng một thời điểm trong quá khứ, nhưng lịch sử gia phả snapshot (parent-child lineage) đôi khi không khớp với thời điểm thực tế (ví dụ do thao tác gán lại snapshot thủ công).
- _Giải pháp:_ Iceberg yêu cầu các hệ thống truy vấn phải sử dụng trường `snapshot-log` trong Table Metadata để tìm snapshot ngay trước mốc thời gian được yêu cầu. Điều này đảm bảo truy vấn lấy đúng dữ liệu như nó tồn tại vào thời điểm đó thay vì bị nhầm lẫn bởi lịch sử gia phả.

**Vấn đề 3: Sự tiến hóa an toàn (Safe Evolution) mà không phải viết lại dữ liệu**

- _Vấn đề:_ Khi xóa một cột hoặc một lược đồ phân vùng rồi vô tình tạo lại với cùng tên, hệ thống cũ thường cấp lại ID cũ khiến đọc sai cấu trúc dữ liệu lịch sử.
- _Giải pháp:_ Iceberg duy trì `last-column-id` và `last-partition-id`. Khi một trường mới được thêm vào, nó bắt buộc phải dùng ID tiếp theo lớn hơn `last-column-id`, bất kể tên là gì. Truy vấn lập bản đồ dữ liệu dựa trên ID thay vì tên, do đó chặn đứng hoàn toàn việc dữ liệu cũ bị đọc sai định dạng.

**Vấn đề 4: Tính di động của Bảng (Table Portability)**

- _Vấn đề:_ Ở các định dạng cũ (V1-V3), các đường dẫn trong metadata là đường dẫn tuyệt đối (absolute paths) chứa cả tên máy chủ hoặc bucket (ví dụ `s3://bucket/...`). Điều này khiến việc sao lưu (backup) hoặc di chuyển bảng sang môi trường khác vô cùng khó khăn vì phải quét và viết lại toàn bộ metadata.
- _Giải pháp:_ Từ Phiên bản 4 (v4), Iceberg bổ sung khả năng sử dụng **đường dẫn tương đối (relative locations)**. Đường dẫn tệp được lưu không có gốc URI và tự động kết hợp (resolve) với đường dẫn gốc (`location`) của bảng khi đọc. Giờ đây, chỉ cần thay đổi gốc `location` trong Table Metadata, toàn bộ dữ liệu có thể di chuyển an toàn.