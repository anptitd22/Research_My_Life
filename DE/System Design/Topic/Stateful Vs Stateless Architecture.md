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

| ***Stateful Architecture***                                          | ***Stateless Architecture***                                            |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Scaling requires synchronization of session data.                    | Horizontal scaling is straightforward.                                  |
| Failure in one server can affect sessions stored on it.              | Failures are isolated, impacting only individual requests.              |
| May experience increased latency due to session management.          | Typically faster response times due to lack of session overhead.        |
| Requires more resources to store and manage session state.           | Uses resources efficiently because no session state is stored.          |
| Caching can be complex due to session-specific data.                 | Caching is simpler since requests are independent.                      |
| Deployment can be complex because session data must be synchronized. | Deployment and maintenance are easier due to stateless nature.          |
| Maintains session context to ensure transaction continuity.          | Transactions are handled independently at the request level.            |
| Load balancing may require session affinity (sticky sessions).       | Load balancing is simpler since any server can handle any request.      |
| Developers must manage session handling and related issues.          | Developers can focus mainly on business logic without session concerns. |
## Benefits of Stateful Architecture

Kiến trúc trạng thái (stateful architecture) mang lại một số lợi thế khi các ứng dụng cần duy trì phiên người dùng và ngữ cảnh xuyên suốt nhiều yêu cầu.

- ***Session Persistence:*** Maintains user sessions, allowing smooth transitions across steps or devices.
- ***Efficient Resource Use:*** Stores session data on the server, reducing repeated transfers and processing.
- ***Personalization:*** Uses past interactions to deliver tailored experiences, like recommendations.
- ***Enhanced Security:*** Centralized session management supports strong authentication and encryption.

## Benefits of Stateless Architecture

Kiến trúc phi trạng thái mang lại lợi thế về khả năng mở rộng và tính đơn giản vì mỗi yêu cầu được xử lý độc lập.

- ***High Scalability:*** Easily handles large numbers of requests without session management.
- ***Fault Tolerance:*** Each request is independent, so failures in one area don’t affect others.
- ***Simplified Load Balancing:*** Requests can be evenly distributed without sticky sessions.
- ***Better Performance:*** No session overhead, resulting in faster responses and lower latency.

Nguồn: [[Clippings/DE/System Design/Stateful Vs Stateless Architecture|Stateful Vs Stateless Architecture]]
