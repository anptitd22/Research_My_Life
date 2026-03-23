![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_5683449e5db7beca.png)

**I. Khái niệm**

Là một container đóng gói mọi thứ mà một application cần để chạy (thư viện, môi trường, source code, ...)

Dễ dàng chia sẻ

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_2456cef89d60f434.png)

- Không bị ràng buộc bởi hệ điều hành

- Thuận tiện cho việc cài đặt

- Đóng gói các ứng dụng

- Tránh bị xung đột

- 1 docker image -> docker container khi nó chạy trên 1 docker engine

**II. Docker Image**

Là một bản thiết kế của container chứa các mã, thư viện, biến môi trường,.. cần thiết để chạy một application

- Cấu tạo từ các layer bất biến, khi thay đổi image docker sẽ thêm 1 layer khác ở bên trên nó

Layer:

- Khi chạy dockerfile, mỗi construction (câu lệnh) sẽ tạo một layer, có khả năng tái sử dụng và mở rộng,

- Mỗi lớp đại diện cho sự thay đổi của hệ thống (thêm, sửa, xóa)
    
- Layer cuối cùng để chạy một container
    

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_4a14a4e59702f639.png)

- Có thể tận dụng layer cho nhiều image

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_650ed349ce405732.png)

- Các bước chạy layer sẽ được lưu vào cache

- Là chìa khóa để thiết kế một image nhanh, gọn

- Ví dụ:

FROM alpine:latest AS downloader

ARG POSTGRESQL_CONNECTOR_VERSION=42.7.4

RUN apk add --no-cache curl

WORKDIR /downloads

RUN curl -L -o postgresql-${POSTGRESQL_CONNECTOR_VERSION}.jar \

[https://repo1.maven.org/maven2/org/postgresql/postgresql/${POSTGRESQL_CONNECTOR_VERSION}/postgresql-${POSTGRESQL_CONNECTOR_VERSION}.jar](https://repo1.maven.org/maven2/org/postgresql/postgresql/$%7BPOSTGRESQL_CONNECTOR_VERSION%7D/postgresql-$%7BPOSTGRESQL_CONNECTOR_VERSION%7D.jar)

- Dockerfile trên sẽ tạo ra 3 layer và 1 layer image cuối (lớp ghi- writable layer) và alpine là image base
    

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_1c09f839c366c618.png)

- Khi một container được chạy nó sẽ sử dụng union filesystem terminology để hợp nhất các layer vào filesystem được docker sử dụng -> cho phép tái sử dụng các layer khác nhau, có thể xem sự thay đổi thông qua câu lệnh 'docker inspect <container ID>'

- Lower Directories: các file layer từ image
    
- Upper Directory: là không gian duy nhất dành riêng cho các container đó, cho phép một image cho thể sử dụng cho nhiều container và một sự thay đổi của một container sẽ không được thấy bởi container khác
    
- Merged Directory:
    

Image Base:

- Image Base là nền tảng để xây dựng các image khác. Có thể sử dụng bất kỳ image nào làm image base. Tuy nhiên, một số image được tạo ra có chủ đích như những khối xây dựng, cung cấp nền tảng hoặc điểm khởi đầu cho một ứng dụng.

**III. Docker file**

- Là một bản thiết kế của container, là một công thức build docker image, là snapshot mà application được đóng gói cùng với các dependence của tệp

- Command:

Some of the most common instructions in a Dockerfile include:

- FROM <image> - this specifies the base image that the build will extend.
    
- WORKDIR <path> - this instruction specifies the "working directory" or the path in the image where files will be copied and commands will be executed.
    
- COPY <host-path> <image-path> - this instruction tells the builder to copy files from the host and put them into the container image.
    
- RUN <command> - this instruction tells the builder to run the specified command.
    
- ENV <name> <value> - this instruction sets an environment variable that a running container will use.
    
- EXPOSE <port-number> - this instruction sets configuration on the image that indicates a port the image would like to expose.
    
- USER <user-or-uid> - this instruction sets the default user for all subsequent instructions.
    
- CMD ["<command>", "<arg1>"] - this instruction sets the default command a container using this image will run.
    

To read through all of the instructions or go into greater detail, check out the [](https://docs.docker.com/engine/reference/builder/)[Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

- Nhanh chóng đóng gói các dự án mới vào container với 'docker init'.

- Tip build chuẩn: [https://docs.docker.com/build/building/best-practices/](https://docs.docker.com/build/building/best-practices/)

- Cách tag, publish: [https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/)

- Sử dụng cache:

- Bất kỳ thay đổi nào đối với các tệp được sao chép vào image bằng lệnh COPY hoặc ADD. Docker theo dõi mọi thay đổi đối với các tệp trong thư mục dự án của bạn. Cho dù đó là thay đổi về nội dung hay các thuộc tính như quyền truy cập, Docker đều coi những sửa đổi này là tác nhân để vô hiệu hóa bộ nhớ cache.
    
- Khi một lớp bị vô hiệu hóa, tất cả các lớp tiếp theo cũng sẽ bị vô hiệu hóa. Nếu bất kỳ layer nào trước đó, bao gồm cả image base hoặc các layer trung gian, đã bị vô hiệu hóa do thay đổi, Docker sẽ đảm bảo rằng các layer tiếp theo phụ thuộc vào nó cũng sẽ bị vô hiệu hóa. Điều này giúp quá trình xây dựng được đồng bộ và ngăn ngừa sự không nhất quán.
    
- nên --no-cache cho các câu lệnh curl vì nó sẽ làm nặng image đôi khi các file tải về đã xóa mà muốn tải lại nhưng do kết quả lưu ở cache nên nó sẽ bỏ qua bước đó
    

Multi-stage builds:

- Tách biệt các dependence trong quá trình runner.

- Xây dựng đa giai đoạn (Multi-stage builds) đưa nhiều giai đoạn vào Dockerfile của bạn, mỗi giai đoạn có một mục đích cụ thể. Hãy hình dung nó như khả năng chạy các phần khác nhau của quá trình xây dựng trong nhiều môi trường khác nhau, đồng thời. Bằng cách tách biệt môi trường xây dựng khỏi môi trường chạy cuối cùng, bạn có thể giảm đáng kể kích thước image và attack surface. Điều này đặc biệt có lợi cho các ứng dụng có nhiều dependencies trong quá trình xây dựng.

**IV. Registry**

- Là kho lưu trữ docker image nơi đẩy image và lấy image

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_6f99f583555139d5.png)

- Một số kho:

- Docker Hub (Mặc định)
    
- Amazon ECR
    
- Github Container Registry
    
- Azure Container Registry
    
- Internal Container Registry
    

**V. Docker Compose**

- Giúp cấu hình và chạy các ứng dụng đa container, đảm bảo tính nhất quán của dự án

Dockerfile so với Compose file Dockerfile cung cấp các hướng dẫn để xây dựng image container, trong khi Compose file định nghĩa các container đang chạy. Thông thường, Compose file sẽ tham chiếu đến Dockerfile để xây dựng image dùng cho một dịch vụ cụ thể.

**VI.Docker network**

So sánh driver bride, host

Container được cấp phát CPU, memory, ... như nào

Docker khác VM như nào

  

**VII. Docker Volume**

Volume là kho lưu trữ dữ liệu bền vững cho các container, được tạo và quản lý bởi Docker. Bạn có thể tạo volume một cách rõ ràng bằng lệnh `docker volume create`, hoặc Docker có thể tạo volume trong quá trình tạo container hoặc service.

Khi bạn tạo một volume, nó được lưu trữ trong một thư mục trên máy chủ Docker. Khi bạn gắn volume đó vào một container, thư mục này chính là thứ được gắn vào container. Điều này tương tự như cách hoạt động của bind mount, ngoại trừ việc volume được quản lý bởi Docker và được tách biệt khỏi chức năng cốt lõi của máy chủ.

When to use volumes

Volumes là cơ chế được ưu tiên để lưu trữ dữ liệu được tạo ra và sử dụng bởi các container Docker. Trong khi bind mounts phụ thuộc vào cấu trúc thư mục và hệ điều hành của máy chủ, volumes được quản lý hoàn toàn bởi Docker. Volumes là lựa chọn tốt cho các trường hợp sử dụng sau:

- Việc sao lưu hoặc di chuyển các volume dễ dàng hơn so với việc sử dụng bind mount.
    
- Bạn có thể quản lý dung lượng lưu trữ bằng các lệnh CLI của Docker hoặc API của Docker.
    
- Volumes hoạt động trên cả container Linux và Windows.
    
- Volumes có thể được chia sẻ an toàn hơn giữa nhiều container.
    
- New volumes có thể được điền sẵn nội dung bằng container hoặc build.
    
- Khi ứng dụng của bạn yêu cầu hiệu năng I/O cao.
    

  

Volume không phải là lựa chọn tốt nếu bạn cần truy cập các tệp từ máy chủ, vì volume được quản lý hoàn toàn bởi Docker. Hãy sử dụng bind mount nếu bạn cần truy cập các tệp hoặc thư mục từ cả container và máy chủ.

Việc sử dụng volume thường tốt hơn so với việc ghi dữ liệu trực tiếp vào container, bởi vì volume không làm tăng kích thước của các container sử dụng nó. Sử dụng volume cũng nhanh hơn; việc ghi vào lớp ghi được của container(container's writable layer) yêu cầu trình điều khiển lưu trữ([storage driver](https://docs.docker.com/engine/storage/drivers/)) để quản lý hệ thống tệp. The storage driver cung cấp một union filesystem, sử dụng nhân Linux. Lớp trừu tượng bổ sung này làm giảm hiệu suất so với việc sử dụng volume, vốn ghi trực tiếp vào hệ thống tệp của máy chủ.

Nếu container của bạn tạo ra dữ liệu trạng thái không bền vững, hãy cân nhắc sử dụng mount tmpfs để tránh lưu trữ dữ liệu ở bất kỳ đâu vĩnh viễn và để tăng hiệu suất của container bằng cách tránh ghi vào lớp ghi được của container(the container's writable layer).

  

Các volume sử dụng cơ chế truyền liên kết rprivate (riêng tư đệ quy), và cơ chế truyền liên kết này không thể cấu hình được đối với các volume.

A volume's lifecycle

Nội dung của một volume tồn tại bên ngoài vòng đời của một container nhất định. Khi một container bị hủy, lớp ghi (writable layer) cũng bị hủy theo. Việc sử dụng volume đảm bảo rằng dữ liệu được lưu giữ ngay cả khi container sử dụng nó bị xóa.

Một volume nhất định có thể được gắn vào nhiều container cùng lúc. Khi không có container nào đang chạy sử dụng volume đó, volume vẫn khả dụng cho Docker và không bị

xóa tự động. Bạn có thể xóa các volume không sử dụng bằng lệnh `docker volume prune`.

  

Mounting a volume over existing data

Nếu bạn mount a _non-empty volume_ vào một thư mục trong container mà trong đó đã có các tệp hoặc thư mục, thì các tệp đã tồn tại trước đó sẽ bị che khuất bởi thao tác mount. Điều này tương tự như việc bạn lưu các tệp vào thư mục /mnt trên máy chủ Linux, rồi sau đó gắn ổ USB vào /mnt. Nội dung của /mnt sẽ bị che khuất bởi nội dung của ổ USB cho đến khi ổ USB được tháo ra.

Với container, không có cách nào đơn giản để gỡ bỏ điểm mount nhằm hiển thị lại các tập tin bị ẩn. Lựa chọn tốt nhất của bạn là tạo lại container mà không có điểm mount đó.

Nếu bạn gắn một volume trống (_empty volume)_ vào một thư mục trong container mà trong đó đã có các tệp hoặc thư mục, thì các tệp hoặc thư mục này sẽ được sao chép vào volume theo mặc định. Tương tự, nếu bạn khởi động một container và chỉ định một volume chưa tồn tại, một volume trống sẽ được tạo cho bạn. Đây là một cách tốt để điền trước dữ liệu mà một container khác cần.

Để ngăn Docker sao chép các tệp đã tồn tại của container vào một volume trống, hãy sử dụng tùy chọn volume-nocopy, xem Tùy chọn cho --mount([Options for --mount](https://docs.docker.com/engine/storage/volumes/#options-for---mount).).

  

Named and anonymous volumes

Một volume có thể được đặt tên hoặc ẩn danh. Các volume ẩn danh được đặt một tên ngẫu nhiên, đảm bảo là duy nhất trong một máy chủ Docker nhất định. Giống như các volume được đặt tên, các volume ẩn danh vẫn tồn tại ngay cả khi bạn xóa container sử dụng chúng, ngoại trừ trường hợp bạn sử dụng cờ --rm khi tạo container, trong trường hợp đó volume ẩn danh được liên kết với container sẽ bị xóa. Xem phần Xóa các volume ẩn danh([Remove anonymous volumes](https://docs.docker.com/engine/storage/volumes/#remove-anonymous-volumes)).

Nếu bạn tạo nhiều container liên tiếp, mỗi container đều sử dụng volume ẩn danh, thì mỗi container sẽ tạo một volume riêng. Volume ẩn danh không được tự động tái sử dụng hoặc chia sẻ giữa các container. Để chia sẻ một volume ẩn danh giữa hai hoặc nhiều container, bạn phải gắn volume ẩn danh bằng ID volume ngẫu nhiên.

  

Syntax

Để gắn kết một volume bằng lệnh `docker run`, bạn có thể sử dụng cờ `--mount` hoặc `--volume`.

```

docker run --mount type=volume,src=<volume-name>,dst=<mount-path>

docker run --volume <volume-name>:<mount-path>

```

Nhìn chung, nên sử dụng tùy chọn --mount. Sự khác biệt chính là cờ --mount rõ ràng hơn và hỗ trợ tất cả các tùy chọn có sẵn.

Bạn phải sử dụng --mount nếu muốn:

- Specify [](https://docs.docker.com/engine/storage/volumes/#use-a-volume-driver)[volume driver options](https://docs.docker.com/engine/storage/volumes/#use-a-volume-driver)
    
- Mount a [](https://docs.docker.com/engine/storage/volumes/#mount-a-volume-subdirectory)[volume subdirectory](https://docs.docker.com/engine/storage/volumes/#mount-a-volume-subdirectory)
    
- Mount a volume into a Swarm service
    

  
  

**VIII. Docker bind mount**

Khi sử dụng bind mount, một tệp hoặc thư mục trên máy chủ được gắn từ máy chủ vào container. Ngược lại, khi sử dụng volume, một thư mục mới được tạo trong thư mục lưu trữ của Docker trên máy chủ, và Docker quản lý nội dung của thư mục đó.

Dùng khi

- Chia sẻ mã nguồn hoặc các tệp tạo lập giữa môi trường phát triển trên máy chủ Docker và container.
    
- Khi bạn muốn tạo hoặc tạo ra các tệp trong một container và lưu trữ các tệp đó vào hệ thống tệp của máy chủ.
    
- Chia sẻ các tệp cấu hình từ máy chủ đến các container. Đây là cách Docker cung cấp khả năng phân giải DNS cho các container theo mặc định, bằng cách mount tệp /etc/resolv.conf từ máy chủ vào mỗi container.
    

Tính năng gắn kết thư mục (bind mount) cũng có sẵn cho quá trình builds: bạn có thể gắn kết mã nguồn từ máy chủ vào build container để kiểm tra, kiểm tra cú pháp hoặc biên dịch dự án.

**Bind-mounting over existing data**

Nếu bind mount tệp hoặc thư mục vào một thư mục trong container mà các tệp hoặc thư mục đó đã tồn tại, thì các tệp đã tồn tại trước đó sẽ bị che khuất bởi thao tác mount. Điều này tương tự như việc bạn lưu các tệp vào thư mục /mnt trên máy chủ Linux, rồi gắn ổ USB vào /mnt. Nội dung của /mnt sẽ bị che khuất bởi nội dung của ổ USB cho đến khi ổ USB được tháo ra.

Với container, không có cách nào đơn giản để gỡ bỏ điểm mount nhằm hiển thị lại các tập tin bị ẩn. Lựa chọn tốt nhất của bạn là tạo lại container mà không có điểm mount đó.

**Considerations and constraints**

Theo mặc định, các mount liên kết có quyền ghi vào các tệp trên máy chủ.

Một tác dụng phụ của việc sử dụng bind mount là bạn có thể thay đổi hệ thống tập tin của máy chủ thông qua các tiến trình chạy trong container, bao gồm việc tạo, sửa đổi hoặc xóa các tập tin hoặc thư mục hệ thống quan trọng. Khả năng này có thể gây ra những vấn đề về bảo mật. Ví dụ, nó có thể ảnh hưởng đến các tiến trình không phải Docker trên hệ thống máy chủ.

Bạn có thể sử dụng tùy chọn `readonly` hoặc `ro` để ngăn container ghi dữ liệu vào thư mục được mount.

Các điểm gắn kết (bind mount) được tạo trên máy chủ Docker daemon, chứ không phải trên máy khách - client.

Nếu bạn đang sử dụng remote daemon Docker, bạn không thể tạo blind mount để truy cập các tệp trên máy khách-client trong container.

Đối với Docker Desktop, tiến trình nền (daemon) chạy bên trong máy ảo Linux, chứ không phải trực tiếp trên máy chủ gốc. Docker Desktop có các cơ chế tích hợp sẵn giúp xử lý việc mount thư mục một cách minh bạch, cho phép bạn chia sẻ đường dẫn hệ thống tệp của máy chủ gốc với các container chạy trong máy ảo.

Các container có bind mount được liên kết chặt chẽ với máy chủ.

Việc gắn kết thư mục (bind mount) phụ thuộc vào việc hệ thống tệp của máy chủ có cấu trúc thư mục cụ thể hay không. Điều này có nghĩa là các container sử dụng gắn kết thư mục có thể bị lỗi nếu chạy trên một máy chủ khác không có cùng cấu trúc thư mục.

**Syntax**

Để tạo liên kết gắn kết (bind mount), bạn có thể sử dụng cờ --mount hoặc --volume.

```

docker run --mount type=bind,src=<host-path>,dst=<container-path>

docker run --volume <host-path>:<container-path>

```

Nhìn chung, nên sử dụng tùy chọn --mount. Sự khác biệt chính là cờ --mount rõ ràng hơn và hỗ trợ tất cả các tùy chọn có sẵn.

Nếu bạn sử dụng tham số --volume để mount một tệp hoặc thư mục chưa tồn tại trên máy chủ Docker, Docker sẽ tự động tạo thư mục đó trên máy chủ cho bạn. Thư mục này luôn được tạo dưới dạng một thư mục con.

Tùy chọn --mount không tự động tạo thư mục nếu đường dẫn gắn kết được chỉ định không tồn tại trên máy chủ. Thay vào đó, nó sẽ báo lỗi:

```

docker run --mount type=bind,src=/dev/noexist,dst=/mnt/foo alpine

docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /dev/noexist.

```

Options for --mount

Cờ --mount bao gồm nhiều cặp key-value, được phân tách bằng dấu phẩy và mỗi cặp gồm một bộ <key>=<value>. Thứ tự của các khóa không quan trọng.

docker run --mount type=bind,src=<host-path>,dst=<container-path>[,<key>=<value>...]

Các tùy chọn hợp lệ cho --mount type=bind bao gồm:

  

|   |   |
|---|---|
 
|source, src|Vị trí của tệp hoặc thư mục trên máy chủ. Đây có thể là đường dẫn tuyệt đối hoặc tương đối.|
|destination, dst, target|Đường dẫn nơi tệp hoặc thư mục được mount trong container. Phải là đường dẫn tuyệt đối.|
|readonly, ro|Nếu có, tùy chọn này sẽ khiến thư mục được mount vào container ở chế độ chỉ đọc.|
|bind-propagation|Nếu có, nó sẽ thay đổi quá trình bind-propagation.|

  

Ví dụ:

```

docker run --mount type=bind,src=.,dst=/project,ro,bind-propagation=rshared

```

Options for --volume

Cờ --volume hoặc -v bao gồm ba trường, được phân tách bằng dấu hai chấm (:). Các trường phải được sắp xếp đúng thứ tự.

```

docker run -v <host-path>:<container-path>[:opts]

```

Trường đầu tiên là đường dẫn trên máy chủ để mount vào container. Trường thứ hai là đường dẫn nơi tệp hoặc thư mục được mount trong container.

Trường thứ ba là tùy chọn, và là một danh sách các tùy chọn được phân tách bằng dấu phẩy. Các tùy chọn hợp lệ cho --volume với bind mount bao gồm:

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_6ec8fa4ff98c3419.png)

[https://docs.docker.com/engine/storage/bind-mounts/#considerations-and-constraints](https://docs.docker.com/engine/storage/bind-mounts/#considerations-and-constraints)

  

Ví dụ:

```

docker run -v .:/project:ro,rshared

```

tmpfs mounts

Volumes và bind mounts cho phép bạn chia sẻ tập tin giữa máy chủ và container để dữ liệu có thể được lưu giữ ngay cả sau khi container dừng hoạt động.

Nếu bạn đang chạy Docker trên Linux, bạn có thêm một lựa chọn thứ ba: tmpfs mount. Khi bạn tạo một container với tmpfs mount, container đó có thể tạo các tệp bên ngoài lớp ghi được của container(container's writable layer).

Khác với các volume và bind mount, tmpfs mount chỉ là tạm thời và chỉ được lưu trữ trong bộ nhớ của máy chủ. Khi container dừng hoạt động, tmpfs mount sẽ bị xóa và các tệp được ghi vào đó sẽ không được lưu giữ.

Việc sử dụng tmpfs (TPF) là lựa chọn tốt nhất trong trường hợp bạn không muốn dữ liệu được lưu trữ lâu dài trên máy chủ hoặc bên trong container. Điều này có thể vì lý do bảo mật hoặc để bảo vệ hiệu năng của container khi ứng dụng của bạn cần ghi một lượng lớn dữ liệu trạng thái không được lưu trữ lâu dài.

Lưu ý:

Các điểm tmpfs mount trong Docker ánh xạ trực tiếp đến tmpfs trong nhân Linux. Do đó, dữ liệu tạm thời có thể được ghi vào tệp hoán đổi (swap file), và nhờ vậy được lưu trữ lâu dài trên hệ thống tệp.

  

Mounting over existing data

Nếu bạn tạo một mount tmpfs vào một thư mục trong container mà trong đó đã có các tệp hoặc thư mục, thì các tệp đã tồn tại trước đó sẽ bị che khuất bởi mount này. Điều này tương tự như việc bạn lưu các tệp vào /mnt trên máy chủ Linux, rồi sau đó gắn ổ USB vào /mnt. Nội dung của /mnt sẽ bị che khuất bởi nội dung của ổ USB cho đến khi ổ USB được tháo ra.

Với container, không có cách nào đơn giản để gỡ bỏ điểm mount nhằm hiển thị lại các tập tin bị ẩn. Lựa chọn tốt nhất của bạn là tạo lại container mà không có điểm mount đó.

Limitations of tmpfs mounts

Khác với volumes và bind mounts, bạn không thể chia sẻ các mount tmpfs giữa các container.

Chức năng này chỉ khả dụng nếu bạn đang chạy Docker trên Linux.

Việc thiết lập quyền trên tmpfs có thể khiến chúng bị đặt lại sau khi khởi động lại container. Trong một số trường hợp, việc thiết lập uid/gid có thể là giải pháp tạm thời.

Syntax

Để gắn kết tmpfs bằng lệnh docker run, bạn có thể sử dụng cờ --mount hoặc --tmpfs.

```

docker run --mount type=tmpfs,dst=<mount-path>

docker run --tmpfs <mount-path>

```

Xem thêm: [https://docs.docker.com/engine/storage/tmpfs/](https://docs.docker.com/engine/storage/tmpfs/)

  

**containerd image store with Docker Engine**

  

[**https://docs.docker.com/engine/storage/containerd/**](https://docs.docker.com/engine/storage/containerd/)

  

  

**IX. Docker Engine**

Docker Engine là môi trường chạy container tiêu chuẩn trong ngành, hoạt động trên nhiều hệ điều hành Linux (CentOS, Debian, Fedora, RHEL và Ubuntu) và Windows Server. Docker tạo ra các công cụ đơn giản và phương pháp đóng gói phổ quát, gom tất cả các phụ thuộc của ứng dụng vào trong một container, sau đó được chạy trên Docker Engine. Docker Engine cho phép các ứng dụng được đóng gói trong container chạy ổn định ở bất cứ đâu trên bất kỳ cơ sở hạ tầng nào, giải quyết vấn đề "địa ngục phụ thuộc" cho các nhà phát triển và nhóm vận hành, đồng thời loại bỏ vấn đề "it works on my laptop!".

  

**X. OverlayFS storage driver**

OverlayFS is a union filesystem.

Trang này gọi Linux kernel driver là OverlayFS và Docker storage driver là overlay2.

Lưu ý:

Docker Engine 29.0 trở lên sử dụng kho lưu trữ containerd image ([containerd image store](https://docs.docker.com/engine/storage/containerd/)) theo mặc định. Trình điều khiển overlay2 là trình điều khiển lưu trữ cũ đã được thay thế bởi overlayfs containerd snapshotter. Để biết thêm thông tin, hãy xem [Select a storage driver](https://docs.docker.com/engine/storage/drivers/select-storage-driver/).

For fuse-overlayfs driver, check [Rootless mode documentation](https://docs.docker.com/engine/security/rootless/).

Prerequisites

Trình điều khiển overlay2 được hỗ trợ nếu bạn đáp ứng các điều kiện tiên quyết sau:

- Phiên bản 4.0 trở lên của nhân Linux, hoặc RHEL hoặc CentOS sử dụng phiên bản 3.10.0-514 của nhân trở lên.
    
- Trình điều khiển overlay2 được hỗ trợ trên hệ thống tệp tin dựa trên xfs, nhưng chỉ khi d_type=true được bật.
    
- Sử dụng lệnh xfs_info để kiểm tra xem tùy chọn ftype có được đặt thành 1 hay không. Để định dạng hệ thống tệp xfs đúng cách, hãy sử dụng cờ -n ftype=1.
    

  

Việc thay đổi trình điều khiển lưu trữ(storage driver) sẽ khiến các container và image hiện có không thể truy cập được trên hệ thống cục bộ. Hãy sử dụng lệnh `docker save` để lưu bất kỳ image nào bạn đã xây dựng hoặc đẩy chúng lên Docker Hub hoặc một registry riêng trước khi thay đổi trình điều khiển lưu trữ(storage driver), để bạn không cần phải tạo lại chúng sau này.

  

**How the overlay2 driver works**

OverlayFS xếp chồng hai thư mục trên cùng một máy chủ Linux và hiển thị chúng như một thư mục duy nhất. Các thư mục này được gọi là các lớp (layers), và quá trình hợp nhất được gọi là gắn kết hợp nhất (union mount). OverlayFS gọi thư mục phía dưới là lowerdir và thư mục phía trên là upperdir. Chế độ xem hợp nhất được hiển thị thông qua thư mục riêng của nó có tên là merged.

Trình điều khiển overlay2 hỗ trợ tối đa 128 lower OverlayFS layers. Khả năng này mang lại hiệu suất tốt hơn cho các lệnh Docker liên quan đến lớp như docker build và docker commit, đồng thời tiêu thụ ít inode hơn trên hệ thống tệp hỗ trợ.

  

Xem cấu trúc thư mục OFS: [https://docs.docker.com/engine/storage/drivers/overlayfs-driver/](https://docs.docker.com/engine/storage/drivers/overlayfs-driver/)

  

**OverlayFS and Docker Performance**

Overlay2 có thể hoạt động tốt hơn btrfs. Tuy nhiên, cần lưu ý những chi tiết sau:

- Page caching: OverlayFS hỗ trợ page cache sharing. Nhiều container truy cập cùng một tệp sẽ chia sẻ một mục nhập page cache duy nhất cho tệp đó. Điều này giúp trình điều khiển overlay2 tiết kiệm bộ nhớ và là một lựa chọn tốt cho các trường hợp sử dụng mật độ cao như PaaS.
    
- Copyup: Cũng giống như các hệ thống tệp copy-on-write khác, OverlayFS thực hiện các thao tác sao chép lên (copy-up) mỗi khi một container ghi vào tệp lần đầu tiên. Điều này có thể làm tăng độ trễ trong thao tác ghi, đặc biệt đối với các tệp lớn. Tuy nhiên, một khi tệp đã được sao chép lên, tất cả các thao tác ghi tiếp theo vào tệp đó sẽ diễn ra ở lớp trên, mà không cần thực hiện thêm các thao tác sao chép lên nữa.
    
- Performance best practices: Các nguyên tắc thực hành tốt nhất về hiệu suất chung sau đây áp dụng cho OverlayFS.
    
    - Use fast storage: Ổ cứng thể rắn (SSD) cung cấp tốc độ đọc và ghi nhanh hơn so với ổ đĩa cơ học.
        
    - Use volumes for write-heavy workloads: Volumes cung cấp hiệu năng tốt nhất và ổn định nhất cho các tác vụ ghi dữ liệu nhiều. Điều này là do chúng bỏ qua trình storage driver và không phát sinh bất kỳ chi phí tiềm tàng nào do cấp phát mỏng (thin provisioning) và sao chép khi ghi (copy-on-write) gây ra. Volumes còn có những lợi ích khác, chẳng hạn như cho phép bạn chia sẻ dữ liệu giữa các container và duy trì dữ liệu ngay cả khi không có container nào đang chạy sử dụng chúng.
        

**Limitations on OverlayFS compatibility**

open(2): OverlayFS chỉ triển khai một tập hợp con của các tiêu chuẩn POSIX. Điều này có thể dẫn đến một số thao tác của OverlayFS vi phạm các tiêu chuẩn POSIX. Một trong những thao tác đó là thao tác sao chép lên (copy-up). Giả sử ứng dụng của bạn gọi fd1=open("foo", O_RDONLY) và sau đó fd2=open("foo", O_RDWR). Trong trường hợp này, ứng dụng của bạn mong đợi fd1 và fd2 tham chiếu đến cùng một tệp. Tuy nhiên, do một thao tác sao chép lên xảy ra sau lần gọi thứ hai đến open(2), các mô tả tham chiếu đến các tệp khác nhau. fd1 tiếp tục tham chiếu đến tệp trong image(lowerdir) và fd2 tham chiếu đến tệp trong container (upperdir). Một giải pháp khắc phục cho vấn đề này là chạm vào các tệp, điều này gây ra thao tác sao chép lên. Tất cả các thao tác open(2) tiếp theo, bất kể chế độ truy cập chỉ đọc hay đọc-ghi, đều tham chiếu đến tệp trong container (upperdir).

Lệnh yum được biết là sẽ bị ảnh hưởng trừ khi gói yum-plugin-ovl được cài đặt. Nếu gói yum-plugin-ovl không có sẵn trong bản phân phối của bạn, chẳng hạn như RHEL/CentOS trước phiên bản 6.8 hoặc 7.2, bạn có thể cần chạy lệnh `touch /var/lib/rpm/*` trước khi chạy `yum install`. Gói này thực hiện giải pháp khắc phục sự cố bằng lệnh `touch` được đề cập ở trên cho lệnh yum.

rename(2): OverlayFS không hỗ trợ đầy đủ lệnh gọi hệ thống rename(2). Ứng dụng của bạn cần phát hiện lỗi và chuyển sang chiến lược "sao chép và hủy liên kết".

**I. Khái niệm**

Là một container đóng gói mọi thứ mà một application cần để chạy (thư viện, môi trường, source code, ...)

Dễ dàng chia sẻ

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_2456cef89d60f434.png)

- Không bị ràng buộc bởi hệ điều hành

- Thuận tiện cho việc cài đặt

- Đóng gói các ứng dụng

- Tránh bị xung đột

- 1 docker image -> docker container khi nó chạy trên 1 docker engine

**II. Docker Image**

Là một bản thiết kế của container chứa các mã, thư viện, biến môi trường,.. cần thiết để chạy một application

- Cấu tạo từ các layer bất biến, khi thay đổi image docker sẽ thêm 1 layer khác ở bên trên nó

Layer:

- Khi chạy dockerfile, mỗi construction (câu lệnh) sẽ tạo một layer, có khả năng tái sử dụng và mở rộng,

- Mỗi lớp đại diện cho sự thay đổi của hệ thống (thêm, sửa, xóa)
    
- Layer cuối cùng để chạy một container
    

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_4a14a4e59702f639.png)

- Có thể tận dụng layer cho nhiều image

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_650ed349ce405732.png)

- Các bước chạy layer sẽ được lưu vào cache

- Là chìa khóa để thiết kế một image nhanh, gọn

- Ví dụ:

FROM alpine:latest AS downloader

ARG POSTGRESQL_CONNECTOR_VERSION=42.7.4

RUN apk add --no-cache curl

WORKDIR /downloads

RUN curl -L -o postgresql-${POSTGRESQL_CONNECTOR_VERSION}.jar \

[https://repo1.maven.org/maven2/org/postgresql/postgresql/${POSTGRESQL_CONNECTOR_VERSION}/postgresql-${POSTGRESQL_CONNECTOR_VERSION}.jar](https://repo1.maven.org/maven2/org/postgresql/postgresql/$%7BPOSTGRESQL_CONNECTOR_VERSION%7D/postgresql-$%7BPOSTGRESQL_CONNECTOR_VERSION%7D.jar)

- Dockerfile trên sẽ tạo ra 3 layer và 1 layer image cuối (lớp ghi- writable layer) và alpine là image base
    

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_1c09f839c366c618.png)

- Khi một container được chạy nó sẽ sử dụng union filesystem terminology để hợp nhất các layer vào filesystem được docker sử dụng -> cho phép tái sử dụng các layer khác nhau, có thể xem sự thay đổi thông qua câu lệnh 'docker inspect <container ID>'

- Lower Directories: các file layer từ image
    
- Upper Directory: là không gian duy nhất dành riêng cho các container đó, cho phép một image cho thể sử dụng cho nhiều container và một sự thay đổi của một container sẽ không được thấy bởi container khác
    
- Merged Directory:
    

Image Base:

- Image Base là nền tảng để xây dựng các image khác. Có thể sử dụng bất kỳ image nào làm image base. Tuy nhiên, một số image được tạo ra có chủ đích như những khối xây dựng, cung cấp nền tảng hoặc điểm khởi đầu cho một ứng dụng.

**III. Docker file**

- Là một bản thiết kế của container, là một công thức build docker image, là snapshot mà application được đóng gói cùng với các dependence của tệp

- Command:

Some of the most common instructions in a Dockerfile include:

- FROM <image> - this specifies the base image that the build will extend.
    
- WORKDIR <path> - this instruction specifies the "working directory" or the path in the image where files will be copied and commands will be executed.
    
- COPY <host-path> <image-path> - this instruction tells the builder to copy files from the host and put them into the container image.
    
- RUN <command> - this instruction tells the builder to run the specified command.
    
- ENV <name> <value> - this instruction sets an environment variable that a running container will use.
    
- EXPOSE <port-number> - this instruction sets configuration on the image that indicates a port the image would like to expose.
    
- USER <user-or-uid> - this instruction sets the default user for all subsequent instructions.
    
- CMD ["<command>", "<arg1>"] - this instruction sets the default command a container using this image will run.
    

To read through all of the instructions or go into greater detail, check out the [](https://docs.docker.com/engine/reference/builder/)[Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

- Nhanh chóng đóng gói các dự án mới vào container với 'docker init'.

- Tip build chuẩn: [https://docs.docker.com/build/building/best-practices/](https://docs.docker.com/build/building/best-practices/)

- Cách tag, publish: [https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/)

- Sử dụng cache:

- Bất kỳ thay đổi nào đối với các tệp được sao chép vào image bằng lệnh COPY hoặc ADD. Docker theo dõi mọi thay đổi đối với các tệp trong thư mục dự án của bạn. Cho dù đó là thay đổi về nội dung hay các thuộc tính như quyền truy cập, Docker đều coi những sửa đổi này là tác nhân để vô hiệu hóa bộ nhớ cache.
    
- Khi một lớp bị vô hiệu hóa, tất cả các lớp tiếp theo cũng sẽ bị vô hiệu hóa. Nếu bất kỳ layer nào trước đó, bao gồm cả image base hoặc các layer trung gian, đã bị vô hiệu hóa do thay đổi, Docker sẽ đảm bảo rằng các layer tiếp theo phụ thuộc vào nó cũng sẽ bị vô hiệu hóa. Điều này giúp quá trình xây dựng được đồng bộ và ngăn ngừa sự không nhất quán.
    
- nên --no-cache cho các câu lệnh curl vì nó sẽ làm nặng image đôi khi các file tải về đã xóa mà muốn tải lại nhưng do kết quả lưu ở cache nên nó sẽ bỏ qua bước đó
    

Multi-stage builds:

- Tách biệt các dependence trong quá trình runner.

- Xây dựng đa giai đoạn (Multi-stage builds) đưa nhiều giai đoạn vào Dockerfile của bạn, mỗi giai đoạn có một mục đích cụ thể. Hãy hình dung nó như khả năng chạy các phần khác nhau của quá trình xây dựng trong nhiều môi trường khác nhau, đồng thời. Bằng cách tách biệt môi trường xây dựng khỏi môi trường chạy cuối cùng, bạn có thể giảm đáng kể kích thước image và attack surface. Điều này đặc biệt có lợi cho các ứng dụng có nhiều dependencies trong quá trình xây dựng.

**IV. Registry**

- Là kho lưu trữ docker image nơi đẩy image và lấy image

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_6f99f583555139d5.png)

- Một số kho:

- Docker Hub (Mặc định)
    
- Amazon ECR
    
- Github Container Registry
    
- Azure Container Registry
    
- Internal Container Registry
    

**V. Docker Compose**

- Giúp cấu hình và chạy các ứng dụng đa container, đảm bảo tính nhất quán của dự án

Dockerfile so với Compose file Dockerfile cung cấp các hướng dẫn để xây dựng image container, trong khi Compose file định nghĩa các container đang chạy. Thông thường, Compose file sẽ tham chiếu đến Dockerfile để xây dựng image dùng cho một dịch vụ cụ thể.

**VI.Docker network**

So sánh driver bride, host

Container được cấp phát CPU, memory, ... như nào

Docker khác VM như nào

  

**VII. Docker Volume**

Volume là kho lưu trữ dữ liệu bền vững cho các container, được tạo và quản lý bởi Docker. Bạn có thể tạo volume một cách rõ ràng bằng lệnh `docker volume create`, hoặc Docker có thể tạo volume trong quá trình tạo container hoặc service.

Khi bạn tạo một volume, nó được lưu trữ trong một thư mục trên máy chủ Docker. Khi bạn gắn volume đó vào một container, thư mục này chính là thứ được gắn vào container. Điều này tương tự như cách hoạt động của bind mount, ngoại trừ việc volume được quản lý bởi Docker và được tách biệt khỏi chức năng cốt lõi của máy chủ.

When to use volumes

Volumes là cơ chế được ưu tiên để lưu trữ dữ liệu được tạo ra và sử dụng bởi các container Docker. Trong khi bind mounts phụ thuộc vào cấu trúc thư mục và hệ điều hành của máy chủ, volumes được quản lý hoàn toàn bởi Docker. Volumes là lựa chọn tốt cho các trường hợp sử dụng sau:

- Việc sao lưu hoặc di chuyển các volume dễ dàng hơn so với việc sử dụng bind mount.
    
- Bạn có thể quản lý dung lượng lưu trữ bằng các lệnh CLI của Docker hoặc API của Docker.
    
- Volumes hoạt động trên cả container Linux và Windows.
    
- Volumes có thể được chia sẻ an toàn hơn giữa nhiều container.
    
- New volumes có thể được điền sẵn nội dung bằng container hoặc build.
    
- Khi ứng dụng của bạn yêu cầu hiệu năng I/O cao.
    

  

Volume không phải là lựa chọn tốt nếu bạn cần truy cập các tệp từ máy chủ, vì volume được quản lý hoàn toàn bởi Docker. Hãy sử dụng bind mount nếu bạn cần truy cập các tệp hoặc thư mục từ cả container và máy chủ.

Việc sử dụng volume thường tốt hơn so với việc ghi dữ liệu trực tiếp vào container, bởi vì volume không làm tăng kích thước của các container sử dụng nó. Sử dụng volume cũng nhanh hơn; việc ghi vào lớp ghi được của container(container's writable layer) yêu cầu trình điều khiển lưu trữ([storage driver](https://docs.docker.com/engine/storage/drivers/)) để quản lý hệ thống tệp. The storage driver cung cấp một union filesystem, sử dụng nhân Linux. Lớp trừu tượng bổ sung này làm giảm hiệu suất so với việc sử dụng volume, vốn ghi trực tiếp vào hệ thống tệp của máy chủ.

Nếu container của bạn tạo ra dữ liệu trạng thái không bền vững, hãy cân nhắc sử dụng mount tmpfs để tránh lưu trữ dữ liệu ở bất kỳ đâu vĩnh viễn và để tăng hiệu suất của container bằng cách tránh ghi vào lớp ghi được của container(the container's writable layer).

  

Các volume sử dụng cơ chế truyền liên kết rprivate (riêng tư đệ quy), và cơ chế truyền liên kết này không thể cấu hình được đối với các volume.

A volume's lifecycle

Nội dung của một volume tồn tại bên ngoài vòng đời của một container nhất định. Khi một container bị hủy, lớp ghi (writable layer) cũng bị hủy theo. Việc sử dụng volume đảm bảo rằng dữ liệu được lưu giữ ngay cả khi container sử dụng nó bị xóa.

Một volume nhất định có thể được gắn vào nhiều container cùng lúc. Khi không có container nào đang chạy sử dụng volume đó, volume vẫn khả dụng cho Docker và không bị

xóa tự động. Bạn có thể xóa các volume không sử dụng bằng lệnh `docker volume prune`.

  

Mounting a volume over existing data

Nếu bạn mount a _non-empty volume_ vào một thư mục trong container mà trong đó đã có các tệp hoặc thư mục, thì các tệp đã tồn tại trước đó sẽ bị che khuất bởi thao tác mount. Điều này tương tự như việc bạn lưu các tệp vào thư mục /mnt trên máy chủ Linux, rồi sau đó gắn ổ USB vào /mnt. Nội dung của /mnt sẽ bị che khuất bởi nội dung của ổ USB cho đến khi ổ USB được tháo ra.

Với container, không có cách nào đơn giản để gỡ bỏ điểm mount nhằm hiển thị lại các tập tin bị ẩn. Lựa chọn tốt nhất của bạn là tạo lại container mà không có điểm mount đó.

Nếu bạn gắn một volume trống (_empty volume)_ vào một thư mục trong container mà trong đó đã có các tệp hoặc thư mục, thì các tệp hoặc thư mục này sẽ được sao chép vào volume theo mặc định. Tương tự, nếu bạn khởi động một container và chỉ định một volume chưa tồn tại, một volume trống sẽ được tạo cho bạn. Đây là một cách tốt để điền trước dữ liệu mà một container khác cần.

Để ngăn Docker sao chép các tệp đã tồn tại của container vào một volume trống, hãy sử dụng tùy chọn volume-nocopy, xem Tùy chọn cho --mount([Options for --mount](https://docs.docker.com/engine/storage/volumes/#options-for---mount).).

  

Named and anonymous volumes

Một volume có thể được đặt tên hoặc ẩn danh. Các volume ẩn danh được đặt một tên ngẫu nhiên, đảm bảo là duy nhất trong một máy chủ Docker nhất định. Giống như các volume được đặt tên, các volume ẩn danh vẫn tồn tại ngay cả khi bạn xóa container sử dụng chúng, ngoại trừ trường hợp bạn sử dụng cờ --rm khi tạo container, trong trường hợp đó volume ẩn danh được liên kết với container sẽ bị xóa. Xem phần Xóa các volume ẩn danh([Remove anonymous volumes](https://docs.docker.com/engine/storage/volumes/#remove-anonymous-volumes)).

Nếu bạn tạo nhiều container liên tiếp, mỗi container đều sử dụng volume ẩn danh, thì mỗi container sẽ tạo một volume riêng. Volume ẩn danh không được tự động tái sử dụng hoặc chia sẻ giữa các container. Để chia sẻ một volume ẩn danh giữa hai hoặc nhiều container, bạn phải gắn volume ẩn danh bằng ID volume ngẫu nhiên.

  

Syntax

Để gắn kết một volume bằng lệnh `docker run`, bạn có thể sử dụng cờ `--mount` hoặc `--volume`.

```

docker run --mount type=volume,src=<volume-name>,dst=<mount-path>

docker run --volume <volume-name>:<mount-path>

```

Nhìn chung, nên sử dụng tùy chọn --mount. Sự khác biệt chính là cờ --mount rõ ràng hơn và hỗ trợ tất cả các tùy chọn có sẵn.

Bạn phải sử dụng --mount nếu muốn:

- Specify [](https://docs.docker.com/engine/storage/volumes/#use-a-volume-driver)[volume driver options](https://docs.docker.com/engine/storage/volumes/#use-a-volume-driver)
    
- Mount a [](https://docs.docker.com/engine/storage/volumes/#mount-a-volume-subdirectory)[volume subdirectory](https://docs.docker.com/engine/storage/volumes/#mount-a-volume-subdirectory)
    
- Mount a volume into a Swarm service
    

  
  

**VIII. Docker bind mount**

Khi sử dụng bind mount, một tệp hoặc thư mục trên máy chủ được gắn từ máy chủ vào container. Ngược lại, khi sử dụng volume, một thư mục mới được tạo trong thư mục lưu trữ của Docker trên máy chủ, và Docker quản lý nội dung của thư mục đó.

Dùng khi

- Chia sẻ mã nguồn hoặc các tệp tạo lập giữa môi trường phát triển trên máy chủ Docker và container.
    
- Khi bạn muốn tạo hoặc tạo ra các tệp trong một container và lưu trữ các tệp đó vào hệ thống tệp của máy chủ.
    
- Chia sẻ các tệp cấu hình từ máy chủ đến các container. Đây là cách Docker cung cấp khả năng phân giải DNS cho các container theo mặc định, bằng cách mount tệp /etc/resolv.conf từ máy chủ vào mỗi container.
    

Tính năng gắn kết thư mục (bind mount) cũng có sẵn cho quá trình builds: bạn có thể gắn kết mã nguồn từ máy chủ vào build container để kiểm tra, kiểm tra cú pháp hoặc biên dịch dự án.

**Bind-mounting over existing data**

Nếu bind mount tệp hoặc thư mục vào một thư mục trong container mà các tệp hoặc thư mục đó đã tồn tại, thì các tệp đã tồn tại trước đó sẽ bị che khuất bởi thao tác mount. Điều này tương tự như việc bạn lưu các tệp vào thư mục /mnt trên máy chủ Linux, rồi gắn ổ USB vào /mnt. Nội dung của /mnt sẽ bị che khuất bởi nội dung của ổ USB cho đến khi ổ USB được tháo ra.

Với container, không có cách nào đơn giản để gỡ bỏ điểm mount nhằm hiển thị lại các tập tin bị ẩn. Lựa chọn tốt nhất của bạn là tạo lại container mà không có điểm mount đó.

**Considerations and constraints**

Theo mặc định, các mount liên kết có quyền ghi vào các tệp trên máy chủ.

Một tác dụng phụ của việc sử dụng bind mount là bạn có thể thay đổi hệ thống tập tin của máy chủ thông qua các tiến trình chạy trong container, bao gồm việc tạo, sửa đổi hoặc xóa các tập tin hoặc thư mục hệ thống quan trọng. Khả năng này có thể gây ra những vấn đề về bảo mật. Ví dụ, nó có thể ảnh hưởng đến các tiến trình không phải Docker trên hệ thống máy chủ.

Bạn có thể sử dụng tùy chọn `readonly` hoặc `ro` để ngăn container ghi dữ liệu vào thư mục được mount.

Các điểm gắn kết (bind mount) được tạo trên máy chủ Docker daemon, chứ không phải trên máy khách - client.

Nếu bạn đang sử dụng remote daemon Docker, bạn không thể tạo blind mount để truy cập các tệp trên máy khách-client trong container.

Đối với Docker Desktop, tiến trình nền (daemon) chạy bên trong máy ảo Linux, chứ không phải trực tiếp trên máy chủ gốc. Docker Desktop có các cơ chế tích hợp sẵn giúp xử lý việc mount thư mục một cách minh bạch, cho phép bạn chia sẻ đường dẫn hệ thống tệp của máy chủ gốc với các container chạy trong máy ảo.

Các container có bind mount được liên kết chặt chẽ với máy chủ.

Việc gắn kết thư mục (bind mount) phụ thuộc vào việc hệ thống tệp của máy chủ có cấu trúc thư mục cụ thể hay không. Điều này có nghĩa là các container sử dụng gắn kết thư mục có thể bị lỗi nếu chạy trên một máy chủ khác không có cùng cấu trúc thư mục.

**Syntax**

Để tạo liên kết gắn kết (bind mount), bạn có thể sử dụng cờ --mount hoặc --volume.

```

docker run --mount type=bind,src=<host-path>,dst=<container-path>

docker run --volume <host-path>:<container-path>

```

Nhìn chung, nên sử dụng tùy chọn --mount. Sự khác biệt chính là cờ --mount rõ ràng hơn và hỗ trợ tất cả các tùy chọn có sẵn.

Nếu bạn sử dụng tham số --volume để mount một tệp hoặc thư mục chưa tồn tại trên máy chủ Docker, Docker sẽ tự động tạo thư mục đó trên máy chủ cho bạn. Thư mục này luôn được tạo dưới dạng một thư mục con.

Tùy chọn --mount không tự động tạo thư mục nếu đường dẫn gắn kết được chỉ định không tồn tại trên máy chủ. Thay vào đó, nó sẽ báo lỗi:

```

docker run --mount type=bind,src=/dev/noexist,dst=/mnt/foo alpine

docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /dev/noexist.

```

Options for --mount

Cờ --mount bao gồm nhiều cặp key-value, được phân tách bằng dấu phẩy và mỗi cặp gồm một bộ <key>=<value>. Thứ tự của các khóa không quan trọng.

docker run --mount type=bind,src=<host-path>,dst=<container-path>[,<key>=<value>...]

Các tùy chọn hợp lệ cho --mount type=bind bao gồm:

  

|   |   |
|---|---|
 
|source, src|Vị trí của tệp hoặc thư mục trên máy chủ. Đây có thể là đường dẫn tuyệt đối hoặc tương đối.|
|destination, dst, target|Đường dẫn nơi tệp hoặc thư mục được mount trong container. Phải là đường dẫn tuyệt đối.|
|readonly, ro|Nếu có, tùy chọn này sẽ khiến thư mục được mount vào container ở chế độ chỉ đọc.|
|bind-propagation|Nếu có, nó sẽ thay đổi quá trình bind-propagation.|

  

Ví dụ:

```

docker run --mount type=bind,src=.,dst=/project,ro,bind-propagation=rshared

```

Options for --volume

Cờ --volume hoặc -v bao gồm ba trường, được phân tách bằng dấu hai chấm (:). Các trường phải được sắp xếp đúng thứ tự.

```

docker run -v <host-path>:<container-path>[:opts]

```

Trường đầu tiên là đường dẫn trên máy chủ để mount vào container. Trường thứ hai là đường dẫn nơi tệp hoặc thư mục được mount trong container.

Trường thứ ba là tùy chọn, và là một danh sách các tùy chọn được phân tách bằng dấu phẩy. Các tùy chọn hợp lệ cho --volume với bind mount bao gồm:

![](file:///tmp/lu599412u78ad.tmp/lu599412u78ap_tmp_6ec8fa4ff98c3419.png)

[https://docs.docker.com/engine/storage/bind-mounts/#considerations-and-constraints](https://docs.docker.com/engine/storage/bind-mounts/#considerations-and-constraints)

  

Ví dụ:

```

docker run -v .:/project:ro,rshared

```

tmpfs mounts

Volumes và bind mounts cho phép bạn chia sẻ tập tin giữa máy chủ và container để dữ liệu có thể được lưu giữ ngay cả sau khi container dừng hoạt động.

Nếu bạn đang chạy Docker trên Linux, bạn có thêm một lựa chọn thứ ba: tmpfs mount. Khi bạn tạo một container với tmpfs mount, container đó có thể tạo các tệp bên ngoài lớp ghi được của container(container's writable layer).

Khác với các volume và bind mount, tmpfs mount chỉ là tạm thời và chỉ được lưu trữ trong bộ nhớ của máy chủ. Khi container dừng hoạt động, tmpfs mount sẽ bị xóa và các tệp được ghi vào đó sẽ không được lưu giữ.

Việc sử dụng tmpfs (TPF) là lựa chọn tốt nhất trong trường hợp bạn không muốn dữ liệu được lưu trữ lâu dài trên máy chủ hoặc bên trong container. Điều này có thể vì lý do bảo mật hoặc để bảo vệ hiệu năng của container khi ứng dụng của bạn cần ghi một lượng lớn dữ liệu trạng thái không được lưu trữ lâu dài.

Lưu ý:

Các điểm tmpfs mount trong Docker ánh xạ trực tiếp đến tmpfs trong nhân Linux. Do đó, dữ liệu tạm thời có thể được ghi vào tệp hoán đổi (swap file), và nhờ vậy được lưu trữ lâu dài trên hệ thống tệp.

  

Mounting over existing data

Nếu bạn tạo một mount tmpfs vào một thư mục trong container mà trong đó đã có các tệp hoặc thư mục, thì các tệp đã tồn tại trước đó sẽ bị che khuất bởi mount này. Điều này tương tự như việc bạn lưu các tệp vào /mnt trên máy chủ Linux, rồi sau đó gắn ổ USB vào /mnt. Nội dung của /mnt sẽ bị che khuất bởi nội dung của ổ USB cho đến khi ổ USB được tháo ra.

Với container, không có cách nào đơn giản để gỡ bỏ điểm mount nhằm hiển thị lại các tập tin bị ẩn. Lựa chọn tốt nhất của bạn là tạo lại container mà không có điểm mount đó.

Limitations of tmpfs mounts

Khác với volumes và bind mounts, bạn không thể chia sẻ các mount tmpfs giữa các container.

Chức năng này chỉ khả dụng nếu bạn đang chạy Docker trên Linux.

Việc thiết lập quyền trên tmpfs có thể khiến chúng bị đặt lại sau khi khởi động lại container. Trong một số trường hợp, việc thiết lập uid/gid có thể là giải pháp tạm thời.

Syntax

Để gắn kết tmpfs bằng lệnh docker run, bạn có thể sử dụng cờ --mount hoặc --tmpfs.

```

docker run --mount type=tmpfs,dst=<mount-path>

docker run --tmpfs <mount-path>
```

Xem thêm: [https://docs.docker.com/engine/storage/tmpfs/](https://docs.docker.com/engine/storage/tmpfs/)

  

**containerd image store with Docker Engine**

  

[**https://docs.docker.com/engine/storage/containerd/**](https://docs.docker.com/engine/storage/containerd/)

  

  

**IX. Docker Engine**

Docker Engine là môi trường chạy container tiêu chuẩn trong ngành, hoạt động trên nhiều hệ điều hành Linux (CentOS, Debian, Fedora, RHEL và Ubuntu) và Windows Server. Docker tạo ra các công cụ đơn giản và phương pháp đóng gói phổ quát, gom tất cả các phụ thuộc của ứng dụng vào trong một container, sau đó được chạy trên Docker Engine. Docker Engine cho phép các ứng dụng được đóng gói trong container chạy ổn định ở bất cứ đâu trên bất kỳ cơ sở hạ tầng nào, giải quyết vấn đề "địa ngục phụ thuộc" cho các nhà phát triển và nhóm vận hành, đồng thời loại bỏ vấn đề "it works on my laptop!".

  

**X. OverlayFS storage driver**

OverlayFS is a union filesystem.

Trang này gọi Linux kernel driver là OverlayFS và Docker storage driver là overlay2.

Lưu ý:

Docker Engine 29.0 trở lên sử dụng kho lưu trữ containerd image ([containerd image store](https://docs.docker.com/engine/storage/containerd/)) theo mặc định. Trình điều khiển overlay2 là trình điều khiển lưu trữ cũ đã được thay thế bởi overlayfs containerd snapshotter. Để biết thêm thông tin, hãy xem [Select a storage driver](https://docs.docker.com/engine/storage/drivers/select-storage-driver/).

For fuse-overlayfs driver, check [Rootless mode documentation](https://docs.docker.com/engine/security/rootless/).

Prerequisites

Trình điều khiển overlay2 được hỗ trợ nếu bạn đáp ứng các điều kiện tiên quyết sau:

- Phiên bản 4.0 trở lên của nhân Linux, hoặc RHEL hoặc CentOS sử dụng phiên bản 3.10.0-514 của nhân trở lên.
    
- Trình điều khiển overlay2 được hỗ trợ trên hệ thống tệp tin dựa trên xfs, nhưng chỉ khi d_type=true được bật.
    
- Sử dụng lệnh xfs_info để kiểm tra xem tùy chọn ftype có được đặt thành 1 hay không. Để định dạng hệ thống tệp xfs đúng cách, hãy sử dụng cờ -n ftype=1.
    

  

Việc thay đổi trình điều khiển lưu trữ(storage driver) sẽ khiến các container và image hiện có không thể truy cập được trên hệ thống cục bộ. Hãy sử dụng lệnh `docker save` để lưu bất kỳ image nào bạn đã xây dựng hoặc đẩy chúng lên Docker Hub hoặc một registry riêng trước khi thay đổi trình điều khiển lưu trữ(storage driver), để bạn không cần phải tạo lại chúng sau này.

  

**How the overlay2 driver works**

OverlayFS xếp chồng hai thư mục trên cùng một máy chủ Linux và hiển thị chúng như một thư mục duy nhất. Các thư mục này được gọi là các lớp (layers), và quá trình hợp nhất được gọi là gắn kết hợp nhất (union mount). OverlayFS gọi thư mục phía dưới là lowerdir và thư mục phía trên là upperdir. Chế độ xem hợp nhất được hiển thị thông qua thư mục riêng của nó có tên là merged.

Trình điều khiển overlay2 hỗ trợ tối đa 128 lower OverlayFS layers. Khả năng này mang lại hiệu suất tốt hơn cho các lệnh Docker liên quan đến lớp như docker build và docker commit, đồng thời tiêu thụ ít inode hơn trên hệ thống tệp hỗ trợ.

  

Xem cấu trúc thư mục OFS: [https://docs.docker.com/engine/storage/drivers/overlayfs-driver/](https://docs.docker.com/engine/storage/drivers/overlayfs-driver/)

  

**OverlayFS and Docker Performance**

Overlay2 có thể hoạt động tốt hơn btrfs. Tuy nhiên, cần lưu ý những chi tiết sau:

- Page caching: OverlayFS hỗ trợ page cache sharing. Nhiều container truy cập cùng một tệp sẽ chia sẻ một mục nhập page cache duy nhất cho tệp đó. Điều này giúp trình điều khiển overlay2 tiết kiệm bộ nhớ và là một lựa chọn tốt cho các trường hợp sử dụng mật độ cao như PaaS.
    
- Copyup: Cũng giống như các hệ thống tệp copy-on-write khác, OverlayFS thực hiện các thao tác sao chép lên (copy-up) mỗi khi một container ghi vào tệp lần đầu tiên. Điều này có thể làm tăng độ trễ trong thao tác ghi, đặc biệt đối với các tệp lớn. Tuy nhiên, một khi tệp đã được sao chép lên, tất cả các thao tác ghi tiếp theo vào tệp đó sẽ diễn ra ở lớp trên, mà không cần thực hiện thêm các thao tác sao chép lên nữa.
    
- Performance best practices: Các nguyên tắc thực hành tốt nhất về hiệu suất chung sau đây áp dụng cho OverlayFS.
    
    - Use fast storage: Ổ cứng thể rắn (SSD) cung cấp tốc độ đọc và ghi nhanh hơn so với ổ đĩa cơ học.
        
    - Use volumes for write-heavy workloads: Volumes cung cấp hiệu năng tốt nhất và ổn định nhất cho các tác vụ ghi dữ liệu nhiều. Điều này là do chúng bỏ qua trình storage driver và không phát sinh bất kỳ chi phí tiềm tàng nào do cấp phát mỏng (thin provisioning) và sao chép khi ghi (copy-on-write) gây ra. Volumes còn có những lợi ích khác, chẳng hạn như cho phép bạn chia sẻ dữ liệu giữa các container và duy trì dữ liệu ngay cả khi không có container nào đang chạy sử dụng chúng.
        

**Limitations on OverlayFS compatibility**

open(2): OverlayFS chỉ triển khai một tập hợp con của các tiêu chuẩn POSIX. Điều này có thể dẫn đến một số thao tác của OverlayFS vi phạm các tiêu chuẩn POSIX. Một trong những thao tác đó là thao tác sao chép lên (copy-up). Giả sử ứng dụng của bạn gọi fd1=open("foo", O_RDONLY) và sau đó fd2=open("foo", O_RDWR). Trong trường hợp này, ứng dụng của bạn mong đợi fd1 và fd2 tham chiếu đến cùng một tệp. Tuy nhiên, do một thao tác sao chép lên xảy ra sau lần gọi thứ hai đến open(2), các mô tả tham chiếu đến các tệp khác nhau. fd1 tiếp tục tham chiếu đến tệp trong image(lowerdir) và fd2 tham chiếu đến tệp trong container (upperdir). Một giải pháp khắc phục cho vấn đề này là chạm vào các tệp, điều này gây ra thao tác sao chép lên. Tất cả các thao tác open(2) tiếp theo, bất kể chế độ truy cập chỉ đọc hay đọc-ghi, đều tham chiếu đến tệp trong container (upperdir).

Lệnh yum được biết là sẽ bị ảnh hưởng trừ khi gói yum-plugin-ovl được cài đặt. Nếu gói yum-plugin-ovl không có sẵn trong bản phân phối của bạn, chẳng hạn như RHEL/CentOS trước phiên bản 6.8 hoặc 7.2, bạn có thể cần chạy lệnh `touch /var/lib/rpm/*` trước khi chạy `yum install`. Gói này thực hiện giải pháp khắc phục sự cố bằng lệnh `touch` được đề cập ở trên cho lệnh yum.

rename(2): OverlayFS không hỗ trợ đầy đủ lệnh gọi hệ thống rename(2). Ứng dụng của bạn cần phát hiện lỗi và chuyển sang chiến lược "sao chép và hủy liên kết".