## Hive Architecture

![alt][images/Tools/HMS/Hive_arc.png]

## Hive Metastore Architecture

![alt][images/Tools/HMS/HMS_arc2.png]

Các thành phần chính của HMS:
- UI: Thông qua giao diện người dùng nó là nơi người dùng submit truy vấn và làm các tác vụ hệ thống khác. Vào năm 2011, Hệ thống đã cung cấp giao diện command-line và UI đã đang phát triển.
- Driver: Là thành phần cho phép 

## Hive Metastore Modes

![alt][images/Tools/HMS/HMS_arc.png]

Có 3 modes deployment:
- Embedded Metastore
- Local Metastore
- Remote Metastore

**i. Embedded Metastore**

Theo mặc định, trong Hive, dịch vụ metastore chạy trong cùng một JVM với dịch vụ Hive. Ở chế độ này, nó sử dụng cơ sở dữ liệu Derby nhúng được lưu trữ trên hệ thống tệp cục bộ. Do đó, cả dịch vụ metastore và dịch vụ Hive đều chạy trong cùng một JVM bằng cách sử dụng cơ sở dữ liệu Derby nhúng.

