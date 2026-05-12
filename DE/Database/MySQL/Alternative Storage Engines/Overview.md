Storage engines là các thành phần của MySQL xử lý các thao tác SQL cho các loại bảng khác nhau. InnoDB là công cụ lưu trữ mặc định và phổ biến nhất, và Oracle khuyến nghị sử dụng nó cho các bảng ngoại trừ các trường hợp sử dụng đặc biệt. (Câu lệnh [`CREATE TABLE`](https://dev.mysql.com/doc/refman/9.7/en/create-table.html "15.1.24 CREATE TABLE Statement") trong MySQL 9.7 tạo bảng InnoDB theo mặc định.)

Máy chủ MySQL sử dụng kiến ​​trúc công cụ lưu trữ có thể cắm thêm, cho phép các  storage engines được tải vào và gỡ bỏ khỏi máy chủ MySQL đang hoạt động.

Để xác định máy chủ của bạn hỗ trợ những storage engines nào, hãy sử dụng câu lệnh SHOW ENGINES. Giá trị trong cột Support cho biết liệu công cụ đó có thể được sử dụng hay không. Giá trị YES, NO hoặc DEFAULT cho biết công cụ đó có sẵn, không có sẵn hoặc có sẵn và hiện đang được đặt làm công cụ lưu trữ mặc định.

Chương này đề cập đến các trường hợp sử dụng cho các MySQL storage engines chuyên dụng. Nó không đề cập đến InnoDB storage engine mặc định hoặc NDB storage engine, vốn được đề cập trong [Chapter 17, _The InnoDB Storage Engine_](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine"), [Chapter 25, _MySQL NDB Cluster 9.7_](https://dev.mysql.com/doc/refman/9.7/en/mysql-cluster.html "Chapter 25 MySQL NDB Cluster 9.7"). Đối với người dùng nâng cao, chương này cũng bao gồm mô tả về kiến ​​trúc storage engine có thể cắm thêm (xem [Section 18.11, “Overview of MySQL Storage Engine Architecture”](https://dev.mysql.com/doc/refman/9.7/en/pluggable-storage-overview.html "18.11 Overview of MySQL Storage Engine Architecture")).

Để biết thông tin về các tính năng được cung cấp trong các MySQL Server binaries thương mại, hãy xem mục [_MySQL Editions_](https://www.mysql.com/products/) trên trang web MySQL. Các công cụ lưu trữ khả dụng có thể phụ thuộc vào phiên bản MySQL bạn đang sử dụng.

Để tìm câu trả lời cho các câu hỏi thường gặp về công cụ lưu trữ MySQL, hãy xem [Section A.2, “MySQL 9.7 FAQ: Storage Engines”](https://dev.mysql.com/doc/refman/9.7/en/faqs-storage-engines.html "A.2 MySQL 9.7 FAQ: Storage Engines")

## MySQL 9.7 Supported Storage Engines

Nguồn: https://dev.mysql.com/doc/refman/9.7/en/storage-engines.html


