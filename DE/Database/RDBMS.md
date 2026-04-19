## Overview

![alt][images/Database/DBMS/introduction.png]

Component:

1. **Trình quản lý giao tiếp máy khách (Client Communications Manager):** Thành phần này chịu trách nhiệm thiết lập và ghi nhớ trạng thái kết nối cho phía gọi (có thể là máy khách hoặc máy chủ phần mềm trung gian). Nó phản hồi các lệnh SQL từ phía gọi, đồng thời trả về kết quả dữ liệu cũng như các thông báo điều khiển (như mã kết quả, thông báo lỗi)
2. **Trình quản lý tiến trình (Process Manager):** Có nhiệm vụ gán một "luồng tính toán" (thread of computation) cho các lệnh SQL và đảm bảo luồng dữ liệu/điều khiển được kết nối với máy khách. Nhiệm vụ quan trọng nhất của thành phần này là **kiểm soát tiếp nhận (admission control)** – quyết định xem hệ thống nên bắt đầu xử lý truy vấn ngay lập tức hay trì hoãn cho đến khi có đủ tài nguyên.
3. **Bộ xử lý truy vấn quan hệ (Relational Query Processor):** Đảm nhiệm việc kiểm tra quyền của người dùng đối với truy vấn và biên dịch văn bản SQL thành một "kế hoạch truy vấn" (query plan) nội bộ. Kế hoạch này sau đó được bộ thực thi kế hoạch xử lý thông qua một tập hợp các toán tử thực hiện các phép toán quan hệ như kết nối (joins), chọn lọc (selection), phép chiếu (projection), và tính toán tổng hợp (aggregation).
4. **Trình quản lý lưu trữ giao dịch (Transactional Storage Manager):** Quản lý mọi yêu cầu truy cập (đọc) và thao tác dữ liệu (tạo, cập nhật, xóa). Nó bao gồm các thuật toán và cấu trúc dữ liệu để tổ chức/truy cập dữ liệu trên đĩa (access methods), quản lý bộ đệm, đồng thời tương tác với bộ quản lý khóa (lock manager) và quản lý nhật ký (log manager) để đảm bảo tính nhất quán (các thuộc tính ACID) cho các giao dịch.
5. **Các thành phần và tiện ích dùng chung (Shared Components and Utilities):** Bao gồm các mô-đun quan trọng hỗ trợ toàn bộ hệ thống như: trình quản lý danh mục (catalog manager) dùng để xác thực, phân tích và tối ưu hóa truy vấn; trình quản lý bộ nhớ (memory manager) dùng để cấp phát/thu hồi bộ nhớ động. Nhóm này cũng chứa các tiện ích chạy độc lập giúp cơ sở dữ liệu luôn được tối ưu hóa và hoạt động đáng tin cậy.

For example:

Hãy xem xét một tương tác cơ sở dữ liệu đơn giản nhưng điển hình tại sân bay, trong Nhân viên kiểm soát cửa sẽ nhấp vào biểu mẫu để yêu cầu danh sách hành khách cho một chuyến tàu. chuyến bay. Việc nhấp vào nút này sẽ dẫn đến một giao dịch truy vấn duy nhất hoạt động. Đại khái như sau:

1. Máy tính cá nhân tại cổng sân bay (đóng vai trò là **"máy khách" - client**) gọi một API, từ đó API này sẽ truyền thông qua mạng để thiết lập kết nối với **Trình quản lý Truyền thông Máy khách** (Client Communications Manager) của một DBMS. Trong một số trường hợp, kết nối này được thiết lập trực tiếp giữa máy khách và máy chủ cơ sở dữ liệu, ví dụ thông qua giao thức kết nối ODBC hoặc JDBC. Mô hình này được gọi là hệ thống **"hai tầng" (two-tier)** hoặc **"máy chủ - máy khách" (client-server)**.Trong các trường hợp khác, máy khách có thể giao tiếp với một **"máy chủ tầng trung gian"** (như máy chủ web, bộ giám sát xử lý giao dịch, hoặc các thành phần tương tự); máy chủ này sẽ sử dụng một giao thức để làm trung gian (proxy) cho việc truyền thông giữa máy khách và DBMS. Mô hình này thường được gọi là hệ thống **"ba tầng" (three-tier)**.Trong nhiều kịch bản dựa trên nền tảng web, thậm chí còn có thêm một tầng **"máy chủ ứng dụng"** (application server) nằm giữa máy chủ web và DBMS, tạo thành kiến trúc **bốn tầng**. Với nhiều lựa chọn như vậy, một DBMS điển hình cần phải tương thích với nhiều giao thức kết nối khác nhau được sử dụng bởi các trình điều khiển máy khách (client drivers) và các hệ thống trung gian (middleware). Tuy nhiên, về cơ bản, trách nhiệm của (Client Communications Manager) trong DBMS đối với tất cả các giao thức này là gần như nhau:
	- - **Thiết lập và lưu giữ** trạng thái kết nối cho bên gọi (dù đó là máy khách hay máy chủ trung gian).
	- **Phản hồi** các câu lệnh SQL từ bên gọi.
	- **Trả về** cả dữ liệu và các thông điệp điều khiển (mã kết quả, thông báo lỗi, v.v.) một cách phù hợp.
	Trong ví dụ đơn giản của chúng ta, trình quản lý truyền thông sẽ xác thực thông tin bảo mật của máy khách, thiết lập trạng thái để ghi nhớ chi tiết của kết nối mới cũng như câu lệnh SQL hiện tại giữa các lần gọi, và chuyển tiếp yêu cầu đầu tiên của máy khách vào sâu hơn trong DBMS để xử lý.
2. Khi nhận được câu lệnh SQL đầu tiên từ máy khách, DBMS phải chỉ định một **"luồng tính toán" (thread of computation)** cho câu lệnh đó. Đồng thời, hệ thống phải đảm bảo rằng các đầu ra dữ liệu và đầu ra điều khiển của luồng này được kết nối với máy khách thông qua trình quản lý truyền thông. Những nhiệm vụ này là công việc của **Trình quản lý Tiến trình DBMS** (DBMS Process Manager). Quyết định quan trọng nhất mà DBMS cần thực hiện ở giai đoạn này của truy vấn liên quan đến **kiểm soát nhập nạp (admission control)**: liệu hệ thống nên bắt đầu xử lý truy vấn ngay lập tức, hay trì hoãn việc thực thi cho đến khi có đủ tài nguyên hệ thống để dành riêng cho truy vấn này. Chúng ta sẽ thảo luận chi tiết về Quản lý Tiến trình trong Chương 2.
3. Một khi đã được chấp nhận và cấp phát một luồng kiểm soát, truy vấn của nhân viên tại cổng sân bay có thể bắt đầu thực thi. Quá trình này được thực hiện bằng cách kích hoạt mã nguồn trong **Bộ xử lý Truy vấn Quan hệ** (Relational Query Processor). Tập hợp các mô-đun này sẽ kiểm tra xem người dùng có được cấp quyền để thực hiện truy vấn đó hay không, sau đó biên dịch văn bản truy vấn SQL của người dùng thành một **kế hoạch truy vấn nội bộ** (internal query plan). Sau khi biên dịch xong, kế hoạch truy vấn kết quả sẽ được xử lý thông qua **bộ thực thi kế hoạch** (plan executor). Bộ thực thi kế hoạch bao gồm một tập hợp các "toán tử" (các thuật toán quan hệ đã được triển khai) để thực thi bất kỳ truy vấn nào. Các toán tử điển hình sẽ thực hiện các nhiệm vụ xử lý truy vấn quan hệ bao gồm: **Joins** (Kết nối), Selection** (Chọn), **Projection** (Chiếu), **Aggregation** (Gom nhóm/Tổng hợp), **Sorting** (Sắp xếp)... và các lời gọi yêu cầu bản ghi dữ liệu từ các lớp thấp hơn của hệ thống. Trong ví dụ về truy vấn của chúng ta, một tập hợp con nhỏ của các toán tử này — được lắp ghép bởi quá trình tối ưu hóa truy vấn — sẽ được triệu hồi để đáp ứng yêu cầu của nhân viên sân bay. Chúng ta sẽ thảo luận chi tiết về bộ xử lý truy vấn trong Chương 4.
4. Tại lớp dưới cùng của kế hoạch truy vấn cho nhân viên tại cổng, có một hoặc nhiều toán tử thực hiện nhiệm vụ yêu cầu dữ liệu từ cơ sở dữ liệu. Các toán tử này sẽ gọi lệnh để lấy dữ liệu từ **Trình quản lý Lưu trữ Giao dịch** của DBMS (phía dưới Hình 1.1), nơi quản lý tất cả các lệnh truy cập dữ liệu (**đọc**) và thao tác dữ liệu (**tạo, cập nhật, xóa**). Hệ thống lưu trữ bao gồm các thuật toán và cấu trúc dữ liệu để tổ chức và truy cập dữ liệu trên đĩa (gọi là các **"phương thức truy cập" - access methods**), bao gồm các cấu trúc cơ bản như bảng (tables) và chỉ mục (indexes). Nó cũng bao gồm một mô-đun **quản lý bộ đệm** (buffer management) để quyết định thời điểm và loại dữ liệu nào cần được chuyển từ đĩa vào bộ nhớ đệm (RAM). Quay lại với ví dụ của chúng ta, trong quá trình truy cập dữ liệu bằng các phương thức truy cập, truy vấn của nhân viên sân bay phải kích hoạt mã nguồn **quản lý giao dịch** để đảm bảo các đặc tính **"ACID"** nổi tiếng của giao dịch (sẽ được thảo luận chi tiết hơn ở Mục 5.1). Trước khi truy cập dữ liệu, các **khóa (locks)** sẽ được lấy từ trình quản lý khóa để đảm bảo việc thực thi diễn ra chính xác trong bối cảnh có các truy vấn khác đang chạy đồng thời. Nếu truy vấn của nhân viên sân bay có bao gồm việc cập nhật cơ sở dữ liệu, nó sẽ tương tác với **trình quản lý nhật ký (log manager)** để đảm bảo rằng giao dịch đó có tính bền vững nếu được xác nhận (commit), hoặc sẽ được hoàn tác hoàn toàn nếu bị hủy bỏ (abort). Trong Chương 5, chúng ta sẽ thảo luận chi tiết hơn về quản lý lưu trữ và bộ đệm; Chương 6 sẽ đề cập đến kiến trúc nhất quán của giao dịch.
5. Tại thời điểm này trong "vòng đời" của truy vấn ví dụ, nó đã bắt đầu truy cập vào các bản ghi dữ liệu và sẵn sàng sử dụng chúng để tính toán kết quả cho máy khách. Quá trình này được thực hiện bằng cách **"tháo gỡ ngăn xếp" (unwinding the stack)** các hoạt động mà chúng ta đã mô tả từ đầu đến giờ. Cụ thể quy trình diễn ra như sau:
	- **Trả quyền điều khiển:** Các phương thức truy cập (access methods) trả lại quyền điều khiển cho các toán tử của bộ thực thi truy vấn. Các toán tử này sẽ điều phối việc tính toán các **bộ kết quả (result tuples)** từ dữ liệu gốc trong cơ sở dữ liệu.
	- **Truyền tải dữ liệu:** Khi các bộ kết quả được tạo ra, chúng được đưa vào một bộ đệm dành cho trình quản lý truyền thông máy khách, nơi sẽ chuyển kết quả ngược lại cho bên gọi.
	- **Xử lý tập dữ liệu lớn:** Đối với các tập kết quả lớn, máy khách thường sẽ thực hiện thêm các lời gọi bổ sung để lấy thêm dữ liệu dần dần (**incrementally**). Điều này dẫn đến nhiều vòng lặp thông qua trình quản lý truyền thông, bộ thực thi truy vấn và trình quản lý lưu trữ.
   **Kết thúc quy trình:** Trong ví dụ đơn giản của chúng ta, khi kết thúc truy vấn, giao dịch sẽ được hoàn tất và kết nối được đóng lại. Điều này dẫn đến một chuỗi các hoạt động dọn dẹp: **Trình quản lý giao dịch:** Giải phóng các trạng thái dữ liệu của giao dịch đó.  **Trình quản lý tiến trình:** Giải phóng bất kỳ cấu trúc điều khiển nào đã cấp cho truy vấn. **Trình quản lý truyền thông:** Dọn dẹp trạng thái truyền thông của kết nối.

## Section 1: **Client Communications Manager**

**Trình quản lý giao tiếp máy khách (Client Communications Manager)** đóng vai trò là cửa ngõ đầu tiên của Hệ quản trị cơ sở dữ liệu (DBMS) để giao tiếp với thế giới bên ngoài. Thành phần này hoạt động chi tiết qua các khía cạnh sau:

**1. Hỗ trợ đa dạng các mô hình kết nối và giao thức**

- **Mô hình hai lớp (Two-tier / Client-server):** Khách hàng hoặc ứng dụng (client) giao tiếp trực tiếp với máy chủ cơ sở dữ liệu thông qua các giao thức kết nối tiêu chuẩn như ODBC hoặc JDBC.
- **Mô hình ba hoặc bốn lớp (Three-tier / Four-tier):** Khách hàng giao tiếp qua một máy chủ trung gian (như web server, application server, hoặc hệ thống giám sát giao dịch), sau đó máy chủ trung gian này đóng vai trò làm proxy để giao tiếp với DBMS. Dù ở mô hình nào, trình quản lý giao tiếp cũng phải tương thích với nhiều giao thức kết nối khác nhau do các driver máy khách và hệ thống phần mềm trung gian sử dụng.

**2. Các nhiệm vụ cốt lõi trong vòng đời kết nối** Bất kể giao thức kết nối là gì, trách nhiệm của Trình quản lý giao tiếp máy khách luôn bao gồm các bước sau:

- **Thiết lập và xác thực:** Khi có yêu cầu kết nối, nó chịu trách nhiệm xác thực danh tính và quyền (security credentials) của máy khách.
- **Quản lý trạng thái:** Thiết lập và ghi nhớ trạng thái kết nối cũng như các chi tiết về lệnh SQL hiện tại xuyên suốt các lần gọi lệnh. Khi truy vấn kết thúc và giao dịch hoàn tất, thành phần này cũng có nhiệm vụ dọn dẹp trạng thái giao tiếp cho kết nối đó.
- **Chuyển tiếp yêu cầu:** Nhận các lệnh SQL từ máy khách (hoặc máy chủ trung gian) và chuyển tiếp chúng vào sâu bên trong DBMS (như Process Manager và Query Processor) để xử lý.
- **Trả về kết quả và thông báo:** Khi truy vấn được thực thi xong, các toán tử sẽ đẩy dữ liệu kết quả vào bộ đệm, và trình quản lý giao tiếp sẽ lấy dữ liệu từ đó để gửi về cho phía gọi. Nó cũng chịu trách nhiệm trả về các thông báo điều khiển, mã kết quả và thông báo lỗi.

**3. Quản lý bộ đệm giao tiếp máy khách (Client communication buffers)** SQL thường hoạt động theo mô hình "kéo" (pull model), trong đó máy khách liên tục gửi các yêu cầu FETCH để lấy về từng bản ghi (tuple) hoặc các nhóm nhỏ bản ghi.

- Để tăng tốc độ trả kết quả, DBMS thường cố gắng xử lý vượt lên trước luồng yêu cầu FETCH này để **tải trước (prefetch)** kết quả.
- Trình quản lý giao tiếp có thể sử dụng các socket giao tiếp của máy khách như một hàng đợi (queue) để lưu trữ sẵn các bản ghi vừa được tạo ra, giúp dữ liệu sẵn sàng ngay khi máy khách có yêu cầu lấy thêm.

## Section 2: process manager

**Trình quản lý tiến trình (Process Manager)** hoạt động như một "nhạc trưởng" phân bổ tài nguyên xử lý ngay khi nhận được lệnh SQL từ máy khách. Nó có nhiệm vụ gán một "luồng tính toán" (thread of computation) cho mỗi lệnh, đảm bảo dữ liệu và luồng điều khiển được kết nối chính xác với máy khách thông qua trình quản lý giao tiếp. Ngoài ra, nó cũng là chốt chặn đầu tiên thực hiện **kiểm soát tiếp nhận (admission control)** để quyết định xem lệnh có nên được xử lý ngay hay phải chờ.

Chi tiết hơn, Process Manager hoạt động xoay quanh việc quản lý và ánh xạ các **DBMS worker** (thực thể xử lý các yêu cầu SQL thay mặt cho một máy khách) vào các tiến trình hoặc luồng của hệ điều hành (OS). Có 3 mô hình kiến trúc cốt lõi mà Process Manager sử dụng để xử lý các yêu cầu đồng thời:

**1. Mô hình Tiến trình cho mỗi DBMS Worker (Process per DBMS worker)**

- Mỗi DBMS worker được ánh xạ trực tiếp thành một tiến trình độc lập của hệ điều hành.
- **Ưu điểm:** Mô hình này dễ lập trình, các tiến trình được bảo vệ lẫn nhau tránh lỗi ghi đè bộ nhớ, và tận dụng tốt các công cụ gỡ lỗi của hệ điều hành.
- **Nhược điểm:** Tiêu tốn rất nhiều bộ nhớ khi có lượng lớn người dùng kết nối (do tiến trình chứa nhiều trạng thái ngữ cảnh hơn luồng). Hơn nữa, nó đòi hỏi cấu trúc dữ liệu dùng chung (như vùng đệm, bảng khóa) phải được thiết lập trên bộ nhớ dùng chung (shared memory) của hệ điều hành.
- Được sử dụng bởi: PostgreSQL, IBM DB2 (trên các OS thiếu hỗ trợ đa luồng tốt), và cấu hình mặc định của Oracle,.

**2. Mô hình Luồng cho mỗi DBMS Worker (Thread per DBMS worker)**

- Toàn bộ hoạt động của các worker được gói gọn trong một tiến trình đa luồng (multi-threaded process) duy nhất. Một hoặc vài luồng điều phối (dispatcher) sẽ lắng nghe kết nối mới và cấp phát một luồng riêng cho mỗi máy khách.
- **Ưu điểm:** Mở rộng rất tốt với số lượng kết nối lớn, chi phí chuyển đổi ngữ cảnh (context switch) thấp, và dễ dàng chia sẻ các cấu trúc dữ liệu chung trong cùng một không gian bộ nhớ,,.
- **Các biến thể:** Để tránh rủi ro về hiệu năng luồng của một số hệ điều hành, nhiều hệ thống (như Sybase, Informix, Microsoft SQL Server) đã tự xây dựng **các luồng trọng lượng nhẹ (lightweight DBMS threads)** tự lên lịch ngay trong không gian người dùng (user-space) thay vì dùng luồng của OS,,,. Các hệ thống như MySQL hay IBM DB2 thì trực tiếp dùng luồng của hệ điều hành.

**3. Mô hình Nhóm tiến trình / Nhóm luồng (Process / Thread Pool)**

- Thay vì phân bổ riêng lẻ, các DBMS worker được ghép kênh (multiplexed) thông qua một nhóm (pool) các tiến trình hoặc luồng có kích thước giới hạn,. Một tiến trình trung tâm giữ các kết nối, khi có yêu cầu tới, nó sẽ ném cho một tiến trình/luồng đang rảnh trong nhóm. Xử lý xong, tiến trình/luồng đó được trả lại nhóm để phục vụ yêu cầu tiếp theo,.
- **Ưu điểm:** Kế thừa sự đơn giản nhưng giải quyết được bài toán hao phí bộ nhớ, mở rộng xuất sắc cho hàng ngàn kết nối đồng thời. Kích thước nhóm có thể co giãn linh hoạt dựa trên tải hệ thống,.
- Được sử dụng bởi: Tùy chọn mở rộng của Oracle (process pool) và thiết lập mặc định của Microsoft SQL Server (thread pool),.

**Tổ chức dữ liệu và bộ nhớ chia sẻ** Dù sử dụng mô hình nào, các tiến trình/luồng không bao giờ hoạt động độc lập hoàn toàn vì chúng cùng thao tác trên một cơ sở dữ liệu. Process Manager phải điều phối việc dùng chung các cấu trúc dữ liệu quan trọng:

- **Bộ đệm I/O ổ đĩa (Disk I/O buffers):** Bao gồm vùng đệm (buffer pool) chứa các trang dữ liệu và phần đuôi nhật ký (log tail) chứa các bản ghi log đang chờ được ghi xuống đĩa.
- **Bảng khóa (Lock table):** Dùng chung cho tất cả các worker để đảm bảo tính nhất quán và quản lý các khóa giao dịch.
- **Bộ đệm giao tiếp máy khách (Client communication buffers):** Dùng để hàng đợi trước (prefetch) kết quả truy vấn vào bộ đệm mạng hoặc bộ đệm ứng dụng, giúp tăng tốc độ trả về.

**Kiểm soát tiếp nhận (admission control)** trong Hệ quản trị cơ sở dữ liệu (DBMS) là cơ chế quyết định xem hệ thống nên bắt đầu xử lý một truy vấn ngay lập tức hay phải trì hoãn cho đến khi có đủ tài nguyên. Mục tiêu cốt lõi của quá trình này là ngăn chặn hệ thống bị sụt giảm hiệu suất nghiêm trọng (thrashing) khi phải xử lý khối lượng công việc tăng cao, thường là do thiếu hụt bộ nhớ để duy trì dữ liệu trong bộ đệm hoặc do các giao dịch liên tục bế tắc (deadlock) khi cạnh tranh khóa.

Khi có bộ kiểm soát tiếp nhận tốt, hệ thống sẽ **suy thoái một cách nhẹ nhàng (graceful degradation)** khi bị quá tải: thời gian trễ của mỗi giao dịch có thể tăng lên tỷ lệ thuận với số lượng yêu cầu gửi đến, nhưng **tổng lượng công việc hệ thống xử lý được (throughput) vẫn duy trì ở mức tối đa**.

Cơ chế kiểm soát tiếp nhận thường được triển khai qua **hai tầng (two tiers)** như sau:

**1. Tầng thứ nhất (Tại quá trình điều phối - Dispatcher):** Hệ thống áp dụng một chính sách kiểm soát đơn giản nhằm đảm bảo **số lượng kết nối của máy khách (client connections) được giữ ở dưới một ngưỡng nhất định**. Điều này giúp ngăn ngừa việc tiêu thụ quá mức các tài nguyên cơ bản như kết nối mạng. Ở một số hệ thống, tầng kiểm soát này bị bỏ qua nếu họ giả định rằng việc giới hạn kết nối đã được xử lý bởi các hệ thống trung gian (như application server hoặc web server).

**2. Tầng thứ hai (Bên trong bộ xử lý truy vấn quan hệ):** Đây là bộ kiểm soát tiếp nhận thực thi (execution admission controller), hoạt động sau khi truy vấn đã được phân tích cú pháp và tối ưu hóa. Tầng này sẽ đưa ra quyết định xem **truy vấn nên bị hoãn lại, được bắt đầu thực thi nhưng cấp ít tài nguyên hơn, hay được thực thi ngay mà không có ràng buộc nào**.

Để đưa ra quyết định ở tầng thứ hai, bộ kiểm soát tiếp nhận phụ thuộc chặt chẽ vào thông tin ước lượng từ **trình tối ưu hóa truy vấn (query optimizer)**, bao gồm:

- **Chi phí lưu trữ (Disk I/O):** Các thiết bị đĩa nào sẽ bị truy cập, cùng với ước tính số lượng thao tác đọc/ghi ngẫu nhiên và tuần tự trên mỗi thiết bị.
- **Tải CPU (CPU load):** Ước tính mức tiêu thụ CPU dựa trên các toán tử trong kế hoạch truy vấn và số lượng bản ghi cần xử lý.
- **Không gian bộ nhớ (Memory footprint):** **Đây là thông số quan trọng nhất**, bao gồm yêu cầu bộ nhớ cho các cấu trúc dữ liệu của truy vấn, đặc biệt là không gian để sắp xếp (sorting) và băm (hashing) các đầu vào lớn.

Vì áp lực bộ nhớ (memory pressure) là nguyên nhân chính gây ra tình trạng thrashing, **nhiều hệ thống DBMS sử dụng mức tiêu thụ bộ nhớ và số lượng luồng thực thi (DBMS workers) đang hoạt động làm tiêu chí cốt lõi để quyết định tiếp nhận**. Khi đã quyết định tiếp nhận, chính sách này cũng giúp hệ thống có thể đảm bảo cung cấp chính xác phần RAM đã hứa cho các thao tác phức tạp của truy vấn đó trong suốt quá trình chạy.

## Section 3: Relational Query Processor

Bộ xử lý truy vấn quan hệ (Relational Query Processor) có nhiệm vụ nhận một câu lệnh SQL khai báo, xác thực, tối ưu hóa thành một kế hoạch thực thi dạng luồng dữ liệu (dataflow) và thực thi nó. Hoạt động của bộ xử lý này trải qua 4 giai đoạn chính sau đây:

**1. Phân tích cú pháp và cấp quyền (Query Parsing and Authorization)** Nhiệm vụ đầu tiên là đảm bảo truy vấn được viết đúng và người dùng có quyền chạy nó:

- **Phân tích và chuẩn hóa tên:** Các tên bảng trong truy vấn được chuẩn hóa thành tên đầy đủ (thường gồm 4 phần: server.database.schema.table). Hệ thống cũng gọi trình quản lý danh mục (catalog manager) để lấy siêu dữ liệu (metadata) nhằm đảm bảo các tham chiếu cột là chính xác và xác định kiểu dữ liệu để giải quyết các hàm và toán tử.
- **Kiểm tra quyền (Authorization):** Hệ thống kiểm tra xem người dùng có quyền thực thi (SELECT/DELETE/INSERT/UPDATE) trên các bảng hoặc đối tượng hay không. Một số hệ thống sẽ trì hoãn việc kiểm tra bảo mật chi tiết (như bảo mật cấp hàng) cho đến giai đoạn thực thi để có thể chia sẻ kế hoạch truy vấn giữa các người dùng mà không cần biên dịch lại.

**2. Viết lại truy vấn (Query Rewrite)** Mô-đun này làm nhiệm vụ đơn giản hóa và chuẩn hóa truy vấn thông qua các phép biến đổi logic mà không làm thay đổi ngữ nghĩa của nó:

- **Khai triển View (View expansion):** Hệ thống thay thế các tham chiếu view bằng các bảng và điều kiện thực tế cấu thành nên view đó.
- **Biến đổi logic điều kiện (Logical rewriting):** Tối ưu hóa các biểu thức hằng số và áp dụng logic Boolean. Ví dụ: biểu thức `NOT Emp.Salary > 1000000` sẽ được đổi thành `Emp.Salary <= 1000000`, hoặc hệ thống có thể phát hiện các điều kiện sai hiển nhiên (như `x < 75000 AND x > 1000000`) để bỏ qua truy vấn ngay lập tức mà không cần truy xuất cơ sở dữ liệu,.
- **Tối ưu hóa ngữ nghĩa và làm phẳng truy vấn con (Semantic optimization & Subquery flattening):** Hệ thống có thể loại bỏ các phép kết nối (join) thừa nếu phát hiện có ràng buộc khóa ngoại (foreign key) đảm bảo dữ liệu luôn tồn tại,. Nó cũng có xu hướng "làm phẳng" các truy vấn con lồng nhau để tối ưu hóa tốt hơn trong một khối duy nhất.

**3. Tối ưu hóa truy vấn (Query Optimizer)** Trình tối ưu hóa biến đổi cấu trúc nội bộ của truy vấn thành một **kế hoạch truy vấn (query plan)** hiệu quả. Kế hoạch này giống như một đồ thị luồng dữ liệu liên kết các toán tử lại với nhau.

- Nó sử dụng các dữ liệu thống kê (như biểu đồ histogram hoặc lấy mẫu) để ước lượng số lượng bản ghi (selectivity estimation) sẽ trả về sau mỗi bước.
- Hệ thống khám phá không gian rộng lớn của các kế hoạch (bao gồm nhiều kiểu cây thực thi hoặc cấu trúc kết nối khác nhau) thông qua các thuật toán tìm kiếm như quy hoạch động (dynamic programming) hoặc tìm kiếm từ trên xuống (top-down),. Kế hoạch tối ưu thường được biên dịch thành một cấu trúc dữ liệu để diễn dịch hoặc mã op-codes (tương tự như Java byte codes).
- Để giảm thiểu thời gian tối ưu hóa trong các lần sau, các kế hoạch này thường được lưu trong bộ nhớ đệm (query plan cache) để tái sử dụng ngay lập tức nếu một câu lệnh tương tự được gọi.

**4. Thực thi truy vấn (Query Executor)** Trình thực thi nhận kế hoạch truy vấn dưới dạng một đồ thị luồng dữ liệu và thường hoạt động theo **mô hình vòng lặp (iterator model)**:

- Mỗi toán tử trong kế hoạch truy vấn (như quét file, quét chỉ mục, sort, hash-join) là một lớp con của đối tượng iterator, cung cấp hàm `get_next()`. Hàm này liên tục lấy từng dữ liệu trả về cho toán tử cha, giúp hệ thống không cần tạo nhiều luồng (single-threaded) mà vẫn đạt hiệu quả rất cao do kết hợp chặt chẽ giữa luồng dữ liệu và luồng điều khiển.
- Ở cuối cùng của biểu đồ, trình thực thi sẽ tương tác với các **Phương thức truy cập (Access Methods)** để đọc dữ liệu đĩa thông qua bộ đệm. Để tăng hiệu năng, các đối số tìm kiếm (SARGs) được đẩy sâu xuống lớp phương thức truy cập, giúp bộ máy chỉ đưa các bản ghi thực sự thỏa mãn điều kiện lên lớp trên, tránh lãng phí vòng lặp CPU cho việc kiểm tra thừa thãi,.
- Đối với các lệnh sửa đổi dữ liệu (INSERT, UPDATE, DELETE), trình thực thi còn phải bổ sung thêm các toán tử nhằm tránh lỗi "Halloween problem" (ví dụ: cập nhật cùng một bản ghi nhiều lần khi nó di chuyển trong cấu trúc cây do chính truy vấn đang cập nhật gây ra) bằng cách vật lý hóa ID của bản ghi (Record-ID materialization) trước khi thực thi cập nhật thực sự,.

## Section 4: Transactional Storage Manager

Trình quản lý lưu trữ giao dịch (Transactional Storage Manager) là một trong những thành phần đồ sộ và phức tạp nhất của Hệ quản trị cơ sở dữ liệu (DBMS). Đây là một hệ thống nguyên khối (monolithic) bao gồm **4 thành phần chính đan xen chặt chẽ với nhau** nhằm đảm bảo khả năng đọc/ghi dữ liệu trên đĩa và duy trì các thuộc tính ACID của giao dịch:

**1. Trình quản lý khóa (Lock Manager) - Đảm bảo tính cô lập và kiểm soát đồng thời:**

- Nhiệm vụ chính là đảm bảo việc thực thi các giao dịch đồng thời mà không làm sai lệch dữ liệu.
- Hầu hết các hệ thống DBMS thương mại sử dụng cơ chế **Khóa hai giai đoạn nghiêm ngặt (Strict Two-Phase Locking - 2PL)**, trong đó giao dịch sẽ lấy khóa chia sẻ (shared lock) khi đọc và khóa độc quyền (exclusive lock) khi ghi, và giữ toàn bộ khóa cho đến khi giao dịch kết thúc.
- Trình quản lý khóa duy trì một **bảng khóa (lock table)** trong bộ nhớ và một **bảng giao dịch (transaction table)** để theo dõi trạng thái khóa của từng giao dịch. Nó cũng sử dụng một tiến trình phát hiện bế tắc (deadlock detector) để tìm và tự động hủy các giao dịch bị bế tắc.
- Ngoài ra, nó còn sử dụng các **chốt (latches)** – nhẹ hơn khóa – để bảo vệ quyền truy cập độc quyền cho các cấu trúc dữ liệu nội bộ của DBMS trong thời gian cực ngắn (ví dụ: bảo vệ bảng trang của bộ đệm).

**2. Trình quản lý nhật ký (Log Manager) - Đảm bảo tính nguyên tử và bền vững:**

- Chịu trách nhiệm duy trì, ghi nhận các thay đổi để phục vụ việc khôi phục hệ thống khi xảy ra sự cố (crash recovery) hoặc hoàn tác (rollback) khi giao dịch bị hủy bỏ.
- Trình quản lý này tuân thủ nghiêm ngặt **giao thức Ghi nhật ký trước (Write-Ahead Logging - WAL)** với 3 quy tắc cốt lõi: (1) Mọi thay đổi dữ liệu phải tạo ra bản ghi nhật ký và đẩy xuống đĩa trước khi trang dữ liệu đó được lưu, (2) Các bản ghi nhật ký phải được ghi theo thứ tự, và (3) Giao dịch chỉ được phản hồi thành công khi bản ghi "commit" đã được lưu an toàn vào đĩa.
- Hầu hết các hệ thống thiết lập chế độ STEAL/NOT-FORCE, cho phép tối ưu hoá việc đọc/ghi nhưng đòi hỏi bộ quản lý phải áp dụng quy trình khôi phục phức tạp dựa trên điểm LSN (Log Sequence Number) và các trạm kiểm soát mờ (fuzzy checkpoints).

**3. Trình quản lý vùng đệm (Buffer Manager) - Xử lý I/O không gian và thời gian:**

- Dữ liệu từ đĩa được lưu tạm trên một vùng bộ nhớ lớn chia sẻ chung gọi là **vùng đệm (buffer pool)**.
- Vùng đệm chứa một bảng băm quản lý siêu dữ liệu của từng trang (page) bao gồm: **cờ bẩn (dirty bit)** (đánh dấu trang đã bị sửa đổi) và **bộ đếm ghim (pin count)** (ngăn không cho hệ thống thay thế trang khi nó đang được sử dụng).
- Thay vì dựa vào hệ điều hành, DBMS tự quyết định thời điểm và vị trí lưu trữ dữ liệu để tránh tình trạng "đệm kép" (double buffering) gây lãng phí tài nguyên CPU và bộ nhớ. Nó cũng sử dụng các thuật toán thay thế trang chuyên biệt như LRU-2 thay vì LRU tiêu chuẩn, giúp xử lý tốt đặc thù quét toàn bảng của cơ sở dữ liệu.

**4. Các phương thức truy cập (Access Methods) - Tổ chức dữ liệu vật lý:**

- Bao gồm các thuật toán và cấu trúc tổ chức dữ liệu như tệp tin không có thứ tự (heap files) và các loại chỉ mục (như B+-tree, hash indexes).
- Phương thức truy cập cung cấp một API có khả năng tiếp nhận trực tiếp các **đối số tìm kiếm (SARGs)** để lọc bản ghi ngay ở cấp lưu trữ đĩa, tránh việc đưa dữ liệu thừa lên lớp xử lý truy vấn.
- Các phương thức truy cập kết hợp sâu sắc với cơ chế khóa và nhật ký. Ví dụ, trên cây B+, hệ thống sử dụng các kỹ thuật "chốt khóa B+-tree" tinh vi (như right-link scheme) để cho phép nhiều giao dịch thao tác cấu trúc cây cùng lúc mà không gây xung đột.

**Sự phụ thuộc lẫn nhau (Interdependencies):** Một điểm đặc trưng của Trình quản lý lưu trữ giao dịch là 4 thành phần trên không hoạt động độc lập mà **liên kết chặt chẽ và phụ thuộc sâu sắc vào nhau**. Chẳng hạn, giao thức WAL của trình quản lý nhật ký ngầm định yêu cầu trình quản lý khóa phải dùng giao thức 2PL nghiêm ngặt để đảm bảo an toàn khi hoàn tác. Tương tự, các lệnh thay đổi cấu trúc cây của phương thức truy cập đòi hỏi bộ nhật ký và bộ khóa phải có mã nguồn tuỳ chỉnh phức tạp (như "next-key locking" để chống lại lỗi phantom). Bộ quản lý vùng đệm là bộ phận hiếm hoi duy trì được tính độc lập cục bộ, miễn là các thành phần khác quản lý tốt các bộ đếm ghim (pin).

## Section 5: Shared Components and Utilities

Trong kiến trúc của Hệ quản trị cơ sở dữ liệu (DBMS), nhóm **Các thành phần và tiện ích dùng chung (Shared Components and Utilities)** bao gồm các mô-đun hoạt động độc lập hoặc hỗ trợ xuyên suốt cho các thành phần khác, giúp hệ thống hoạt động ổn định, tối ưu và dễ quản trị. Chi tiết hoạt động của các thành phần này được chia thành 5 mảng chính như sau:

**1. Trình quản lý danh mục (Catalog Manager)** Thành phần này lưu trữ **siêu dữ liệu (metadata)** về cấu trúc của hệ thống, bao gồm tên người dùng, lược đồ, bảng, cột, chỉ mục và các mối quan hệ.

- Để đơn giản hóa, siêu dữ liệu này được lưu trữ dưới dạng các bảng cơ sở dữ liệu thông thường, cho phép người dùng dùng chung một ngôn ngữ (SQL) để truy vấn.
- Tuy nhiên, vì trình phân tích cú pháp và tối ưu hóa truy vấn gọi đến danh mục rất thường xuyên, các dữ liệu danh mục có lưu lượng truy cập cao sẽ được lưu tạm (cache) vào bộ nhớ chính dưới dạng cấu trúc mạng lưới đối tượng phi chuẩn hóa (denormalized) để truy xuất cực nhanh.

**2. Trình cấp phát bộ nhớ (Memory Allocator)** Khác với bộ đệm (buffer pool) dùng để chứa dữ liệu đĩa, DBMS cần cấp phát một lượng lớn bộ nhớ động cho việc tối ưu hóa truy vấn và thực thi các phép toán như nối băm (hash joins) hay sắp xếp (sorts). Các DBMS thương mại xử lý việc này thông qua **trình cấp phát bộ nhớ dựa trên ngữ cảnh (context-based memory allocator)**.

- **Ngữ cảnh bộ nhớ (Memory Context)** là một cấu trúc duy trì danh sách các vùng nhớ liên tiếp. API của nó cho phép: tạo mới một ngữ cảnh, cấp phát một phần bộ nhớ trong ngữ cảnh đó, hoặc xóa/đặt lại toàn bộ ngữ cảnh cùng lúc.
- **Lợi ích:** Cơ chế này giúp lập trình viên quản lý bộ nhớ linh hoạt hơn bộ thu gom rác (garbage collection) tự động vì họ có thể kiểm soát chính xác không gian và thời gian giải phóng bộ nhớ. Ví dụ, sau khi trình tối ưu hóa chọn xong kế hoạch truy vấn, nó chỉ cần xóa toàn bộ ngữ cảnh bộ nhớ của phiên tối ưu đó trong một thao tác duy nhất, giúp tránh rò rỉ bộ nhớ (memory leaks) và giảm thiểu chi phí hiệu năng khi gọi hàm `malloc()`/`free()` liên tục từ hệ điều hành.

**3. Hệ thống con quản lý đĩa (Disk Management Subsystems)** Thành phần này quản lý việc ánh xạ các bảng dữ liệu vào các thiết bị vật lý, phân vùng logic hoặc tệp tin.

- Nó giải quyết các hạn chế của hệ thống tệp hệ điều hành (như giới hạn kích thước tệp hoặc số lượng tệp được mở) bằng cách phân bổ một bảng qua nhiều tệp, hoặc gộp nhiều bảng vào một tệp.
- Hệ thống này phải xử lý độ phức tạp khi DBMS chạy trên các thiết bị lưu trữ hiện đại như Mạng lưu trữ (SAN) hay RAID. Các thiết bị này thường giả lập giao diện đĩa đơn giản nhưng có đặc tính hiệu năng rất phức tạp. Ví dụ, cấu hình RAID 5 tối ưu không gian nhưng lại có hiệu năng ghi rất kém (write penalty) so với RAID 1 (mirroring). DBMS phải được tinh chỉnh để tương tác tốt với những đặc thù phần cứng này.

**4. Dịch vụ sao chép (Replication Services)** Được sử dụng để tạo các bản sao lưu dự phòng (warm standby) hoặc phân phối dữ liệu ở các khu vực địa lý khác nhau. Có 3 phương pháp sao chép chính:

- **Sao chép vật lý (Physical Replication):** Sao chép toàn bộ cơ sở dữ liệu định kỳ, nhưng không mở rộng tốt với dữ liệu lớn.
- **Dựa trên Trigger (Trigger-Based Replication):** Dùng trigger để ghi lại các thay đổi vào một bảng riêng và gửi đi. Cách này gây suy giảm hiệu năng nghiêm trọng cho hệ thống đang chạy.
- **Dựa trên nhật ký (Log-Based Replication):** Đây là phương pháp tối ưu và được các DBMS lớn sử dụng. Một tiến trình (log sniffer) sẽ chặn các bản ghi nhật ký (log) và gửi đến hệ thống đích để "phát lại" (replay). Phương pháp này tốn rất ít tài nguyên, mở rộng tốt, liên tục cập nhật tăng dần và đảm bảo tính nhất quán của giao dịch.

**5. Quản trị, Giám sát và Tiện ích (Administration, Monitoring, and Utilities)** Đây là nhóm công cụ quyết định tính dễ quản lý của hệ thống. Đặc điểm quan trọng nhất của các công cụ hiện đại là chúng phải chạy được **trực tuyến (online)** – tức là hoạt động song song khi các truy vấn của người dùng vẫn đang diễn ra, đáp ứng nhu cầu hoạt động 24/7. Chúng bao gồm:

- **Thu thập thống kê (Optimizer Statistics Gathering):** Quét các bảng để xây dựng dữ liệu thống kê (như biểu đồ phân phối) giúp trình tối ưu hóa lập kế hoạch tốt hơn.
- **Tổ chức lại vật lý (Physical Reorganization):** Tái cấu trúc lại bảng hoặc chỉ mục bị phân mảnh trong nền (background) mà không cần khóa toàn bộ hệ thống (holding locks).
- **Sao lưu/Xuất dữ liệu (Backup/Export):** Thực hiện sao lưu mờ (fuzzy dump) kết hợp với logic nhật ký để đảm bảo tính nhất quán mà không phải dừng hệ thống.
- **Tải dữ liệu hàng loạt (Bulk Load):** Tiện ích để chèn một lượng dữ liệu khổng lồ vào hệ thống ở tốc độ cao bằng cách bỏ qua các bước xử lý SQL thông thường.
- **Giám sát và kiểm soát tài nguyên (Monitoring & Resource Governors):** Cung cấp các bảng ảo để theo dõi hiệu năng (bộ nhớ, khóa, thời gian chạy). Các công cụ kiểm soát có thể tự động cảnh báo hoặc thậm chí hủy (abort) các truy vấn tiêu tốn quá nhiều tài nguyên để bảo vệ toàn hệ thống.