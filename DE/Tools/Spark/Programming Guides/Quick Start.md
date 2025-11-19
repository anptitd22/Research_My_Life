- [Interactive Analysis with the Spark Shell](https://spark.apache.org/docs/latest/quick-start.html#interactive-analysis-with-the-spark-shell)
    - [Basics](https://spark.apache.org/docs/latest/quick-start.html#basics)
    - [More on Dataset Operations](https://spark.apache.org/docs/latest/quick-start.html#more-on-dataset-operations)
    - [Caching](https://spark.apache.org/docs/latest/quick-start.html#caching)
- [Self-Contained Applications](https://spark.apache.org/docs/latest/quick-start.html#self-contained-applications)
- [Where to Go from Here](https://spark.apache.org/docs/latest/quick-start.html#where-to-go-from-here)

Hướng dẫn này cung cấp phần giới thiệu nhanh về cách sử dụng Spark. Trước tiên, chúng tôi sẽ giới thiệu API thông qua shell tương tác của Spark (bằng Python hoặc Scala), sau đó hướng dẫn cách viết ứng dụng bằng Java, Scala và Python. 

Để làm theo hướng dẫn này, trước tiên hãy tải xuống bản phát hành Spark đóng gói từ trang [Spark website](https://spark.apache.org/downloads.html). Vì chúng ta sẽ không sử dụng HDFS, bạn có thể tải xuống gói cho bất kỳ phiên bản Hadoop nào. 

Lưu ý rằng, trước Spark 2.0, giao diện lập trình chính của Spark là Resilient Distributed Dataset (RDD). Sau Spark 2.0, RDD được thay thế bằng Dataset, một giao diện được định kiểu mạnh tương tự RDD, nhưng được tối ưu hóa tốt hơn. Giao diện RDD vẫn được hỗ trợ, và bạn có thể tìm hiểu thêm thông tin chi tiết tại [RDD programming guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html). Tuy nhiên, chúng tôi thực sự khuyên bạn nên chuyển sang sử dụng Dataset, giao diện này có hiệu suất tốt hơn RDD. Xem [SQL programming guide](https://spark.apache.org/docs/latest/sql-programming-guide.html) để biết thêm thông tin về Dataset.

# Interactive Analysis with the Spark Shell

## Basics

Shell của Spark cung cấp một cách đơn giản để học API, cũng như một công cụ mạnh mẽ để phân tích dữ liệu một cách tương tác. Shell này có sẵn trong Scala (chạy trên máy ảo Java và do đó là một cách tốt để sử dụng các thư viện Java hiện có) hoặc Python. Khởi động nó bằng cách chạy lệnh sau trong thư mục Spark: