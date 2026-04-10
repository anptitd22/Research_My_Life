# Database Replication in System Design

Database replication là một quá trình tạo và duy trì nhiều bản sao của một database sang các servers khác nhau. Nó giúp cải thiện system availability, reliability, và performance  bằng cách đảm bảo dữ liệu vẫn có thể truy cập được ngay cả khi một máy chủ bị lỗi.

- Nếu một database server bị lỗi, một replica khác sẽ tiếp tục serving requests.
- Thao tác read có thể được phân tán sang các replicas để giảm load trên 1 server
- Các bản sao của data đảo bảo rằng hệ thống duy trì hoạt động kể cả lỗi

Ví dụ: một website e-com có thể sử dụng một primary database để thực hiện thao tác write và các replica databases để handle read requests của người dùng

## Working

Đây là bước giải thích tại sao database replica hoạt động:

![alt][images/Database/Database_Replication/replica_1.png]

1. Xác định Primary database (Source): A primary (hoặc master) được chọn làm main source thông tin đáng tin cậy nhất, nơi phát sinh các thay đổi dữ liệu.
2. Thiết lập Replica Databases (Target): 1 hoặc nhiều replicas (hoặc secondary databases) được config để nhận data từ primary database
3. Data Changes Captured: Các updates, inserts, deletes ở primary database được ghi lại, thường thông qua một transaction log hoặc cơ chế thay đổi thu thập dữ liệu
4. Apply Changes on Replicas: The Replicas áp dụng các updates này để giữ đồng bộ (sync) với primary database
5. Monitor và Maintain Synchronization: Hệ thống đảm bảo replicas luôn cập nhật thông tin (stay up-to-date) và handles issues như delays hoặc conflics trong lúc đồng bộ (sync)
6. Thao tác read hoặc write: Các chương trình có thể read data từ replicas (để giảm tải trên primary) và write vào primary, phụ thuộc vào replication model (e.h, Master-Slave, Master-Master)

## Types

### 1. Master-Slave Replication

Trong mô hình sao chép này, một database đóng vai trò là  primary server trong khi các database khác duy trì các bản sao dữ liệu của nó.

- The master database xử lý tất cả các thao tác ghi như thêm, cập nhật và xóa.
- Slave databases sao chép dữ liệu từ master chính và thường xử lý các thao tác đọc.

### 2. Master-Master Replication / Multi-Master Replication

Trong cấu hình này, nhiều databases đóng vai trò là masters và có thể chấp nhận cả thao tác đọc và ghi.

- Các thay đổi được thực hiện trong một master database sẽ được sao chép sang master databases khác.
- Cải thiện availability và cho phép thực hiện các thao tác ghi phân tán trên nhiều máy chủ.
### 3. Snapshot Replication

Phương pháp này sao chép toàn bộ database bằng cách snapshot tại một thời điểm cụ thể.

- Một bản sao hoàn chỉnh của database được tạo ra và chuyển đến các máy chủ khác.
- Thích hợp cho các hệ thống có tần suất thay đổi dữ liệu không cao.

### 4. Transactional Replication

Transactional replication synchronizes liệu bằng cách sao chép các thay đổi ngay khi chúng xảy ra.

- Các thay đổi được thực hiện trong publisher database sẽ nhanh chóng được gửi đến subscriber databases.
- Đảm bảo tính nhất quán (consistency) dữ liệu gần như thời gian thực giữa nhiều databases.
### 5. Merge Replication

Merge replication cho phép nhiều cơ sở dữ liệu cập nhật dữ liệu một cách độc lập và sau đó đồng bộ hóa các thay đổi.

- Các thay đổi được thực hiện trong publisher database sẽ nhanh chóng được gửi đến subscriber databases.
- Conflicts được phát hiện và giải quyết trong quá trình đồng bộ hóa (synchronization).

## Strategies

## Configurations

Để đạt được các mục tiêu cụ thể liên quan đến consistency, availability và performance, database replication có thể được thiết lập và vận hành theo nhiều cách khác nhau.

### 1. Synchronous Replication Configuration

Trong synchronous replication, các thay đổi dữ liệu được sao chép đến replicas ngay lập tức trước khi transaction hoàn tất.

- Transaction chỉ được hoàn tất sau khi ít nhất một replica xác nhận đã nhận được bản cập nhật.
- Đảm bảo tính consistency dữ liệu cao vì primary và replicas luôn được đồng bộ hóa hoàn toàn (synchronized).

### 2. Asynchronous Replication Configuration

Trong asynchronous replication, primary database gửi các bản cập nhật đến replicas mà không cần chờ xác nhận.

- The primary database hoàn tất transaction ngay lập tức, giúp cải thiện performance và tốc độ.
- Replicas có thể nhận được bản cập nhật với delay nhỏ, điều này có thể dẫn đến inconsistency dữ liệu tạm thời.

### 3. Semi-synchronous Replication Configuration

Semi-synchronous replicatio là một phương pháp lai kết hợp các đặc điểm của sao chép đồng bộ (synchronous) và không đồng bộ (asynchronous).

- The primary database chờ xác nhận từ ít nhất một replica trước khi committing the transaction.
- Other replicas nhận cập nhật không đồng bộ (asynchronously), giúp cải thiện hiệu suất đồng thời duy trì tính nhất quán (consistency) hợp lý.
## Importance

Database replication rất quan trọng vì nhiều lý do.

- ****High Availability****
- ****Disaster Recovery****
- ****Load Balancer / Load Balancing****
- ****Fault Tolerance****
- ****Scalability****
- ****Data Locality****

## Challenges

Một số thách thức của Database Replication là:

- ****Data Consistency****
- ****Complexity****
- ****Cost****
- ****Conflict Resolution****
- ****Latency****

Source: https://www.geeksforgeeks.org/system-design/database-replication-and-their-types-in-system-design/


------------------------------------------------------------------
# Continue (Optional)

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