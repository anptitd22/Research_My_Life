# Normalization

Chuẩn hóa là một quy trình quan trọng trong thiết kế cơ sở dữ liệu, giúp cải thiện hiệu quả, tính nhất quán (consistency) và độ chính xác của cơ sở dữ liệu. Nó giúp việc quản lý và bảo trì dữ liệu dễ dàng hơn, đồng thời đảm bảo cơ sở dữ liệu có khả năng thích ứng với những thay đổi trong nhu cầu nghiệp vụ.

- Chuẩn hóa cơ sở dữ liệu là quá trình tổ chức các thuộc tính của cơ sở dữ liệu để giảm thiểu hoặc loại bỏ sự dư thừa dữ liệu (dữ liệu giống nhau nhưng nằm ở các vị trí khác nhau).
- Việc dữ liệu dư thừa làm tăng kích thước cơ sở dữ liệu một cách không cần thiết do cùng một dữ liệu được lặp lại ở nhiều nơi. Các vấn đề về tính không nhất quán (consistency) cũng phát sinh trong các thao tác chèn, xóa và cập nhật.
- Trong mô hình quan hệ, tồn tại các phương pháp chuẩn để định lượng hiệu quả của cơ sở dữ liệu. Các phương pháp này được gọi là dạng chuẩn, và có các thuật toán để chuyển đổi một cơ sở dữ liệu nhất định thành dạng chuẩn.

![alt][images/Database/Database_Normalization/Before_Normalization.png]

![alt][images/Database/Database_Normalization/After_Normalization.png]

- Trước khi chuẩn hóa: Bảng dễ bị trùng lặp và có các bất thường (chèn, cập nhật và xóa). 
- Sau khi chuẩn hóa: Dữ liệu được chia thành các bảng logic để đảm bảo tính nhất quán (consistency), tránh trùng lặp và loại bỏ các bất thường, giúp cơ sở dữ liệu hoạt động hiệu quả và đáng tin cậy.

Các vấn đề trong mối quan hệ Nhân viên_Phòng ban

- Lỗi khi thêm dữ liệu: Nếu một phòng ban mới được tạo nhưng chưa có nhân viên nào được chỉ định vào phòng ban đó, chúng ta không thể lưu trữ vị trí của phòng ban đó vì cần có bản ghi nhân viên để thêm vào. 
- Lỗi cập nhật: Nếu vị trí của phòng Nhân sự thay đổi, chúng ta phải cập nhật thông tin ở nhiều dòng (cho cả Nick Wise và Lily Case). Nếu bỏ sót một dòng, dữ liệu sẽ trở nên không nhất quán. 
- Lỗi xóa dữ liệu: Nếu tất cả nhân viên trong bộ phận CNTT nghỉ việc, chúng ta sẽ mất thông tin của bộ phận, bao gồm cả địa điểm. 
- Dữ liệu dư thừa: Thông tin về vị trí phòng ban được lặp lại cho mọi nhân viên trong cùng một phòng ban.

## Normal Forms in DBMS

| Normal Forms                          | Description of Normal Forms                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ****First Normal Form (1NF)****       | Một quan hệ được gọi là ở dạng chuẩn thứ nhất ([first normal form](https://www.geeksforgeeks.org/dbms/first-normal-form-1nf/)) nếu mọi thuộc tính trong quan hệ đó đều là thuộc tính đơn giá trị.                                                                                                                                                                                                                                                                                                         |
| ****Second Normal Form (2NF)****      | Một quan hệ ở dạng chuẩn thứ nhất (FNF) mà mọi thuộc tính không phải khóa chính đều phụ thuộc hoàn toàn vào khóa chính, thì quan hệ đó ở dạng chuẩn thứ hai (2NF) ([Second Normal Form (2NF).](https://www.geeksforgeeks.org/dbms/second-normal-form-2nf/)).                                                                                                                                                                                                                                              |
| ****Third Normal Form (3NF)****       | Một quan hệ ở dạng chuẩn thứ ba (3NF) ([third normal form](https://www.geeksforgeeks.org/dbms/third-normal-form-3nf/)) nếu không có sự phụ thuộc bắc cầu đối với các thuộc tính không phải là thuộc tính chính, cũng như ở dạng chuẩn thứ hai (2NF). Một quan hệ ở dạng 3NF nếu ít nhất một trong các điều kiện sau đây đúng trong mọi sự phụ thuộc hàm không tầm thường X –> Y.<br><br>- X là một siêu khóa.<br>- Y là một thuộc tính chính (mỗi phần tử của Y là một phần của một số khóa ứng cử viên). |
| ****Boyce-Codd Normal Form (BCNF)**** | Đối với dạng BCNF, mối quan hệ phải thỏa mãn các điều kiện sau:<br><br>- Mối quan hệ này phải ở dạng chuẩn tắc thứ 3 ([Normal Form](https://www.geeksforgeeks.org/dbms/normal-forms-in-dbms/)).<br>- X phải là khóa siêu cấp cho mọi phụ thuộc chức năng (FD) X−>Y trong một quan hệ nhất định.                                                                                                                                                                                                           |
| ****Fourth Normal Form (4NF)****      | A relation R is in [4NF](https://www.geeksforgeeks.org/dbms/introduction-of-4th-and-5th-normal-form-in-dbms/) if and only if the following conditions are satisfied: <br><br>- It should be in the [Boyce-Codd Normal Form (BCNF)](https://www.geeksforgeeks.org/dbms/boyce-codd-normal-form-bcnf/).<br>- The table should not have any Multi-valued Dependency.                                                                                                                                          |
| ****Fifth Normal Form (5NF)****       | A relation R is in [5NF](https://www.geeksforgeeks.org/dbms/what-is-fifth-normal-form-5nf-in-dbms/) if and only if it satisfies the following conditions:<br><br>- R should be already in 4NF. <br>- It cannot be further non loss decomposed (join dependency)                                                                                                                                                                                                                                           |

Trích: https://www.geeksforgeeks.org/dbms/introduction-of-database-normalization/

# Denormalization

Trích: https://www.geeksforgeeks.org/dbms/denormalization-in-databases/
