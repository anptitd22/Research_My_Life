Kiến trúc có trạng thái (Stateful) so với kiến ​​trúc không trạng thái (Stateless) định nghĩa cách hệ thống quản lý dữ liệu phiên của máy khách trong quá trình tương tác. Điều này ảnh hưởng đến khả năng mở rộng, hiệu suất và thiết kế hệ thống.

- ***Stateful:*** Máy chủ lưu trữ dữ liệu phiên giữa nhiều yêu cầu.
- ***Stateless:*** Mỗi yêu cầu là độc lập và không lưu trữ phiên làm việc nào.
- ***Usage:*** Hệ thống không trạng thái (stateless) được ưu tiên sử dụng cho các hệ thống phân tán và có khả năng mở rộng.

![alt][images/System_Design/stateless_and_statefull_1.png]

## Stateful Architecture

Máy chủ duy trì trạng thái hoặc thông tin phiên của mỗi máy khách. Điều này có nghĩa là máy chủ theo dõi dữ liệu và ngữ cảnh của máy khách trong suốt nhiều tương tác hoặc yêu cầu.

- Thường liên quan đến việc lưu trữ dữ liệu phiên trong bộ nhớ máy chủ, cơ sở dữ liệu hoặc các cơ chế lưu trữ khác.

- Ví dụ bao gồm các ứng dụng web truyền thống sử dụng phiên máy chủ để lưu trữ dữ liệu người dùng hoặc nội dung giỏ hàng.

Ví dụ: Một trang web mua sắm trực tuyến, nơi máy chủ lưu trữ thông tin đăng nhập và giỏ hàng của người dùng. Nếu người dùng thêm sản phẩm vào giỏ hàng, máy chủ sẽ ghi nhớ những sản phẩm đó trong suốt phiên đăng nhập.

## Stateless Architecture

Máy chủ không lưu trữ bất kỳ thông tin phiên làm việc nào của máy khách giữa các yêu cầu. Mỗi yêu cầu từ máy khách được coi là một giao dịch độc lập.

- Để duy trì phiên người dùng, các kiến ​​trúc không trạng thái thường sử dụng các kỹ thuật như JSON Web Tokens (JWT) hoặc cookie phía máy khách để lưu trữ dữ liệu phiên.

- Được thiết kế để có khả năng mở rộng và chịu lỗi tốt hơn vì chúng không yêu cầu tài nguyên máy chủ để duy trì trạng thái của máy khách.

- Ví dụ bao gồm các API RESTful, trong đó mỗi yêu cầu chứa tất cả thông tin cần thiết để máy chủ xử lý độc lập.

Ví dụ: Một API REST cho ứng dụng di động, trong đó mỗi yêu cầu đều bao gồm một mã thông báo xác thực (JWT). Máy chủ xác minh mã thông báo và xử lý yêu cầu mà không lưu trữ thông tin phiên.

## Stateful Vs Stateless Architecture

Dưới đây là những điểm khác biệt giữa kiến ​​trúc có trạng thái (stateful) và kiến ​​trúc không trạng thái (stateless):

| ***Stateful Architecture***                                                      | ***Stateless Architecture***                                                                                   |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Việc mở rộng quy mô đòi hỏi phải đồng bộ hóa dữ liệu phiên.                      | Mở rộng theo chiều ngang rất đơn giản.                                                                         |
| Sự cố ở một máy chủ có thể ảnh hưởng đến các phiên được lưu trữ trên máy chủ đó. | Các lỗi này xảy ra riêng lẻ, chỉ ảnh hưởng đến từng yêu cầu riêng biệt.                                        |
| Có thể gặp phải hiện tượng độ trễ tăng lên do quản lý phiên.                     | Thông thường thời gian phản hồi nhanh hơn do không cần xử lý dữ liệu phiên.                                    |
| Cần nhiều tài nguyên hơn để lưu trữ và quản lý trạng thái phiên.                 | Sử dụng tài nguyên hiệu quả vì không lưu trữ trạng thái phiên.                                                 |
| Việc lưu vào bộ nhớ đệm có thể phức tạp do dữ liệu phụ thuộc vào phiên.          | Việc lưu vào bộ nhớ đệm đơn giản hơn vì các yêu cầu độc lập với nhau.                                          |
| Việc triển khai có thể phức tạp vì dữ liệu phiên phải được đồng bộ hóa.          | Việc triển khai và bảo trì trở nên dễ dàng hơn nhờ bản chất không lưu trạng thái.                              |
| Duy trì ngữ cảnh phiên để đảm bảo tính liên tục của giao dịch.                   | Các giao dịch được xử lý độc lập ở cấp độ yêu cầu.                                                             |
| Cân bằng tải có thể yêu cầu tính liên kết phiên (phiên cố định).                 | Cân bằng tải đơn giản hơn vì bất kỳ máy chủ nào cũng có thể xử lý bất kỳ yêu cầu nào.                          |
| Các nhà phát triển phải quản lý việc xử lý phiên và các vấn đề liên quan.        | Các nhà phát triển có thể tập trung chủ yếu vào logic nghiệp vụ mà không cần lo lắng về vấn đề phiên làm việc. |
## Benefits of Stateful Architecture

Kiến trúc trạng thái (stateful architecture) mang lại một số lợi thế khi các ứng dụng cần duy trì phiên người dùng và ngữ cảnh xuyên suốt nhiều yêu cầu.

- ***Session Persistence:*** Duy trì phiên người dùng, cho phép chuyển đổi mượt mà giữa các bước hoặc thiết bị.
- ***Efficient Resource Use:*** Lưu trữ dữ liệu phiên trên máy chủ, giảm thiểu việc truyền tải và xử lý lặp đi lặp lại.
- ***Personalization:*** Sử dụng các tương tác trong quá khứ để cung cấp trải nghiệm phù hợp, chẳng hạn như các đề xuất.
- ***Enhanced Security:*** Quản lý phiên tập trung hỗ trợ xác thực và mã hóa mạnh mẽ.

## Benefits of Stateless Architecture

Kiến trúc phi trạng thái mang lại lợi thế về khả năng mở rộng và tính đơn giản vì mỗi yêu cầu được xử lý độc lập.

- ***High Scalability:*** Dễ dàng xử lý số lượng lớn yêu cầu mà không cần quản lý phiên.
- ***Fault Tolerance:*** Mỗi yêu cầu là độc lập, vì vậy lỗi ở một khu vực sẽ không ảnh hưởng đến các khu vực khác.
- ***Simplified Load Balancing:*** Các yêu cầu có thể được phân bổ đều mà không cần phiên cố định.
- ***Better Performance:*** Không phát sinh chi phí xử lý phiên, dẫn đến phản hồi nhanh hơn và độ trễ thấp hơn.

Nguồn: [[Clippings/DE/System Design/Stateful Vs Stateless Architecture|Stateful Vs Stateless Architecture]]
