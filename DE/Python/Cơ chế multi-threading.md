Mô-đun này xây dựng các giao diện luồng cấp cao hơn dựa trên mô-đun [`_thread`](https://docs.python.org/3/library/_thread.html#module-_thread "_thread: Low-level threading API.") cấp thấp hơn.
Tình trạng: not WASI.
Mô-đun này không hoạt động hoặc không khả dụng trên WebAssembly. Xem [WebAssembly platforms](https://docs.python.org/3/library/intro.html#wasm-availability) để có thêm nhiều thông tin

## Introduction

Mô-đun đa luồng cung cấp một cách để chạy nhiều luồng (các đơn vị nhỏ hơn của một process) concurrently trong single process. Nó cho phép tạo và quản lý các luồng, giúp thực hiện các tác vụ song song, chia sẻ không gian bộ nhớ. Threads đặc biệt hữu ích khi các tác vụ bị giới hạn bởi I/O, chẳng hạn như các thao tác tệp hoặc thực hiện các yêu cầu mạng, nơi phần lớn thời gian được dành để chờ đợi các tài nguyên bên ngoài.

Một trường hợp sử dụng điển hình của đa luồng là quản lý một nhóm các thread xử lý có thể thực hiện nhiều tác vụ đồng thời. Dưới đây là một ví dụ cơ bản về việc tạo và khởi tạo các luồng bằng cách sử dụng `Thread`:

```python
import threading
import time

def crawl(link, delay=3):
    print(f"crawl started for {link}")
    time.sleep(delay)  # Blocking I/O (simulating a network request)
    print(f"crawl ended for {link}")

links = [
    "https://python.org",
    "https://docs.python.org",
    "https://peps.python.org",
]

# Start threads for each link
threads = []
for link in links:
    # Using `args` to pass positional arguments and `kwargs` for keyword arguments
    t = threading.Thread(target=crawl, args=(link,), kwargs={"delay": 2})
    threads.append(t)

# Start each thread
for t in threads:
    t.start()

# Wait for all threads to finish
for t in threads:
    t.join()
```

>[!note]
>Xem thêm: [`concurrent.futures.ThreadPoolExecutor`](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.ThreadPoolExecutor "concurrent.futures.ThreadPoolExecutor") cung cấp giao diện cấp cao hơn để đẩy các tác vụ sang thread nền mà không làm chặn quá trình thực thi của calling thread, đồng thời vẫn có thể truy xuất kết quả khi cần.
>[`queue`](https://docs.python.org/3/library/queue.html#module-queue "queue: A synchronized queue class.") cung cấp giao diện an toàn cho đa luồng để trao đổi dữ liệu giữa các luồng đang chạy.
>[`asyncio`](https://docs.python.org/3/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O.") cung cấp một cách tiếp cận khác để đạt được khả năng xử lý đồng thời các tác vụ mà không cần sử dụng nhiều luồng hệ điều hành.

**CPython implementation detail**: Trong CPython, do [Global Interpreter Lock](https://docs.python.org/3/glossary.html#term-global-interpreter-lock), chỉ có một thread có thể thực thi mã Python tại một thời điểm (mặc dù một số thư viện hướng đến hiệu năng có thể khắc phục hạn chế này). Nếu bạn muốn ứng dụng của mình tận dụng tốt hơn tài nguyên tính toán của các máy đa lõi, bạn nên sử dụng đa xử lý (multiprocessing) hoặc concurrent.futures.ProcessPoolExecutor. Tuy nhiên, đa luồng vẫn là một mô hình phù hợp nếu bạn muốn chạy nhiều tác vụ phụ thuộc vào I/O đồng thời.

## GIL and performance considerations

Không giống như mô-đun [`multiprocessing`](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing "multiprocessing: Process-based parallelism.") , sử dụng các processes (tiến trình) riêng biệt để bỏ qua khóa trình thông dịch toàn cục (GIL), mô-đun đa luồng hoạt động trong một tiến trình duy nhất, có nghĩa là tất cả các luồng chia sẻ cùng một không gian bộ nhớ. Tuy nhiên, GIL hạn chế hiệu suất của đa luồng đối với các tác vụ phụ thuộc vào CPU, vì chỉ có một luồng có thể thực thi mã bytecode Python tại một thời điểm. Mặc dù vậy, đa luồng vẫn là một công cụ hữu ích để đạt được tính concurrency trong nhiều trường hợp.

Kể từ Python 3.13, các bản dựng [free-threaded](https://docs.python.org/3/glossary.html#term-free-threading) có thể vô hiệu hóa GIL, cho phép thực thi song song thực sự các luồng, nhưng tính năng này không khả dụng theo mặc định (xem PEP 703).

## Reference

functions:

- threading.active_count()
- threading.current_thread()
- threading.excepthook(_args_, _/_)
- threading.__excepthook__
- threading.get_ident()
- threading.get_native_id()
- threading.enumerate()
- threading.settrace(_func_)
- threading.settrace_all_threads(_func_)
- threading.gettrace()
- threading.setprofile(_func_)
- threading.setprofile_all_threads(_func_)
- threading.getprofile()
- threading.stack_size([_size_])
- threading.TIMEOUT_MAX

### Thread-local data

Dữ liệu cục bộ theo luồng là dữ liệu có giá trị cụ thể cho từng luồng. Nếu bạn có dữ liệu muốn chỉ áp dụng cho một luồng, hãy tạo một đối tượng cục bộ và sử dụng các thuộc tính của nó

### Thread objects

Lớp Thread đại diện cho một hoạt động được chạy trong một luồng điều khiển riêng biệt. Có hai cách để chỉ định hoạt động này: bằng cách truyền một đối tượng có thể gọi được vào hàm tạo, hoặc bằng cách ghi đè phương thức `run()` trong lớp con. Không nên ghi đè bất kỳ phương thức nào khác (ngoại trừ hàm tạo) trong lớp con. Nói cách khác, chỉ nên ghi đè các phương thức __init__() và `run()` của lớp này.

Sau khi một đối tượng luồng được tạo, hoạt động của nó phải được bắt đầu bằng cách gọi phương thức [`start()`](https://docs.python.org/3/library/threading.html#threading.Thread.start "threading.Thread.start") của luồng. Thao tác này sẽ gọi phương thức [`run()`](https://docs.python.org/3/library/threading.html#threading.Thread.run "threading.Thread.run") trong một luồng điều khiển riêng biệt.

Khi hoạt động của luồng bắt đầu, luồng được coi là 'đang hoạt động'. Nó ngừng hoạt động khi phương thức run() của nó kết thúc – hoặc bình thường, hoặc do ném ra một ngoại lệ không được xử lý. Phương thức is_alive() kiểm tra xem luồng có đang hoạt động hay không.

Các luồng khác có thể gọi phương thức join() của một luồng. Điều này sẽ chặn luồng gọi cho đến khi luồng có phương thức join() được gọi kết thúc.

Mỗi luồng có một tên. Tên này có thể được truyền vào hàm tạo, và có thể được đọc hoặc thay đổi thông qua thuộc tính `name`.

Nếu phương thức `run()` gây ra ngoại lệ, hàm `threading.excepthook()` sẽ được gọi để xử lý ngoại lệ đó. Theo mặc định, `threading.excepthook()` sẽ âm thầm bỏ qua SystemExit.

Một luồng có thể được gắn cờ là "luồng nền" (daemon thread). Ý nghĩa của cờ này là toàn bộ chương trình Python sẽ thoát khi chỉ còn lại các luồng nền. Giá trị ban đầu được kế thừa từ luồng tạo ra nó. Cờ này có thể được thiết lập thông qua thuộc tính `daemon` hoặc đối số của hàm tạo `daemon`.

>[!note]
>Các luồng nền (daemon threads) bị dừng đột ngột khi tắt máy. Tài nguyên của chúng (như các tệp đang mở, giao dịch cơ sở dữ liệu, v.v.) có thể không được giải phóng đúng cách. Nếu bạn muốn các luồng của mình dừng một cách nhẹ nhàng, hãy biến chúng thành luồng không phải nền (non-daemon) và sử dụng cơ chế báo hiệu phù hợp, chẳng hạn như Sự kiện (Event).


Có một đối tượng "luồng chính"; đối tượng này tương ứng với luồng điều khiển ban đầu trong chương trình Python. Nó không phải là một luồng nền (daemon thread).

Có một đối tượng "luồng chính"; đối tượng này tương ứng với luồng điều khiển ban đầu trong chương trình Python. Nó không phải là một luồng nền (daemon thread).

Có khả năng các "đối tượng luồng giả" được tạo ra. Đây là các đối tượng luồng tương ứng với "các luồng ngoại lai", là các luồng điều khiển được khởi tạo bên ngoài mô-đun luồng, chẳng hạn như trực tiếp từ mã C. Các đối tượng luồng giả có chức năng hạn chế; chúng luôn được coi là đang hoạt động và ở chế độ nền, và không thể được kết nối. Chúng không bao giờ bị xóa, vì không thể phát hiện sự kết thúc của các luồng ngoại lai.

_class_ threading.Thread(_group=None_, _target=None_, _name=None_, _args=()_, _kwargs={}_, _*_, _daemon=None_, _context=None_)[¶](https://docs.python.org/3/library/threading.html#threading.Thread "Link to this definition")

Hàm tạo này luôn phải được gọi với các đối số từ khóa. Các đối số là: 

- Thuộc tính `group` nên là `None`; được dành riêng cho việc mở rộng trong tương lai khi lớp `ThreadGroup` được triển khai. `
- `target` là đối tượng có thể gọi được sẽ được phương thức `run()` gọi. Mặc định là `None`, nghĩa là không có gì được gọi.
- `name` là tên của luồng. Theo mặc định, một tên duy nhất được tạo ra có dạng “Thread-N” trong đó N là một số thập phân nhỏ, hoặc “Thread-N (target)” trong đó “target” là target.__name__ nếu đối số target được chỉ định.
- `args` là một danh sách hoặc bộ các đối số cho lời gọi hàm mục tiêu. Mặc định là `(`).
- ` kwargs `là một từ điển chứa các đối số từ khóa cho lời gọi hàm mục tiêu. Mặc định là {}. 
- Nếu không phải là None, daemon sẽ thiết lập rõ ràng xem luồng đó có phải là luồng nền hay không. Nếu là None (mặc định), thuộc tính daemonic sẽ được kế thừa từ luồng hiện tại.

`context` là giá trị `Context` được sử dụng khi khởi tạo luồng. Giá trị mặc định là `None`, cho biết cờ `sys.flags.thread_inherit_context` kiểm soát hành vi. Nếu cờ là `true`, các luồng sẽ bắt đầu với một bản sao của ngữ cảnh của người gọi phương thức `start()`. Nếu là `false`, chúng sẽ bắt đầu với một ngữ cảnh trống. Để bắt đầu rõ ràng với một ngữ cảnh trống, hãy truyền một thể hiện mới của `Context()`. Để bắt đầu rõ ràng với một bản sao của ngữ cảnh hiện tại, hãy truyền giá trị từ `copy_context()`. Cờ này mặc định là `true` trên các bản dựng free-threaded và `false` trong các trường hợp khác.

Nếu lớp con ghi đè phương thức khởi tạo, nó phải đảm bảo gọi phương thức khởi tạo của lớp cơ sở (Thread.__init__()) trước khi thực hiện bất kỳ thao tác nào khác với luồng.

**start()**

Bắt đầu hoạt động của thread.

Hàm này chỉ được gọi tối đa một lần cho mỗi đối tượng luồng. Nó sắp xếp để phương thức run() của đối tượng được gọi trong một luồng điều khiển riêng biệt.

Phương thức này sẽ gây ra lỗi RuntimeError nếu được gọi nhiều hơn một lần trên cùng một đối tượng luồng.

Nếu được hỗ trợ, hãy đặt tên luồng hệ điều hành thành `threading.Thread.name`. Tên này có thể bị cắt ngắn tùy thuộc vào giới hạn tên luồng của hệ điều hành.

**run()**

Phương thức thể hiện hoạt động của luồng.

Bạn có thể ghi đè phương thức này trong lớp con. Phương thức run() chuẩn sẽ gọi đối tượng có thể gọi được truyền vào hàm tạo của đối tượng dưới dạng đối số mục tiêu, nếu có, với các đối số vị trí và từ khóa được lấy từ các đối số args và kwargs tương ứng.

Việc sử dụng danh sách hoặc bộ dữ liệu làm đối số `args` được truyền cho luồng có thể đạt được hiệu quả tương tự.

Ví dụ:

```python
from threading import Thread
t = Thread(target=print, args=[1])
t.run()

t = Thread(target=print, args=(1,))
t.run()
```

**join(_timeout=None_)**

Chờ cho đến khi luồng kết thúc. Thao tác này sẽ chặn luồng gọi cho đến khi luồng có phương thức join() được gọi kết thúc – một cách bình thường hoặc do một ngoại lệ không được xử lý – hoặc cho đến khi hết thời gian chờ tùy chọn.

Khi tham số timeout xuất hiện và không phải là None, nó phải là một số thực chỉ định thời gian chờ cho thao tác tính bằng giây (hoặc phần thập phân của giây). Vì join() luôn trả về None, bạn phải gọi is_alive() sau khi gọi join() để quyết định xem có xảy ra lỗi hết thời gian chờ hay không – nếu luồng vẫn còn hoạt động, thì lệnh gọi join() đã hết thời gian chờ.

Một thread chỉ có thể joined nhiều lần.

Phương thức `join()` sẽ ném ra lỗi `RuntimeError` nếu cố gắng tham gia vào luồng hiện tại vì điều đó sẽ gây ra tình trạng bế tắc. Việc tham gia vào một luồng trước khi nó được khởi tạo cũng là một lỗi và các nỗ lực thực hiện điều đó sẽ gây ra cùng một ngoại lệ.

Nếu cố gắng tham gia vào một luồng nền đang chạy ở giai đoạn cuối của quá trình hoàn tất Python, hàm join() sẽ gây ra lỗi PythonFinalizationError.

**name**[](https://docs.python.org/3/library/threading.html#threading.Thread.name "Link to this definition")

Một chuỗi ký tự chỉ được sử dụng cho mục đích nhận dạng. Nó không có ý nghĩa ngữ nghĩa. Nhiều luồng có thể được đặt cùng một tên. Tên ban đầu được thiết lập bởi hàm tạo.

getName()[¶](https://docs.python.org/3/library/threading.html#threading.Thread.getName "Link to this definition")

setName()[¶](https://docs.python.org/3/library/threading.html#threading.Thread.setName "Link to this definition")

**ident**

Mã định danh luồng (thread identifier) ​​của luồng này hoặc None nếu luồng chưa được khởi tạo. Đây là một số nguyên khác 0. Xem hàm get_ident(). Mã định danh luồng có thể được tái sử dụng khi một luồng kết thúc và một luồng khác được tạo. Mã định danh vẫn khả dụng ngay cả sau khi luồng đã kết thúc.

**native_id**

Mã định danh luồng (Thread ID - TID) của luồng này, được hệ điều hành (kernel) gán. Đây là một số nguyên không âm, hoặc None nếu luồng chưa được khởi tạo. Xem hàm get_native_id(). Giá trị này có thể được sử dụng để xác định duy nhất luồng cụ thể này trên toàn hệ thống (cho đến khi luồng kết thúc, sau đó giá trị này có thể được hệ điều hành tái sử dụng).

>[!note]
>Tương tự như ID tiến trình, ID luồng chỉ có giá trị (đảm bảo duy nhất trên toàn hệ thống) từ thời điểm luồng được tạo cho đến khi luồng kết thúc.

**is_alive()**

Trả về trạng thái hoạt động của luồng.

Phương thức này trả về True ngay trước khi phương thức run() bắt đầu cho đến ngay sau khi phương thức run() kết thúc. Hàm module enumerate() trả về một danh sách tất cả các luồng đang hoạt động.

**daemon**

Một giá trị boolean cho biết luồng này có phải là luồng nền (True) hay không (False). Giá trị này phải được thiết lập trước khi gọi phương thức start(), nếu không sẽ xảy ra lỗi RuntimeError. Giá trị ban đầu của nó được kế thừa từ luồng tạo ra nó; luồng chính không phải là luồng nền và do đó tất cả các luồng được tạo trong luồng chính đều mặc định là daemon = False.

Toàn bộ chương trình Python sẽ thoát khi không còn luồng nào không phải là luồng nền (daemon) đang hoạt động.

isDaemon()

setDaemon()

### Lock objects

A primitive lock là một cơ chế đồng bộ hóa mà khi bị khóa, nó không thuộc sở hữu của một luồng cụ thể nào. Trong Python, hiện tại nó là cơ chế đồng bộ hóa cấp thấp nhất hiện có, được triển khai trực tiếp bởi mô-đun mở rộng _thread.

Một primitive lock có một trong hai trạng thái: "locked" hoặc "unlocked". Nó được tạo ra ở trạng thái unlocked. Nó có hai phương thức cơ bản: `acquire()` và `release()`. Khi trạng thái là unlocked, phương thức `acquire()` sẽ thay đổi trạng thái thành locked và trả về ngay lập tức. Khi trạng thái là locked, phương thức `acquire()` sẽ chặn cho đến khi một lệnh gọi đến `release()` trong một luồng khác thay đổi trạng thái thành unlocked, sau đó lệnh gọi `acquire()` sẽ đặt lại trạng thái thành locked và trả về. Phương thức `release()` chỉ nên được gọi trong trạng thái locked; nó sẽ thay đổi trạng thái thành unlocked và trả về ngay lập tức. Nếu cố gắng giải phóng một khóa đang ở trạng thái unlocked, một lỗi RuntimeError sẽ được báo cáo.

Khóa cũng hỗ trợ [context management protocol](https://docs.python.org/3/library/threading.html#with-locks)

Khi có nhiều hơn một luồng bị chặn trong phương thức `acquire()` để chờ trạng thái chuyển sang unlocked, chỉ có một luồng tiếp tục khi lệnh `release()` đặt lại trạng thái về unlocked; luồng nào trong số các luồng đang chờ sẽ tiếp tục không được xác định và có thể khác nhau giữa các triển khai.

Tất cả các phương thức đều được thực thi một cách nguyên tử (atomic).

**_class_ threading.Lock**

Lớp này triển khai các đối tượng khóa cơ bản. Một khi một luồng đã giành được khóa, các nỗ lực giành lại khóa tiếp theo sẽ bị chặn cho đến khi nó được giải phóng; bất kỳ luồng nào cũng có thể giải phóng nó.

- acquire(_blocking=True_, _timeout=-1_)[¶](https://docs.python.org/3/library/threading.html#threading.Lock.acquire "Link to this definition")
- release()[¶](https://docs.python.org/3/library/threading.html#threading.Lock.release "Link to this definition")
- locked(): Trả về True nếu khóa được giành quyền.

### RLock objects

A reentrant lock là một cơ chế đồng bộ hóa có thể được cùng một luồng chiếm giữ nhiều lần. Về mặt nội bộ, nó sử dụng các khái niệm "owning thread" và "ecursion level" ngoài trạng thái khóa/mở khóa được sử dụng bởi các khóa cơ bản. Ở trạng thái khóa, một luồng nào đó sở hữu khóa; ở trạng thái mở khóa, không có luồng nào sở hữu nó.

Các luồng gọi phương thức acquire() của khóa để khóa nó và phương thức release() để mở khóa.

>[!note]
>Khóa tái nhập hỗ trợ [context management protocol](https://docs.python.org/3/library/threading.html#with-locks), vì vậy nên sử dụng [`with`](https://docs.python.org/3/reference/compound_stmts.html#with) thay vì gọi thủ công `acquire()` và `release()` để xử lý việc giành quyền và giải phóng khóa cho một khối mã.

Các cặp lệnh acquire()/release() của RLock có thể lồng nhau, không giống như acquire()/release() của Lock. Chỉ có lệnh release() cuối cùng (lệnh release() của cặp lệnh ngoài cùng) mới đặt lại khóa về trạng thái mở khóa và cho phép một luồng khác bị chặn trong acquire() tiếp tục hoạt động.

Phương thức acquire()/release() phải được sử dụng theo cặp: mỗi lần gọi acquire phải có một lần gọi release trong luồng đã giành được khóa. Việc không gọi release đủ số lần khóa đã được giành được có thể dẫn đến tình trạng bế tắc.

_class_ threading.RLock

- acquire(_blocking=True_, _timeout=-1_)[](https://docs.python.org/3/library/threading.html#threading.RLock.acquire "Link to this definition")
- release()[](https://docs.python.org/3/library/threading.html#threading.RLock.release "Link to this definition")
- locked()[](https://docs.python.org/3/library/threading.html#threading.RLock.locked "Link to this definition")

### Condition objects

Biến điều kiện luôn được liên kết với một loại khóa nào đó; khóa này có thể được truyền vào hoặc sẽ được tạo mặc định. Việc truyền vào một khóa rất hữu ích khi nhiều biến điều kiện phải chia sẻ cùng một khóa. Khóa là một phần của đối tượng điều kiện: bạn không cần phải theo dõi nó riêng biệt.

Biến điều kiện tuân theo giao thức quản lý ngữ cảnh: việc sử dụng câu lệnh `with` sẽ giành được khóa liên kết trong suốt thời gian thực thi của khối lệnh bên trong. Các phương thức `acquire()` và `release()` cũng gọi các phương thức tương ứng của khóa liên kết.

Biến điều kiện tuân theo [context management protocol](https://docs.python.org/3/library/threading.html#with-locks): việc sử dụng câu lệnh `with` sẽ giành được khóa liên kết trong suốt thời gian thực thi của khối lệnh bên trong. Các Các phương thức khác phải được gọi khi khóa liên kết đang được giữ. Phương thức wait() giải phóng khóa, sau đó chặn cho đến khi một luồng khác đánh thức nó bằng cách gọi notify() hoặc notify_all(). Sau khi được đánh thức, wait() sẽ giành lại khóa và trả về. Cũng có thể chỉ định thời gian chờ.phương thức `acquire()` và `release()` cũng gọi các phương thức tương ứng của khóa liên kết.

Các phương thức khác phải được gọi khi khóa liên kết đang được giữ. Phương thức `wait()` giải phóng khóa, sau đó chặn cho đến khi một luồng khác đánh thức nó bằng cách gọi `notify()` hoặc `notify_all()`. Sau khi được đánh thức, `wait()` sẽ giành lại khóa và trả về. Cũng có thể chỉ định thời gian chờ.

Lưu ý: các phương thức `notify()` và `notify_all()` không giải phóng khóa; điều này có nghĩa là luồng hoặc các luồng được đánh thức sẽ không trả về ngay lập tức từ lệnh `wait()` của chúng, mà chỉ khi luồng đã gọi `notify()` hoặc `notify_all()` cuối cùng từ bỏ quyền sở hữu khóa.

Phong cách lập trình điển hình sử dụng biến điều kiện sử dụng khóa để đồng bộ hóa quyền truy cập vào một trạng thái được chia sẻ; các luồng quan tâm đến một thay đổi trạng thái cụ thể sẽ gọi `wait()` liên tục cho đến khi chúng thấy trạng thái mong muốn, trong khi các luồng sửa đổi trạng thái sẽ gọi `notify()` hoặc `notify_all()` khi chúng thay đổi trạng thái theo cách mà nó có thể là trạng thái mong muốn của một trong các luồng đang chờ. Ví dụ, đoạn mã sau là một tình huống nhà sản xuất-người tiêu dùng chung với dung lượng bộ đệm không giới hạn:

```python
# Consume one item
with cv:
    while not an_item_is_available():
        cv.wait()
    get_an_available_item()

# Produce one item
with cv:
    make_an_item_available()
    cv.notify()
```

Việc kiểm tra điều kiện của ứng dụng bằng vòng lặp `while` là cần thiết vì phương thức `wait()` có thể trả về sau một khoảng thời gian dài tùy ý, và điều kiện đã kích hoạt cuộc gọi notify() có thể không còn đúng nữa. Đây là đặc điểm vốn có của lập trình đa luồng. Phương thức `wait_for()` có thể được sử dụng để tự động hóa việc kiểm tra điều kiện và đơn giản hóa việc tính toán thời gian chờ:

```python
# Consume an item
with cv:
    cv.wait_for(an_item_is_available)
    get_an_available_item()
```

Để lựa chọn giữa `notify()` và `notify_all()`, hãy xem xét liệu một thay đổi trạng thái có cần thiết cho chỉ một hay nhiều luồng đang chờ. Ví dụ: trong một tình huống producer-customer điển hình, việc thêm một mục vào bộ đệm chỉ cần đánh thức một luồng người tiêu dùng.

_class_ threading.Condition(_lock=None_)[¶](https://docs.python.org/3/library/threading.html#threading.Condition "Link to this definition")

- acquire(_*args_)[¶](https://docs.python.org/3/library/threading.html#threading.Condition.acquire "Link to this definition")
- release()[¶](https://docs.python.org/3/library/threading.html#threading.Condition.release "Link to this definition")
- locked()[¶](https://docs.python.org/3/library/threading.html#threading.Condition.locked "Link to this definition")
- wait(_timeout=None_)[¶](https://docs.python.org/3/library/threading.html#threading.Condition.wait "Link to this definition")
- wait_for(_predicate_, _timeout=None_)[¶](https://docs.python.org/3/library/threading.html#threading.Condition.wait_for "Link to this definition")
- notify(_n=1_)[¶](https://docs.python.org/3/library/threading.html#threading.Condition.notify "Link to this definition")
- notify_all()[¶](https://docs.python.org/3/library/threading.html#threading.Condition.notify_all "Link to this definition")

### Semaphore objects

Đây là một trong những cơ chế đồng bộ hóa lâu đời nhất trong lịch sử khoa học máy tính, được phát minh bởi nhà khoa học máy tính người Hà Lan thời kỳ đầu Edsger W. Dijkstra (ông đã sử dụng các tên `P()` và `V()` thay vì `acquire()` và `release()`).

Semaphore quản lý một bộ đếm nội bộ, bộ đếm này sẽ giảm đi sau mỗi lần gọi hàm `acquire()` và tăng lên sau mỗi lần gọi hàm `release()`. Bộ đếm không bao giờ được phép xuống dưới 0; khi hàm `acquire()` phát hiện bộ đếm bằng 0, nó sẽ chặn lại và chờ cho đến khi một luồng khác gọi hàm `release()`.

Semaphore cũng hỗ trợ [context management protocol](https://docs.python.org/3/library/threading.html#with-locks).

_class_ threading.Semaphore(_value=1_)

Lớp này triển khai các đối tượng semaphore. Một semaphore quản lý một bộ đếm nguyên tử biểu thị số lần gọi `release()` trừ đi số lần gọi `acquire()`, cộng với một giá trị ban đầu. Phương thức `acquire()` sẽ chặn nếu cần thiết cho đến khi nó có thể trả về mà không làm cho bộ đếm âm. Nếu không được cung cấp, giá trị mặc định là 1.

Tham số tùy chọn cung cấp giá trị ban đầu cho bộ đếm nội bộ; giá trị mặc định là 1. Nếu giá trị được cung cấp nhỏ hơn 0, lỗi ValueError sẽ được báo cáo.

- acquire(_blocking=True_, _timeout=None_)
	- Lấy tín hiệu cờ hiệu.
	- Khi được gọi mà không có đối số:
		- Nếu bộ đếm nội bộ lớn hơn 0 khi vào lệnh, hãy giảm nó đi 1 và trả về True ngay lập tức.
		- Nếu bộ đếm nội bộ bằng 0 khi vào hàm, hãy chặn cho đến khi được đánh thức bởi lệnh gọi release(). Sau khi được đánh thức (và bộ đếm lớn hơn 0), hãy giảm bộ đếm đi 1 và trả về True. Chính xác một luồng sẽ được đánh thức bởi mỗi lệnh gọi release(). Không nên dựa vào thứ tự các luồng được đánh thức.
	- Khi được gọi với tham số blocking được đặt thành False, sẽ không chặn. Nếu một lệnh gọi không có đối số sẽ chặn, hãy trả về False ngay lập tức; ngược lại, hãy thực hiện điều tương tự như khi được gọi mà không có đối số, và trả về True.
	- Khi được gọi với thời gian chờ khác None, hàm sẽ chặn trong tối đa timeout giây. Nếu thao tác acquire không hoàn tất thành công trong khoảng thời gian đó, hãy trả về False. Ngược lại, hãy trả về True.
- release(_n=1_)
	- Giải phóng một semaphore, tăng bộ đếm nội bộ lên n. Khi nó bằng 0 lúc bắt đầu và các luồng khác đang chờ nó lớn hơn 0 một lần nữa, hãy đánh thức n luồng đó.

_class_ threading.BoundedSemaphore(_value=1_)

Lớp triển khai các đối tượng semaphore giới hạn. Semaphore giới hạn kiểm tra để đảm bảo giá trị hiện tại của nó không vượt quá giá trị ban đầu. Nếu vượt quá, lỗi ValueError sẽ được ném ra. Trong hầu hết các trường hợp, semaphore được sử dụng để bảo vệ các tài nguyên có dung lượng hạn chế. Nếu semaphore được giải phóng quá nhiều lần, đó là dấu hiệu của một lỗi. Nếu không được cung cấp, giá trị mặc định là 1.

### [`Semaphore`](https://docs.python.org/3/library/threading.html#threading.Semaphore "threading.Semaphore") example[¶](https://docs.python.org/3/library/threading.html#semaphore-example "Link to this heading")

Semaphore thường được sử dụng để bảo vệ các tài nguyên có dung lượng hạn chế, ví dụ như máy chủ cơ sở dữ liệu. Trong bất kỳ trường hợp nào mà kích thước của tài nguyên là cố định, bạn nên sử dụng semaphore có giới hạn. Trước khi tạo bất kỳ luồng công nhân nào, luồng chính của bạn sẽ khởi tạo semaphore:

```python
maxconnections = 5
# ...
pool_sema = BoundedSemaphore(value=maxconnections)
```

Sau khi được tạo ra, các luồng công nhân sẽ gọi các phương thức acquire và release của semaphore khi cần kết nối với máy chủ:

```python
with pool_sema:
    conn = connectdb()
    try:
        # ... use connection ...
    finally:
        conn.close()
```

Việc sử dụng semaphore có giới hạn giúp giảm thiểu khả năng lỗi lập trình khiến semaphore được giải phóng nhiều hơn mức được cấp phát mà không bị phát hiện.

### Event objects

Đây là một trong những cơ chế đơn giản nhất để giao tiếp giữa các luồng: một luồng báo hiệu một sự kiện và các luồng khác chờ đợi sự kiện đó.

Một đối tượng sự kiện quản lý một cờ nội bộ có thể được đặt thành true bằng phương thức `set()` và đặt lại thành false bằng phương thức `clear()`. Phương thức `wait()` sẽ chặn cho đến khi cờ có giá trị true.

is_set()

Trả về True nếu và chỉ nếu cờ nội bộ là true.

Phương thức isSet là một bí danh đã lỗi thời cho phương thức này.

set()

Đặt cờ nội bộ thành true. Tất cả các luồng đang chờ cờ này trở thành true sẽ được đánh thức. Các luồng gọi wait() sau khi cờ này trở thành true sẽ không bị chặn.

clear()

Đặt lại cờ nội bộ về false. Sau đó, các luồng gọi wait() sẽ bị chặn cho đến khi set() được gọi để đặt lại cờ nội bộ về true.

wait(_timeout=None_)

Chặn cho đến khi cờ nội bộ là false và thời gian chờ (nếu có) chưa hết hạn. Giá trị trả về thể hiện lý do phương thức chặn này trả về; True nếu trả về vì cờ nội bộ được đặt thành true, hoặc False nếu có thời gian chờ và cờ nội bộ không trở thành true trong thời gian chờ đã cho.

Khi tham số timeout xuất hiện và không phải là None, nó phải là một số thực chỉ định thời gian chờ cho thao tác tính bằng giây, hoặc phần thập phân của giây.

Ví dụ:

```python
import threading
import time

event = threading.Event()

def worker(name):
    print(f"{name} đang đợi tín hiệu...")
    event.wait() # Dừng tại đây
    print(f"{name} đã nhận được tín hiệu và bắt đầu làm việc!")

# Tạo 3 luồng cùng đợi
threading.Thread(target=worker, args=("Luồng 1",)).start()
threading.Thread(target=worker, args=("Luồng 2",)).start()

print("Main: Đang chuẩn bị dữ liệu trong 3 giây...")
time.sleep(3)
event.set() # Phát tín hiệu cho TẤT CẢ các luồng đang đợi
```

Tái sử dụng:

```python
import threading
import time

# Tạo sự kiện: Có hộp đến vị trí hay chưa?
box_arrived = threading.Event()

def packing_machine():
    for i in range(1, 4): # Giả sử đóng gói 3 hộp
        print(f"--- Đang đợi hộp số {i} ---")
        
        # 1. Đợi tín hiệu đèn xanh
        box_arrived.wait() 
        
        print(f"Máy: Đã thấy hộp số {i}. Đang đóng gói...")
        time.sleep(1) # Giả sử đóng gói mất 1 giây
        print(f"Máy: Xong hộp số {i}!")
        
        # 2. QUAN TRỌNG: Sau khi xong, phải tắt đèn xanh đi (reset)
        # để lần lặp sau wait() mới có tác dụng chặn lại
        box_arrived.clear()
        print(f"Máy: Đã reset cảm biến, đợi hộp tiếp theo.\n")

def conveyor_sensor():
    for i in range(1, 4):
        time.sleep(2) # Cứ 2 giây có 1 hộp chạy đến
        print(f"Cảm biến: Hộp số {i} đã vào vị trí!")
        
        # 3. Phát tín hiệu cho máy chạy
        box_arrived.set()

# Chạy hệ thống
threading.Thread(target=packing_machine).start()
threading.Thread(target=conveyor_sensor).start()
```

### Timer objects

Lớp này đại diện cho một hành động chỉ nên được thực thi sau khi một khoảng thời gian nhất định trôi qua — một bộ hẹn giờ. Timer là một lớp con của Thread và do đó cũng hoạt động như một ví dụ về việc tạo các luồng tùy chỉnh.

Bộ hẹn giờ được khởi động, giống như các luồng, bằng cách gọi phương thức Timer.start. Bộ hẹn giờ có thể được dừng (trước khi hành động của nó bắt đầu) bằng cách gọi phương thức cancel(). Khoảng thời gian mà bộ hẹn giờ sẽ chờ trước khi thực hiện hành động của nó có thể không hoàn toàn giống với khoảng thời gian do người dùng chỉ định.

Ví dụ:

```python
def hello():
    print("hello, world")

t = Timer(30.0, hello)
t.start()  # after 30 seconds, "hello, world" will be printed
```

_class_ threading.Timer(_interval_, _function_, _args=None_, _kwargs=None_)

Tạo một bộ hẹn giờ sẽ chạy một hàm với các đối số `args` và đối số từ khóa `kwargs`, sau khi khoảng thời gian `interval` trôi qua. Nếu `args` là `None` (mặc định), thì một danh sách rỗng sẽ được sử dụng. Nếu `kwargs` là `None` (mặc định), thì một từ điển rỗng sẽ được sử dụng.

cancel()

Dừng bộ đếm thời gian và hủy bỏ việc thực thi hành động của bộ đếm thời gian. Điều này chỉ có tác dụng nếu bộ đếm thời gian vẫn đang ở trạng thái chờ.

```python
from threading import Timer

def show_chat_box():
    print("Chào bạn, tôi có thể giúp gì không?")

# Bắt đầu hẹn giờ 10 giây
t = Timer(10.0, show_chat_box)
t.start()

# Giả sử ở giây thứ 5, khách hàng tự bấm vào nút Chat rồi
# Chúng ta sẽ hủy cái hẹn giờ tự động kia đi
t.cancel() 
print("Khách đã chủ động chat, hủy hẹn giờ tự động.")
```

### Barrier objects

Lớp này cung cấp một cơ chế đồng bộ hóa đơn giản để sử dụng bởi một số lượng luồng cố định cần chờ đợi lẫn nhau. Mỗi luồng cố gắng vượt qua rào cản bằng cách gọi phương thức wait() và sẽ bị chặn cho đến khi tất cả các luồng đã thực hiện các lệnh gọi wait() của chúng. Tại thời điểm này, các luồng được giải phóng đồng thời.

Rào chắn có thể được tái sử dụng nhiều lần cho cùng số lượng luồng.

Ví dụ, đây là một cách đơn giản để đồng bộ hóa luồng máy khách và máy chủ:

```python
b = Barrier(2, timeout=5)

def server():
    start_server()
    b.wait()
    while True:
        connection = accept_connection()
        process_server_connection(connection)

def client():
    b.wait()
    while True:
        connection = make_connection()
        process_client_connection(connection)
```

_class_ threading.Barrier(_parties_, _action=None_, _timeout=None_)

Tạo một đối tượng rào chắn cho số lượng luồng tham gia. Một hành động, nếu được cung cấp, là một hàm có thể gọi được bởi một trong các luồng khi chúng được giải phóng. timeout là giá trị thời gian chờ mặc định nếu không có giá trị nào được chỉ định cho phương thức wait().

wait(_timeout=None_)

Vượt qua rào cản. Khi tất cả các luồng tham gia vào rào cản đã gọi hàm này, chúng sẽ được giải phóng đồng thời. Nếu có thời gian chờ được cung cấp, nó sẽ được ưu tiên sử dụng hơn bất kỳ thời gian chờ nào được cung cấp cho hàm tạo của lớp.

Giá trị trả về là một số nguyên trong khoảng từ 0 đến 1, khác nhau đối với mỗi luồng. Điều này có thể được sử dụng để chọn một luồng thực hiện một số tác vụ quản trị đặc biệt, ví dụ:

```python
i = barrier.wait()
if i == 0:
    # Only one thread needs to print this
    print("passed the barrier")
```

Nếu một hành động được cung cấp cho hàm tạo, một trong các luồng sẽ gọi hành động đó trước khi được giải phóng. Nếu cuộc gọi này gây ra lỗi, rào chắn sẽ được đặt vào trạng thái bị phá vỡ.

Nếu cuộc gọi bị hết thời gian chờ, rào chắn sẽ chuyển sang trạng thái bị hỏng.

Phương pháp này có thể gây ra ngoại lệ BrokenBarrierError nếu rào chắn bị phá vỡ hoặc đặt lại trong khi một luồng đang chờ.

reset()

Hãy trả về trạng thái mặc định, trống rỗng của rào chắn. Bất kỳ luồng nào đang chờ trên đó sẽ nhận được ngoại lệ BrokenBarrierError.

Lưu ý rằng việc sử dụng chức năng này có thể yêu cầu một số đồng bộ hóa bên ngoài nếu có các luồng khác mà trạng thái của chúng chưa được biết. Nếu một rào cản bị phá vỡ, tốt hơn hết là nên bỏ qua nó và tạo một rào cản mới.

abort()

Đánh dấu rào chắn ở trạng thái bị phá vỡ. Điều này khiến mọi lệnh gọi wait() hiện tại hoặc trong tương lai đều thất bại với lỗi BrokenBarrierError. Ví dụ, hãy sử dụng điều này nếu một trong các luồng cần hủy bỏ, để tránh tình trạng ứng dụng bị kẹt.

Có lẽ sẽ tốt hơn nếu chỉ cần tạo một rào chắn với giá trị thời gian chờ hợp lý để tự động ngăn chặn một trong các luồng gặp sự cố.

parties

Số lượng thread cần thiết để vượt qua rào cản.

n_waiting

Số lượng luồng hiện đang chờ trong hàng đợi.

broken

Một biến boolean có giá trị True nếu rào chắn đang ở trạng thái bị phá vỡ.

_exception_ threading.BrokenBarrierError[](https://docs.python.org/3/library/threading.html#threading.BrokenBarrierError "Link to this definition")

This exception, a subclass of [`RuntimeError`](https://docs.python.org/3/library/exceptions.html#RuntimeError "RuntimeError"), is raised when the [`Barrier`](https://docs.python.org/3/library/threading.html#threading.Barrier "threading.Barrier") object is reset or broken.

```python
import threading
import time

# Rào chắn cần 3 người (luồng)
barrier = threading.Barrier(3)

def player(name):
    print(f"{name} đã vào phòng, đang chờ đồng đội...")
    try:
        # Đợi tại rào chắn
        worker_id = barrier.wait()
        print(f"{name} bắt đầu chiến đấu! (ID luồng: {worker_id})")
    except threading.BrokenBarrierError:
        print(f"{name}: Rào chắn hỏng rồi, giải tán!")

# Tạo 3 người chơi
threading.Thread(target=player, args=("Người chơi A",)).start()
time.sleep(1)
threading.Thread(target=player, args=("Người chơi B",)).start()
time.sleep(2)
threading.Thread(target=player, args=("Người chơi C",)).start()
```

## Using locks, conditions, and semaphores in the `with` statement[¶](https://docs.python.org/3/library/threading.html#using-locks-conditions-and-semaphores-in-the-with-statement "Link to this heading")

Tất cả các đối tượng được cung cấp bởi mô-đun này có phương thức acquire và release đều có thể được sử dụng làm trình quản lý ngữ cảnh cho câu lệnh with. Phương thức acquire sẽ được gọi khi khối lệnh được vào và phương thức release sẽ được gọi khi khối lệnh được thoát. Do đó, đoạn mã sau đây:

```python
with some_lock:
    # do something...
```

tương đương với:

```python
some_lock.acquire()
try:
    # do something...
finally:
    some_lock.release()
```

Hiện tại, các đối tượng Lock, RLock, Condition, Semaphore và BoundedSemaphore có thể được sử dụng như các trình quản lý ngữ cảnh câu lệnh with.

Tái sử dụng:

```python
import threading
import time

# Rào chắn cần 3 người (luồng)
barrier = threading.Barrier(3)

def player(name):
    print(f"{name} đã vào phòng, đang chờ đồng đội...")
    try:
        # Đợi tại rào chắn
        worker_id = barrier.wait()
        print(f"{name} bắt đầu chiến đấu! (ID luồng: {worker_id})")
    except threading.BrokenBarrierError:
        print(f"{name}: Rào chắn hỏng rồi, giải tán!")

# Tạo 3 người chơi
threading.Thread(target=player, args=("Người chơi A",)).start()
time.sleep(1)
threading.Thread(target=player, args=("Người chơi B",)).start()
time.sleep(2)
threading.Thread(target=player, args=("Người chơi C",)).start()
```

Nguồn: https://docs.python.org/3/library/threading.html

------------------------------------------------------------------

Tóm tắt:

Một process sẽ tạo ra được nhiều thread và bị giới hạn bởi GIL
Các thread chung bộ nhớ ở trong một process
- Thread main: 
	- Vẫn tiếp tục chạy ngay cả khi chương trình python kết thúc
	- Các thread con tạo từ thread cha cũng là thread main do kế thừa thuộc tính từ luồng cha
	- Khởi tạo một thread bằng phương thức run()
	- Chờ đợi một thread bằng phương thức join()
- Thread daemon:
	- Sẽ dừng đột ngột ngay khi chương trình python kết thúc

-Nếu `Semaphore` là máy đếm thẻ bài, thì **Event** giống như một chiếc **Đèn giao thông** đơn giản chỉ có hai màu: **Xanh** (True) và **Đỏ** (False).

**Timer** là một loại luồng đặc biệt trong Python (một lớp con của `Thread`). Thay vì chạy ngay lập tức khi bạn gọi `.start()`, nó sẽ **đợi** một khoảng thời gian nhất định rồi mới thực hiện công việc.

Nói một cách dễ hiểu, `Timer` giống như một cái **Đồng hồ hẹn giờ nấu ăn**. Bạn đặt giờ, nó đếm ngược, và khi hết giờ thì nó "reng reng" (thực hiện hàm bạn giao cho).

Nếu `Event` là một hiệu lệnh bắn súng để tất cả cùng chạy, thì **Barrier** (Rào chắn) giống như một **"Điểm tập kết"** hoặc **"Trạm thu phí"**.

Nó được dùng khi bạn có một nhóm luồng và bạn muốn đảm bảo rằng **không ông nào được đi tiếp cho đến khi tất cả các ông khác đã đến đông đủ tại điểm đó.**
