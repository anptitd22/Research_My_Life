# Các storage chủ yếu
- InnoDB: storage engine mặc định trong MySQL 9.3. InnoDB là một công cụ lưu trữ an toàn giao dịch (tuân thủ ACID) dành cho MySQL, có khả năng commit, rollback và khôi phục sau sự cố để bảo vệ dữ liệu người dùng. InnoDB row-level locking (không cần nâng cấp lên khóa chi tiết hơn) và Oracle-style consistent nonlocking reads giúp tăng cường khả năng multi-user concurrency and performance. InnoDB lưu trữ dữ liệu người dùng trong các index cụm để giảm I/O cho các truy vấn phổ biến dựa trên khóa chính. Để duy trì tính toàn vẹn dữ liệu, InnoDB cũng hỗ trợ các ràng buộc toàn vẹn tham chiếu KHÓA NGOẠI. Để biết thêm thông tin về InnoDB, xem [Chapter 17, _The InnoDB Storage Engine_](https://dev.mysql.com/doc/refman/9.3/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine").

- MyISAM: Các bảng này có dung lượng nhỏ. [Table-level locking](https://dev.mysql.com/doc/refman/9.3/en/glossary.html#glos_table_lock "table lock") giới hạn hiệu suất trong các khối lượng công việc đọc/ghi, do đó, nó thường được sử dụng trong các khối lượng công việc read-only hoặc chủ yếu đọc trong data warehousing và web.
	- Thực tế, MyISAM được dùng để tạo các bảng chỉ đọc, ít nghiệp vụ, không transaction ví dụ như (bảng lưu xã, huyện, tỉnh, ....)

- Memory: Lưu trữ toàn bộ dữ liệu trong RAM, giúp truy cập nhanh trong các môi trường yêu cầu tra cứu nhanh dữ liệu không quan trọng. Công cụ này trước đây được gọi là công cụ HEAP. Các trường hợp sử dụng của nó đang giảm dần; InnoDB với vùng bộ nhớ đệm cung cấp một giải pháp đa năng và bền vững để lưu trữ hầu hết hoặc toàn bộ dữ liệu trong bộ nhớ, và NDBCLUSTER cung cấp khả năng tra cứu key-value nhanh chóng cho các tập dữ liệu phân tán lớn.
	-  Trên thực tế, các bảng memory dùng để tạo các bảng test nhanh hay tạo các bảng đã được tính toán trước để hiển thị nhanh (vd: tiki,...)

| Tiêu chí              | InnoDB               | MyISAM                   |
| --------------------- | -------------------- | ------------------------ |
| **Transaction**       | ✔ Có (ACID)          | ✖ Không                  |
| **Foreign Key**       | ✔ Có                 | ✖ Không                  |
| **Row-level locking** | ✔                    | ✖ Chỉ table-level        |
| **Crash recovery**    | ✔ Có redo log        | ✖ Không                  |
| **Full-text search**  | ✔ (MySQL 5.6+)       | ✔ (từ lâu)               |
| **Tốc độ**            | Chậm hơn khi chỉ đọc | Nhanh hơn cho read-heavy |
| **Khôi phục lỗi**     | Tự động              | Dễ corrupt dữ liệu       |

| Feature                                | MyISAM       | Memory           | InnoDB       | Archive      | NDB          |
| -------------------------------------- | ------------ | ---------------- | ------------ | ------------ | ------------ |
| B-tree indexes                         | Yes          | Yes              | Yes          | No           | No           |
| Backup/point-in-time recovery (note 1) | Yes          | Yes              | Yes          | Yes          | Yes          |
| Cluster database support               | No           | No               | No           | No           | Yes          |
| Clustered indexes                      | No           | No               | Yes          | No           | No           |
| Compressed data                        | Yes (note 2) | No               | Yes          | Yes          | No           |
| Data caches                            | No           | N/A              | Yes          | No           | Yes          |
| Encrypted data                         | Yes (note 3) | Yes (note 3)     | Yes (note 4) | Yes (note 3) | Yes (note 5) |
| Foreign key support                    | No           | No               | Yes          | No           | Yes          |
| Full-text search indexes               | Yes          | No               | Yes (note 6) | No           | No           |
| Geospatial data type support           | Yes          | No               | Yes          | Yes          | Yes          |
| Geospatial indexing support            | Yes          | No               | Yes (note 7) | No           | No           |
| Hash indexes                           | No           | Yes              | No (note 8)  | No           | Yes          |
| Index caches                           | Yes          | N/A              | Yes          | No           | Yes          |
| Locking granularity                    | Table        | Table            | Row          | Row          | Row          |
| MVCC                                   | No           | No               | Yes          | No           | No           |
| Replication support (note 1)           | Yes          | Limited (note 9) | Yes          | Yes          | Yes          |
| Storage limits                         | 256TB        | RAM              | 64TB         | None         | 384EB        |
| T-tree indexes                         | No           | No               | No           | No           | Yes          |
| Transactions                           | No           | No               | Yes          | No           | Yes          |
| Update statistics for data dictionary  | Yes          | Yes              | Yes          | Yes          | Yes          |

Trích: https://dev.mysql.com/doc/refman/9.3/en/storage-engines.html

# InnoDB

[17.1.1 Benefits of Using InnoDB Tables](https://dev.mysql.com/doc/refman/9.3/en/innodb-benefits.html)

[17.1.2 Best Practices for InnoDB Tables](https://dev.mysql.com/doc/refman/9.3/en/innodb-best-practices.html)

[17.1.3 Verifying that InnoDB is the Default Storage Engine](https://dev.mysql.com/doc/refman/9.3/en/innodb-check-availability.html)

[17.1.4 Testing and Benchmarking with InnoDB](https://dev.mysql.com/doc/refman/9.3/en/innodb-benchmarking.html)

InnoDB là một công cụ lưu trữ đa năng, cân bằng giữa độ tin cậy cao và hiệu suất cao. Trong MySQL 9.3, InnoDB là công cụ lưu trữ MySQL mặc định. Trừ khi bạn đã cấu hình một công cụ lưu trữ mặc định khác, việc phát hành câu lệnh `CREATE TABLE` mà không có mệnh đề ENGINE sẽ tạo ra một bảng InnoDB.
## Key Advantages of InnoDB
- Các hoạt động DML của nó tuân theo mô hình ACID, với các giao dịch có khả năng commit, rollback và phục hồi sau sự cố để bảo vệ dữ liệu người dùng. Xem [Section 17.2, “InnoDB and the ACID Model”](https://dev.mysql.com/doc/refman/9.3/en/mysql-acid.html "17.2 InnoDB and the ACID Model").
- Row-level locking và Oracle-style consistent reads giúp tăng cường khả năng đồng thời và hiệu suất của nhiều người dùng. Xem [Section 17.7, “InnoDB Locking and Transaction Model”](https://dev.mysql.com/doc/refman/9.3/en/innodb-locking-transaction-model.html "17.7 InnoDB Locking and Transaction Model").
- Các bảng InnoDB sắp xếp dữ liệu của bạn trên đĩa để tối ưu hóa các truy vấn dựa trên khóa chính. Mỗi bảng InnoDB có một chỉ mục khóa chính được gọi là clustered index, giúp sắp xếp dữ liệu nhằm giảm thiểu thao tác I/O cho việc tra cứu khóa chính. Xem [Section 15.1.22.5, “FOREIGN KEY Constraints”](https://dev.mysql.com/doc/refman/9.3/en/create-table-foreign-keys.html "15.1.22.5 FOREIGN KEY Constraints").

đọc thêm: https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/innodb-consistent-read.html

**InnoDB Storage Engine Features**

|Feature|Support|
|---|---|
|**B-tree indexes**|Yes|
|**Backup/point-in-time recovery** (Implemented in the server, rather than in the storage engine.)|Yes|
|**Cluster database support**|No|
|**Clustered indexes**|Yes|
|**Compressed data**|Yes|
|**Data caches**|Yes|
|**Encrypted data**|Yes (Implemented in the server via encryption functions; In MySQL 5.7 and later, data-at-rest encryption is supported.)|
|**Foreign key support**|Yes|
|**Full-text search indexes**|Yes (Support for FULLTEXT indexes is available in MySQL 5.6 and later.)|
|**Geospatial data type support**|Yes|
|**Geospatial indexing support**|Yes (Support for geospatial indexing is available in MySQL 5.7 and later.)|
|**Hash indexes**|No (InnoDB utilizes hash indexes internally for its Adaptive Hash Index feature.)|
|**Index caches**|Yes|
|**Locking granularity**|Row|
|**MVCC**|Yes|
|**Replication support** (Implemented in the server, rather than in the storage engine.)|Yes|
|**Storage limits**|64TB|
|**T-tree indexes**|No|
|**Transactions**|Yes|
|**Update statistics for data dictionary**|Yes|
Để so sánh các tính năng của InnoDB với các công cụ lưu trữ khác được cung cấp cùng với MySQL, hãy xem bảng Tính năng của công cụ lưu trữ trong [Chapter 18, _Alternative Storage Engines_](https://dev.mysql.com/doc/refman/9.3/en/storage-engines.html "Chapter 18 Alternative Storage Engines").

# MyISAM

[18.2.1 MyISAM Startup Options](https://dev.mysql.com/doc/refman/9.3/en/myisam-start.html)

[18.2.2 Space Needed for Keys](https://dev.mysql.com/doc/refman/9.3/en/key-space.html)

[18.2.3 MyISAM Table Storage Formats](https://dev.mysql.com/doc/refman/9.3/en/myisam-table-formats.html)

[18.2.4 MyISAM Table Problems](https://dev.mysql.com/doc/refman/9.3/en/myisam-table-problems.html)

MyISAM dựa trên công cụ lưu trữ ISAM cũ hơn (và không còn khả dụng nữa) nhưng có nhiều tiện ích mở rộng hữu ích.

**MyISAM Storage Engine Features**

| Feature                                                                                           | Support                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **B-tree indexes**                                                                                | Yes                                                                                                                                                       |
| **Backup/point-in-time recovery** (Implemented in the server, rather than in the storage engine.) | Yes                                                                                                                                                       |
| **Cluster database support**                                                                      | No                                                                                                                                                        |
| **Clustered indexes**                                                                             | No                                                                                                                                                        |
| **Compressed data**                                                                               | Yes (Compressed MyISAM tables are supported only when using the compressed row format. Tables using the compressed row format with MyISAM are read only.) |
| **Data caches**                                                                                   | No                                                                                                                                                        |
| **Encrypted data**                                                                                | Yes (Implemented in the server via encryption functions.)                                                                                                 |
| **Foreign key support**                                                                           | No                                                                                                                                                        |
| **Full-text search indexes**                                                                      | Yes                                                                                                                                                       |
| **Geospatial data type support**                                                                  | Yes                                                                                                                                                       |
| **Geospatial indexing support**                                                                   | Yes                                                                                                                                                       |
| **Hash indexes**                                                                                  | No                                                                                                                                                        |
| **Index caches**                                                                                  | Yes                                                                                                                                                       |
| **Locking granularity**                                                                           | Table                                                                                                                                                     |
| **MVCC**                                                                                          | No                                                                                                                                                        |
| **Replication support** (Implemented in the server, rather than in the storage engine.)           | Yes                                                                                                                                                       |
| **Storage limits**                                                                                | 256TB                                                                                                                                                     |
| **T-tree indexes**                                                                                | No                                                                                                                                                        |
| **Transactions**                                                                                  | No                                                                                                                                                        |
| **Update statistics for data dictionary**                                                         | Yes                                                                                                                                                       |
Mỗi bảng MyISAM được lưu trữ trên đĩa trong hai tệp. Các tệp có tên bắt đầu bằng tên bảng và có phần mở rộng để chỉ định loại tệp. The data file có phần mở rộng **.MYD (MYData)**. The index file có phần mở rộng **.MYI (MYIndex)**. Định nghĩa bảng được lưu trữ trong từ điển dữ liệu MySQL.

Để chỉ rõ rằng bạn muốn có bảng MyISAM, hãy chỉ ra điều đó bằng tùy chọn bảng ENGINE:
```sql
CREATE TABLE t (i INT) ENGINE = MYISAM;
```

Trong MySQL 9.3, thông thường cần phải sử dụng ENGINE để chỉ định công cụ lưu trữ MyISAM vì InnoDB là công cụ mặc định.

Bạn có thể kiểm tra hoặc sửa chữa các bảng MyISAM bằng [**mysqlcheck**](https://dev.mysql.com/doc/refman/9.3/en/mysqlcheck.html "6.5.3 mysqlcheck — A Table Maintenance Program") client hoặc [**myisamchk**](https://dev.mysql.com/doc/refman/9.3/en/myisamchk.html "6.6.4 myisamchk — MyISAM Table-Maintenance Utility") utility. Bạn cũng có thể nén các bảng MyISAM bằng [**myisampack**](https://dev.mysql.com/doc/refman/9.3/en/myisampack.html "6.6.6 myisampack — Generate Compressed, Read-Only MyISAM Tables") để chiếm ít dung lượng hơn. Xem [Section 6.5.3, “mysqlcheck — A Table Maintenance Program”](https://dev.mysql.com/doc/refman/9.3/en/mysqlcheck.html "6.5.3 mysqlcheck — A Table Maintenance Program"), [Section 6.6.4, “myisamchk — MyISAM Table-Maintenance Utility”](https://dev.mysql.com/doc/refman/9.3/en/myisamchk.html "6.6.4 myisamchk — MyISAM Table-Maintenance Utility"), và [Section 6.6.6, “myisampack — Generate Compressed, Read-Only MyISAM Tables”](https://dev.mysql.com/doc/refman/9.3/en/myisampack.html "6.6.6 myisampack — Generate Compressed, Read-Only MyISAM Tables").

Trong MySQL 9.3, công cụ lưu trữ MyISAM không hỗ trợ phân vùng. Các bảng MyISAM đã phân vùng được tạo trong các phiên bản MySQL trước đó không thể được sử dụng trong MySQL 9.3. Để biết thêm thông tin, hãy xem [Section 26.6.2, “Partitioning Limitations Relating to Storage Engines”](https://dev.mysql.com/doc/refman/9.3/en/partitioning-limitations-storage-engines.html "26.6.2 Partitioning Limitations Relating to Storage Engines"). Để được trợ giúp về việc nâng cấp các bảng này để có thể sử dụng trong MySQL 9.3, hãy xem [Section 3.5, “Changes in MySQL 9.3”](https://dev.mysql.com/doc/refman/9.3/en/upgrading-from-previous-series.html "3.5 Changes in MySQL 9.3").

Bảng MyISAM có các đặc điểm sau:
- `MyISAM` supports concurrent inserts
- ....

# MEMORY
Cơ chế lưu trữ MEMORY (trước đây gọi là HEAP) tạo các bảng chuyên dụng với nội dung được lưu trữ trong bộ nhớ. Do dữ liệu dễ bị hỏng, sự cố phần cứng hoặc mất điện, chỉ nên sử dụng các bảng này làm vùng làm việc tạm thời hoặc bộ nhớ đệm chỉ đọc cho dữ liệu được lấy từ các bảng khác.

| Feature                                                                                           | Support                                                   |
| ------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| **B-tree indexes**                                                                                | Yes                                                       |
| **Backup/point-in-time recovery** (Implemented in the server, rather than in the storage engine.) | Yes                                                       |
| **Cluster database support**                                                                      | No                                                        |
| **Clustered indexes**                                                                             | No                                                        |
| **Compressed data**                                                                               | No                                                        |
| **Data caches**                                                                                   | N/A                                                       |
| **Encrypted data**                                                                                | Yes (Implemented in the server via encryption functions.) |
| **Foreign key support**                                                                           | No                                                        |
| **Full-text search indexes**                                                                      | No                                                        |
| **Geospatial data type support**                                                                  | No                                                        |
| **Geospatial indexing support**                                                                   | No                                                        |
| **Hash indexes**                                                                                  | Yes                                                       |
| **Index caches**                                                                                  | N/A                                                       |
| **Locking granularity**                                                                           | Table                                                     |
| **MVCC**                                                                                          | No                                                        |
| **Replication support** (Implemented in the server, rather than in the storage engine.)           | Limited (See the discussion later in this section.)       |
| **Storage limits**                                                                                | RAM                                                       |
| **T-tree indexes**                                                                                | No                                                        |
| **Transactions**                                                                                  | No                                                        |
| **Update statistics for data dictionary**                                                         | Yes                                                       |
|                                                                                                   |                                                           |

- [When to Use MEMORY or NDB Cluster](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-compared-cluster "When to Use MEMORY or NDB Cluster")
    
- [Partitioning](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-partitioning "Partitioning")
    
- [Performance Characteristics](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-performance-characteristics "Performance Characteristics")
    
- [Characteristics of MEMORY Tables](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-characteristics-of-memory-tables "Characteristics of MEMORY Tables")
    
- [DDL Operations for MEMORY Tables](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-ddl-operations-for-memory-tables "DDL Operations for MEMORY Tables")
    
- [Indexes](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-indexes "Indexes")
    
- [User-Created and Temporary Tables](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-user-created-and-temporary-tables "User-Created and Temporary Tables")
    
- [Loading Data](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-loading-data "Loading Data")
    
- [MEMORY Tables and Replication](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-tables-replication "MEMORY Tables and Replication")
    
- [Managing Memory Use](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-managing-memory-use "Managing Memory Use")
    
- [Additional Resources](https://dev.mysql.com/doc/refman/9.3/en/memory-storage-engine.html#memory-storage-engine-additional-resources "Additional Resources")

## When to Use MEMORY or NDB Cluster
Các nhà phát triển muốn triển khai các ứng dụng sử dụng công cụ lưu trữ MEMORY cho dữ liệu quan trọng, có tính khả dụng cao hoặc được cập nhật thường xuyên nên cân nhắc xem NDB Cluster có phải là lựa chọn tốt hơn hay không. Một trường hợp sử dụng điển hình cho công cụ MEMORY bao gồm các đặc điểm sau:
- Các thao tác liên quan đến dữ liệu tạm thời, không quan trọng như quản lý phiên hoặc lưu trữ đệm. Khi máy chủ MySQL dừng hoặc khởi động lại, dữ liệu trong các bảng MEMORY sẽ bị mất.
- Lưu trữ trong bộ nhớ cho phép truy cập nhanh và độ trễ thấp. Khối lượng dữ liệu có thể chứa hoàn toàn trong bộ nhớ mà không khiến hệ điều hành phải hoán đổi các trang bộ nhớ ảo.
NDB Cluster cung cấp các tính năng giống như công cụ MEMORY nhưng có mức hiệu suất cao hơn và cung cấp các tính năng bổ sung không có trong MEMORY:
- Row-level locking và hoạt động đa luồng để giảm thiểu tranh chấp giữa các clients.
- Khả năng mở rộng ngay cả với các câu lệnh kết hợp có ghi.
- Hoạt động sao lưu đĩa tùy chọn để đảm bảo độ bền dữ liệu.

## Partitioning
Không thể phân vùng các bảng MEMORY.

## Performance Characteristics
Hiệu suất của `MEMORY` bị hạn chế bởi sự tranh chấp phát sinh từ việc thực thi luồng đơn và chi phí khóa bảng khi xử lý các bản cập nhật. Điều này hạn chế khả năng mở rộng khi tải tăng, đặc biệt đối với các lệnh kết hợp bao gồm ghi.

Mặc dù xử lý bảng MEMORY trong bộ nhớ, chúng không nhất thiết nhanh hơn bảng InnoDB trên một busy server, cho các truy vấn đa năng hoặc trong khối lượng công việc đọc/ghi. Cụ thể, việc khóa bảng liên quan đến việc thực hiện cập nhật có thể làm chậm việc sử dụng đồng thời các bảng MEMORY từ nhiều phiên.

Tùy thuộc vào loại truy vấn được thực hiện trên bảng MEMORY, bạn có thể tạo index dưới dạng cấu trúc dữ liệu băm mặc định (để tra cứu các giá trị đơn dựa trên khóa duy nhất) hoặc cấu trúc dữ liệu B-tree đa năng (cho tất cả các loại truy vấn liên quan đến toán tử bằng, bất đẳng thức hoặc phạm vi như nhỏ hơn hoặc lớn hơn). Các phần sau minh họa cú pháp để tạo cả hai loại chỉ mục. Một vấn đề hiệu suất phổ biến là sử dụng chỉ mục băm mặc định trong các khối lượng công việc mà chỉ mục B-tree hiệu quả hơn.

## Characteristics of MEMORY Tables
Cơ chế lưu trữ MEMORY không tạo bất kỳ tệp nào trên đĩa. Định nghĩa bảng được lưu trữ trong từ điển dữ liệu MySQL.


