Hệ thống phân cấp tài nguyên

![alt][images/Cloud/hierarchy1.png]

![alt][images/Cloud/hierarchy2.png]

# Organization

Nó đại diện cho toàn bộ công ty bạn (như mycompany.com) và được liên kết với tài khoản Google workspace hoặc Cloud Identity của doanh nghiệp

# Folders

Group project, phân theo từng phòng ban

# Projects

Là container cơ bản nơi các tài nguyên thực sự tồn tại, mọi thứ tạo ra: máy ảo (VM), storage buckets, databases, ...

Nơi quản lý API, quản lý thanh toán, kiểm soát tài nguyên, ...

# Resources

Đây là các dịch vụ thực tế sử dụng: Compute Engine instance, BigQuery, ... 

Mọi tài nguyên đơn lẻ đều nằm trong project

# Policy Inheritance

![alt][images/Cloud/policy_inheritance1.png]

Mọi chính sách của lớp cha sẽ được lớp con kế thừa

