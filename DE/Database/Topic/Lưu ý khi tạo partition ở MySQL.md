# I. Một khóa chính và 1 unique

Để hiểu tại sao MySQL lại bắt buộc **tất cả các cột trong hàm phân vùng phải nằm trong mọi khóa chính (Primary Key) và khóa duy nhất (Unique Key)**, chúng ta sẽ đi qua một ví dụ thực tế về quản lý tài khoản người dùng.

Cơ chế này tồn tại để MySQL đảm bảo **tính duy nhất của dữ liệu trên toàn bộ ổ đĩa mà không làm sụt giảm hiệu năng**.

---

## 1. Kịch bản ví dụ (Bảng bị LỖI)

Giả sử bạn muốn tạo một bảng quản lý người dùng (`users`). Bạn muốn:

1. Định danh mỗi người bằng một số tăng tự động (`id`) làm **Khóa chính**.
    
2. Đảm bảo tên đăng nhập (`username`) là **Duy nhất** (không ai trùng ai).
    
3. Phân vùng bảng này theo **Năm đăng ký** (`year_registered`) để dễ dàng xóa dữ liệu cũ sau này.
    

Bạn thiết kế câu lệnh SQL như sau:

SQL

```
-- CÂU LỆNH NÀY SẼ BỊ LỖI (MySQL không cho phép chạy)
CREATE TABLE users (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    year_registered INT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY (username)
)
PARTITION BY RANGE (year_registered) (
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p2026 VALUES LESS THAN (2027)
);
```

### ❌ Kết quả:

MySQL sẽ chặn lại ngay lập tức và báo lỗi: `ERROR 1503 (HY000): A PRIMARY KEY must include all columns in the table's partitioning function.`

---

## 2. Giải thích: Tại sao MySQL lại cấm điều đó?

Hãy tưởng tượng nếu MySQL "nhắm mắt cho qua" và cho phép tạo bảng trên. Chuyện gì sẽ xảy ra khi bạn chèn dữ liệu?

Bây giờ, bảng đang có 2 phân vùng: `p2025` và `p2026`.

- Trong phân vùng `p2025`, đã có sẵn một người dùng tên là **"alex"** (có `id` = 1, đăng ký năm 2025).
    

Hôm nay (năm 2026), một người dùng mới vào đăng ký cũng lấy tên là **"alex"**.

1. Bạn chạy lệnh: `INSERT INTO users (username, year_registered) VALUES ('alex', 2026);`
    
2. Dựa vào hàm phân vùng `PARTITION BY RANGE (year_registered)`, MySQL biết dòng này phải cất vào phân vùng `p2026`.
    
3. Tuy nhiên, vì cột `username` có ràng buộc `UNIQUE` (duy nhất), MySQL có nghĩa vụ phải kiểm tra xem cái tên **"alex"** này đã tồn tại trong hệ thống chưa.
    

**Vấn đề xuất hiện ở đây:** Vì cột phân vùng là `year_registered` chứ không phải `username`, MySQL **không biết** cái tên "alex" cũ đang nằm ở phân vùng nào. Để đảm bảo tên này chưa bị ai lấy, MySQL sẽ phải "lục tung" tất cả các phân vùng còn lại (quét qua `p2024`, `p2025`,...).

> 💡 Nếu bảng có 100 phân vùng, MySQL phải lục cả 100 phân vùng chỉ để kiểm tra 1 lệnh `INSERT`. Điều này làm tiêu tan hoàn toàn lợi ích tăng tốc của việc phân vùng!

Để tránh thảm họa hiệu năng này, MySQL đặt ra quy định: **Muốn phân vùng theo cột nào, thì cột đó phải nằm trong tất cả các khóa Unique/Primary.** Nhờ vậy, khi kiểm tra trùng lặp, MySQL chỉ cần nhảy bổ vào đúng 1 phân vùng duy nhất để kiểm tra.

---

## 3. Cách giải quyết (Sửa lại cho đúng)

Để sửa lỗi trên, bạn có 2 cách tùy thuộc vào logic nghiệp vụ của bạn:

### Cách 1: Gộp cột phân vùng vào Khóa chính và Khóa duy nhất (Khuyên dùng)

Bạn chấp nhận thiết lập khóa chính và khóa duy nhất dưới dạng "Khóa phức hợp" (Composite Key), chứa cả năm đăng ký.

SQL

```
-- CÂU LỆNH CHẠY THÀNH CÔNG
CREATE TABLE users (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    year_registered INT NOT NULL,
    PRIMARY KEY (id, year_registered),      -- Đã thêm year_registered vào đây
    UNIQUE KEY (username, year_registered)  -- Đã thêm year_registered vào đây
)
PARTITION BY RANGE (year_registered) (
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p2026 VALUES LESS THAN (2027)
);
```

_Lưu ý của cách này:_ Ràng buộc unique bây giờ là _cặp_ `(username, year_registered)`. Nghĩa là năm 2025 có một người tên "alex" thì năm 2026 vẫn có thể có một người trùng tên "alex" nữa.

### Cách 2: Phân vùng dựa trên chính Khóa chính (Nếu bắt buộc username không được trùng)

Nếu bạn tuyệt đối không cho phép trùng `username` trên toàn hệ thống qua các năm, bạn không thể dùng `year_registered` làm hàm phân vùng đơn lẻ được. Bạn phải chuyển sang phân vùng theo `id` hoặc `HASH(username)`.

SQL

```
-- CÂU LỆNH CHẠY THÀNH CÔNG
CREATE TABLE users (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    year_registered INT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY (username)
)
-- Phân vùng bằng thuật toán KEY dựa trên Khóa chính id
PARTITION BY KEY(id) 
PARTITIONS 4; 
```

### 💡 Tóm lại

MySQL bắt buộc quy tắc này để đảm bảo khi bạn làm bất kỳ thao tác gì (thêm, sửa, kiểm tra trùng lặp), nó **chỉ cần mở đúng 1 file phân vùng duy nhất ra xử lý**, giúp hệ thống luôn đạt tốc độ tối đa.

# II. Khóa chính

Nếu bảng của bạn chỉ có **một Khóa chính (Primary Key) duy nhất** thông thường và không có bất kỳ khóa `UNIQUE` nào khác, thì quy tắc của MySQL vẫn giữ nguyên:

> **Cột dùng để phân vùng bắt buộc phải nằm bên trong Khóa chính đó.**

Hãy cùng xem ví dụ cụ thể để thấy điều gì xảy ra nếu bạn cố tình vi phạm và cách giải quyết nó ra sao.

---

## 1. Kịch bản lỗi khi chỉ có Primary Key

Giả sử bạn có một bảng quản lý đơn hàng (`orders`). Bạn chỉ có một khóa chính duy nhất là `order_id` (mã đơn hàng tăng tự động). Bạn muốn phân vùng bảng này theo cột `order_date` (ngày tạo đơn hàng) để sau này dễ dàng xóa các đơn hàng từ nhiều năm trước.

Bạn viết câu lệnh như sau:

SQL

```
-- CÂU LỆNH NÀY SẼ BỊ LỖI
CREATE TABLE orders (
    order_id INT NOT NULL AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    amount DECIMAL(10,2),
    PRIMARY KEY (order_id) -- Chỉ có 1 khóa chính thông thường
)
PARTITION BY RANGE COLUMNS(order_date) (
    PARTITION p2025 VALUES LESS THAN ('2026-01-01'),
    PARTITION p2026 VALUES LESS THAN ('2027-01-01')
);
```

### ❌ Kết quả:

MySQL sẽ lập tức từ chối và báo lỗi tương tự: `ERROR 1503 (HY000): A PRIMARY KEY must include all columns in the table's partitioning function.`

### Tại sao lại lỗi?

Bởi vì Khóa chính (`PRIMARY KEY`) thực chất cũng là một loại Khóa duy nhất (`UNIQUE KEY`). Khi bạn chèn một đơn hàng mới có `order_id = 99` vào năm 2026 (phân vùng `p2026`), MySQL có nhiệm vụ phải đảm bảo rằng `order_id = 99` này **chưa từng tồn tại** ở bất kỳ phân vùng nào khác (như `p2025`).

Nếu cột phân vùng là `order_date` (chỉ dựa vào ngày tháng), MySQL sẽ không thể biết các `order_id` khác đang nằm ở đâu để mà kiểm tra trùng lặp, trừ khi nó phải đi lục tung từng phân vùng một. Để ngăn chặn việc giảm hiệu năng này, MySQL bắt bạn phải đưa `order_date` vào trong khóa chính.

---

## 2. Cách giải quyết chuẩn xác

Để sửa lỗi này, bạn có 2 hướng giải quyết cực kỳ đơn giản:

### Cách 1: Tạo Khóa chính phức hợp (Composite Primary Key)

Bạn gộp cả cột phân vùng vào làm một phần của Khóa chính. Đây là cách làm phổ biến nhất trong thực tế.

SQL

```
-- CÂU LỆNH CHẠY THÀNH CÔNG
CREATE TABLE orders (
    order_id INT NOT NULL AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    amount DECIMAL(10,2),
    PRIMARY KEY (order_id, order_date) -- Đã thêm cột phân vùng vào Khóa chính
)
PARTITION BY RANGE COLUMNS(order_date) (
    PARTITION p2025 VALUES LESS THAN ('2026-01-01'),
    PARTITION p2026 VALUES LESS THAN ('2027-01-01')
);
```

_Lưu ý:_ Cơ chế `AUTO_INCREMENT` của cột `order_id` vẫn hoạt động hoàn toàn bình thường khi nằm trong khóa chính phức hợp (đối với Engine InnoDB).

### Cách 2: Phân vùng dựa trên chính Khóa chính (`order_id`)

Nếu bạn không muốn thay đổi cấu trúc khóa chính, bạn bắt buộc phải thay đổi chiến lược phân vùng. Thay vì phân vùng theo ngày tháng, bạn hãy phân vùng theo khoảng ID hoặc băm (HASH) theo ID.

SQL

```
-- CÂU LỆNH CHẠY THÀNH CÔNG (Phân vùng theo khoảng ID)
CREATE TABLE orders (
    order_id INT NOT NULL AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    amount DECIMAL(10,2),
    PRIMARY KEY (order_id)
)
PARTITION BY RANGE (order_id) (
    PARTITION p_trien_vong_1 VALUES LESS THAN (1000000),  -- 1 triệu đơn hàng đầu
    PARTITION p_trien_vong_2 VALUES LESS THAN (2000000)   -- các đơn hàng tiếp theo
);
```

---

## 💡 Trường hợp duy nhất KHÔNG CẦN đưa cột vào Khóa chính là gì?

Đó là khi bảng của bạn **hoàn toàn KHÔNG CÓ khóa chính và cũng KHÔNG CÓ khóa unique nào cả**.

SQL

```
-- CÂU LỆNH VẪN CHẠY THÀNH CÔNG (Vì bảng không có khóa nào để ràng buộc)
CREATE TABLE log_data (
    log_type VARCHAR(20),
    log_message TEXT,
    log_date DATE NOT NULL
)
PARTITION BY RANGE COLUMNS(log_date) (
    PARTITION p_old VALUES LESS THAN ('2026-01-01'),
    PARTITION p_new VALUES LESS THAN (MAXVALUE)
);
```

_(Tuy nhiên trong thực tế, thiết kế một bảng dữ liệu lớn mà không có khóa chính là điều rất hiếm và không được khuyến khích vì nó gây khó khăn cho việc quản lý và đồng bộ dữ liệu)._

# III. Composite Key

Thực ra, hiểu **Composite Key (Khóa phức hợp)** là một secondary index thì **vừa đúng mà vừa chưa đúng**, tùy thuộc vào việc bạn gán nó làm nhiệm vụ gì.

Để hiểu rõ bản chất, chúng ta cần phân biệt cách InnoDB (Storage Engine mặc định của MySQL) biểu diễn dữ liệu trên cấu trúc cây **B+Tree** (biến thể phổ biến nhất của B-Tree dùng trong database).

---

## 1. Đính chính: Composite Key là Primary Index hay Secondary Index?

Trong cấu trúc lưu trữ của InnoDB, bảng dữ liệu chính là một cấu trúc cây. Một Khóa phức hợp (Composite Key - khóa gồm nhiều cột ghép lại) có thể đóng hai vai trò:

### Trường hợp 1: Nó là Clustered Index (Primary Index)

Nếu lúc tạo bảng, bạn chỉ định: `PRIMARY KEY (order_id, order_date)`.

- Lúc này, đây là **Khóa chính phức hợp**.
    
- Trên cây B+Tree của Khóa chính, các nút lá (Leaf Nodes) sẽ chứa **toàn bộ dữ liệu của cả dòng đó** (tất cả các cột khác như `customer_id`, `amount`,...).
    

### Trường hợp 2: Nó là Secondary Index (Chỉ mục phụ)

Nếu bạn đã có một khóa chính khác, và bạn tạo thêm: `UNIQUE KEY (username, year_registered)` hoặc `INDEX (lastname, firstname)`.

- Lúc này, nó chính là một **Secondary Index phức hợp**.
    
- Trên cây B+Tree của chỉ mục phụ này, các nút lá **KHÔNG** chứa toàn bộ dữ liệu của dòng, mà chúng chỉ chứa: **Giá trị của các cột trong khóa đó** + **Con trỏ (Pointer) trỏ về Khóa chính** để khi cần thì quay lại cây chính tìm dữ liệu (gọi là hành động _Lookup/Bookmark Lookup_).
    

---

## 2. Cách Composite Key biểu diễn trên cây B+Tree

Dù là Primary hay Secondary, quy tắc sắp xếp các cột trong Composite Key trên cây B+Tree tuân theo nguyên tắc: **Từ trái qua phải (Left-to-Right)**. Đây gọi là **Thứ tự từ điển (Lexicographical Order)**.

Giả sử bạn có một Composite Key gồm 2 cột: `(Cột_A, Cột_B)`. Hệ thống sẽ sắp xếp cây B+Tree theo quy tắc:

1. So sánh và sắp xếp theo `Cột_A` trước.
    
2. Nếu `Cột_A` của hai dòng bằng nhau, hệ thống mới xét tiếp đến `Cột_B` để sắp xếp.
    

### 🌲 Minh họa trực quan các nút trên cây:

Hãy nhìn vào cách các phần tử `(Cột_A, Cột_B)` được phân bổ và sắp xếp từ trái qua phải trên các nút của cây:

Plaintext

```
                      [ (2, "B") | (5, "A") ]  <-- Nút cha (Root/Internal Node)
                             /         \
                            /           \
  [ (1, "A") | (2, "A") | (2, "B") ]     [ (5, "A") | (5, "C") | (6, "A") ]  <-- Nút lá (Leaf Nodes)
```

**Nhận xét từ sơ đồ trên:**

- Các giá trị được xếp tăng dần theo số (Cột A): `1 -> 2 -> 5 -> 6`.
    
- Tại vị trí số `2` bị trùng, hệ thống sẽ xếp theo chữ cái (Cột B): `(2, "A")` đứng trước `(2, "B")`.
    
- Tại vị trí số `5` bị trùng, chữ `"A"` đứng trước chữ `"C"`: `(5, "A")` đứng trước `(5, "C")`.
    

---

## 3. Hệ quả cực kỳ quan trọng: Quy tắc tiền tố bên trái nhất (Leftmost Prefix Rule)

Chính vì cách biểu diễn xếp hàng "anh đứng trước, em đứng sau" này trên B+Tree, việc bạn viết câu lệnh truy vấn `WHERE` sẽ quyết định MySQL có dùng được index hay không.

Dựa vào cây chỉ mục `(Cột_A, Cột_B)` ở trên:

- **Trường hợp 1: Bạn tìm theo `WHERE Cột_A = 2`** 👉 **DÙNG ĐƯỢC INDEX.** Vì cây được xếp theo Cột_A trước, MySQL chỉ cần đi từ gốc xuống và tìm thấy ngay vùng chứa số 2.
    
- **Trường hợp 2: Bạn tìm theo `WHERE Cột_A = 2 AND Cột_B = "B"`** 👉 **DÙNG ĐƯỢC INDEX HOÀN HẢO.** MySQL tìm đến số 2, sau đó thu hẹp vùng tìm kiếm dựa trên chữ "B".
    
- **Trường hợp 3: Bạn tìm theo `WHERE Cột_B = "B"` (Bỏ qua Cột_A)** 👉 **KHÔNG DÙNG ĐƯỢC INDEX (Phải quét toàn bộ bảng - Full Table Scan).** _Tại sao?_ Vì chữ "B" nằm rải rác ở khắp các nhánh cây (nó đứng sau số 2, đứng sau số 5,...). Trên cấu trúc cây, các chữ "B" không đứng cạnh nhau, nên MySQL không thể thực hiện thuật toán tìm kiếm nhị phân chuyển hướng trên cây được.
    

### 💡 Tóm lại

Composite Key biểu diễn trên B+Tree bằng cách **gộp các giá trị của các cột lại thành một chuỗi định danh duy nhất** và sắp xếp chúng tuần tự từ cột đầu tiên đến cột cuối cùng. Khi thiết kế Composite Key, cột nào tuỳ chọn tìm kiếm nhiều nhất/độ lọc cao nhất bắt buộc phải được đặt ở **bên trái cùng**.