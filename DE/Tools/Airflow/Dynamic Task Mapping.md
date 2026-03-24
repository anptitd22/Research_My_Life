# Dynamic Task Mapping[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#dynamic-task-mapping "Link to this heading")

## Simple mapping

Ở dạng đơn giản nhất, bạn có thể lặp qua một danh sách được định nghĩa trực tiếp trong tệp DAG của mình bằng cách sử dụng hàm `expand()` thay vì gọi trực tiếp tác vụ của bạn.

Nếu bạn muốn xem ví dụ đơn giản về Dynamic Task Mapping, bạn có thể xem bên dưới:

```
#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
"""Example DAG demonstrating the usage of dynamic task mapping."""

from __future__ import annotations
from datetime import datetime

from airflow.sdk import DAG, task

with DAG(dag_id="example_dynamic_task_mapping", schedule=None, start_date=datetime(2022, 3, 4)) as dag:

    @task
    def add_one(x: int):
        return x + 1

    @task
    def sum_it(values):
        total = sum(values)
        print(f"Total was {total}")

    added_values = add_one.expand(x=[1, 2, 3])
    sum_it(added_values)

with DAG(
    dag_id="example_task_mapping_second_order", schedule=None, catchup=False, start_date=datetime(2022, 3, 4)
) as dag2:

    @task
    def get_nums():
        return [1, 2, 3]

    @task
    def times_2(num):
        return num * 2

    @task
    def add_10(num):
        return num + 10

    _get_nums = get_nums()
    _times_2 = times_2.expand(num=_get_nums)
    add_10.expand(num=_times_2)
```

Thao tác này sẽ hiển thị "Tổng cộng là 9" trong nhật ký tác vụ khi được thực thi.

Đây là cấu trúc Dag thu được:

![alt][images/Tools/Airflow/Dynamic_task_mapping_1.png]

Chế độ xem dạng lưới cũng giúp bạn dễ dàng xem các nhiệm vụ đã được lập bản đồ trong bảng chi tiết:

![alt][images/Tools/Airflow/Dtm_2.png]

...
### Task-generated Mapping

...

### Repeated mapping

...

### Adding parameters that do not expand

Bên cạnh việc truyền các đối số được mở rộng trong quá trình chạy, cũng có thể truyền các đối số không thay đổi — để phân biệt rõ ràng giữa hai loại này, chúng ta sử dụng các hàm khác nhau: `expand()` cho mapped arguments và `partial()` cho unmapped ones.

```
@task
def add(x: int, y: int):
    return x + y


added_values = add.partial(y=10).expand(x=[1, 2, 3])
# This results in add function being expanded to
# add(x=1, y=10)
# add(x=2, y=10)
# add(x=3, y=10)
```

Điều này sẽ dẫn đến các giá trị là 11, 12 và 13.

Điều này cũng hữu ích để truyền các thông tin như ID kết nối, tên bảng cơ sở dữ liệu hoặc tên nhóm lưu trữ (bucket) cho các tác vụ.

### Mapping over multiple parameters

...

### Named mapping

Theo mặc định, các tác vụ được ánh xạ được gán một chỉ số nguyên. Có thể ghi đè chỉ số nguyên cho mỗi tác vụ được ánh xạ trong giao diện người dùng Airflow bằng một tên dựa trên đầu vào của tác vụ. Điều này được thực hiện bằng cách cung cấp một mẫu Jinja cho tác vụ với `map_index_template`. Thông thường, nó sẽ trông giống như `map_index_template="{{ task.<property> }}`  khi phần mở rộng trông giống như `.expand(<property>=...)` . Mẫu này được hiển thị sau khi mỗi tác vụ được mở rộng được thực thi bằng cách sử dụng ngữ cảnh tác vụ. Điều này có nghĩa là bạn có thể tham chiếu các thuộc tính trên tác vụ như sau:

```
from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator


# The two expanded task instances will be named "2024-01-01" and "2024-01-02".
SQLExecuteQueryOperator.partial(
    ...,
    sql="SELECT * FROM data WHERE date = %(date)s",
    map_index_template="""{{ task.parameters['date'] }}""",
).expand(
    parameters=[{"date": "2024-01-01"}, {"date": "2024-01-02"}],
)
```

Trong ví dụ trên, các phiên bản tác vụ được mở rộng sẽ được đặt tên là “2024-01-01” và “2024-01-02”. Tên này sẽ hiển thị trong giao diện người dùng Airflow thay vì “0” và “1” tương ứng.

Vì mẫu được hiển thị sau khối thực thi chính, nên cũng có thể chèn động vào ngữ cảnh hiển thị. Điều này hữu ích khi logic để hiển thị một tên mong muốn khó thể hiện bằng cú pháp mẫu Jinja, đặc biệt là trong một hàm luồng tác vụ. Ví dụ:

```
from airflow.sdk import get_current_context


@task(map_index_template="{{ my_variable }}")
def my_task(my_value: str):
    context = get_current_context()
    context["my_variable"] = my_value * 3
    ...  # Normal execution...


# The task instances will be named "aaa" and "bbb".
my_task.expand(my_value=["a", "b"])
```

## Mapping with non-TaskFlow operators

Có thể sử dụng `partial` và `expand` với các classic operator. Một số đối số không thể ánh xạ và phải được truyền cho `partial()`, chẳng hạn như `task_id`, `queue`, `pool` và hầu hết các đối số khác của `BaseOperator`.

```
#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
"""Example DAG demonstrating the usage of dynamic task mapping with non-TaskFlow operators."""

from __future__ import annotations

from datetime import datetime

from airflow.sdk import DAG, BaseOperator


class AddOneOperator(BaseOperator):
    """A custom operator that adds one to the input."""

    def __init__(self, value, **kwargs):
        super().__init__(**kwargs)
        self.value = value

    def execute(self, context):
        return self.value + 1


class SumItOperator(BaseOperator):
    """A custom operator that sums the input."""

    template_fields = ("values",)

    def __init__(self, values, **kwargs):
        super().__init__(**kwargs)
        self.values = values

    def execute(self, context):
        total = sum(self.values)
        print(f"Total was {total}")
        return total


with DAG(
    dag_id="example_dynamic_task_mapping_with_no_taskflow_operators",
    schedule=None,
    start_date=datetime(2022, 3, 4),
    catchup=False,
):
    # map the task to a list of values
    add_one_task = AddOneOperator.partial(task_id="add_one").expand(value=[1, 2, 3])

    # aggregate (reduce) the mapped tasks results
    sum_it_task = SumItOperator(task_id="sum_it", values=add_one_task.output)
```

>[!note]
>Chỉ những đối số từ khóa mới được phép truyền cho hàm `partial()`.

### Mapping over result of classic operators

Nếu bạn muốn sử dụng phép ánh xạ lên kết quả của một classic operator, bạn nên tham chiếu rõ ràng đến kết quả đầu ra, thay vì chính toán tử đó.

```
# Create a list of data inputs.
extract = ExtractOperator(task_id="extract")

# Expand the operator to transform each input.
transform = TransformOperator.partial(task_id="transform").expand(input=extract.output)

# Collect the transformed inputs, expand the operator to load each one of them to the target.
load = LoadOperator.partial(task_id="load").expand(input=transform.output)
```

### Mixing TaskFlow and classic operators

Trong ví dụ này, bạn có một quy trình chuyển dữ liệu định kỳ đến một bucket S3 và muốn áp dụng cùng một quy trình xử lý cho mọi tập tin đến, bất kể mỗi lần có bao nhiêu tập tin.

```
from datetime import datetime

from airflow.sdk import DAG
from airflow.sdk import task
from airflow.providers.amazon.aws.hooks.s3 import S3Hook
from airflow.providers.amazon.aws.operators.s3 import S3ListOperator


with DAG(dag_id="mapped_s3", start_date=datetime(2020, 4, 7)) as dag:
    list_filenames = S3ListOperator(
        task_id="get_input",
        bucket="example-bucket",
        prefix='incoming/provider_a/{{ data_interval_start.strftime("%Y-%m-%d") }}',
    )

    @task
    def count_lines(aws_conn_id, bucket, filename):
        hook = S3Hook(aws_conn_id=aws_conn_id)

        return len(hook.read_key(filename, bucket).splitlines())

    @task
    def total(lines):
        return sum(lines)

    counts = count_lines.partial(aws_conn_id="aws_default", bucket=list_filenames.bucket).expand(
        filename=list_filenames.output
    )

    total(lines=counts)
```

## Assigning multiple parameters to a non-TaskFlow operator[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#assigning-multiple-parameters-to-a-non-taskflow-operator "Link to this heading")

Đôi khi, một tiến trình upstream cần chỉ định nhiều đối số cho một tiến trình downstream. Để làm điều này, bạn có thể sử dụng hàm `expand_kwargs`, hàm này nhận một chuỗi các ánh xạ để ánh xạ tới.

```
BashOperator.partial(task_id="bash").expand_kwargs(
    [
        {"bash_command": "echo $ENV1", "env": {"ENV1": "1"}},
        {"bash_command": "printf $ENV2", "env": {"ENV2": "2"}},
    ],
)
```

Thao tác này tạo ra hai phiên bản tác vụ khi chạy, lần lượt in ra số 1 và 2.

Ngoài ra, cũng có thể kết hợp `expand_kwargs` với hầu hết các đối số của toán tử, ví dụ như `op_kwargs` của PythonOperator.

```
def print_args(x, y):
    print(x)
    print(y)
    return x + y


PythonOperator.partial(task_id="task-1", python_callable=print_args).expand_kwargs(
    [
        {"op_kwargs": {"x": 1, "y": 2}, "show_return_value_in_logs": True},
        {"op_kwargs": {"x": 3, "y": 4}, "show_return_value_in_logs": False},
    ]
)
```

Tương tự như thao tác `expand`, bạn cũng có thể ánh xạ tới một XCom trả về list các dict, hoặc một list các XCom mà mỗi XCom trả về một dict. Sử dụng lại ví dụ S3 ở trên, bạn có thể sử dụng một tác vụ được ánh xạ để thực hiện "phân nhánh" và sao chép tệp vào các bucket khác nhau:

```
list_filenames = S3ListOperator(...)  # Same as the above example.


@task
def create_copy_kwargs(filename):
    if filename.rsplit(".", 1)[-1] not in ("json", "yml"):
        dest_bucket_name = "my_text_bucket"
    else:
        dest_bucket_name = "my_other_bucket"
    return {
        "source_bucket_key": filename,
        "dest_bucket_key": filename,
        "dest_bucket_name": dest_bucket_name,
    }


copy_kwargs = create_copy_kwargs.expand(filename=list_filenames.output)

# Copy files to another bucket, based on the file's extension.
copy_filenames = S3CopyObjectOperator.partial(
    task_id="copy_files", source_bucket_name=list_filenames.bucket
).expand_kwargs(copy_kwargs)
```

## Mapping over a task group

Tương tự như tác vụ TaskFlow, bạn cũng có thể gọi `expand` hoặc `expand_kwargs` trên một hàm được trang trí bằng `@task_group` để tạo một nhóm tác vụ được ánh xạ:

>[!note]
>Phần này đã lược bỏ phần hướng dẫn thực hiện từng nhiệm vụ riêng lẻ để ngắn gọn hơn.

```
@task_group
def file_transforms(filename):
    return convert_to_yaml(filename)


file_transforms.expand(filename=["data1.json", "data2.json"])
```

Trong ví dụ trên, tác vụ `convert_to_yaml` được mở rộng thành hai thể hiện tác vụ trong quá trình chạy. Thể hiện đầu tiên sẽ nhận "data1.json" làm đầu vào, và thể hiện thứ hai nhận "data2.json".

### Value references in a task group function[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#value-references-in-a-task-group-function "Link to this heading")

Một điểm khác biệt quan trọng giữa hàm tác vụ (`@task`) và hàm nhóm tác vụ (`@task_group`) là, vì task group không có người thực hiện liên kết, nên code trong hàm task group không thể giải quyết các đối số được truyền vào; giá trị thực chỉ được giải quyết khi tham chiếu được truyền vào tác vụ.

Ví dụ, đoạn mã này sẽ không hoạt động:

```
@task
def my_task(value):
    print(value)


@task_group
def my_task_group(value):
    if not value:  # DOES NOT work as you'd expect!
        task_a = EmptyOperator(...)
    else:
        task_a = PythonOperator(...)
    task_a << my_task(value)


my_task_group.expand(value=[0, 1, 2])
```


Khi mã trong `my_task_group` được thực thi, `value` vẫn chỉ là một tham chiếu, chứ không phải giá trị thực, vì vậy nhánh `if not value` sẽ không hoạt động như bạn mong muốn. Tuy nhiên, nếu bạn truyền tham chiếu đó vào một tác vụ, nó sẽ được giải quyết khi tác vụ được thực thi, và ba thể hiện `my_task` do đó sẽ nhận được lần lượt là 1, 2 và 3.

Do đó, điều quan trọng cần nhớ là, nếu bạn định thực hiện bất kỳ thao tác logic nào với một giá trị được truyền vào hàm task group, bạn phải luôn sử dụng một tác vụ để chạy logic đó, chẳng hạn như `@task.branch` (hoặc `BranchPythonOperator`) cho các điều kiện và các phương thức ánh xạ tác vụ cho các vòng lặp.

>[!note]
>Task-mapping trong một task group đã được ánh xạ là không được phép. 
>
>Hiện tại, việc ánh xạ tác vụ lồng ghép bên trong một task group đã được ánh xạ là không được phép. Mặc dù khía cạnh kỹ thuật của tính năng này không quá khó, nhưng chúng tôi đã quyết định cố tình bỏ qua tính năng này vì nó làm tăng đáng kể độ phức tạp của giao diện người dùng và có thể không cần thiết cho các trường hợp sử dụng thông thường. Hạn chế này có thể được xem xét lại trong tương lai tùy thuộc vào phản hồi của người dùng.

### Depth-first execution[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#depth-first-execution "Link to this heading")

Nếu một task group được ánh xạ chứa nhiều tác vụ, tất cả các tác vụ trong nhóm sẽ được mở rộng “cùng nhau” dựa trên cùng một đầu vào. Ví dụ:

```
@task_group
def file_transforms(filename):
    converted = convert_to_yaml(filename)
    return replace_defaults(converted)


file_transforms.expand(filename=["data1.json", "data2.json"])
```

Vì nhóm `file_transforms` được mở rộng thành hai nhóm, nên các tác vụ `convert_to_yaml` và `replace_defaults` sẽ trở thành hai thể hiện riêng biệt trong quá trình thực thi.

Có thể đạt được hiệu quả tương tự bằng cách mở rộng hai nhiệm vụ một cách riêng biệt như sau:

```
converted = convert_to_yaml.expand(filename=["data1.json", "data2.json"])
replace_defaults.expand(filename=converted)
```

Tuy nhiên, điểm khác biệt là một task group cho phép mỗi tác vụ bên trong chỉ phụ thuộc vào “đầu vào liên quan” của nó. Đối với ví dụ trên, tác vụ `replace_defaults` sẽ chỉ phụ thuộc vào `convert_to_yaml` của cùng một nhóm mở rộng, chứ không phải các trường hợp của cùng một tác vụ nhưng thuộc nhóm khác. Chiến lược này, được gọi là thực thi theo chiều sâu (_depth-first execution_) (trái ngược với thực thi theo chiều rộng _breath-first execution_ đơn giản không có nhóm), cho phép phân tách tác vụ hợp lý hơn, các quy tắc phụ thuộc chi tiết hơn và phân bổ tài nguyên chính xác hơn — sử dụng ví dụ trên, tác vụ `replace_defaults` đầu tiên có thể chạy trước khi `convert_to_yaml("data2.json")` hoàn thành và không cần quan tâm đến việc nó có thành công hay không.

### Depending on a mapped task group’s output[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#depending-on-a-mapped-task-group-s-output "Link to this heading")

Tương tự như task group được ánh xạ, việc dựa vào kết quả đầu ra của task group được ánh xạ cũng sẽ tự động tổng hợp kết quả của nhóm đó:

```
@task_group
def add_to(value):
    value = add_one(value)
    return double(value)


results = add_to.expand(value=[1, 2, 3])
consumer(results)  # Will receive [4, 6, 8].
```

Cũng có thể thực hiện bất kỳ thao tác nào như là kết quả từ một tác vụ được ánh xạ thông thường.

#### Branching on a mapped task group’s output[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#branching-on-a-mapped-task-group-s-output "Link to this heading")

Mặc dù không thể triển khai logic phân nhánh (ví dụ: sử dụng `@task.branch`) trên kết quả của một tác vụ được ánh xạ, nhưng vẫn có thể phân nhánh dựa trên đầu vào của một task group. Ví dụ sau đây minh họa việc thực thi một trong ba tác vụ dựa trên đầu vào của một task group được ánh xạ.

```
inputs = ["a", "b", "c"]


@task_group(group_id="my_task_group")
def my_task_group(input):
    @task.branch
    def branch(element):
        if "a" in element:
            return "my_task_group.a"
        elif "b" in element:
            return "my_task_group.b"
        else:
            return "my_task_group.c"

    @task
    def a():
        print("a")

    @task
    def b():
        print("b")

    @task
    def c():
        print("c")

    branch(input) >> [a(), b(), c()]


my_task_group.expand(input=inputs)
```

## Filtering items from a mapped task

Một tác vụ được ánh xạ có thể loại bỏ bất kỳ phần tử nào khỏi việc được chuyển tiếp đến các tác vụ tiếp theo bằng cách trả về `None`. Ví dụ: nếu chúng ta chỉ muốn sao chép các tệp từ một nhóm S3 sang nhóm khác với các phần mở rộng nhất định, chúng ta có thể triển khai `create_copy_kwargs` như sau:

```
@task
def create_copy_kwargs(filename):
    # Skip files not ending with these suffixes.
    if filename.rsplit(".", 1)[-1] not in ("json", "yml"):
        return None
    return {
        "source_bucket_key": filename,
        "dest_bucket_key": filename,
        "dest_bucket_name": "my_other_bucket",
    }


# copy_kwargs and copy_files are implemented the same.
```

Điều này khiến `copy_files` chỉ mở rộng đối với các tệp `.json` và `.yml`, trong khi bỏ qua các loại tệp còn lại.

## Transforming expanding data[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#transforming-expanding-data "Link to this heading")

Vì việc chuyển đổi định dạng dữ liệu đầu ra cho việc ánh xạ tác vụ là khá phổ biến, đặc biệt là từ một toán tử không phải TaskFlow, nơi định dạng đầu ra đã được xác định trước và không thể dễ dàng chuyển đổi (như `create_copy_kwargs` trong ví dụ trên), nên có thể sử dụng một hàm `map()` đặc biệt để dễ dàng thực hiện loại chuyển đổi này. Do đó, ví dụ trên có thể được sửa đổi như sau:

```
from airflow.exceptions import AirflowSkipException

list_filenames = S3ListOperator(...)  # Unchanged.


def create_copy_kwargs(filename):
    if filename.rsplit(".", 1)[-1] not in ("json", "yml"):
        raise AirflowSkipException(f"skipping {filename!r}; unexpected suffix")
    return {
        "source_bucket_key": filename,
        "dest_bucket_key": filename,
        "dest_bucket_name": "my_other_bucket",
    }


copy_kwargs = list_filenames.output.map(create_copy_kwargs)

# Unchanged.
copy_filenames = S3CopyObjectOperator.partial(...).expand_kwargs(copy_kwargs)
```

Có một vài điều cần lưu ý:

1. Đối số có thể gọi của hàm `map()` (ví dụ: `create_copy_kwargs`) không được là một task, mà phải là một hàm Python thông thường. Phép biến đổi là một phần của quá trình "tiền xử lý" của task tiếp theo (tức là `copy_files`), chứ không phải là một task độc lập trong DAG. 
2. Hàm này luôn nhận chính xác một đối số vị trí. Hàm này được gọi cho mỗi mục trong đối tượng có thể lặp được dùng để task-mapping, tương tự như cách hoạt động của hàm `map()` tích hợp sẵn trong Python. 
3. Vì hàm được thực thi như một phần của tác vụ tiếp theo, bạn có thể sử dụng bất kỳ kỹ thuật hiện có nào để viết hàm tác vụ. Ví dụ, để đánh dấu một thành phần là bị bỏ qua, bạn nên ném ngoại lệ AirflowSkipException. Lưu ý rằng việc trả về `None` không hoạt động ở đây.

## Combining upstream data (aka “zipping”)[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#combining-upstream-data-aka-zipping "Link to this heading")

## Concatenating multiple upstreams[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#concatenating-multiple-upstreams "Link to this heading")

## What data types can be expanded?[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#what-data-types-can-be-expanded "Link to this heading")

## How do templated fields and mapped arguments interact?[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#how-do-templated-fields-and-mapped-arguments-interact "Link to this heading")

## Placing limits on mapped tasks[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#placing-limits-on-mapped-tasks "Link to this heading")

## Automatically skipping zero-length maps[](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html#automatically-skipping-zero-length-maps "Link to this heading")


