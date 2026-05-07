**Table of Contents**

[19.1 Configuring Replication](https://dev.mysql.com/doc/refman/9.7/en/replication-configuration.html)

[19.2 Replication Implementation](https://dev.mysql.com/doc/refman/9.7/en/replication-implementation.html)

[19.3 Replication Security](https://dev.mysql.com/doc/refman/9.7/en/replication-security.html)

[19.4 Replication Solutions](https://dev.mysql.com/doc/refman/9.7/en/replication-solutions.html)

[19.5 Replication Notes and Tips](https://dev.mysql.com/doc/refman/9.7/en/replication-notes.html)

Replication cho phép sao chép dữ liệu từ một máy chủ cơ sở dữ liệu MySQL (được gọi là nguồn) sang một hoặc nhiều máy chủ cơ sở dữ liệu MySQL khác (được gọi là bản sao). Theo mặc định, Replication là bất đồng bộ; các bản sao không cần phải kết nối liên tục để nhận cập nhật từ nguồn. Tùy thuộc vào cấu hình, bạn có thể sao chép tất cả các database, các database được chọn hoặc thậm chí các bảng được chọn trong một database.

Ưu điểm của việc sao chép dữ liệu trong MySQL bao gồm:

- Scale-out solutions: Các giải pháp mở rộng quy mô phân tán tải trọng giữa nhiều bản sao để cải thiện hiệu suất. Trong môi trường này, tất cả các thao tác ghi và cập nhật phải diễn ra trên máy chủ nguồn. Tuy nhiên, các thao tác đọc có thể diễn ra trên một hoặc nhiều bản sao. Mô hình này có thể cải thiện hiệu suất ghi (vì máy chủ nguồn được dành riêng cho việc cập nhật), đồng thời tăng tốc độ đọc đáng kể trên số lượng bản sao ngày càng tăng.
- Data security: Đảm bảo an toàn dữ liệu vì bản sao có thể tạm dừng quá trình sao chép, cho phép chạy các dịch vụ sao lưu trên bản sao mà không làm hỏng dữ liệu nguồn tương ứng.
- Analytics: Dữ liệu phân tích trực tiếp có thể được tạo trên nguồn, trong khi việc phân tích thông tin có thể diễn ra trên bản sao mà không ảnh hưởng đến hiệu suất của nguồn.
- Long-distance data distribution: Để phân phối dữ liệu đường dài, bạn có thể sử dụng sao chép để tạo bản sao cục bộ của dữ liệu cho địa điểm từ xa sử dụng, mà không cần truy cập vĩnh viễn vào nguồn dữ liệu gốc.

Để biết thông tin về cách sử dụng sao chép dữ liệu trong các trường hợp như vậy, hãy xem [Section 19.4, “Replication Solutions”](https://dev.mysql.com/doc/refman/9.7/en/replication-solutions.html "19.4 Replication Solutions").

MySQL 9.7 hỗ trợ nhiều phương pháp sao chép dữ liệu khác nhau. Phương pháp truyền thống dựa trên việc sao chép các sự kiện từ binary log của nguồn và yêu cầu các tệp nhật ký cũng như vị trí trong đó phải được đồng bộ hóa giữa nguồn và bản sao. Phương pháp mới hơn dựa trên mã định danh giao dịch toàn cầu (GTID) là phương pháp giao dịch và do đó không yêu cầu làm việc với các tệp nhật ký hoặc vị trí trong các tệp này, điều này giúp đơn giản hóa đáng kể nhiều tác vụ sao chép dữ liệu thông thường. Sao chép dữ liệu bằng GTID đảm bảo tính nhất quán giữa nguồn và bản sao miễn là tất cả các giao dịch đã committed trên nguồn cũng đã được áp dụng trên bản sao. Để biết thêm thông tin về GTID và sao chép dựa trên GTID trong MySQL, hãy xem [Section 19.1.3, “Replication with Global Transaction Identifiers”](https://dev.mysql.com/doc/refman/9.7/en/replication-gtids.html "19.1.3 Replication with Global Transaction Identifiers"). Để biết thông tin về việc sử dụng sao chép dựa trên vị trí tệp nhật ký nhị phân, hãy xem [Section 19.1, “Configuring Replication”](https://dev.mysql.com/doc/refman/9.7/en/replication-configuration.html "19.1 Configuring Replication").

Trong MySQL, sao chép dữ liệu hỗ trợ nhiều loại đồng bộ hóa khác nhau. Loại đồng bộ hóa ban đầu là sao chép một chiều, asynchronous replication, trong đó một máy chủ đóng vai trò là nguồn, trong khi một hoặc nhiều máy chủ khác đóng vai trò là bản sao. Điều này trái ngược với sao chép đồng bộ, một đặc điểm của NDB Cluster (xem  [Chapter 25, _MySQL NDB Cluster 9.7_](https://dev.mysql.com/doc/refman/9.7/en/mysql-cluster.html "Chapter 25 MySQL NDB Cluster 9.7")). Trong MySQL 9.7, ngoài tính năng sao chép không đồng bộ tích hợp sẵn, còn hỗ trợ sao chép bán đồng bộ. Với cơ chế sao chép bán đồng bộ, một lệnh commit được thực hiện trên máy chủ nguồn sẽ bị chặn trước khi trả về phiên đã thực hiện giao dịch cho đến khi ít nhất một bản sao xác nhận rằng nó đã nhận và ghi lại các sự kiện cho giao dịch; xem [Section 19.4.10, “Semisynchronous Replication”](https://dev.mysql.com/doc/refman/9.7/en/replication-semisync.html "19.4.10 Semisynchronous Replication"). MySQL 9.7 cũng hỗ trợ sao chép trì hoãn, trong đó một bản sao cố ý chậm hơn máy chủ nguồn ít nhất một khoảng thời gian được chỉ định; xem Mục  [Section 19.4.11, “Delayed Replication”](https://dev.mysql.com/doc/refman/9.7/en/replication-delayed.html "19.4.11 Delayed Replication"). Đối với các trường hợp mà…

Có một số giải pháp khả thi để thiết lập sao chép dữ liệu giữa các máy chủ, và phương pháp tốt nhất cần sử dụng phụ thuộc vào sự hiện diện của dữ liệu và loại công cụ bạn đang sử dụng. Để biết thêm thông tin về các tùy chọn khả dụng, hãy xem [Section 19.1.2, “Setting Up Binary Log File Position Based Replication”](https://dev.mysql.com/doc/refman/9.7/en/replication-howto.html "19.1.2 Setting Up Binary Log File Position Based Replication").

Có hai loại định dạng sao chép cốt lõi: Sao chép dựa trên câu lệnh (Statement Based Replication - SBR), sao chép toàn bộ câu lệnh SQL, và Sao chép dựa trên hàng (Row Based Replication - RBR), chỉ sao chép các hàng đã thay đổi. Bạn cũng có thể sử dụng loại thứ ba, Sao chép hỗn hợp (Mixed Based Replication - MBR). Để biết thêm thông tin về các định dạng sao chép khác nhau, xem [Section 19.1.2, “Setting Up Binary Log File Position Based Replication”](https://dev.mysql.com/doc/refman/9.7/en/replication-howto.html "19.1.2 Setting Up Binary Log File Position Based Replication").

Việc sao chép được kiểm soát thông qua một số tùy chọn và biến khác nhau. Để biết thêm thông tin, hãy xem  [Section 19.1.6, “Replication and Binary Logging Options and Variables”](https://dev.mysql.com/doc/refman/9.7/en/replication-options.html "19.1.6 Replication and Binary Logging Options and Variables"). Các biện pháp bảo mật bổ sung có thể được áp dụng cho cấu trúc sao chép, như được mô tả trong  [Section 19.3, “Replication Security”](https://dev.mysql.com/doc/refman/9.7/en/replication-security.html "19.3 Replication Security").

Bạn có thể sử dụng sao chép dữ liệu để giải quyết nhiều vấn đề khác nhau, bao gồm hiệu suất, hỗ trợ sao lưu các cơ sở dữ liệu khác nhau và như một phần của giải pháp lớn hơn để giảm thiểu sự cố hệ thống. Để biết thông tin về cách giải quyết những vấn đề này, hãy xem [Section 19.4, “Replication Solutions”](https://dev.mysql.com/doc/refman/9.7/en/replication-solutions.html "19.4 Replication Solutions").

Để biết các ghi chú và mẹo về cách xử lý các kiểu dữ liệu và câu lệnh khác nhau trong quá trình sao chép, bao gồm chi tiết về các tính năng sao chép, khả năng tương thích phiên bản, nâng cấp và các sự cố tiềm ẩn cũng như cách giải quyết, hãy xem [Section 19.5, “Replication Notes and Tips”](https://dev.mysql.com/doc/refman/9.7/en/replication-notes.html "19.5 Replication Notes and Tips"). Để tìm câu trả lời cho một số câu hỏi thường gặp của những người mới làm quen với MySQL Replication, hãy xem [Section A.14, “MySQL 9.7 FAQ: Replication”](https://dev.mysql.com/doc/refman/9.7/en/faqs-replication.html "A.14 MySQL 9.7 FAQ: Replication").

Để biết thông tin chi tiết về việc triển khai sao chép, cách thức hoạt động của sao chép, quy trình và nội dung của nhật ký nhị phân, các luồng nền và các quy tắc được sử dụng để quyết định cách ghi lại và sao chép các câu lệnh, hãy xem [Section 19.2, “Replication Implementation”](https://dev.mysql.com/doc/refman/9.7/en/replication-implementation.html "19.2 Replication Implementation").



