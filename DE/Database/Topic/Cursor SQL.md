Trong SQL, con trỏ (cursor) là một đối tượng cơ sở dữ liệu được sử dụng để xử lý dữ liệu từng hàng một, hữu ích khi cần xử lý từng hàng thay vì xử lý hàng loạt. Nó lưu trữ dữ liệu tạm thời cho các thao tác như SELECT, UPDATE hoặc DELETE.

- Hữu ích để áp dụng logic tùy chỉnh, các phép toán có điều kiện hoặc cập nhật từng bước.
- Thường được sử dụng trong PL/SQL hoặc T-SQL cho các tác vụ như vòng lặp, logic điều kiện hoặc các thủ tục phức tạp.

![[cusor_sql_1.png]]

## Use of Cursor

Nên sử dụng con trỏ chuột một cách cẩn thận vì chúng hữu ích trong các trường hợp như:

- Thực hiện logic điều kiện từng hàng một
- Lặp qua dữ liệu để tính toán hoặc chuyển đổi các trường.
- Lặp lại các tập kết quả để cập nhật hoặc biến đổi có điều kiện.
- Xử lý cấu trúc dữ liệu phân cấp hoặc đệ quy
- Thực hiện các tác vụ dọn dẹp không thể thực hiện bằng một truy vấn SQL duy nhất.

## Types of Cursors in SQL

SQL cung cấp hai loại con trỏ chính, mỗi loại phù hợp với các tình huống khác nhau và tùy thuộc vào mức độ kiểm soát mà chúng ta muốn.

### 1. Implicit Cursors

Trong PL/SQL, khi thực hiện các thao tác INSERT, UPDATE hoặc DELETE, một con trỏ ngầm định sẽ tự động được tạo ra. Con trỏ này lưu giữ dữ liệu cần chèn hoặc xác định các hàng cần cập nhật hoặc xóa. Chúng ta có thể gọi con trỏ này là con trỏ SQL trong mã của mình. Nó được quản lý hoàn toàn bởi công cụ SQL mà không cần khai báo rõ ràng.

***Useful Attributes**:

- ****%FOUND****:Đúng nếu thao tác SQL ảnh hưởng đến ít nhất một hàng.
- ****%NOTFOUND****: Đúng nếu không có hàng nào bị ảnh hưởng.
- ****%ROWCOUNT****: Trả về số lượng hàng bị ảnh hưởng
- ****%ISOPEN****:  Kiểm tra xem con trỏ có đang mở hay không.
### Example: Using Implicit Cursor for Bulk Updates

Chương trình này cập nhật một bảng bằng cách tăng lương của mỗi nhân viên lên 1500. Sau khi cập nhật, thuộc tính SQL%ROWCOUNT được sử dụng để tìm ra số lượng hàng bị ảnh hưởng bởi thao tác này.

****Query:****

```
DECLARE    
   total_rows number;   
BEGIN   
   UPDATE Emp   
   SET Salary = Salary + 1500;   
   
   total_rows := SQL%ROWCOUNT;  
   
   dbms_output.put_line(total_rows || ' rows updated.');   
END;
```

****Output****

```
5 Emp selected    
PL/SQL procedure successfully completed.
```

****Explanation:****

- BULK_ROWCOUNT và %BULK_EXCEPTIONS được sử dụng với câu lệnh FORALL.
- Câu lệnh FORALL được sử dụng để thực hiện nhiều thao tác DML một cách hiệu quả.
- BULK_ROWCOUNT trả về số lượng hàng bị ảnh hưởng bởi mỗi thao tác DML.
- BULK_EXCEPTIONS trả về thông tin về các ngoại lệ xảy ra trong quá trình thực hiện các thao tác hàng loạt.

### 2. Explicit Cursors

Đây là các con trỏ do người dùng định nghĩa, được tạo ra một cách rõ ràng bởi người dùng cho các thao tác tùy chỉnh. Chúng cung cấp quyền kiểm soát hoàn toàn đối với mọi phần trong vòng đời của chúng: khai báo, mở, truy xuất, đóng và giải phóng. Chúng được sử dụng để truy xuất dữ liệu từng hàng một với quyền kiểm soát hoàn toàn vòng đời của con trỏ.

#### ****Explicit cursors are useful when:****

- Chúng ta cần lặp qua các kết quả một cách thủ công.
- Mỗi hàng cần được xử lý bằng logic riêng.
- Chúng ta cần truy cập vào các thuộc tính của hàng trong quá trình xử lý.

### Example: Using an Explicit Cursor

Dưới đây là một ví dụ hoàn chỉnh về việc khai báo, mở, truy xuất, đóng và giải phóng con trỏ. Quy trình này minh họa cách sử dụng con trỏ tường minh cho các thao tác từng hàng, bao gồm quản lý tài nguyên và truy xuất kết quả.

****Query:****

```
DECLARE emp_cursor CURSOR FOR SELECT Name, Salary FROM Employees;  
  
BEGIN  
    OPEN emp_cursor;  
  FETCH NEXT FROM emp_cursor INTO @Name, @Salary;  
   WHILE @@FETCH_STATUS = 0  
   BEGIN  
      PRINT 'Name: ' + @Name + ', Salary: ' + CAST(@Salary AS VARCHAR);  
      FETCH NEXT FROM emp_cursor INTO @Name, @Salary;  
   END;  
  
   CLOSE emp_cursor;  
  
   DEALLOCATE emp_cursor;  
END;
```

****Output:****

![alt][images/Database/cusor_sql_2.png]

## Cursor Syntax Breakdown in SQL


## Advantages of Using Cursors

- ****Row-by-row processing:****
- ****Iterative handling:****
- ****Handles complex relationships:****
- ****Conditional operations:****
- ****Flexible for complex cases:****

## Limitations of Cursors

- ****Slow performance:**** 
- ****High resource usage:****
- ****Complex syntax:****
- ****Not suitable for large data:****

Nguồn: https://www.geeksforgeeks.org/sql/what-is-cursor-in-sql/
