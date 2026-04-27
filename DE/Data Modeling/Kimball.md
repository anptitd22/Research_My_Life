# Chương 1: Data Warehousing, Business Intelligence, and Dimensional Modeling Primer

Chương 1 của tài liệu đóng vai trò là nền tảng, trình bày bức tranh toàn cảnh về mục tiêu, các khái niệm cốt lõi của mô hình chiều, các kiến trúc kho dữ liệu và đính chính các lầm tưởng phổ biến. Dưới đây là chi tiết các vấn đề và khái niệm được đề cập:

**1. Mục tiêu của Kho dữ liệu và Trí tuệ doanh nghiệp (DW/BI)** Hệ thống DW/BI có bản chất và nhu cầu khác biệt hoàn toàn so với các hệ thống tác nghiệp (operational systems) xử lý giao dịch hàng ngày. Các mục tiêu cốt lõi của một hệ thống DW/BI bao gồm:

- **Dễ dàng tiếp cận (Simple and fast):** Dữ liệu phải trực quan, dễ hiểu đối với người dùng kinh doanh và các truy vấn phải trả về kết quả nhanh chóng.
- **Trình bày nhất quán:** Dữ liệu phải đáng tin cậy, được làm sạch và sử dụng chung một nhãn/định nghĩa trên toàn doanh nghiệp.
- **Thích ứng với thay đổi:** Hệ thống phải được thiết kế để xử lý những thay đổi về dữ liệu và nhu cầu một cách linh hoạt mà không làm gián đoạn các ứng dụng hiện có.
- **Kịp thời và bảo mật:** Phải cung cấp thông tin trong thời gian phù hợp và bảo vệ thông tin tuyệt mật của tổ chức.
- **Hỗ trợ ra quyết định:** Là nền tảng đáng tin cậy để đưa ra các quyết định tạo ra giá trị cho doanh nghiệp.
- **Sự chấp nhận của người dùng:** Đây là thước đo thành công cuối cùng của hệ thống.

**2. Ẩn dụ về Quản lý DW/BI**

- **Ẩn dụ về xuất bản (Publishing Metaphor):** Người quản lý DW/BI giống như một tổng biên tập tạp chí. Thay vì chỉ tập trung vào công nghệ bên trong, họ phải tập trung ra bên ngoài để hiểu độc giả (người dùng kinh doanh), xuất bản thông tin chất lượng cao, đáng tin cậy và liên tục điều chỉnh để đáp ứng nhu cầu ra quyết định.
- **Ẩn dụ về nhà hàng (Restaurant Metaphor):** Như đã trình bày ở trên, hệ thống ETL là "nhà bếp" nơi chuẩn bị dữ liệu thô, còn khu vực trình bày và ứng dụng BI là "phòng ăn" phục vụ thực khách với thực đơn dễ hiểu và chất lượng.

**3. Các khái niệm cốt lõi trong Mô hình Chiều (Dimensional Modeling)** Mô hình chiều giải quyết hai yêu cầu: dữ liệu dễ hiểu và hiệu suất truy vấn nhanh. Nó có thể triển khai dưới dạng **Lược đồ hình sao (Star schemas)** trong CSDL quan hệ hoặc **Khối đa chiều (OLAP Cubes)**. Hai thành phần chính bao gồm:

- **Bảng sự kiện (Fact Tables):** Chứa các phép đo lường (measurements) của quy trình nghiệp vụ, thường là dữ liệu số. Mỗi dòng trong bảng sự kiện tương ứng với một sự kiện ở mức độ chi tiết cụ thể, gọi là **hạt (grain)**. Hữu ích nhất là các fact có thể cộng gộp toàn phần (additive).
- **Bảng chiều (Dimension Tables):** Cung cấp ngữ cảnh mô tả (ai, cái gì, ở đâu, khi nào, như thế nào) cho các sự kiện. Đây là nguồn cho các thao tác lọc (filtering), nhóm (grouping) và dán nhãn báo cáo. Các bảng chiều thường được phi chuẩn hóa (denormalized/flattened) để tối ưu hiệu suất, tránh việc phân mảnh dữ liệu thành nhiều bảng nhỏ (được gọi là **snowflaking**). Khi bảng sự kiện và bảng chiều kết nối với nhau, chúng tạo ra một cấu trúc đơn giản, đối xứng, linh hoạt với sự thay đổi và cho phép "Drill Down" (đào sâu dữ liệu) vô hạn nếu bảng sự kiện chứa dữ liệu ở mức độ chi tiết nhất (atomic data).

**4. Các Kiến trúc DW/BI** Tài liệu phân tích kiến trúc của Kimball (như đã giải thích) và so sánh với 3 mô hình thay thế:

- **Independent Data Mart (Data Mart Độc lập):** Các bộ phận tự xây dựng kho dữ liệu riêng rẽ. Cách này tạo ra các "ốc đảo" dữ liệu không đồng nhất, gây tranh cãi về tính chính xác và không được khuyên dùng.
- **Hub-and-Spoke (Corporate Information Factory - Inmon):** Dữ liệu được tải vào một Kho dữ liệu Doanh nghiệp (EDW) được chuẩn hóa (3NF) ở mức chi tiết. Sau đó, dữ liệu mới được cung cấp tới các Data Mart theo mô hình chiều hoặc tổng hợp để người dùng truy vấn.
- **Hybrid (Lai):** Kết hợp EDW chuẩn hóa 3NF của Inmon để lưu trữ, nhưng phần trình bày cho người dùng hoàn toàn sử dụng mô hình chiều của Kimball và tuân thủ Kiến trúc Bus Doanh nghiệp.

**5. 5 Lầm tưởng phổ biến về Mô hình Chiều (Myths)**

- **Lầm tưởng 1: Chỉ dành cho dữ liệu tổng hợp (Summary Data).** Sự thật: Mô hình chiều cần chứa dữ liệu ở mức chi tiết (atomic) để đáp ứng mọi câu hỏi bất ngờ, dữ liệu tổng hợp chỉ dùng để bổ trợ hiệu suất.
- **Lầm tưởng 2: Chỉ dành cho từng bộ phận, không dành cho toàn doanh nghiệp.** Sự thật: Mô hình chiều được tổ chức xung quanh các quy trình nghiệp vụ (business processes) dùng chung cho toàn doanh nghiệp chứ không theo ranh giới phòng ban.
- **Lầm tưởng 3: Không có khả năng mở rộng.** Sự thật: Các mô hình chiều có thể mở rộng rất tốt và chứa các bảng sự kiện với cấu trúc giống hệt nhau ở nhiều cơ sở dữ liệu quy mô lớn.
- **Lầm tưởng 4: Chỉ hỗ trợ các mục đích sử dụng đã biết trước.** Sự thật: Bằng cách lưu trữ dữ liệu ở mức chi tiết nhất, mô hình chiều có tính đối xứng và vô cùng linh hoạt trước các truy vấn ngẫu nhiên chưa được dự đoán.
- **Lầm tưởng 5: Không thể tích hợp.** Sự thật: Dữ liệu hoàn toàn có thể được tích hợp liền mạch nếu tuân thủ chặt chẽ Kiến trúc Bus thông qua việc dùng chung các **Chiều đồng nhất (Conformed Dimensions)**.

**6. Triển khai Agile với Mô hình Chiều** Việc xây dựng DW/BI theo phương pháp Agile hoàn toàn phù hợp với tư tưởng của Kimball: tập trung vào giá trị doanh nghiệp, hợp tác chặt chẽ với người dùng và phát triển lặp lại. Bằng cách sử dụng **Ma trận Bus (Bus matrix)** và các **Chiều đồng nhất**, các nhóm phát triển có thể thiết kế theo từng phân hệ nhỏ (từng quy trình nghiệp vụ) một cách nhanh chóng mà vẫn đảm bảo tính gắn kết và tích hợp ở quy mô toàn doanh nghiệp sau này.

# Chương 2: Kimball Dimensional Modeling Techniques Overview

Chương 2 của tài liệu đóng vai trò như một danh sách chính thức, tóm tắt toàn bộ các kỹ thuật và mẫu thiết kế mô hình đa chiều (dimensional modeling) của Kimball,. Các khái niệm và vấn đề trong chương này được phân chia thành các nhóm chủ đề chính sau:

**1. Các Khái niệm Cơ bản (Fundamental Concepts)**

- **Thu thập yêu cầu kinh doanh và thực tế dữ liệu:** Thiết kế phải bắt đầu từ việc hiểu các chỉ số đo lường hiệu suất, vấn đề kinh doanh cốt lõi và đánh giá tính khả thi của dữ liệu từ hệ thống nguồn,.
- **Thiết kế hợp tác:** Các mô hình chiều không nên được thiết kế độc lập mà phải thông qua các buổi hội thảo có tính tương tác cao với đại diện từ phía doanh nghiệp và quản trị dữ liệu,.
- **Quy trình thiết kế 4 bước (Four-Step Dimensional Design Process):** Mọi thiết kế đều bắt buộc phải trải qua bốn quyết định trọng tâm:
    1. **Chọn quy trình nghiệp vụ (Business Processes):** Xác định các hoạt động vận hành tạo ra các thước đo hiệu suất (như nhận đơn hàng, thanh toán),,.
	    - Quy trình nghiệp vụ là các hoạt động tác nghiệp nền tảng do tổ chức thực hiện, thường được diễn đạt bằng các động từ hành động (ví dụ: nhận đơn hàng, lập hóa đơn, đăng ký khóa học, hoặc xử lý yêu cầu bồi thường).
		- Các quy trình này được hỗ trợ bởi các hệ thống tác nghiệp và là nơi tạo ra hoặc thu thập các chỉ số đo lường hiệu suất (metrics).
		- Việc thiết kế phải tập trung vào **các quy trình nghiệp vụ** thay vì tập trung vào ranh giới của các phòng ban/bộ phận, điều này giúp đảm bảo dữ liệu được cung cấp một cách nhất quán trên toàn bộ doanh nghiệp.
    2. **Xác định mức độ hạt (Declare the Grain):** Xác định chính xác ý nghĩa của một dòng (row) trong bảng sự kiện, đảm bảo mọi dữ liệu trong bảng phải cùng một mức độ chi tiết để tránh tính toán trùng lặp,. Mức độ hạt nguyên thủy (atomic) là tốt nhất để đáp ứng các truy vấn linh hoạt,.
	    - Đây là bước mang tính bước ngoặt của mọi thiết kế chiều, nhằm trả lời chính xác cho câu hỏi: **"Một dòng (row) trong bảng sự kiện đại diện cho điều gì?"**.
		- Mức độ hạt hoạt động như một bản hợp đồng ràng buộc toàn bộ thiết kế và **bắt buộc phải được xác định trước** khi chọn các chiều hoặc thước đo.
		- Mức độ hạt nên được diễn đạt bằng các thuật ngữ kinh doanh.
		- Lý tưởng nhất, mô hình nên lưu trữ dữ liệu ở **mức độ hạt chi tiết nhất (atomic grain)**—tức là mức độ thấp nhất mà quy trình nghiệp vụ thu thập được. Dữ liệu chi tiết nhất mang lại sự linh hoạt phân tích tối đa, giúp hệ thống chịu đựng được các truy vấn ngẫu nhiên (ad-hoc) không lường trước từ người dùng.
		- **Quy tắc bất di bất dịch:** Tuyệt đối không được trộn lẫn các mức độ hạt khác nhau trong cùng một bảng sự kiện.
    3. **Xác định các chiều (Identify the Dimensions):** Xác định các ngữ cảnh ("ai, cái gì, ở đâu, khi nào, như thế nào") bao quanh sự kiện,,.
	    - Chiều trả lời cho câu hỏi: _"Người dùng doanh nghiệp mô tả dữ liệu sinh ra từ các sự kiện đo lường như thế nào?"_. Các chiều cung cấp ngữ cảnh mô tả **"ai, cái gì, ở đâu, khi nào, tại sao và như thế nào"** bao quanh sự kiện nghiệp vụ đó.
		- Bảng chiều chứa các thuộc tính được các ứng dụng BI sử dụng để lọc (filtering) và nhóm (grouping) dữ liệu.
		- Khi mức độ hạt đã được cố định ở Bước 2, bạn có thể dễ dàng xác định tất cả các chiều có thể có. Mỗi chiều chỉ nên mang một giá trị duy nhất khi gắn với một dòng sự kiện cụ thể.
    4. **Xác định các thước đo (Identify the Facts):** Xác định các con số đo lường sinh ra từ quy trình nghiệp vụ đó,.
	    - Thước đo được xác định bằng cách trả lời câu hỏi: _"Quy trình này đang đo lường điều gì?"_.
		- Thước đo là kết quả sinh ra từ sự kiện nghiệp vụ và hầu như luôn là các **dữ liệu số (numeric facts)**. Những thước đo hữu ích nhất là các số liệu có thể cộng gộp toàn phần (additive), chẳng hạn như số lượng đặt hàng hay số tiền chi phí.
		- Tất cả các thước đo được đưa vào bảng sự kiện **phải tuân thủ tuyệt đối mức độ hạt** đã tuyên bố ở Bước 2. Nếu một thước đo thuộc về một mức độ hạt khác, nó bắt buộc phải được đặt ở một bảng sự kiện khác.
		- **Sự linh hoạt:** Mô hình chiều có khả năng mở rộng dễ dàng (thêm chiều mới, thêm số đo mới) mà không làm ảnh hưởng đến các truy vấn BI hiện tại.

**2. Kỹ thuật Bảng Sự kiện cơ bản (Basic Fact Table Techniques)**

- **Cấu trúc bảng sự kiện:** Chứa các số liệu đo lường và các khóa ngoại (foreign keys) để liên kết tới bảng chiều.
- **Tính cộng gộp:** Các sự kiện có thể là cộng gộp toàn phần (additive), cộng gộp bán phần (semi-additive - ví dụ số dư kho không thể cộng theo thời gian), hoặc không thể cộng gộp (non-additive).
	- - **Additive (Cộng gộp toàn phần):** Bạn có thể cộng con số này theo bất kỳ chiều nào.
	    - _Ví dụ:_ Doanh số. Bạn cộng theo Ngày, theo Cửa hàng hay theo Sản phẩm đều ra kết quả có ý nghĩa.
	- **Semi-additive (Cộng gộp bán phần):** Con số có thể cộng theo một số chiều, nhưng **không thể cộng theo thời gian**.
	    - _Ví dụ:_ Số dư tài khoản hoặc Tồn kho. Nếu hôm nay bạn có 100 triệu, ngày mai có 100 triệu, bạn không thể nói cả 2 ngày bạn có 200 triệu. Nhưng bạn có thể cộng số dư của 10 khách hàng khác nhau để biết tổng tiền trong ngân hàng.
	- **Non-additive (Không thể cộng gộp):** Thường là các tỉ lệ hoặc đơn giá.
	    - _Ví dụ:_ Tỉ suất lợi nhuận (10%). Bạn không thể cộng 10% của món hàng A với 10% của món hàng B để ra 20%. Bạn phải tính lại dựa trên tổng doanh thu và tổng vốn.
- **Giá trị Null:** Bảng sự kiện không được chứa giá trị Null ở các khóa ngoại; thay vào đó, phải dùng một dòng mặc định trong bảng chiều để thay thế,.
	- **Lý do:** Nếu bạn Join Fact với Dimension mà khóa ngoại bị Null, dòng dữ liệu đó sẽ biến mất khỏi kết quả báo cáo (do phép Inner Join).
	- **Cách xử lý:** Trong bảng Dimension, ta tạo một dòng "mồi" với ID là `-1` (Ví dụ: "Khách hàng vãng lai" hoặc "Không xác định"). Trong bảng Fact, nếu không biết khách hàng là ai, ta điền `-1` thay vì để Null.
- **3 Loại Bảng Sự kiện (Fact Tables) cơ bản:**
    - **Transaction (Giao dịch):** Mỗi dòng đại diện cho một sự kiện đo lường tại một thời điểm cụ thể, chứa nhiều chiều chi tiết nhất.
    - **Periodic Snapshot (Chụp nhanh định kỳ):** Tổng hợp dữ liệu theo các khoảng thời gian chuẩn (như mỗi ngày, mỗi tháng), thường chứa nhiều số đo và luôn có dữ liệu dù không có hoạt động phát sinh,.
    - **Accumulating Snapshot (Chụp nhanh lũy kế):** Có một dòng duy nhất cho vòng đời của một thực thể (như quy trình xử lý đơn hàng) và được cập nhật liên tục khi thực thể đó đi qua các mốc thời gian khác nhau.
- **Bảng sự kiện không có số đo (Factless Fact Tables):** Dùng để ghi nhận các sự kiện chỉ có sự tham gia của các chiều mà không có con số nào (ví dụ: điểm danh học sinh), hoặc ghi nhận các vùng phủ sóng sự kiện,.

**3. Kỹ thuật Bảng Chiều cơ bản (Basic Dimension Techniques)**

- **Khóa thay thế (Surrogate Keys):** Bảng chiều bắt buộc phải sử dụng các khóa nguyên vô nghĩa làm khóa chính thay vì sử dụng khóa tự nhiên (natural key) của hệ thống vận hành để tránh trùng lặp hoặc theo dõi lịch sử.
- **Chiều suy biến (Degenerate Dimensions):** Đây là các số kiểm soát giao dịch (ví dụ: số hóa đơn, số biên nhận) được đưa trực tiếp vào bảng sự kiện dưới dạng khóa, nhưng không có bảng chiều riêng đính kèm.
- **Bảng chiều phi chuẩn hóa:** Nên thiết kế các bảng chiều phẳng (denormalized), gộp các phân cấp nhiều-một (nhiều cấp độ) vào cùng một dòng để tăng tốc độ truy vấn.
- **Vai trò của chiều (Role-Playing Dimensions):** Một bảng chiều vật lý (như Date) có thể được tham chiếu bởi nhiều khóa ngoại trong cùng một bảng sự kiện với các vai trò khác nhau (ví dụ: ngày đặt hàng, ngày giao hàng).
- **Chiều rác (Junk Dimensions):** Thu gom các mã cờ (flags) và các chỉ báo trạng thái rời rạc vào một bảng chiều chung duy nhất thay vì tạo quá nhiều chiều nhỏ.
- **Snowflakes & Outriggers:** Khuyến cáo hạn chế việc phân mảnh bảng chiều (Snowflaking) vì nó làm giảm hiệu suất, nhưng cho phép một bảng chiều tham chiếu đến một chiều thứ cấp khác (Outrigger) trong các trường hợp đặc biệt,.

**4. Integration via Conformed Dimensions**

4.1. Conformed Dimensions

Một bảng Dimension được gọi là "Conformed" (Đồng nhất) khi nó có cấu trúc và nội dung dữ liệu giống hệt nhau khi được sử dụng ở nhiều bảng Fact khác nhau.
Nó đóng vai trò như một bộ từ điển dùng chung cho toàn bộ doanh nghiệp.
- **Ví dụ:** Bảng `Dim_Product` được dùng cho cả Fact Bán hàng, Fact Tồn kho và Fact Thu mua. Dù ở báo cáo nào, mã sản phẩm `123` vẫn phải tên là "iPhone 15" và thuộc danh mục "Điện thoại".

Thách thức lớn nhất trong OLAP là việc thực hiện các phân tích xuyên suốt (Cross-process analysis).
- Nếu không đồng nhất:
	- Phòng Sales định nghĩa khách hàng theo mã CRM.
	- Phòng Chăm sóc khách hàng định nghĩa khách hàng theo số điện thoại.
	- **Hậu quả:** Bạn không thể biết một khách hàng hay phàn nàn có phải là người đóng góp doanh số lớn nhất hay không, vì hai bảng Fact không có "tiếng nói chung".

4.2. Hai hình thức của Chiều đồng nhất

A. Identical (Giống hệt nhau)

Bảng Dimension được copy y nguyên hoặc dùng chung một bảng vật lý duy nhất.
- _Ví dụ:_ `Dim_Date` là bảng giống hệt nhau cho mọi quy trình.

B. Shrunken Rollups (Tập con thu gọn)

Đây là trường hợp đặc biệt nhưng cực kỳ quan trọng. Một bảng Dimension thu gọn chứa một tập con các hàng và cột của bảng Dimension chi tiết hơn, nhưng định nghĩa và khóa (Key) phải khớp nhau.
_Ví dụ:_
- `Dim_Product`: Chi tiết đến từng mã SKU (dùng cho Fact Bán hàng).
- `Dim_Brand`: Chỉ chi tiết đến mức Thương hiệu (dùng cho Fact Ngân sách/Kế hoạch).
- **Tính đồng nhất:** Thương hiệu "Apple" trong bảng thu gọn phải có cùng khóa và thuộc tính với thương hiệu "Apple" trong bảng chi tiết.

4.3. Drill-Across (Truy vấn xuyên suốt)

Đơn giản có nghĩa là thực hiện các truy vấn riêng biệt trên hai hoặc nhiều bảng dữ liệu, trong đó tiêu đề hàng của mỗi truy vấn bao gồm các thuộc tính giống hệt nhau đã được chuẩn hóa. Tập kết quả từ hai truy vấn được căn chỉnh bằng cách thực hiện thao tác sắp xếp-hợp nhất trên các tiêu đề hàng thuộc tính chiều chung. Các nhà cung cấp công cụ BI gọi chức năng này bằng nhiều tên khác nhau, bao gồm "stitch" và "multipass query"

Vì cả hai bảng Fact đều "treo" vào cùng một `Product_Key = 10`, bạn có thể thực hiện một báo cáo tích hợp cực kỳ quyền lực:
**Câu hỏi:** _"Danh sách sản phẩm có số lượng bán trong ngày lớn hơn số lượng tồn kho hiện tại (Cảnh báo cháy hàng)"_
**Cách hệ thống hoạt động:**
1. **Bước 1:** Truy vấn `Fact_Sales` để lấy tổng `Quantity_Sold` của `Product_Key = 10`. (Kết quả: 50).
2. **Bước 2:** Truy vấn `Fact_Inventory` để lấy `Quantity_On_Hand` của `Product_Key = 10`. (Kết quả: 5).
3. **Bước 3 (Integration):** Nối kết quả của Bước 1 và Bước 2 dựa trên **Conformed Dimension (`Product_Key`)**.

| **Product_Name** | **Doanh số hôm nay** | **Tồn kho hiện tại** | **Trạng thái**           |
| ---------------- | -------------------- | -------------------- | ------------------------ |
| iPhone 15 Pro    | 50                   | 5                    | **Cảnh báo: Cháy hàng!** |

# Chương 3: Retail Sales

Chương 3 của tài liệu sử dụng hệ thống bán lẻ (Retail Sales) làm ví dụ kinh điển để minh họa cách xây dựng một mô hình chiều từ đầu. Mặc dù sử dụng ngành bán lẻ, các vấn đề và kỹ thuật được trình bày ở đây là kiến thức nền tảng, có thể áp dụng cho mọi loại hình doanh nghiệp,.

Dưới đây là chi tiết các khái niệm và vấn đề được trình bày trong Chương 3:

**1. Quy trình Thiết kế 4 bước (Four-Step Dimensional Design Process)** Để xây dựng một mô hình chiều, tác giả nhấn mạnh việc phải đi qua 4 bước nhất quán, kết hợp giữa việc hiểu yêu cầu nghiệp vụ và thực tế dữ liệu nguồn,:

- **Bước 1: Chọn quy trình nghiệp vụ (Select the Business Process):** Các quy trình nghiệp vụ là các hoạt động vận hành của tổ chức (ví dụ: giao dịch bán hàng tại quầy POS),. Việc xác định đúng quy trình giúp thiết lập mục tiêu thiết kế rõ ràng.
- **Bước 2: Xác định mức độ hạt (Declare the Grain):** Đây là bước quan trọng nhất, quyết định chính xác một dòng (row) trong bảng sự kiện đại diện cho điều gì. Trong ví dụ bán lẻ, mức độ hạt là "một dòng cho mỗi sản phẩm được quét trên mã hóa đơn". Mô hình nên lưu trữ dữ liệu ở mức độ chi tiết nhất (atomic data) để đảm bảo tính linh hoạt và có thể đáp ứng các truy vấn phân tích ngẫu nhiên,.
- **Bước 3: Xác định các chiều (Identify the Dimensions):** Dựa trên mức độ hạt, ta xác định các ngữ cảnh (ai, cái gì, ở đâu, khi nào, như thế nào) bao quanh sự kiện đo lường. Với bán lẻ, các chiều là: Ngày (Date), Sản phẩm (Product), Cửa hàng (Store), Khuyến mãi (Promotion), Thu ngân (Cashier), và Phương thức thanh toán (Payment Method),.
- **Bước 4: Xác định các thước đo/sự kiện (Identify the Facts):** Đây là các con số được sinh ra từ quy trình. Ví dụ: số lượng bán, giá vốn, doanh thu. Các thước đo này được chia làm ba loại: cộng gộp toàn phần (additive), không thể cộng gộp (non-additive - ví dụ tỷ lệ phần trăm), hoặc các số đo phái sinh (derived facts) được tính toán từ các số đo khác như lợi nhuận gộp.

**2. Chi tiết về các Bảng Chiều (Dimension Table Details)** Chương 3 đi sâu vào thiết kế các bảng chiều quan trọng:

- **Chiều Ngày (Date Dimension):** Mọi mô hình chiều đều cần bảng Date. Người dùng không nên chỉ dùng định dạng ngày tháng có sẵn trong SQL mà cần một bảng chiều riêng để lưu trữ các thông tin như ngày lễ, mùa, giai đoạn tài chính, hay chỉ báo ngày cuối tuần để phục vụ nhu cầu lọc và phân tích,.
- **Chiều Sản phẩm (Product Dimension):** Chứa hàng chục thuộc tính mô tả và cấu trúc phân cấp (hierarchy) từ Ngành hàng -> Danh mục -> Thương hiệu -> Sản phẩm. Kỹ thuật cốt lõi là luôn phi chuẩn hóa (flattened/denormalize) tất cả các phân cấp này vào chung một bảng chiều duy nhất để tối ưu hiệu suất và sự đơn giản,.
    - _Giá trị số (Numeric values):_ Nếu một con số dùng để tính toán thì nó là Fact, nhưng nếu nó ổn định và dùng để lọc/nhóm (như giá niêm yết chuẩn) thì nó cũng nên làm thuộc tính trong bảng chiều,.
- **Chiều Khuyến mãi (Promotion / Causal Dimension):** Giải thích "tại sao" khách hàng mua sản phẩm (ví dụ: do quảng cáo, giảm giá, trưng bày).

**3. Các Khái niệm Đặc biệt và Kỹ thuật Nâng cao**

- **Chiều suy biến (Degenerate Dimensions):** Đối với dữ liệu giao dịch, mã số hóa đơn (POS transaction ticket number) phục vụ như một mã kiểm soát. Nó được lưu trực tiếp vào bảng sự kiện dưới dạng một khóa, nhưng không có bảng chiều riêng nào liên kết với nó,.
- **Khóa thay thế (Surrogate Keys):** Các bảng chiều phải dùng khóa thay thế (chuỗi số nguyên vô nghĩa) thay vì khóa tự nhiên từ hệ thống nguồn (Natural keys) nhằm tránh lỗi khi mã sản phẩm bị tái sử dụng hoặc để hỗ trợ việc theo dõi lịch sử thay đổi.
- **Bảng sự kiện không có số đo (Factless Fact Tables):** Dùng để ghi nhận các sự kiện xảy ra nhưng không tạo ra con số nào. Ví dụ tiêu biểu là bảng _Độ phủ khuyến mãi (Promotion Coverage)_, dùng để ghi nhận những sản phẩm nào được áp dụng khuyến mãi ở cửa hàng nào vào ngày nào, dù sản phẩm đó có được khách hàng mua hay không,,. Việc kết hợp bảng này với bảng bán hàng giúp trả lời các câu hỏi về những sự kiện "không xảy ra" (ví dụ: sản phẩm có khuyến mãi nhưng không bán được),.
- **Khả năng mở rộng (Extensibility):** Mô hình chiều cực kỳ linh hoạt. Bạn có thể thêm chiều mới hoặc số đo mới vào bảng sự kiện mà không làm hỏng hay thay đổi kết quả của các truy vấn và ứng dụng BI hiện có.

**4. Các sai lầm phổ biến cần tránh (Patterns to Avoid)**

- **Snowflaking (Phân mảnh bảng chiều):** Đây là quá trình chuẩn hóa (normalize) bảng chiều (ví dụ tách riêng bảng Category ra khỏi Product). Tuy giúp tiết kiệm không gian lưu trữ, nhưng Snowflaking tạo ra mạng lưới bảng phức tạp, làm giảm đáng kể khả năng dễ hiểu đối với người dùng và suy giảm hiệu suất truy vấn,,. Lời khuyên là hãy chống lại mong muốn chuẩn hóa bảng chiều.
- **Chiều Outrigger:** Cho phép một bảng chiều tham chiếu đến một bảng chiều khác. Kỹ thuật này hợp lệ nhưng chỉ nên dùng hết sức hạn chế vì nó gây ra các tác động tiêu cực giống như Snowflaking,.
- **Bảng sự kiện rết (Centipede Fact Tables):** Xảy ra khi nhà thiết kế đưa quá nhiều khóa ngoại (đại diện cho mọi cấp độ của một hệ thống phân cấp hoặc vô số chiều nhỏ lẻ) vào bảng sự kiện, khiến bảng sự kiện có "hàng trăm cái chân",. Cần gom nhóm các yếu tố có mối tương quan phân cấp thành các bảng chiều lớn hơn thay vì rải rác chúng ra.

# Chương 4: Inventory

Chương 4 của tài liệu tiếp tục sử dụng ngành bán lẻ nhưng tiến ngược lên chuỗi giá trị để giải quyết bài toán Quản lý Kho hàng (Inventory). Thông qua ví dụ này, chương tập trung vào việc định hình Kiến trúc Bus của Kho dữ liệu Doanh nghiệp và các kỹ thuật tích hợp dữ liệu toàn tổ chức.

Dưới đây là chi tiết các khái niệm và vấn đề cốt lõi được trình bày trong Chương 4:

**1. Chuỗi giá trị (Value Chain)** Chuỗi giá trị xác định luồng chảy logic của các hoạt động nghiệp vụ chính trong tổ chức (ví dụ: phát hành đơn mua hàng -> nhập kho -> xuất đến cửa hàng -> bán lẻ),. Mỗi quy trình trong chuỗi sinh ra các số đo với mức độ chi tiết (grain) và chu kỳ thời gian khác nhau, do đó mỗi quy trình thường cần ít nhất một bảng sự kiện (fact table) riêng biệt để lưu trữ,.

**2. Ba Mô hình Kho hàng (Three Inventory Models)** Do đặc thù của quy trình kho, tài liệu giới thiệu 3 mô hình sự kiện bổ trợ cho nhau:

- **Chụp nhanh định kỳ (Inventory Periodic Snapshot):** Đo lường mức tồn kho ở các khoảng thời gian đều đặn (như mỗi ngày, mỗi tuần) để tạo thành một chuỗi các lớp dữ liệu,. Mô hình này lý tưởng cho các quy trình kho được bổ sung liên tục. Lưu ý quan trọng là nó chứa các số đo **bán cộng gộp (semi-additive facts)**, chẳng hạn như số lượng tồn kho có thể cộng theo cửa hàng nhưng tuyệt đối không thể cộng gộp qua các ngày,,.
- **Giao dịch kho (Inventory Transaction):** Ghi nhận mọi giao dịch làm thay đổi mức tồn kho tại thời điểm nó xảy ra (ví dụ: nhập hàng, xuất hàng, hao hụt),.
- **Chụp nhanh lũy kế (Inventory Accumulating Snapshot):** Được dùng để theo dõi vòng đời của một lô hàng/sản phẩm từ lúc nhận đến khi rời khỏi kho qua các cột mốc thời gian chuẩn (như ngày nhận, ngày kiểm tra, ngày xếp lên kệ, ngày xuất),. Mô hình này phù hợp cho các quy trình có điểm bắt đầu và kết thúc rõ ràng.

**3. Kiến trúc Bus và Ma trận Bus (Enterprise DW Bus Architecture & Matrix)** Để xây dựng một kho dữ liệu tích hợp toàn doanh nghiệp một cách linh hoạt, Kimball đề xuất Kiến trúc Bus.

- **Kiến trúc Bus:** Cung cấp một khung thiết kế chuẩn hóa giúp bạn có thể phát triển kho dữ liệu theo từng giai đoạn (incremental) mà các phân hệ vẫn ghép nối được với nhau một cách liền mạch.
- **Ma trận Bus (Bus Matrix):** Là công cụ cốt lõi để lập kế hoạch kiến trúc này. Ma trận bao gồm các hàng là **Quy trình nghiệp vụ (Business Processes)** và các cột là các **Chiều (Dimensions)**. Ma trận giúp chia nhỏ một dự án DW/BI khổng lồ thành các phần dễ quản lý (thực hiện từng hàng một) mà không tạo ra các "ốc đảo" dữ liệu (stovepipes).
- **Ma trận Cơ hội/Bên liên quan (Opportunity/Stakeholder Matrix):** Là một biến thể trong đó các cột chiều được thay thế bằng các phòng ban kinh doanh (Sales, Marketing, Finance...) để biểu thị phòng ban nào quan tâm đến quy trình nào, hỗ trợ xác định ưu tiên dự án.

**4. Các Chiều Đồng nhất (Conformed Dimensions)** Đây là "chất keo" kết dính Kiến trúc Bus. Các chiều đồng nhất là những bảng chiều được thiết kế chuẩn hóa và dùng chung cho nhiều quy trình nghiệp vụ.

- **Drilling Across (Kết hợp dữ liệu):** Nhờ các chiều đồng nhất, người dùng có thể thực hiện truy vấn trên nhiều bảng sự kiện khác nhau (ví dụ: nối dữ liệu Nhập kho và Bán hàng) vào cùng một báo cáo một cách chính xác.
- **Chiều thu gọn (Shrunken Dimensions):** Là một tập con của chiều đồng nhất, thường dùng khi một quy trình nghiệp vụ chỉ thu thập dữ liệu ở mức tổng hợp (ví dụ: dự báo doanh số theo tháng và thương hiệu, thay vì theo ngày và từng sản phẩm cụ thể) hoặc chỉ sử dụng một nhóm nhỏ dữ liệu,.
- **Hỗ trợ Agile:** Các chiều đồng nhất thúc đẩy phương pháp Agile vì bạn chỉ cần xây dựng và duy trì bảng chiều một lần, sau đó tái sử dụng chúng trên toàn bộ các dự án sau này, giúp tiết kiệm chi phí và thời gian phát triển.

**5. Các Số đo Đồng nhất và Quản trị Dữ liệu (Conformed Facts & Data Governance)**

- **Số đo Đồng nhất:** Bên cạnh các chiều, nếu một con số đo lường (như doanh thu, chi phí) xuất hiện ở nhiều bảng sự kiện khác nhau, chúng bắt buộc phải có chung công thức tính toán và đơn vị đo để dùng chung một tên. Nếu chúng có ý nghĩa khác nhau, chúng phải được đặt tên khác nhau để tránh gây nhầm lẫn.
- **Quản trị Dữ liệu (Data Governance):** Việc thống nhất các định nghĩa cho chiều và số đo không chỉ là công việc của IT. Cần có sự tham gia của các Quản trị viên Dữ liệu (Data Stewards) và lãnh đạo cấp cao từ phía doanh nghiệp để đạt được sự đồng thuận trong toàn tổ chức,.

# Chương 5: Procurement

Chương 5 của tài liệu sử dụng quy trình Mua sắm (Procurement) làm ví dụ nền tảng để trình bày các kỹ thuật thiết kế, trong đó nổi bật và quan trọng nhất là toàn bộ các phương pháp xử lý **Chiều thay đổi chậm (Slowly Changing Dimensions - SCD) từ cơ bản đến nâng cao lai tạp (hybrid)**,.

Dưới đây là chi tiết các khái niệm và vấn đề được trình bày trong Chương 5:

**1. Giao dịch Mua sắm và Ma trận Bus (Procurement Transactions & Bus Matrix)** Quy trình mua sắm bao gồm một chuỗi các giao dịch phức tạp như: yêu cầu mua hàng, đơn đặt hàng, thông báo giao hàng, biên nhận kho và thanh toán cho nhà cung cấp.

- **Vấn đề: Một hay nhiều bảng sự kiện giao dịch?** Người thiết kế thường phân vân giữa việc gộp tất cả vào một bảng hay tách ra. Kimball khuyên rằng nếu các giao dịch có các chiều (dimensionality) khác nhau (ví dụ: chiết khấu chỉ áp dụng cho thanh toán chứ không áp dụng cho các bước khác), hoặc đến từ các hệ thống nguồn riêng biệt, chúng nên được thiết kế thành các **bảng sự kiện giao dịch độc lập** (Multiple Transaction Fact Tables),.

**2. Bảng sự kiện Chụp nhanh Tích lũy (Complementary Procurement Snapshot)** Bên cạnh các bảng giao dịch rời rạc, chương này nhắc lại việc sử dụng một bảng **Chụp nhanh tích lũy (Accumulating Snapshot)** để đo lường toàn bộ quy trình mua sắm,. Bảng này chứa nhiều khóa ngày tháng đại diện cho các mốc thời gian (ngày đặt hàng, ngày yêu cầu, ngày nhận, ngày thanh toán) và các thước đo khoảng thời gian chờ (lag/duration facts) giữa các bước để đánh giá hiệu suất của chuỗi cung ứng.

**3. Các kỹ thuật Chiều Thay đổi Chậm Cơ bản (Slowly Changing Dimension Basics)** Chương 5 là nơi định nghĩa chính thức và chi tiết các loại SCD để giải quyết việc thuộc tính của chiều thay đổi theo thời gian:

- **Type 0 (Giữ nguyên bản gốc - Retain Original):** Thuộc tính không bao giờ thay đổi sau khi được tạo. Dữ liệu sự kiện luôn được nhóm theo giá trị nguyên thủy này (ví dụ: ngày sinh, mã số nhận dạng ban đầu),.
- **Type 1 (Ghi đè - Overwrite):** Giá trị cũ trong bảng chiều bị ghi đè bằng giá trị mới nhất. Phương pháp này dễ triển khai nhưng **phá hủy dữ liệu lịch sử** và sẽ làm thay đổi kết quả của các báo cáo tổng hợp trước đó,.
- **Type 2 (Thêm dòng mới - Add New Row):** Đây là phương pháp phổ biến nhất để theo dõi lịch sử chính xác. Mỗi khi có thay đổi, một dòng mới được thêm vào bảng chiều với khóa thay thế (surrogate key) mới. Kỹ thuật này yêu cầu thêm ít nhất ba cột quản lý: Ngày bắt đầu hiệu lực (Row Effective Date), Ngày hết hạn (Row Expiration Date), và Cờ trạng thái hiện hành (Current Row Indicator),,,.
- **Type 3 (Thêm thuộc tính mới - Add New Attribute):** Thay vì thêm dòng, ta thêm một cột mới vào bảng chiều để bảo lưu giá trị cũ, trong khi cột chính bị ghi đè bằng giá trị mới. Kỹ thuật này hiếm dùng, chủ yếu dùng khi doanh nghiệp muốn xem đồng thời cả hai "thực tế thay thế" (alternate reality) của giá trị cũ và mới,,.
- **Type 4 (Thêm Mini-Dimension):** Dùng để giải quyết các "bảng chiều quái vật" (monster dimension) nơi có các thuộc tính thay đổi rất nhanh (như phân khúc độ tuổi, mức thu nhập). Các thuộc tính này được tách ra thành một **bảng chiều nhỏ (mini-dimension)**, sau đó khóa của mini-dimension được đưa trực tiếp vào bảng sự kiện làm khóa ngoại,,.

**4. Các kỹ thuật SCD Lai (Hybrid SCD Techniques)** Trong nhiều trường hợp phức tạp, doanh nghiệp muốn vừa giữ được lịch sử (as-was) vừa muốn báo cáo toàn bộ dữ liệu theo trạng thái hiện tại (as-is). Kimball đưa ra các mô hình lai:

- **Type 5 (Mini-Dimension + Type 1 Outrigger):** Nâng cấp từ Type 4, kỹ thuật này thêm một khóa ngoại (được cập nhật liên tục theo Type 1) vào bảng chiều chính trỏ đến bảng mini-dimension hiện tại. Điều này cho phép phân tích trạng thái hiện hành của khách hàng mà không cần phải đi qua (join) bảng sự kiện,,.
- **Type 6 (Thêm thuộc tính Type 1 vào bảng chiều Type 2):** (Tên gọi là 6 vì 2+3+1=6 hoặc 2x3x1=6). Bảng chiều vừa chứa các dòng lịch sử (Type 2), vừa chứa một cột thuộc tính hiện hành (Current Attribute) được ghi đè liên tục (Type 1) trên tất cả các dòng của cùng một thực thể. Nhờ đó, người dùng có thể nhóm dữ liệu theo giá trị lịch sử hoặc giá trị mới nhất tùy ý,,.
- **Type 7 (Sử dụng đồng thời Type 1 và Type 2):** Bảng sự kiện chứa hai khóa ngoại cho cùng một chiều: một khóa thay thế trỏ đến bảng chiều Type 2 để lấy ngữ cảnh lịch sử, và một khóa tự nhiên/bền vững (durable key) trỏ đến một chế độ xem (View) Type 1 của chiều đó (chỉ chứa trạng thái mới nhất). Phương pháp này linh hoạt cho phép báo cáo đồng thời theo cả trạng thái lúc xảy ra sự kiện và trạng thái hiện tại,,.

# Chương 6: Order Management

Chương 6 của tài liệu tập trung vào chủ đề **Quản lý Đơn hàng (Order Management)**, một quy trình cốt lõi đối với bất kỳ tổ chức nào có hoạt động bán sản phẩm hoặc dịch vụ. Chương này đi sâu vào các kỹ thuật thiết kế mô hình chiều cho các giao dịch đặt hàng, lập hóa đơn, và toàn bộ vòng đời thực hiện đơn hàng. Dưới đây là chi tiết các khái niệm và vấn đề được trình bày:

**1. Ma trận Bus cho Quản lý Đơn hàng (Order Management Bus Matrix)** Quy trình quản lý đơn hàng không chỉ là một sự kiện đơn lẻ mà là một chuỗi (pipeline) các quy trình như: Báo giá, Đặt hàng, Giao hàng, Lập hóa đơn, Nhận thanh toán và Xử lý hàng trả lại. Mỗi quy trình này sinh ra các bảng sự kiện (fact tables) riêng biệt nhưng chia sẻ chung các chiều đồng nhất (như Ngày, Khách hàng, Sản phẩm) để đảm bảo tính tích hợp thông qua Kiến trúc Bus.

**2. Thiết kế Lược đồ Sự kiện Giao dịch Đơn hàng (Order Transactions)** Mức độ hạt (grain) tự nhiên và chi tiết nhất cho bảng sự kiện đơn hàng là **một dòng cho mỗi dòng chi tiết của đơn hàng (order line item)**.

- **Chuẩn hóa bảng sự kiện (Fact Normalization):** Tác giả cảnh báo về xu hướng chuẩn hóa các số đo (ví dụ: gom tất cả các loại số tiền vào một cột duy nhất kèm theo một chiều phân loại số đo). Kỹ thuật này làm bùng nổ số lượng dòng (tăng gấp nhiều lần) và làm khó các phép tính toán học chéo giữa các số đo trong SQL. Trừ khi tập hợp các số đo vô cùng lớn nhưng dữ liệu lại cực kỳ thưa thớt, nếu không, hãy để các số đo thành các cột riêng biệt trong cùng một dòng.

**3. Các Kỹ thuật Chiều Quan trọng (Dimension Techniques)**

- **Vai trò của chiều (Dimension Role Playing):** Một giao dịch đơn hàng thường chứa nhiều mốc ngày tháng (ví dụ: ngày đặt hàng, ngày yêu cầu giao). Không cần tạo nhiều bảng chiều thời gian vật lý, thay vào đó ta dùng một bảng Date duy nhất và tạo ra các **khung nhìn (views/aliases)** khác nhau cho mỗi vai trò để kết nối vào bảng sự kiện.
- **Chiều Sản phẩm (Product Dimension):** Nhấn mạnh việc bắt buộc sử dụng khóa thay thế (surrogate keys) thay cho mã tự nhiên của sản phẩm, đồng thời cần làm phong phú bảng bằng các thuộc tính mô tả.
- **Chiều Khách hàng (Customer Dimension):** Xem xét cấu trúc phân cấp khách hàng (ví dụ: địa chỉ giao hàng - ship-to và địa chỉ thanh toán - bill-to). Nếu mối quan hệ này cố định, có thể gộp vào chung một bảng; nếu phức tạp (nhiều-nhiều), có thể cần tách thành các chiều riêng biệt.
- **Chiều Thỏa thuận/Giao dịch (Deal Dimension):** Tương tự chiều khuyến mãi, chiều này mô tả các điều khoản, phụ cấp, và ưu đãi áp dụng cho từng dòng đơn hàng.
- **Chiều suy biến cho Số đơn hàng (Degenerate Dimension):** Mã số hoặc số biên nhận từ hệ thống vận hành (như Order Number) được lưu trực tiếp vào bảng sự kiện dưới dạng khóa suy biến, không cần có bảng chiều riêng đính kèm.
- **Chiều Rác (Junk Dimensions):** Thay vì tạo hàng chục bảng chiều nhỏ lẻ cho các mã cờ (flags) hay chỉ báo (indicators) có rất ít giá trị, nên gom tất cả chúng lại thành một "chiều rác" duy nhất để giảm bớt sự rườm rà cho thiết kế.

**4. Hai Sai lầm với Dữ liệu Header/Line Cần tránh (Patterns to Avoid)**

- **Sai lầm 1:** Xử lý Header của đơn hàng như một bảng chiều khổng lồ và kết nối nó với bảng sự kiện Line Item. Điều này làm bảng chiều phình to (có thể bằng 20% kích thước bảng sự kiện) và làm giảm sút hiệu suất truy vấn nghiêm trọng.
- **Sai lầm 2:** Tạo hai bảng sự kiện riêng biệt (một cho Header, một cho Line Item) không kế thừa các chiều của nhau. Việc buộc hệ thống BI phải join trực tiếp hai bảng sự kiện thông qua mã số đơn hàng là thao tác tối kỵ vì gây ra rủi ro đếm trùng dữ liệu và làm truy vấn chạy rất chậm.

**5. Xử lý Thước đo / Số đo Đặc biệt (Special Facts Handling)**

- **Phân bổ Thước đo (Allocated Facts):** Nếu có các con số sinh ra ở cấp độ toàn bộ đơn hàng (header) như phí vận chuyển, thiết kế tốt nhất là phải **phân bổ (allocate)** các khoản này xuống từng dòng chi tiết (line item) để có thể phân tích và cộng gộp qua mọi chiều.
- **Đa tiền tệ (Multiple Currencies):** Để đáp ứng nhu cầu quy đổi, bảng sự kiện nên chứa một cặp cột cho mỗi số đo tài chính: một cột cho tiền tệ nguyên bản (local currency) và một cột quy đổi chuẩn (standard corporate currency). Nếu nhu cầu phức tạp hơn, có thể dùng thêm bảng sự kiện tỷ giá riêng.
- **Đa đơn vị đo lường (Multiple Units of Measure):** Bảng sự kiện nên lưu một số lượng chuẩn kèm theo các hệ số chuyển đổi (conversion factors) ngay trong dòng đó, cho phép tính toán ra các đơn vị khác (thùng, kiện, hộp) mà không bắt người dùng phải tự tìm hệ số ở nơi khác.

**6. Giao dịch Lập hóa đơn và Lợi nhuận (Invoicing & P&L)**

- Lập hóa đơn thường là thời điểm ghi nhận doanh thu. Bảng sự kiện lập hóa đơn có thể được mở rộng để chứa các yếu tố của Báo cáo Lợi nhuận (Profit and Loss - P&L), bao gồm doanh thu, chiết khấu, và mọi loại chi phí (sản xuất, kho bãi, vận chuyển) để tính ra lợi nhuận ròng cuối cùng.
- **Đo lường Mức độ Dịch vụ (Service Level Performance):** Đánh giá hiệu suất giao hàng bằng các con số định lượng (như số ngày giao trễ) ở bảng sự kiện, kết hợp với các mô tả định tính (sớm, đúng hạn, trễ) trong một bảng chiều.
- **Chiều Kiểm toán (Audit Dimension):** Một bảng chiều đặc biệt được gắn vào bảng sự kiện để theo dõi các siêu dữ liệu (metadata) của quá trình ETL khi dòng dữ liệu đó được tạo ra (ví dụ: chất lượng dữ liệu, phiên bản phân bổ chi phí).

**7. Chụp nhanh Tích lũy cho Vòng đời Đơn hàng (Accumulating Snapshot for Order Fulfillment)** Bên cạnh bảng giao dịch đơn lẻ, để phân tích tốc độ (velocity) của toàn bộ đường ống thực hiện đơn hàng (từ lúc đặt hàng -> sản xuất -> lưu kho -> giao hàng -> lập hóa đơn), ta sử dụng mô hình **Chụp nhanh tích lũy**.

- Mỗi dòng đại diện cho một chi tiết đơn hàng, dòng này sẽ liên tục được cập nhật trạng thái khi đơn hàng đi qua các mốc quy trình.
- Bảng chứa rất nhiều khóa ngày tháng (mỗi cột là một mốc thời gian) dùng chung bảng chiều Ngày thông qua khái niệm nhập vai (role-playing).
- Bảng sự kiện thường lưu sẵn các khoảng thời gian tính bằng ngày, gọi là **độ trễ (Lags)** giữa các bước, để người dùng dễ dàng phân tích hiệu suất tổng thể.

# Chương 7: Accounting

Dựa vào tài liệu được cung cấp, Chương 7 tập trung vào lĩnh vực **Kế toán (Accounting)**, cụ thể là dữ liệu Sổ cái chung (General Ledger - G/L), các chuỗi quy trình lập ngân sách, và đặc biệt là cách xử lý các cấu trúc phân cấp (hierarchies) phức tạp thường gặp trong tài chính.

Dưới đây là chi tiết các khái niệm và vấn đề cốt lõi được trình bày trong chương này:

**1. Dữ liệu Sổ cái chung (General Ledger Data)**

- **Chụp nhanh định kỳ (Periodic Snapshot):** Dữ liệu kế toán thường được xem xét dưới dạng các ảnh chụp nhanh hàng tháng. Mức độ hạt là một dòng cho mỗi tài khoản vào cuối mỗi kỳ kế toán.
- **Hệ thống tài khoản (Chart of Accounts):** Đây là nền tảng của G/L, thường sử dụng "khóa thông minh" (intelligent keys) gồm nhiều phần ghép lại với nhau. Tuy nhiên, trong kho dữ liệu, hệ thống tài khoản này cần được tách ra và lưu trữ dưới dạng một bảng chiều chi tiết.
- **Số dư cuối kỳ (Period Close):** Bảng sự kiện chụp nhanh sẽ lưu số dư của tài khoản vào cuối kỳ. Số dư này là một thước đo **bán cộng gộp (semi-additive)**, nghĩa là có thể cộng theo các chiều khác (như tài khoản, phòng ban) nhưng không thể cộng gộp qua các kỳ thời gian.
- **Sự thật Từ đầu năm đến nay (Year-to-Date Facts - YTD):** Tài liệu khuyên **không nên** lưu trữ trực tiếp các số liệu YTD trong bảng sự kiện vì yêu cầu của người dùng có thể dễ dàng biến tấu thành "từ đầu quý đến nay" hoặc "từ đầu kỳ đến nay". Thay vào đó, cách linh hoạt và đáng tin cậy nhất là để các ứng dụng BI hoặc khối OLAP tự tính toán các chỉ số YTD.

**2. Giao dịch Nhật ký Sổ cái (General Ledger Journal Transactions)** Bên cạnh ảnh chụp nhanh, cần có một bảng sự kiện thứ hai ghi nhận các giao dịch chi tiết tạo nên số dư đó:

- Mỗi dòng tương ứng với một bút toán ghi Nợ (Debit) hoặc Có (Credit).
- **Đa lịch tài chính (Multiple Fiscal Calendars):** Thường cần sử dụng nhiều chiều Ngày nhập vai (role-playing) để phân biệt giữa "ngày hạch toán" (posting date) và "ngày hiệu lực kế toán" (effective accounting date), hoặc để hỗ trợ các lịch tài chính đa dạng của doanh nghiệp.

**3. Chuỗi quy trình Lập ngân sách (Budgeting Chain)** Quy trình này là một chuỗi các sự kiện bao gồm: Lập ngân sách (Budgets) -> Cam kết chi (Commitments) -> Thanh toán (Payments).

- Với ngân sách, mức độ hạt hợp lý là ghi nhận **thay đổi thuần (net change)** của từng khoản mục ngân sách trong một phòng ban vào một tháng, chứ không phải ghi nhận toàn bộ số dư dự kiến.
- Người dùng có thể dùng kỹ thuật _Drill-Across_ (kết hợp dữ liệu nhiều bảng thông qua các chiều đồng nhất) để so sánh các khoản cam kết so với ngân sách hiện tại.

**4. Giải quyết các Cấu trúc Phân cấp (Dealing with Hierarchies) - Vấn đề trọng tâm** Chương 7 đi sâu vào các kỹ thuật mô hình hóa cấu trúc phân cấp, đặc biệt là cấu trúc phòng ban/tổ chức:

- **Phân cấp độ sâu cố định (Fixed Depth):** Chuỗi quan hệ nhiều-một có số cấp cố định. Cách tốt nhất là làm phẳng (flatten) và đưa chúng thành các thuộc tính trên bảng chiều. Cách này dễ hiểu và cho hiệu suất truy vấn nhanh nhất.
- **Phân cấp hơi rách/Độ sâu thay đổi nhẹ (Slightly Ragged):** Ví dụ như phân cấp địa lý (từ 3 đến 6 cấp). Bạn nên ép chúng vào cấu trúc cố định với số lượng cấp tối đa, và dùng quy tắc nghiệp vụ để điền dữ liệu vào các cấp bị trống.
- **Phân cấp rách hoàn toàn/Độ sâu không xác định (Ragged Variable Depth):** Điển hình là Sơ đồ tổ chức, nơi một nhánh có 3 cấp nhưng nhánh khác có tới 10 cấp.
    - **Sử dụng Bảng cầu nối (Hierarchy Bridge Tables):** Đây là phương pháp mạnh mẽ nhất. Tạo một bảng cầu nối chứa _một dòng cho mọi đường dẫn (path) có thể có từ một nút cha đến tất cả các nút con của nó_. Bảng này cho phép truy vấn duyệt toàn bộ cây phân cấp một cách linh hoạt bằng SQL chuẩn.
    - Bảng cầu nối cũng giải quyết được bài toán **Sở hữu chung (Shared Ownership)** khi một tổ chức con thuộc về nhiều tổ chức cha.
    - **Phân cấp thay đổi theo thời gian (Time Varying):** Bằng cách gắn thêm nhãn thời gian bắt đầu (Begin Effective Date) và kết thúc (End Effective Date) vào Bảng cầu nối, hệ thống có thể theo dõi sự thay đổi của sơ đồ tổ chức qua từng thời kỳ.
- **Các phương pháp thay thế:** Sử dụng _chuỗi đường dẫn (Pathstring)_ giúp duyệt cây nhanh nhưng lại rất khó bảo trì hoặc thay đổi vì mọi sự sắp xếp lại đều yêu cầu phải dán nhãn lại toàn bộ cấu trúc nhánh bên dưới.

**5. Bảng sự kiện hợp nhất (Consolidated Fact Tables)** Nếu người dùng thường xuyên phải kết hợp và so sánh dữ liệu từ nhiều quy trình khác nhau (ví dụ điển hình nhất là **Thực tế so với Ngân sách - Actual vs. Budget**), thiết kế tốt nhất là tạo ra một bảng sự kiện hợp nhất.

- Bảng này kết hợp các con số từ hai quy trình vào cùng một dòng và cung cấp sẵn cột dung sai (variance) đã được tính toán.
- Điều này giúp loại bỏ gánh nặng cho các công cụ BI và người dùng trong việc tự kết nối (Drill-Across) các kết quả phức tạp.

**6. Vai trò của Khối đa chiều (OLAP) trong Kế toán** Các sản phẩm OLAP đã đóng một vai trò quan trọng trong báo cáo tài chính từ rất lâu. Trong khi cơ sở dữ liệu quan hệ lưu trữ dữ liệu nền tảng rất tốt, các khối OLAP thường được dùng làm lớp trình bày (presentation layer) cuối cùng để cung cấp hiệu suất truy vấn cực nhanh và xử lý các phép tính tài chính chéo phức tạp mà SQL truyền thống khó thực hiện được.

# Chương 8: Customer Relationship Management

Chương 8 của tài liệu tập trung vào chủ đề **Quản trị Quan hệ Khách hàng (Customer Relationship Management - CRM)**. Lĩnh vực này áp dụng cho mọi ngành nghề có liên quan đến con người hoặc tổ chức (khách hàng, công dân, bệnh nhân, sinh viên, v.v.). Chương này đi sâu vào các kỹ thuật thiết kế bảng chiều khách hàng và cách xử lý các hành vi phức tạp của họ.

Dưới đây là chi tiết các khái niệm và vấn đề được trình bày trong Chương 8:

**1. Tổng quan về CRM (CRM Overview)**

- **CRM Tác nghiệp (Operational CRM) và CRM Phân tích (Analytic CRM):** CRM tác nghiệp là các hệ thống được dùng để tương tác trực tiếp với khách hàng. Tuy nhiên, để có được "góc nhìn 360 độ" toàn cảnh về khách hàng, hệ thống DW/BI phải đóng vai trò là cốt lõi (Analytic CRM). Kho dữ liệu thu thập, tích hợp thông tin từ nhiều nguồn và trả kết quả phân tích về lại cho hệ thống tác nghiệp để đưa ra các quyết định theo thời gian thực (ví dụ: đề xuất sản phẩm tiếp theo, quyết định cấp tín dụng),,.

**2. Xây dựng Bảng Chiều Khách hàng (Customer Dimension Attributes)**

- **Phân tích Tên và Địa chỉ (Name and Address Parsing):** Không nên sử dụng các cột chung chung (như Name-1, Address-1) vì chúng vô dụng cho việc phân tích và phân khúc. Tên và địa chỉ cần được phân tách thành các thành phần cơ bản (tiểu xưng, tên, họ, số nhà, thành phố, mã bưu điện...) để chuẩn hóa và làm sạch,. Đồng thời, cần cân nhắc vấn đề **quốc tế hóa (International Considerations)** bằng cách hỗ trợ bộ mã Unicode cho các bảng chữ cái khác nhau và lưu ý đến quy tắc dịch thuật (localization),.
- **Ngày tháng gắn với khách hàng (Customer-Centric Dates):** Các thông tin như "Ngày mua hàng đầu tiên" nên được mô hình hóa bằng cách dùng một khóa ngoại trỏ đến bảng chiều Ngày (Date Dimension) với vai trò như một Outrigger (bảng phụ).
- **Sự kiện Tổng hợp làm Thuộc tính (Aggregated Facts as Dimension Attributes):** Để dễ dàng lọc dữ liệu (ví dụ: tìm khách hàng đã chi tiêu trên một số tiền nhất định), các số đo tổng hợp (như tổng chi tiêu trọn đời) hoặc các nhãn phân loại (như "Người chi tiêu cao") nên được đưa trực tiếp vào bảng chiều dưới dạng thuộc tính hoặc các dải giá trị (value bands),.
- **Thuộc tính Phân khúc và Điểm số (Segmentation Attributes and Scores):** Phân khúc hành vi (ví dụ: khối RFI - Recency, Frequency, Intensity đo lường độ gần đây, tần suất và mức độ mua hàng) cần được chấm điểm và gán các nhãn (tags) hành vi theo chuỗi thời gian để tiện cho việc khai phá dữ liệu,.

**3. Kỹ thuật Outrigger và Cấu trúc Phân cấp**

- **Outrigger cho nhóm thuộc tính lực lượng thấp:** Khi có một khối dữ liệu lớn nhưng ít biến đổi (ví dụ: hàng trăm chỉ số nhân khẩu học của một quận/huyện), thay vì lặp lại dữ liệu này cho từng khách hàng, ta có thể tách chúng ra một bảng Outrigger riêng và gắn khóa ngoại vào bảng khách hàng. Tuy nhiên, tác giả cảnh báo không nên lạm dụng Outrigger (tránh biến thành mô hình Snowflake).
- **Phân cấp Khách hàng (Customer Hierarchy Considerations):** Đối với khách hàng doanh nghiệp, các phân cấp phức tạp (từ chi nhánh đến tổng công ty) nên được xử lý bằng cấu trúc phân cấp độ sâu cố định (nếu đơn giản) hoặc dùng **Bảng cầu nối (Bridge Tables)** nếu là phân cấp rách/độ sâu không xác định như đã trình bày ở Chương 7,,.

**4. Bảng Cầu nối (Bridge Tables) cho Chiều Đa trị**

- **Thuộc tính thưa thớt (Bridge Table for Sparse Attributes):** Khi hệ thống phải thu thập hàng trăm thuộc tính, nhưng mỗi khách hàng chỉ có vài thuộc tính (ví dụ: biểu mẫu ứng tuyển có các cặp Name-Value), việc tạo hàng trăm cột là không khả thi. Giải pháp là dùng bảng cầu nối để liên kết khách hàng với các cặp Giá trị - Tên (Name-Value pairs),,.
- **Nhiều liên hệ khách hàng (Multiple Customer Contacts):** Khi một khách hàng doanh nghiệp có nhiều người liên hệ với các vai trò khác nhau, một bảng cầu nối sẽ được đặt giữa bảng Chiều Khách hàng và bảng Chiều Người liên hệ để giải quyết mối quan hệ nhiều-nhiều,.

**5. Xử lý Hành vi Khách hàng Phức tạp (Complex Customer Behavior)**

- **Nhóm thuần tập/Nhóm nghiên cứu hành vi (Behavior Study Groups):** Đối với các truy vấn phức tạp (đòi hỏi xử lý nhiều bước để tìm ra một tập khách hàng cụ thể), kết quả tập hợp các khóa khách hàng đó nên được lưu lại thành một bảng "Study Group" tĩnh. Bảng này sau đó được dùng làm bộ lọc cho các bảng sự kiện khác mà không cần chạy lại truy vấn gốc,.
- **Chiều Bước cho các hành vi tuần tự (Step Dimension for Sequential Behavior):** Để phân tích các luồng hành vi theo từng bước (ví dụ: các thao tác nhấp chuột trên website), ta thêm một "Chiều Bước" vào bảng sự kiện, cho biết sự kiện hiện tại là bước thứ mấy và còn bao nhiêu bước nữa thì kết thúc chuỗi hành vi,.
- **Bảng sự kiện Khoảng thời gian (Timespan Fact Tables):** Để theo dõi một trạng thái kéo dài (ví dụ: khách hàng nằm trong danh sách cảnh báo gian lận), bảng sự kiện sẽ dùng một cặp ngày/giờ (Begin Effective và End Effective) để xác định chính xác khoảng thời gian trạng thái đó có hiệu lực,.
- **Gắn thẻ Bảng Sự kiện (Tagging Fact Tables):** Các bảng sự kiện có thể được bổ sung các chiều đặc biệt để ghi nhận **Chỉ số Hài lòng (Satisfaction Indicators)** (như chuyến bay bị trễ, mất hành lý) hoặc **Kịch bản Bất thường (Abnormal Scenario Indicators)** (như hàng hóa bị rớt lại) để giải thích lý do tại sao quy trình không đi theo kịch bản chuẩn,,.

**6. Tích hợp Dữ liệu Khách hàng (Customer Data Integration)** Tích hợp dữ liệu khách hàng từ nhiều nguồn là bài toán khó nhất.

- **Quản lý Dữ liệu Chính (Master Data Management - MDM):** Nếu doanh nghiệp có hệ thống MDM tập trung ở mức tác nghiệp, kho dữ liệu sẽ hưởng lợi. Nếu không, hệ thống ETL (Downstream MDM) phải tự gánh vác việc ghép nối (matching) và loại bỏ trùng lặp (deduplication) thông qua phần mềm chuyên dụng,.
- **Đồng nhất một phần (Partial Conformity):** Nếu không thể tích hợp toàn bộ, hãy bắt đầu bằng cách thêm các thuộc tính phân loại mức độ cao (như danh mục khách hàng) vào tất cả các bảng chiều khách hàng ở các hệ thống nguồn khác nhau để làm bước đệm,.

Tài liệu cũng nhắc nhở về việc **tránh Join trực tiếp Bảng Sự kiện với Bảng Sự kiện (Avoiding Fact-to-Fact Table Joins)** bằng cách sử dụng kỹ thuật Multipass SQL,, và cảnh báo về những sự đánh đổi về chất lượng dữ liệu khi yêu cầu thiết kế hệ thống có độ trễ thấp (Real-time/Low Latency).

# Chương 17: Kimball Lifecycle Overview

Chương 17 chuyển hướng từ các kỹ thuật mô hình hóa chi tiết sang cái nhìn toàn cảnh về **Vòng đời Kimball DW/BI (Kimball DW/BI Lifecycle)**,. Đây là một lộ trình chi tiết (Roadmap) quản lý dự án từ lúc phôi thai cho đến khi bảo trì và mở rộng, giúp đội ngũ DW/BI làm đúng việc, đúng thời điểm,.

Dưới đây là chi tiết các khái niệm và giai đoạn trong Vòng đời Kimball DW/BI:

**1. Các Nguyên tắc Cốt lõi của Vòng đời** Vòng đời Kimball dựa trên các nguyên lý nền tảng: Luôn tập trung vào nhu cầu của doanh nghiệp, cung cấp dữ liệu được cấu trúc theo mô hình chiều cho người dùng và thực hiện dự án theo hướng lặp lại (iterative), chia thành các phần có thể quản lý được thay vì làm một dự án khổng lồ.

**2. Hoạt động Khởi động Vòng đời (Lifecycle Launch Activities)** Giai đoạn này tập trung vào việc định hình và lấy yêu cầu cho dự án:

- **Lập kế hoạch và Quản lý Chương trình/Dự án:**
    - _Đánh giá mức độ sẵn sàng_ của tổ chức và xác định phạm vi dự án (scoping). Tài liệu cảnh báo về **"Quy luật Quá mức" (Law of Too)**: tránh một dự án có thời gian quá ngắn, quá nhiều nguồn dữ liệu, quá nhiều người dùng ở quá nhiều địa điểm với các yêu cầu quá đa dạng,.
    - _Tổ chức nhân sự:_ Yêu cầu một nhóm liên chức năng từ phía Nghiệp vụ (Nhà tài trợ - Sponsor, Người dẫn dắt - Driver, Người dùng cuối) và phía IT (Quản lý dự án, Kiến trúc sư kỹ thuật, Người mô hình hóa dữ liệu, Chuyên gia ETL/BI),,,,,,.
- **Xác định Yêu cầu Nghiệp vụ (Business Requirements Definition):**
    - Đây là hoạt động sống còn. Đội ngũ phải phỏng vấn người dùng kinh doanh và các chuyên gia dữ liệu hệ thống nguồn để hiểu quy trình nghiệp vụ thay vì chỉ hỏi họ "cần báo cáo gì",,.
    - _Ma trận ưu tiên (Prioritization Grid):_ Sau khi thu thập, các yêu cầu/quy trình sẽ được đánh giá trên một ma trận gồm hai trục: **Tác động Kinh doanh tiềm năng (Potential Business Impact)** và **Tính khả thi (Feasibility)**. Các dự án nằm ở góc trên cùng bên phải (Tác động cao - Khả thi cao) sẽ được chọn làm ưu tiên triển khai đầu tiên,.

**3. Ba Luồng Công việc Song song (Three Concurrent Tracks)** Sau khi chốt yêu cầu, dự án triển khai đồng thời 3 luồng công việc,,:

- **Luồng Công nghệ (Technology Track):**
    - _Thiết kế Kiến trúc Kỹ thuật:_ Đóng vai trò như bản thiết kế (blueprint) của một ngôi nhà, giúp chuẩn bị hạ tầng, tích hợp các dịch vụ ETL, BI và siêu dữ liệu (metadata),.
    - _Lựa chọn và Cài đặt Sản phẩm:_ Chọn mua các công cụ phần mềm dựa trên bản thiết kế kiến trúc. Tài liệu nhấn mạnh không bao giờ được chọn công cụ trước khi hiểu rõ yêu cầu kiến trúc,.
- **Luồng Dữ liệu (Data Track):**
    - _Mô hình hóa Đa chiều:_ Thiết kế mô hình logic thông qua các buổi hội thảo tương tác với người dùng,.
    - _Thiết kế Vật lý:_ Thiết lập các tiêu chuẩn đặt tên cơ sở dữ liệu, phân vùng và tạo các bảng tổng hợp (aggregations) - cách hiệu quả nhất để tăng tốc độ truy vấn,,,.
    - _Thiết kế & Phát triển ETL:_ Thường là phần việc tốn nhiều thời gian và công sức nhất, bao gồm trích xuất, làm sạch, đồng nhất và tải dữ liệu,.
- **Luồng Ứng dụng BI (BI Applications Track):**
    - Xây dựng một danh sách từ 10-15 báo cáo tiêu chuẩn, các ứng dụng phân tích và bảng điều khiển (dashboards) thiết yếu để đáp ứng nhu cầu cốt lõi của người dùng, giúp họ dễ dàng tiếp cận dữ liệu mà không cần phải tự viết các truy vấn phức tạp,.

**4. Hoạt động Triển khai, Bảo trì và Mở rộng (Wrap-up, Maintenance, and Growth)**

- **Triển khai (Deployment):** Điểm hội tụ của 3 luồng trên. Đòi hỏi phải có sự kiểm thử toàn hệ thống (end-to-end testing), đảm bảo chất lượng dữ liệu, thiết lập hệ thống hỗ trợ nhiều tầng và đào tạo kỹ lưỡng cho người dùng,,.
- **Bảo trì và Mở rộng (Maintenance and Growth):** Hỗ trợ người dùng liên tục, giám sát hiệu suất kỹ thuật,,. Khi hệ thống thành công sẽ kéo theo yêu cầu phân tích dữ liệu mới, lúc này dự án lại vòng về điểm xuất phát của Vòng đời Kimball để triển khai giai đoạn (iteration) tiếp theo trên Kiến trúc Bus.

**5. Top 10 Cạm bẫy Cần Tránh (Common Pitfalls to Avoid)** Chương 17 liệt kê các lỗi chí mạng có thể làm thất bại toàn bộ hệ thống DW/BI,,,:

- 10: Quá đam mê công nghệ và dữ liệu thay vì tập trung vào mục tiêu của doanh nghiệp.
- 9: Thiếu một nhà tài trợ từ phía doanh nghiệp (Business Sponsor) có tầm nhìn và quyền lực.
- 8: Thiết kế một dự án "tầm cỡ dải ngân hà" kéo dài nhiều năm thay vì chia nhỏ thành các chặng lặp lại dễ quản lý.
- 7: Dồn hết sức để xây dựng cấu trúc chuẩn hóa (3NF) nhưng hết ngân sách trước khi có thể xây dựng phần trình bày đa chiều cho người dùng.
- 6: Ưu tiên sự dễ dàng trong quá trình xử lý ETL phía sau hơn là ưu tiên hiệu suất truy vấn và tính thân thiện với người dùng ở phía trước.
- 2: Mặc định rằng kinh doanh, yêu cầu, dữ liệu và công nghệ là tĩnh (không thay đổi).
- 1: Quên mất rằng **thành công của DW/BI gắn liền trực tiếp với sự chấp nhận của người dùng**. Nếu họ không dùng nó để ra quyết định, mọi nỗ lực đều vô nghĩa.
# Chương 18: Dimensional Modeling Process and Tasks

Chương 18 đánh dấu một sự chuyển hướng quan trọng của tài liệu. Thay vì tiếp tục trình bày các kỹ thuật thiết kế mô hình chiều cho từng ngành nghề, chương này tập trung vào **quy trình, nhiệm vụ và chiến thuật** để thực hiện một dự án mô hình hóa đa chiều trong thực tế. Quá trình này không diễn ra theo một đường thẳng mà là một sự lặp lại liên tục (iterative process) từ cấp cao xuống chi tiết,.

Dưới đây là chi tiết các khái niệm và các bước trong Quy trình Mô hình hóa Chiều (Dimensional Modeling Process) được trình bày ở Chương 18:

**1. Tổng quan và Thành phần tham gia (Modeling Process Overview)**

- **Không làm việc cô lập:** Người thiết kế dữ liệu (data modeler) là người dẫn dắt, nhưng tuyệt đối không được tự thiết kế mô hình một mình,.
- **Sự tham gia của doanh nghiệp:** Bắt buộc phải có sự tham gia của đại diện phía người dùng doanh nghiệp (business representatives) và chuyên gia quản trị dữ liệu (data stewards) để cung cấp ngữ cảnh, giải thích quy tắc nghiệp vụ và thúc đẩy sự đồng thuận chung,.
- **Đầu vào và Đầu ra:** Quá trình bắt đầu với đầu vào là Ma trận Bus sơ bộ cùng tài liệu yêu cầu nghiệp vụ. Đầu ra cốt lõi bao gồm: Mô hình chiều cấp cao, thiết kế bảng chiều/sự kiện chi tiết, và danh sách các vấn đề (issues log).

**2. Xem xét Yêu cầu Nghiệp vụ và Thực tế Dữ liệu (Review Business Requirements & Data Realities)**

- Trước khi bắt tay vào thiết kế, nhóm phải hiểu rõ các mục tiêu, quy trình kinh doanh và chỉ số hiệu suất của tổ chức.
- **Hồ sơ hóa dữ liệu (Data Profiling):** Đây là một bước phân tích kỹ thuật bắt buộc để khám phá nội dung, cấu trúc, và các mối quan hệ thực tế trong hệ thống dữ liệu nguồn (thường thông qua các câu lệnh SQL hoặc công cụ chuyên dụng),. Mục tiêu là để xác minh xem dữ liệu nguồn có khả thi để sử dụng hay không, những lỗi nào có thể kiểm soát được, và hệ thống ETL sẽ cần phải chuyển đổi những gì.
- Thiết lập và thống nhất các tiêu chuẩn đặt tên (naming conventions) cho toàn bộ mô hình.

**3. Thiết kế Mô hình Chiều Cấp cao (High-Level Dimensional Model)**

- Dựa trên Ma trận Bus, nhóm thiết kế tạo ra một bản phác thảo ban đầu ở mức độ thực thể. Công cụ thường dùng là **Biểu đồ bong bóng (Bubble chart)**, một sơ đồ đồ họa trực quan bao gồm một bảng sự kiện ở giữa được bao quanh bởi các bảng chiều,.
- Bản phác thảo này giúp xác định và tuyên bố rõ ràng **mức độ hạt (grain)** của bảng sự kiện và các chiều đi kèm.
- Biểu đồ bong bóng là công cụ giao tiếp tuyệt vời cho những người không chuyên về kỹ thuật và giúp nhóm thiết kế thống nhất hướng đi trước khi bị sa lầy vào các chi tiết vụn vặt.

**4. Phát triển Mô hình Chiều Chi tiết (Detailed Dimensional Model Development)**

- Sau khi mô hình cấp cao được duyệt, nhóm bắt đầu đi sâu vào việc định nghĩa từng thuộc tính, từng cột của các bảng chiều và bảng sự kiện,.
- **Bảng tính Thiết kế (Design Worksheets):** Đây là tài liệu giao chéo quan trọng nhất. Mỗi bảng chiều/sự kiện sẽ có một bảng tính ghi rõ: tên cột, loại dữ liệu, mô tả, phương pháp xử lý chiều thay đổi chậm (SCD type 1, 2, 3...).
- **Ánh xạ Nguồn đến Đích (Source-to-Target Mapping):** Nằm ngay trong bảng tính thiết kế, phần này cung cấp chỉ dẫn cho đội ngũ ETL, cho biết mỗi thuộc tính được lấy từ hệ thống nguồn nào, bảng nào, trường nào và quy tắc chuyển đổi (ETL rules) áp dụng ra sao,.
- Trong quá trình này, nếu phát hiện ra các bảng sự kiện, bảng chiều mới hoặc có sự thay đổi, **Ma trận Bus phải được cập nhật** liên tục vì đây là công cụ giao tiếp và lập kế hoạch cốt lõi,.

**5. Đánh giá và Xác nhận Mô hình (Review and Validate the Model)**

- Khi nhóm thiết kế đã tự tin với mô hình, họ phải tiến hành đánh giá lại với những bên liên quan, đặc biệt là người dùng doanh nghiệp.
- **Chiến thuật đánh giá:** Tránh làm người dùng bị choáng ngợp bởi quá nhiều chi tiết kỹ thuật. Nên bắt đầu bằng bức tranh tổng thể (Ma trận Bus), đi vào Biểu đồ bong bóng, và sau đó chỉ xem xét sâu các chiều quan trọng nhất (như Khách hàng, Sản phẩm),.
- Cần minh họa trực quan các đường dẫn phân cấp (hierarchical drill paths) trong bảng chiều để người dùng dễ hình dung (ví dụ: Phòng ban -> Danh mục -> Thương hiệu -> Sản phẩm),.
- **Kiểm chứng bằng câu hỏi thực tế:** Lấy các câu hỏi nghiệp vụ từ tài liệu yêu cầu ban đầu và mô phỏng cách mô hình sẽ trả lời các câu hỏi đó. Điều này giúp chứng minh giá trị của hệ thống DW/BI.

**Kết luận chương:** Quá trình thiết kế mô hình đa chiều kết thúc bằng việc hoàn thiện bộ tài liệu thiết kế. Mục tiêu cuối cùng là tạo ra một mô hình vừa đáp ứng đúng yêu cầu kinh doanh, vừa khả thi về mặt dữ liệu nguồn, đồng thời cung cấp một bản thiết kế rõ ràng, chi tiết để chuyển giao cho đội ngũ phát triển hệ thống ETL (được trình bày ở các chương tiếp theo).

# Chương 19: ETL Subsystems and Techniques

Chương 19 tập trung vào **Hệ thống Trích xuất, Chuyển đổi và Tải (ETL Subsystems and Techniques)**. Quá trình xây dựng hệ thống ETL thường tiêu tốn một phần thời gian và công sức lớn nhất trong việc xây dựng môi trường DW/BI. Chương này định nghĩa 34 hệ thống con (subsystems) thiết yếu thường có mặt trong khu vực xử lý nền (back room) của một kho dữ liệu đa chiều.

**1. Thu thập Yêu cầu và Ràng buộc (Round Up the Requirements)** Trước khi thiết kế hệ thống ETL, kiến trúc sư phải xem xét các yêu cầu và ràng buộc cốt lõi:

- **Nhu cầu kinh doanh (Business Needs):** Xác định các chỉ số hiệu suất cốt lõi (KPI) và các mục tiêu kết hợp/đào sâu dữ liệu (drill-down/drill-across) cần hỗ trợ.
- **Tuân thủ (Compliance):** Các yêu cầu về tính pháp lý và báo cáo đòi hỏi hệ thống không được phép thay đổi, mà phải bảo toàn nghiêm ngặt lịch sử để có thể kiểm toán được.
- **Chất lượng dữ liệu (Data Quality):** Đảm bảo độ chính xác và tin cậy của dữ liệu.
- **Bảo mật, Tích hợp và Giao diện (Security, Data Integration, BI Delivery Interfaces):** Cung cấp các giao diện phân phối dữ liệu cho các công cụ BI và đảm bảo khả năng tích hợp, bảo mật.
- **Độ trễ dữ liệu (Data Latency):** Xác định tần suất tải dữ liệu (ví dụ: chạy theo lô hàng ngày, hay yêu cầu dữ liệu độ trễ thấp/thời gian thực).
- **Lưu trữ và Nguồn gốc (Archiving and Lineage):** Lưu trữ dữ liệu theo thời gian và khả năng truy vết quá trình biến đổi của dữ liệu.

**2. Các Nhóm Hệ thống con của ETL (34 ETL Subsystems)** Chương 19 phân chia 34 hệ thống con thành 4 nhóm chức năng chính:

**A. Nhóm Trích xuất (Extracting - 3 hệ thống con)** Tập trung vào việc đọc, hiểu và đưa dữ liệu từ các hệ thống nguồn vào môi trường ETL. Hệ thống phải xử lý nhiều định dạng phức tạp như CSDL quan hệ (RDBMS), tệp văn bản (flat files), XML, nhật ký web (web logs) hoặc các hệ thống ERP phức tạp, thậm chí cả mã ngôn ngữ thủ tục từ các hệ thống cũ (legacy).

**B. Nhóm Làm sạch và Đồng nhất (Cleaning and Conforming - 5 hệ thống con)**

- **Hệ thống làm sạch dữ liệu (Data Cleansing System):** Sử dụng các bộ lọc (quality screens) để kiểm tra và phát hiện lỗi dữ liệu.
- **Lược đồ sự kiện lỗi (Error Event Schema):** Bất cứ khi nào bộ lọc phát hiện lỗi, sự kiện này được ghi lại vào một bảng sự kiện lỗi đặc biệt chỉ tồn tại trong khu vực ETL, đi kèm với bảng chi tiết lỗi ghi nhận chính xác bảng và cột nào vi phạm.
- **Bộ lắp ráp Chiều Kiểm toán (Audit Dimension Assembler):** Gắn một "Chiều kiểm toán" vào các dòng của bảng sự kiện tại thời điểm dữ liệu được tạo ra, dùng để lưu trữ các siêu dữ liệu (metadata) về chất lượng luồng dữ liệu hoặc phiên bản ETL đã được sử dụng.
- **Hệ thống Loại bỏ trùng lặp và Đồng nhất (Deduplication & Conforming System):** Xử lý việc loại bỏ các bản ghi trùng lặp và tích hợp dữ liệu từ nhiều nguồn để tạo ra các Chiều đồng nhất (Conformed Dimensions) và Thước đo đồng nhất (Conformed Facts).

**C. Nhóm Cung cấp/Tải dữ liệu (Delivering - 13 hệ thống con)** Mục tiêu chính của nhóm này là cấu trúc vật lý và tải dữ liệu vào các mô hình chiều ở khu vực trình bày.

- **Quản lý Chiều thay đổi chậm (SCD Manager):** Áp dụng logic cho các thay đổi thuộc tính chiều (như Type 1, 2, 3, 4, 5, 6, 7) và quản lý các cột housekeeping như ngày hiệu lực, ngày hết hạn và cờ báo trạng thái hiện tại.
- **Tạo và Gắn Khóa thay thế (Surrogate Key Generator & Pipeline):** Tạo các khóa nguyên vô nghĩa thay cho khóa tự nhiên. Quá trình tra cứu khóa này (Surrogate Key Pipeline) phải giải quyết được các lỗi toàn vẹn tham chiếu nếu không tìm thấy khóa hợp lệ tương ứng với dữ kiện.
- **Quản lý Cấu trúc phân cấp và Chiều đặc biệt (Hierarchy & Special Dimensions Manager):** Xử lý các phân cấp độ sâu biến đổi (ragged hierarchies) và các chiều đặc biệt như chiều rác (junk dimensions), chiều tĩnh (static dimensions) hoặc chiều thu nhỏ (shrunken subset dimensions).
- **Xây dựng Bảng Sự kiện và Bảng Cầu nối (Fact Table & Multivalued Bridge Table Builders):** Nạp dữ liệu vào bảng sự kiện và xử lý các mối quan hệ đa trị phức tạp (nhiều-nhiều) thông qua bảng cầu nối.
- **Xử lý Dữ liệu Đến muộn (Late Arriving Data Handler):** Xử lý các sự kiện thực tế đến trễ hơn so với thời điểm xảy ra, đòi hỏi hệ thống ETL phải tìm kiếm lùi lại lịch sử để gán đúng khóa chiều có hiệu lực tại thời điểm đó.
- **Xây dựng Dữ liệu tổng hợp và Khối OLAP (Aggregate & OLAP Cube Builder):** Tạo các bảng tổng hợp và khối OLAP, đây là giải pháp quan trọng nhất để tăng tốc độ truy vấn cho toàn bộ DW/BI.
- **Quản lý Phân phối dữ liệu (Data Propagation Manager):** Trích xuất dữ liệu đã đồng nhất từ khu vực trình bày của kho dữ liệu để phân phối cho các đối tác, khách hàng hoặc chuyển đổi định dạng cung cấp cho các công cụ khai phá dữ liệu (data mining).

**D. Nhóm Quản lý Môi trường ETL (Managing - 13 hệ thống con)** Đảm bảo hệ thống ETL hoạt động ổn định, có thể tự động hóa và đáng tin cậy.

- **Lập lịch công việc và Giám sát luồng xử lý (Job Scheduler & Workflow Monitor):** Tự động hóa, quản lý lịch trình và giám sát thứ tự thực hiện của các tiến trình ETL.
- **Sao lưu, Phục hồi và Khởi động lại (Backup, Recovery and Restart System):** Sao lưu vật lý toàn bộ môi trường và tạo ra cơ chế phục hồi hệ thống khi có sự cố một cách an toàn.
- **Kiểm soát Phiên bản (Version Control System):** Lưu lại các bản chụp (snapshots) của logic và siêu dữ liệu ETL, hỗ trợ so sánh sự khác biệt giữa các phiên bản và khôi phục khi cần thiết.
- **Hệ thống Sắp xếp (Sorting System):** Hỗ trợ việc sắp xếp trong cơ sở dữ liệu để tối ưu tốc độ ETL, tránh tình trạng xử lý dữ liệu bị chậm hoặc quá tải RAM.
- **Theo dõi Nguồn gốc và Phụ thuộc (Lineage and Dependency Tracking):** Cho phép truy xuất ngược nguồn gốc của một trường dữ liệu từ báo cáo BI về lại hệ thống nguồn (lineage), và theo chiều xuôi, xác định những thành phần nào sẽ bị ảnh hưởng nếu dữ liệu tại hệ thống nguồn thay đổi (dependency).

# Chương 20: ETL System Design and Development Process and Tasks

Chương 20 của tài liệu tập trung vào **Quy trình và Các tác vụ Thiết kế, Phát triển Hệ thống ETL (ETL System Design and Development Process and Tasks)**. Nếu Chương 19 giới thiệu 34 hệ thống con về mặt kiến trúc, thì Chương 20 cung cấp một kế hoạch thực tế gồm 10 bước để trực tiếp xây dựng hệ thống ETL.

Dưới đây là chi tiết các khái niệm và vấn đề được trình bày trong Chương 20:

**1. Lập kế hoạch và Thiết kế ETL (ETL System Planning and Design)** Quá trình phát triển ETL bắt đầu bằng một kế hoạch cấp cao dựa trên các tài liệu từ khâu thiết kế vật lý và ánh xạ nguồn-đích (source-to-target mapping).

- **Bước 1: Lập sơ đồ kế hoạch cấp cao (Draw the High-Level Plan):** Vẽ một sơ đồ trực quan về luồng dữ liệu từ nguồn đến đích để giải quyết các vấn đề liên quan đến việc trích xuất và lưu trữ tạm (staging).
- **Bước 2: Chọn công cụ ETL (Choose an ETL Tool):** Đánh giá xem nên mua công cụ ETL chuyên dụng hay tự viết mã (hand-coding) dựa trên sự phức tạp của biến đổi dữ liệu, chất lượng dữ liệu và chi phí.
- **Bước 3: Xây dựng các chiến lược mặc định (Develop Default Strategies):** Đặt ra các nguyên tắc chung cho hệ thống, chẳng hạn như: chiến lược trích xuất (lưu trữ lịch sử, trích xuất toàn bộ hay tăng dần), chiến lược lưu trữ (archiving), kiểm soát chất lượng dữ liệu, và cách xử lý chiều thay đổi chậm (SCD).
- **Bước 4: Khoan sâu vào các lược đồ bảng (Develop Detailed Table Schematics):** Xác minh dữ liệu đối với các bảng chiều có cấu trúc phân cấp (ví dụ: Product -> Subcategory -> Category). Tài liệu khuyên nên kiểm tra tính hợp lệ của quan hệ nhiều-một (many-to-one) ngay từ bước chuẩn bị, nhưng không nhất thiết phải chuẩn hóa (snowflake) dữ liệu trong khu vực staging ETL nếu hệ thống nguồn đã được chuẩn hóa.
- **Bước 5: Phát triển Đặc tả Hệ thống ETL (Develop the ETL Specification Document):** Tổng hợp tất cả mọi quy tắc thành một tài liệu chi tiết duy nhất, bao gồm: tần suất tải, khối lượng dữ liệu, cách xử lý dữ liệu đến muộn, cách phân vùng, và quy tắc lập bản đồ (mapping) chi tiết.

**2. Tải Dữ liệu Lịch sử Một lần (One-Time Historic Data Load)**

- **Bước 6: Tải bảng chiều (Populate Dimension Tables with Historic Data):** Tải dữ liệu lịch sử vào bảng chiều trước tiên. Quá trình này bao gồm việc gán **Khóa thay thế (Surrogate Key Assignment)** (thường dùng bộ tạo số nguyên tuần tự) để thay thế khóa tự nhiên. Bạn cũng phải cố gắng áp dụng các quy tắc về lịch sử (như SCD Type 2) cho dữ liệu trong quá khứ nếu hệ thống nguồn có lưu lại lịch sử thay đổi.
- **Bước 7: Tải bảng sự kiện (Fact Table Historic Load):** Sau khi đã có bảng chiều, dữ liệu bảng sự kiện mới được tải.
    - **Chuyển đổi thực tế:** Dữ liệu sự kiện có thể yêu cầu tính toán các số đo phái sinh (derived facts) hoặc chuyển đổi đơn vị. Cần chú ý cẩn thận với các sự kiện ở mức hạt độ khác nhau bị trộn lẫn.
    - **Quy trình khóa thay thế (Surrogate Key Pipeline):** Hệ thống ETL tra cứu khóa tự nhiên từ dữ liệu nguồn vào bảng chiều để lấy ra khóa thay thế tương ứng và đưa vào bảng sự kiện. Với tải lịch sử, phải dùng logic "BETWEEN" trên ngày tháng (row effective/expiration date) để đảm bảo gán đúng phiên bản khóa của bảng chiều tại thời điểm xảy ra sự kiện.
    - Bổ sung thêm khóa nối với **Chiều kiểm toán (Audit Dimension)** để ghi nhận siêu dữ liệu về quá trình chạy ETL (ví dụ: cờ báo lỗi, phiên bản logic).

**3. Xử lý Tải Tăng dần (Incremental Processing)** Khi hệ thống đã chạy, việc cập nhật diễn ra thông qua tải tăng dần (incremental load).

- **Bước 8 & 9: Xử lý tăng dần cho Bảng chiều và Bảng sự kiện:** Thay vì thay thế toàn bộ dữ liệu, hệ thống chỉ cập nhật những gì thay đổi.
    - Hệ thống xác định xem dòng từ nguồn là mới tạo, thay đổi thuộc tính (chạy logic SCD Type 1, 2, 3...) hay không có gì thay đổi.
    - Đối với bảng sự kiện, dữ liệu được kết nối với các chiều. Nếu khóa tự nhiên bị lỗi hoặc chưa tồn tại trong bảng chiều (lỗi toàn vẹn tham chiếu), hệ thống ETL thay vì làm treo luồng dữ liệu, nên sinh ra một "dummy dimension row" (dòng chiều tạm) hoặc ghi vào bảng sự kiện lỗi để xử lý sau. Cần quan tâm đặc biệt tới **dữ liệu đến muộn (Late Arriving Data)** để lùi lại và cập nhật khóa chính xác mà không phá hỏng các báo cáo đã xuất bản.

**4. Vận hành và Tự động hóa (System Operation and Automation)**

- **Bước 10: Vận hành tự động:** Mục tiêu tối thượng của hệ thống ETL là có thể tự động chạy "lights-out" (không cần sự can thiệp của con người). Hệ thống phải lên lịch thực thi công việc (Job Scheduler) theo đúng trình tự phụ thuộc và có khả năng bẫy lỗi, xử lý ngoại lệ và gửi cảnh báo.

**5. Đánh đổi trong Kho dữ liệu Thời gian thực (Real-Time Data Warehousing Considerations)** Cuối chương, tác giả đi sâu vào yêu cầu cung cấp dữ liệu có độ trễ thấp (low latency) hay thời gian thực:

- **Sự đánh đổi về chất lượng dữ liệu:** Khi yêu cầu tải dữ liệu chuyển từ chạy lô (batch) hàng ngày sang tải vi mô (micro-batch) hoặc luồng trực tiếp (streaming/real-time), các quy trình dọn dẹp, kiểm tra chéo và đồng nhất dữ liệu chuyên sâu sẽ bị bỏ qua vì không đủ thời gian. Dữ liệu "nóng" có thể thiếu sót hoặc sai lệch cho đến khi quá trình chạy batch cuối ngày ghi đè để chỉnh sửa lại (reconcile).
- **Phân vùng thời gian thực (Real-Time Partitions):** Để giải quyết vấn đề hiệu suất và cập nhật liên tục, dữ liệu ngay lúc này được nạp vào một phân vùng bộ nhớ đệm đặc biệt (hot partition). Các phân vùng thời gian thực này thường nằm trên RAM (không có aggregate hay index để tải cực nhanh). Đến cuối chu kỳ (ví dụ cuối ngày/tháng), phân vùng này được hợp nhất (merge) vào bảng sự kiện lịch sử cố định.

# Chương 21: Big Data Analytics

Chương 21 của tài liệu bàn về chủ đề **Phân tích Dữ liệu lớn (Big Data Analytics)**. Tác giả xem dữ liệu lớn không phải là một lĩnh vực hoàn toàn tách biệt, mà là một sự mở rộng tự nhiên của sứ mệnh Kho dữ liệu và Trí tuệ doanh nghiệp (DW/BI). Dù công nghệ có thay đổi, các nguyên tắc cốt lõi đã được đúc kết trong suốt 30 năm qua vẫn hoàn toàn có giá trị đối với Big Data.

Dưới đây là chi tiết các khái niệm và vấn đề được trình bày trong Chương 21, phân chia theo các nhóm thực hành tốt nhất (Best Practices):

**1. Tổng quan về Dữ liệu lớn (Big Data Overview)**

- Dữ liệu lớn mang đến các phân tích ở quy mô khổng lồ và độ phức tạp cao, chẳng hạn như: phân tích hệ gen, theo dõi nhóm thuần tập, trạng thái máy bay đang bay, đồng hồ đo điện thông minh, cảm biến tòa nhà, phân tích gian lận tài chính, hay phân tích sự rời bỏ của khách hàng (churn analysis).
- Các hệ thống quản trị cơ sở dữ liệu quan hệ (RDBMS) hiện tại phải mở rộng để xử lý phạm vi dữ liệu đa dạng hơn, bao gồm các cấu trúc phức tạp như vector, ma trận, dữ liệu văn bản phi cấu trúc, hình ảnh, và cặp tên-giá trị (name-value pairs).

**2. Thực hành tốt nhất về Quản lý (Management Best Practices)**

- **Trì hoãn việc xây dựng các hệ thống di sản (Delay Building Legacy Environments):** Môi trường dữ liệu lớn đang thay đổi quá nhanh. Do đó, tổ chức chưa nên vội vàng cố định một hệ thống cốt lõi mang tính lâu dài ngay thời điểm này.
- **Quản lý kết quả thử nghiệm (Sandbox results) và Khai tử hệ thống (Sunsetting):** Cần có quy trình rõ ràng cho các môi trường thử nghiệm dữ liệu lớn và kế hoạch ngừng hoạt động các hệ thống lỗi thời.

**3. Thực hành tốt nhất về Kiến trúc (Architecture Best Practices)**

- **Quy hoạch Xa lộ Dữ liệu (Data Highway Planning):** Tác giả đề xuất một kiến trúc gồm 5 bộ đệm dữ liệu (caches) với mức độ trễ (latency) và chất lượng dữ liệu tăng dần:
    1. **Raw Source (Tức thời):** Dữ liệu thô dùng để can thiệp ngay lập tức (ví dụ: phát hiện gian lận thẻ tín dụng).
    2. **Real Time Cache (Tính bằng giây):** Phục vụ các báo cáo hoạt động tức thời.
    3. **Business Activity Cache (Tính bằng phút):** Theo dõi các hoạt động đang diễn ra.
    4. **Top Line Cache (24 giờ):** Cung cấp bức tranh tổng quan trong ngày cho quản lý cấp cao.
    5. **DW and Long Time Series (Hàng ngày/Định kỳ):** Dành cho báo cáo truyền thống, phân tích lịch sử và khai phá dữ liệu.
- **Dòng chảy ngược về các bộ đệm trước (Implement Backflow to Earlier Caches):** Dữ liệu chiều đã được quản trị kỹ lưỡng (như Khách hàng, Sản phẩm) và các khóa thay thế bền vững (durable keys) từ Kho dữ liệu nên được "đẩy ngược" lên các bộ đệm ở hệ thống nguồn. Điều này giúp các phân tích ở lớp dữ liệu "nóng" (độ trễ thấp) có thể kết nối ngay lập tức với các thuộc tính phong phú của hệ thống DW/BI.
- **Khai thác phân tích ngay trong cơ sở dữ liệu (Exploit In-Database Analytics):** Tận dụng tối đa khả năng mở rộng của các RDBMS (như các hàm UDF - User Defined Functions) để kết hợp sức mạnh của SQL với các phân tích tính toán trực tiếp bên trong CSDL, thay vì phải trích xuất dữ liệu ra ngoài.

**4. Thực hành tốt nhất về Mô hình hóa Dữ liệu (Data Modeling Best Practices)** Tác giả nhấn mạnh rằng các quy tắc mô hình chiều kinh điển vẫn cực kỳ quan trọng đối với Big Data:

- **Tư duy đa chiều (Think Dimensionally):** Luôn chia thế giới thành các Chiều (Dimensions - ngữ cảnh) và Sự kiện (Facts - con số đo lường) vì người dùng doanh nghiệp thấy cách tổ chức này trực quan và dễ hiểu nhất, bất kể định dạng dữ liệu là gì.
- **Neo các chiều bằng Khóa thay thế bền vững (Anchor Dimensions with Durable Surrogate Keys):** Tuyệt đối không dùng khóa tự nhiên của hệ thống nguồn vì chúng rất dễ trùng lặp hoặc không tương thích. Khóa thay thế bền vững là nền tảng để kết nối dữ liệu đa nguồn.
- **Tích hợp dữ liệu có cấu trúc và phi cấu trúc:** Dữ liệu lớn có thể không bao giờ được đưa hết vào CSDL quan hệ mà nằm ở nền tảng Hadoop hoặc Grid. Dù vậy, chúng vẫn cần được tích hợp với nhau thông qua các chiều đồng nhất (conformed dimensions).
- **Sử dụng Chiều thay đổi chậm (Use Slowly Changing Dimensions - SCD):** Quản lý sự thay đổi của dữ liệu theo thời gian vẫn là một thực hành thiết yếu không thể bỏ qua trong thế giới Big Data.
- **Khai báo cấu trúc tại thời điểm phân tích (Declare Data Structure at Analysis Time):** Đối với dữ liệu lớn/phi cấu trúc, đôi khi người dùng không nên cố gắng ép dữ liệu vào một cấu trúc cứng nhắc ngay từ đầu, mà chỉ định hình cấu trúc (late binding/data virtualization) khi truy vấn và phân tích.

**5. Thực hành tốt nhất về Quản trị Dữ liệu (Data Governance Best Practices)**

- **Quyền riêng tư (Privacy):** Việc thu thập và phân tích dữ liệu lớn mang lại rủi ro rất cao về quyền riêng tư. Quản trị dữ liệu phải đặt vấn đề bảo vệ thông tin lên hàng đầu.
- **Mô hình hóa đa chiều (Dimensionalizing):** Việc tích hợp dữ liệu lớn đòi hỏi các nhà quản trị dữ liệu phải nỗ lực tiêu chuẩn hóa các thuộc tính để hệ thống vẫn có thể ghép nối và so sánh được với dữ liệu của tổ chức.