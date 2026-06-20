## What is event streaming?

Event streaming is the digital equivalent of the human body’s central nervous system. It is the technological foundation for the ‘always-on’ world where businesses are increasingly software-defined and automated, and where the user of software is more software.

Technically speaking, event streaming is the practice of capturing data in real-time from event sources like databases, sensors, mobile devices, cloud services, and software applications in the form of streams of events; storing these event streams durably for later retrieval; manipulating, processing, and reacting to the event streams in real-time as well as retrospectively; and routing the event streams to different destination technologies as needed. Event streaming thus ensures a continuous flow and interpretation of data so that the right information is at the right place, at the right time.

## What can I use event streaming for?

Event streaming is applied to a [wide variety of use cases](https://kafka.apache.org/powered-by) across a plethora of industries and organizations. Its many examples include:

- To process payments and financial transactions in real-time, such as in stock exchanges, banks, and insurances.
- To track and monitor cars, trucks, fleets, and shipments in real-time, such as in logistics and the automotive industry.
- To continuously capture and analyze sensor data from IoT devices or other equipment, such as in factories and wind parks.
- To collect and immediately react to customer interactions and orders, such as in retail, the hotel and travel industry, and mobile applications.
- To monitor patients in hospital care and predict changes in condition to ensure timely treatment in emergencies.
- To connect, store, and make available data produced by different divisions of a company.
- To serve as the foundation for data platforms, event-driven architectures, and microservices.

## Apache Kafka® is an event streaming platform. What does that mean?

Kafka kết hợp ba khả năng chính để bạn có thể triển khai các trường hợp sử dụng truyền phát event từ đầu đến cuối với một giải pháp duy nhất đã được kiểm chứng:

- To **publish** (write) and **subscribe to** (read) streams of events, including continuous import/export of your data from other systems.
- To **store** streams of events durably and reliably for as long as you want.
- To **process** streams of events as they occur or retrospectively.

And all this functionality is provided in a distributed, highly scalable, elastic, fault-tolerant, and secure manner. Kafka can be deployed on bare-metal hardware, virtual machines, and containers, and on-premises as well as in the cloud. You can choose between self-managing your Kafka environments and using fully managed services offered by a variety of vendors.

## How does Kafka work in a nutshell?

Kafka is a distributed system consisting of **servers** and **clients** that communicate via a high-performance [TCP network protocol](https://kafka.apache.org/protocol.html). It can be deployed on bare-metal hardware, virtual machines, and containers in on-premise as well as cloud environments.

**Servers** : Kafka is run as a cluster of one or more servers that can span multiple datacenters or cloud regions. Some of these servers form the storage layer, called the brokers. Other servers run [Kafka Connect](https://kafka.apache.org/documentation/#connect) to continuously import and export data as event streams to integrate Kafka with your existing systems such as relational databases as well as other Kafka clusters. To let you implement mission-critical use cases, a Kafka cluster is highly scalable and fault-tolerant: if any of its servers fails, the other servers will take over their work to ensure continuous operations without any data loss.

**Clients** : They allow you to write distributed applications and microservices that read, write, and process streams of events in parallel, at scale, and in a fault-tolerant manner even in the case of network problems or machine failures. Kafka ships with some such clients included, which are augmented by [dozens of clients](https://cwiki.apache.org/confluence/x/3gDVAQ) provided by the Kafka community: clients are available for Java and Scala including the higher-level [Kafka Streams](https://kafka.apache.org/documentation/streams/) library, for Go, Python, C/C++, and many other programming languages as well as REST APIs.

## Main Concepts and Terminology

Event ghi lại thực tế rằng “điều gì đó đã xảy ra” trong thế giới hoặc trong doanh nghiệp của bạn. Nó cũng được gọi là record hoặc message trong tài liệu. Khi bạn đọc hoặc ghi dữ liệu vào Kafka, bạn thực hiện điều này dưới dạng các event. Về mặt khái niệm, một event có khóa, giá trị, dấu thời gian và các tiêu đề metadata tùy chọn. Đây là một ví dụ về event:
- Event key: “Alice”
- Event value: “Made a payment of $200 to Bob”
- Event timestamp: “Jun. 25, 2020 at 2:06 p.m.”

**Producers** là các ứng dụng client publish (ghi) các event vào Kafka, còn các **consumers** là những subscribe (đọc và xử lý) các event này. Trong Kafka, các producers và consumers hoàn toàn tách biệt và không phụ thuộc vào nhau, đây là yếu tố thiết kế quan trọng để đạt được khả năng mở rộng cao mà Kafka nổi tiếng. Ví dụ, các producers không bao giờ cần phải chờ consumer. Kafka cung cấp nhiều đảm bảo khác nhau, chẳng hạn như khả năng xử lý event chính xác một lần.

Các sự kiện được tổ chức và lưu trữ bền vững trong các topic. Nói một cách đơn giản, topic tương tự như một thư mục trong hệ thống tệp, và các sự kiện là các tệp trong thư mục đó. Ví dụ về tên topic có thể là “payments”. Các topic trong Kafka luôn có nhiều producers và nhiều subscriber: một topic có thể có không, một hoặc nhiều producers ghi event vào đó, cũng như không, một hoặc nhiều consumer subscribe nhận các event này. Các event trong một topic có thể được đọc bao nhiêu lần tùy thích - không giống như các hệ thống message truyền thống, các event không bị xóa sau khi consume. Thay vào đó, bạn xác định thời gian Kafka nên giữ lại các event của mình thông qua cài đặt cấu hình cho mỗi topic, sau đó các event cũ sẽ bị loại bỏ. Hiệu suất của Kafka về cơ bản là không đổi đối với kích thước dữ liệu, vì vậy việc lưu trữ dữ liệu trong thời gian dài hoàn toàn ổn.

Các topic được phân vùng, nghĩa là một topic được trải rộng trên một số "bucket" nằm trên các Kafka brokers khác nhau. Việc phân tán dữ liệu này rất quan trọng đối với khả năng mở rộng vì nó cho phép các ứng dụng client đọc và ghi dữ liệu từ/đến nhiều brokers cùng một lúc. Khi một event mới được public lên một topic, nó thực sự được thêm vào một trong các phân vùng của topic đó. Các event có cùng khóa event (ví dụ: ID khách hàng hoặc ID phương tiện) được ghi vào cùng một phân vùng, và Kafka đảm bảo rằng bất kỳ consumer nào của một phân vùng chủ đề nhất định sẽ luôn đọc các event của phân vùng đó theo đúng thứ tự chúng được ghi.

![[kafka_induction1.png]]

Hình minh họa: Chủ đề ví dụ này có bốn phân vùng P1-P4. Hai máy khách producer khác nhau đang public các event mới lên topic một cách độc lập bằng cách ghi các event qua mạng vào các phân vùng của chủ đề. Các event có cùng khóa (được biểu thị bằng màu sắc trong hình) được ghi vào cùng một phân vùng. Lưu ý rằng cả hai producer đều có thể ghi vào cùng một phân vùng nếu thích hợp.

Để đảm bảo dữ liệu của bạn có khả năng chịu lỗi và tính sẵn sàng cao, mỗi topic có thể được sao chép, thậm chí giữa các khu vực địa lý hoặc trung tâm dữ liệu khác nhau, sao cho luôn có nhiều broker lưu trữ bản sao dữ liệu trong trường hợp xảy ra sự cố, cần bảo trì, v.v. Một thiết lập production phổ biến là hệ số sao chép là 3, tức là luôn có ba bản sao dữ liệu của bạn. Việc sao chép này được thực hiện ở cấp độ phân vùng topic.

Tài liệu giới thiệu này sẽ đủ để bạn nắm được những kiến ​​thức cơ bản. Nếu quan tâm, bạn có thể tham khảo [[Design]] trong tài liệu để hiểu rõ hơn về các khái niệm khác nhau của Kafka.

## Kafka APIs

Ngoài các công cụ dòng lệnh dành cho các tác vụ quản trị và vận hành, Kafka còn có năm API cốt lõi dành cho Java và Scala:

- Admin API dùng để quản lý và kiểm tra các topic, broker và các đối tượng Kafka khác.
- Producer API dùng để public (ghi) một luồng event vào một hoặc nhiều Kafka topic.
- Consumer API dùng để subscribe (đọc) một hoặc nhiều topic và xử lý luồng event được tạo ra cho chúng.
- Kafka Streams API để triển khai các ứng dụng xử lý luồng dữ liệu và các dịch vụ vi mô. Nó cung cấp các chức năng cấp cao hơn để xử lý các luồng event, bao gồm các phép biến đổi, các thao tác có trạng thái như tổng hợp và kết hợp, phân vùng, xử lý dựa trên thời gian event, và nhiều hơn nữa. Dữ liệu đầu vào được đọc từ một hoặc nhiều topic để tạo ra dữ liệu đầu ra cho một hoặc nhiều topic, từ đó chuyển đổi hiệu quả các luồng đầu vào thành các luồng đầu ra.
- Kafka Connect API xây dựng và vận hành các trình kết nối nhập/xuất dữ liệu có thể tái sử dụng để consume (đọc) hoặc produce (ghi) các luồng event từ và đến các hệ thống và ứng dụng bên ngoài, cho phép chúng tích hợp với Kafka. Ví dụ, một trình kết nối đến cơ sở dữ liệu quan hệ như PostgreSQL có thể ghi lại mọi thay đổi đối với một tập hợp các bảng. Tuy nhiên, trên thực tế, bạn thường không cần phải tự triển khai các trình kết nối của riêng mình vì cộng đồng Kafka đã cung cấp hàng trăm trình kết nối sẵn sàng sử dụng.

## Where to go from here

- To get hands-on experience with Kafka, follow the [Quickstart](https://kafka.apache.org/quickstart).
- To understand Kafka in more detail, read the [Documentation](https://kafka.apache.org/documentation/). You also have your choice of [Kafka books and academic papers](https://kafka.apache.org/books-and-papers).
- Browse through the [Use Cases](https://kafka.apache.org/powered-by) to learn how other users in our world-wide community are getting value out of Kafka.
- Join a [local Kafka meetup group](https://kafka.apache.org/events) and [watch talks from Kafka Summit](https://kafka-summit.org/past-events/), the main conference of the Kafka community.
  
Nguồn: https://kafka.apache.org/43/getting-started/introduction/
- [[Design]]