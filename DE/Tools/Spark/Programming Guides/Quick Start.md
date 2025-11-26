- [[#Basics|Basics]]
- [[#More on Dataset Operations|More on Dataset Operations]]
- [[#Caching|Caching]]

Hướng dẫn này cung cấp phần giới thiệu nhanh về cách sử dụng Spark. Trước tiên, chúng tôi sẽ giới thiệu API thông qua shell tương tác của Spark (bằng Python hoặc Scala), sau đó hướng dẫn cách viết ứng dụng bằng Java, Scala và Python. 

Để làm theo hướng dẫn này, trước tiên hãy tải xuống bản phát hành Spark đóng gói từ trang [Spark website](https://spark.apache.org/downloads.html). Vì chúng ta sẽ không sử dụng HDFS, bạn có thể tải xuống gói cho bất kỳ phiên bản Hadoop nào. 

Lưu ý rằng, trước Spark 2.0, giao diện lập trình chính của Spark là Resilient Distributed Dataset (RDD). Sau Spark 2.0, RDD được thay thế bằng Dataset, một giao diện được định kiểu mạnh tương tự RDD, nhưng được tối ưu hóa tốt hơn. Giao diện RDD vẫn được hỗ trợ, và bạn có thể tìm hiểu thêm thông tin chi tiết tại [RDD programming guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html). Tuy nhiên, chúng tôi thực sự khuyên bạn nên chuyển sang sử dụng Dataset, giao diện này có hiệu suất tốt hơn RDD. Xem [SQL programming guide](https://spark.apache.org/docs/latest/sql-programming-guide.html) để biết thêm thông tin về Dataset.

# Interactive Analysis with the Spark Shell

## Basics

Shell của Spark cung cấp một cách đơn giản để học API, cũng như một công cụ mạnh mẽ để phân tích dữ liệu một cách tương tác. Shell này có sẵn trong Scala (chạy trên máy ảo Java và do đó là một cách tốt để sử dụng các thư viện Java hiện có) hoặc Python. Khởi động nó bằng cách chạy lệnh sau trong thư mục Spark:
~~~tabs
tab: Python
```python
./bin/pyspark
```
Hoặc nếu PySpark được cài đặt với pip trong môi trường hiện tại của bạn:
```
pyspark
```
Sự trừu tượng hóa chính của Spark là một tập hợp phân tán các mục được gọi là Tập dữ liệu (Dataset). Tập dữ liệu có thể được tạo từ Hadoop InputFormats (chẳng hạn như các tệp HDFS) hoặc bằng cách chuyển đổi các Tập dữ liệu khác. Do tính chất động của Python, chúng ta không cần Tập dữ liệu phải được định kiểu mạnh trong Python. Do đó, tất cả các Tập dữ liệu trong Python đều là Dataset[Row], và chúng ta gọi nó là DataFrame để thống nhất với khái niệm khung dữ liệu trong Pandas và R. Hãy tạo một DataFrame mới từ văn bản của tệp README trong thư mục nguồn Spark:
```python
>>> textFile = spark.read.text("README.md")
```
Bạn có thể lấy giá trị trực tiếp từ DataFrame bằng cách gọi một số hành động, hoặc chuyển đổi DataFrame để lấy giá trị mới. Để biết thêm chi tiết, vui lòng đọc [API doc](https://spark.apache.org/docs/latest/api/python/index.html#pyspark.sql.DataFrame).
```python
>>> textFile.count()  # Number of rows in this DataFrame
126

>>> textFile.first()  # First row in this DataFrame
Row(value=u'# Apache Spark')
```
Bây giờ, hãy chuyển đổi DataFrame này thành một DataFrame mới. Chúng ta gọi hàm filter để trả về một DataFrame mới với một tập hợp con các dòng trong tệp.
```python
>>> linesWithSpark = textFile.filter(textFile.value.contains("Spark"))
```
Chúng ta có thể kết nối các chuyển đổi và hành động với nhau:
```python
>>> textFile.filter(textFile.value.contains("Spark")).count()  # How many lines contain "Spark"?
15
```
tab: Scala
skip
~~~
## More on Dataset Operations

Các hành động và chuyển đổi tập dữ liệu có thể được sử dụng cho các phép tính phức tạp hơn. Giả sử chúng ta muốn tìm dòng có nhiều từ nhất:
~~~tabs
tab: Python
```python
>>> from pyspark.sql import functions as sf
>>> textFile.select(sf.size(sf.split(textFile.value, "\s+")).name("numWords")).agg(sf.max(sf.col("numWords"))).collect()
[Row(max(numWords)=15)]
```
Đầu tiên, lệnh này ánh xạ một dòng thành một giá trị số nguyên và đặt bí danh là "numWords", tạo ra một DataFrame mới. agg được gọi trên DataFrame đó để tìm số từ lớn nhất. Các đối số cho select và agg đều là Column, chúng ta có thể sử dụng df.colName để lấy một cột từ DataFrame. Chúng ta cũng có thể import pyspark.sql.functions, cung cấp rất nhiều hàm tiện lợi để xây dựng một Column mới từ một Column cũ.

Một mô hình luồng dữ liệu phổ biến là MapReduce, được Hadoop phổ biến. Spark có thể triển khai luồng MapReduce một cách dễ dàng:
```python
>>> wordCounts = textFile.select(sf.explode(sf.split(textFile.value, "\s+")).alias("word")).groupBy("word").count()
```
Ở đây, chúng ta sử dụng hàm explode trong select để chuyển đổi một Dataset gồm các dòng thành một Dataset gồm các từ, sau đó kết hợp groupBy và count để tính số lượng từ trong tệp dưới dạng một DataFrame gồm 2 cột: "word" và "count". Để thu thập số lượng từ trong shell, chúng ta có thể gọi collect:
```python
>>> wordCounts.collect()
[Row(word=u'online', count=1), Row(word=u'graphs', count=1), ...]
```

tab: Scala
skip
~~~

## Caching

Spark cũng hỗ trợ việc kéo các tập dữ liệu vào bộ nhớ đệm trong bộ nhớ cụm. Điều này rất hữu ích khi dữ liệu được truy cập nhiều lần, chẳng hạn như khi truy vấn một tập dữ liệu nhỏ "nóng" hoặc khi chạy một thuật toán lặp như PageRank. Ví dụ đơn giản, hãy đánh dấu tập dữ liệu linesWithSpark của chúng ta để được lưu vào bộ nhớ đệm:

```python
>>>linesWithSpark.cache()

>>>linesWithSpark.count()
15

>>>linesWithSpark.count()
15
```

Có vẻ ngớ ngẩn khi sử dụng Spark để khám phá và lưu trữ đệm một tệp văn bản 100 dòng. Điều thú vị là những hàm này có thể được sử dụng trên các tập dữ liệu rất lớn, ngay cả khi chúng được phân chia thành hàng chục hoặc hàng trăm node. Bạn cũng có thể thực hiện việc này một cách tương tác bằng cách kết nối bin/pyspark với một cụm, như được mô tả trong [RDD programming guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html#using-the-shell).

# Self-Contained Applications

Giả sử chúng ta muốn viết một ứng dụng độc lập sử dụng Spark API. Chúng ta sẽ hướng dẫn một ứng dụng đơn giản bằng Scala (với sbt), Java (với Maven) và Python (pip).
~~~tabs
tab: Python
Bây giờ chúng tôi sẽ chỉ cho bạn cách viết ứng dụng bằng Python API (PySpark). Nếu bạn đang xây dựng một ứng dụng hoặc thư viện PySpark đóng gói, bạn có thể thêm nó vào tệp setup.py như sau:
```
install_requires=[
        'pyspark==4.0.1'
    ]
```
Ví dụ, chúng ta sẽ tạo một ứng dụng Spark đơn giản, SimpleApp.py:
```python
"""SimpleApp.py"""
from pyspark.sql import SparkSession

logFile = "YOUR_SPARK_HOME/README.md"  # Should be some file on your system
spark = SparkSession.builder.appName("SimpleApp").getOrCreate()
logData = spark.read.text(logFile).cache()

numAs = logData.filter(logData.value.contains('a')).count()
numBs = logData.filter(logData.value.contains('b')).count()

print("Lines with a: %i, lines with b: %i" % (numAs, numBs))

spark.stop()
```
Chương trình này chỉ đếm số dòng chứa 'a' và số dòng chứa 'b' trong một tệp văn bản. Lưu ý rằng bạn cần thay YOUR_SPARK_HOME bằng vị trí cài đặt Spark. Giống như các ví dụ về Scala và Java, chúng tôi sử dụng SparkSession để tạo Bộ dữ liệu. Đối với các ứng dụng sử dụng các lớp tùy chỉnh hoặc thư viện của bên thứ ba, chúng tôi cũng có thể thêm các phụ thuộc mã vào spark-submit thông qua đối số --py-files của nó bằng cách đóng gói chúng vào một tệp .zip (xem spark-submit --help để biết chi tiết). SimpleApp đủ đơn giản để chúng tôi không cần chỉ định bất kỳ phụ thuộc mã nào.

Chúng ta có thể chạy ứng dụng này bằng cách sử dụng tập lệnh bin/spark-submit:
```python
# Use spark-submit to run your application
$ YOUR_SPARK_HOME/bin/spark-submit \
  --master "local[4]" \
  SimpleApp.py
...
Lines with a: 46, Lines with b: 23
```
Nếu bạn đã cài đặt pip PySpark vào môi trường của mình (ví dụ: pip install pyspark), bạn có thể chạy ứng dụng bằng trình thông dịch Python thông thường hoặc sử dụng 'spark-submit' được cung cấp tùy theo ý muốn.
```python
# Use the Python interpreter to run your application
$ python SimpleApp.py
...
Lines with a: 46, Lines with b: 23
```
Các công cụ quản lý phụ thuộc khác như Conda và pip cũng có thể được sử dụng cho các lớp tùy chỉnh hoặc thư viện của bên thứ ba. Xem thêm [Python Package Management](https://spark.apache.org/docs/latest/api/python/tutorial/python_packaging.html).
tab: Scala
skip
~~~

# Where to Go from Here

Xin chúc mừng vì bạn đã chạy ứng dụng Spark đầu tiên!
- Để có cái nhìn tổng quan sâu sắc về API, hãy bắt đầu với [RDD programming guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html) và [SQL programming guide](https://spark.apache.org/docs/latest/sql-programming-guide.html) hoặc xem menu “Hướng dẫn lập trình” để biết các thành phần khác.
    
- Để chạy ứng dụng trên cụm, hãy xem phần [deployment overview](https://spark.apache.org/docs/latest/cluster-overview.html).
    
- Cuối cùng, Spark bao gồm một số ví dụ mẫu trong thư mục ví dụ ([Python](https://github.com/apache/spark/tree/master/examples/src/main/python), [Scala](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples), [Java](https://github.com/apache/spark/tree/master/examples/src/main/java/org/apache/spark/examples), [R](https://github.com/apache/spark/tree/master/examples/src/main/r)). Bạn có thể chạy chúng như sau:
    
# For Python examples, use spark-submit directly:
./bin/spark-submit examples/src/main/python/pi.py

# For Scala and Java, use run-example:
./bin/run-example SparkPi

# For R examples, use spark-submit directly:
./bin/spark-submit examples/src/main/r/dataframe.R
