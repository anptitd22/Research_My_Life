# Kiến Trúc và Hệ Sinh Thái Apache Hive Metastore: Nền Tảng Quản Lý Siêu Dữ Liệu Trong Các Hệ Thống Hồ Dữ Liệu Hiện Đại

Sự ra đời của Apache Hive tại Facebook vào khoảng năm 2008 đã đánh dấu một bước ngoặt quan trọng trong lịch sử xử lý dữ liệu lớn. Vào thời điểm đó, việc truy vấn các tập dữ liệu khổng lồ trên Hệ thống tệp phân tán Hadoop (HDFS) đòi hỏi các kỹ sư phải viết mã Java MapReduce phức tạp, một rào cản lớn đối với các nhà phân tích dữ liệu vốn quen thuộc với SQL. Để giải quyết vấn đề này, Hive đã được thiết kế như một lớp trừu tượng kiểu kho dữ liệu, cho phép thực hiện các truy vấn SQL (được gọi là HiveQL) trên dữ liệu phân tán. Trái tim của hệ thống này, và cũng là thành phần quan trọng nhất đối với khả năng tương tác của toàn bộ hệ sinh thái dữ liệu lớn, chính là Hive Metastore (HMS).  

Hive Metastore phục vụ như một kho lưu trữ trung tâm cho tất cả các thông tin về cấu trúc của các bảng và phân vùng trong hồ dữ liệu. Trong các hệ thống cơ sở dữ liệu truyền thống, siêu dữ liệu (metadata) thường được tích hợp chặt chẽ với công cụ lưu trữ và xử lý. Tuy nhiên, trong mô hình của Hadoop và các kiến trúc hồ dữ liệu hiện đại, siêu dữ liệu được tách rời hoàn toàn khỏi dữ liệu thực tế. Sự tách biệt này cho phép dữ liệu tồn tại dưới dạng các tệp phẳng (như Parquet, ORC, CSV) trên các hệ thống lưu trữ như HDFS hoặc Amazon S3, trong khi Hive Metastore cung cấp "bản đồ" để các công cụ tính toán có thể hiểu và truy vấn các tệp đó như thể chúng là các bảng quan hệ.  

## Phân Tích Kiến Trúc Kỹ Thuật và Cơ Chế Hoạt Động

Kiến trúc của Hive Metastore được thiết kế dựa trên nguyên tắc tách biệt giữa giao diện dịch vụ và lưu trữ dữ liệu thực tế. Về cơ bản, HMS bao gồm hai thành phần nền tảng: Dịch vụ Metastore (Metastore Service) và Cơ sở dữ liệu phụ trợ (Backend Database). Dịch vụ Metastore đóng vai trò là lớp API, nhận các yêu cầu từ các máy khách (clients) và thực hiện các thao tác đọc/ghi vào cơ sở dữ liệu quan hệ phía sau.  

![alt][images/Tools/HMS/HMS_workflow.png]

### Giao Diện Dịch Vụ và Giao Thức Thrift

Để đảm bảo tính trung lập và khả năng tương thích cao với nhiều ngôn ngữ lập trình cũng như các công cụ khác nhau, Hive Metastore sử dụng giao thức Apache Thrift. Thrift cung cấp một khung làm việc cho phép định nghĩa các kiểu dữ liệu và giao diện dịch vụ trong một tệp định nghĩa duy nhất, sau đó tạo ra mã nguồn cho nhiều ngôn ngữ như Java, C++, Python, và PHP.  

Các máy khách như HiveServer2, Apache Spark, Presto hoặc Trino không kết nối trực tiếp với cơ sở dữ liệu quan hệ của Metastore. Thay vào đó, chúng giao tiếp thông qua giao diện Thrift này. Điều này mang lại một lớp bảo mật quan trọng: người quản trị chỉ cần cấp quyền truy cập cơ sở dữ liệu cho duy nhất dịch vụ HMS, thay vì phải chia sẻ thông tin đăng nhập JDBC với mọi người dùng hoặc mọi nút trong cụm máy tính. Ngoài ra, việc sử dụng Thrift cho phép HMS hoạt động như một dịch vụ phi trạng thái (stateless), có nghĩa là nhiều thực thể HMS có thể được triển khai sau một bộ cân bằng tải để đảm bảo tính sẵn sàng cao (High Availability).  

### Lớp Ánh Xạ Đối Tượng - Quan Hệ (ORM) và DataNucleus

Bên trong dịch vụ HMS, việc tương tác với cơ sở dữ liệu quan hệ không được thực hiện thông qua các câu lệnh SQL thuần túy mà thông qua một giải pháp Object-Relational Mapping (ORM) có tên là DataNucleus (trước đây là JPOX). DataNucleus cho phép các đối tượng siêu dữ liệu của Hive (như Table, Partition, Database) được duy trì trong bất kỳ cơ sở dữ liệu quan hệ nào được nó hỗ trợ.  

Mặc dù giải pháp ORM mang lại tính linh hoạt cao trong việc hỗ trợ nhiều loại cơ sở dữ liệu khác nhau, nó cũng tạo ra một số rào cản về hiệu suất. Các truy vấn được tạo ra bởi DataNucleus đôi khi không được tối ưu hóa cho các thao tác phức tạp như liệt kê hàng triệu phân vùng. Điều này dẫn đến sự ra đời của tính năng "Direct SQL", cho phép HMS bỏ qua lớp ORM và thực hiện các truy vấn SQL viết tay trực tiếp đối với cơ sở dữ liệu quan hệ cho các tác vụ quan trọng về hiệu suất.  

| Thành phần            | Chức năng chính                                     | Công nghệ sử dụng                     |
| --------------------- | --------------------------------------------------- | ------------------------------------- |
| **Metastore Service** | Cung cấp API, xử lý logic nghiệp vụ siêu dữ liệu    | Apache Thrift, Java                   |
| **ORM Layer**         | Ánh xạ đối tượng Java vào các bảng quan hệ          | DataNucleus (JPOX)                    |
| **Backend Database**  | Lưu trữ vĩnh viễn các bảng siêu dữ liệu             | MySQL, PostgreSQL, Oracle, SQL Server |
| **Client API**        | Cho phép các công cụ tính toán yêu cầu siêu dữ liệu | HiveMetaStoreClient                   |

## Mô Hình Dữ Liệu và Phân Cấp Đối Tượng

Siêu dữ liệu trong Hive Metastore được tổ chức theo một cấu trúc phân cấp chặt chẽ, giúp phản ánh cách dữ liệu được tổ chức trên hệ thống tệp và cách nó được trừu tượng hóa trong ngôn ngữ SQL.  

### Các Thực Thể Siêu Dữ Liệu Nền Tảng

Ở cấp độ cao nhất là Catalog (Danh mục), một thực thể mới được giới thiệu trong các phiên bản Hive gần đây (3.0 trở đi) để hỗ trợ triển khai đa catalog. Tuy nhiên, trong hầu hết các cài đặt truyền thống, thực thể chính đầu tiên mà người dùng tương tác là Database (Cơ sở dữ liệu). Một Database đóng vai trò là không gian tên (namespace) cho các bảng và chứa thông tin về vị trí mặc định trên HDFS hoặc S3.  

Thực thể quan quan trọng nhất là Table (Bảng). Metadata của một bảng bao gồm danh sách các cột, kiểu dữ liệu, thông tin về chủ sở hữu, và quan trọng nhất là Storage Descriptor (Mô tả lưu trữ). Storage Descriptor chứa các chi tiết kỹ thuật như:  

- Vị trí thực tế của dữ liệu (URL đến thư mục trên HDFS hoặc S3).  
    
- Định dạng tệp đầu vào và đầu ra (InputFormat/OutputFormat).  
    
- Thông tin về SerDe (Serializer/Deserializer) để hướng dẫn cách đọc và ghi dữ liệu từ các tệp.  
    

### Phân Vùng (Partition) và Tối Ưu Hóa Truy Cập

Đối với các tập dữ liệu lớn, việc lưu trữ toàn bộ dữ liệu trong một thư mục duy nhất là không hiệu quả. Hive sử dụng khái niệm Partition (Phân vùng) để chia nhỏ dữ liệu thành các thư mục con dựa trên giá trị của các cột phân vùng (ví dụ: ngày, tháng, khu vực).  

Mỗi phân vùng trong Hive Metastore có thể có các cột và thông tin lưu trữ riêng biệt, điều này cho phép thay đổi lược đồ (schema) cho dữ liệu mới mà không ảnh hưởng đến các phân vùng cũ. Khi một truy vấn SQL có điều kiện lọc trên cột phân vùng (ví dụ: `WHERE dt = '2023-10-01'`), công cụ thực thi sẽ yêu cầu HMS chỉ cung cấp vị trí của các thư mục liên quan, giúp bỏ qua việc quét toàn bộ bảng (partition pruning).  

| Bảng Cơ sở dữ liệu | Mô tả chức năng                             | Các trường dữ liệu chính              |
| ------------------ | ------------------------------------------- | ------------------------------------- |
| **DBS**            | Lưu trữ thông tin về cơ sở dữ liệu          | NAME, DB_LOCATION_URI, OWNER_NAME     |
| **TBLS**           | Lưu trữ thông tin về bảng                   | TBL_NAME, DB_ID, OWNER, TBL_TYPE      |
| **SDS**            | Lưu trữ mô tả lưu trữ (Storage Descriptors) | LOCATION, INPUT_FORMAT, OUTPUT_FORMAT |
| **PARTITIONS**     | Lưu trữ các giá trị phân vùng               | PART_NAME, TBL_ID, SD_ID              |
| **COLUMNS_V2**     | Lưu trữ định nghĩa cột                      | COLUMN_NAME, TYPE_NAME, INTEGER_IDX   |

## Các Chế Độ Triển Khai và Cấu Hình Hệ Thống

Việc lựa chọn chế độ triển khai Hive Metastore ảnh hưởng trực tiếp đến tính ổn định, bảo mật và khả năng mở rộng của hệ thống dữ liệu. Có ba chế độ chính được sử dụng phổ biến trong thực tế.  

### Chế Độ Nhúng (Embedded Metastore)

Trong chế độ nhúng, cả dịch vụ Metastore và cơ sở dữ liệu (thường là Apache Derby) đều chạy trong cùng một máy ảo Java (JVM) với ứng dụng Hive. Đây là chế độ mặc định khi cài đặt Hive nhưng nó có một hạn chế nghiêm trọng: cơ sở dữ liệu Derby chỉ cho phép một kết nối duy nhất tại một thời điểm. Nếu một người dùng khác cố gắng mở một phiên làm việc Hive mới, hệ thống sẽ báo lỗi do không thể khóa tệp cơ sở dữ liệu. Do đó, chế độ này chỉ phù hợp cho mục đích kiểm thử đơn vị (unit testing) hoặc trình diễn tính năng.  

### Chế Độ Cục Bộ (Local Metastore)

Chế độ cục bộ di chuyển cơ sở dữ liệu siêu dữ liệu sang một tiến trình riêng biệt, có thể nằm trên một máy chủ khác, trong khi dịch vụ Metastore vẫn chạy trong cùng JVM với HiveServer2 hoặc Spark. Cơ chế này cho phép nhiều người dùng kết nối cùng lúc thông qua một cơ sở dữ liệu quan hệ mạnh mẽ hơn như MySQL hoặc PostgreSQL. Tuy nhiên, mỗi tiến trình máy khách Hive vẫn cần phải biết thông tin đăng nhập JDBC để kết nối với cơ sở dữ liệu, điều này gây khó khăn cho việc quản trị tập trung và bảo mật mật khẩu.  

### Chế Độ Từ Xa (Remote Metastore)

Đây là chế độ được khuyến nghị cho các môi trường sản xuất (production). Trong cấu hình này, dịch vụ Metastore chạy như một tiến trình máy chủ Thrift độc lập trên các nút chuyên dụng. Tất cả các ứng dụng khách (Hive, Spark, Impala, Trino) kết nối với máy chủ này thông qua mạng.  

Ưu điểm vượt trội của chế độ từ xa bao gồm:

- **Bảo mật**: Thông tin đăng nhập cơ sở dữ liệu chỉ được cấu hình trên máy chủ Metastore, không cần chia sẻ cho người dùng cuối.  
    
- **Khả năng mở rộng**: Có thể chạy nhiều máy chủ Metastore đằng sau một bộ cân bằng tải để xử lý hàng ngàn yêu cầu đồng thời.  
    
- **Quản trị**: Việc nâng cấp cơ sở dữ liệu hoặc thay đổi cấu hình diễn ra tập trung tại một nơi duy nhất.  
    

|Chế độ triển khai|Dịch vụ HMS|Cơ sở dữ liệu|Khả năng đáp ứng|Khuyên dùng|
|---|---|---|---|---|
|**Embedded**|Cùng JVM với Client|Derby (Nhúng)|Chỉ 1 người dùng|Unit Testing|
|**Local**|Cùng JVM với Client|RDBMS bên ngoài|Đa người dùng|Môi trường nhỏ|
|**Remote**|JVM độc lập (Thrift)|RDBMS bên ngoài|Rất cao (HA)|Sản xuất/Cloud|

## Quản Trị và Cấu Hình Qua hive-site.xml

Việc định cấu hình Hive Metastore chủ yếu dựa vào tệp `hive-site.xml`, nơi chứa các tham số quan trọng để xác định cách dịch vụ kết nối với cơ sở dữ liệu và cách các máy khách tìm thấy dịch vụ.  

### Các Tham Số Kết Nối Nền Tảng

Để thiết lập một Metastore từ xa, tham số quan trọng nhất là `javax.jdo.option.ConnectionURL`, xác định chuỗi kết nối JDBC đến cơ sở dữ liệu như MySQL hoặc PostgreSQL. Đi kèm với đó là các tham số `javax.jdo.option.ConnectionDriverName`, `javax.jdo.option.ConnectionUserName` và `javax.jdo.option.ConnectionPassword`.  

Đối với máy khách, tham số `hive.metastore.uris` là bắt buộc để chỉ định danh sách các URI của máy chủ Metastore (ví dụ: `thrift://metastore-host:9083`). Nếu danh sách này trống, Hive sẽ mặc định quay lại chế độ nhúng. Ngoài ra, `hive.metastore.warehouse.dir` xác định thư mục gốc trên hệ thống tệp lưu trữ dữ liệu của các bảng không phải là bảng ngoại (managed tables).  

### Cấu Hình Nâng Cao Cho Hiệu Suất và Tính Sẵn Sàng

Để tối ưu hóa cho tải cao, người quản trị thường điều chỉnh các tham số liên quan đến luồng (threads). `hive.metastore.server.min.threads` và `hive.metastore.server.max.threads` kiểm soát kích thước của nhóm luồng xử lý yêu cầu Thrift, với giá trị tối đa có thể lên tới 100.000 trong các phiên bản hiện đại.  

Từ phiên bản Hive 4.0.0 trở đi, HMS hỗ trợ phát hiện dịch vụ động thông qua Apache ZooKeeper bằng tham số `hive.metastore.service.discovery.mode`. Điều này cho phép các máy khách tự động tìm thấy các thực thể Metastore đang hoạt động mà không cần cấu hình danh sách máy chủ tĩnh, tăng cường khả năng phục hồi sau lỗi của hệ thống.  

|Tham số cấu hình|Ý nghĩa|Ví dụ giá trị|
|---|---|---|
|`javax.jdo.option.ConnectionURL`|Địa chỉ JDBC của DB Metastore|`jdbc:mysql://host/db?createDatabaseIfNotExist=true`|
|`hive.metastore.uris`|Danh sách các máy chủ Thrift HMS|`thrift://10.0.0.1:9083,thrift://10.0.0.2:9083`|
|`hive.metastore.warehouse.dir`|Vị trí lưu trữ dữ liệu mặc định|`hdfs://namenode:8020/user/hive/warehouse`|
|`hive.metastore.try.direct.sql`|Cho phép truy vấn SQL trực tiếp|`true`|
|`datanucleus.autoStartMechanism`|Cơ chế khởi tạo lược đồ DB|`SchemaTable`|

## Hiệu Suất và Thách Thức Khi Quy Mô Tăng Trưởng

Mặc dù Hive Metastore là một tiêu chuẩn công nghiệp, nó không phải là không có những hạn chế khi xử lý quy mô dữ liệu ở mức petabyte với hàng triệu đối tượng siêu dữ liệu.  

### Vấn Đề Bùng Nổ Phân Vùng (Partition Explosion)

Một trong những vấn đề phổ biến nhất trong các hệ thống dữ liệu lớn là việc tạo ra quá nhiều phân vùng nhỏ. Nếu một bảng được phân vùng theo những cột có độ chọn lọc cao (high cardinality) như `user_id` hoặc `timestamp` (đến từng giây), số lượng phân vùng có thể tăng vọt lên hàng triệu.  

Việc bùng nổ phân vùng gây ra các tác động tiêu cực sau:

- **Áp lực lên Cơ sở dữ liệu**: Mỗi phân vùng là một bản ghi trong RDBMS. Các truy vấn liệt kê phân vùng sẽ trở nên cực kỳ chậm do phải thực hiện các phép nối (join) lớn giữa các bảng như `PARTITIONS` và `PARTITION_PARAMS`.  
    
- **Kế hoạch truy vấn chậm**: Khi một công cụ như Spark hoặc Hive biên dịch truy vấn, nó phải yêu cầu siêu dữ liệu cho tất cả các phân vùng liên quan. Quá trình này có thể mất nhiều phút nếu số lượng phân vùng quá lớn.  
    
- **Sự cố bộ nhớ**: Máy chủ Metastore hoặc máy phối hợp truy vấn (Query Coordinator) có thể bị lỗi tràn bộ nhớ (OOM) khi cố gắng xử lý danh sách phân vùng khổng lồ.  
    

### Giải Pháp Tối Ưu Hóa: Direct SQL và Caching

Để giải quyết các vấn đề hiệu suất, Hive Metastore cung cấp cơ chế "Direct SQL". Như đã đề cập, cơ chế này thay thế lớp ORM bằng các câu lệnh SQL được tối ưu hóa riêng cho các loại cơ sở dữ liệu như MySQL hay PostgreSQL. Tham số `hive.metastore.try.direct.sql` nên được bật trong các hệ thống lớn để giảm đáng kể thời gian truy xuất phân vùng. Tuy nhiên, cần lưu ý rằng Direct SQL có thể gặp lỗi trên một số phiên bản cơ sở dữ liệu nhất định, ví dụ như lỗi phân biệt chữ hoa/chữ thường trong tên bảng trên PostgreSQL. Trong trường hợp đó, HMS sẽ tự động quay lại sử dụng DataNucleus, gây ra sự sụt giảm hiệu suất đột ngột.  

Ngoài ra, việc kết hợp giữa phân vùng (partitioning) và phân cụm (bucketing) là một chiến lược hiệu quả. Phân vùng giúp chia nhỏ dữ liệu ở cấp độ thư mục, trong khi phân cụm sử dụng hàm băm (hash) để chia dữ liệu thành các tệp có kích thước quản lý được trong mỗi thư mục. Điều này giúp duy trì hiệu suất truy vấn mà không làm quá tải danh mục siêu dữ liệu của HMS.  

## Khả Năng Tương Tác Giữa Các Công Cụ Tính Toán

Sức mạnh thực sự của Hive Metastore nằm ở chỗ nó không chỉ phục vụ Apache Hive mà còn là nguồn sự thật duy nhất (single source of truth) cho nhiều công cụ tính toán khác.  

### Tích Hợp Với Apache Spark

Apache Spark là một trong những máy khách phổ biến nhất của Hive Metastore. Để Spark có thể đọc và ghi các bảng Hive, người dùng phải khởi tạo `SparkSession` với tính năng hỗ trợ Hive thông qua phương thức `.enableHiveSupport()`.  

Vấn đề phức tạp nhất khi sử dụng Spark với HMS là sự không tương thích về phiên bản. Spark đi kèm với các thư viện máy khách Hive tích hợp (thường là phiên bản 2.3.9), nhưng Metastore thực tế có thể đang chạy phiên bản 3.1.x hoặc 4.0.x. Để giải quyết điều này, Spark cung cấp các cấu hình như:  

- `spark.sql.hive.metastore.version`: Chỉ định phiên bản của Metastore đang kết nối.  
    
- `spark.sql.hive.metastore.jars`: Chỉ định vị trí các tệp JAR cần thiết để giao tiếp với phiên bản Metastore đó (có thể là `builtin`, `maven` hoặc một đường dẫn cụ thể).  
    
- `spark.sql.hive.metastore.sharedPrefixes`: Danh sách các tiền tố lớp (class prefixes) cần được chia sẻ giữa bộ nạp lớp của Spark và Hive, điển hình là trình điều khiển JDBC của MySQL hoặc Postgres.  
    

### Tối Ưu Hóa Trino và Presto

Trino (trước đây là PrestoSQL) và PrestoDB được thiết kế để truy vấn hồ dữ liệu với tốc độ cực nhanh. Vì HMS có thể trở thành nút thắt cổ chai cho các truy vấn tương tác, Trino triển khai một hệ thống bộ nhớ đệm (caching) siêu dữ liệu tinh vi.  

Các tham số quan trọng trong Trino bao gồm:

- `hive.metastore-cache-ttl`: Thời gian siêu dữ liệu được lưu trong bộ nhớ đệm trước khi hết hạn.  
    
- `hive.metastore-cache-maximum-size`: Số lượng đối tượng siêu dữ liệu tối đa được lưu trữ.  
    
- `hive.max-partitions-per-scan`: Giới hạn số lượng phân vùng tối đa mà một truy vấn có thể quét (mặc định thường là 1.000.000) để ngăn chặn các truy vấn "hủy diệt" làm quá tải hệ thống.  
    

Việc sử dụng bộ nhớ đệm giúp Trino giảm đáng kể số lượng cuộc gọi RPC đến máy chủ Metastore Thrift, từ đó cải thiện thời gian phản hồi cho các truy vấn lặp lại trên cùng một tập dữ liệu.  

## Dịch Vụ Metastore Trên Đám Mây: Glue và Dataproc

Khi các tổ chức chuyển dịch dữ liệu lên đám mây, việc tự vận hành một cụm Hive Metastore trở nên tốn kém và phức tạp. Các nhà cung cấp đám mây đã giới thiệu các giải pháp thay thế được quản lý hoàn toàn.  

### AWS Glue Data Catalog

AWS Glue Data Catalog là một dịch vụ danh mục siêu dữ liệu phi máy chủ, tương thích với API của Hive Metastore. Điểm mạnh lớn nhất của Glue là khả năng tự động hóa thông qua các "Crawler". Các trình thu thập thông tin này sẽ quét các tệp trên S3, tự động suy diễn lược đồ (schema), định dạng tệp và tạo ra các định nghĩa bảng trong danh mục.  

Tuy nhiên, Glue không phải là một bản sao hoàn hảo của HMS. Nó thiếu một số tính năng nâng cao như hỗ trợ đầy đủ cho các ràng buộc (constraints) của Hive. Ngoài ra, hiệu suất của Glue khi xử lý các bảng có hàng trăm ngàn phân vùng có thể gặp khó khăn nếu không sử dụng "Partition Indexes" – một tính năng của AWS giúp tăng tốc độ tìm kiếm phân vùng bằng cách lập chỉ mục các cột khóa.  

### Google Dataproc Metastore

Ngược lại với cách tiếp cận "tương thích API" của AWS Glue, Google Dataproc Metastore (DPMS) cung cấp một dịch vụ Apache Hive Metastore thực thụ được quản lý hoàn toàn. DPMS chạy phiên bản mã nguồn mở của HMS trên cơ sở hạ tầng của Google, đảm bảo tính tương thích 100% cho các công cụ dựa trên Hadoop.  

DPMS nổi bật với khả năng "Metadata Federation". Tính năng này cho phép các tổ chức hợp nhất nhiều nguồn siêu dữ liệu – bao gồm các thực thể DPMS khác nhau, tập dữ liệu BigQuery và hồ dữ liệu Dataplex – thành một điểm cuối duy nhất. Điều này cực kỳ hữu ích trong các kiến trúc "Data Mesh", nơi mỗi đơn vị kinh doanh có thể sở hữu Metastore riêng nhưng vẫn cần chia sẻ dữ liệu trên toàn doanh nghiệp.  

|Đặc điểm|AWS Glue|GCP Dataproc Metastore|HMS Tự vận hành|
|---|---|---|---|
|**Loại dịch vụ**|Phi máy chủ (Serverless)|Được quản lý (PaaS)|Tự quản lý (IaaS/On-prem)|
|**Tính tương thích**|Tương thích API HMS|HMS mã nguồn mở gốc|HMS mã nguồn mở gốc|
|**Tự động hóa**|Glue Crawlers mạnh mẽ|Hỗ trợ nhập (Import)|Thủ công|
|**Cơ chế giá**|Theo đối tượng/yêu cầu|Theo cấu hình máy chủ|Theo hạ tầng|
|**Phạm vi**|Chỉ AWS|GCP (có hỗ trợ Hybrid)|Mọi nơi|

## Bảo Mật, Quản Trị và Quyền Truy Cập

Vì Hive Metastore nắm giữ "chìa khóa" để hiểu dữ liệu trong hồ dữ liệu, bảo mật cho thành phần này là ưu tiên hàng đầu.  

### Xác Thực và Ủy Quyền

Trong các môi trường doanh nghiệp truyền thống, HMS được bảo vệ bằng Kerberos để đảm bảo chỉ những người dùng và dịch vụ hợp lệ mới có thể kết nối. Sau khi được xác thực, quyền truy cập vào các cơ sở dữ liệu và bảng cụ thể thường được quản lý thông qua Apache Ranger hoặc tích hợp với các hệ thống kiểm soát truy cập dựa trên vai trò (RBAC).  

Apache Ranger cung cấp khả năng quản lý chính sách tập trung, cho phép định nghĩa các quyền chi tiết đến mức cột hoặc thậm chí là lọc dòng (row filtering) dựa trên thuộc tính của người dùng. Khi một truy vấn được gửi đến, công cụ tính toán sẽ kiểm tra với chính sách của Ranger để xem người dùng có quyền truy cập vào siêu dữ liệu mà HMS cung cấp hay không.  

### Quản Trị Dữ Liệu và Dòng Dữ Liệu (Lineage)

Hive Metastore cũng đóng vai trò là nền tảng cho việc quản trị dữ liệu (Data Governance) thông qua việc tích hợp với Apache Atlas. Atlas thu thập siêu dữ liệu từ HMS để xây dựng biểu đồ dòng dữ liệu (lineage), cho phép các tổ chức theo dõi cách dữ liệu được biến đổi từ các bảng nguồn sang các bảng đích. Điều này cực kỳ quan trọng đối với việc tuân thủ các quy định như GDPR hoặc HIPAA, nơi tổ chức cần biết chính xác dữ liệu nhạy cảm đang được lưu trữ và xử lý ở đâu.  

Trong thế giới đám mây, Microsoft Purview cung cấp các khả năng tương tự bằng cách quét (scanning) các máy chủ Hive Metastore để trích xuất siêu dữ liệu kỹ thuật, phân loại dữ liệu và hiển thị dòng dữ liệu trên giao diện hợp nhất. Purview có thể kết nối với các Metastore trong Azure Databricks hoặc các cụm HDInsight để mang lại cái nhìn toàn diện về tài sản dữ liệu của doanh nghiệp.  

## Metastore Trong Kỷ Nguyên Lakehouse: Iceberg và Delta Lake

Sự trỗi dậy của các định dạng bảng hiện đại như Apache Iceberg và Delta Lake đã thay đổi vai trò của Hive Metastore từ một trình quản lý tệp chi tiết thành một trình quản lý trạng thái bảng.  

### Sự Thay Đổi Về Cách Lưu Trữ Siêu Dữ Liệu

Trong các bảng Hive truyền thống, Metastore lưu trữ vị trí của mọi phân vùng và mọi tệp trong cơ sở dữ liệu quan hệ của nó. Đối với Iceberg, danh sách các tệp dữ liệu thực tế được lưu trữ trong các tệp JSON hoặc Avro ngay trên hệ thống lưu trữ (S3/HDFS) cùng với dữ liệu.  

Hive Metastore bây giờ chỉ cần lưu trữ một "con trỏ" (pointer) đến vị trí của tệp siêu dữ liệu mới nhất của bảng đó. Khi một giao dịch ghi mới hoàn tất, Iceberg sẽ cập nhật con trỏ này trong HMS thông qua một thao tác cập nhật nguyên tử (atomic update). Cách tiếp cận này giúp loại bỏ hoàn toàn vấn đề bùng nổ phân vùng trong cơ sở dữ liệu HMS, vì Metastore không còn phải theo dõi hàng triệu dòng cho các phân vùng nữa.  

### Metadata Federation và Unity Catalog

Để quản lý hàng ngàn Metastore phân tán, các giải pháp như Databricks Unity Catalog đã ra đời. Unity Catalog hoạt động như một lớp quản trị thống nhất bên trên các Hive Metastore hiện có, cho phép các tổ chức áp dụng các chính sách bảo mật đồng nhất trên nhiều đám mây và nhiều tài khoản khác nhau.  

Việc tích hợp "HMS Federation" vào Unity Catalog cho phép các doanh nghiệp tận dụng các tính năng hiện đại như kiểm soát truy cập mịn, gắn thẻ dữ liệu (tagging) và kiểm toán (auditing) mà không cần phải di chuyển siêu dữ liệu từ các Metastore cũ sang hệ thống mới. Đây là minh chứng cho thấy Hive Metastore vẫn là nền tảng không thể thiếu, ngay cả khi các công nghệ mới hơn đang dần thay thế các chức năng lưu trữ chi tiết của nó.  

## Phân Tích Thực Tiễn và Khuyến Nghị Vận Hành

Dựa trên các nghiên cứu về kiến trúc và hiệu suất, việc vận hành Hive Metastore trong môi trường doanh nghiệp đòi hỏi sự chú trọng vào các khía cạnh sau để đảm bảo hệ thống hoạt động ổn định và hiệu quả.

### Chiến Lược Lựa Chọn Cơ Sở Dữ Liệu Phụ Trợ

Mặc dù HMS hỗ trợ nhiều loại cơ sở dữ liệu, MySQL và PostgreSQL là hai lựa chọn phổ biến nhất do tính phổ biến và khả năng mở rộng tốt trên đám mây dưới dạng các dịch vụ được quản lý (như AWS RDS hoặc Azure SQL Database).  

Đối với các hệ thống cực lớn, việc tinh chỉnh cơ sở dữ liệu quan hệ là cần thiết. Người quản trị nên đảm bảo rằng các bảng siêu dữ liệu như `PARTITIONS` và `TBLS` có các chỉ mục (indexes) phù hợp và cơ sở dữ liệu có đủ tài nguyên CPU/RAM để xử lý các phép nối phức tạp trong quá trình lập kế hoạch truy vấn. Ngoài ra, việc sử dụng các phiên bản cơ sở dữ liệu được hỗ trợ tối thiểu (ví dụ: MySQL 5.6.17+, PostgreSQL 9.1.13+, Oracle 11g+) là bắt buộc để tránh các lỗi không mong muốn về tính nhất quán của lược đồ.  

### Quản Lý Tệp Nhỏ và Hiệu Suất Lưu Trữ

Vấn đề "tệp nhỏ" (small files) không chỉ ảnh hưởng đến hiệu suất đọc của các công cụ tính toán mà còn làm tăng tải lên Metastore và NameNode của HDFS. Mỗi tệp là một thực thể cần được HMS theo dõi gián tiếp qua các Storage Descriptors.  

Để duy trì một hệ sinh thái Hive khỏe mạnh, các tổ chức nên:

- **Tăng kích thước tệp**: Khuyên dùng các tệp Parquet hoặc ORC có kích thước lớn hơn 100MB (lý tưởng là 128MB đến 1GB).  
    
- **Sử dụng Data Compaction**: Định kỳ thực hiện các thao tác nén dữ liệu để gộp các tệp nhỏ thành tệp lớn hơn, giúp giảm bớt số lượng đối tượng cần quản lý trong danh mục siêu dữ liệu.  
    
- **Cấu hình giới hạn an toàn**: Thiết lập các tham số như `hive.max-partitions-per-scan` trong các công cụ truy vấn để ngăn chặn tình trạng một truy vấn duy nhất có thể làm tê liệt toàn bộ cụm máy chủ.  
    

## Kết Luận

Apache Hive Metastore đã đi một chặng đường dài từ vai trò là một thành phần bổ trợ cho Hive tại Facebook để trở thành một "xương sống" không thể thay thế của hệ sinh thái dữ liệu lớn hiện đại. Khả năng của nó trong việc trừu tượng hóa các cấu trúc tệp phức tạp thành các bảng SQL quen thuộc đã dân chủ hóa việc truy cập dữ liệu cho hàng triệu người dùng trên toàn thế giới.  

Mặc dù phải đối mặt với những thách thức về hiệu suất khi quy mô dữ liệu tăng lên và sự cạnh tranh từ các dịch vụ đám mây phi máy chủ, HMS vẫn tiếp tục tiến hóa. Sự hỗ trợ cho các định dạng bảng hiện đại như Iceberg và sự xuất hiện của các lớp quản trị liên hợp như Unity Catalog cho thấy HMS sẽ tiếp tục tồn tại như một lớp khám phá dữ liệu và điều phối giao dịch quan trọng.  

Đối với các chuyên gia dữ liệu, việc hiểu sâu về cơ chế hoạt động của Hive Metastore – từ cấu hình Thrift, mô hình dữ liệu quan hệ cho đến các kỹ thuật tối ưu hóa Direct SQL – là điều kiện tiên quyết để xây dựng và vận hành các nền tảng dữ liệu lớn ổn định, an toàn và có khả năng mở rộng trong kỷ nguyên số.  

[

![](https://t1.gstatic.com/faviconV2?url=https://versent.com.au/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

versent.com.au

How does Hive Metastore work? - Versent

Mở trong cửa sổ mới](https://versent.com.au/blog/how-does-hive-metastore-work/)[

![](https://t0.gstatic.com/faviconV2?url=https://medium.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

medium.com

How does Hive Metastore work? - Medium

Mở trong cửa sổ mới](https://medium.com/versent-tech-blog/how-does-hive-metastore-work-e9ba9f236454)[

![](https://t3.gstatic.com/faviconV2?url=https://www.dremio.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

dremio.com

Hive Metastore - Dremio

Mở trong cửa sổ mới](https://www.dremio.com/wiki/hive-metastore/)[

![](https://t1.gstatic.com/faviconV2?url=https://hive.apache.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

hive.apache.org

Apache Hive

Mở trong cửa sổ mới](https://hive.apache.org/)[

![](https://t1.gstatic.com/faviconV2?url=https://hive.apache.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

hive.apache.org

Design - Apache Hive

Mở trong cửa sổ mới](https://hive.apache.org/development/desingdocs/design/)[

![](https://t0.gstatic.com/faviconV2?url=https://data-flair.training/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

data-flair.training

Different Ways to Configure Hive Metastore - DataFlair

Mở trong cửa sổ mới](https://data-flair.training/blogs/apache-hive-metastore/)[

![](https://t3.gstatic.com/faviconV2?url=https://www.starburst.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

starburst.io

Hive Architecture and Hive Connector - Starburst

Mở trong cửa sổ mới](https://www.starburst.io/blog/hive-architecture/)[

![](https://t2.gstatic.com/faviconV2?url=https://www.quora.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

quora.com

What are the different types of metastores that Hive provides? - Quora

Mở trong cửa sổ mới](https://www.quora.com/What-are-the-different-types-of-metastores-that-Hive-provides)[

![](https://t0.gstatic.com/faviconV2?url=https://yandex.cloud/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

yandex.cloud

Apache Hive™ Metastore clusters | Yandex Cloud - Documentation

Mở trong cửa sổ mới](https://yandex.cloud/en/docs/metadata-hub/concepts/metastore)[

![](https://t3.gstatic.com/faviconV2?url=https://mageswaran1989.medium.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

mageswaran1989.medium.com

Big Data Play Ground For Engineers : Hive and Metastore | by Mageswaran D - Medium

Mở trong cửa sổ mới](https://mageswaran1989.medium.com/big-data-play-ground-for-engineers-hive-and-metastore-15a977169eb7)[

![](https://t3.gstatic.com/faviconV2?url=https://dev-cloudera.4shared.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

dev-cloudera.4shared.com

Configuring the Hive Metastore for CDH - Cloudera Manager

Mở trong cửa sổ mới](https://dev-cloudera.4shared.com/static/help/topics/cdh_ig_hive_metastore_configure.html)[

![](https://t3.gstatic.com/faviconV2?url=https://cwiki.apache.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

cwiki.apache.org

Hive Metastore Administration - Confluence Mobile - Apache ...

Mở trong cửa sổ mới](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Metastore+Administration)[

![](https://t3.gstatic.com/faviconV2?url=https://trino.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

trino.io

Metastores — Trino 480 Documentation

Mở trong cửa sổ mới](https://trino.io/docs/current/object-storage/metastores.html)[

![](https://t3.gstatic.com/faviconV2?url=https://docs.datahub.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

docs.datahub.com

Hive Metastore - DataHub

Mở trong cửa sổ mới](https://docs.datahub.com/docs/generated/ingestion/sources/hive-metastore)[

![](https://t0.gstatic.com/faviconV2?url=https://data-flair.training/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

data-flair.training

What is the difference between local and remote Metastore? - DataFlair

Mở trong cửa sổ mới](https://data-flair.training/forums/topic/what-is-the-difference-between-local-and-remote-metastore/)[

![](https://t1.gstatic.com/faviconV2?url=https://hive.apache.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

hive.apache.org

AdminManual Metastore Administration - Apache Hive

Mở trong cửa sổ mới](https://hive.apache.org/docs/latest/admin/adminmanual-metastore-administration/)[

![](https://t3.gstatic.com/faviconV2?url=https://blog.devgenius.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

blog.devgenius.io

Hive Metastore. A bridge between files and exposed… | by Amit Singh Rathore - Dev Genius

Mở trong cửa sổ mới](https://blog.devgenius.io/hive-metastore-a9dc9e139cf2)[

![](https://t0.gstatic.com/faviconV2?url=https://lakefs.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

lakefs.io

Hive Metastore's Dilemma: Performance Limits - lakeFS

Mở trong cửa sổ mới](https://lakefs.io/blog/hive-metastore-it-didnt-age-well/)[

![](https://t1.gstatic.com/faviconV2?url=http://www.openkb.info/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

openkb.info

Hive metastore errors out "Direct SQL failed, falling back to ORM" - Open Knowledge Base

Mở trong cửa sổ mới](http://www.openkb.info/2014/12/hive-metastore-errors-out-direct-sql.html)[

![](https://t3.gstatic.com/faviconV2?url=https://www.e6data.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

e6data.com

Managing Permissions in Hive Metastore for Lakehouses - e6data

Mở trong cửa sổ mới](https://www.e6data.com/blog/a-comprehensive-guide-for-managing-permissions-in-hive-metastore-for-lakehouses)[

![](https://t3.gstatic.com/faviconV2?url=https://www.pepperdata.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

pepperdata.com

Hive Performance Tuning Tips for Hive Query Optimization - Pepperdata

Mở trong cửa sổ mới](https://www.pepperdata.com/blog/hive-query-tuning/)[

![](https://t0.gstatic.com/faviconV2?url=https://towardsdatascience.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

towardsdatascience.com

Must-Know Techniques for Handling Big Data in Hive

Mở trong cửa sổ mới](https://towardsdatascience.com/must-know-techniques-for-handling-big-data-in-hive-fa70e020141d/)[

![](https://t0.gstatic.com/faviconV2?url=https://medium.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

medium.com

Apache Hive Best Practices and Optimization Techniques - Medium

Mở trong cửa sổ mới](https://medium.com/@bkvs88/apache-hive-best-practices-and-optimization-techniques-c749df681380)[

![](https://t2.gstatic.com/faviconV2?url=https://www.packtpub.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

packtpub.com

Deploying Hive Metastore - Packt+ | Advance your knowledge in tech

Mở trong cửa sổ mới](https://www.packtpub.com/en-CO/product/apache-hive-cookbook-9781782161080/chapter/1-dot-developing-hive-1/section/deploying-hive-metastore-ch01lvl1sec04)[

![](https://t1.gstatic.com/faviconV2?url=https://docs.dataedo.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

docs.dataedo.com

Apache Hive Metastore support - Dataedo documentation

Mở trong cửa sổ mới](https://docs.dataedo.com/docs/documenting-technology/supported-databases/apache-hive-metastore/)[

![](https://t0.gstatic.com/faviconV2?url=https://support.hpe.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

support.hpe.com

How do I configure Hive Client to use embedded vs remote metastore - HPE Support

Mở trong cửa sổ mới](https://support.hpe.com/hpesc/public/docDisplay?docId=sf000090313en_us&docLocale=en_US)[

![](https://t3.gstatic.com/faviconV2?url=https://docs.aws.amazon.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

docs.aws.amazon.com

Optimizing read performance - AWS Prescriptive Guidance

Mở trong cửa sổ mới](https://docs.aws.amazon.com/prescriptive-guidance/latest/apache-iceberg-on-aws/best-practices-read.html)[

![](https://t1.gstatic.com/faviconV2?url=https://hive.apache.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

hive.apache.org

Documentation - Apache Hive

Mở trong cửa sổ mới](https://hive.apache.org/docs/latest/)[

![](https://t1.gstatic.com/faviconV2?url=https://docs.cloudera.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

docs.cloudera.com

Apache Hive Performance Tuning - Cloudera Documentation

Mở trong cửa sổ mới](https://docs.cloudera.com/cdw-runtime/cloud/hive-performance-tuning/hive_performance_tuning.pdf)[

![](https://t0.gstatic.com/faviconV2?url=https://data-flair.training/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

data-flair.training

7 Best Hive Optimization Techniques - Hive Performance - DataFlair

Mở trong cửa sổ mới](https://data-flair.training/blogs/hive-optimization-techniques/)[

![](https://t3.gstatic.com/faviconV2?url=https://trino.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

trino.io

Ecosystem: Data lake components - Trino

Mở trong cửa sổ mới](https://trino.io/ecosystem/data-lake.html)[

![](https://t3.gstatic.com/faviconV2?url=https://spark.apache.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

spark.apache.org

Hive Tables - Spark 4.1.1 Documentation - Apache Spark

Mở trong cửa sổ mới](https://spark.apache.org/docs/latest/sql-data-sources-hive-tables.html)[

![](https://t0.gstatic.com/faviconV2?url=https://medium.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

medium.com

Local Data Lake Setup: Spark, Delta Lake, Hive Metastore, and Trino | by Pradeep - Medium

Mở trong cửa sổ mới](https://medium.com/@prad8531/local-delta-lake-setup-spark-delta-lake-hive-metastore-and-trino-c902d695bf6b)[

![](https://t2.gstatic.com/faviconV2?url=https://community.databricks.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

community.databricks.com

How to know Legacy Metastore connection to SQL DB (used to store metadata) - Databricks Community

Mở trong cửa sổ mới](https://community.databricks.com/t5/administration-architecture/how-to-know-legacy-metastore-connection-to-sql-db-used-to-store/td-p/112268)[

![](https://t0.gstatic.com/faviconV2?url=https://medium.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

medium.com

External Hive Metastores on Databricks on AWS. | by Syed Ismail - Medium

Mở trong cửa sổ mới](https://medium.com/@syedahmedismail98/external-hive-metastores-on-databricks-on-aws-8e29490caa51)[

![](https://t1.gstatic.com/faviconV2?url=https://jaceklaskowski.gitbooks.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

jaceklaskowski.gitbooks.io

Configuration Properties · The Internals of Spark SQL - Jacek Laskowski (@jaceklaskowski)

Mở trong cửa sổ mới](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/hive/configuration-properties.html)[

![](https://t3.gstatic.com/faviconV2?url=https://ta.thinkingdata.cn/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

ta.thinkingdata.cn

Hive connector storage caching — Trino 435 Documentation

Mở trong cửa sổ mới](https://ta.thinkingdata.cn/presto-docs/connector/hive-caching.html)[

![](https://t2.gstatic.com/faviconV2?url=https://www.alibabacloud.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

alibabacloud.com

Hive connector - E-MapReduce - Alibaba Cloud Documentation Center

Mở trong cửa sổ mới](https://www.alibabacloud.com/help/en/emr/emr-on-ecs/user-guide/hive-connector-1)[

![](https://t1.gstatic.com/faviconV2?url=https://prestodb.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

prestodb.io

Hive Connector - Presto 0.296 Documentation

Mở trong cửa sổ mới](https://prestodb.io/docs/current/connector/hive.html)[

![](https://t3.gstatic.com/faviconV2?url=https://trino.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

trino.io

Hive connector — Trino 480 Documentation

Mở trong cửa sổ mới](https://trino.io/docs/current/connector/hive.html)[

![](https://t0.gstatic.com/faviconV2?url=https://lakefs.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

lakefs.io

Metadata Management: Hive Metastore vs AWS Glue - lakeFS

Mở trong cửa sổ mới](https://lakefs.io/blog/metadata-management-hive-metastore-vs-aws-glue/)[

![](https://t0.gstatic.com/faviconV2?url=https://www.perarduaconsulting.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

perarduaconsulting.com

Comparing Apache Hive, AWS Glue, and Google Data Catalog - Perardua Consulting

Mở trong cửa sổ mới](https://www.perarduaconsulting.com/post/comparing-apache-hive-aws-glue-and-google-data-catalog)[

![](https://t0.gstatic.com/faviconV2?url=https://risingwave.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

risingwave.com

Iceberg Catalog Comparison: Hive Metastore vs AWS Glue vs REST vs Nessie | RisingWave

Mở trong cửa sổ mới](https://risingwave.com/blog/iceberg-catalog-comparison-guide/)[

![](https://t3.gstatic.com/faviconV2?url=https://dbdocs.io/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

dbdocs.io

Top 6 Data Catalog Tools Ranked in 2025 (With a Developer-Friendly Surprise!) - dbdocs.io

Mở trong cửa sổ mới](https://dbdocs.io/blog/posts/2025-04-11-top-6-data-catalog-tools/)[

![](https://t0.gstatic.com/faviconV2?url=https://medium.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

medium.com

Hive vs AWS Glue: How Do These Two ETL Solutions Compare? | by DataTechBridge

Mở trong cửa sổ mới](https://medium.com/@DataTechBridge/from-hive-to-glue-should-you-migrate-your-etl-workloads-to-awss-serverless-platform-23e68f979d08)[

![](https://t2.gstatic.com/faviconV2?url=https://aws.amazon.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

aws.amazon.com

Improve query performance using AWS Glue partition indexes | AWS Big Data Blog

Mở trong cửa sổ mới](https://aws.amazon.com/blogs/big-data/improve-query-performance-using-aws-glue-partition-indexes/)[

![](https://t0.gstatic.com/faviconV2?url=https://docs.cloud.google.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

docs.cloud.google.com

Dataproc Metastore overview - Google Cloud Documentation

Mở trong cửa sổ mới](https://docs.cloud.google.com/dataproc-metastore/docs/overview)[

![](https://t3.gstatic.com/faviconV2?url=https://www.devopsschool.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

devopsschool.com

Google Cloud Dataproc Metastore Tutorial: Architecture, Pricing, Use Cases, and Hands-On Guide for Data analytics and pipelines - DevOpsSchool

Mở trong cửa sổ mới](https://www.devopsschool.com/tutorials/google-cloud-dataproc-metastore-tutorial-architecture-pricing-use-cases-and-hands-on-guide-for-data-analytics-and-pipelines/)[

![](https://t0.gstatic.com/faviconV2?url=https://cloud.google.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

cloud.google.com

Four Dataproc Metastore deployments patterns | Google Cloud Blog

Mở trong cửa sổ mới](https://cloud.google.com/blog/products/data-analytics/four-dataproc-metastore-deployments-patterns/)[

![](https://t0.gstatic.com/faviconV2?url=https://medium.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

medium.com

AWS vs. GCP vs. Azure: Comparing Cloud Data Engineering Tools for Storage, Processing, Streaming, and More | by Shreeraj Gujar | Medium

Mở trong cửa sổ mới](https://medium.com/@shreerajgujar/aws-vs-gcp-vs-0653fded1bba)[

![](https://t2.gstatic.com/faviconV2?url=https://www.databricks.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

databricks.com

Announcing General Availability of Hive Metastore and AWS Glue Federation in Unity Catalog | Databricks Blog

Mở trong cửa sổ mới](https://www.databricks.com/blog/announcing-public-preview-hive-metastore-and-aws-glue-federation-unity-catalog)[

![](https://t1.gstatic.com/faviconV2?url=https://www.jamesserra.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

jamesserra.com

Microsoft Purview FAQ | James Serra's Blog

Mở trong cửa sổ mới](https://www.jamesserra.com/archive/2024/09/microsoft-purview-faq/)[

![](https://t3.gstatic.com/faviconV2?url=https://learn.microsoft.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

learn.microsoft.com

Connect to and manage Azure Databricks in Microsoft Purview

Mở trong cửa sổ mới](https://learn.microsoft.com/en-us/purview/register-scan-azure-databricks)[

![](https://t3.gstatic.com/faviconV2?url=https://learn.microsoft.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

learn.microsoft.com

Connect to and manage Hive Metastore databases in Microsoft Purview

Mở trong cửa sổ mới](https://learn.microsoft.com/en-us/purview/register-scan-hive-metastore-source)[

![](https://t1.gstatic.com/faviconV2?url=https://docs.cloudera.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

docs.cloudera.com

Setting up the backend Hive metastore database - Cloudera Documentation

Mở trong cửa sổ mới](https://docs.cloudera.com/cdw-runtime/1.5.4/hive-metastore/topics/hive_setting_up_database_for_hms.html)