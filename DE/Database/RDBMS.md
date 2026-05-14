![alt][images/Database/rdbms_1.png]

## 1. Client Communications Manager

Đây là "cổng chào" của hệ thống, chịu trách nhiệm thiết lập và quản lý các kết nối.

- **Local Client Protocols:** Xử lý các kết nối từ các ứng dụng chạy ngay trên cùng một server với DB (thông qua Shared Memory hoặc Unix Domain Sockets). Tốc độ này thường cực nhanh vì không qua lớp mạng.
    
- **Remote Client Protocols:** Quản lý các kết nối từ xa qua TCP/IP. Nó chịu trách nhiệm bắt tay (handshake), xác thực bảo mật (SSL/TLS) và duy trì phiên làm việc (session).
    

## 2. Process Manager

Nhiệm vụ của khối này là điều phối tài nguyên hệ thống để xử lý các yêu cầu.

- **Admission Control:** Bộ phận "bảo vệ". Nó quyết định xem hệ thống có đang quá tải hay không để cho phép một truy vấn mới được chạy hay phải xếp hàng chờ, nhằm tránh tình trạng sập hệ thống (OOM hoặc CPU 100%).
    
- **Dispatch and Scheduling:** Sau khi được nhận vào, thành phần này sẽ phân bổ công việc cho các luồng xử lý (Threads hoặc Processes) và ưu tiên thứ tự thực hiện.
    

## 3. Relational Query Processor

Đây là "bộ não" thực hiện các phép tính toán logic SQL.

- **Query Parsing and Authorization:** Kiểm tra xem câu lệnh SQL của bạn viết đúng ngữ pháp chưa và bạn có quyền truy cập vào bảng đó hay không.
    
- **Query Rewrite:** Tối ưu hóa câu lệnh ở mức logic (ví dụ: chuyển đổi các subquery thành join nếu hiệu quả hơn).
    
- **Query Optimizer:** **Thành phần quan trọng nhất.** Nó sẽ tính toán hàng ngàn phương án (Execution Plans) để tìm ra cách lấy dữ liệu ít tốn kém nhất (dựa trên thống kê index, CPU, I/O).
	- Sử dụng EXPLAIN statement 
    
- **Plan Executor:** Nhận bản kế hoạch từ Optimizer và bắt đầu ra lệnh cho lớp lưu trữ bên dưới để lấy dữ liệu.
    
- **DDL and Utility:** Xử lý các lệnh thay đổi cấu trúc bảng như `CREATE`, `ALTER`, `DROP`.
    

## 4. Transactional Storage Manager

Lớp này trực tiếp làm việc với dữ liệu trên đĩa cứng và đảm bảo tính an toàn dữ liệu.

- **Access Methods:** Các thuật toán để đọc dữ liệu (quét toàn bộ bảng hay đi qua B-Tree Index, Hash Index).
    
- **Lock Manager:** Quản lý việc khóa dữ liệu. Đảm bảo khi bạn đang sửa một dòng thì người khác không thể vào ghi đè lên, giúp duy trì tính nhất quán.
    
- **Buffer Manager:** Quản lý vùng đệm trên RAM. Dữ liệu từ đĩa được đưa lên RAM để xử lý nhanh hơn; nó quyết định dữ liệu nào được giữ lại và dữ liệu nào bị đẩy ra khỏi RAM.
	- Thuật toán thay thế (LRU)
    
- **Log Manager:** Ghi lại mọi thay đổi vào file Log (như WAL - Write Ahead Logging). Nếu hệ thống mất điện, Log Manager sẽ dùng các file này để khôi phục lại dữ liệu về trạng thái an toàn.
    

---

## 5. Shared Components and Utilities

Các dịch vụ hỗ trợ chạy xuyên suốt toàn bộ hệ thống.

- **Catalog Manager:** Lưu trữ metadata (dữ liệu về dữ liệu) như cấu trúc bảng, tên cột, kiểu dữ liệu, các ràng buộc...
    
- **Memory Manager:** Cấp phát và quản lý vùng nhớ cho toàn bộ các thành phần của DBMS.
    
- **Administration Monitoring & Utilities:** Cung cấp các công cụ để Admin theo dõi sức khỏe hệ thống (CPU, RAM sử dụng) và cấu hình thông số.
    
- **Replication and Loading Services:** Quản lý việc đồng bộ dữ liệu sang các server khác (như Master-Slave) và hỗ trợ nạp dữ liệu lớn (bulk load).
    
- **Batch Utilities:** Chạy các tiến trình ngầm định kỳ hoặc các tác vụ xử lý hàng loạt nặng nề.
    

---

### **Mối liên hệ giữa các khối:**

Khi bạn gửi một câu lệnh `SELECT`:

1. **Communications Manager** nhận kết nối.
    
2. **Process Manager** cấp luồng xử lý.
    
3. **Query Processor** phân tích và chọn đường đi tốt nhất.
    
4. **Storage Manager** kiểm tra RAM (Buffer), nếu không có thì đọc từ Đĩa, sau đó trả dữ liệu ngược lên trên.