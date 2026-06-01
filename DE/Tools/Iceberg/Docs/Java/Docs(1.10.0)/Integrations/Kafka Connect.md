## Apache Iceberg Sink Connector

Apache Iceberg Sink Connector cho Kafka Connect là một bộ kết nối sink để ghi dữ liệu từ Kafka vào các bảng Iceberg.

## Features

- Commit coordination for centralized Iceberg commits
    
- Exactly-once delivery semantics
    
- Multi-table fan-out
    
- Automatic table creation and schema evolution
    
- Field name mapping via Iceberg’s column mapping functionality
    
## Installation

[https://github.com/apache/iceberg](https://github.com/apache/iceberg)

The connector zip archive is created as part of the Iceberg build. You can run the build via: 

```bash
cd kafka-connect

./gradlew -x test -x integrationTest clean build
```
