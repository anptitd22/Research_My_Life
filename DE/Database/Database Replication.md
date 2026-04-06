Sao chép cơ sở dữ liệu là quá trình sao chép dữ liệu từ một cơ sở dữ liệu sang một hoặc nhiều máy chủ để đảm bảo availability and reliability

- Cải thiện khả năng truy cập dữ liệu bằng cách lưu giữ nhiều bản sao cơ sở dữ liệu.
- Tăng khả năng chịu lỗi nếu cơ sở dữ liệu chính gặp sự cố.
- Hỗ trợ khôi phục và sao lưu dữ liệu sau sự cố.
- Hỗ trợ cân bằng tải giữa các máy chủ.
![alt][images/Database/Database_Replication/master_slave.png]

## Types of Database Replication

Types of Database Replication include Single-leader (Master-Slave), Multi-leader (Master-Master), and Peer-to-Peer replication.

## 1. Master-Slave Replication

Trong cơ chế master-slave, tất cả các thao tác ghi được xử lí bởi master database, và dữ liệu được cập nhật đã được đồng bộ hóa với các slave databases.

- Master xử lý tất cả các thao insert, upload  và delete.
- Slaves duy trì các bản sao dữ liệu được nhân bản từ master.
- Cải thiện đọc dữ liệu bằng cách sử dụng slave để query
- Cung cấp khả năng chịu lỗi cơ bản.
- Thường được sử dụng trong các ứng dụng đọc dữ liệu nhiều và nặng.

Ví dụ: Hãy tưởng tượng một thư viện có hai chi nhánh
- ****Master branch**** : Chi nhánh chính chứa  thư viện chính lưu giữ bộ sưu tập sách gốc và sách được cập nhật.
- ****Slave branch:**** : Đây là một chi nhánh nhỏ hơn, nhận sách mới từ chi nhánh chính theo định kỳ. Sinh viên chỉ có thể mượn những cuốn sách hiện có tại chi nhánh phụ.
![alt][images/Database/Database_Replication/master_slave_worker.png]

### Working

Mô hình sao chép master-slave hoạt động bằng cách sao chép các thay đổi dữ liệu từ cơ sở dữ liệu chính (master) sang một hoặc nhiều cơ sở dữ liệu phụ (slave) theo trình tự.

- ****Write Operation:**** Máy chủ chính ghi lại các thay đổi trong transaction log.
- ****Log Reading:**** Luồng sao chép đọc nhật ký giao dịch.
- ****Data Transfer:**** Các thay đổi được gửi từ master đến slave thông qua mạng.
- ****Sync Update:**** Slave áp dụng các thay đổi nhận được vào dữ liệu của nó.
- ****Confirmation:**** Slave gửi lời xác nhận (acknowledment) lại cho master.

### Applications

Kiến trúc này được sử dụng rộng rãi trong các hệ thống yêu cầu hiệu năng cao và read scalability.



