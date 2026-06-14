**Table of Contents**

[26.1 Overview of Partitioning in MySQL](https://dev.mysql.com/doc/refman/9.7/en/partitioning-overview.html)

[26.2 Partitioning Types](https://dev.mysql.com/doc/refman/9.7/en/partitioning-types.html)

[26.3 Partition Management](https://dev.mysql.com/doc/refman/9.7/en/partitioning-management.html)

[26.4 Partition Pruning](https://dev.mysql.com/doc/refman/9.7/en/partitioning-pruning.html)

[26.5 Partition Selection](https://dev.mysql.com/doc/refman/9.7/en/partitioning-selection.html)

[26.6 Restrictions and Limitations on Partitioning](https://dev.mysql.com/doc/refman/9.7/en/partitioning-limitations.html)

Chương này thảo luận về user-defined partitioning.

>[!note]
>Phân vùng bảng khác với phân vùng được sử dụng bởi các window functions. Để biết thêm [Section 14.20, “Window Functions”](https://dev.mysql.com/doc/refman/9.7/en/window-functions.html "14.20 Window Functions").

Trong MySQL 9.7, tính năng phân vùng được hỗ trợ bởi các công cụ lưu trữ InnoDB và NDB.

MySQL 9.7 hiện không hỗ trợ phân vùng bảng bằng bất kỳ công cụ lưu trữ nào khác ngoài `InnoDB` hoặc `NDB`, chẳng hạn như `MyISAM`. Việc cố gắng tạo bảng phân vùng bằng công cụ lưu trữ không hỗ trợ phân vùng gốc sẽ thất bại với lỗi `ER_CHECK_NOT_IMPLEMENTED`.

Nếu bạn đang biên dịch MySQL 9.7 từ mã nguồn, việc cấu hình bản dựng với hỗ trợ `InnoDB` là đủ để tạo ra các tệp nhị phân có hỗ trợ phân vùng cho các bảng `InnoDB`. Để biết thêm thông tin, hãy xem [Section 2.8, “Installing MySQL from Source”](https://dev.mysql.com/doc/refman/9.7/en/source-installation.html "2.8 Installing MySQL from Source").

Không cần thực hiện thêm bất cứ thao tác nào để kích hoạt hỗ trợ phân vùng bởi `InnoDB` (ví dụ: không cần thêm bất kỳ mục đặc biệt nào trong tệp `my.cnf`).

Không thể vô hiệu hóa tính năng hỗ trợ phân vùng của công cụ lưu trữ `InnoDB`.

Xem [Section 26.1, “Overview of Partitioning in MySQL”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-overview.html "26.1 Overview of Partitioning in MySQL"), để tìm hiểu về phân vùng và các khái niệm liên quan đến phân vùng.

Hệ thống hỗ trợ nhiều loại phân vùng khác nhau, cũng như phân vùng con; xem [Section 26.2, “Partitioning Types”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-types.html "26.2 Partitioning Types"), và [Section 26.2.6, “Subpartitioning”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-subpartitions.html "26.2.6 Subpartitioning").

[Section 26.3, “Partition Management”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-management.html "26.3 Partition Management"), đề cập đến các phương pháp thêm, xóa và sửa đổi các phân vùng trong các bảng đã được phân vùng.

[Section 26.3.4, “Maintenance of Partitions”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-maintenance.html "26.3.4 Maintenance of Partitions"), thảo luận về các lệnh bảo trì bảng được sử dụng với các bảng được phân vùng.

Bảng `PARTITIONS` trong cơ sở dữ liệu `INFORMATION_SCHEMA` cung cấp thông tin về các phân vùng và bảng được phân vùng. Xem [Section 28.3.26, “The INFORMATION_SCHEMA PARTITIONS Table”](https://dev.mysql.com/doc/refman/9.7/en/information-schema-partitions-table.html "28.3.26 The INFORMATION_SCHEMA PARTITIONS Table"), để biết thêm thông tin; để xem một số ví dụ về truy vấn đối với bảng này, hãy xem [Section 26.2.7, “How MySQL Partitioning Handles NULL”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-handling-nulls.html "26.2.7 How MySQL Partitioning Handles NULL").

Đối với các sự cố đã biết liên quan đến phân vùng trong MySQL 9.7, xem [Section 26.6, “Restrictions and Limitations on Partitioning”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-limitations.html "26.6 Restrictions and Limitations on Partitioning").

Bạn cũng có thể thấy các tài liệu tham khảo sau đây hữu ích khi làm việc với các bảng được phân vùng.

Tài liệu tham khảo bổ sung. Các nguồn thông tin khác về phân vùng do người dùng định nghĩa trong MySQL bao gồm:

- [MySQL Partitioning Forum](https://forums.mysql.com/list.php?106)
Đây là diễn đàn thảo luận chính thức dành cho những người quan tâm hoặc đang thử nghiệm công nghệ phân vùng MySQL. Diễn đàn này đăng tải các thông báo và cập nhật từ các nhà phát triển MySQL và những người khác. Diễn đàn được giám sát bởi các thành viên của Nhóm Phát triển và Tài liệu về Phân vùng.
- [PlanetMySQL](http://www.planetmysql.org/)
Đây là một trang tin tức về MySQL, đăng tải các bài viết liên quan đến MySQL, rất hữu ích cho bất kỳ ai sử dụng MySQL. Chúng tôi khuyến khích bạn truy cập trang này để tìm các liên kết đến blog của những người đang làm việc với phân vùng MySQL, hoặc để thêm blog của riêng bạn vào danh sách các blog được đề cập.

Nguồn: [[Chapter 26 Partitioning]]