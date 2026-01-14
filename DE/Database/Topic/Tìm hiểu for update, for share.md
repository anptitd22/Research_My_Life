For update và for share: đều là khóa chủ động hay pessimistic
- For update: khóa row không cho transaction khác đọc, update cho đến khi transaction đó commit hoặc roll_back
- For share: khóa row cho các transaction khác đọc nhưng không thể update cho đến khi transaction đó commit hoặc roll_back
Nguồn:
https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html
Tình huống thực tế:
- Trong một hệ thống ngân hàng:
	- Một người cùng một lúc click thực hiện 2 transaction trừ tiền
	- Tài khoản có 1.000.000 và thực hiện giao dịch trừ 700.000, điều kiện thực hiện giao dịch thành công là tài khoản > 700.000
	- Đối với for share: khi select 2 transaction vẫn đồng thời hiện 1.000.000 và trừ tiền dẫn đến -400.000
	- Đối với for update: khi select 2 transaction thì transaction nào thực hiện trước sẽ lock và thực hiện trừ tiền, transaction sau sẽ bị load cho đến khi transaction trước commit và tài khoản sẽ còn 300.000 