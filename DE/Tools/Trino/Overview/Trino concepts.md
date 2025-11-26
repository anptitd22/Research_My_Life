- [[#Overview|Overview]]
- [[#Architecture|Architecture]]
	- [[#Architecture#Cluster|Cluster]]
	- [[#Architecture#Node|Node]]
	- [[#Architecture#Coordinator|Coordinator]]
	- [[#Architecture#Worker|Worker]]
- [[#Client|Client]]
- [[#Plugin (skip)|Plugin (skip)]]
- [[#Data source|Data source]]
	- [[#Data source#Connector|Connector]]
	- [[#Data source#Catalog|Catalog]]
	- [[#Data source#Schema (skip)|Schema (skip)]]
	- [[#Data source#Table (skip)|Table (skip)]]
- [[#Query execution model|Query execution model]]
	- [[#Query execution model#Statement|Statement]]
	- [[#Query execution model#Query|Query]]
	- [[#Query execution model#Stage|Stage]]
	- [[#Query execution model#Task|Task]]
	- [[#Query execution model#Split|Split]]
	- [[#Query execution model#Driver|Driver]]
	- [[#Query execution model#Operator|Operator]]
	- [[#Query execution model#Exchange|Exchange]]

## Overview

Để hiểu về Trino, trước tiên bạn phải hiểu các thuật ngữ và khái niệm được sử dụng trong tài liệu Trino.

Mặc dù các câu lệnh và truy vấn rất dễ hiểu, nhưng với tư cách là người dùng cuối, bạn nên quen thuộc với các khái niệm như giai đoạn (stage) và phân tách (split) để tận dụng tối đa Trino nhằm thực hiện các truy vấn hiệu quả. Là quản trị viên Trino hoặc người đóng góp cho Trino, bạn nên hiểu cách các khái niệm về giai đoạn (stage) của Trino tương ứng với các tác vụ (task) và cách các tác vụ (task) chứa một tập hợp các trình điều khiển xử lý dữ liệu (driver). 

Phần này cung cấp định nghĩa chắc chắn về các khái niệm cốt lõi được tham chiếu trong toàn bộ Trino và các phần này được sắp xếp từ tổng quát nhất đến cụ thể nhất.

## Architecture

Trino là một công cụ truy vấn phân tán xử lý dữ liệu song song trên nhiều máy chủ. Có hai loại máy chủ Trino: máy chủ điều phối (coordinator) và máy chủ công tác (worker). Các phần sau đây mô tả các máy chủ này và các thành phần khác trong kiến ​​trúc của Trino.

![alt text](/images/trino_architecture.png)

### Cluster

Một cụm Trino bao gồm một số node Trino - một coordinator và không hoặc nhiều workers. Người dùng kết nối với coordinator bằng công cụ truy vấn SQL của họ. Coordinator cộng tác với các workers. Coordinator và các workers truy cập các nguồn dữ liệu được kết nối. Quyền truy cập này được cấu hình trong catalogs. 

Việc xử lý mỗi truy vấn là một hoạt động có trạng thái. Khối lượng công việc được điều phối bởi coordinator và phân bổ song song cho tất cả các workers trong cụm. Mỗi node chạy Trino trong một phiên bản JVM, và việc xử lý được parallelized hơn nữa bằng các luồng.

### Node

Bất kỳ máy chủ Trino nào trong một cụm Trino cụ thể đều được coi là một node của cụm. Về mặt kỹ thuật, điều này đề cập đến tiến trình Java chạy chương trình Trino, nhưng node thường được sử dụng để chỉ máy tính đang chạy tiến trình do khuyến nghị chỉ nên chạy một tiến trình Trino trên mỗi máy tính.

### Coordinator

Trino coordinator là máy chủ chịu trách nhiệm phân tích cú pháp các câu lệnh, lập kế hoạch truy vấn và quản lý các node Trino worker. Đây là "bộ não" của một cài đặt Trino và cũng là node mà máy khách kết nối để gửi các câu lệnh thực thi. Mỗi cài đặt Trino phải có một Trino coordinator cùng với một hoặc nhiều Trino worker. Cho mục đích phát triển hoặc thử nghiệm, một phiên bản Trino duy nhất có thể được cấu hình để thực hiện cả hai vai trò. 

Coordinator theo dõi hoạt động của từng worker và điều phối việc thực hiện truy vấn. Coordinator tạo ra một mô hình logic của truy vấn bao gồm một loạt các giai đoạn (stages), sau đó được chuyển đổi thành một loạt các tác vụ (tasks) được kết nối chạy trên một cụm Trino workers. 

Coordinator giao tiếp với workers và clients bằng cách sử dụng REST API.

### Worker

Trino worker là một máy chủ trong cài đặt Trino, chịu trách nhiệm thực thi các tác vụ và xử lý dữ liệu. Các node worker lấy dữ liệu từ các kết nối và trao đổi dữ liệu trung gian với nhau. Coordinator chịu trách nhiệm lấy kết quả từ các worker và trả về kết quả cuối cùng cho clients. 

Khi một tiến trình Trino worker khởi động, nó sẽ tự quảng cáo tới máy chủ khám phá trong coordinator, giúp Trino coordinator có thể thực hiện tác vụ. 

Worker giao tiếp với những worker khác và Trino coordinator bằng cách sử dụng REST API

## Client

Máy khách cho phép bạn kết nối với Trino, gửi truy vấn SQL và nhận kết quả. Máy khách có thể truy cập tất cả các nguồn dữ liệu đã cấu hình bằng [catalogs](https://trino.io/docs/current/overview/concepts.html#trino-concept-catalog). Máy khách là các ứng dụng máy khách hoặc trình điều khiển máy khách và thư viện đầy đủ tính năng cho phép bạn kết nối với bất kỳ ứng dụng nào hỗ trợ trình điều khiển đó, hoặc thậm chí là ứng dụng hoặc tập lệnh tùy chỉnh của riêng bạn.

Các ứng dụng của khách hàng bao gồm các công cụ dòng lệnh, ứng dụng máy tính để bàn, ứng dụng dựa trên web và các giải pháp phần mềm dưới dạng dịch vụ với các tính năng như tạo truy vấn SQL tương tác bằng trình soạn thảo hoặc giao diện người dùng phong phú để tạo truy vấn đồ họa, chạy truy vấn và hiển thị kết quả, trực quan hóa bằng biểu đồ và đồ thị, báo cáo và tạo bảng điều khiển.

Ứng dụng khách hỗ trợ các ngôn ngữ truy vấn hoặc thành phần giao diện người dùng khác để xây dựng truy vấn phải dịch từng yêu cầu sang SQL như Trino hỗ trợ.

Bạn có thể tìm thấy thêm thông tin chi tiết trong [Trino client documentation](https://trino.io/docs/current/client.html).

## Plugin (skip)

## Data source 

### Connector

Trino có thể truy vấn nhiều loại dữ liệu bằng cách sử dụng một giao diện chung gọi là SPI (Giao diện Nhà cung cấp Dịch vụ), cho phép công cụ cốt lõi xử lý các tương tác với từng nguồn dữ liệu như nhau. Mỗi đầu nối sau đó phải triển khai SPI, bao gồm việc hiển thị metadata, số liệu thống kê, vị trí dữ liệu và thiết lập một hoặc nhiều kết nối với nguồn dữ liệu cơ bản.

![alt text](/images/trino_SPI.png)

### Catalog

Trino catalogs là tập hợp các thuộc tính cấu hình được sử dụng để truy cập một nguồn dữ liệu cụ thể, bao gồm trình kết nối cần thiết và bất kỳ thông tin chi tiết nào khác như thông tin đăng nhập và URL. Catalog được định nghĩa trong các tệp thuộc tính được lưu trữ trong thư mục cấu hình Trino. Tên của tệp thuộc tính xác định tên của catalog. Ví dụ: tệp thuộc tính etc/example.properties trả về tên catalog example.

Bạn có thể cấu hình và sử dụng nhiều catalogs, với các connector khác nhau hoặc giống hệt nhau, để truy cập các nguồn dữ liệu khác nhau. Ví dụ: nếu bạn có hai data lake, bạn có thể cấu hình hai catalog trong một cụm Trino duy nhất, cả hai đều sử dụng Hive connector, cho phép bạn truy vấn dữ liệu từ cả hai cụm, ngay cả trong cùng một truy vấn SQL. Bạn cũng có thể sử dụng Hive connector cho một catalog để truy cập hồ dữ liệu và sử dụng Iceberg connector cho một catalog khác để truy cập kho dữ liệu (lakehouse). Hoặc, bạn có thể cấu hình các catalog khác nhau để truy cập các cơ sở dữ liệu PostgreSQL khác nhau. Việc kết hợp các catalog khác nhau được xác định bởi nhu cầu của bạn chỉ truy cập các nguồn dữ liệu khác nhau.

Một catalog chứa một hoặc nhiều schema, mà các schema này lại chứa các đối tượng như table, views hoặc materialized views. Khi đề cập đến các đối tượng như bảng trong Trino, tên đầy đủ luôn bắt nguồn từ một catalog. Ví dụ: tên bảng đầy đủ của example.test_data.test tham chiếu đến bảng thử nghiệm trong test_data schema trong example catalog.

### Schema (skip)

### Table (skip)

## Query execution model

![alt text](/images/trino_query_plan.png)

![alt text](/images/trino_query_plan2.png)

![alt text](/images/trino_stage_to_task.png)

![alt text](/images/trino_task.png)

![alt text](/images/trino_splits.png)

Trino thực thi các câu lệnh SQL và chuyển các câu lệnh này thành các truy vấn, được thực thi trên một cụm phân tán gồm các coordinator và worker.

### Statement

Trino thực thi các câu lệnh SQL tương thích với ANSI. Khi tài liệu của Trino đề cập đến một câu lệnh, tức là đề cập đến các câu lệnh được định nghĩa trong tiêu chuẩn SQL ANSI, bao gồm các mệnh đề, biểu thức và vị từ. 

Một số độc giả có thể tò mò tại sao phần này lại liệt kê các khái niệm riêng biệt cho câu lệnh và truy vấn. Điều này là cần thiết vì trong Trino, câu lệnh chỉ đơn giản là biểu diễn văn bản của một câu lệnh được viết bằng SQL. Khi một câu lệnh được thực thi, Trino tạo một truy vấn cùng với một kế hoạch truy vấn, sau đó được phân phối cho một loạt các trình xử lý Trino.

### Query

Khi Trino phân tích cú pháp một câu lệnh, nó sẽ chuyển đổi câu lệnh đó thành một truy vấn và tạo ra một kế hoạch truy vấn phân tán, sau đó được hiện thực hóa thành một chuỗi các giai đoạn được kết nối với nhau chạy trên các trình xử lý của Trino. Khi bạn truy xuất thông tin về một truy vấn trong Trino, bạn sẽ nhận được snapshot của mọi thành phần tham gia vào việc tạo ra tập kết quả để phản hồi câu lệnh. 

Sự khác biệt giữa câu lệnh (statement) và truy vấn (query) rất đơn giản. Câu lệnh có thể được hiểu là văn bản SQL được truyền đến Trino, trong khi truy vấn đề cập đến cấu hình và các thành phần được khởi tạo để thực thi câu lệnh đó. Truy vấn bao gồm các giai đoạn, tác vụ, phân tách, kết nối và các thành phần và nguồn dữ liệu khác hoạt động phối hợp để tạo ra kết quả.

### Stage

Khi Trino thực thi một truy vấn, nó sẽ chia quá trình thực thi thành một hệ thống phân cấp các giai đoạn. Ví dụ: nếu Trino cần tổng hợp dữ liệu từ một tỷ hàng được lưu trữ trong Hive, nó sẽ tạo một giai đoạn gốc để tổng hợp đầu ra của nhiều giai đoạn khác, tất cả đều được thiết kế để triển khai các phần khác nhau của một kế hoạch truy vấn phân tán. 

Hệ thống phân cấp các giai đoạn tạo nên một truy vấn giống như một cây. Mỗi truy vấn đều có một giai đoạn gốc, chịu trách nhiệm tổng hợp đầu ra từ các giai đoạn khác. Các giai đoạn là những gì coordinator sử dụng để mô hình hóa một kế hoạch truy vấn phân tán, nhưng bản thân các giai đoạn không chạy trên các Trino workers.

### Task

Như đã đề cập ở phần trước, các giai đoạn mô hình hóa một phần cụ thể của kế hoạch truy vấn phân tán, nhưng bản thân các giai đoạn (stages) không thực thi trên các Trino worker. Để hiểu cách một giai đoạn được thực thi, bạn cần hiểu rằng một giai đoạn được triển khai như một chuỗi các tác vụ (tasks) được phân bổ trên một mạng lưới các Trino worker.  

Task là "work horse" trong kiến ​​trúc Trino khi một kế hoạch truy vấn phân tán được phân tách thành một chuỗi các giai đoạn, sau đó được chuyển đổi thành các nhiệm vụ (task), rồi sau đó thực hiện hoặc xử lý các phân tách. Một task Trino có đầu vào và đầu ra, và cũng giống như một giai đoạn có thể được thực thi song song bởi một chuỗi các nhiệm vụ (tasks), một task được thực thi song song với một chuỗi các trình điều khiển (drivers).

### Split

Các tác vụ (tasks) hoạt động theo chế độ split, là các phần của một tập dữ liệu lớn hơn. Các giai đoạn ở cấp thấp nhất của một kế hoạch truy vấn phân tán sẽ truy xuất dữ liệu thông qua các phần split từ các trình kết nối, và các giai đoạn trung gian ở cấp cao hơn của một kế hoạch truy vấn phân tán sẽ truy xuất dữ liệu từ các giai đoạn khác. 

Khi Trino lập lịch truy vấn, coordinator sẽ truy vấn một trình kết nối để lấy danh sách tất cả các split khả dụng cho một bảng. Coordinator sẽ theo dõi máy nào đang chạy tác vụ nào và split nào đang được xử lý bởi tác vụ nào.

Bonus (tóm gọn) : Split trong Trino đại diện cho một đơn vị dữ liệu nhỏ nhất mà một worker có thể xử lý độc lập

- Coordinator chia nhỏ truy vấn thành các stage, mỗi stage được chia tiếp thành nhiều tasks.
    
- Mỗi task được phân cho một worker và xử lý một hoặc nhiều splits.
    
- Mỗi split chứa metadata

### Driver

Mỗi tác vụ chứa một hoặc nhiều trình điều khiển song song. Các trình điều khiển này tác động lên dữ liệu và kết hợp các toán tử để tạo ra đầu ra, sau đó được tổng hợp bởi một tác vụ và chuyển đến một tác vụ khác ở một giai đoạn khác. Trình điều khiển là một chuỗi các thể hiện toán tử, hoặc bạn có thể hình dung trình điều khiển như một tập hợp các toán tử vật lý trong bộ nhớ. Đây là mức độ song song thấp nhất trong kiến ​​trúc Trino. Một trình điều khiển có một đầu vào và một đầu ra.

### Operator

Toán tử sử dụng, biến đổi và tạo ra dữ liệu. Ví dụ: quét bảng sẽ lấy dữ liệu từ một kết nối và tạo ra dữ liệu có thể được sử dụng bởi các toán tử khác, còn toán tử lọc sẽ sử dụng dữ liệu và tạo ra một tập hợp con bằng cách áp dụng một vị từ lên dữ liệu đầu vào.  

### Exchange

Trao đổi dữ liệu giữa các node Trino cho các giai đoạn khác nhau của truy vấn. Các tác vụ tạo dữ liệu vào bộ đệm đầu ra và sử dụng dữ liệu từ các tác vụ khác bằng exchange client.