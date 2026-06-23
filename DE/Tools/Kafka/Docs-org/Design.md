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

4. Tải dữ liệu ngoại tuyến (Offline Data Load)

Khả năng lưu trữ (persistence) mạnh mẽ của Kafka cho phép các Consumer chạy theo cơ chế "batch" – tức là chỉ thỉnh thoảng hoạt động để nạp dữ liệu hàng loạt vào các hệ thống ngoại tuyến như Hadoop hoặc Data Warehouse. Công việc nạp dữ liệu có thể được chạy song song hoàn toàn dựa trên từng cặp node/topic/partition, và nếu có tác vụ nào gặp sự cố, nó chỉ việc khởi động lại ngay từ vị trí offset ban đầu mà không có rủi ro trùng lặp dữ liệu.

## Message Delivery Semantics

Kafka cung cấp đủ 3 cấu hình ngữ nghĩa giao hàng:

- **At most once (Tối đa một lần):** Message có thể bị mất nhưng tuyệt đối không bao giờ được gửi lại hoặc xử lý lặp lại.
- **At least once (Ít nhất một lần):** Message sẽ không bao giờ bị mất, nhưng trong một số trường hợp có thể bị gửi hoặc xử lý nhiều lần (trùng lặp).
- **Exactly once (Chính xác một lần):** Đây là mức độ lý tưởng nhất, mỗi message được xử lý một lần và chỉ một lần duy nhất.

Bài toán này được chia làm hai phần riêng biệt: sự đảm bảo khi Producer xuất bản tin nhắn và sự đảm bảo khi Consumer đọc tin nhắn.

1. Từ góc nhìn của Producer (Đảm bảo xuất bản tin nhắn)

Trong Kafka, một tin nhắn được coi là "đã cam kết" (committed) chỉ khi toàn bộ các máy chủ bản sao đồng bộ (ISR - In-Sync Replicas) của phân vùng đó đã ghi nó vào log của mình. Một khi đã cam kết, tin nhắn sẽ không bị mất miễn là còn ít nhất một máy chủ sống sót.

Tuy nhiên, rủi ro xảy ra khi mạng bị lỗi:

- **Vấn đề trùng lặp (At-least-once):** Nếu Producer gửi tin đi nhưng gặp lỗi mạng và không nhận được phản hồi, nó sẽ không biết tin nhắn đã được cam kết hay chưa. Trước phiên bản Kafka 0.11.0.0, Producer buộc phải gửi lại tin nhắn, dẫn đến nguy cơ tin nhắn bị ghi đúp vào hệ thống, tạo ra ngữ nghĩa **At-least-once**.
- **Gửi lũy đẳng (Idempotent Delivery):** Từ phiên bản 0.11.0.0, Kafka giải quyết triệt để vấn đề trên bằng tùy chọn gửi lũy đẳng. Hệ thống gán cho mỗi Producer một ID duy nhất và mỗi tin nhắn một số thứ tự (sequence number). Dựa vào thông tin này, Broker có thể tự động nhận diện và loại bỏ các tin nhắn bị Producer gửi lại do lỗi mạng, đảm bảo không có bản ghi trùng lặp nào được ghi vào log.
- **Gửi theo Giao dịch (Transactional Delivery):** Cũng từ bản 0.11.0.0, Producer hỗ trợ ghi tin nhắn trên nhiều topic/partition dưới dạng nguyên tử (atomic) thông qua các giao dịch (transactions), đảm bảo rằng tất cả tin nhắn cùng thành công hoặc cùng thất bại.

Tùy vào nhu cầu thực tế về độ trễ, Producer hoàn toàn có thể tự cấu hình mức độ đảm bảo này (ví dụ: gửi bất đồng bộ hoàn toàn để tối ưu độ trễ, hoặc chờ cho đến khi tin nhắn được cam kết hoàn toàn trên các ISR).

2. Từ góc nhìn của Consumer (Đảm bảo tiêu thụ tin nhắn)

Mọi bản sao trong Kafka đều có chung một bản log và các offset (vị trí tin nhắn) giống hệt nhau, và Consumer tự kiểm soát vị trí đọc của mình trong log đó. Việc đạt được ngữ nghĩa nào phụ thuộc hoàn toàn vào **thứ tự Consumer lưu lại vị trí offset và thời điểm xử lý dữ liệu**:

- **At-most-once:** Consumer đọc tin nhắn -> **Lưu vị trí (offset)** -> Xử lý tin nhắn. Nếu ứng dụng sập ngay sau khi lưu vị trí nhưng chưa kịp xử lý xong, khi khởi động lại, nó sẽ tiếp tục đọc từ vị trí mới và bỏ qua các tin nhắn cũ chưa được xử lý thành công. Kafka cho phép triển khai chế độ này bằng cách vô hiệu hóa tính năng tự động thử lại (retries) và chủ động chốt offset trước khi xử lý.
- **At-least-once (Mặc định):** Consumer đọc tin nhắn -> Xử lý tin nhắn -> **Lưu vị trí (offset)**. Nếu ứng dụng sập sau khi xử lý xong nhưng chưa kịp lưu vị trí, ứng dụng mới khởi động lên sẽ đọc lại từ vị trí cũ và xử lý lại các tin nhắn đó, tạo ra hiện tượng trùng lặp.

3. Đạt được ngữ nghĩa Exactly-once (Chính xác một lần)

Việc đạt được **Exactly-once** khi luân chuyển dữ liệu từ một topic Kafka sang một topic Kafka khác (như trong ứng dụng Kafka Streams) được kết hợp nhờ sức mạnh của **Giao dịch (Transactions)**:

- Vị trí (offset) của Consumer trên thực tế được Kafka lưu trữ như một tin nhắn bên trong một topic nội bộ.
- Quá trình Consumer lưu cập nhật offset và quá trình xuất dữ liệu ra topic mới được thực hiện trong **cùng một giao dịch (transaction)**.
- Nếu giao dịch bị hủy bỏ (aborted), offset sẽ tự động hoàn tác về vị trí cũ. Các Consumer ở đầu ra (output) khi được cấu hình mức độ cách ly `read_committed` sẽ chỉ nhìn thấy các tin nhắn thuộc về giao dịch đã cam kết thành công, do đó không bao giờ đọc phải dữ liệu bị lỗi hoặc trùng lặp.

**Khi ghi dữ liệu ra hệ thống bên ngoài Kafka:** Để đạt Exactly-once khi kết nối hệ thống ngoài, cách tốt nhất là **lưu trữ offset của Consumer vào cùng một nơi với dữ liệu đầu ra** thay vì sử dụng cơ chế commit hai pha (two-phase commit) phức tạp. Ví dụ: hệ thống Kafka Connect có thể lưu dữ liệu và offset vào HDFS cùng nhau, đảm bảo dữ liệu luôn được cập nhật chính xác một lần mà không sợ rủi ro lệch pha.

## Using Transactions

1. Bản chất của transaction trong Kafka

Transaction trong Kafka có một điểm khác biệt lớn so với các hệ thống message khác: Consumer và Producer hoạt động hoàn toàn tách biệt, và **chỉ có Producer mới có tính chất giao dịch (transactional)**.

Bí quyết để đạt được Exactly-once nằm ở chỗ Producer có khả năng **cập nhật vị trí của Consumer** (được gọi là "committed offset") thông qua một bản ghi transaction. Việc ghi dữ liệu mới và cập nhật offset được thực hiện một cách nguyên tử (atomic), đảm bảo rằng cả hai hành động này cùng thành công hoặc cùng thất bại.

2. Ba yếu tố cốt lõi để xử lý Exactly-once

Để triển khai giao dịch chính xác bằng Producer và Consumer, bạn cần tuân thủ 3 nguyên tắc sau:

- **Phân bổ độc quyền:** Consumer phải sử dụng cơ chế gán phân vùng (partition assignment) để đảm bảo nó là consumer duy nhất trong nhóm (consumer group) đang xử lý phân vùng đó tại một thời điểm.
- **Cập nhật nguyên tử:** Producer phải sử dụng transaction để đảm bảo mọi bản ghi nó sinh ra và mọi offset nó cập nhật thay cho Consumer đều được thực hiện trọn vẹn trong một transaction.
- **Mô hình 1-1:** Để xử lý trơn tru các transaction khi hệ thống xảy ra quá trình phân bổ lại (rebalancing), người ta khuyến nghị sử dụng **một phiên bản Producer cho mỗi phiên bản Consumer**. Các sơ đồ phức tạp hơn vẫn có thể thực hiện nhưng sẽ đi kèm với sự phức tạp rất lớn.

3. Cấu hình thiết yếu

Để cơ chế giao dịch hoạt động đúng, cả Producer và Consumer cần được cấu hình chặt chẽ:

- **Với Consumer:** Bắt buộc cấu hình `isolation.level=read_committed` (để hệ thống không đọc phải các bản ghi thuộc về transaction bị hủy hoặc chưa hoàn tất) và `enable.auto.commit=false` (để ứng dụng tự kiểm soát việc lưu offset thông qua Producer).
- **Với Producer:** Bắt buộc cấu hình `transactional.id` bằng một chuỗi định danh. Việc này không chỉ biến Producer thành dạng transaction mà còn giúp Kafka nhận diện nếu ứng dụng bị khởi động lại, từ đó tự động hủy (abort) các transaction đang treo từ phiên làm việc trước đó.

4. Quản lý lỗi và Hủy giao dịch (Aborted Transactions)

Khi một transaction hoàn tất (commit) hoặc bị hủy (abort), Kafka sẽ ghi các "bản ghi đánh dấu" (transaction marker records) vào hệ thống để biểu thị kết quả của giao dịch đó. Nhờ các marker này, một Consumer được cấu hình `read_committed` có thể nhận biết và tự động bỏ qua các bản ghi của giao dịch bị hủy.

Việc xử lý lỗi đối với Producer giao dịch đã được chuẩn hóa với các nhóm ngoại lệ (exceptions) rõ ràng:

- **Lỗi tự phục hồi:** `RetriableException` và `RefreshRetriableException` là các lỗi tạm thời, client sẽ tự động thử lại ngầm mà không làm phiền đến ứng dụng.
- **Lỗi cần hủy giao dịch (AbortableException):** Lỗi này sẽ được đẩy lên ứng dụng. Ứng dụng bắt buộc phải chủ động hủy transaction hiện tại, đồng thời phải khởi tạo (reset) lại vị trí của Consumer để xử lý lại các bản ghi của transaction vừa hỏng.
- **Lỗi nghiêm trọng cấp ứng dụng:** Bao gồm `ApplicationRecoverableException`, `InvalidConfigurationException`, và `KafkaException`. Với các lỗi này, ứng dụng phải tự có chiến lược phục hồi riêng, thường bao gồm cả việc khởi động lại Producer.

Ví dụ mã mẫu để xử lý các ngoại lệ transaction: [Transaction Client Demo](https://github.com/apache/kafka/blob/trunk/examples/src/main/java/kafka/examples/TransactionalClientDemo.java)

**Chiến lược phục hồi đơn giản nhất:** Khi gặp ngoại lệ không thể tự phục hồi hoặc transaction bị hủy, một chính sách đơn giản và hiệu quả là **hủy bỏ và tạo lại hoàn toàn** các đối tượng Kafka Producer và Kafka Consumer từ đầu. Khi Consumer mới được tạo lại, nó sẽ kích hoạt quá trình rebalance của nhóm, tự động kéo về offset đã commit cuối cùng, qua đó "tua lại" chính xác trạng thái trước khi transaction bị lỗi. Đối với các ứng dụng phức tạp hơn, có thể giữ nguyên đối tượng Consumer và sử dụng hàm `KafkaConsumer.seek` để chủ động tua lại vị trí mong muốn.

## The Share Consumer

Để đáp ứng một số quy trình xử lý tin nhắn truyền thống không hoàn toàn phù hợp với Consumer Group thông thường (nơi 1 partition chỉ dành cho 1 consumer), Kafka cung cấp mô hình **Share Groups**:

- **Nhiều người cùng đọc một partition:** Các thành viên trong Share Group có thể hợp tác tiêu thụ dữ liệu, trong đó một partition có thể được chia cho nhiều consumer cùng xử lý.
- **Vượt giới hạn Partition:** Số lượng Share Consumer có thể lớn hơn số lượng partition của topic.
- **Khóa từng bản ghi (Record Locks):** Khi một Share Consumer kéo dữ liệu, hệ thống sẽ cấp cho nó một khoảng thời gian khóa bản ghi (mặc định 30 giây). Trong lúc bị khóa, các consumer khác trong nhóm sẽ không thể lấy được tin nhắn này.
- **Xác nhận riêng lẻ:** Từng bản ghi sẽ được Share Consumer xử lý riêng lẻ (tương tự như RabbitMQ). Nếu xử lý thành công, nó gửi Acknowledge; nếu không thể xử lý, nó có thể từ chối (Reject), hoặc giải phóng (Release) khóa để các consumer khác lấy lại tin nhắn đó.

## Replication

1. Đơn vị Sao chép và Mô hình Leader - Follower

Trong Kafka, đơn vị sao chép cơ bản là **phân vùng (topic partition)**. Dưới điều kiện hoạt động bình thường, mỗi phân vùng có duy nhất một **Leader** và một số lượng các **Followers**.

- Toàn bộ các thao tác ghi (writes) đều được chuyển thẳng đến Leader. Thao tác đọc có thể được phục vụ bởi Leader hoặc các Follower.
- Các Follower hoạt động tương tự như một Consumer thông thường: chúng chủ động kéo (pull) các tin nhắn từ Leader để cập nhật vào log của chính mình. Nhờ cơ chế kéo này, các Follower có thể tự động gom nhóm (batch) các mục log lại với nhau để tối ưu hóa hiệu suất áp dụng dữ liệu.

2. Danh sách Bản sao Đồng bộ (In-Sync Replicas - ISR)

Để tự động xử lý lỗi, Kafka cần định nghĩa thế nào là một node "còn sống" (alive). Một node được coi là "đồng bộ" (in-sync) nếu nó thỏa mãn 2 điều kiện:

1. Duy trì một phiên hoạt động tích cực (active session) với máy chủ điều phối (Controller), ví dụ như gửi nhịp tim (heartbeat) định kỳ.
2. Nếu là Follower, nó phải sao chép dữ liệu từ Leader mà không được phép tụt lại "quá xa".

Tất cả các node thỏa mãn điều kiện này được tập hợp thành một danh sách gọi là **In-Sync Replicas (ISR)**. Nếu một Follower bị chết (mất session) hoặc bị tụt hậu thời gian quá giới hạn quy định, nó sẽ lập tức bị Controller hoặc Leader gỡ bỏ khỏi danh sách ISR.

3. Thiết kế Quorum độc đáo: ISR thay vì Luật Đa số (Majority Vote)

Nhiều hệ thống phân tán (như ZooKeeper hay HDFS Namenode) sử dụng **Luật đa số (Majority Vote)** để bầu Leader và xác nhận dữ liệu. Điểm yếu của luật đa số là nó đòi hỏi rất nhiều bản sao: để chịu đựng được f máy bị lỗi, hệ thống cần duy trì 2f+1 bản sao (ví dụ: cần tới 5 bản sao để chịu được 2 lỗi). Điều này làm tốn gấp 5 lần không gian đĩa cứng và giảm 1/5 thông lượng, không phù hợp cho hệ thống lưu trữ khối lượng dữ liệu khổng lồ như Kafka.

Để giải quyết bài toán này, **Kafka chọn một hướng đi riêng:**

- Hệ thống duy trì động danh sách ISR. Bất kỳ node nào nằm trong ISR đều đủ điều kiện để được bầu làm Leader.
- Một tin nhắn ghi vào Kafka chỉ được coi là "đã chốt" (committed) khi **tất cả** các bản sao đang nằm trong ISR đều đã nhận được.
- Bầu Leader mới: Khi Leader sập, chỉ những thành viên nằm trong ISR mới có quyền lên làm Leader mới.
- Nhờ cơ chế này, Kafka chỉ cần cấu hình f+1 bản sao để chịu được f lỗi mà không làm mất dữ liệu đã chốt. Việc này giúp tiết kiệm đáng kể không gian lưu trữ và cải thiện thông lượng so với hệ thống dựa trên luật đa số.

Ngoài ra, giao thức phục hồi của Kafka không bắt buộc một node bị sập phải giữ nguyên vẹn dữ liệu khi khởi động lại. Trước khi được gia nhập lại ISR, node đó buộc phải đồng bộ hóa lại toàn bộ để đề phòng trường hợp mất dữ liệu khi bị sập nguồn. Thiết kế này giúp Kafka tránh việc phải gọi lệnh `fsync` xuống ổ đĩa cho mỗi lần ghi, giúp giữ được hiệu năng cực kỳ cao.

4. Bầu cử Leader khi tất cả đều chết (Unclean Leader Election)

Điều gì xảy ra nếu toàn bộ các node chứa bản sao của một phân vùng đều sập? Lúc này, tính cam kết dữ liệu không còn được đảm bảo, và hệ thống phải đánh đổi giữa **Tính khả dụng (Availability)** và **Tính nhất quán (Consistency)**:

- **Ưu tiên nhất quán (Mặc định từ bản 0.11.0.0):** Hệ thống sẽ chờ cho đến khi có một node từng thuộc danh sách ISR sống lại và chọn nó làm Leader. Phương pháp này đảm bảo không mất dữ liệu, nhưng hệ thống sẽ bị ngừng hoạt động (unavailable) chừng nào node đó chưa phục hồi.
- **Ưu tiên khả dụng:** Hệ thống chọn ngay node đầu tiên thức dậy làm Leader, bất kể nó có nằm trong ISR hay không (Unclean Leader Election). Điều này giúp hệ thống hoạt động lại lập tức, nhưng log của node thiếu sót này sẽ trở thành "nguồn chân lý", dẫn đến việc một số dữ liệu đã chốt trước đó có thể bị mất.

5. Đảm bảo Tính khả dụng và Độ bền vững (Guarantees)

Producer có thể điều chỉnh mức độ an toàn mong muốn bằng cấu hình `acks` (đợi 0, 1, hoặc tất cả replicas). Nếu cấu hình `acks=all`, Kafka đảm bảo tin nhắn không bị mất chừng nào còn ít nhất 1 node trong ISR sống sót.

Để ưu tiên tối đa độ bền vững thay vì tính khả dụng, người dùng có thể cấu hình thông số **min.insync.replicas**. Cấu hình này chỉ định kích thước tối thiểu mà danh sách ISR phải có. Nếu số lượng node trong ISR giảm xuống dưới mức này, phân vùng sẽ từ chối các yêu cầu ghi mới (bị mất tính khả dụng) nhằm đảm bảo dữ liệu luôn được ghi đủ số lượng bản sao an toàn.

6. Quản lý hệ thống Bản sao (Replica Management)

Kafka phân bổ đều các phân vùng (round-robin) cũng như vai trò Leadership trên khắp các máy chủ trong cụm để cân bằng tải. Một máy chủ sẽ đóng vai trò đặc biệt gọi là **"Controller"**, có nhiệm vụ quản lý vòng đời của các broker. Khi một broker bị lỗi, thay vì mỗi phân vùng trên broker đó phải tự động mở một cuộc bầu cử riêng lẻ tốn thời gian, Controller sẽ phát hiện và gom nhóm (batch) việc thông báo thay đổi Leadership cho nhiều phân vùng cùng lúc. Cơ chế tập trung này làm cho quá trình khôi phục sau sự cố trở nên nhanh chóng và ít tốn kém tài nguyên hơn rất nhiều.

## Log Compaction

Thông thường, Kafka xóa dữ liệu cũ dựa trên hai yếu tố thô sơ: **Thời gian** (hết 7 ngày thì xóa) hoặc **Kích thước** (quá 1GB thì xóa). Cách này rất tốt cho dữ liệu dạng sự kiện (Log, Clickstream) vì mỗi tin nhắn độc lập với nhau.

Tuy nhiên, với dữ liệu dạng thay đổi trạng thái (ví dụ: cập nhật Email của User), chúng ta chỉ quan tâm đến **trạng thái cuối cùng (mới nhất)** của User đó.

- Nếu xóa theo thời gian, ta có thể vô tình xóa mất Email của một User đã lâu không cập nhật.    
- Nếu lưu lại tất cả lịch sử thay đổi từ xưa đến nay, ổ đĩa sẽ phình to vô hạn.
    
> **Giải pháp là Log Compaction:** Thay vì xóa theo thời gian, Kafka sẽ quét và xóa các bản ghi cũ, **chỉ giữ lại duy nhất bản ghi có giá trị mới nhất cho mỗi Key.**

1. Ứng dụng thực tiễn (Use Cases)

Log Compaction đặc biệt hữu ích cho các luồng dữ liệu theo dõi sự thay đổi của dữ liệu có thể cập nhật (mutable data), điển hình như:

- **Đồng bộ thay đổi cơ sở dữ liệu (Database change subscription):** Khi luồng Kafka chứa các cập nhật từ Database, hệ thống cần duy trì trạng thái dữ liệu hoàn chỉnh để nạp lại vào bộ đệm (cache), máy chủ tìm kiếm hoặc Hadoop khi có sự cố, thay vì chỉ lấy các bản ghi thay đổi theo thời gian thực.
- **Event Sourcing:** Một mẫu thiết kế ứng dụng dùng chính luồng thay đổi (log) làm kho lưu trữ chính (primary store) cho ứng dụng.
- **Phục hồi tính sẵn sàng cao (High-Availability Journaling):** Các hệ thống xử lý luồng (như Apache Samza) có thể ghi lại các thay đổi về trạng thái cục bộ của chúng (ví dụ: các phép đếm, gom nhóm) vào Kafka. Nếu máy tính toán bị sập, một quy trình mới có thể đọc lại dữ liệu này để khôi phục toàn bộ trạng thái và tiếp tục chạy.

2. Cơ chế hoạt động (Log Compaction Basics)

![[kafka_introduction2.png]]

Về mặt logic, một log được cấu hình compaction sẽ chia làm 2 phần:

- **Head (Đầu Nhật Ký - Vùng "Bẩn" / Dirty):** Nằm ở phía cuối log, nơi các tin nhắn mới đang liên tục được ghi vào. Vùng này **chưa được nén**, các offset (chỉ số) liên tiếp nhau (36, 37, 38...) và có thể chứa nhiều tin nhắn trùng Key.
- **Tail (Đuôi Nhật Ký - Vùng "Sạch" / Clean):** Đã trải qua quá trình nén. Tại đây, mỗi Key chỉ có duy nhất một giá trị mới nhất.

### Điểm đặc biệt về Offset:

Khi nén, các bản ghi cũ bị xóa đi khiến các số Offset không còn liên tục nữa (ví dụ từ offset 35 nhảy thẳng lên 38). Tuy nhiên:

- **Offset của tin nhắn không bao giờ thay đổi.** Nó là định danh vĩnh viễn.
    
- Nếu Client yêu cầu đọc từ một offset đã bị xóa (ví dụ offset 36), Kafka sẽ tự động trả về tin nhắn ở offset lớn hơn gần nhất (offset 38).

**Đặc điểm của quá trình tinh gọn (nén):**

- **Bảo toàn Offset (Offset Immutability):** Dù tin nhắn bị xóa đi, các tin nhắn còn lại trong phần Tail vẫn giữ nguyên offset gốc được cấp phát lúc mới ghi. Offset không bao giờ thay đổi. Nếu một consumer gọi đọc từ một offset đã bị xóa, hệ thống chỉ đơn giản trả về tin nhắn có offset hợp lệ cao hơn tiếp theo.
- **Đánh dấu xóa (Tombstones):** Để xóa hoàn toàn một key, Producer gửi một tin nhắn có key đó nhưng phần thân (payload) mang giá trị `null`. Bản ghi này được gọi là "tombstone" (bia mộ). Marker này sẽ dọn sạch mọi tin nhắn cũ của key đó, nhưng chính nó cũng sẽ bị xóa khỏi log sau một khoảng thời gian (cấu hình qua `delete.retention.ms`, mặc định là 24 giờ) để giải phóng không gian,.

![[kafka_introduction3.png]]

3. Tiến trình dọn dẹp ở nền (The Log Cleaner)

Việc tinh gọn được thực hiện bởi "log cleaner" – một nhóm các luồng chạy ngầm (background threads). Quá trình dọn dẹp không hề gây gián đoạn hay chặn (block) các thao tác đọc ghi của client và có thể được giới hạn băng thông I/O để không ảnh hưởng đến hiệu suất hệ thống.

Cụ thể, một luồng compactor hoạt động theo các bước:

1. Chọn tệp log đang có tỷ lệ giữa phần Head và phần Tail cao nhất để tối ưu dung lượng dọn dẹp.
2. Quét phần Head và tạo một bảng băm (hash table) rất nhỏ gọn trong bộ nhớ để lưu lại offset mới nhất của từng key. Bảng này chỉ tốn chính xác 24 bytes cho mỗi mục.
3. Quét lại log từ đầu đến cuối, sao chép dữ liệu sang các phân đoạn (segments) mới, đồng thời loại bỏ những tin nhắn có key đã xuất hiện với offset lớn hơn trong bảng băm.
4. Hoán đổi trực tiếp các tệp log cũ bằng tệp log mới và sạch.

5. Những đảm bảo từ Kafka (Guarantees)

Khi sử dụng Log Compaction, Kafka cam kết các điều sau:

- **Giữ nguyên thứ tự:** Compaction chỉ gỡ bỏ tin nhắn chứ không bao giờ đảo lộn thứ tự của chúng.
- **Đọc không bỏ sót đối với người dùng theo kịp:** Bất kỳ Consumer nào theo kịp phần Head của log đều sẽ đọc được toàn bộ các tin nhắn gốc tuần tự.
- **Trạng thái cuối hoàn hảo cho người dùng đọc từ đầu:** Một Consumer mới bắt đầu đọc từ đầu log (offset 0) sẽ luôn thấy được trạng thái cuối cùng của mọi bản ghi theo đúng thứ tự chúng được ghi vào. Nó cũng sẽ nhận được các thông báo xóa (tombstones) miễn là nó bắt kịp đến phần Head của log trong khoảng thời gian trước khi tombstone bị hết hạn.

5. Cấu hình kiểm soát độ trễ Compaction (Configuration)

Người dùng có thể tinh chỉnh hành vi của compactor thông qua các thông số:

- **min.compaction.lag.ms**: Cấu hình thời gian tối thiểu một tin nhắn phải nằm trong phần Head (không bị tinh gọn). Điều này đảm bảo ứng dụng có đủ độ trễ để kịp đọc mọi sự kiện cập nhật trước khi chúng bị nén lại.
- **max.compaction.lag.ms**: Cấu hình khoảng thời gian trễ tối đa để đảm bảo một tin nhắn (ngay cả trên log có tốc độ ghi thấp) không bị tồn đọng vô thời hạn mà chắc chắn sẽ đủ điều kiện để được đưa vào diện xem xét tinh gọn,.

## Quotas

Trong các cụm (cluster) chia sẻ (multi-tenant), Kafka sử dụng các hạn mức để bảo vệ chống lại các client cư xử tồi (tiêu thụ quá đà gây DOS hệ thống).

- **Network Bandwidth Quotas:** Giới hạn tốc độ bytes/giây.
- **Request Rate Quotas:** Giới hạn tỷ lệ sử dụng CPU/luồng mạng (Network và I/O threads). Thay vì trả lỗi, Broker xử lý vi phạm hạn mức bằng cách giữ im lặng kết nối (mute) hoặc phản hồi lại một khoảng trễ (delay) ngay lập tức, báo cho client ngừng gửi thêm yêu cầu, tạo ra một áp lực ngược (back-pressure) hiệu quả.

Nguồn: https://kafka.apache.org/43/design/design