MySQL thường sử dụng index có số lượng bản ghi nhỏ nhất (index có tính chọn lọc cao nhất).
Ví dụ:
- Có index(age) và index(height)
```sql
  Select * from table where age > 50 and height > 160 
```
- Nếu index(age) > 50 có 500k bản ghi và index(height) > 160 có 300k bản ghi
- Sẽ chọn index(height)

Thông thường khi một hệ thống lớn thường dễ đến việc sử dụng sai index -> Mình phải tự chọn index bằng câu lệnh `FORCE INDEX()`
```sql
Select * from table force index(age) where age > 50 and height > 160 
```
