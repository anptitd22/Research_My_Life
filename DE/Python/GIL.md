global interpreter lock[¶](https://docs.python.org/3/glossary.html#term-global-interpreter-lock "Link to this term")

Cơ chế được trình thông dịch [CPython](https://docs.python.org/3/glossary.html#term-CPython) sử dụng để đảm bảo chỉ có một luồng thực thi mã bytecode Python tại một thời điểm. Điều này đơn giản hóa việc triển khai CPython bằng cách làm cho mô hình đối tượng (bao gồm cả các kiểu dữ liệu tích hợp quan trọng như dict) an toàn một cách ngầm định trước sự truy cập đồng thời. Việc khóa toàn bộ trình thông dịch giúp cho trình thông dịch dễ dàng hoạt động đa luồng hơn, nhưng lại làm giảm đi phần lớn khả năng song song hóa mà các máy tính đa bộ xử lý cung cấp.

Tuy nhiên, một số mô-đun mở rộng, dù là tiêu chuẩn hay của bên thứ ba, được thiết kế để giải phóng GIL khi thực hiện các tác vụ tính toán chuyên sâu như nén hoặc băm. Ngoài ra, GIL luôn được giải phóng khi thực hiện thao tác I/O.

Kể từ Python 3.13, GIL có thể bị vô hiệu hóa bằng cách sử dụng cấu hình biên dịch --disable-gil. Sau khi biên dịch Python với tùy chọn này, mã phải được chạy với -X gil=0 hoặc sau khi thiết lập biến môi trường PYTHON_GIL=0. Tính năng này giúp cải thiện hiệu suất cho các ứng dụng đa luồng và giúp sử dụng CPU đa lõi hiệu quả hơn. Để biết thêm chi tiết, hãy xem PEP 703.

Trong các phiên bản trước của API C của Python, một hàm có thể khai báo rằng nó yêu cầu GIL phải được giữ để có thể sử dụng. Điều này đề cập đến việc có một trạng thái luồng được gắn kèm.

global state

Dữ liệu có thể truy cập được trong toàn bộ chương trình, chẳng hạn như các biến cấp mô-đun, biến lớp hoặc các biến tĩnh C trong [extension modules](https://docs.python.org/3/glossary.html#term-extension-module). Trong các chương trình đa luồng, trạng thái toàn cục được chia sẻ giữa các luồng thường yêu cầu đồng bộ hóa để tránh [race conditions](https://docs.python.org/3/glossary.html#term-race-condition) và [data races](https://docs.python.org/3/glossary.html#term-data-race).

hash-based pyc

Tệp bộ nhớ đệm mã bytecode sử dụng mã băm thay vì thời gian sửa đổi cuối cùng của tệp nguồn tương ứng để xác định tính hợp lệ của nó. Xem [Cached bytecode invalidation](https://docs.python.org/3/reference/import.html#pyc-invalidation).

Nguồn: https://docs.python.org/3/glossary.html#term-CPython

------------------------------------------------------------------

Đọc thêm: https://www.geeksforgeeks.org/python/what-is-the-python-global-interpreter-lock-gil/


