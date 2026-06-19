## Introduction

Lần đầu tiên tôi làm việc với hàng đợi là vào năm 2019 khi còn là Kỹ sư Dữ liệu tại Terragon. Tôi quản lý một hệ thống mà trong đó tôi nhận dữ liệu thông qua một endpoint, sau đó đẩy dữ liệu vào hàng đợi để một dịch vụ khác tiêu thụ và xử lý, rồi lại đẩy vào một hàng đợi khác, và cuối cùng dịch vụ đó sẽ tiêu thụ và xử lý.

Tôi thấy điều này rất thú vị vì trước đây tôi chủ yếu quen thuộc với các chu kỳ request-response thông thường mà chúng ta đều học khi bắt đầu làm việc với backend. Điều buồn cười là tôi đã từng sử dụng Celery cho các dự án cá nhân nhưng lúc đó tôi không hiểu nó là "queues and workers". Tôi chỉ coi nó như một "mẹo" để làm cho các requests "chạy nhanh hơn".

Bạn thấy đấy, sau 6-7 năm xây dựng phần mềm trong nhiều ngành công nghiệp khác nhau, hầu như không có hệ thống nào tôi làm việc mà không sử dụng một loại hàng đợi nào đó. Từ `pgboss`, `Redis Bull`, `SQS` và hai cái tôi thích nhất: `Kafka` và `RabbitMQ`.

Hệ thống xử lý dữ liệu mà tôi bắt đầu bài viết này được xây dựng bằng cả hai công nghệ đó, và tôi luôn tự hỏi, điều gì tạo nên sự khác biệt giữa chúng? Tại sao tôi nên sử dụng `Kafka` thay vì `RabbitMQ` và ngược lại?

Vì vậy, tôi quyết định dành thời gian tìm hiểu, nghiên cứu, hồi tưởng và nắm bắt cả hai hệ thống. Mục đích, ưu điểm và nhược điểm của chúng.

Một điều tôi học được về kỹ thuật phần mềm là bạn có thể đạt được bất cứ điều gì với bất cứ thứ gì nếu bạn biết cách tùy chỉnh nó sao cho phù hợp với sở thích của mình. Vì vậy, bạn có thể nhận thấy mọi người sử dụng Kafka cho những mục đích mà RabbitMQ được xây dựng cho và ngược lại. Trong bài viết này, chúng ta sẽ tập trung vào mục đích sử dụng và những điểm khác biệt rõ rệt.

Một điều tôi học được về kỹ thuật phần mềm là bạn có thể đạt được bất cứ điều gì với bất cứ thứ gì nếu bạn biết cách tùy chỉnh nó sao cho phù hợp với sở thích của mình. Vì vậy, bạn có thể nhận thấy mọi người sử dụng Kafka cho những mục đích mà RabbitMQ được xây dựng cho và ngược lại. Trong bài viết này, chúng ta sẽ tập trung vào mục đích sử dụng và những điểm khác biệt rõ rệt.

## Understanding Asynchronous Systems

Khi các hệ thống phụ trợ ngày càng phức tạp, nhu cầu xử lý bất đồng bộ trở nên thiết yếu. Người dùng mong muốn phản hồi nhanh chóng, nhưng nhiều thao tác cần thời gian, từ gửi email, xử lý thanh toán đến tạo báo cáo. Đây là lúc hàng đợi phát huy tác dụng.

Cả RabbitMQ và Kafka đều được sử dụng rộng rãi để xây dựng các hệ thống bất đồng bộ này, nhưng các kỹ sư thường gặp khó khăn trong việc quyết định công cụ nào phù hợp với vấn đề nào.

Chìa khóa để đưa ra quyết định này là hiểu rõ hai mô hình khác biệt:
- Background Workers
- Event-Driven Processing

## Background Workers

Background workers là phương án đơn giản hơn trong hai phương án.

Ở đây, bạn có một tác vụ cần được xử lý, nhưng bạn không muốn xử lý nó trong chu trình request–response vì sẽ mất quá nhiều thời gian. Vì vậy, thay vào đó, bạn đẩy tác vụ vào một hàng đợi và để một dịch vụ riêng biệt (một worker) xử lý nó một cách bất đồng bộ.

Các ví dụ điển hình bao gồm:

- Sending emails
- Processing payments
- Generating PDFs
- Calling third-party APIs

Trong cấu hình này:

- Một task chỉ nên do một worker đảm nhiệm.
- Khi nhiệm vụ hoàn thành, nó sẽ kết thúc.
- Bản thân task không phải là điều bạn quan tâm lâu dài.

## Event-Driven Processing

Xử lý theo sự kiện khác biệt cả về quy mô và mục đích.

Mặc dù background workers có thể được điều khiển bởi event theo nghĩa rộng, nhưng kiến ​​trúc event-driven thường bao gồm nhiều tiến trình độc lập phản ứng với cùng một event.

Ví dụ, khi người dùng hoàn tất thanh toán, bạn có thể muốn kích hoạt một số hành động sau:

- Service A — Send a confirmation email
- Service B — Update the payment service
- Service C — Update the order service
- Service D — Select a driver for delivery
- Service E — Notify the vendor of a successful order
- Service F — Run analytics later in the night
- …and possibly more in the future

Ở giai đoạn này, vấn đề không còn là hoàn thành một task đơn lẻ nữa. Mà là truyền tải thông tin về việc một sự kiện đã xảy ra và cho phép nhiều dịch vụ phản hồi theo cách riêng của chúng.

Một chi tiết quan trọng ở đây là thời gian. Mặc dù hệ thống hoạt động dựa trên event, nhưng không phải tất cả người dùng đều xử lý event ngay lập tức. Một số dịch vụ phản hồi trong thời gian thực, trong khi những dịch vụ khác (như phân tích hoặc báo cáo) có thể xử lý cùng một event đó sau vài giờ hoặc vài ngày.

Sự khác biệt này rất quan trọng, lập trình hướng sự kiện không phải lúc nào cũng có nghĩa là hoạt động trong thời gian thực.

## Why This Distinction Matters

Các hệ thống làm việc nền tập trung vào việc hoàn thành công việc. Các hệ thống hướng events tập trung vào việc phản ứng với các events.

Giờ đây, khi chúng ta đã làm rõ sự khác biệt giữa các background workers và event-driven processing, đây là lúc Kafka và RabbitMQ phát huy vai trò của mình.

## Kafka and RabbitMQ

Nói một cách đơn giản, RabbitMQ được xây dựng để background processing, trong khi Kafka được xây dựng cho các event-driven systems và có những lý do kiến ​​trúc rõ ràng cho điều này.

RabbitMQ được thiết kế để chuyển một task đến consumer, đảm bảo task đó được xử lý và sau đó loại bỏ nó khỏi hàng đợi. Một message được gửi đến hàng đợi, được consume bởi một worker, được xác nhận và bị xóa. RabbitMQ cũng cung cấp các cơ chế tích hợp sẵn cho việc thử lại, hàng đợi thư chết và phân phối công bằng, đảm bảo rằng một task chỉ được xử lý bởi một consumer tại một thời điểm.

Ngược lại, Kafka được xây dựng như một distributed log. Khi một event được ghi vào Kafka, nó sẽ được lưu giữ ở đó trong một khoảng thời gian được cấu hình (ngày, tuần, hoặc thậm chí vô thời hạn). Consumer không consume và xóa tin nhắn mà thay vào đó họ đọc từ log theo tốc độ của riêng mình. Nhiều consumer có thể đọc cùng một event, và một consumer thậm chí có thể quay lại và đọc lại các event từ nhiều giờ hoặc nhiều ngày trước đó. 

Sự khác biệt về khả năng lưu trữ dữ liệu này là rất quan trọng. Kafka lưu giữ các event, cho phép consumer đọc, tạm dừng, tiếp tục hoặc phát lại chúng khi cần. RabbitMQ không lưu giữ messages sau khi chúng được xác nhận (sau khi một job được xử lý), mà sẽ bị xóa khỏi hàng đợi.

Hiểu rõ những khác biệt này sẽ giúp ta hiểu tại sao RabbitMQ phù hợp với các background workers, trong khi Kafka lại phù hợp với các kiến ​​trúc event-driven.

## The Key Technical Differences

Tôi sẽ phân tích những khác biệt kỹ thuật quan trọng nhất cần lưu ý khi lựa chọn giữa các công cụ này.

**Message Retention**

RabbitMQ xử lý các tin nhắn giống như các mục trong danh sách việc cần làm. Khi một worker nhận nhiệm vụ, xử lý và xác nhận, message sẽ bị xóa. Điều này hoàn toàn hợp lý đối với các tác vụ nền, tại sao phải lưu giữ hồ sơ của mọi email bạn đã gửi hoặc mọi tệp PDF bạn đã tạo?

Kafka xử lý các messages như các mục nhập trong nhật ký. Mỗi event được ghi vào nhật ký và lưu giữ trong một khoảng thời gian đã được cấu hình. Điều này có nghĩa là Dịch vụ F (phân tích) có thể xử lý sự kiện thanh toán lúc nửa đêm, ngay cả khi nó xảy ra lúc 2 giờ chiều. Sự kiện vẫn còn đó, chờ được đọc.

**Replay Capability**

Đây là lúc Kafka phát huy tối đa ưu điểm. Giả sử bạn đã triển khai một lỗi trong dịch vụ phân tích của mình khiến doanh thu tuần trước bị tính toán sai. Với Kafka, bạn có thể sửa lỗi, tua lại tiến trình xử lý của consumer về thứ Hai tuần trước và xử lý lại tất cả các sự kiện thanh toán đó. Với RabbitMQ, những thông báo đó sẽ biến mất. Bạn sẽ cần phải xây dựng lại dữ liệu từ cơ sở dữ liệu của mình (nếu bạn có lưu trữ nó).

**Consumer Model**

Kafka sử dụng mô hình kéo (push model). Consumers yêu cầu message khi chúng sẵn sàng xử lý. Điều này cho phép các consumers kiểm soát thông lượng của chúng và cho phép chúng tạm dừng và tiếp tục bất cứ lúc nào.

RabbitMQ sử dụng mô hình đẩy (push model). Broker sẽ đẩy các messages đến consumers khi chúng đến. RabbitMQ xử lý việc phân phối và cố gắng cân bằng tải giữa các consumers. Điều này hoạt động rất tốt đối với các nhóm worker, nơi bạn muốn phân bổ công việc một cách công bằng.

## When to Choose RabbitMQ

RabbitMQ thể hiện ưu điểm vượt trội khi bạn xây dựng các background worker truyền thống. Dưới đây là những trường hợp RabbitMQ là lựa chọn phù hợp:

- Bạn chỉ cần xử lý một tác vụ một lần và thế là xong. Gửi email chào mừng, xử lý hình ảnh đã tải lên, gọi API thanh toán. Đây là những thao tác "gửi đi và quên". Sau khi hoàn thành, bạn không cần message đó nữa.

- Sự đơn giản trong vận hành rất quan trọng. Về mặt khái niệm, RabbitMQ dễ thiết lập và quản lý hơn. Bạn tạo hàng đợi, producers gửi messages, consumers xử lý chúng. Quá trình học tập dễ dàng hơn và chi phí vận hành thấp hơn.

- Bạn cần các mô hình xử lý tác vụ tích hợp sẵn. RabbitMQ có hỗ trợ tích hợp tuyệt vời cho hàng đợi tác vụ, cơ chế thử lại, hàng đợi thư chết và hàng đợi ưu tiên. Các mô hình này được tích hợp sẵn trong hệ thống và hoạt động ngay lập tức.

- Yêu cầu về thông lượng của bạn ở mức trung bình. Nếu bạn xử lý hàng nghìn tác vụ mỗi giây (không phải hàng triệu), RabbitMQ sẽ xử lý tốt mà không cần đến sự phức tạp trong vận hành của Kafka.

## When to Choose Kafka

Kafka là lựa chọn phù hợp khi bạn xây dựng các event-driven systems với những yêu cầu cụ thể:

- Nhiều dịch vụ cần phản hồi cùng một event. Sự kiện thanh toán mà tôi đã đề cập trước đó? Sáu dịch vụ khác nhau quan tâm đến nó, và tất cả đều cần xử lý nó một cách độc lập. Kafka giúp việc này trở nên đơn giản, mỗi dịch vụ có consumer group riêng và đọc cùng một event với tốc độ riêng của mình.

- Bạn cần khả năng phát lại. Cho dù đó là để khắc phục lỗi, xử lý lại dữ liệu với logic mới hay cung cấp dữ liệu lịch sử cho một dịch vụ mới, khả năng lưu trữ của Kafka đều cho phép điều này. Điều này rất quan trọng khi bạn xây dựng các data pipelines hoặc hệ thống phân tích.

- Bạn đang xây dựng một **data pipeline**. Nếu các event chảy qua hệ thống của bạn đại diện cho những thay đổi trạng thái mà nhiều hệ thống hạ nguồn cần biết, thì Kafka được thiết kế dành riêng cho việc này. Bạn có thể có cả consumers thời gian thực và consumers theo lô đọc cùng một luồng dữ liệu.

- Khả năng mở rộng là một vấn đề cần quan tâm. Kafka được xây dựng để xử lý hàng triệu messages mỗi giây trên các cụm phân tán. Nếu bạn đang xử lý các luồng sự kiện có khối lượng lớn (dữ liệu clickstream, cảm biến IoT, phân tích thời gian thực), kiến ​​trúc của Kafka sẽ xử lý tốt hơn.

- Tại Terragon, chúng tôi sử dụng Kafka cho data pipeline cốt lõi của mình. Dữ liệu sẽ đến từ nhiều nguồn khác nhau, được ghi vào các Kafka topics, và sau đó nhiều dịch vụ sẽ tiêu thụ dữ liệu đó - một số để xử lý thời gian thực, một số để rút ra thông tin chi tiết. Khả năng cho phép các consumers khác nhau đọc dữ liệu từ cùng một luồng dữ liệu với tốc độ khác nhau chính xác là những gì chúng tôi cần.

## Common Mistakes I’ve Seen

Sử dụng Kafka cho các hàng đợi công việc đơn giản. Tôi đã thấy nhiều nhóm thiết lập các cụm Kafka chỉ để gửi email hoặc xử lý các tác vụ nền đơn giản. Chi phí vận hành của Kafka (ZooKeeper, broker, managing partitions, consumer groups) là rất đáng kể. Đối với các hàng đợi công việc đơn giản, điều này là quá mức cần thiết. Hãy sử dụng RabbitMQ hoặc thậm chí Redis với Bull.

Sử dụng RabbitMQ khi bạn cần dữ liệu lịch sử. Một nhóm đã từng cố gắng xây dựng một hệ thống phân tích dữ liệu bằng RabbitMQ. Họ nhanh chóng nhận ra rằng nếu bất kỳ consumer nào gặp sự cố, họ sẽ mất dữ liệu. Cuối cùng, họ phải lưu trữ mọi thứ vào cơ sở dữ liệu chỉ để có bản sao lưu, về cơ bản là tạo ra phiên bản nhật ký kém chất lượng của Kafka.

Chưa kể đến độ phức tạp trong vận hành. Kafka mạnh mẽ nhưng phức tạp. Bạn cần hiểu về partition, consumer group, rebalancing và quản lý offset. Nếu nhóm của bạn nhỏ hoặc thiếu kinh nghiệm với hệ thống phân tán, gánh nặng vận hành có thể lớn hơn lợi ích. Đôi khi, công cụ "kém hơn" mà nhóm của bạn thực sự có thể quản lý lại là lựa chọn tốt hơn.

Bỏ qua các trường hợp ngoại lệ. Điều gì xảy ra khi consumer ngừng hoạt động trong hai giờ? Với RabbitMQ, các message sẽ được xếp hàng chờ trong bộ nhớ (điều này có thể gây ra sự cố). Với Kafka, chúng nằm trong nhật ký chờ được đọc. Không có cái nào tốt hơn cái nào một cách tuyệt đối, điều đó phụ thuộc vào trường hợp sử dụng của bạn.

## Conclusion

RabbitMQ và Kafka đều là các message systems, nhưng chúng được xây dựng cho những mục đích hoàn toàn khác nhau. RabbitMQ là một trình message broker được thiết kế để thực hiện các tác vụ. Kafka là một nền tảng truyền phát sự kiện được thiết kế để broadcasting các sự kiện và xây dựng các data pipelines.

Sự khác biệt giữa background workers và event-driven processing là chìa khóa để lựa chọn giữa chúng. Nếu bạn đang xây dựng một hệ thống mà các tác vụ cần được xử lý một lần và hoàn tất, RabbitMQ có lẽ là lựa chọn đơn giản và tốt hơn. Nếu bạn đang xây dựng một hệ thống mà nhiều dịch vụ cần phản hồi các sự kiện, hoặc nơi bạn cần phát lại dữ liệu lịch sử, Kafka là công cụ phù hợp.

Bạn có thể đạt được bất cứ điều gì với bất cứ công cụ nào. Tôi đã thấy Kafka được sử dụng cho các hàng đợi công việc đơn giản và RabbitMQ được mở rộng để xử lý các event-driven systems. Nhưng một số công cụ giúp giải quyết một số vấn đề dễ dàng hơn những công cụ khác. Hiểu được mục đích sử dụng của từng công cụ và đối chiếu nó với vấn đề thực tế của bạn sẽ giúp bạn tránh được rất nhiều sự phức tạp không cần thiết.