Phần này giúp bạn hiểu rõ hơn về Trino để các quản trị viên và người dùng tiềm năng biết được những gì có thể mong đợi từ Trino.

## What Trino is not

Vì nhiều thành viên trong cộng đồng gọi Trino là cơ sở dữ liệu nên việc bắt đầu bằng định nghĩa về những gì về Trino không phải là là điều hợp lý. 

Đừng nhầm lẫn việc Trino hiểu SQL với việc nó cung cấp các tính năng của một cơ sở dữ liệu chuẩn. Trino không phải là một cơ sở dữ liệu quan hệ đa năng. Nó không phải là sự thay thế cho các cơ sở dữ liệu như MySQL, PostgreSQL hay Oracle. Trino không được thiết kế để xử lý Online Transaction Processing (OLTP). Điều này cũng đúng với nhiều cơ sở dữ liệu khác được thiết kế và tối ưu hóa cho kho dữ liệu hoặc phân tích.

## What Trino is

Trino là một công cụ được thiết kế để truy vấn hiệu quả khối lượng dữ liệu khổng lồ bằng các truy vấn phân tán. Nếu bạn làm việc với hàng terabyte hoặc petabyte dữ liệu, rất có thể bạn đang sử dụng các công cụ tương tác với Hadoop và HDFS. Trino được thiết kế như một giải pháp thay thế cho các công cụ truy vấn HDFS bằng các pipeline của các tác vụ MapReduce, chẳng hạn như Hive hoặc Pig, nhưng Trino không chỉ giới hạn ở việc truy cập HDFS. Trino có thể và đã được mở rộng để hoạt động trên nhiều loại nguồn dữ liệu khác nhau, bao gồm cơ sở dữ liệu quan hệ truyền thống và các nguồn dữ liệu khác như Cassandra.

Trino được thiết kế để xử lý kho dữ liệu và phân tích: phân tích dữ liệu, tổng hợp lượng lớn dữ liệu và tạo báo cáo. Các khối lượng công việc này thường được phân loại là Online Analytical Processing (OLAP).
