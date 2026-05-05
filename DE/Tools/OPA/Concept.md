## What is OPA?

OPA tách biệt việc ra quyết định chính sách khỏi việc thực thi chính sách. Khi phần mềm của bạn cần đưa ra quyết định về chính sách, nó sẽ truy vấn OPA và cung cấp dữ liệu có cấu trúc (ví dụ: JSON) làm đầu vào. OPA chấp nhận dữ liệu có cấu trúc bất kỳ làm đầu vào.

![alt][images/OPA/opa_concept1.png]

OPA đưa ra các quyết định chính sách bằng cách đánh giá đầu vào truy vấn so với các chính sách và dữ liệu. OPA và Rego không phụ thuộc vào lĩnh vực cụ thể, vì vậy bạn có thể mô tả hầu hết mọi loại bất biến trong các chính sách của mình. Ví dụ:
- Người dùng nào có thể truy cập tài nguyên nào. 
- Những subnets nào cho phép lưu lượng truy cập đi ra? 
- Khối lượng công việc cần được triển khai đến những cụm máy chủ nào? 
- Các tệp nhị phân có thể được tải xuống từ những kho lưu trữ nào? 
- Các khả năng hệ điều hành nào mà container có thể sử dụng để thực thi? 
- Hệ thống có thể truy cập được vào những khung giờ nào trong ngày?

Các quyết định chính sách không chỉ giới hạn ở các câu trả lời đơn giản có/không hoặc cho phép/từ chối. Giống như dữ liệu đầu vào của truy vấn, các chính sách của bạn có thể tạo ra dữ liệu có cấu trúc tùy ý làm đầu ra.

Hãy xem một ví dụ. Giả sử bạn làm việc cho một tổ chức có cơ sở hạ tầng mạng như sau:

![alt][images/OPA/opa_concept2.png]

Hệ thống này bao gồm ba loại linh kiện: 
- Máy chủ lắng nghe trên một loạt các cổng (cho các giao thức khác nhau như http, ssh, v.v.) 
- Mạng lưới kết nối các máy chủ và có thể là mạng công cộng hoặc mạng riêng. Mạng công cộng được kết nối với Internet. 
- Cổng kết nối máy chủ với mạng.

Tất cả các máy chủ, mạng và cổng đều được cấu hình bằng cách sử dụng cơ sở hạ tầng dưới dạng mã. Cấu hình cơ sở hạ tầng được chỉ định trong định dạng JSON sau:

```rego
{
  "servers": [
    { "id": "app", "protocols": ["https", "ssh"], "ports": ["p1", "p2", "p3"] },
    { "id": "db", "protocols": ["mysql"], "ports": ["p3"] },
    { "id": "cache", "protocols": ["memcache"], "ports": ["p3"] },
    { "id": "ci", "protocols": ["http"], "ports": ["p1", "p2"] },
    { "id": "busybox", "protocols": ["telnet"], "ports": ["p1"] }
  ],
  "networks": [
    { "id": "net1", "public": false },
    { "id": "net2", "public": false },
    { "id": "net3", "public": true },
    { "id": "net4", "public": true }
  ],
  "ports": [
    { "id": "p1", "network": "net1" },
    { "id": "p2", "network": "net3" },
    { "id": "p3", "network": "net2" }
  ]
}
```

Tổ chức của bạn đã thiết lập chính sách bảo mật sau đây và cần phải thực hiện chính sách này:

- Các máy chủ có thể truy cập từ Internet không được phép để lộ giao thức 'http' không an toàn.
- Máy chủ không được phép để lộ giao thức 'telnet'.

Chính sách này cần được thực thi khi cấp phát máy chủ, mạng và cổng, và nhóm tuân thủ muốn định kỳ kiểm tra hệ thống để tìm ra các máy chủ vi phạm chính sách.

Hãy cùng tìm hiểu xem OPA có thể giúp thực hiện chính sách này như thế nào.
## Writing Policy with Rego

Các chính sách OPA được thể hiện bằng một ngôn ngữ khai báo cấp cao gọi là Rego. Rego (phát âm là "ray-go") được thiết kế đặc biệt để thể hiện các chính sách trên các cấu trúc dữ liệu phân cấp phức tạp. Để biết thông tin chi tiết về Rego, hãy xem tài liệu về [Policy Language](https://www.openpolicyagent.org/docs/policy-language).

>[!tip]
>Các ví dụ bên dưới có tính tương tác! Nếu bạn chỉnh sửa dữ liệu đầu vào ở trên chứa máy chủ, mạng và cổng, kết quả đầu ra bên dưới sẽ thay đổi. Tương tự, nếu bạn chỉnh sửa các truy vấn hoặc quy tắc trong các ví dụ bên dưới, kết quả đầu ra cũng sẽ thay đổi. Khi đọc phần này, hãy thử thay đổi dữ liệu đầu vào, truy vấn và quy tắc và quan sát sự khác biệt trong kết quả đầu ra.
>Chúng cũng có thể được chạy cục bộ trên máy tính của bạn bằng cách sử dụng [`opa eval` command, here are setup instructions.](https://www.openpolicyagent.org/docs#running-opa)

### Basic Syntax

Để thực hiện chính sách bảo mật của chúng tôi, trước tiên chúng ta cần truy cập và kiểm tra dữ liệu cơ sở hạ tầng. Khi OPA đánh giá các chính sách, nó sẽ liên kết dữ liệu được cung cấp trong truy vấn với một biến toàn cục có tên là input. Bạn có thể tham chiếu đến các phần cụ thể của dữ liệu input bằng cách sử dụng toán tử . (dấu chấm).

```rego
package servers

output := input.servers
```

```
[
  {
    "id": "app",
    "ports": [
      "p1",
      "p2",
      "p3"
    ],
    "protocols": [
      "https",
      "ssh"
    ]
  },
  {
    "id": "db",
    "ports": [
      "p3"
    ],
    "protocols": [
      "mysql"
    ]
  },
  {
    "id": "cache",
    "ports": [
      "p3"
    ],
    "protocols": [
      "memcache"
    ]
  },
  {
    "id": "ci",
    "ports": [
      "p1",
      "p2"
    ],
    "protocols": [
      "http"
    ]
  },
  {
    "id": "busybox",
    "ports": [
      "p1"
    ],
    "protocols": [
      "telnet"
    ]
  }
]
```

Để tham chiếu đến các phần tử mảng, bạn có thể sử dụng cú pháp dấu ngoặc vuông quen thuộc:

```rego
package servers

output := input.servers[0].protocols[0]
```

```
"https"
```

>[!tip]
>Bạn có thể sử dụng cùng cú pháp dấu ngoặc vuông nếu các khóa chứa các ký tự khác ngoài [a-zA-Z0-9_]. Ví dụ: input["foo~bar"].

Nếu bạn tham chiếu đến một giá trị không tồn tại, OPA sẽ trả về undefined. Undefined có nghĩa là OPA không thể tìm thấy bất kỳ kết quả nào.

```rego
package servers

output := input.foobar
```

```
undefined
```

Các quyết định chính sách đơn giản nhất được đưa ra bằng cách viết các biểu thức thực hiện các phép toán logic trên dữ liệu đầu vào. Ví dụ, chúng ta có thể kiểm tra xem một máy chủ có ID cụ thể hay không bằng cách sử dụng phép kiểm tra bằng với toán tử `==`.

```rego
package servers

output := input.servers[0].id == "app"
```

```
true
```

OPA bao gồm một tập hợp các hàm tích hợp sẵn ([built-in functions](https://www.openpolicyagent.org/docs/policy-reference/builtins)) mà bạn có thể sử dụng để thực hiện các thao tác thông thường như xử lý chuỗi, khớp biểu thức chính quy, phép toán số học, phép tổng hợp, v.v.

```rego
package servers

output := count(input.servers[0].protocols) >= 1
```

```
true
```

Để các truy vấn tạo ra kết quả, tất cả các biểu thức trong truy vấn phải đúng hoặc được định nghĩa. Bạn có thể tách các biểu thức trên nhiều dòng (hoặc tùy chọn nối chúng bằng `;` - nghĩa là phép AND, trên cùng một dòng):

```rego
package servers

output if {
    input.servers[0].id == "app"
    input.servers[0].protocols[0] == "https"
}
```

or

```rego
package servers

output if {
    input.servers[0].id == "app"; input.servers[0].protocols[0] == "https"
}
```

```
true
```

Nếu bất kỳ biểu thức nào trong truy vấn không đúng (hoặc không được định nghĩa), kết quả sẽ là không xác định. Trong ví dụ bên dưới, biểu thức thứ hai là false:

```rego
package servers

output if {
    input.servers[0].id == "app"
    input.servers[0].protocols[0] == "telnet"
}
```

```
undefined
```
#### Note
Các biểu thức chỉ được kết hợp với nhau bằng toán tử `AND` khi chúng nằm trong cùng một phần thân quy tắc. Trong ví dụ này, các kiểm tra đối với `input.servers` được kết hợp bằng toán tử `OR` vì chúng nằm trong các phần thân quy tắc khác nhau. Xem phần [Logical OR](https://www.openpolicyagent.org/docs#logical-or) trong mục Quy tắc bên dưới để biết thêm chi tiết.

```rego
output if {
    input.servers[0].id == "app"
}

output if {
    input.servers[0].protocols[0] == "telnet"
}
```

Bạn có thể lưu trữ các giá trị trong các biến trung gian bằng cách sử dụng toán tử := (phép gán) để giúp các quy tắc phức tạp dễ đọc hơn. Các biến có thể được tham chiếu giống như dữ liệu đầu vào và cũng giống như dữ liệu đầu vào, chúng không thể thay đổi.

```rego
package servers

output if {
    s := input.servers[0]
    s.id == "app"
    p := s.protocols[0]
    p == "https"
}
```

Khi OPA đánh giá các biểu thức, nó sẽ tìm các giá trị cho các biến sao cho tất cả các biểu thức đều đúng. Nếu không có phép gán biến nào làm cho tất cả các biểu thức đều đúng, kết quả sẽ không xác định.

```rego
package servers

output if {
    s := input.servers[0]
    s.id == "app"
    s.protocols[1] == "telnet"
}
```

Hãy tưởng tượng bạn cần kiểm tra xem có mạng nào là mạng công cộng hay không. Nhớ lại rằng các mạng được cung cấp bên trong một mảng:

`[{"id": "net1", "public": false}, {"id": "net2", "public": false}, ...]`

Để giải quyết vấn đề này, thoạt đầu bạn có thể nghĩ đến việc kiểm tra từng mạng riêng lẻ bằng cách kiểm tra các chỉ số mảng cụ thể như sau:

```rego
package servers

# if any are true, the result of the exists_public_network is true.

exists_public_network if input.networks[0].public == true
# or
exists_public_network if input.networks[1].public == true
# or
exists_public_network if input.networks[2].public == true
# or
exists_public_network if input.networks[3].public == true
# ...
```

Cách tiếp cận này có vấn đề, có thể có quá nhiều mạng để liệt kê tĩnh, và số lượng mạng có thể không được biết trước.

Giống như các ngôn ngữ khai báo khác (ví dụ: SQL), việc lặp trong Rego diễn ra một cách ngầm định khi bạn chèn biến vào biểu thức. Giải pháp cho trường hợp này là thay thế chỉ số mảng bằng một biến:

```rego
package servers

exists_public_network if {
    some i
    input.networks[i].public == true
}
```

Bây giờ, truy vấn yêu cầu các giá trị của i sao cho toàn bộ biểu thức trở nên đúng. Khi bạn thay thế các biến trong tham chiếu, OPA tự động tìm các phép gán biến thỏa mãn tất cả các biểu thức trong truy vấn. Giống như các biến trung gian, OPA trả về giá trị của các biến.

Bạn có thể thay thế bao nhiêu biến tùy thích để thực hiện vòng lặp lồng nhau. Ví dụ, để tìm hiểu xem có máy chủ nào sử dụng giao thức "http" không an toàn hay không, bạn có thể viết như sau:

```rego
package servers

http_server if {
    some i, j
    input.servers[i].protocols[j] == "http"
}
```

Nếu các biến xuất hiện nhiều lần, phép gán sẽ thỏa mãn tất cả các biểu thức. Ví dụ, để tìm ID của các cổng được kết nối với mạng công cộng, bạn có thể viết:

```rego
package servers

exposed_ports contains port_id if {
    some i, j
    port_id := input.ports[i].id
    input.ports[i].network == input.networks[j].id
    input.networks[j].public
}
```

Giống như các tham chiếu trỏ đến các trường không tồn tại hoặc các biểu thức không khớp, nếu OPA không thể tìm thấy bất kỳ phép gán biến nào thỏa mãn tất cả các biểu thức, thì kết quả sẽ không xác định.

```rego
package servers

ssh_server if {
    some i
    # there is no assignment of i that satisfies the expression
    input.servers[i].protocols[i] == "ssh"
}
```

#### FOR SOME and FOR ALL

Mặc dù việc lặp lại thông thường đóng vai trò là một khối xây dựng mạnh mẽ, Rego cũng cung cấp các cách để thể hiện rõ ràng hơn cụm từ "DÀNH CHO MỘT SỐ NGƯỜI" và "DÀNH CHO TẤT CẢ MỌI NGƯỜI".

##### FOR SOME (`some`)

`some ... in ...` được sử dụng để lặp lại tập hợp (đối số cuối cùng của nó), và sẽ liên kết các biến của nó (khóa, vị trí giá trị) với các mục trong tập hợp. Nó giới thiệu các liên kết mới cho việc đánh giá phần còn lại của body quy tắc.

Bằng cách sử dụng `some`, chúng ta có thể diễn đạt các quy tắc đã giới thiệu ở trên theo nhiều cách khác nhau:

```rego
package servers

public_network contains net.id if {
    some net in input.networks # some network exists and..
    net.public                 # it is public.
}

shell_accessible contains server.id if {
    some server in input.servers
    "telnet" in server.protocols
}

shell_accessible contains server.id if {
    some server in input.servers
    "ssh" in server.protocols
}
```

Chi tiết về `some ... in ...`, đọc [the documentation of the `in` operator](https://www.openpolicyagent.org/docs/policy-language#membership-and-iteration-in).

##### FOR ALL (`every`)

Mở rộng các ví dụ trên, từ "every" cho phép chúng ta diễn đạt một cách ngắn gọn rằng một điều kiện đúng với tất cả các phần tử của một miền.

```rego
{
  "servers": [
    {
      "id": "busybox",
      "protocols": ["http", "ftp"]
    },
    {
      "id": "db",
      "protocols": ["mysql", "ssh"]
    },
    {
      "id": "web",
      "protocols": ["https"]
    }
  ]
}
```

```rego
package servers

no_telnet_exposed if {
    every server in input.servers {
        not "telnet" in server.protocols
    }
}
```

Tìm hiểu thêm về [Every Keyword](https://www.openpolicyagent.org/docs/policy-language#every-keyword)

### Policy Rules

Rego cho phép bạn đóng gói và tái sử dụng logic bằng các quy tắc. Quy tắc chỉ đơn giản là các câu lệnh logic "nếu-thì". Quy tắc có thể là "complete" hoặc "partial".

#### Complete Rules

Các quy tắc hoàn chỉnh là các câu lệnh điều kiện (if-then) gán một giá trị duy nhất cho một biến. Mỗi quy tắc bao gồm `head` và `body` .

- **Rule Head**
	- Phần đầu của quy tắc là phần nằm trước phần thân của quy tắc và xác định kết quả mà quy tắc tạo ra. Nó chỉ định tên của quy tắc, giá trị của nó (nếu có) và liệu nó tạo ra một tài liệu hoàn chỉnh, một tập hợp một phần hay một đối tượng một phần. Ví dụ: allow := true (tài liệu hoàn chỉnh), users contains name (tập hợp một phần), hoặc permissions[user] := perms (đối tượng một phần). Phần đầu xác định cách quy tắc đóng góp vào cây tài liệu ảo.
- **Rule Body**
	- Phần thân của quy tắc chứa các điều kiện và biểu thức mà tất cả đều phải được đánh giá là đúng thì quy tắc mới tạo ra kết quả. Nó được đặt trong dấu ngoặc nhọn và bao gồm một hoặc nhiều biểu thức được phân tách bởi dấu chấm phẩy hoặc xuống dòng. Ví dụ: { input.user == "admin"; input.method == "GET" }. Khi OPA đánh giá một quy tắc, nó sẽ tìm kiếm các liên kết biến làm cho tất cả các biểu thức trong phần thân đều đúng.

```rego
package rules

# head
exists_public_network := true if {
    # body
    some net in input.networks # some network exists and..
    net.public                 # it is public.
}
```

Bạn có thể truy vấn giá trị được tạo ra bởi các quy tắc giống như bất kỳ giá trị nào khác (chẳng hạn như `input` hoặc các biến do bạn tự tạo):

```
package rules

exists_public_network := true if {
    some net in input.networks # some network exists and..
    net.public                 # it is public.
}

another_rule := {
    "public_networks": exists_public_network,
}
```

Tất cả các giá trị được tạo ra bởi các quy tắc có thể được truy vấn thông qua biến dữ liệu toàn cục từ các gói khác được tải vào OPA.

```rego
package another_package

yet_another_rule := {
    "public_networks": data.rules.exists_public_network,
}
```

>[!note]
>Bạn có thể truy vấn giá trị của bất kỳ quy tắc nào được tải vào OPA bằng cách tham chiếu đến nó bằng đường dẫn tuyệt đối. Đường dẫn của một quy tắc luôn có dạng: `data.<package-path>.<rule-name>`.

Nếu bạn bỏ qua phần `= <value>` trong tiêu đề quy tắc, giá trị mặc định sẽ là true. Bạn có thể viết lại ví dụ trên như sau mà không làm thay đổi ý nghĩa:

```rego
exists_public_network if {
    some net in input.networks
    net.public
}
```

Để định nghĩa hằng số, hãy bỏ qua phần thân quy tắc. Khi bạn bỏ qua phần thân quy tắc, nó sẽ mặc định là true. Vì phần thân quy tắc là true, nên phần đầu quy tắc luôn luôn đúng/được định nghĩa.

```rego
package servers

max_allowed_protocols := 5
```

Các hằng số được định nghĩa như thế này có thể được truy vấn giống như bất kỳ giá trị nào khác:

```rego
count(input.servers[0].protocols) < max_allowed_protocols
```

Nếu OPA không tìm thấy các phép gán biến thỏa mãn phần thân quy tắc, chúng ta nói rằng quy tắc đó không được định nghĩa. Ví dụ, nếu đầu vào được cung cấp cho OPA không bao gồm mạng công cộng thì exists_public_network sẽ không được định nghĩa (điều này không giống với false). Bên dưới, OPA được cung cấp một tập hợp các mạng đầu vào khác nhau (không có mạng nào là mạng công cộng):

```rego
{
  "networks": [
    { "id": "n1", "public": false },
    { "id": "n2", "public": false }
  ]
}
```

```rego
package rules

exists_public_network if {
    some net in input.networks
    net.public
}
```

#### Partial Rules

Các quy tắc một phần là các câu lệnh điều kiện (if-then) tạo ra một tập hợp các giá trị và gán tập hợp đó cho một biến. Trong ví dụ bên dưới, `public_network contains net.id` là rule head. và `some net in input.networks; net.public` là rule body. Bạn có thể truy vấn toàn bộ tập hợp các giá trị giống như bất kỳ giá trị nào khác.

```rego
package example

# head
public_network contains net.id if {
    # body
    some net in input.networks # some network exists and..
    net.public                 # it is public.
}
```

Sử dụng từ khóa `in`, chúng ta có thể dùng danh sách này để kiểm tra xem một giá trị khác có nằm trong tập hợp được định nghĩa bởi `public_network` hay không:

```rego
package example

allow if "net3" in public_network
```

Bạn cũng có thể lặp qua tập hợp các giá trị bằng cách tham chiếu đến các phần tử của tập hợp thông qua một biến:

```rego
package example

allow if {
    some net in public_network
    net == "net3"
}
```

Ngoài việc định nghĩa một phần các tập hợp, bạn cũng có thể định nghĩa một phần các cặp khóa/giá trị (hay còn gọi là đối tượng). Xem phần [Rules](https://www.openpolicyagent.org/docs/latest/policy-language/#rules) trong hướng dẫn ngôn ngữ để biết thêm thông tin.

#### Logical OR

Khi bạn kết hợp nhiều biểu thức với nhau trong một truy vấn, bạn đang thể hiện phép toán logic AND. Để thể hiện phép toán logic OR trong Rego, bạn định nghĩa nhiều quy tắc có cùng tên. Hãy xem một ví dụ.

Hãy tưởng tượng bạn muốn biết liệu có máy chủ nào cung cấp giao thức cho phép máy khách truy cập shell hay không. Để xác định điều này, bạn có thể định nghĩa một quy tắc hoàn chỉnh, trong đó khai báo shell_accessible là true nếu bất kỳ máy chủ nào cung cấp giao thức "telnet" hoặc "ssh":

```rego
{
  "servers": [
    {
      "id": "busybox",
      "protocols": ["http", "telnet"]
    },
    {
      "id": "db",
      "protocols": ["mysql", "ssh"]
    },
    {
      "id": "web",
      "protocols": ["https"]
    }
  ]
}
```

```rego
package example.logical_or

default shell_accessible := false

shell_accessible if {
    input.servers[_].protocols[_] == "telnet"
}

shell_accessible if {
    input.servers[_].protocols[_] == "ssh"
}
```

>[!note]
>Từ khóa default cho OPA biết rằng cần gán một giá trị cho biến nếu tất cả các quy tắc khác có cùng tên đều chưa được định nghĩa.

Khi bạn sử dụng toán tử logic OR với các quy tắc một phần, mỗi định nghĩa quy tắc sẽ đóng góp vào tập hợp các giá trị được gán cho biến. Ví dụ, ví dụ trên có thể được sửa đổi để tạo ra một tập hợp các máy chủ cung cấp kết nối "telnet" hoặc "ssh".

```rego
package example.logical_or

shell_accessible contains server.id if {
    server := input.servers[_]
    server.protocols[_] == "telnet"
}

shell_accessible contains server.id if {
    server := input.servers[_]
    server.protocols[_] == "ssh"
}
```

>[!note]
>Hãy xem bài đăng trên [blog post](https://www.styra.com/blog/how-to-express-or-in-rego/), nó đi sâu hơn vào chủ đề này, trình bày các phương pháp khác nhau để diễn đạt toán tử OR trong ngôn ngữ Rego thông dụng cho các trường hợp sử dụng khác nhau.
