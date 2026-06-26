- [[#System-level access control files|System-level access control files]]
	- [[#System-level access control files#Configuration|Configuration]]
	- [[#System-level access control files#Refresh|Refresh]]
	- [[#System-level access control files#Catalog, schema, and table access|Catalog, schema, and table access]]
		- [[#Catalog, schema, and table access#Visibility|Visibility]]
		- [[#Catalog, schema, and table access#Catalog rules|Catalog rules]]
		- [[#Catalog, schema, and table access#Schema rules|Schema rules]]
		- [[#Catalog, schema, and table access#Table rules|Table rules]]
		- [[#Catalog, schema, and table access#Column constraint|Column constraint]]
		- [[#Catalog, schema, and table access#Filter and mask environment|Filter and mask environment]]
		- [[#Catalog, schema, and table access#Function rules|Function rules]]

Để bảo mật quyền truy cập dữ liệu trong cụm máy chủ của bạn, bạn có thể triển khai kiểm soát truy cập dựa trên tệp, trong đó quyền truy cập vào dữ liệu và các thao tác được xác định bởi các quy tắc được khai báo trong các tệp JSON được cấu hình thủ công.

Có hai loại kiểm soát truy cập dựa trên tập tin:
- **System-level access control** sử dụng plugin kiểm soát truy cập với một tệp JSON duy nhất quy định các quy tắc ủy quyền cho toàn bộ cụm máy chủ.
- **Catalog-level access control** sử dụng các tệp JSON riêng lẻ cho mỗi catalog để kiểm soát chi tiết dữ liệu trong catalog đó, bao gồm cả quyền truy cập column-level.

## System-level access control files

Plugin kiểm soát truy cập cho phép bạn chỉ định các quy tắc ủy quyền cho cụm máy chủ trong một tệp JSON duy nhất.

### Configuration

Để sử dụng plugin kiểm soát truy cập, hãy thêm tệp `etc/access-control.properties` chứa hai thuộc tính bắt buộc: `access-control.name`, phải được đặt thành file, và `security.config-file`, phải được đặt thành vị trí của tệp cấu hình. Vị trí tệp cấu hình có thể trỏ đến ổ đĩa cục bộ hoặc đến một điểm http endpoint. Ví dụ, nếu tệp cấu hình có tên `rules.json` nằm trong thư mục `etc`, hãy thêm tệp `etc/access-control.properties` với nội dung sau:

```
access-control.name=file
security.config-file=etc/rules.json
```

Nếu cấu hình cần được tải qua http endpoint `http://trino-test/config` và được đóng gói thành một đối tượng JSON, có thể truy cập thông qua khóa dữ liệu, thì tệp `etc/access-control.properties` sẽ có dạng như sau:

```
access-control.name=file
security.config-file=http://trino-test/config
security.json-pointer=/data
```

### Refresh

Theo mặc định, khi có thay đổi đối với tệp quy tắc JSON, Trino phải được khởi động lại để tải các thay đổi. Có một thuộc tính tùy chọn để làm mới các thuộc tính mà không cần khởi động lại Trino. Chu kỳ làm mới được chỉ định trong tệp `etc/access-control.properties`:

```
security.refresh-period=1s
```

### Catalog, schema, and table access

Quyền truy cập vào catalog, schema, bảng và chế độ xem được kiểm soát bởi các quy tắc catalog, schema và bảng. Các quy tắc catalog là các quy tắc tổng quát được sử dụng để hạn chế tất cả quyền truy cập hoặc quyền ghi vào catalog. Chúng không cấp rõ ràng bất kỳ quyền cụ thể nào cho schema hoặc bảng. Các quy tắc bảng và schema được sử dụng để chỉ định ai có thể tạo, xóa, sửa đổi, chọn, chèn, loại bỏ, v.v. đối với schema và bảng.

Đối với mỗi bộ quy tắc, quyền truy cập được dựa trên quy tắc phù hợp đầu tiên, được đọc từ trên xuống dưới trong tệp cấu hình. Nếu không có quy tắc nào phù hợp, quyền truy cập sẽ bị từ chối.

Nếu không có quy tắc nào được cung cấp, thì quyền truy cập sẽ được cấp. Bạn có thể thu hồi quyền truy cập bằng cách thêm một phần với một tập hợp quy tắc trống ở cấp độ cụ thể đó, ví dụ:

```json
{
  "schemas": []
}
```

Ở cấp độ catalog bạn phải thêm một quy tắc "giả" duy nhất cho mỗi catalog có thể truy cập được.

>[!note]
>Các quy tắc này không áp dụng cho các bảng do hệ thống định nghĩa trong schema `information_schema`.

Bảng sau đây tóm tắt các quyền cần thiết cho mỗi lệnh SQL:

|SQL command|Catalog|Schema|Table|Note|
|---|---|---|---|---|
|SHOW CATALOGS||||Always allowed|
|SHOW SCHEMAS|read-only|any*|any*|Allowed if catalog is [visible](https://trino-io.translate.goog/docs/current/security/file-system-access-control.html?_x_tr_sl=en&_x_tr_tl=vi&_x_tr_hl=vi&_x_tr_pto=tc#system-file-auth-visibility)|
|SHOW TABLES|read-only|any*|any*|Allowed if schema [visible](https://trino-io.translate.goog/docs/current/security/file-system-access-control.html?_x_tr_sl=en&_x_tr_tl=vi&_x_tr_hl=vi&_x_tr_pto=tc#system-file-auth-visibility)|
|CREATE SCHEMA|read-only|owner|||
|DROP SCHEMA|all|owner|||
|SHOW CREATE SCHEMA|all|owner|||
|ALTER SCHEMA … RENAME TO|all|owner*||Ownership is required on both old and new schemas|
|ALTER SCHEMA … SET AUTHORIZATION|all|owner|||
|CREATE TABLE|all||owner||
|DROP TABLE|all||owner||
|ALTER TABLE … RENAME TO|all||owner*|Ownership is required on both old and new tables|
|ALTER TABLE … SET PROPERTIES|all||owner||
|CREATE VIEW|all||owner||
|DROP VIEW|all||owner||
|ALTER VIEW … RENAME TO|all||owner*|Ownership is required on both old and new views|
|REFRESH MATERIALIZED VIEW|all||update||
|COMMENT ON TABLE|all||owner||
|COMMENT ON COLUMN|all||owner||
|ALTER TABLE … ADD COLUMN|all||owner||
|ALTER TABLE … DROP COLUMN|all||owner||
|ALTER TABLE … RENAME COLUMN|all||owner||
|SHOW COLUMNS|read-only||any||
|SELECT FROM table|read-only||select||
|SELECT FROM view|read-only||select, grant_select||
|INSERT INTO|all||insert||
|DELETE FROM|all||delete||
|UPDATE|all||update|

Các quyền cần thiết để thực thi các functions:

|SQL command|Catalog|Function permission|Note|
|---|---|---|---|
|`SELECT function()`||`execute`, `grant_execute*`|`grant_execute` is required when the function is used in a `SECURITY DEFINER` view.|
|`CREATE FUNCTION`|`all`|`ownership`|Not all connectors support [Catalog user-defined functions](https://trino-io.translate.goog/docs/current/udf/introduction.html?_x_tr_sl=en&_x_tr_tl=vi&_x_tr_hl=vi&_x_tr_pto=tc#udf-catalog).|
|`DROP FUNCTION`|`all`|`ownership`|Not all connectors support [Catalog user-defined functions](https://trino-io.translate.goog/docs/current/udf/introduction.html?_x_tr_sl=en&_x_tr_tl=vi&_x_tr_hl=vi&_x_tr_pto=tc#udf-catalog).|
#### Visibility

Để một catalog, schema hoặc bảng hiển thị trong lệnh SHOW, người dùng phải có ít nhất một quyền trên mục đó hoặc bất kỳ mục lồng nhau nào. Các mục lồng nhau không cần phải tồn tại sẵn vì bất kỳ quyền nào cũng có thể làm cho mục đó hiển thị. Cụ thể:
- `catalog`: Hiển thị nếu người dùng là chủ sở hữu của bất kỳ schema lồng nhau nào, có quyền truy cập vào bất kỳ bảng hoặc hàm lồng nhau nào, hoặc có quyền thiết lập thuộc tính phiên trong catalog.
- `schema`: Hiển thị nếu người dùng là chủ sở hữu của schema hoặc có quyền truy cập vào bất kỳ bảng hoặc hàm lồng nhau nào.
- `table`: Hiển thị nếu người dùng có bất kỳ quyền nào trên bảng.

#### Catalog rules

Mỗi quy tắc trong catalog bao gồm các trường sau:
- `user` (optional): regex to match against username. Defaults to `.*`.
- `role` (optional): regex to match against role names. Defaults to `.*`.
- `group` (optional): regex to match against group names. Defaults to `.*`.
- `catalog` (optional): regex to match against catalog name. Defaults to `.*`.
- `allow` (required): string indicating whether a user has access to the catalog. This value can be `all`, `read-only` or `none`, and defaults to `none`. Setting this value to `read-only` has the same behavior as the `read-only` system access control plugin.

Để một quy tắc được áp dụng, tên người dùng phải khớp với biểu thức chính quy được chỉ định trong thuộc tính `user`.

Đối với tên vai trò, một quy tắc có thể được áp dụng nếu ít nhất một trong các vai trò hiện đang được kích hoạt khớp với biểu thức chính quy của `role` đó.

Đối với tên nhóm, một quy tắc có thể được áp dụng nếu ít nhất một tên nhóm của người dùng này khớp với biểu thức chính quy của `group`.

>[!note]
>Theo mặc định, tất cả người dùng đều có quyền truy cập vào `system` catalog. Bạn có thể ghi đè hành vi này bằng cách thêm một quy tắc.
>Các giá trị boolean `true` và `false` cũng được hỗ trợ như các giá trị cũ cho thuộc tính `allow`, để đảm bảo khả năng tương thích ngược. `true` tương ứng với `all`, và `false` tương ứng với `none`.

Ví dụ, nếu bạn muốn chỉ cho phép vai trò `admin` truy cập vào `mysql` và `system` catalog, cho phép người dùng từ nhóm `finance` và `human_resources` truy cập vào `postgres` catalog, cho phép tất cả người dùng truy cập vào `hive` catalog, và từ chối tất cả các quyền truy cập khác, bạn có thể sử dụng các quy tắc sau:

```json
{
  "catalogs": [
    {
      "role": "admin",
      "catalog": "(mysql|system)",
      "allow": "all"
    },
    {
      "group": "finance|human_resources",
      "catalog": "postgres",
      "allow": true
    },
    {
      "catalog": "hive",
      "allow": "all"
    },
    {
      "user": "alice",
      "catalog": "postgresql",
      "allow": "read-only"
    },
    {
      "catalog": "system",
      "allow": "none"
    }
  ]
}
```

Để các quy tắc dựa trên nhóm được áp dụng, người dùng cần được chỉ định vào các nhóm bởi [Group provider](https://trino-io.translate.goog/docs/current/develop/group-provider.html?_x_tr_sl=en&_x_tr_tl=vi&_x_tr_hl=vi&_x_tr_pto=tc).

#### Schema rules

Mỗi quy tắc schema bao gồm các trường sau:
- `user` (optional): regex to match against username. Defaults to `.*`.
- `role` (optional): regex to match against role names. Defaults to `.*`.
- `group` (optional): regex to match against group names. Defaults to `.*`.
- `catalog` (optional): regex to match against catalog name. Defaults to `.*`.
- `schema` (optional): regex to match against schema name. Defaults to `.*`.
- `owner` (required): boolean indicating whether the user is to be considered an owner of the schema. Defaults to `false`.

Ví dụ, để cấp quyền sở hữu tất cả các schema cho vai trò `admin`, coi tất cả người dùng là chủ sở hữu của `default.default` schema và ngăn người dùng `guest` sở hữu bất kỳ schema nào, bạn có thể sử dụng các quy tắc sau:

```json
{
  "schemas": [
    {
      "role": "admin",
      "schema": ".*",
      "owner": true
    },
    {
      "user": "guest",
      "owner": false
    },
    {
      "catalog": "default",
      "schema": "default",
      "owner": true
    }
  ]
}
```

#### Table rules

Mỗi quy tắc bảng bao gồm các trường sau:
- `user` (optional): regex to match against username. Defaults to `.*`.
- `role` (optional): regex to match against role names. Defaults to `.*`.
- `group` (optional): regex to match against group names. Defaults to `.*`.
- `catalog` (optional): regex to match against catalog name. Defaults to `.*`.
- `schema` (optional): regex to match against schema name. Defaults to `.*`.
- `table` (optional): regex to match against table names. Defaults to `.*`.
- `privileges` (required): zero or more of `SELECT`, `INSERT`, `DELETE`, `UPDATE`, `OWNERSHIP`, `GRANT_SELECT`
- `columns` (optional): list of column constraints.
- `filter` (optional): boolean filter expression for the table.
- `filter_environment` (optional): environment use during filter evaluation.

#### Column constraint

Các ràng buộc này có thể được sử dụng để hạn chế quyền truy cập vào dữ liệu cột.
- `name`: name of the column.
- `allow` (optional): if false, column can not be accessed.
- `mask` (optional): mask expression applied to column.
- `mask_environment` (optional): environment use during mask evaluation

#### Filter and mask environment
- `user` (optional): Tên người dùng dùng để kiểm tra quyền truy vấn con trong mask.

>[!note]
>Các quy tắc này không áp dụng cho `information_schema`.
>`mask` có thể chứa các biểu thức điều kiện như `IF` hoặc `CASE`, giúp tạo ra mask có điều kiện.

Ví dụ dưới đây định nghĩa chính sách truy cập bảng như sau:
- Vai trò `admin` có đầy đủ quyền hạn trên tất cả các bảng và schema.
- Tất cả người dùng đều có quyền `SELECT` trên bảng `default.hr.employees`, nhưng bảng chỉ được lọc để hiển thị hàng dữ liệu của người dùng hiện tại.
- Tất cả người dùng đều có quyền `SELECT` trên tất cả các bảng trong `default.default` schema, ngoại trừ cột `address` bị chặn và cột `ssn` bị ẩn.

```json
{
  "tables": [
    {
      "role": "admin",
      "privileges": ["SELECT", "INSERT", "DELETE", "UPDATE", "OWNERSHIP"]
    },
    {
      "user": "banned_user",
      "privileges": []
    },
    {
      "catalog": "default",
      "schema": "hr",
      "table": "employee",
      "privileges": ["SELECT"],
      "filter": "user = current_user",
      "filter_environment": {
        "user": "system_user"
      }
    },
    {
      "catalog": "default",
      "schema": "default",
      "table": ".*",
      "privileges": ["SELECT"],
      "columns" : [
         {
            "name": "address",
            "allow": false
         },
         {
            "name": "SSN",
            "mask": "'XXX-XX-' + substring(credit_card, -4)",
            "mask_environment": {
              "user": "system_user"
            }
         }
      ]
    }
  ]
}
```

#### Function rules

Các quy tắc này kiểm soát khả năng người dùng tạo, xóa và thực thi các functions.

Khi các quy tắc này tồn tại, việc cấp quyền dựa trên quy tắc phù hợp đầu tiên, được xử lý từ trên xuống dưới. Nếu không có quy tắc nào phù hợp, quyền truy cập sẽ bị từ chối. Nếu không có quy tắc functions nào, chỉ các functions trong `system.builtin` mới có thể được thực thi.

>[!note]
>Người dùng luôn có quyền truy cập vào các hàm trong  `system.builtin` schema, và bạn không thể ghi đè hành vi này bằng cách thêm một quy tắc.

Mỗi quy tắc functions bao gồm các trường sau:
- `user` (optional): regular expression to match against username. Defaults to `.*`.
- `role` (optional): regular expression to match against role names. Defaults to `.*`.
- `group` (optional): regular expression to match against group names. Defaults to `.*`.
- `catalog` (optional): regular expression to match against catalog name. Defaults to `.*`.
- `schema` (optional): regular expression to match against schema name. Defaults to `.*`.
- `function` (optional): regular expression to match against function names. Defaults to `.*`.
- `privileges` (required): zero or more of `EXECUTE`, `GRANT_EXECUTE`, `OWNERSHIP`.

Cần thận trọng khi cấp quyền truy cập vào `system` schema của catalog, vì đây là schema mà Trino sử dụng cho các chức năng bảng như truy vấn. Các chức năng bảng này có thể được sử dụng để truy cập hoặc sửa đổi dữ liệu cơ bản của catalog.

Ví dụ sau cho phép người dùng `admin` thực thi hàm bảng `system.query` trong bất kỳ catalog nào, và cho phép tất cả người dùng tạo, xóa và thực thi các functions (bao gồm cả các `SECURITY DEFINER` views) trong `hive.function`  schema:

```json
{
  "functions": [
    {
      "user": "admin",
      "schema": "system",
      "function": "query",
      "privileges": [
        "EXECUTE"
      ]
    },
    {
      "catalog": "hive",
      "schema": "function",
      "privileges": [
        "EXECUTE", "GRANT_EXECUTE", "OWNERSHIP"
      ]
    }
  ]
}
```

Xem thêm:
- **Thủ tục (Procedure rules):** Ai được dùng lệnh `CALL` để chạy các thủ tục quản trị hệ thống (ví dụ: dọn dẹp bộ nhớ đệm, đăng ký table ngoại vi).
- **Thuộc tính phiên làm việc (Session property rules):** Ai được phép thay đổi các cấu hình khi chạy query (ví dụ: bật/tắt tính năng tối ưu hóa, giới hạn tài nguyên).    
- **Quản lý truy vấn (Query rules):** Ai được quyền chạy truy vấn (`execute`), xem tiến độ/lịch sử truy vấn (`view`), hoặc hủy/kill một truy vấn đang chạy (`kill`). _Lưu ý: User luôn có quyền xem hoặc hủy truy vấn của chính mình._
- **Giả mạo danh tính (Impersonation rules):** Cho phép một tài khoản (ví dụ: tài khoản Admin hoặc tài khoản của hệ thống tự động) được quyền chạy truy vấn **dưới danh nghĩa (impersonate)** một user khác.
- **Thông tin hệ thống (System information rules):** Ai được quyền truy cập vào các link API nhạy cảm của Trino (như xem trạng thái các Node, các luồng Thread xử lý) hoặc ra lệnh tắt cụm Trino một cách an toàn (Graceful shutdown).
- **Ủy quyền sở hữu (Authorization rules):** Ai được phép đổi chủ sở hữu (`SET AUTHORIZATION`) của một Schema, Table hoặc View sang cho người khác.

Nguồn: https://trino.io/docs/current/security/file-system-access-control.html