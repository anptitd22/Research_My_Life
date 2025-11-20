Trang này cung cấp tổng quan về kiến ​​trúc triển khai AIStor Server từ góc độ production.

Việc triển khai AIStor Server bao gồm ít nhất 4 máy chủ có lưu trữ đồng nhất và tính toán tài nguyên.

![alt text](/images/Pasted_image_20251120235330.png)

Các máy chủ này tạo thành một nhóm máy chủ (**server pool**), trong đó Máy chủ AIStor trình bày tổng hợp tính toán, bộ nhớ và lưu trữ dưới dạng một tài nguyên duy nhất cho máy khách. Mỗi nhóm bao gồm một hoặc nhiều [erasure sets](https://docs.min.io/enterprise/aistor-object-store/operations/core-concepts/erasure-coding/).

Mỗi Máy chủ AIStor đều có bức tranh hoàn chỉnh về cấu trúc phân tán, sao cho ứng dụng có thể kết nối và chỉ đạo các hoạt động trên bất kỳ node nào trong quá trình triển khai.

Các ứng dụng thường không nên tự quản lý những kết nối đó, vì bất kỳ thay đổi nào trong kiến trúc triển khai cũng sẽ yêu cầu cập nhật lại ứng dụng. Trong môi trường vận hành thực tế (production), thay vào đó nên triển khai một bộ cân bằng tải (load balancer) hoặc một thành phần điều phối mạng tương tự để quản lý các kết nối đến cụm AIStor Server.

![alt text](/images/Pasted_image_20251121000549.png)

Các ứng dụng khách hàng có thể sử dụng bất kỳ SDK hoặc thư viện nào tương thích với S3 để tương tác với việc triển khai AIStor Server.

![alt text](/images/Pasted_image_20251121000744.png)

AIStor cung cấp các SDK tập trung vào S3 cho nhiều ngôn ngữ để thuận tiện cho nhà phát triển. Các thư viện này chỉ cung cấp chức năng API S3 và không bao gồm mã bổ sung để hỗ trợ các tính năng lưu trữ đám mây không liên quan.

Bạn có thể mở rộng dung lượng lưu trữ khả dụng của Máy chủ AIStor thông qua việc mở rộng nhóm (**pool**)

Mỗi pool bao gồm một nhóm node độc lập với các erasure set riêng của chúng. [Adding new pools](https://docs.min.io/enterprise/aistor-object-store/operations/scaling/) đòi hỏi phải cập nhật toàn bộ các node trong hệ thống với topology mới. Trong các cụm đa-pool, khi xử lý một request, node AIStor Server nhận được phải xác định pool nào sẽ được sử dụng để định tuyến yêu cầu đó.

![alt text](/images/Pasted_image_20251121001312.png)

Việc mở rộng pool yêu cầu cập nhật bất kỳ bộ cân bằng tải (load balancer) hoặc các một thành phần điều phối mạng tương tự nào với bố cục tên máy chủ mới để đảm bảo phân bổ tải đều trên các mode.



