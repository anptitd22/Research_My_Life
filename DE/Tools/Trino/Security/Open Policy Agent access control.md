Plugin kiểm soát truy cập Open Policy Agent cho phép sử dụng [Open Policy Agent (OPA)](https://www.openpolicyagent.org/) làm công cụ ủy quyền để kiểm soát truy cập chi tiết vào các catalogs, schemas, tables và nhiều hơn nữa trong Trino. Các chính sách được định nghĩa trong OPA, và Trino kiểm tra các đặc quyền kiểm soát truy cập trong OPA.

## Requirements

- Triển khai [OPA deployment](https://www.openpolicyagent.org/docs/latest/#running-opa)
- Kết nối mạng từ cụm Trino đến máy chủ OPA

Sau khi đáp ứng đầy đủ các yêu cầu, bạn có thể tiến hành thiết lập Trino và OPA với cấu hình kiểm soát truy cập mong muốn.

## Trino configuration


Để chỉ sử dụng OPA cho việc kiểm soát truy cập, hãy tạo tệp etc/access-control.properties với cấu hình tối thiểu sau.

```config
access-control.name=opa
opa.policy.uri=https://opa.example.com/v1/data/trino/allow
```

Để kết hợp kiểm soát truy cập OPA với các hệ thống kiểm soát truy cập dựa trên tệp (file-based) hoặc các hệ thống kiểm soát truy cập khác, hãy làm theo hướng dẫn về [Multiple access control systems](https://trino.io/docs/current/security/built-in-system-access-control.html#multiple-access-control).

Bảng sau liệt kê các thuộc tính cấu hình cho việc kiểm soát truy cập OPA.

 OPA access control configuration properties[#](https://trino.io/docs/current/security/opa-access-control.html#id1 "Link to this table")

|                                              |                                                                                                                                                                                                                                                                                                |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `opa.policy.uri`                             | The **required** URI for the OPA endpoint, for example, `https://opa.example.com/v1/data/trino/allow`.                                                                                                                                                                                         |
| `opa.policy.row-filters-uri`                 | The **optional** URI for fetching row filters - if not set no row filtering is applied. For example, `https://opa.example.com/v1/data/trino/rowFilters`.                                                                                                                                       |
| `opa.policy.column-masking-uri`              | The **optional** URI for fetching column masks - if not set no masking is applied. For example, `https://opa.example.com/v1/data/trino/columnMask`.                                                                                                                                            |
| `opa.policy.batch-column-masking-uri`        | The **optional** URI for fetching columns masks in batches; must **not** be used with `opa.policy.column-masking-uri`. For example, `http://opa.example.com/v1/data/trino/batchColumnMasks`.                                                                                                   |
| `opa.policy.batched-uri`                     | The **optional** URI for activating batch mode for certain authorization queries where batching is applicable, for example `https://opa.example.com/v1/data/trino/batch`. Batch mode is described [Batch mode](https://trino.io/docs/current/security/opa-access-control.html#opa-batch-mode). |
| `opa.log-requests`                           | Configure if request details, including URI, headers and the entire body, are logged prior to sending them to OPA. Defaults to `false`.                                                                                                                                                        |
| `opa.log-responses`                          | Configure if OPA response details, including URI, status code, headers and the entire body, are logged. Defaults to `false`.                                                                                                                                                                   |
| `opa.allow-permission-management-operations` | Configure if permission management operations are allowed. Find more details in [Permission management](https://trino.io/docs/current/security/opa-access-control.html#opa-permission-management). Defaults to `false`.                                                                        |
| `opa.http-client.*`                          | Optional HTTP client configurations for the connection from Trino to OPA, for example `opa.http-client.http-proxy` for configuring the HTTP proxy. Find more details in [HTTP client properties](https://trino.io/docs/current/admin/properties-http-client.html).                             |
| `opa.context-file`                           | Optional properties file, containing user defined properties (e.g. tenant namespace, tier or cluster) to be included in the OPA query context.                                                                                                                                                 |

### Logging

Khi tính năng ghi nhật ký yêu cầu hoặc phản hồi được bật, thông tin chi tiết sẽ được ghi lại ở cấp độ `DEBUG` trong trình ghi nhật ký` io.trino.plugin.opa.OpaHttpClient`. Cấu hình ghi nhật ký của Trino phải được cập nhật để bao gồm class này, nhằm đảm bảo các mục nhật ký được tạo.

Lưu ý rằng việc bật các tùy chọn này sẽ tạo ra lượng dữ liệu nhật ký rất lớn.

### Permission management

Các thao tác sau được cho phép hoặc bị từ chối dựa trên thiết lập của `opa.allow-permission-management-operations`. Nếu được đặt thành `true`, các thao tác này được cho phép. Nếu được đặt thành `false`, chúng bị từ chối. Trong cả hai trường hợp, không có yêu cầu nào được gửi đến OPA.

- `GrantSchemaPrivilege`
- `DenySchemaPrivilege`
- `RevokeSchemaPrivilege`
- `GrantTablePrivilege`
- `DenyTablePrivilege`
- `RevokeTablePrivilege`
- `CreateRole`
- `DropRole`
- `GrantRoles`
- `RevokeRoles`

Cài đặt mặc định là false do tính phức tạp và những hậu quả không lường trước được khi sử dụng các quyền và vai trò kiểu SQL cùng với OPA.

Bạn phải bật tính năng quản lý quyền nếu hệ thống bảo mật tùy chỉnh khác trong Trino có khả năng quản lý quyền và được sử dụng cùng với tính năng kiểm soát truy cập OPA.

Ngoài ra, người dùng luôn được phép hiển thị thông tin về vai trò (SHOW ROLES), bất kể cài đặt này. Các thao tác sau đây luôn được cho phép.

- `ShowRoles`
- `ShowCurrentRoles`
- `ShowRoleGrants`

## OPA configuration

Cơ chế kiểm soát truy cập OPA trong Trino sẽ liên hệ với OPA cho mỗi truy vấn và đưa ra yêu cầu ủy quyền. OPA phải trả về phản hồi chứa trường cho phép kiểu boolean, xác định xem thao tác đó có được cho phép hay không.

Các chính sách trong OPA được định nghĩa bằng ngôn ngữ chính sách chuyên dụng Rego. Tìm thêm thông tin trong [detailed documentation](https://www.openpolicyagent.org/docs/latest/policy-language/). Sau khi cài đặt và cấu hình ban đầu trong Trino, các chính sách này là khía cạnh cấu hình chính cho thiết lập kiểm soát truy cập của bạn.

Một truy vấn từ cơ chế kiểm soát truy cập OPA trong Trino tới OPA chứa `context` và `action` làm các trường cấp cao nhất.

Đối tượng`context`  chứa tất cả thông tin ngữ cảnh khác về truy vấn.

- `identity`: Thông tin nhận dạng của người dùng thực hiện thao tác, bao gồm hai trường sau:
	- `user`: username
	- `groups`: danh sách các nhóm mà người dùng này thuộc về
- `queryId`: Query id
- `softwareStack`: Thông tin về ngăn xếp phần mềm gửi yêu cầu đến OPA. Các thông tin sau đây được bao gồm:
	- `trinoVersion`: Version of Trino used

Đối tượng `action` chứa thông tin về hành động nào được thực hiện trên tài nguyên nào. Các trường sau được cung cấp.

- `operation`: Thao tác được thực hiện, ví dụ: `SelectFromColumns`
- `resource`: thông tin về các đối tượng được truy cập
- `targetResource`: thông tin về bất kỳ đối tượng mới nào được tạo ra, nếu có.
- `grantee`: người nhận tài trợ của một hoạt động tài trợ

### Example requests to OPA

Việc truy cập vào một bảng sẽ tạo ra một truy vấn tương tự như ví dụ sau.

```
{
  "context": {
    "identity": {
      "user": "foo",
      "groups": ["some-group"]
    },
    "queryId": "20250718_081710_03427_trino",
    "softwareStack": {
      "trinoVersion": "434"
    }
  },
  "action": {
    "operation": "SelectFromColumns",
    "resource": {
      "table": {
        "catalogName": "example_catalog",
        "schemaName": "example_schema",
        "tableName": "example_table",
        "columns": [
          "column1",
          "column2",
          "column3"
        ]
      }
    }
  }
}
```

Tham số `targetResource` được sử dụng trong trường hợp tạo một tài nguyên mới, khác với tài nguyên hiện có. Ví dụ, khi đổi tên một bảng.

```
{
  "context": {
    "identity": {
      "user": "foo",
      "groups": ["some-group"]
    },
    "queryId": "20250718_081710_03427_trino",
    "softwareStack": {
      "trinoVersion": "434"
    }
  },
  "action": {
    "operation": "RenameTable",
    "resource": {
      "table": {
        "catalogName": "example_catalog",
        "schemaName": "example_schema",
        "tableName": "example_table"
      }
    },
    "targetResource": {
      "table": {
        "catalogName": "example_catalog",
        "schemaName": "example_schema",
        "tableName": "new_table_name"
      }
    }
  }
}
```

## Row filtering

Tính năng lọc hàng cho phép Trino loại bỏ một số hàng khỏi kết quả trước khi trả về cho người gọi, kiểm soát dữ liệu mà người dùng khác nhau có thể xem. Plugin hỗ trợ truy xuất định nghĩa bộ lọc từ OPA bằng cách cấu hình điểm cuối OPA cho quá trình xử lý lọc hàng với `opa.policy.row-filters-uri`

```rego
  package trino
  import future.keywords.in
  import future.keywords.if
  import future.keywords.contains

  default allow := true

  table_resource := input.action.resource.table
  is_admin {
    input.context.identity.user == "admin"
  }

  rowFilters contains {"expression": "user_type <> 'customer'"} if {
      not is_admin
      table_resource.catalogName == "sample_catalog"
      table_resource.schemaName == "sample_schema"
      table_resource.tableName == "restricted_table"
  }
```

Phản hồi mà plugin mong đợi là một mảng các đối tượng, mỗi đối tượng có định dạng {"expression":"clause"}. Mỗi biểu thức về cơ bản hoạt động như một mệnh đề `WHERE` bổ sung. Tập lệnh cũng có thể trả về nhiều bộ lọc hàng cho một yêu cầu OPA duy nhất, và tất cả các bộ lọc sau đó được áp dụng.

Mỗi đối tượng có thể chứa một trường định danh (identity). Trường định danh cho phép Trino đánh giá các bộ lọc hàng này theo một định danh khác, sao cho bộ lọc có thể nhắm mục tiêu vào một cột mà người dùng yêu cầu không thể nhìn thấy.

## Column masking

### Batch column masking

## Batch mode