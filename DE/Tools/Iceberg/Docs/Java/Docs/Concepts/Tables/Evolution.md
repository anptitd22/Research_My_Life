
- [[#Schema evolution|Schema evolution]]
- [[#Correctness|Correctness]]
- [[#Partition evolution|Partition evolution]]
- [[#Sort order evolution|Sort order evolution]]

Iceberg hỗ trợ tính năng table evolution tại chỗ. Bạn có thể schema evolution giống như SQL -- ngay cả trong các cấu trúc lồng nhau -- hoặc thay đổi bố cục phân vùng khi khối lượng dữ liệu thay đổi. Iceberg không yêu cầu những thao tác rườm rà tốn kém, chẳng hạn như viết lại dữ liệu bảng hoặc di chuyển sang bảng mới. 

Ví dụ, phân vùng bảng Hive không thể thay đổi, do đó việc chuyển từ bố cục phân vùng theo ngày sang bố cục phân vùng theo giờ đòi hỏi phải có một bảng mới. Và vì các truy vấn phụ thuộc vào phân vùng, các truy vấn phải được viết lại cho bảng mới. Trong một số trường hợp, ngay cả những thay đổi đơn giản như đổi tên cột cũng không được hỗ trợ hoặc có thể gây ra sự cố về tính chính xác của dữ liệu.

### Schema evolution

Iceberg hỗ trợ những thay schema evolution sau:

- Add -- add a new column to the table or to a nested struct
    
- Drop -- remove an existing column from the table or a nested struct
    
- Rename -- rename an existing column or field in a nested struct
    
- Update -- widen the type of a column, struct field, map key, map value, or list element
    
- Reorder -- change the order of columns or fields in a nested struct
    
Iceberg schema evolution là những thay đổi metadata, do đó không cần phải viết lại tệp dữ liệu để thực hiện cập nhật.

Lưu ý rằng các khóa bản đồ không hỗ trợ việc thêm hoặc xóa các trường cấu trúc có thể làm thay đổi tính bình đẳng.

### Correctness

Iceberg đảm bảo rằng các thay đổi trong quá trình phát triển lược đồ là độc lập và không có tác dụng phụ, không cần viết lại tệp:

1. Các cột được thêm vào không bao giờ đọc các giá trị hiện có từ cột khác.
    
2. Việc xóa một cột hoặc trường sẽ không làm thay đổi giá trị ở bất kỳ cột nào khác.
    
3. Việc cập nhật một cột hoặc trường không làm thay đổi giá trị ở bất kỳ cột nào khác.
    
4. Việc thay đổi thứ tự các cột hoặc trường trong một cấu trúc không làm thay đổi các giá trị liên quan đến tên cột hoặc trường.
    
Iceberg sử dụng ID duy nhất để theo dõi từng cột trong bảng. Khi bạn thêm một cột, cột đó sẽ được gán một ID mới để dữ liệu hiện có không bao giờ bị sử dụng nhầm.

Các định dạng theo dõi cột theo tên có thể vô tình khôi phục lại cột đã xóa nếu tên được sử dụng lại, điều này vi phạm điều số 1.

Các định dạng theo dõi các cột theo vị trí không thể xóa các cột mà không thay đổi tên được sử dụng cho từng cột, điều này vi phạm điều số 2.

### Partition evolution

Phân vùng bảng Iceberg có thể được cập nhật trong một bảng hiện có vì các truy vấn không tham chiếu trực tiếp đến các giá trị phân vùng.

Khi bạn phát triển một đặc tả phân vùng, dữ liệu cũ được ghi bằng đặc tả trước đó sẽ không thay đổi. Dữ liệu mới được ghi bằng đặc tả mới trong một bố cục mới. Metadata cho mỗi phiên bản phân vùng được lưu trữ riêng biệt. Do đó, khi bạn bắt đầu viết truy vấn, bạn sẽ được lập kế hoạch phân tách. Đây là nơi mỗi bố cục phân vùng lập kế hoạch các tệp riêng biệt bằng bộ lọc mà nó suy ra cho bố cục phân vùng cụ thể đó. Dưới đây là hình ảnh minh họa một ví dụ giả định:


![alt text](images/Pasted_image_20251121130605.png)

  
Dữ liệu năm 2008 được phân vùng theo tháng. Bắt đầu từ năm 2009, bảng được cập nhật để dữ liệu được phân vùng theo ngày. Cả hai bố cục phân vùng có thể cùng tồn tại trong cùng một bảng.

Iceberg sử dụng phân vùng ẩn(hidden partition), do đó bạn không cần phải viết truy vấn cho một bố cục phân vùng cụ thể để tăng tốc. Thay vào đó, bạn có thể viết truy vấn để chọn dữ liệu cần thiết, và Iceberg sẽ tự động loại bỏ các tệp không chứa dữ liệu khớp.

Partition evolution là một hoạt động lên metadata và không ghi đè lên các tệp một cách chủ động.

API bảng Java của Iceberg cung cấp API updateSpec để cập nhật thông số phân vùng. Ví dụ: đoạn mã sau có thể được sử dụng để cập nhật thông số phân vùng nhằm thêm một trường phân vùng mới, đặt các giá trị cột id vào 8 nhóm và xóa một catalog trường phân vùng hiện có:

```java
Table sampleTable = ...;
sampleTable.updateSpec()
    .addField(bucket("id", 8))
    .removeField("category")
    .commit();
```

Spark hỗ trợ cập nhật thông số phân vùng thông qua câu lệnh `ALTER TABLE SQL`, xem thêm chi tiết trong [Spark SQL](https://iceberg.apache.org/docs/latest/spark-ddl/#alter-table-add-partition-field).

### Sort order evolution

Tương tự như đặc tả phân vùng, thứ tự sắp xếp Iceberg cũng có thể được cập nhật trong một bảng hiện có. Khi bạn phát triển một thứ tự sắp xếp, dữ liệu cũ được ghi theo thứ tự trước đó sẽ không thay đổi. Các công cụ luôn có thể chọn ghi dữ liệu theo thứ tự sắp xếp mới nhất hoặc không được sắp xếp khi việc sắp xếp quá tốn kém.

API bảng Java của Iceberg cung cấp API `replaceSortOrder` để cập nhật thứ tự sắp xếp. Ví dụ: đoạn mã sau có thể được sử dụng để tạo một thứ tự sắp xếp mới với cột id được sắp xếp theo thứ tự tăng dần với giá trị null đứng cuối, và cột category được sắp xếp theo thứ tự giảm dần với giá trị null đứng đầu:

```java
Table sampleTable = ...;
sampleTable.replaceSortOrder()
   .asc("id", NullOrder.NULLS_LAST)
   .dec("category", NullOrder.NULL_FIRST)
   .commit();
```

Spark hỗ trợ cập nhật thứ tự sắp xếp thông qua câu lệnh ALTER TABLE SQL, xem thêm chi tiết trong [Spark SQL](https://iceberg.apache.org/docs/latest/spark-ddl/#alter-table-add-partition-field).