
- [[#`Catalog`|`Catalog`]]
	- [[#`Catalog`#`__enter__()`|`__enter__()`]]

## `Catalog`

Bases: `ABC`

Base Catalog cho các thao tác trên bảng như - create, drop, load, list và các thao tác khác.

The catalog table APIs chấp nhận một mã định danh bảng, là tên bảng được phân loại đầy đủ. Mã định danh có thể là một chuỗi hoặc một bộ chuỗi. Nếu mã định danh là một chuỗi, nó sẽ được chia thành một bộ với dấu '.' . Nếu là một bộ, nó sẽ được sử dụng nguyên trạng.

The catalog namespace APIs tuân theo một quy ước tương tự, trong đó chúng cũng chấp nhận một mã định danh không gian tên có thể là một chuỗi hoặc một bộ chuỗi.

Thuộc tính:

|Name|Type|Description|
|---|---|---|
|`name`|`str`|Name of the catalog.|
|`properties`|`[Properties](https://py.iceberg.apache.org/reference/pyiceberg/typedef/#pyiceberg.typedef.Properties "Properties = Dict[str, Any] module-attribute (pyiceberg.typedef.Properties)")`|Catalog properties.|

Source code in `pyiceberg/catalog/__init__.py`

### `__enter__()`

Nhập trình quản lý context.

Trả về:

| Name      | Type        | Description           |
| --------- | ----------- | --------------------- |
| `Catalog` | `'Catalog'` | The catalog instance. |



