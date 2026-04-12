**yield_atom**:                           "(" [`yield_expression`](https://docs.python.org/3/reference/expressions.html#grammar-token-python-grammar-yield_expression) ")"
**yield_from**:                            "yield" "from" [`expression`](https://docs.python.org/3/reference/expressions.html#grammar-token-python-grammar-expression)
**yield_expression**:                "yield"  [`yield_list`](https://docs.python.org/3/reference/expressions.html#grammar-token-python-grammar-yield_list) | [`yield_from`](https://docs.python.org/3/reference/expressions.html#grammar-token-python-grammar-yield_from)

Khi một hàm có chứa từ khóa `yield`, nó trở thành **Generator Function**. Khi bạn gọi hàm này, nó **không chạy code bên trong ngay lập tức**, mà nó trả về một đối tượng gọi là **Generator**(đối tượng có thể lặp). 

- **Lần gọi đầu tiên (`next()`):** Hàm chạy từ đầu đến khi gặp dòng `yield` đầu tiên. Nó trả về giá trị sau `yield` và dừng lại.
- **Lần gọi tiếp theo:** Hàm chạy tiếp từ nơi nó vừa dừng. Toàn bộ biến cục bộ từ lần trước vẫn còn nguyên.
- **Khi hết code:** Nó sẽ tung ra lỗi `StopIteration` để báo rằng "tôi hết hàng rồi".

###  1. Sự khác biệt cốt lõi: `return` vs `yield`

- **`return`:** Kết thúc hàm và trả về giá trị. Mọi trạng thái của hàm bị xóa sạch khỏi bộ nhớ.
- **`yield`:** Trả về một giá trị cho người gọi, nhưng **đóng băng** hàm tại dòng đó. Khi bạn gọi tiếp, hàm sẽ "tỉnh dậy" và chạy tiếp từ dòng ngay sau dòng `yield` đó.

### 2. send()

- Đóng vai trò vừa lấy dữ liệu từ yield và đẩy dữ liệu từ phương thức send() vào lại.

Ví dụ:

```python
def may_tinh_tong():
    tong = 0
    print("--- Máy tính tổng đã sẵn sàng! ---")
    while True:
        # yield nhả ra 'tong' hiện tại và ĐỢI nhận số mới vào 'so_moi'
        so_moi = yield tong 
        
        if so_moi is None:
            break # Thoát nếu không gửi gì
            
        tong += so_moi
        print(f"-> Đã nhận: {so_moi}, Tổng mới: {tong}")

# 1. Khởi tạo generator
gen = may_tinh_tong()

# 2. Bước quan trọng: 'Mồi' (Prime) generator
# Bạn phải gọi next() hoặc .send(None) lần đầu để hàm chạy đến dòng yield đầu tiên
dau_tien = next(gen) 
print(f"Giá trị ban đầu: {dau_tien}")

# 3. Bắt đầu gửi dữ liệu vào
print(f"Kết quả trả về từ send: {gen.send(10)}") # Gửi 10, nhận về 10
print(f"Kết quả trả về từ send: {gen.send(20)}") # Gửi 20, nhận về 30
print(f"Kết quả trả về từ send: {gen.send(5)}")  # Gửi 5, nhận về 35
```
- Lần 1: next(gen) -> yield -> Lúc này tong = 0 và đóng băng
- Lần 2: gen.send(10) -> release -> biến ở yield: so moi = 10 -> tong = 10 -> yield -> trả ra tong = 10 -> đóng băng
- Lần 3: ...
- Lần 4: ...

### 3. yield from

`yield from <iterable>` cho phép bạn nhả ra toàn bộ giá trị từ một danh sách hoặc một generator khác mà không cần viết vòng lặp `for`.

```python
def kho_hang():
    yield "Táo"
    yield "Lê"

def dai_ly():
    yield "Kẹo"
    yield from kho_hang() # Nhả trực tiếp Táo và Lê từ kho_hang
    yield "Bánh"
```

#### 6.2.10.1. Generator-iterator methods

- generator.__next__()
- generator.send(_value_)
- generator.throw(_value_)
- generator.throw(_type_[, _value_[, _traceback_]])
- generator.close()

#### 6.2.10.2. Examples

```python
def echo(value=None):
    print("Execution starts when 'next()' is called for the first time.")
    try:
        while True:
            try:
                value = (yield value)
            except Exception as e:
                value = e
    finally:
        print("Don't forget to clean up when 'close()' is called.")

generator = echo(1)
print(next(generator))


print(next(generator))

print(generator.send(2))

print(generator.throw(TypeError, "spam"))

generator.close()
```

Thực tế:

```python
import time

def he_thong_xu_ly_don_hang():
    print("[Hệ thống] Đang khởi động máy chủ...")
    tong_doanh_thu = 0
    try:
        while True:
            # yield nhả ra tổng doanh thu và đợi nhận giá trị đơn hàng mới
            don_hang_moi = (yield tong_doanh_thu)
            
            try:
                if don_hang_moi is None:
                    continue
                
                # Giả lập xử lý đơn hàng
                if don_hang_moi < 0:
                    raise ValueError("Giá trị đơn hàng không hợp lệ!")
                
                print(f"[Hệ thống] Đã nhận đơn hàng: {don_hang_moi}$")
                tong_doanh_thu += don_hang_moi
                time.sleep(0.5) # Giả lập thời gian xử lý
                
            except ValueError as e:
                # Dùng để hứng lỗi từ nội bộ hoặc từ phương thức .throw()
                print(f"[Cảnh báo] Lỗi xử lý: {e}")
                # Sau khi xử lý lỗi, hàm vẫn tiếp tục vòng lặp
                
    except GeneratorExit:
        print("[Hệ thống] Nhận lệnh đóng hệ thống (Close).")
    finally:
        print(f"[Hệ thống] Đã dọn dẹp bộ nhớ. Doanh thu cuối cùng: {tong_doanh_thu}$")

# --- BẮT ĐẦU ĐIỀU KHIỂN ---

# 1. Khởi tạo
admin = he_thong_xu_ly_don_hang()

# 2. Mồi (Prime) hệ thống
# Chạy đến dòng yield đầu tiên
print(f"Trạng thái ban đầu: {next(admin)}$")

# 3. Sử dụng .send() để gửi đơn hàng
print(f"Tổng hiện tại: {admin.send(100)}$")
print(f"Tổng hiện tại: {admin.send(250)}$")

# 4. Sử dụng .throw() để ném một lỗi từ bên ngoài vào
# Giả sử quản lý phát hiện gian lận và ném lỗi vào vị trí luồng đang đứng
print("\n--- Phát hiện nghi vấn, ném lỗi vào hệ thống ---")
admin.throw(ValueError, "Phát hiện gian lận trong đơn hàng này!")

# 5. Gửi tiếp đơn hàng bình thường sau khi đã xử lý lỗi (Hệ thống không chết)
print(f"Tổng hiện tại sau lỗi: {admin.send(50)}$")

# 6. Sử dụng .close() để tắt hệ thống
print("\n--- Kết thúc ngày làm việc ---")
admin.close()
```

#### 6.2.10.3. Asynchronous generator functions

#### 6.2.10.4. Asynchronous generator-iterator methods

Nguồn: https://docs.python.org/3/reference/expressions.html#yieldexpr
