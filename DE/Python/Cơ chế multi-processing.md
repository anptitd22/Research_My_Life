Multiprocessing là một package hỗ trợ tạo ra các tiến trình sử dụng một API tương tự như [`threading`](https://docs.python.org/3/library/threading.html#module-threading "threading: Thread-based parallelism.") module. Multiprocessing package cung cấp cả local và remote concurrency, giải quyết hiệu quả vấn đề [Global Interpreter Lock](https://docs.python.org/3/glossary.html#term-global-interpreter-lock) bằng cách sử dụng các tiến trình con thay vì luồng. Cùng với đó, Multiprocessing module cho phép lập trình viên tận dụng tối đa khả năng của nhiều bộ xử lý trên một máy tính nhất định. Nó chạy trên cả hệ điều hành POSIX và Windows.

Multiprocessing module cũng giới thiệu  [`Pool`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.pool.Pool "multiprocessing.pool.Pool") object nó cung cấp một phương tiện thuận lợi cho việc tính toán parallelizing  cho nhiều hàm giá trị đầu vào, phân phối dữ liệu đầu vào cho các tiến trình (data parallelism). Ví dụ:

```
from multiprocessing import Pool

def f(x):
    return x*x

if __name__ == '__main__':
    with Pool(5) as p:
        print(p.map(f, [1, 2, 3]))
```

Multiprocessing modul cũng giới thiệu các API không có tương tự trong [`threading`](https://docs.python.org/3/library/threading.html#module-threading "threading: Thread-based parallelism.") module, chẳng hạn như [`terminate`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Process.terminate "multiprocessing.Process.terminate"), [`interrupt`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Process.interrupt "multiprocessing.Process.interrupt") or [`kill`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Process.kill "multiprocessing.Process.kill")một tiến trình đang chạy.

### The [`Process`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Process "multiprocessing.Process") class

Trong multiprocessing, các tiến trình được tạo ra bằng cách tạo một đối tượng [`Process`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Process "multiprocessing.Process") và sau đó gọi phương thức [`start()`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Process.start "multiprocessing.Process.start") của nó. Process tuân theo API của [`threading.Thread`](https://docs.python.org/3/library/threading.html#threading.Thread "threading.Thread"). Một ví dụ đơn giản về multiprocess là:

```
from multiprocessing import Process

def f(name):
    print('hello', name)

if __name__ == '__main__':
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
```

Để minh họa các ID process riêng lẻ liên quan, đây là một ví dụ mở rộng.

```
from multiprocessing import Process
import os

def info(title):
    print(title)
    print('module name:', __name__)
    print('parent process:', os.getppid())
    print('process id:', os.getpid())

def f(name):
    info('function f')
    print('hello', name)

if __name__ == '__main__':
    info('main line')
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
```

Để hiểu rõ hơn lý do tại sao cần có phần `if __name__ == '__main__'`, hãy xem [Programming guidelines](https://docs.python.org/3/library/multiprocessing.html#multiprocessing-programming).

Các arguments của hàm [`Process`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Process "multiprocessing.Process") thường cần phải không thể được giải mã (unpickleable) từ bên trong tiến trình con. Nếu bạn thử gõ trực tiếp ví dụ trên vào REPL, nó có thể dẫn đến lỗi [`AttributeError`](https://docs.python.org/3/library/exceptions.html#AttributeError "AttributeError") trong tiến trình con khi cố gắng định vị hàm `f` trong mô-đun __main__.

