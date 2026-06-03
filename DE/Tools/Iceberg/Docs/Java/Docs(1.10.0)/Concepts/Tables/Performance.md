- [[#Scan planning|Scan planning]]
	- [[#Scan planning#Metadata filtering|Metadata filtering]]
	- [[#Scan planning#Data filtering|Data filtering]]

Iceberg được thiết kế cho các bảng dữ liệu khổng lồ và được sử dụng trong môi trường sản xuất, nơi một bảng duy nhất có thể chứa hàng chục petabyte dữ liệu.

Ngay cả các bảng có dung lượng nhiều petabyte cũng có thể được đọc từ một nút duy nhất, mà không cần đến công cụ SQL phân tán để sàng lọc metadata của bảng.

## Scan planning

Lập kế hoạch quét là quá trình tìm kiếm các tập tin trong một bảng cần thiết cho một truy vấn.

Việc lập kế hoạch trong bảng Iceberg phù hợp với một nút duy nhất vì siêu dữ liệu của Iceberg có thể được sử dụng để loại bỏ các tệp siêu dữ liệu không cần thiết, ngoài việc lọc các tệp dữ liệu không chứa dữ liệu phù hợp.

Lập kế hoạch quét nhanh từ một nút duy nhất cho phép:

- Giảm độ trễ của các truy vấn SQL bằng cách loại bỏ bước lập kế hoạch quét phân tán.
- Bất kỳ quy trình độc lập nào của máy khách đều có thể truy cập và đọc dữ liệu trực tiếp từ các bảng Iceberg.

### Metadata filtering

Iceberg sử dụng hai cấp độ metadata để theo dõi các tệp trong snapshot.

- **Manifest files** lưu trữ danh sách các tệp dữ liệu, cùng với dữ liệu phân vùng và số liệu thống kê cấp cột của mỗi tệp dữ liệu.
- **Manifest list** lưu trữ danh sách các manifest của snapshot, cùng với phạm vi giá trị cho từng trường phân vùng.

Để lập kế hoạch quét nhanh, Iceberg trước tiên lọc các manifest bằng cách sử dụng phạm vi giá trị phân vùng trong manifest list. Sau đó, nó đọc từng manifest để lấy các tệp dữ liệu. Với phương pháp này, manifest list hoạt động như một chỉ mục trên các tệp manifest, giúp việc lập kế hoạch có thể thực hiện mà không cần đọc tất cả các manifest.

Ngoài phạm vi giá trị phân vùng, manifest list cũng lưu trữ số lượng tệp được thêm hoặc xóa trong manifest để tăng tốc các thao tác như hết hạn snapshot.

### Data filtering

Manifest files bao gồm một bộ dữ liệu phân vùng và số liệu thống kê column-level cho mỗi tệp dữ liệu.

Trong quá trình lập kế hoạch, các điều kiện truy vấn được tự động chuyển đổi thành các điều kiện trên dữ liệu phân vùng và được áp dụng trước tiên để lọc các tệp dữ liệu. Tiếp theo, số lượng giá trị ở column-level, số lượng giá trị null, giới hạn dưới và giới hạn trên được sử dụng để loại bỏ các tệp không khớp với điều kiện truy vấn.

Bằng cách sử dụng giới hạn trên và giới hạn dưới để lọc các tập dữ liệu trong quá trình lập kế hoạch, Iceberg sử dụng dữ liệu được phân cụm để loại bỏ các điểm chia tách mà không cần chạy các tác vụ. Trong một số trường hợp, điều này mang lại hiệu suất được cải thiện gấp 10 lần ([[Introducing Iceberg_ Tables designed for object stores Presentation.pdf]]).

