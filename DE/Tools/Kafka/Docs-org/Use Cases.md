## Messaging

Kafka hoạt động tốt như một sự thay thế cho các message broker truyền thống. Các message broker được sử dụng vì nhiều lý do (để tách biệt quá trình xử lý khỏi quá trình tạo dữ liệu, để lưu trữ tạm thời các tin nhắn chưa được xử lý, v.v.). So với hầu hết các messaging systems, Kafka có thông lượng (throughput) tốt hơn, phân vùng tích hợp, sao chép và khả năng chịu lỗi, điều này làm cho nó trở thành một giải pháp tốt cho các ứng dụng xử lý message quy mô lớn.

Theo kinh nghiệm của chúng tôi, việc sử dụng messaging thường có lưu lượng tương đối thấp, nhưng có thể yêu cầu độ trễ đầu cuối thấp và thường phụ thuộc vào sự đảm bảo độ bền mạnh mẽ mà Kafka cung cấp.

Trong lĩnh vực này, Kafka có thể so sánh với các messaging systems truyền thống như ActiveMQ hoặc RabbitMQ.

## Website Activity Tracking

Mục đích sử dụng ban đầu của Kafka là để xây dựng lại quy trình theo dõi hoạt động người dùng dưới dạng một tập hợp các luồng dữ liệu publish-subscribe theo thời gian thực. Điều này có nghĩa là hoạt động trên trang web (lượt xem trang, tìm kiếm hoặc các hành động khác mà người dùng có thể thực hiện) được publish đến các topics trung tâm, với một  cho mỗi loại hoạt động. Các luồng dữ liệu này có sẵn để đăng ký cho nhiều trường hợp sử dụng khác nhau, bao gồm xử lý thời gian thực, giám sát thời gian thực và tải vào Hadoop hoặc các hệ thống kho dữ liệu ngoại tuyến để xử lý và báo cáo ngoại tuyến.

Việc theo dõi hoạt động thường có khối lượng dữ liệu rất lớn vì nhiều thông báo hoạt động được tạo ra cho mỗi lượt xem trang của người dùng.

## Metrics

Kafka thường được sử dụng để giám sát dữ liệu hoạt động. Điều này bao gồm việc tổng hợp số liệu thống kê từ các ứng dụng phân tán để tạo ra các nguồn cấp dữ liệu hoạt động tập trung.

## Log Aggregation

Nhiều người sử dụng Kafka để thay thế cho giải pháp tổng hợp log. Thông thường, việc tổng hợp log thu thập các tệp log vật lý từ các máy chủ và đặt chúng ở một vị trí trung tâm (có thể là máy chủ tệp hoặc HDFS) để xử lý. Kafka trừu tượng hóa các chi tiết của tệp và cung cấp một sự trừu tượng rõ ràng hơn về dữ liệu log hoặc event dưới dạng luồng message. Điều này cho phép xử lý với độ trễ thấp hơn và hỗ trợ dễ dàng hơn cho nhiều nguồn dữ liệu và consume dữ liệu phân tán. So với các hệ thống tập trung vào log như Scribe hoặc Flume, Kafka cung cấp hiệu suất tương đương, đảm bảo độ bền mạnh mẽ hơn nhờ cơ chế sao chép và độ trễ end-to-end thấp hơn nhiều.

## Stream Processing

Nhiều người dùng Kafka xử lý dữ liệu trong các pipelines xử lý gồm nhiều giai đoạn, trong đó dữ liệu đầu vào thô được consumed từ các Kafka topic và sau đó được tổng hợp, làm giàu hoặc chuyển đổi thành các topic mới để consume hoặc xử lý tiếp theo. Ví dụ, một pipeline xử lý để đề xuất bài báo có thể thu thập nội dung bài báo từ nguồn cấp dữ liệu RSS và publish nó vào topic “bài báo”; quá trình xử lý tiếp theo có thể chuẩn hóa hoặc loại bỏ nội dung trùng lặp này và publish nội dung bài báo đã được làm sạch vào một topic mới; giai đoạn xử lý cuối cùng có thể cố gắng đề xuất nội dung này cho người dùng. Các pipeline xử lý như vậy tạo ra đồ thị luồng dữ liệu thời gian thực dựa trên các topic riêng lẻ. Bắt đầu từ phiên bản 0.10.0.0, một thư viện xử lý luồng dữ liệu nhẹ nhưng mạnh mẽ có tên là [Kafka Streams](https://kafka.apache.org/documentation/streams) đã có sẵn trong Apache Kafka để thực hiện việc xử lý dữ liệu như mô tả ở trên. Ngoài Kafka Streams, các công cụ xử lý luồng dữ liệu mã nguồn mở khác bao gồm [Apache Storm](https://storm.apache.org/) and [Apache Samza](https://samza.apache.org/).

## Event Sourcing

Event sourcing là một kiểu thiết kế ứng dụng trong đó các thay đổi trạng thái được ghi lại dưới dạng một chuỗi các bản ghi được sắp xếp theo thời gian. Khả năng hỗ trợ lưu trữ dữ liệu log rất lớn của Kafka khiến nó trở thành một hệ thống phụ trợ tuyệt vời cho một ứng dụng được xây dựng theo kiểu này.

## Commit Log

Kafka có thể đóng vai trò như một loại commit-log bên ngoài cho một hệ thống phân tán. Log này giúp sao chép dữ liệu giữa các node và hoạt động như một cơ chế đồng bộ hóa lại cho các node bị lỗi để khôi phục dữ liệu của chúng. Tính năng [log compaction](https://kafka.apache.org/documentation.html#compaction) trong Kafka hỗ trợ việc sử dụng này. Trong trường hợp này, Kafka tương tự như dự án [Apache BookKeeper](https://bookkeeper.apache.org/).