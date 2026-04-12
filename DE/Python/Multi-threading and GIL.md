# Thread states and the global interpreter lock

Trừ khi sử dụng bản [free-threaded build](https://docs.python.org/3/glossary.html#term-free-threaded-build) của [CPython](https://docs.python.org/3/glossary.html#term-CPython), trình thông dịch Python nói chung không an toàn cho đa luồng. Để hỗ trợ các chương trình Python đa luồng, cần có một khóa toàn cục, được gọi là khóa trình thông dịch toàn cục (GIL), mà một luồng phải giữ trước khi truy cập các đối tượng Python. Nếu không có khóa này, ngay cả những thao tác đơn giản nhất cũng có thể gây ra sự cố trong một chương trình đa luồng: ví dụ, khi hai luồng đồng thời tăng số lượng tham chiếu của cùng một đối tượng, số lượng tham chiếu có thể chỉ được tăng một lần thay vì hai lần.

Do đó, chỉ luồng nào nắm giữ GIL mới có thể thao tác trên các đối tượng Python hoặc gọi API C của Python.

Để mô phỏng tính concurrency, trình thông dịch thường xuyên cố gắng chuyển đổi luồng giữa các lệnh bytecode (xem [`sys.setswitchinterval()`](https://docs.python.org/3/library/sys.html#sys.setswitchinterval "sys.setswitchinterval")). Đây là lý do tại sao khóa cũng cần thiết cho tính an toàn luồng trong mã Python thuần túy.

Ngoài ra, global interpreter lock được giải phóng xung quanh các thao tác I/O, chẳng hạn như đọc hoặc ghi vào tệp. Theo API C, điều này được thực hiện bằng cách tách trạng thái luồng ([detaching the thread state](https://docs.python.org/3/c-api/threads.html#detaching-thread-state)).

Trình thông dịch Python lưu giữ một số thông tin cục bộ của luồng bên trong một cấu trúc dữ liệu gọi là [`PyThreadState`](https://docs.python.org/3/c-api/threads.html#c.PyThreadState "PyThreadState"), được biết đến như là trạng thái luồng ([thread state](https://docs.python.org/3/glossary.html#term-thread-state)). Mỗi luồng có một con trỏ cục bộ trỏ đến PyThreadState; trạng thái luồng được tham chiếu bởi con trỏ này được coi là trạng thái đã được gắn kết ([attached](https://docs.python.org/3/glossary.html#term-attached-thread-state)).

Một luồng chỉ có thể có một trạng thái luồng được gắn kết tại một thời điểm ([attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state)). Trạng thái luồng được gắn kết thường tương tự như việc giữ GIL (Global Interpreter Lock), ngoại trừ trên các bản dựng đa luồng tự do. Trên các bản dựng có bật GIL, việc gắn kết một trạng thái luồng sẽ bị chặn cho đến khi có thể giành được GIL. Tuy nhiên, ngay cả trên các bản dựng có GIL bị tắt, vẫn cần phải có một trạng thái luồng được gắn kết, vì trình thông dịch cần theo dõi những luồng nào có thể truy cập các đối tượng Python.

Nguồn: https://docs.python.org/3/c-api/threads.html