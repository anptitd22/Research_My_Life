Hướng dẫn này sẽ giúp bạn bắt đầu và chạy Apache Iceberg™ bằng Apache Spark™, ​​bao gồm mã mẫu để làm nổi bật một số tính năng mạnh mẽ. Bạn có thể tìm hiểu thêm về thời gian chạy Spark của Iceberg bằng cách xem phần Spark.

- [Docker-Compose](https://iceberg.apache.org/spark-quickstart/#docker-compose)
- [Creating a table](https://iceberg.apache.org/spark-quickstart/#creating-a-table)
- [Writing Data to a Table](https://iceberg.apache.org/spark-quickstart/#writing-data-to-a-table)
- [Reading Data from a Table](https://iceberg.apache.org/spark-quickstart/#reading-data-from-a-table)
- [Adding A Catalog](https://iceberg.apache.org/spark-quickstart/#adding-a-catalog)
- [Next Steps](https://iceberg.apache.org/spark-quickstart/#next-steps)

### Docker-Compose

Cách nhanh nhất để bắt đầu là sử dụng tệp docker-compose sử dụng [tabulario/spark-iceberg](https://hub.docker.com/r/tabulario/spark-iceberg) chứa cụm Spark cục bộ với Iceberg catalog đã được cấu hình. Để sử dụng, bạn cần cài đặt [Docker CLI](https://docs.docker.com/get-docker/) cũng như [Docker Compose CLI](https://github.com/docker/compose-cli/blob/main/INSTALL.md).

Sau khi có những thứ đó, hãy lưu yaml bên dưới vào một tệp có tên là `docker-compose.yml`:

```
services:
  spark-iceberg:
    image: tabulario/spark-iceberg
    container_name: spark-iceberg
    build: spark/
    networks:
      iceberg_net:
    depends_on:
      - rest
      - minio
    volumes:
      - ./warehouse:/home/iceberg/warehouse
      - ./notebooks:/home/iceberg/notebooks/notebooks
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
    ports:
      - 8888:8888
      - 8080:8080
      - 10000:10000
      - 10001:10001
  rest:
    image: apache/iceberg-rest-fixture
    container_name: iceberg-rest
    networks:
      iceberg_net:
    ports:
      - 8181:8181
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
      - CATALOG_WAREHOUSE=s3://warehouse/
      - CATALOG_IO__IMPL=org.apache.iceberg.aws.s3.S3FileIO
      - CATALOG_S3_ENDPOINT=http://minio:9000
  minio:
    image: minio/minio
    container_name: minio
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=minio
    networks:
      iceberg_net:
        aliases:
          - warehouse.minio
    ports:
      - 9001:9001
      - 9000:9000
    command: ["server", "/data", "--console-address", ":9001"]
  mc:
    depends_on:
      - minio
    image: minio/mc
    container_name: mc
    networks:
      iceberg_net:
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
    entrypoint: |
      /bin/sh -c "
      until (/usr/bin/mc alias set minio http://minio:9000 admin password) do echo '...waiting...' && sleep 1; done;
      /usr/bin/mc rm -r --force minio/warehouse;
      /usr/bin/mc mb minio/warehouse;
      /usr/bin/mc policy set public minio/warehouse;
      tail -f /dev/null
      "
networks:
  iceberg_net:
```

Tiếp theo, khởi động các container docker bằng lệnh này:

```
docker-compose up
```

Sau đó, bạn có thể chạy bất kỳ lệnh nào sau đây để bắt đầu Spark session.

~~~tabs
tab: SparkSQL
```
INSERT INTO demo.nyc.taxis
VALUES (1, 1000371, 1.8, 15.32, 'N'), (2, 1000372, 2.5, 22.15, 'N'), (2, 1000373, 0.9, 9.01, 'N'), (1, 1000374, 8.4, 42.13, 'Y');
```

tab: Spark-Shell
```
import org.apache.spark.sql.Row

val schema = spark.table("demo.nyc.taxis").schema
val data = Seq(
    Row(1: Long, 1000371: Long, 1.8f: Float, 15.32: Double, "N": String),
    Row(2: Long, 1000372: Long, 2.5f: Float, 22.15: Double, "N": String),
    Row(2: Long, 1000373: Long, 0.9f: Float, 9.01: Double, "N": String),
    Row(1: Long, 1000374: Long, 8.4f: Float, 42.13: Double, "Y": String)
)
val df = spark.createDataFrame(spark.sparkContext.parallelize(data), schema)
df.writeTo("demo.nyc.taxis").append()
```

tab: PySpark
```
schema = spark.table("demo.nyc.taxis").schema
data = [
    (1, 1000371, 1.8, 15.32, "N"),
    (2, 1000372, 2.5, 22.15, "N"),
    (2, 1000373, 0.9, 9.01, "N"),
    (1, 1000374, 8.4, 42.13, "Y")
  ]
df = spark.createDataFrame(data, schema)
df.writeTo("demo.nyc.taxis").append()
```
~~~


### Reading Data from a Table

Để đọc bảng, chỉ cần sử dụng tên bảng Iceberg.

~~~tabs
tab: SparkSQL
```
SELECT * FROM demo.nyc.taxis;
```

tab: Spark-Shell
```
val df = spark.table("demo.nyc.taxis").show()
```

tab: PySpark
```
df = spark.table("demo.nyc.taxis").show()
```
~~~

### Adding A Catalog

Iceberg có một số catalog back-ends có thể được sử dụng để theo dõi bảng, chẳng hạn như JDBC, Hive MetaStore và Glue. Catalog được cấu hình bằng các thuộc tính trong `spark.sql.catalog.(catalog_name)`. Trong hướng dẫn này, chúng tôi sử dụng JDBC, nhưng bạn có thể làm theo các hướng dẫn này để cấu hình các loại catalog khác. Để tìm hiểu thêm, hãy xem trang [Catalog](https://iceberg.apache.org/docs/latest/spark-configuration/#catalogs) trong phần Spark.

Cấu hình này tạo ra một catalog dựa trên đường dẫn có tên là `local` cho các bảng trong `$PWD/warehouse` và thêm hỗ trợ cho các bảng Iceberg vào danh mục tích hợp của Spark.

~~~tabs

tab: CLI
```
spark-sql --packages org.apache.iceberg:iceberg-spark-runtime-4.0_2.13:1.10.0\
    --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions \
    --conf spark.sql.catalog.spark_catalog=org.apache.iceberg.spark.SparkSessionCatalog \
    --conf spark.sql.catalog.spark_catalog.type=hive \
    --conf spark.sql.catalog.local=org.apache.iceberg.spark.SparkCatalog \
    --conf spark.sql.catalog.local.type=hadoop \
    --conf spark.sql.catalog.local.warehouse=$PWD/warehouse \
    --conf spark.sql.defaultCatalog=local
```

tab: spark-defaults.conf
```
spark.jars.packages                                  org.apache.iceberg:iceberg-spark-runtime-4.0_2.13:1.10.0
spark.sql.extensions                                 org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions
spark.sql.catalog.spark_catalog                      org.apache.iceberg.spark.SparkSessionCatalog
spark.sql.catalog.spark_catalog.type                 hive
spark.sql.catalog.local                              org.apache.iceberg.spark.SparkCatalog
spark.sql.catalog.local.type                         hadoop
spark.sql.catalog.local.warehouse                    $PWD/warehouse
spark.sql.defaultCatalog                             local
```	
~~~

> [!note]
>Nếu Iceberg catalog của bạn không được đặt làm catalog mặc định, bạn sẽ phải chuyển sang catalog này bằng cách thực hiện lệnh USE local;

### Next steps

#### Adding Iceberg to Spark[](https://iceberg.apache.org/spark-quickstart/#adding-iceberg-to-spark "Permanent link")

Nếu bạn đã có môi trường Spark, bạn có thể thêm Iceberg bằng cách sử dụng tùy chọn `--packages`.

~~~tabs

tab: SparkSQL
```
spark-sql --packages org.apache.iceberg:iceberg-spark-runtime-4.0_2.13:1.10.0
```

tab: Spark-Shell
```
spark-shell --packages org.apache.iceberg:iceberg-spark-runtime-4.0_2.13:1.10.0
```

tab: PySpark
```
pyspark --packages org.apache.iceberg:iceberg-spark-runtime-4.0_2.13:1.10.0
```
~~~

>[!note]
>Nếu bạn muốn đưa Iceberg vào cài đặt Spark, hãy thêm Iceberg Spark runtime vào thư mục jars của Spark. Bạn có thể tải xuống runtime bằng cách truy cập trang [Releases](https://iceberg.apache.org/releases/)

#### Learn More

Bây giờ bạn đã làm quen với Iceberg và Spark, hãy xem tài liệu [Iceberg-Spark docs](https://iceberg.apache.org/docs/latest/spark-ddl/) để tìm hiểu thêm!
