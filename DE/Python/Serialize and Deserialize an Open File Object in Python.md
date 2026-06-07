Tuần tự hóa (Serialization) đề cập đến quá trình chuyển đổi một đối tượng thành định dạng có thể dễ dàng lưu trữ hoặc truyền tải, chẳng hạn như luồng byte. Mặt khác, giải tuần tự hóa (Deserialization) liên quan đến việc tái tạo đối tượng từ dạng đã được serialization. Khi xử lý các thao tác với tệp, thông thường người ta sẽ serialization dữ liệu trước khi ghi vào tệp và deserialization khi đọc lại. Trong bài viết này, chúng ta sẽ tìm hiểu cách serialization và deserialization một đối tượng tệp đang mở trong Python.

## Serialize and Deserialize an Open File Object in Python

Dưới đây là các cách để serialization và deserialization một đối tượng tập tin đang mở trong Python:

- Using Pickle
- Using JSON
- Using YAML

### Serialize and Deserialize an Open File Object Using Pickle

```
import pickle

# Serialize
data = {'name': 'John', 'age': 30}
with open('data.pickle', 'wb') as file:
    pickle.dump(data, file)

# Deserialize
with open('data.pickle', 'rb') as file:
    serialized_data = file.read()
    file.seek(0)
    loaded_data = pickle.load(file)

print("Type of serialized data:", type(serialized_data))
print("\nDeserialized data:", loaded_data)
print("Type of deserialized data:", type(loaded_data))
```

**Output**

```
Type of serialized data: <class 'bytes'>

Deserialized data: {'name': 'John', 'age': 30}
Type of deserialized data: <class 'dict'>
```

Xem thêm: https://docs.python.org/3.12/library/pickle.html

### Serialize and Deserialize an Open File Object Using JSON

```
import json

# Serialize
data = {'name': 'John', 'age': 30}
with open('data.json', 'w') as file:
    json.dump(data, file)

# Deserialize
with open('data.json', 'r') as file:
    serialized_data = file.read()
    file.seek(0)
    loaded_data = json.load(file)

print("Type of serialized data:", type(serialized_data))
print("\nDeserialized data:", loaded_data)
print("Type of deserialized data:", type(loaded_data))
```

**Output**

```
Type of serialized data: <class 'str'>

Deserialized data: {'name': 'John', 'age': 30}
Type of deserialized data: <class 'dict'>
```

### Serialize and Deserialize an Open File Object Using YAML

```
import yaml

# Serialize
data = {'name': 'John', 'age': 30}
with open('data.yaml', 'w') as file:
    yaml.dump(data, file)

# Deserialize
with open('data.yaml', 'r') as file:
    serialized_data = file.read()
    file.seek(0)
    loaded_data = yaml.safe_load(file)

print("Type of serialized data:", type(serialized_data))
print("Deserialized data:", loaded_data)
print("Type of deserialized data:", type(loaded_data))
```

****Output****

```
Type of serialized data: <class 'str'>    
Deserialized data: {'age': 30, 'name': 'John'}  
Type of deserialized data: <class 'dict'>
```