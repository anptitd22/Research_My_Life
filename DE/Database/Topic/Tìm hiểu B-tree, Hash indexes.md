# B-tree

B-tree có thể được sử dụng để so sánh cột trong các biểu thức sử dụng các toán tử =, >, >=, <, <= hoặc BETWEEN. Index cũng có thể được sử dụng cho các phép so sánh LIKE nếu đối số của LIKE là một chuỗi hằng số không bắt đầu bằng ký tự đại diện. Ví dụ, các câu lệnh SELECT sau sử dụng chỉ mục:

```sql
SELECT * FROM tbl_name WHERE key_col LIKE 'Patrick%';
SELECT * FROM tbl_name WHERE key_col LIKE 'Pat%_ck%';
```

- Trong câu lệnh đầu tiên, chỉ những hàng có 'Patrick' <= key_col < 'Patricl' được xem xét. Trong câu lệnh thứ hai, chỉ những hàng có 'Pat' <= key_col < 'Pau' được xem xét.

Các câu lệnh SELECT sau đây không sử dụng chỉ mục:

```sql
SELECT * FROM tbl_name WHERE key_col LIKE '%Patrick%';
SELECT * FROM tbl_name WHERE key_col LIKE other_col;
```

- Nếu bạn sử dụng ... LIKE '%string%' và chuỗi đó dài hơn ba ký tự, MySQL sẽ sử dụng thuật toán Turbo Boyer-Moore để khởi tạo mẫu cho chuỗi và sau đó sử dụng pattern này để thực hiện tìm kiếm nhanh hơn.

- Việc tìm kiếm bằng cách sử dụng col_name IS NULL sẽ sử dụng chỉ mục nếu col_name đã được lập chỉ mục.

Bất kỳ chỉ mục nào không bao gồm tất cả các cấp độ AND trong mệnh đề WHERE đều không được sử dụng để tối ưu hóa truy vấn. Nói cách khác, để có thể sử dụng chỉ mục, tiền tố của chỉ mục phải được sử dụng trong mọi nhóm AND.

- Các mệnh đề WHERE sau đây sử dụng chỉ mục:

```sql
... WHERE index_part1=1 AND index_part2=2 AND other_column=3

    /* index = 1 OR index = 2 */
... WHERE index=1 OR A=10 AND index=2

    /* optimized like "index_part1='hello'" */
... WHERE index_part1='hello' AND index_part3=5

    /* Can use index on index1 but not on index2 or index3 */
... WHERE index1=1 AND index2=2 OR index1=3 AND index3=3;
```

- Các mệnh đề WHERE này không sử dụng chỉ mục:

```sql
    /* index_part1 is not used */
... WHERE index_part2=1 AND index_part3=2

    /*  Index is not used in both parts of the WHERE clause  */
... WHERE index=1 OR A=10

    /* No index spans all rows  */
... WHERE index_part1=1 OR index_part2=10
```

Đôi khi MySQL không sử dụng chỉ mục, ngay cả khi chỉ mục đó có sẵn. Một trường hợp xảy ra điều này là khi trình tối ưu hóa ước tính rằng việc sử dụng chỉ mục sẽ yêu cầu MySQL truy cập một tỷ lệ rất lớn các hàng trong bảng. (Trong trường hợp này, việc quét toàn bộ bảng có thể nhanh hơn nhiều vì nó yêu cầu ít thao tác tìm kiếm hơn.) Tuy nhiên, nếu truy vấn như vậy sử dụng LIMIT để chỉ truy xuất một số hàng nhất định, MySQL vẫn sử dụng chỉ mục, bởi vì nó có thể tìm thấy một vài hàng cần trả về trong kết quả nhanh hơn nhiều.

>[!note]
>Bản chất của B-tree là sort các cột được đánh index rồi tìm kiếm

>[!warning]
>B-tree không có tên cụ thể nhưng có thể gọi là balancing tree nhưng chắc chắn không phải binary tree

So sánh với binary tree: https://www.geeksforgeeks.org/dsa/difference-between-binary-tree-and-b-tree/

Tìm hiểu thêm: https://www.geeksforgeeks.org/dsa/introduction-of-b-tree-2/

Các bước tìm kiếm:
![alt][images/database/B_tree_step0.png]

- Step1:
![alt][images/database/B_tree_step1.png]
- Step2
![alt][images/database/B_tree_step2.png]
- Step3
![alt][images/database/B_tree_step3.png]
- Step4
![alt][images/database/B_tree_step4.png]
- Code
```python
class Node:
    def __init__(self):
        self.n = 0
        self.key = [0] * MAX_KEYS
        self.child = [None] * MAX_CHILDREN
        self.leaf = True

def BtreeSearch(x, k):
    i = 0
    while i < x.n and k > x.key[i]:
        i += 1
    if i < x.n and k == x.key[i]:
        return x
    if x.leaf:
        return None
    return BtreeSearch(x.child[i], k)
```
- Time Complexity
![alt][images/database/B_tree_Time_Complexity.png]

# Hash Index

Hash index có một số đặc điểm khác biệt so với những chỉ mục vừa được thảo luận:

- Chúng chỉ được sử dụng cho các phép so sánh bằng nhau sử dụng toán tử = hoặc <=> (nhưng rất nhanh). Chúng không được sử dụng cho các toán tử so sánh như < dùng để tìm một phạm vi giá trị. Các hệ thống dựa vào kiểu tra cứu giá trị đơn được gọi là "key-value"; để sử dụng MySQL cho các ứng dụng như vậy, hãy sử dụng hash index bất cứ khi nào có thể.

- Dữ liệu index được tổ chức theo dạng Key - Value được liên kết với nhau.

- Không thể tối ưu hóa toán tử ORDER BY bằng việc sử dụng HASH index bởi vì nó không thể tìm kiếm được phần từ tiếp theo trong Order.
 
 - MySQL không thể xác định gần đúng số lượng hàng nằm giữa hai giá trị (thông tin này được trình tối ưu hóa phạm vi sử dụng để quyết định sử dụng chỉ mục nào). Điều này có thể ảnh hưởng đến một số truy vấn nếu bạn chuyển đổi bảng MyISAM hoặc InnoDB thành bảng MEMORY được lập hash index.

- Chỉ có thể sử dụng toàn bộ khóa để tìm kiếm một hàng. (Với chỉ B-tree, bất kỳ tiền tố nào nằm bên trái nhất của khóa đều có thể được sử dụng để tìm hàng.)

- Độ phức tạp: O(1)

>[!note]
>Việc chọn index theo kiểu BTREE hay HASH ngoài yếu tố về mục đích sử dụng index thì nó còn phụ thuộc vào mức độ hỗ trợ của Storage Engine. Ví dụ MyISAM, InnoDB hay Archive chỉ hỗ trợ Btree, trong khi Memory lại hỗ trợ cho cả 2.