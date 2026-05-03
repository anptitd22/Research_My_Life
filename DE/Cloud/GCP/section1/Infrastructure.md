# Level 1: Regions

Khu vực đô thị lớn, địa điểm cụ thể, độc lập (London, Tokyo, ...) 

Thiết kế độc lập, cách biệt với nhau: sự cố ở region này không ảnh hưởng đến region khác

Khối nền tảng cho việc phục hồi sau thảm họa (disaster recovery)

![alt][images/Cloud/regions1.png]

# Level 2: Zones

![alt][images/Cloud/zone1.png]

Trong mỗi region có nhiều zones (phân vùng), là khu vực triển khai bên trong region đó 

Các zones được thiết kế riêng biệt theo mặt vật lý (hạ tầng, mạng, ...) nhưng rất gần nhau và được kết nối bằng các liên kết mạng tốc độ cao, độ trễ thấp , sự cố ở zone này không ảnh hưởng đến zone khác -> HA

Một số zone có thể nằm trên cùng một data center nhưng đa số nằm rải rác trên nhiều trung tâm dữ liệu vật lý để dự phòng

## Triển khai

![alt][images/Cloud/zone2.png]


Bảo vệ application khỏi lỗi của một data center duy nhất -> multi-zone 
Bảo vệ khỏi thảm họa cấp vùng -> multi-region

# Level 3: Edge Locations

![alt][images/Cloud/edgelocations1.png]

Gồm hàng ngàn đường dẫn (điểm) đưa user vào mạng tốc độ cao riêng của GG, rải rác khắp thế giới (nhiều hơn regions rất nhiều)

Nhiệm vụ: kết nối user với mạng GG ở vị trí gần họ nhất 

Cloud CDN: Sử dụng các Edge locations này để lưu cache nội dụng sao cho người dùng ở Sydney có thể nhận content từ Sydney server mà không phải US data center -> giảm đáng kể độ trễ, cải thiện trải nghiệm

Google sở hữu một trong những mạng lưới tư nhân lớn nhất thế giới:
- **Premium Tier:** Dữ liệu được truyền tải trên mạng xương sống riêng của Google. Mạng này có băng thông cực lớn và được quản lý chặt chẽ để đảm bảo hiệu suất ổn định, tránh tình trạng nghẽn mạch thường thấy trên internet công cộng.
- **Standard Tier (Ngược lại):** Dữ liệu sẽ di chuyển phần lớn trên internet công cộng (Transit ISP) và chỉ đi vào mạng Google tại vùng (region) đích. Việc này khiến tốc độ phụ thuộc vào chất lượng của các nhà mạng bên thứ ba.
