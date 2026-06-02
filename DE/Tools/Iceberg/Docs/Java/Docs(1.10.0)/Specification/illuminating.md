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