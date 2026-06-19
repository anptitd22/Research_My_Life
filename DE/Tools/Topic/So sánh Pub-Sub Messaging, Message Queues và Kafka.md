---
title: "So sánh Pub-Sub Messaging, Message Queues và Kafka"
source: "https://nghiatdh.medium.com/so-s%C3%A1nh-pub-sub-messaging-message-queues-v%C3%A0-kafka-e2206a5b7f0a"
author:
  - "[[Nghĩa Tạ Đăng Hiếu]]"
published: 2019-08-22
created: 2026-06-19
description: "So sánh Pub-Sub Messaging, Message Queues và Kafka Với sự bùng nổ về cả số lượng và sự đa dạng của các ứng dụng với các thao tác thu thập hoặc xử lý dữ …"
tags:
  - "clippings"
---
[Sitemap](https://nghiatdh.medium.com/sitemap/sitemap.xml)

[Mastodon](https://me.dm/@nghiatdh)

Với sự bùng nổ về cả số lượng và sự đa dạng của các ứng dụng với các thao tác thu thập hoặc xử lý dữ liệu, việc triển khai các connections truyền dữ liệu từ nơi nó được tạo ra đến nhiều nơi khác nhau để xử lý là một phần cơ bản của phát triển ứng dụng. `Messaging` là một công nghệ quan trọng để kết nối dữ liệu trong môi trường này.

`Messaging` là công việc truyền các bản ghi dữ liệu - records - từ hệ thống này sang hệ thống khác thông qua một hệ thống trung gian. Ngược lại với các kết nối trực tiếp, nơi người gửi biết được người nhận và kết nối trực tiếp đến từng người nhận, các giải pháp `messaging` sẽ tách rời việc gửi dữ liệu khỏi việc xử lý dữ liệu. Người gửi không cần biết người nhận nào sẽ thấy dữ liệu đó hoặc khi nào họ sẽ thấy dữ liệu đó.

Các giải pháp `messaging` đơn giản hóa việc phát triển ứng dụng bằng cách cung cấp cho các architects và các dev một thành phần hệ thống đã được chuẩn hóa, có thể tái sử dụng để xử lý tốt luồng dữ liệu để dev có thể tập trung vào core logic trong các ứng dụng của họ. Ngoài định tuyến dữ liệu, `messaging systems` cũng có thể cung cấp các tính năng quan trọng như khả năng chịu lỗi, logging và xử lý phân tán để cải thiện khả năng quản lý và độ tin cậy.

`Messaging` là một thuật ngữ bao gồm một số mô hình khác nhau dựa trên cách dữ liệu được chuyển từ người gửi sang người nhận. Hai mô hình message chính là `message queue` và `publish-subcribe messaging` (thường được gọi là `pub-sub messaging`).

## Message Queue (Nhiều chỗ sai)

`Message Queue` được thiết kế cho các tình huống như `tasklist` hoặc `workqueue`. `Message Queue` nhận được message đến và đảm bảo rằng mỗi message sẽ được gửi cho một topic hay channel và xử lý bởi chính xác một consumer.

Ví dụ: kịch bản một `Message Queue` được sử dụng trong việc phát hành phiếu lương, một điều rất quan trọng là mọi phiếu lương đều phải được phát hành một lần và chỉ một lần duy nhất, trong khi việc xử lý ở máy kiểm tra nào không quan trọng.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/0*V7ySKdgdHSBdzlah.png)

`Message Queue` có thể nâng cao khả năng xử lý message lên cao bằng cách thêm nhiều consumers cho mỗi topic, **nhưng chỉ duy nhất một consumer sẽ nhận được mỗi message ở topic này**. Consumer nào nhận được message nào được xác định bằng cách implementation `Message Queue`. Để đảm bảo rằng một message chỉ được xử lý bởi một consumer, mỗi message sẽ bị xóa khỏi queue sau khi nó đã được một consumer nhận và xử lý (tức là một khi consumer thừa nhận đã sử dụng message đến từ message system).

`Message Queue` hỗ trợ các use cases mà đảm bảo việc mỗi message được xử lý chính xác một và chỉ một lần duy nhất - exactly one, nhưng không cần thiết phải xử lý message theo thứ tự. Trong trường hợp lỗi từ network hoặc từ consumer, `Message Queue` sẽ thử gửi lại message (không nhất thiết phải cho cùng một consumer) và kết quả là message có thể được xử lý không theo thứ tự.

Có thể tóm tắt về Message Queue:

- Exactly One
- Scale processing ( sẽ không đảm bảo thứ tự )
- Còn nếu đảm bảo thứ tự ( sẽ không xử lý song song -> không scale processing )

Tham khảo: [[Pub-Sub System vs Queues]]

## Publish-Subcribe Messaging (Nhiều chỗ sai)

Giống như `Message Queue`, `Publish-Subcribe Messaging` truyền message từ nơi gửi đến nơi sử dụng. Tuy nhiên, trái ngược với `Message Queue`, `Publish-Subcribe Messaging` cho phép nhiều consumer nhận **từng message** trong một topic. Hơn nữa, message pub-sub đảm bảo rằng mỗi consumer nhận được message trong ==một== topic theo thứ tự chính xác mà message system nhận được.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/0*FhLDXCX8PR0-LUl6.png)

Các hệ thống `Publish-Subcribe Messaging` hỗ trợ các trường hợp yêu cầu nhiều consumers nhận từng message và các message đó được nhận theo thứ tự của mỗi consumer.

Ví dụ, một dịch vụ stock sticker có thể được sử dụng bởi một số lượng lớn người dùng và ứng dụng, tất cả những người dùng muốn nhận tất cả message về topic họ chọn. Điều quan trọng đối với những người dùng này là họ nhận được tin nhắn theo thứ tự mà việc nhìn thấy giá cao — giá thấp cho một cổ phiếu rất khác so với việc nhìn thấy giá thấp — giá cao.

Các trường hợp sử dụng pub-sub thường được liên kết với các ứng dụng trạng thái. Các ứng dụng có trạng thái quan tâm đến thứ tự của các tin nhắn nhận được vì thứ tự của các tin nhắn xác định trạng thái của ứng dụng và do đó sẽ ảnh hưởng đến tính chính xác của bất kỳ logic xử lý nào mà ứng dụng có dùng đến.

Có thể tóm tắt về Pub-sub messaging:

- Multi-Subcriber
- Đảm bảo thứ tự

Tham khảo: [[Pub-Sub System vs Queues]]

## Messaging Products

Cả hai mô hình `Message Queue` và `Pub-sub messaging` đều cần thiết trong các ứng dụng hiện đại. Có một số công nghệ hỗ trợ `Message Queue`, `Pub-sub messaging` hoặc cả hai. Các công nghệ như Apache ActiveMQ, Amazon SQS, IBM Websphere MQ, RabbitMQ được phát triển chủ yếu cho các trường hợp sử dụng message queue, trong khi các hệ thống như Amazon SNS, Apache Kafka, RocketMQ và Google Cloud Pub / Sub được thiết kế chủ yếu cho các trường hợp sử dụng pub-sub. Ngoài ra còn có các giải pháp như Apache Pulsar cung cấp hỗ trợ cho cả message queue và pub-sub.

***Hiện nay các nền tảng công nghệ về messaging vừa nêu trên như RabbitMQ, Apache Kafka hay RocketMQ đều bổ sung thêm các tính năng có thể phục vụ của cả hai loại mô hình, tuy nhiên, về bản chất thì mỗi công nghệ chỉ có thể lựa chọn 1 trong 2 mô hình để triển khai. Do đó, dù đảm bảo đầy đủ các chức năng nhưng sẽ luôn có những điểm riêng biệt của mỗi nền tảng công nghệ vừa nêu.***

## Ưu và nhược điểm

Mỗi một mẫu messaging trên đều có ưu điểm và nhược điểm:

- Điểm mạnh của `Message Queue` là nó cho phép phân chia việc xử lý dữ liệu trên nhiều consumers. Từ đó, có thể scale khả năng xử lý dữ liệu.
- Điểm yếu của `Message Queue` là nó không cho phép multi-subcriber - nghĩa là chỉ có một subcriber duy nhất được xử lý một message sau đó message đó sẽ biến mất.
- Trong khi đó, điểm mạnh của `Pub-sub messaging` là cho phép bạn broadcast dữ liệu đến các nơi xử lý dữ liệu.
- Nhưng ngược lại với `Message Queue`, điểm yếu của `Pub-sub messaging` là không có cách nào để scale việc xử lý dữ liệu bởi vì mỗi một tin nhắn đều được truyền đến mọi subcriber.

Từ đó ta thấy rằng sử dụng `Message Queue` hay `Pub-sub messaging` còn tùy vào mục đích sử dụng để tận dụng ưu điểm cũng như hạn chế nhược điểm của mỗi message system.

Tuy vậy, Kafka là một giải pháp mới cho phép bạn có được các lợi thế của mỗi message system truyền thống.

## Kafka

Apache Kafka được định nghĩa như là một `distributed streaming platform`. Vậy thì nó khác gì so với các với các `messaging system` truyền thống.

Trong Kafka, `consumer group` là một nhóm các `consumer`. Cũng như `message queue` thì `consumer group` cho phép bạn phân chia việc xử lý dữ liệu trên nhiều consumer trong một group. Và nó cũng cho phép bạn broadcast các messages đến nhiều groups như `pub-sub messaging`.

Điểm mạnh của mô hình Kafka là mỗi topic đều có cả 2 thuộc tính — vừa có thể scale processing vừa multi-subcriber — thay vì phải chọn 1 trong 2.

Kafka cũng đảm bảo thứ tự message hơn so với `messaging system` truyền thống.

Một `message queue` truyền thống giữ lại các records theo thứ tự trên server, và nếu có nhiều consumer sử dụng chúng từ queue thì server sẽ phân phối các records theo thứ tự đã lưu trữ. Tuy nhiên, mặc dù server phân phối các records theo thứ tự nhưng các records được vận chuyển bất đồng bộ, thế nên chúng có thể không theo thứ tự trên những consumer khác nhau. Điều này có nghĩa là thứ tự các records sẽ bị mất khi có sự tiêu thụ song song. Các `messaging system` truyền thống thường giải quyết vấn đề này bằng cách "exclusive consumer" chỉ cho phép một quá trình consume ở hàng đợi, nhưng tất nhiên điều này có nghĩa là không có sự xử lý dữ liệu song song.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/0*OcS7sCaoIAYFeUwq.png)

Mỗi partition (P0 - P3, P1 - P2) sẽ được tiêu thụ bởi duy nhất một consumer (C1 - C2, C3 - C6 ) trong mỗi group (A, B).

Kafka làm được điều đó tốt hơn. Với các partitions trong các topics, Kafka có thể vừa đảm bảo thứ tự và vừa cân bằng tải trên các bể consumer. Điều này thực hiện được bằng cách gán các partitions trong topic đến các consumers trong consumer group để `mỗi partitions chỉ được consume bởi chính xác một consumer trong group`. Bằng cách này, chúng ta có thể đảm bảo rằng `consumer` chỉ đọc dữ liệu của partition đó và xử lý dữ liệu theo thứ tự. Vì sẽ có nhiều partitions, điều này vẫn đảm bảo cân bằng tải trên trường hợp nhiều consumers.

## Tham khảo

- [https://streaml.io/resources/tutorials/concepts/messaging-and-queuing](https://streaml.io/resources/tutorials/concepts/messaging-and-queuing)
- [https://kafka.apache.org/intro](https://kafka.apache.org/intro)

---
## 1. Bản chất: Phân phối tin nhắn (Pub/Sub) vs. Chia sẻ công việc (Message Queue)

### Mô hình Message Queue (Ví dụ: RabbitMQ) → "Competing Consumers"

Mô hình này hoạt động theo nguyên lý **chia việc**. Một tin nhắn gửi vào Queue chỉ được giao cho **duy nhất một Consumer** xử lý.

- **Cách nó scale:** Nếu lượng tin nhắn đổ về quá nhiều, bạn chỉ cần bật thêm Consumer B, Consumer C lên chung sức với Consumer A. Tin nhắn sẽ được chia đều (Load Balancing) cho các Consumer. Tốc độ xử lý tổng thể tăng lên gấp 3 lần.
    
### Mô hình Pub/Sub truyền thống (Ví dụ: RabbitMQ Fanout Exchange, Redis Pub/Sub)

Mô hình này hoạt động theo nguyên lý **phát thanh (Broadcasting)**. Mỗi Subscriber (người đăng ký) đại diện cho một chức năng/hệ thống hoàn toàn khác nhau (ví dụ: Hệ thống Email, Hệ thống Thống kê). Khi có tin nhắn mới, hệ thống sẽ sao chép và gửi một bản sao đến **tất cả mọi Subscriber**.

- **Vấn đề khi scale:** Giả sử "Hệ thống Email" bị quá tải vì nhận quá nhiều tin nhắn. Theo phản xạ tự nhiên, bạn bật thêm một bản sao nữa của Hệ thống Email (gọi là Email-Worker-2) với hy vọng nó gánh bớt việc.
    
- **Hậu quả:** Vì là Pub/Sub, tin nhắn tiếp theo lại được **gửi đến cả Email-Worker-1 VÀ Email-Worker-2**. Kết quả là khách hàng nhận được 2 email trùng lặp, hệ thống bị nhân đôi áp lực xử lý chứ không hề giảm đi. Bạn không thể scale bằng cách tăng số lượng worker.
    

## 2. Các hệ thống hiện đại giải quyết "điểm yếu" này như thế nào?

Vì điểm yếu chí mạng này, các công nghệ hiện đại ngày nay không bao giờ dùng mô hình Pub/Sub thuần túy nữa. Họ đã kết hợp **Pub/Sub với Message Queue** để tạo ra các cơ chế lai (Hybrid) cực kỳ thông minh:

### Giải pháp của RabbitMQ: Kết hợp Fanout Exchange + Multiple Queues

RabbitMQ cho phép bạn cấu hình mô hình Pub/Sub nhưng ở đầu nhận, mỗi Subscriber sẽ sở hữu riêng một Queue.

- Nếu bạn muốn scale "Hệ thống Email", bạn chỉ cần gắn **nhiều Worker cùng đọc chung vào duy nhất 1 Queue** của Hệ thống Email đó.
    
- Lúc này, mô hình Pub/Sub vẫn đảm bảo "Hệ thống Email" và "Hệ thống Thống kê" đều nhận được tin nhắn, nhưng bên trong Hệ thống Email, các worker có thể scale song song để chia sẻ tải mà không bị trùng lặp dữ liệu.
    

### Giải pháp của Apache Kafka: Khái niệm "Consumer Group"

Kafka nâng cấp cơ chế này lên một tầm cao mới bằng cách định nghĩa **Consumer Group**.

- Ví dụ: Bạn có `Group-Email` (gồm 3 máy chủ) và `Group-Analytics` (gồm 2 máy chủ) cùng subcribe vào một Topic.
    
- Kafka sẽ phát tin nhắn đến **cả 2 Group** (Tính chất Pub/Sub).
    
- Nhưng trong nội bộ `Group-Email`, Kafka sẽ **chia nhỏ các Partition** ra cho 3 máy chủ xử lý, đảm bảo một tin nhắn chỉ do 1 máy chủ trong group nhận (Tính chất Message Queue).