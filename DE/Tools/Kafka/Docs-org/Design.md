- [[#Motivation|Motivation]]
- [[#Persistence|Persistence]]
	- [[#Persistence#Don’t fear the filesystem!|Don’t fear the filesystem!]]
	- [[#Persistence#Constant Time Suffices|Constant Time Suffices]]
- [[#Efficiency|Efficiency]]
	- [[#Efficiency#End-to-end Batch Compression|End-to-end Batch Compression]]
- [[#The Producer|The Producer]]
- [[#Consumer|Consumer]]
- [[#Message Delivery Semantics|Message Delivery Semantics]]
- [[#Using Transactions|Using Transactions]]
- [[#The Share Consumer|The Share Consumer]]
- [[#Replication|Replication]]
- [[#Log Compaction|Log Compaction]]
- [[#Quotas|Quotas]]

## Motivation

Kafka được thiết kế để trở thành một nền tảng thống nhất xử lý tất cả các luồng dữ liệu thời gian thực cho một doanh nghiệp lớn. Điều này đòi hỏi hệ thống phải có **thông lượng cao** (để gom cụm log thời gian thực), khả năng **xử lý mượt mà khối lượng dữ liệu tồn đọng lớn**, **độ trễ thấp** cho các tin nhắn truyền thống, tính **phân tán và chịu lỗi** cao. Các đặc điểm này khiến kiến trúc của Kafka giống một "database log" hơn là một hệ thống nhắn tin (messaging system) thông thường.

## Persistence

### Don’t fear the filesystem!

**Tận dụng hệ thống tệp (Filesystem) & OS Pagecache:** Nhiều người cho rằng "ổ cứng là chậm", nhưng thực tế tốc độ ghi tuần tự (linear writes) trên ổ cứng có thể cực kỳ nhanh (ví dụ: lên đến 600MB/s). Thay vì duy trì bộ nhớ đệm trong bộ nhớ RAM của ứng dụng (in-memory cache) dẫn đến tốn kém không gian bộ nhớ của các đối tượng Java và làm quá tải trình thu gom rác (Garbage Collection), Kafka ghi dữ liệu thẳng xuống ổ đĩa và chuyển trực tiếp vào bộ nhớ đệm cấp hệ điều hành (kernel's pagecache).

### Constant Time Suffices

**Hiệu năng không đổi O(1) (Constant Time):** Hệ thống tin nhắn thông thường thường sử dụng BTree (có độ phức tạp thời gian O(log N)) để duy trì metadata, dẫn đến tốc độ giảm đáng kể khi dữ liệu lớn. Kafka lưu trữ dữ liệu dưới dạng các tệp log và chỉ thao tác đọc/ghi tuần tự ở cuối tệp (append). Việc thao tác O(1) giúp hiệu suất hoàn toàn độc lập với kích thước dữ liệu. Nhờ đó, Kafka có thể lưu trữ thông tin trong thời gian dài (ví dụ: một tuần) thay vì xóa ngay sau khi được tiêu thụ.

## Efficiency

**Gom nhóm (Batching):** Để khắc phục độ trễ mạng và các thao tác I/O nhỏ lẻ, Kafka gom nhiều tin nhắn lại thành các khối lớn (message sets) trước khi gửi qua mạng hoặc ghi vào ổ đĩa.

**Định dạng nhị phân & Zero-Copy:** Kafka duy trì định dạng dữ liệu nhị phân thống nhất từ Producer, Broker đến Consumer. Khi truyền dữ liệu đến Consumer, Kafka tận dụng cơ chế gọi hệ thống `sendfile` trên Linux để chuyển thẳng dữ liệu từ pagecache sang card mạng (socket) mà không cần sao chép trung gian qua user-space, giúp tối đa hóa tốc độ tiêu thụ tiệm cận giới hạn băng thông mạng. 

### End-to-end Batch Compression

Trong một số trường hợp, điểm nghẽn thực sự không phải là CPU hay ổ đĩa mà là băng thông mạng.

**Nén dữ liệu (Compression):** Kafka nén các đợt tin nhắn (batch compression) bằng các thuật toán GZIP, Snappy, LZ4 hoặc ZStandard. Dữ liệu được nén từ Producer, giữ nguyên dạng nén trên Broker (ổ cứng) và chỉ được giải nén khi tới Consumer.

## The Producer

1. Cân bằng tải (Load balancing) và Phân vùng (Partitioning)

- **Gửi trực tiếp không qua trung gian:** Producer trong Kafka gửi dữ liệu trực tiếp đến Broker đang đóng vai trò là "Leader" của partition mà nó muốn gửi, hoàn toàn không đi qua bất kỳ một tầng định tuyến (routing tier) trung gian nào.
- **Tự định tuyến thông minh:** Để biết được Leader của partition đang nằm ở đâu, Producer có thể gửi yêu cầu lấy thông tin metadata đến bất kỳ node (broker) nào trong cụm Kafka, vì tất cả các node đều có khả năng trả lời thông tin về các server đang hoạt động và vị trí của các Leader.
- **Quyền kiểm soát thuộc về Client:** Khách hàng (client) hoàn toàn kiểm soát việc tin nhắn sẽ được xuất bản (publish) vào partition nào.
- **Phân vùng theo ngữ nghĩa (Semantic Partitioning):** Việc chọn partition có thể diễn ra ngẫu nhiên để cân bằng tải, nhưng Kafka cũng cho phép người dùng chỉ định một **khóa (key)** để băm (hash) và ánh xạ tới một partition cụ thể. Ví dụ: nếu bạn chọn _User ID_ làm khóa, tất cả các sự kiện của cùng một người dùng sẽ luôn được gửi vào chung một partition. Điều này cho phép Consumer khi đọc dữ liệu có thể đưa ra các giả định về tính cục bộ (locality) để xử lý theo đúng thứ tự.

2. Gửi bất đồng bộ và Gom nhóm (Asynchronous send & Batching)

- Việc gom lô (batching) là một trong những động lực lớn nhất mang lại hiệu suất cao cho Kafka. Thay vì gửi từng tin nhắn rời rạc, Producer sẽ cố gắng tích lũy dữ liệu trong bộ nhớ (memory) và gửi chúng đi thành các lô lớn hơn trong một request duy nhất.
- Quá trình này có thể được cấu hình linh hoạt: Producer có thể đợi cho đến khi tích lũy đủ một số lượng tin nhắn tối đa nhất định (ví dụ: 64kB) hoặc không đợi lâu hơn một giới hạn thời gian (latency bound) nhất định (ví dụ: 10ms).
- Bằng cách đánh đổi một lượng rất nhỏ độ trễ, cơ chế này cho phép Kafka gửi đi số lượng byte lớn hơn, dẫn đến các thao tác I/O trên server lớn hơn nhưng ít đi, từ đó tăng thông lượng (throughput) lên đáng kể.

3. Tùy chọn Độ bền vững (Durability) và Phản hồi (Acks) 

Không phải mọi trường hợp sử dụng (use case) đều yêu cầu đảm bảo dữ liệu khắt khe giống nhau. Kafka cho phép Producer tự điều chỉnh mức độ bền vững mong muốn thông qua cấu hình `acks` để đánh đổi giữa độ trễ và độ an toàn của dữ liệu:

- **acks=0:** Producer có thể gửi tin nhắn hoàn toàn bất đồng bộ và không cần đợi phản hồi, giúp tối ưu độ trễ.
- **acks=1:** Producer chỉ đợi cho đến khi broker Leader nhận được tin nhắn (không chờ các Follower đồng bộ).
- **acks=all (hoặc -1):** Producer sẽ đợi cho đến khi tin nhắn được cam kết (committed) vào toàn bộ các bản sao đồng bộ (ISR - In-Sync Replicas). Quá trình này có thể mất khoảng 10ms nhưng đảm bảo tin nhắn không bao giờ bị mất miễn là còn ít nhất một bản sao trong ISR sống sót.

4. Đảm bảo giao nhận tin (Message Delivery Semantics)

- **Sự cố trùng lặp (At-least-once):** Khi Producer gửi một tin nhắn và gặp lỗi mạng, nó sẽ không biết liệu lỗi xảy ra _trước_ hay _sau_ khi tin nhắn đã được ghi nhận (committed) trên Broker. Trước bản phát hành 0.11.0.0, Producer buộc phải gửi lại tin nhắn, dẫn đến việc tin nhắn có thể bị ghi lặp lại (ngữ nghĩa At-least-once).
- **Giao hàng lũy đẳng (Exactly-once & Idempotent):** Từ bản 0.11.0.0, Producer hỗ trợ tùy chọn gửi lũy đẳng (idempotent delivery). Kafka giải quyết sự cố trùng lặp bằng cách cấp cho mỗi Producer một ID duy nhất và mỗi tin nhắn được gán một số thứ tự (sequence number). Broker sẽ dùng các thông tin này để tự động loại bỏ các tin nhắn bị gửi lại trùng lặp.
- **Giao dịch (Transactions):** Cũng từ phiên bản 0.11.0.0, Producer hỗ trợ ghi dữ liệu nguyên tử (atomic) tới nhiều topic/partition cùng lúc. Nhờ đó, nếu một giao dịch gặp lỗi và bị hủy bỏ (aborted), sẽ không có phần tin nhắn nào của giao dịch đó được coi là thành công.

## Consumer

1. Cơ chế Pull (Kéo) thay vì Push (Đẩy)

Khác với nhiều hệ thống nhắn tin truyền thống có xu hướng đẩy (push) dữ liệu từ Broker sang Consumer, Kafka sử dụng mô hình **kéo (pull)**, nơi Consumer chủ động kéo dữ liệu về từ Broker. Mô hình này mang lại nhiều lợi thế:

- **Tránh làm quá tải Consumer:** Trong hệ thống push, Broker kiểm soát tốc độ truyền dữ liệu. Nếu tốc độ sinh dữ liệu cao hơn tốc độ xử lý, Consumer sẽ dễ bị nghẽn và quá tải (tương tự như một dạng tấn công từ chối dịch vụ). Với mô hình pull, Consumer có thể xử lý dữ liệu với tốc độ tối đa của riêng nó, nếu bị tụt lại phía sau, nó chỉ đơn giản là từ từ xử lý để bắt kịp.
- **Tối ưu hóa việc gom nhóm (Batching):** Mô hình pull cho phép gom nhóm dữ liệu truyền đi cực kỳ tích cực. Thay vì Broker phải đắn đo xem nên gửi từng tin nhắn hay chờ gom thêm dữ liệu, Consumer trong Kafka luôn chủ động kéo tất cả các tin nhắn có sẵn sau vị trí hiện tại của nó (hoặc theo một cấu hình kích thước tối đa). Điều này mang lại hiệu quả gom nhóm tối ưu mà không làm tăng độ trễ mạng.
- **Long-poll (Chờ dài):** Một điểm yếu phổ biến của cơ chế kéo là "busy-waiting" – Consumer liên tục hỏi xin dữ liệu tạo thành vòng lặp tốn tài nguyên khi Broker đang trống. Kafka khắc phục bằng các tham số "long poll", cho phép yêu cầu của Consumer chờ chặn (block) cho đến khi có dữ liệu mới tới hoặc đủ một lượng byte tối thiểu.

2. Quản lý Vị trí Tiêu thụ (Consumer Position / Offset)

Việc theo dõi xem tin nhắn nào đã được đọc là một bài toán hiệu suất quan trọng:

- **Cách tiếp cận truyền thống:** Đa số các hệ thống lưu trạng thái ngay trên Broker (đã gửi, đã nhận, đã xác nhận). Việc này khiến Broker phải lưu giữ nhiều trạng thái cho từng tin nhắn riêng lẻ, gây tốn kém tài nguyên và dẫn đến rủi ro gửi đúp nếu Consumer đọc xong nhưng chưa kịp gửi xác nhận (ack) thì bị lỗi.
- **Giải pháp của Kafka:** Một topic trong Kafka được chia thành nhiều partition có thứ tự tĩnh, và mỗi partition ở một thời điểm chỉ được đọc bởi đúng một Consumer trong cùng một nhóm (Consumer Group). Do đó, vị trí của Consumer chỉ đơn giản là một số nguyên duy nhất (gọi là **offset**) biểu thị vị trí tin nhắn tiếp theo cần đọc. Trạng thái này vô cùng nhỏ gọn và rất dễ để lưu (checkpoint) định kỳ.
- **Tính năng "Tua lại" (Rewind):** Nhờ cơ chế offset này, Consumer có thể chủ động lùi offset về một vị trí trong quá khứ để đọc lại dữ liệu. Tính năng này vi phạm nguyên tắc của hàng đợi thông thường nhưng lại vô cùng hữu ích, chẳng hạn khi phát hiện có lỗi (bug) logic trong mã nguồn và bạn cần xử lý lại các dữ liệu vừa đọc.

3. Thành viên Tĩnh (Static Membership)

Thông thường, ID của các Consumer trong một nhóm được bộ điều phối (group coordinator) cấp phát động, các ID này mang tính tạm thời và sẽ thay đổi mỗi khi Consumer khởi động lại. Điều này dẫn đến hiện tượng hệ thống phải liên tục "rebalance" (phân bổ lại partition) mỗi khi cập nhật cấu hình hoặc triển khai mã nguồn, gây gián đoạn ứng dụng. Để tăng tính khả dụng, Kafka giới thiệu tính năng **Thành viên Tĩnh**. Người dùng có thể cấu hình một ID cố định (thông qua `GROUP_INSTANCE_ID_CONFIG`). Nhờ có ID này, danh tính của Consumer được hệ thống ghi nhớ và sẽ không bị thay đổi khi khởi động lại, từ đó ngăn chặn việc kích hoạt rebalance không cần thiết.

4. Chia sẻ Tiêu thụ (The Share Consumer)

Để đáp ứng một số quy trình xử lý tin nhắn truyền thống không hoàn toàn phù hợp với Consumer Group thông thường (nơi 1 partition chỉ dành cho 1 consumer), Kafka cung cấp mô hình **Share Groups**:

- **Nhiều người cùng đọc một partition:** Các thành viên trong Share Group có thể hợp tác tiêu thụ dữ liệu, trong đó một partition có thể được chia cho nhiều consumer cùng xử lý.
- **Vượt giới hạn Partition:** Số lượng Share Consumer có thể lớn hơn số lượng partition của topic.
- **Khóa từng bản ghi (Record Locks):** Khi một Share Consumer kéo dữ liệu, hệ thống sẽ cấp cho nó một khoảng thời gian khóa bản ghi (mặc định 30 giây). Trong lúc bị khóa, các consumer khác trong nhóm sẽ không thể lấy được tin nhắn này.
- **Xác nhận riêng lẻ:** Từng bản ghi sẽ được Share Consumer xử lý riêng lẻ (tương tự như RabbitMQ). Nếu xử lý thành công, nó gửi Acknowledge; nếu không thể xử lý, nó có thể từ chối (Reject), hoặc giải phóng (Release) khóa để các consumer khác lấy lại tin nhắn đó.

5. Tải dữ liệu ngoại tuyến (Offline Data Load)

Khả năng lưu trữ (persistence) mạnh mẽ của Kafka cho phép các Consumer chạy theo cơ chế "batch" – tức là chỉ thỉnh thoảng hoạt động để nạp dữ liệu hàng loạt vào các hệ thống ngoại tuyến như Hadoop hoặc Data Warehouse. Công việc nạp dữ liệu có thể được chạy song song hoàn toàn dựa trên từng cặp node/topic/partition, và nếu có tác vụ nào gặp sự cố, nó chỉ việc khởi động lại ngay từ vị trí offset ban đầu mà không có rủi ro trùng lặp dữ liệu.

## Message Delivery Semantics

Kafka cung cấp đủ 3 cấu hình ngữ nghĩa giao hàng:

- **At-most-once:** Tin nhắn có thể mất nhưng không gửi lặp lại.
- **At-least-once:** Mặc định của Kafka. Tin nhắn có thể được nhận nhiều lần nếu xảy ra lỗi mạng làm Producer gửi lại.

## Using Transactions

- **Exactly-once & Transactions:** Từ phiên bản 0.11.0.0, Producer có tính năng lũy đẳng (idempotent), gán mỗi tin nhắn một mã sequence number để Broker loại trừ tin trùng lặp. Tính năng Transaction cho phép Producer ghi nhiều gói tin nhắn và cập nhật cả vị trí Offset một cách nguyên tử (atomic). Một khi transaction bị hủy (aborted), Consumer với chế độ cách ly `read_committed` sẽ không nhìn thấy những dữ liệu chưa được chốt.

## The Share Consumer

## Replication

- **Mô hình Leader - Follower:** Mỗi partition có một Leader và nhiều Followers. Chỉ Leader nhận dữ liệu ghi, các Follower sẽ tự động "pull" dữ liệu từ Leader để đồng bộ.
- **In-Sync Replicas (ISR):** Thay vì dùng luật đa số (majority vote) như ZooKeeper hay Raft để tiết kiệm tài nguyên và không gian đĩa cứng, Kafka tự động theo dõi danh sách các replica đồng bộ (ISR). Nếu một bản sao dừng hoạt động hoặc quá tụt hậu về offset, nó sẽ bị loại khỏi ISR. Dữ liệu chỉ được đánh dấu "committed" khi đã chép đến toàn bộ các máy trong ISR.
- **Bầu cử Leader chưa đồng bộ (Unclean Leader Election):** Đây là một tình huống cân não (tradeoff) giữa tính khả dụng (Availability) và tính nhất quán (Consistency). Khi tất cả replica chết, Kafka cho phép bạn chọn: hoặc đợi một node thuộc ISR sống lại (ưu tiên Consistency), hoặc chọn node đầu tiên thức dậy làm Leader dù node này bị thiếu dữ liệu (ưu tiên Availability).

## Log Compaction

Với các dòng dữ liệu cập nhật theo khóa, thay vì xóa theo thời gian cũ/mới, tính năng Compaction đảm bảo Kafka giữ lại ít nhất phiên bản (giá trị) mới nhất của mỗi khóa (key).

- Tính năng này đặc biệt mạnh mẽ đối với việc sao lưu trạng thái database, event sourcing hay phục hồi hệ thống khi gặp sự cố, vì Consumer mới tham gia có thể quét lại từ đầu và có ngay phiên bản dữ liệu hoàn chỉnh của mọi record.
- Việc xóa dữ liệu diễn ra ở nền (background threads) mà không cản trở việc đọc ghi, dữ liệu xóa sẽ nhận marker gọi là "tombstone" (một bản ghi có khóa nhưng thân rỗng).

## Quotas

Trong các cụm (cluster) chia sẻ (multi-tenant), Kafka sử dụng các hạn mức để bảo vệ chống lại các client cư xử tồi (tiêu thụ quá đà gây DOS hệ thống).

- **Network Bandwidth Quotas:** Giới hạn tốc độ bytes/giây.
- **Request Rate Quotas:** Giới hạn tỷ lệ sử dụng CPU/luồng mạng (Network và I/O threads). Thay vì trả lỗi, Broker xử lý vi phạm hạn mức bằng cách giữ im lặng kết nối (mute) hoặc phản hồi lại một khoảng trễ (delay) ngay lập tức, báo cho client ngừng gửi thêm yêu cầu, tạo ra một áp lực ngược (back-pressure) hiệu quả.

Nguồn: https://kafka.apache.org/43/design/design