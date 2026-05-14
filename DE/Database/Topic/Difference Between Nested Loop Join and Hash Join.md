## What is a Nested Loop?

Đây là một loại thuật toán kết hợp vật lý được sử dụng trong trường hợp kết hợp hai quan hệ. Phép kết hợp này là một kỹ thuật kết hợp nội bộ, có nghĩa là chúng ta không thể nhìn thấy phép kết hợp. Đây là loại kết hợp đơn giản nhất trong tất cả các loại kết hợp. Đây là thuật toán phù hợp nhất cho dữ liệu nhỏ và các giao dịch nhỏ. Trong trường hợp hai quan hệ có tên là R và S, thuật toán cho phép kết hợp vòng lặp lồng nhau sẽ như sau:

```
For each record x of R read in, do  
Use the index on B for S  
Get all the matching records (having B=x.A)  
End
```

## ***What is Hash Join?***

Hash Join cũng là một loại thuật toán kết hợp vật lý được sử dụng trong trường hợp kết hợp hai bảng nội bộ. Vì đây là kỹ thuật kết hợp nội bộ nên chúng ta không thể nhìn thấy kết quả kết hợp. Việc lựa chọn kết hợp được thực hiện tự động bởi trình tối ưu hóa truy vấn. Hash Join được thực hiện bằng hai bước: xây dựng và dò tìm. Trong trường hợp có 2 bảng có tên R và S, thuật toán Hash Join sẽ như sau:

```
Hash records of R, one by one, using A values  
(Use the same M buckets and same hash function h)  
Hash matching pair of records into the same bucket  
End
```

## ***Difference Between Nested Loop Join and Hash Join***


|Nested Loop Join|Hash Join|
|---|---|
|It is processed by forming an outer loop within an inner loop after which the inner loop is individually processed for the fewer entries that it has.|It is specifically used in case of joining of larger tables.|
|The nested join has the least performance in case of large tables.|It has best performance in case of large and sorted and non-indexed inputs.|
|There are two phases in this, outer and inner loop processing.|The two phases in this are build and probe.|
|Steps involved include identifying an outer driving table and assigning the inner table to it, and processing the rows of inner table for every outer table row.|The steps involved are building a Hash table on a small table. It is used to probe the hash value of the Hash table is applicable for each element in the second row.|
|Index range scan is done here.|Full-table scan of the smaller table is done in case of hash join.|
|This uses lesser [RAM](https://www.geeksforgeeks.org/computer-science-fundamentals/random-access-memory-ram/) resources.|It uses more RAM resources.|
|It is the most common type of join.|It is not as common as the nested loop join.|
|Least number of comparisons are required in case of nested loop join.|It needs more comparisons than the nested loop join thereby using more RAM resources.|
|It is the fastest join algorithm due to least number of comparisons.|It is not as fast due to more number of comparisons.|
|It is better than all other types of join for small transactions and small data.|It is not as good as nested loop join in case of smaller data.|
|It is of three types, namely, nested loop join, indexed nested loop join and Temporary indexed nested loop join.|Its types are classic hash join, Grace hash join, hybrid hash join, hash anti join, hash semi-join, recursive hash join and hash bailout.|
|It is not automatically selected.|This join is automatically selected in case there is no specific reason to adopt other types of join algorithms. It is also known as the go-to guy of all the join operators.|

Nguồn: https://www.geeksforgeeks.org/dbms/difference-between-nested-loop-join-and-hash-join/


---
# **What are merge join, hash join, and nested loop?**

Trong bài viết này, chúng ta hãy cùng xem xét ba loại phép nối vật lý chính mà PostgreSQL sử dụng khi thực hiện các phép nối logic bên ngoài và bên trong:

- merge join,
- hash join,
- and nested loop.

Tại sao tôi lại bắt đầu bằng việc đề cập đến sự khác biệt giữa phép nối vật lý và phép nối logic? Đó là vì chúng ở các cấp độ khác nhau. Khi bạn viết các truy vấn SQL sử dụng LEFT OUTER JOIN hoặc RIGHT JOIN, đó là cấp độ logic của phép nối. Nó thường được sử dụng cho các tích Descartes, trong đó các hàng thỏa mãn điều kiện nối được giữ lại. Vậy các truy vấn như vậy được thực thi "bên trong" công cụ RDBMS ở cấp độ vật lý như thế nào? Điều đó phụ thuộc vào nhiều yếu tố.

- kích thước của các bảng liên quan đến phép toán kết hợp.
-  _the amount of memory allocated to “work_mem” for sort operations and “hash_mem_multiplier” for hash operations,_
- sự hiện diện của các _indexes_
- các phép toán so sánh hoặc bằng nhau được áp dụng trong quá trình JOIN

Tùy thuộc vào các điều kiện này, trình lập kế hoạch của PostgreSQL có thể chọn một phương pháp (thuật toán) này hoặc phương pháp khác để thực hiện các phép nối logic.

## Description of each physical JOIN type.

Kết hợp bảng (Merge Join) là phương pháp được sử dụng khi cả hai bảng đều khá lớn và dữ liệu trên các trường (khóa JOIN) đã được sắp xếp thông qua các chỉ mục. Nếu dữ liệu trong bảng chưa được sắp xếp trước, PostgreSQL sẽ sắp xếp chúng trước khi thực hiện thao tác JOIN, điều này có thể làm tăng chi phí. Sự khác biệt chính so với kết hợp băm (Hash Join) là kết hợp bảng yêu cầu dữ liệu được sử dụng trong các khóa JOIN phải được sắp xếp, trong khi kết hợp băm không yêu cầu sắp xếp. Nó chỉ đơn giản là xây dựng một bảng băm và sử dụng bảng đó để tìm các kết quả trùng khớp. Cũng cần lưu ý rằng phép nối hợp nhất (merge join) đặc biệt hữu ích cho các phép nối phạm vi (ví dụ: >=, <=), vì sau khi sắp xếp, nó có thể xử lý hiệu quả các điều kiện như vậy.

Hash join là một phương pháp sử dụng hàm băm, được dùng khi dữ liệu cần thiết cho thao tác JOIN không được sắp xếp. Thuật toán hoạt động bằng cách chọn một trong hai bảng (thường là bảng nhỏ hơn về dung lượng bộ nhớ) và tạo một bản ghi trong bảng băm cho mỗi hàng của nó. Sau đó, bảng còn lại (bảng lớn hơn) được quét, và mỗi hàng được so sánh với bảng băm. Nếu tìm thấy sự trùng khớp với các giá trị cần thiết, các hàng sẽ được kết hợp. Điều đáng nhấn mạnh là PostgreSQL sử dụng bảng nhỏ hơn để tạo bảng băm nhằm giảm thiểu việc sử dụng bộ nhớ. Bảng băm này sau đó được sử dụng để nhanh chóng tìm các phần tử trùng khớp trong bảng lớn thứ hai. Tóm lại, phép nối băm (hash join) thường được sử dụng cho các bảng lớn, khi không có chỉ mục hoặc khi dữ liệu không được sắp xếp, và điều kiện JOIN liên quan đến các phần tử trùng khớp chính xác. Phép nối phạm vi (range join) thường kém hiệu quả hơn với thuật toán nối băm.

Vòng lặp lồng nhau (Nested Loop) là một phương pháp sử dụng các vòng lặp lồng nhau, trong đó, với mỗi giá trị trong một bảng, nó sẽ tìm kiếm giá trị tương ứng trong bảng khác. Cụ thể hơn, nó lấy giá trị đầu tiên từ bảng thứ nhất và so sánh tuần tự giá trị đó với tất cả các giá trị trong bảng thứ hai. Nếu tìm thấy sự trùng khớp, bản ghi đó sẽ được đưa vào tập dữ liệu cuối cùng. Sau khi giá trị từ bảng đầu tiên được so sánh với tất cả các giá trị trong bảng thứ hai, giá trị tiếp theo từ bảng nhỏ hơn đầu tiên được lấy ra, và quá trình so sánh lại tiếp tục với tất cả các giá trị trong bảng thứ hai. Quá trình này tiếp tục cho đến khi mọi giá trị từ bảng đầu tiên được so sánh với mọi giá trị từ bảng thứ hai, đó là lý do tại sao phương pháp này được gọi là "vòng lặp lồng nhau" (vì toàn bộ quá trình tiếp tục cho đến khi tất cả các giá trị được so sánh theo cách lồng nhau).

Khi đọc các bài viết khác trên mạng, tôi thấy thông tin cho rằng vòng lặp lồng nhau “luôn luôn” được sử dụng khi một bảng nhỏ và bảng kia lớn. Điều này KHÔNG nhất thiết đúng, và tôi sẽ cung cấp mã SQL kèm ví dụ và giải thích về kế hoạch truy vấn bằng lệnh “EXPLAIN ANALYZE”.

....

Nguồn: https://gelovolro.medium.com/what-are-merge-join-hash-join-and-nested-loop-example-in-postgresql-29123ca18fd1