asyncio là một thư viện dùng để viết mã xử lý **concurrent** bằng cú pháp async/await.

Asyncio được sử dụng làm nền tảng cho nhiều framework bất đồng bộ Python, cung cấp các máy chủ web và mạng hiệu năng cao, thư viện kết nối cơ sở dữ liệu, hàng đợi tác vụ phân tán, v.v.

```python
import asyncio

async def main():
    print('Hello ...')
    await asyncio.sleep(1)
    print('... World!')

asyncio.run(main())
```

asyncio thường là lựa chọn hoàn hảo cho IO-bound và high-level **structured** network code

asyncio cung cấp một bộ **high-level** APIs để:

- [run Python coroutines](https://docs.python.org/3/library/asyncio-task.html#coroutine) đồng thời và có toàn quyền kiểm soát quá trình thực thi của chúng;
- Thực hiện [network IO and IPC](https://docs.python.org/3/library/asyncio-stream.html#asyncio-streams)
- điều khiển [subprocesses](https://docs.python.org/3/library/asyncio-subprocess.html#asyncio-subprocess)
- phân phối task thông qua [queues](https://docs.python.org/3/library/asyncio-queue.html#asyncio-queues);
- [synchronize](https://docs.python.org/3/library/asyncio-sync.html#asyncio-sync) mã đồng thời

Ngoài ra, còn có các **low-level** API dành cho các nhà phát triển thư viện và framework để:

- Tạo và quản lý các [event loops](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio-event-loop), cung cấp các API bất đồng bộ cho việc kết nối [networking](https://docs.python.org/3/library/asyncio-eventloop.html#loop-create-server), chạy các [subprocesses](https://docs.python.org/3/library/asyncio-eventloop.html#loop-subprocess-exec), xử lý [OS signals](https://docs.python.org/3/library/asyncio-eventloop.html#loop-add-signal-handler), v.v.
- Triển khai các giao thức hiệu quả bằng cách sử dụng các [transports](https://docs.python.org/3/library/asyncio-protocol.html#asyncio-transports-protocols);
- Kết nối các thư viện dựa trên hàm gọi lại và mã có cú pháp async/await.

Tình trạng sẵn có: không có sẵn tại WASI.

Mô-đun này không hoạt động hoặc không khả dụng trên WebAssembly. Xem các nền tảng WebAssembly để biết thêm thông tin.

asyncio REPL

Bạn có thể thử nghiệm với `asyncio` concurrent context trong [REPL](https://docs.python.org/3/glossary.html#term-REPL):

```python
import asyncio
await asyncio.sleep(10, result='hello')
```

REPL này có khả năng tương thích hạn chế với [`PYTHON_BASIC_REPL`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHON_BASIC_REPL). Nên sử dụng REPL mặc định để có đầy đủ chức năng và các tính năng mới nhất.

Tạo ra một sự kiện kiểm toán cpython.run_stdin mà không có đối số nào.

Reference

High-level APIs

- [Runners](https://docs.python.org/3/library/asyncio-runner.html)
- [Coroutines and tasks](https://docs.python.org/3/library/asyncio-task.html)
- [Streams](https://docs.python.org/3/library/asyncio-stream.html)
- [Synchronization Primitives](https://docs.python.org/3/library/asyncio-sync.html)
- [Subprocesses](https://docs.python.org/3/library/asyncio-subprocess.html)
- [Queues](https://docs.python.org/3/library/asyncio-queue.html)
- [Exceptions](https://docs.python.org/3/library/asyncio-exceptions.html)
- [Call Graph Introspection](https://docs.python.org/3/library/asyncio-graph.html)

Low-level APIs

- [Event loop](https://docs.python.org/3/library/asyncio-eventloop.html)
- [Futures](https://docs.python.org/3/library/asyncio-future.html)
- [Transports and Protocols](https://docs.python.org/3/library/asyncio-protocol.html)
- [Policies](https://docs.python.org/3/library/asyncio-policy.html)
- [Platform Support](https://docs.python.org/3/library/asyncio-platforms.html)
- [Extending](https://docs.python.org/3/library/asyncio-extending.html)

Guides and Tutorials

- [High-level API Index](https://docs.python.org/3/library/asyncio-api-index.html)
- [Low-level API Index](https://docs.python.org/3/library/asyncio-llapi-index.html)
- [Developing with asyncio](https://docs.python.org/3/library/asyncio-dev.html)

------------------------------------------------------------------
# A Conceptual Overview of `asyncio`

Có thể bạn đang thắc mắc về một số khái niệm quan trọng của asyncio. Sau khi đọc xong bài viết này, bạn sẽ có thể tự tin trả lời những câu hỏi sau:

- Điều gì đang diễn ra đằng sau hậu trường khi một object đang được chờ đợi?
- Asyncio phân biệt như thế nào giữa một tác vụ không cần thời gian xử lý của CPU (chẳng hạn như yêu cầu mạng hoặc đọc tập tin) so với một tác vụ cần thời gian xử lý của CPU (chẳng hạn như tính giai thừa n)?
- Làm thế nào để viết một biến thể bất đồng bộ của một thao tác, chẳng hạn như lệnh async sleep or database request.

## A conceptual overview part 1: the high-level

Trong phần 1, chúng ta sẽ tìm hiểu các thành phần cơ bản, cấp cao của asyncio: vòng lặp sự kiện, hàm coroutine, đối tượng coroutine, tác vụ và await.

### Event Loop

Được ví như "nhạc trưởng" điều phối mọi thứ. Vòng lặp chứa một danh sách các công việc (jobs) cần chạy, nó sẽ lần lượt lấy từng công việc ra và cấp quyền điều khiển cho công việc đó. Khi công việc tạm dừng hoặc chạy xong, quyền điều khiển được trả lại cho vòng lặp để tiếp tục với công việc khác. Để hệ thống hiệu quả, các công việc phải hoạt động hợp tác và không được chiếm dụng tài nguyên quá lâu. Nếu không còn tác vụ nào đang chờ thực thi, vòng lặp sự kiện đủ thông minh để nghỉ ngơi và tránh lãng phí chu kỳ CPU một cách không cần thiết, và sẽ quay lại khi có thêm việc cần làm.

Việc thực thi hiệu quả phụ thuộc vào việc các công việc chia sẻ và hợp tác tốt với nhau; một công việc 'tham lam' có thể chiếm quyền điều khiển và khiến các công việc khác bị 'bỏ đói', làm cho phương pháp vòng lặp sự kiện tổng thể trở nên khá vô dụng.

```python
import asyncio

# This creates an event loop and indefinitely cycles through
# its collection of jobs.
event_loop = asyncio.new_event_loop()
event_loop.run_forever()
```

### Asynchronous functions and coroutines

Đây là một hàm Python cơ bản, khá nhàm chán:

```python
def hello_printer():
    print(
        "Hi, I am a lowly, simple printer, though I have all I "
        "need in life -- \nfresh paper and my dearly beloved octopus "
        "partner in crime."
    )
```

Việc gọi một hàm thông thường sẽ kích hoạt logic hoặc phần thân của hàm đó:

```
>> hello_printer()
Hi, I am a lowly, simple printer, though I have all I need in life --
fresh paper and my dearly beloved octopus partner in crime.
```

Từ khóa [async def](https://docs.python.org/3/reference/compound_stmts.html#async-def), khác với từ khóa `def` thông thường, biến nó thành một hàm bất đồng bộ (hay còn gọi là "hàm coroutine"). Việc gọi từ khóa này sẽ tạo ra và trả về một đối tượng [coroutine](https://docs.python.org/3/library/asyncio-task.html#coroutine).

```python
async def loudmouth_penguin(magic_number: int):
    print(
     "I am a super special talking penguin. Far cooler than that printer. "
     f"By the way, my lucky number is: {magic_number}."
    )
```

Việc gọi hàm bất đồng bộ `loudmouth_penguin` không thực thi câu lệnh print; thay vào đó, nó tạo ra một đối tượng coroutine:

```
loudmouth_penguin(magic_number=3)
<coroutine object loudmouth_penguin at 0x104ed2740>
```

Các thuật ngữ “hàm coroutine” và “đối tượng coroutine” thường bị nhầm lẫn với nhau thành coroutine. Điều đó có thể gây nhầm lẫn! Trong bài viết này, coroutine được định nghĩa cụ thể là một đối tượng coroutine, hay chính xác hơn, là một thể hiện của [`types.CoroutineType`](https://docs.python.org/3/library/types.html#types.CoroutineType "types.CoroutineType") (coroutine gốc). Lưu ý rằng coroutine cũng có thể tồn tại dưới dạng các thể hiện của [`collections.abc.Coroutine`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Coroutine "collections.abc.Coroutine") – một sự khác biệt quan trọng đối với việc kiểm tra kiểu dữ liệu.

Coroutine đại diện cho phần thân hoặc logic của hàm. Một coroutine phải được khởi tạo một cách rõ ràng; nói cách khác, chỉ tạo ra coroutine thôi thì chưa đủ để khởi động nó. Đặc biệt, coroutine có thể được tạm dừng và tiếp tục tại nhiều điểm khác nhau trong phần thân của hàm. Khả năng tạm dừng và tiếp tục đó chính là điều cho phép hoạt động bất đồng bộ!

Các coroutine và hàm coroutine được xây dựng bằng cách tận dụng chức năng của các [generators](https://docs.python.org/3/glossary.html#term-generator-iterator) và [generator functions](https://docs.python.org/3/glossary.html#term-generator). Hãy nhớ lại, hàm generator là một hàm trả về [`yield`](https://docs.python.org/3/reference/simple_stmts.html#yield), giống như hàm này:

```python
def get_random_number():
    # This would be a bad random number generator!
    print("Hi")
    yield 1
    print("Hello")
    yield 7
    print("Howdy")
    yield 4
    ...
```

Tương tự như hàm coroutine, việc gọi một hàm generator không thực sự chạy hàm đó. Thay vào đó, nó tạo ra một đối tượng generator:

```
get_random_number()
<generator object get_random_number at 0x1048671c0>
```

Bạn có thể chuyển sang kết quả tiếp theo của trình tạo bằng cách sử dụng hàm tích hợp next(). Nói cách khác, trình tạo chạy, sau đó tạm dừng. Ví dụ:

```python
generator = get_random_number()
next(generator)
Hi
1
next(generator)
Hello
7
```

### Tasks

Nói một cách đơn giản, các [tasks](https://docs.python.org/3/library/asyncio-task.html#asyncio-task-obj) là các coroutine (không phải hàm coroutine) được gắn với một event loop. Một task cũng duy trì một danh sách các hàm gọi lại (callback), tầm quan trọng của chúng sẽ được làm rõ trong chốc lát khi chúng ta thảo luận về [`await`](https://docs.python.org/3/reference/expressions.html#await). Cách được khuyến nghị để tạo task là thông qua [`asyncio.create_task()`](https://docs.python.org/3/library/asyncio-task.html#asyncio.create_task "asyncio.create_task").

Việc create_task sẽ tự động lên lịch thực thi cho task đó (bằng cách thêm một hàm gọi (call back) lại để chạy nó trong danh sách việc cần làm của event loop, tức là tập hợp các công việc).

Asyncio tự động liên kết các task với event loop cho bạn. Việc liên kết tự động này được thiết kế có chủ đích trong asyncio vì mục đích đơn giản hóa. Nếu không có nó, bạn sẽ phải theo dõi đối tượng event loop và truyền nó cho bất kỳ hàm coroutine nào muốn tạo tác vụ, làm tăng thêm sự rườm rà không cần thiết cho mã của bạn.

```
coroutine = loudmouth_penguin(magic_number=5)
# This creates a Task object and schedules its execution via the event loop.
task = asyncio.create_task(coroutine)
```

Trước đây, chúng ta đã tự tạo event loop và thiết lập để nó chạy mãi mãi. Trên thực tế, nên sử dụng (và thường thấy) asyncio.run(), phương thức này sẽ event loop và đảm bảo coroutine được cung cấp kết thúc trước khi chuyển sang bước tiếp theo. Ví dụ, nhiều chương trình bất đồng bộ tuân theo thiết lập này:

```python
import asyncio

async def main():
    # Perform all sorts of wacky, wild asynchronous things...
    ...

if __name__ == "__main__":
    asyncio.run(main())
    # The program will not reach the following print statement until the
    # coroutine main() finishes.
    print("coroutine main() is done!")
```

Điều quan trọng cần lưu ý là bản thân task không được thêm vào event loop, mà chỉ có callback task được thêm vào. Điều này có ý nghĩa nếu đối tượng task bạn tạo ra bị thu hồi bộ nhớ trước khi được event loop. Ví dụ, hãy xem xét chương trình này:

```python
async def hello():
    print("hello!")

async def main():
    asyncio.create_task(hello())
    # Other asynchronous instructions which run for a while
    # and cede control to the event loop...
    ...

asyncio.run(main())
```

Vì không có **biến** tham chiếu nào đến đối tượng task được tạo ở dòng 5, nên nó có thể bị thu gom rác (garbage collected) trước khi event loop gọi nó. Các lệnh sau trong hàm main() của coroutine sẽ trả lại quyền điều khiển cho event loop để nó có thể gọi các task khác. Khi event loop cuối cùng cố gắng chạy task, nó có thể thất bại và phát hiện ra rằng đối tượng task không tồn tại! Điều này cũng có thể xảy ra ngay cả khi một coroutine giữ một tham chiếu đến một task nhưng hoàn thành trước khi task đó kết thúc. Khi coroutine thoát, các biến cục bộ sẽ nằm ngoài phạm vi và có thể bị thu gom rác. Trên thực tế, asyncio và trình thu gom rác của Python hoạt động khá tích cực để đảm bảo điều này không xảy ra. Nhưng đó không phải là lý do để bất cẩn!

### await

`await` là một từ khóa trong Python thường được sử dụng theo một trong hai cách khác nhau:

```python
await task
await coroutine
```

Về cơ bản, hành vi của await phụ thuộc vào loại đối tượng đang được chờ đợi.

Việc chờ đợi một task sẽ chuyển quyền điều khiển từ task hoặc coroutine hiện tại sang event loop. Trong quá trình chuyển giao quyền điều khiển, một vài điều quan trọng sẽ xảy ra. Chúng ta sẽ sử dụng ví dụ mã sau để minh họa:

```python
async def plant_a_tree():
    dig_the_hole_task = asyncio.create_task(dig_the_hole())
    await dig_the_hole_task

    # Other instructions associated with planting a tree.
    ...
```

Trong ví dụ này, hãy tưởng tượng event loop đã chuyển quyền điều khiển đến đầu coroutine plant_a_tree(). Như đã thấy ở trên, coroutine tạo một task và sau đó chờ đợi nó. Lệnh await dig_the_hole_task thêm một hàm gọi lại (callback) (sẽ tiếp tục thực thi plant_a_tree()) vào danh sách các hàm callback của đối tượng dig_the_hole_task. Và sau đó, lệnh này chuyển quyền điều khiển cho event loop. Một thời gian sau, event loop sẽ chuyển quyền điều khiển cho dig_the_hole_task và task sẽ hoàn thành bất cứ điều gì nó cần làm. Sau khi task hoàn thành, nó sẽ thêm các hàm gọi lại (callback) khác nhau của mình vào event loop, trong trường hợp này, là một lệnh gọi để tiếp tục thực thi plant_a_tree().

Nói chung, khi task được chờ đợi hoàn thành (dig_the_hole_task), task hoặc coroutine ban đầu (plant_a_tree()) sẽ được thêm lại vào danh sách việc cần làm của event loop để tiếp tục.

Đây là một mô hình tư duy cơ bản nhưng đáng tin cậy. Trên thực tế, việc chuyển giao quyền điều khiển phức tạp hơn một chút, nhưng không đáng kể. Trong phần 2, chúng ta sẽ đi sâu vào các chi tiết giúp điều này trở nên khả thi.

Không giống như các task, việc chờ đợi một coroutine không trả lại quyền điều khiển cho task! Việc gói một coroutine trong một task trước, rồi sau đó chờ đợi task đó sẽ dẫn đến việc trả lại quyền điều khiển. Hành vi của việc chờ đợi coroutine về cơ bản giống như việc gọi một hàm Python thông thường, đồng bộ. Hãy xem xét chương trình này:

```python
import asyncio

async def coro_a():
   print("I am coro_a(). Hi!")

async def coro_b():
   print("I am coro_b(). I sure hope no one hogs the event loop...")

async def main():
   task_b = asyncio.create_task(coro_b())
   num_repeats = 3
   for _ in range(num_repeats):
      await coro_a()
   await task_b

asyncio.run(main())
```

Câu lệnh đầu tiên trong coroutine `main()` tạo ra `task_b` và lên lịch thực thi nó thông qua event loop. Sau đó, `coro_a()` được chờ đợi liên tục. Quyền điều khiển không bao giờ được chuyển giao cho event loop, đó là lý do tại sao chúng ta thấy đầu ra của cả ba lần gọi `coro_a()` trước khi đầu ra của `coro_b()` được thực thi:

```
I am coro_a(). Hi!
I am coro_a(). Hi!
I am coro_a(). Hi!
I am coro_b(). I sure hope no one hogs the event loop...
```

Nếu ta thay đổi `await coro_a()` thành `await asyncio.create_task(coro_a())`, hành vi sẽ thay đổi. Hàm `main()` của coroutine sẽ nhường quyền điều khiển cho event loop với câu lệnh đó. Event loop sau đó sẽ tiếp tục xử lý các công việc đang chờ xử lý, gọi `task_b` và sau đó là task bao bọc `coro_a()` trước khi tiếp tục hàm `main()` của coroutine.

```
I am coro_b(). I sure hope no one hogs the event loop...
I am coro_a(). Hi!
I am coro_a(). Hi!
I am coro_a(). Hi!
```

Hành vi của coroutine `await` có thể gây nhầm lẫn cho rất nhiều người! Ví dụ đó cho thấy việc chỉ sử dụng coroutine `await` có thể vô tình chiếm quyền điều khiển từ các task khác và làm đình trệ event loop. `asyncio.run()` có thể giúp bạn phát hiện những trường hợp như vậy thông qua cờ `debug=True`, cho phép [debug mode](https://docs.python.org/3/library/asyncio-dev.html#asyncio-debug-mode). Trong số những thứ khác, nó sẽ ghi lại bất kỳ coroutine nào chiếm quyền thực thi trong 100ms trở lên.

Thiết kế này cố tình đánh đổi một phần sự rõ ràng về mặt khái niệm xung quanh việc sử dụng await để cải thiện hiệu suất. Mỗi khi một task được chờ đợi, quyền điều khiển cần được truyền lên toàn bộ ngăn xếp cuộc gọi đến event loop. Điều đó nghe có vẻ nhỏ nhặt, nhưng trong một chương trình lớn với nhiều câu lệnh await và callstack sâu, chi phí phát sinh đó có thể tích lũy thành một sự suy giảm hiệu suất đáng kể.

>[!note]
>Vậy await lồng nhau trong task nó sẽ là event loop + call stack

```python
async def rua_bat(): # Coroutine C

print(" C: Đang rửa bát...")

await asyncio.sleep(1) # <--- Điểm nhường quyền

print(" C: Rửa xong!")

  
async def nau_an(): # Coroutine B

print("B: Bắt đầu nấu...")

await rua_bat()

print("B: Nấu xong!")

  
async def main(): # Task A

print("A: Main bắt đầu.")

await nau_an()

print("A: Toàn bộ xong!")
```
## A conceptual overview part 2: the nuts and bolts

Phần 2 sẽ đi sâu vào chi tiết về các cơ chế mà asyncio sử dụng để quản lý luồng điều khiển. Đây là nơi điều kỳ diệu xảy ra. Sau khi đọc xong phần này, bạn sẽ hiểu được await hoạt động như thế nào ở phía sau hậu trường và cách tạo ra các toán tử bất đồng bộ của riêng mình.

### The inner workings of coroutines

Asyncio sử dụng bốn thành phần để truyền quyền điều khiển.

Phương thức `coroutine.send(arg)` được sử dụng để bắt đầu hoặc tiếp tục một coroutine. Nếu coroutine đã bị tạm dừng và hiện đang được tiếp tục, đối số `arg` sẽ được gửi vào như giá trị trả về của câu lệnh `yield` đã tạm dừng nó ban đầu. Nếu coroutine đang được sử dụng lần đầu tiên (trái ngược với việc được tiếp tục), `arg` phải là `None`.

```python
class Rock:
    def __await__(self):
        value_sent_in = yield 7
        print(f"Rock.__await__ resuming with value: {value_sent_in}.")
        return value_sent_in

async def main():
    print("Beginning coroutine main().")
    rock = Rock()
    print("Awaiting rock...")
    value_from_rock = await rock
    print(f"Coroutine received value: {value_from_rock} from rock.")
    return 23

coroutine = main()
intermediate_result = coroutine.send(None)
print(f"Coroutine paused and returned intermediate value: {intermediate_result}.")

print(f"Resuming coroutine and sending in value: 42.")
try:
    coroutine.send(42)
except StopIteration as e:
    returned_value = e.value
print(f"Coroutine main() finished and provided value: {returned_value}.")
```

Như thường lệ, lệnh `yield` tạm dừng quá trình thực thi và trả lại quyền điều khiển cho người gọi. Trong ví dụ trên, lệnh `yield` ở dòng 3 được gọi bởi `... = await rock` ở dòng 11. Nói một cách tổng quát hơn, `await` gọi phương thức `__await__()` của đối tượng được cung cấp. `await` cũng thực hiện một điều rất đặc biệt khác: nó truyền (hoặc "chuyển tiếp") bất kỳ lệnh `yield` nào mà nó nhận được lên chuỗi gọi. Trong trường hợp này, đó là trở lại `... = coroutine.send(None)` ở dòng 16.

Coroutine được tiếp tục thông qua lệnh coroutine.send(42) ở dòng 21. Coroutine tiếp tục từ nơi nó đã tạm dừng (hoặc dừng lại) ở dòng 3 và thực thi các câu lệnh còn lại trong thân của nó. Khi một coroutine kết thúc, nó sẽ đưa ra ngoại lệ StopIteration với giá trị trả về được đính kèm trong thuộc tính value.

Đoạn mã đó tạo ra kết quả như sau:

```
Beginning coroutine main().
Awaiting rock...
Coroutine paused and returned intermediate value: 7.
Resuming coroutine and sending in value: 42.
Rock.__await__ resuming with value: 42.
Coroutine received value: 42 from rock.
Coroutine main() finished and provided value: 23.
```

Bạn nên dừng lại một chút ở đây và đảm bảo rằng mình đã theo dõi các cách thức truyền tải luồng điều khiển và giá trị. Rất nhiều ý tưởng quan trọng đã được đề cập và điều đáng để bạn chắc chắn rằng mình đã hiểu rõ.

Cách duy nhất để nhường quyền điều khiển (hay nói cách khác là từ bỏ quyền điều khiển) từ một coroutine là sử dụng await để chờ một đối tượng thực hiện việc nhường quyền điều khiển trong phương thức __await__ của nó. Điều này nghe có vẻ lạ đối với bạn. Bạn có thể đang nghĩ:

1. Còn việc sử dụng lệnh `yield` trực tiếp trong hàm coroutine thì sao? Hàm coroutine sẽ trở thành một hàm tạo bất đồng bộ [async generator function](https://docs.python.org/3/reference/expressions.html#asynchronous-generator-functions), hoàn toàn khác biệt.
2. Còn việc sử dụng `yield from` bên trong hàm coroutine để gọi đến một generator (thông thường) thì sao? Điều đó gây ra lỗi: `SyntaxError: yield from not allowed in a coroutine`. Điều này được thiết kế có chủ ý vì mục đích đơn giản hóa – chỉ cho phép một cách sử dụng coroutine duy nhất. Ban đầu, `yield from` cũng bị cấm, nhưng sau đó được chấp nhận trở lại để cho phép sử dụng các generator bất đồng bộ. Mặc dù vậy, `yield from` và `await` về cơ bản thực hiện cùng một việc.

### Futures

[future](https://docs.python.org/3/library/asyncio-future.html#asyncio-future-obj)là một đối tượng được dùng để biểu thị trạng thái và kết quả của một phép tính. Thuật ngữ này ám chỉ ý tưởng về điều gì đó vẫn chưa xảy ra hoặc chưa diễn ra, và đối tượng này là một cách để theo dõi điều đó.

Một future có một vài thuộc tính quan trọng. Một trong số đó là trạng thái của nó, có thể là “pending”, “cancelled”, hoặc “done”. Một thuộc tính khác là kết quả của nó, được thiết lập khi trạng thái chuyển sang "đã hoàn thành". Không giống như coroutine, future không đại diện cho phép tính thực tế cần thực hiện; thay vào đó, nó đại diện cho trạng thái và kết quả của phép tính đó, giống như một đèn báo trạng thái (đỏ, vàng hoặc xanh lá cây) hoặc chỉ báo.

Lớp [`asyncio.Task`](https://docs.python.org/3/library/asyncio-task.html#asyncio.Task "asyncio.Task") kế thừa từ [`asyncio.Future`](https://docs.python.org/3/library/asyncio-future.html#asyncio.Future "asyncio.Future") để có được các khả năng khác nhau này. Phần trước nói rằng các task lưu trữ một danh sách các hàm gọi lại, điều này không hoàn toàn chính xác. Trên thực tế, chính lớp `Future` mới là lớp thực hiện logic này, và `Task` kế thừa từ nó.

Bạn cũng có thể sử dụng Futures trực tiếp (không thông qua Tasks). Tasks tự đánh dấu là hoàn thành khi coroutine của nó kết thúc. Futures linh hoạt hơn nhiều và sẽ được đánh dấu là hoàn thành khi bạn yêu cầu. Theo cách này, chúng là giao diện linh hoạt cho phép bạn tự đặt ra các điều kiện chờ đợi và tiếp tục.
### A homemade asyncio.sleep

Chúng ta sẽ xem xét một ví dụ về cách bạn có thể tận dụng future để tạo ra biến thể riêng của mình về chế độ ngủ bất đồng bộ (async_sleep), mô phỏng theo [`asyncio.sleep()`](https://docs.python.org/3/library/asyncio-task.html#asyncio.sleep "asyncio.sleep").

Đoạn mã này đăng ký một vài tác vụ với event loop và sau đó chờ task được tạo bởi asyncio.create_task, tác vụ này bao bọc coroutine async_sleep(3). Chúng ta muốn tác vụ đó chỉ kết thúc sau ba giây trôi qua, nhưng không ngăn cản các tác vụ khác chạy.

```python
async def other_work():
    print("I like work. Work work.")

async def main():
    # Add a few other tasks to the event loop, so there's something
    # to do while asynchronously sleeping.
    work_tasks = [
        asyncio.create_task(other_work()),
        asyncio.create_task(other_work()),
        asyncio.create_task(other_work())
    ]
    print(
        "Beginning asynchronous sleep at time: "
        f"{datetime.datetime.now().strftime("%H:%M:%S")}."
    )
    await asyncio.create_task(async_sleep(3))
    print(
        "Done asynchronous sleep at time: "
        f"{datetime.datetime.now().strftime("%H:%M:%S")}."
    )
    # asyncio.gather effectively awaits each task in the collection.
    await asyncio.gather(*work_tasks)
```

Bên dưới, chúng ta sử dụng một future để cho phép kiểm soát tùy chỉnh thời điểm task đó được đánh dấu là hoàn thành. Nếu future.set_result() (phương thức chịu trách nhiệm đánh dấu future đó là hoàn thành) không bao giờ được gọi, thì task này sẽ không bao giờ kết thúc. Chúng ta cũng đã nhờ đến sự trợ giúp của một task khác, mà chúng ta sẽ thấy ngay sau đây, task này sẽ theo dõi thời gian đã trôi qua và, tương ứng, gọi future.set_result().

Như thường lệ, event loop sẽ thực hiện các task của nó, trao quyền điều khiển cho chúng và nhận lại quyền điều khiển khi chúng tạm dừng hoặc kết thúc. Task watcher_task, chạy coroutine _sleep_watcher(...)_, sẽ được gọi một lần cho mỗi chu kỳ đầy đủ của event loop. Mỗi khi tiếp tục, nó sẽ kiểm tra thời gian và nếu chưa đủ thời gian trôi qua, nó sẽ tạm dừng một lần nữa và trả lại quyền điều khiển cho event loop. Khi đủ thời gian trôi qua, _sleep_watcher(...)_ sẽ đánh dấu future là đã hoàn thành và kết thúc bằng cách thoát khỏi vòng lặp while vô hạn của nó. Vì task hỗ trợ này chỉ được gọi một lần cho mỗi chu kỳ của event loop, bạn sẽ đúng khi nhận thấy rằng giấc ngủ không đồng bộ này sẽ ngủ ít nhất ba giây, chứ không phải chính xác ba giây. Lưu ý rằng điều này cũng đúng với asyncio.sleep.

```python
class YieldToEventLoop:
    def __await__(self):
        yield

async def _sleep_watcher(future, time_to_wake):
    while True:
        if time.time() >= time_to_wake:
            # This marks the future as done.
            future.set_result(None)
            break
        else:
            await YieldToEventLoop()
```

Đây là toàn bộ kết quả đầu ra của chương trình:

```
$ python custom-async-sleep.py
Beginning asynchronous sleep at time: 14:52:22.
I like work. Work work.
I like work. Work work.
I like work. Work work.
Done asynchronous sleep at time: 14:52:25.
```

Có thể bạn cảm thấy cách triển khai chế độ ngủ không đồng bộ này khá phức tạp. Và đúng vậy, nó đúng là như thế. Ví dụ này nhằm mục đích minh họa tính linh hoạt của futures với một ví dụ đơn giản có thể được áp dụng cho các nhu cầu phức tạp hơn. Để tham khảo, bạn có thể triển khai nó mà không cần futures, như sau:

```python
async def simpler_async_sleep(seconds):
    time_to_wake = time.time() + seconds
    while True:
        if time.time() >= time_to_wake:
            return
        else:
            await YieldToEventLoop()
```

Nhưng tạm thời đến đây là hết. Hy vọng bạn đã sẵn sàng tự tin hơn để tìm hiểu về lập trình bất đồng bộ hoặc xem các chủ đề nâng cao trong phần còn lại của tài liệu [`rest of the documentation`](https://docs.python.org/3/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O.").

Nguồn : https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html#a-conceptual-overview-of-asyncio

------------------------------------------------------------------
Extra:

`create_task` kích hoạt việc chạy ngay lập tức, còn `gather` (khi nhận coroutine) sẽ kích hoạt việc chạy tại dòng lệnh đó.

```python
tasks = [download(i) for i in range(5)] # (1)
await asyncio.gather(*tasks)           # (2)
```

- **Tại (1):** Bạn chỉ mới tạo ra một danh sách các "bản thiết kế" (Coroutine objects). **Chưa có code nào được chạy cả.**
- **Tại (2):** Khi hàm `gather` được gọi, nó mới nhìn vào danh sách này, tự mình gọi `create_task` cho từng cái, và bắt đầu chạy đồng thời tất cả.

```python
tasks = [asyncio.create_task(download(i)) for i in range(5)] # (1)
await asyncio.gather(*tasks)                                # (2)
```

- **Tại (1):** Ngay khi vòng lặp chạy đến `create_task`, các task này đã được **đăng ký vào Event Loop và bắt đầu chạy ngay lập tức** ở hậu trường (ngay cả khi bạn chưa gọi `await`).
- **Tại (2):** Dòng `gather` lúc này chỉ đóng vai trò là một "điểm tập kết", đứng đợi cho các task đã chạy từ trước đó hoàn thành.

```python
async def main():
    # Cách A: Dùng create_task
    task = asyncio.create_task(download(1))
    await asyncio.sleep(0.5) # Trong 0.5 giây này, task đã chạy được một nửa!
    await task # Chỉ cần đợi thêm 0.5 giây nữa là xong.

    # Cách B: Dùng Coroutine trực tiếp
    coro = download(2)
    await asyncio.sleep(0.5) # Trong 0.5 giây này, coro chả làm gì cả, nó chỉ là object nằm yên.
    await coro # Bây giờ mới bắt đầu chạy, phải đợi đủ 1 giây nữa.
```

