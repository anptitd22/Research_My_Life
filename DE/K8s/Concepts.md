**I. Khái niệm cơ bản**

Kubernetes là một nền tảng mã nguồn mở, có tính di động và khả mở rộng, dùng để quản lý các khối lượng công việc và dịch vụ được đóng gói trong container, tạo điều kiện thuận lợi cho cả cấu hình khai báo và tự động hóa. Nó có một hệ sinh thái lớn và đang phát triển nhanh chóng. Các dịch vụ, hỗ trợ và công cụ của Kubernetes được cung cấp rộng rãi.

- Do nhu cầu về thiết kế microservice sinh ra hàng trăm, nghìn các container độc lập, kubernetes sinh ra để quản lý chúng như là một công nghệ điều phối
    

Đặc điểm:

- Tính khả dụng cao (high availability or no down time)

- Tính mở rộng cao (scalability or high performance)

- Khả năng phục hồi (disaster recovery - backup and restore)

  

K8s có cấu trúc

- Một master node - một vài worker node có kubelet (tiến trình kubernetes) chạy trên đó có thể giao tiếp với nhau

- Bên trong các node là các container

- Master node sẽ chạy các tiến trình cần thiết để quản lý cụm

- API server = một endpoint truy cập vào cụm kubernetes
    
- Controller Manager = Kiểm tra, theo dõi những gì xảy ra trong cụm (lỗi container, ...)
    
- Scheduler = lập lịch các container dựa trên khối lượng công việc và tài nguyên máy chủ có sẵn
    
- etcd = lưu trữ trạng thái hiện tại của cụm kubernetes trong bất cứ thời điểm nào vì nó chứa tất cả dữ liệu cấu hình bên trong tất cả dữ liệu trạng thái của mỗi node và mỗi container trong node đó
    
    - Nó có thể backup-restore nhờ cơ chế snapshot etcd
        

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_cb7e261a2e81881b.png)

- Các node (worker, master) giao tiếp với nhau thông qua virtual network

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_d3d016315ee2d52d.png)

  

- Các worker node:

- higher workload : Chịu tải nặng nhất
    
- Ứng dụng bên trong thường nặng hơn và tốn nhiều tài nguyên hơn
    
    - Trong khi node master chỉ chạy các tiến trình master nên ít tài nguyên
        

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_274f4ebba9338adf.png)

- Node master rất quan trọng nên thường có các bản sao để khôi phục và dự phòng

- Nếu không có master, cụm sẽ không thể truy cập vào được nữa

  

**II. Kiến trúc sâu**

**1. Pod**

- Thành phần cơ bản và nhỏ nhất trong kubernetes

- Lớp trừu tượng của container (docker)
    
- Tạo môi trường hoặc lớp trên cùng container
    
- Có thể thay thế chúng nếu muốn nên người dùng không cần làm việc trực tiếp với chúng (docker, bất kì công nghệ container nào) mà chỉ làm việc với kubernetes
    

- Pod application:

- Pod application (my-app) chứa ứng dụng chính
    
- Pod database với container riêng
    
- 1 pod thường chạy 1 container application bên trong
    
    - Trường hợp chạy nhiều container: thông thường 1 container chính - một số container phụ (services,...)
        
- Mỗi pod có một IP riêng có thể giao tiếp nội bộ (không phải container application)
    
    - Container application có thể giao tiếp với pod bằng IP đó
        
- Pod là tạm thời, có thể hủy do lỗi, chủ động, hết tài nguyên, ...
    

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_2078565bc4bcced3.png)

- Nếu pod cũ die, pod mới tạo ra với IP khác dẫn đến phải điều chỉnh container giao tiếp IP mới mỗi khi pod khởi động lại

**2. Service**

- Service: là IP tĩnh, cố định được gắn trên mỗi pod, các pod giao tiếp thông qua service

- Vòng đời của service không liên quan đến pod ngay cả khi pod die

- Các node replication cũng sẽ chung 1 service và service sẽ đóng là một cân bằng tải nó sẽ bắt tín hiệu và chuyển đến node ít bận rộn hơn

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_13ca65dc52a2a72b.png)

  

**3. Ingress**

- Bảo mật khi deploy

- Định tuyến lưu lượng truu cập

  

**4. ConfigMap**

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_9919f0c211cbccd8.png)

- Để chạy một pod thông thường cần phải re-build docker -> push docker registry xong pod sẽ pull về và chạy -> rất lâu

  

- ConfigMap: cấu hình bên ngoài cho ứng dụng, chứa dữ liệu như URL của database, 1 số dịch vụ khác mà bạn sử dụng và chỉ cần connect nó với pod, pod sẽ nhận được dữ liệu mà configmap chứa

- Nếu bạn sửa tên hoặc endpoint của service -> chỉ cần sửa configmap là xong mà không cần build lại

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_aa97750eb549e85b.png)

**5. Secret**

**-** Lưu trữ bảo mật mã khóa mật khẩu, chứng chỉ,... cho các application

- Mã khóa nhờ bên thứ 3

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_b0236dfedc2c010e.png)

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_dd143169ea89dd4a.png)

**6. Volumes**

**-** Lưu trữ lâu dài, đáng tin cậy cho các pod ngay cả khi pod bị die

- Gắn địa chỉ bộ nhớ vật lý của ổ cứng lên pod

- Có thể được lưu ở local hoặc remote

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_887b6842a9a31a97.png)

- Kubernetes sẽ không đảm bảo và chịu trách nhiệm cho volumes, admin phải chủ động sao lưu, sao chép, quản lý

**7. Deploy**

![](file:///tmp/lu600071udmnu.tmp/lu600071udmoa_tmp_603469d2b2fd1009.png)

- Khi deployment ta sẽ không làm việc trực tiếp với pod

- được gọi là blueprint pod tạo ra các bản sao của pod khi deploy

**8. Stateful**

- Kiểm tra trạng thái