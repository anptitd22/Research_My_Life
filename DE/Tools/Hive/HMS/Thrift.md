**Bản chất:** Là một **RPC Framework** (gọi hàm từ xa)

Khung phần mềm Apache Thrift, dùng để phát triển các dịch vụ đa ngôn ngữ có khả năng mở rộng, kết hợp một ngăn xếp phần mềm với một công cụ tạo mã để xây dựng các dịch vụ hoạt động hiệu quả và liền mạch giữa C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, OCaml và Delphi cùng các ngôn ngữ khác.

**Writing a .thrift file**

Sau khi cài đặt trình biên dịch Thrift, bạn cần tạo một tệp Thrift. Tệp này là định nghĩa giao diện bao gồm các kiểu Thrift và các Dịch vụ. Các dịch vụ bạn định nghĩa trong tệp này được máy chủ triển khai và được gọi bởi bất kỳ máy khách nào. Trình biên dịch Thrift được sử dụng để tạo tệp Thrift của bạn thành mã nguồn được sử dụng bởi các thư viện máy khách khác nhau và máy chủ mà bạn viết. Để tạo mã nguồn từ tệp Thrift, hãy chạy lệnh sau:

```bash
thrift --gen <language> <Thrift filename>
```

Để tìm hiểu thêm về Apache Thrift [Read the Whitepaper](https://thrift.apache.org/static/files/thrift-20070401.pdf)
### 1. Lớp Định nghĩa (IDL - Interface Definition Language)

Đây là tầng cao nhất, nơi bạn viết file `.thrift`. Nó giống như một **bản hợp đồng kinh tế** mà cả hai bên (Client và Server) đều phải ký vào.

- **Nhiệm vụ:** Định nghĩa cấu trúc dữ liệu (`struct`) và các hàm dịch vụ (`service`).
- **Tính chất:** Nó độc lập hoàn toàn với ngôn ngữ lập trình. Bạn viết một lần, nhưng có thể dùng để sinh ra code cho Java, Python, C++, PHP, Go...
- **Ví dụ:** Bạn định nghĩa hàm `get_table(string name)` trong file này.

### 2. Lớp Mã hóa (Protocol Layer)

Sau khi có dữ liệu từ ứng dụng, Thrift cần quyết định xem sẽ "gói" dữ liệu đó thành định dạng gì để gửi đi.

- **Binary Protocol:** Chuyển dữ liệu thành dạng nhị phân. Đây là lý do Thrift nhanh hơn REST/JSON (vốn là dạng văn bản).
- **Compact Protocol:** Giống Binary nhưng được nén thêm một lần nữa để cực kỳ tiết kiệm băng thông.
- **JSON Protocol:** Dùng cho các trường hợp cần đọc được bằng mắt thường (nhưng ít khi dùng trong Big Data vì chậm).

### 3. Lớp Truyền tải (Transport Layer)

Đây là cách thức dữ liệu di chuyển từ máy này sang máy khác (cách "đặt đường ống").

- **TSocket:** Sử dụng TCP/IP để gửi dữ liệu trực tiếp qua mạng (Đây là cách HMS nói chuyện với Trino ở Port 9083).
- **TFileTransport:** Ghi dữ liệu vào file.
- **TFramedTransport:** Chia dữ liệu thành từng khung (frames) để gửi, thường dùng khi hệ thống cần xử lý không đồng bộ (Non-blocking).

### 4. Lớp Máy chủ (Server Layer)

Đây là cách Server xử lý các yêu cầu đổ về.

- **TSimpleServer:** Chỉ xử lý một yêu cầu tại một thời điểm (Dùng để test).
- **TThreadPoolServer:** Tạo ra một "hồ bơi" các luồng (threads). Mỗi khi có yêu cầu mới, một thread sẽ đứng ra xử lý. Đây là cách **Hive Metastore** thường dùng để phục vụ nhiều kết nối cùng lúc từ Trino/Spark.
- **TNonblockingServer:** Xử lý hàng nghìn kết nối cùng lúc mà không làm treo hệ thống (giống cơ chế của Node.js).

Nguồn: https://thrift.apache.org/
