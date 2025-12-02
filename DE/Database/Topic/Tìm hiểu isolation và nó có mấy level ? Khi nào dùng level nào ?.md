# Isolation level
>[!note] Dirty Read
>T1 write bảng vào bảng A chưa commit, T2 read bảng A sẽ thấy được cả dữ liệu của T1 nhưng nếu T1 rollback thì dữ liệu trên bảng sẽ mất nhưng T2 vẫn read dữ liệu đó

>[!note] Non-repeatable Read
>T1 read bảng A lần 1, T2 write vào bảng A, T1 read bảng A lại lần nữa thấy dữ liệu bị thay đổi

>[!note] Phantom
>T1 truy vấn với điều kiện vào bảng A lần 1, T2 write dữ liệu nằm trong điều kiện đó vào bảng A, T1 truy vấn lại lần 2 thì thấy dữ liệu thay đổi
- Uncommitted Read
- Committed Read
- Repeatable Read
- Serializable
![alt](images/Database/isolation_level.png)
# MYSQL

Tính cô lập của mô hình ACID chủ yếu liên quan đến các giao dịch InnoDB, đặc biệt là [isolation level](https://dev.mysql.com/doc/refman/9.3/en/glossary.html#glos_isolation_level "isolation level") áp dụng cho từng giao dịch. Các tính năng liên quan của MySQL bao gồm:
- The [`autocommit`](https://dev.mysql.com/doc/refman/9.3/en/server-system-variables.html#sysvar_autocommit) setting.
- Transaction isolation levels and the [`SET TRANSACTION`](https://dev.mysql.com/doc/refman/9.3/en/set-transaction.html "15.3.7 SET TRANSACTION Statement") statement. See [Section 17.7.2.1, “Transaction Isolation Levels”](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html "17.7.2.1 Transaction Isolation Levels").
- The low-level details of `InnoDB` [locking](https://dev.mysql.com/doc/refman/9.3/en/glossary.html#glos_locking "locking"). Details can be viewed in the `INFORMATION_SCHEMA` tables (see [Section 17.15.2, “InnoDB INFORMATION_SCHEMA Transaction and Locking Information”](https://dev.mysql.com/doc/refman/9.3/en/innodb-information-schema-transactions.html "17.15.2 InnoDB INFORMATION_SCHEMA Transaction and Locking Information")) and Performance Schema [`data_locks`](https://dev.mysql.com/doc/refman/9.3/en/performance-schema-data-locks-table.html "29.12.13.1 The data_locks Table") and [`data_lock_waits`](https://dev.mysql.com/doc/refman/9.3/en/performance-schema-data-lock-waits-table.html "29.12.13.2 The data_lock_waits Table") tables.
## Transaction Isolation Levels

Transaction isolation là một trong những nền tảng của xử lý cơ sở dữ liệu. Isolation là chữ I trong từ viết tắt [ACID](https://dev.mysql.com/doc/refman/9.3/en/glossary.html#glos_acid "ACID"); isolation level là thiết lập tinh chỉnh sự cân bằng giữa performance và reliability, consistency, và reproducibility of results khi nhiều giao dịch thực hiện thay đổi và truy vấn cùng lúc.

InnoDB cung cấp cả bốn mức cô lập giao dịch được mô tả theo tiêu chuẩn SQL:1992: [`READ UNCOMMITTED`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted), [`READ COMMITTED`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_read-committed), [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read), và [`SERIALIZABLE`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_serializable). Mức cô lập mặc định của InnoDB là [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read).

Người dùng có thể thay đổi isolation level cho một phiên làm việc hoặc cho tất cả các kết nối tiếp theo bằng câu lệnh SET TRANSACTION. Để đặt isolation level mặc định của máy chủ cho tất cả các kết nối, hãy sử dụng tùy chọn `--transaction-isolation` trên dòng lệnh hoặc trong tệp tùy chọn. Để biết thông tin chi tiết về mức cô lập và cú pháp thiết lập mức, hãy xem [Section 15.3.7, “SET TRANSACTION Statement”](https://dev.mysql.com/doc/refman/9.3/en/set-transaction.html "15.3.7 SET TRANSACTION Statement").

InnoDB hỗ trợ từng transaction isolation levels được mô tả ở đây bằng các chiến lược [locking](https://dev.mysql.com/doc/refman/9.3/en/glossary.html#glos_locking "locking") khác nhau. Bạn có thể áp dụng mức độ nhất quán cao với mức [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read) mặc định, cho các thao tác trên dữ liệu quan trọng mà việc tuân thủ ACID là quan trọng. Hoặc bạn có thể nới lỏng các quy tắc nhất quán (consistency) với [`READ COMMITTED`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) hoặc thậm chí [`READ UNCOMMITTED`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted), trong các tình huống như báo cáo hàng loạt, nơi tính nhất quán chính xác (precise consistency) và kết quả lặp lại (repeatable results) không quan trọng bằng việc giảm thiểu chi phí khóa. [`SERIALIZABLE`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_serializable) áp dụng các quy tắc thậm chí còn nghiêm ngặt hơn [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/9.3/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read) và chủ yếu được sử dụng trong các tình huống chuyên biệt, chẳng hạn như với các giao dịch [XA](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_xa) và để khắc phục sự cố concurrency và [deadlocks](https://dev.mysql.com/doc/refman/9.3/en/glossary.html#glos_deadlock "deadlock").

Danh sách sau đây mô tả cách MySQL hỗ trợ các cấp độ giao dịch khác nhau. Danh sách được sắp xếp theo thứ tự từ cấp độ phổ biến nhất đến cấp độ ít được sử dụng nhất.

- `REPEATABLE READ`
Đây là isolation level mặc định cho InnoDB. Các lần đọc nhất quán trong cùng một giao dịch sẽ đọc snapshot được thiết lập bởi lần đọc đầu tiên. Điều này có nghĩa là nếu bạn phát hành nhiều câu lệnh SELECT thuần túy (không khóa) trong cùng một giao dịch, các câu lệnh SELECT này cũng nhất quán với nhau. Xem [Section 17.7.2.3, “Consistent Nonlocking Reads”](https://dev.mysql.com/doc/refman/8.4/en/innodb-consistent-read.html "17.7.2.3 Consistent Nonlocking Reads").
Đối với [locking reads](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_locking_read "locking read") (SELECT với các lệnh FOR UPDATE hoặc FOR SHARE), UPDATE và DELETE, việc khóa phụ thuộc vào việc lệnh đó sử dụng chỉ mục duy nhất với điều kiện tìm kiếm duy nhất hay điều kiện tìm kiếm theo kiểu phạm vi.
- Đối với một unique index có điều kiện tìm kiếm duy nhất, InnoDB chỉ khóa bản ghi chỉ mục được tìm thấy, không khóa [gap](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_gap "gap") trước đó.
- Đối với các điều kiện tìm kiếm khác, InnoDB khóa phạm vi chỉ mục đã quét, sử dụng [gap locks](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_gap_lock "gap lock") hoặc [next-key locks](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_next_key_lock "next-key lock") để chặn việc chèn dữ liệu của các phiên khác vào các khoảng trống được bao phủ bởi phạm vi. Để biết thêm thông tin về khóa khoảng trống và khóa khóa tiếp theo, hãy xem [Section 17.7.1, “InnoDB Locking”](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html "17.7.1 InnoDB Locking").
Không nên kết hợp các câu lệnh khóa (UPDATE, INSERT, DELETE hoặc SELECT ... FOR ...) với các câu lệnh SELECT non-locking trong một giao dịch REPEATABLE READ duy nhất, vì thông thường trong những trường hợp như vậy, bạn cần SERIALIZABLE. Điều này là do câu lệnh SELECT non-locking sẽ hiển thị trạng thái của cơ sở dữ liệu từ chế độ xem đọc, bao gồm các giao dịch đã cam kết trước khi chế độ xem đọc được tạo và trước khi giao dịch hiện tại ghi dữ liệu, trong khi các câu lệnh khóa sử dụng trạng thái gần đây nhất của cơ sở dữ liệu để khóa. Nhìn chung, hai trạng thái bảng khác nhau này không nhất quán với nhau và khó phân tích cú pháp.



