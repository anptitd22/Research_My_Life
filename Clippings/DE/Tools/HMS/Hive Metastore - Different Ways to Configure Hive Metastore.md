---
title: "Hive Metastore - Different Ways to Configure Hive Metastore"
source: "https://data-flair.training/blogs/apache-hive-metastore/"
author:
  - "[[DataFlair Team]]"
published: 2017-09-01
created: 2026-04-17
description: "Apache Hive tutorial cover Hive Metastore Introduction,Configuring Hive Metastore,modes for Hive metastore deployment,Hive Local,embedded & remote metastore"
tags:
  - "clippings"
---
Boost your career with [Data Engineering Courses!!](https://techvidvan.com/data-engineering-courses/?campaign=btah&ref=1)

In this tutorial, we are going to introduce **Hive** **Metastore** in detail. Metastore is the central repository of Hive Metadata. It stores the meta data for Hive tables and relations. For example, Schema and Locations etc.

This Hive tutorial will cover what is Hive Metastore, how the Hive Metastore works, what is Derby in Hive, how to Configure Hive Metastore and What are the Databases Supported by Hive? We will discuss the answer to all the above questions in detail.

So, let’s start Hive Metastore Tutorial.

[![Hive Metastore - Different Ways to Configure Hive Metastore](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-metastore-tutorial-2.jpg)](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-metastore-tutorial-2.jpg)

Hive Metastore – Different Ways to Configure Hive Metastore

## What is Hive Metastore?

**Metastore** is the central repository of Apache Hive metadata. It stores metadata for Hive tables (like their schema and location) and **partitions** in a relational database. It provides client access to this information by using metastore service API.

Hive metastore consists of two fundamental units:

1. A service that provides metastore access to other Apache Hive services.
2. Disk storage for the Hive metadata which is separate from **HDFS** storage.

## Hive Metastore Modes

There are three modes for Hive Metastore deployment:

- Embedded Metastore
- Local Metastore
- Remote Metastore

Let’s now discuss the above three Hive Metastore deployment modes one by one-

**i. Embedded Metastore**

In **Hive** by default, metastore service runs in the same JVM as the Hive service. It uses embedded **derby** database stored on the local file system in this mode. Thus both metastore service and hive service runs in the same JVM by using embedded Derby Database.

But, this mode also has limitation that, as only one embedded Derby database can access the database files on disk at any one time, so only one Hive session could be open at a time.

[![Embedded Deployment mode for Hive Metastore](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-embedded-metastore.jpg)](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-embedded-metastore.jpg)

Embedded Deployment mode for Hive Metastore

If we try to start the second session it produces an error when it attempts to open a connection to the metastore. So, to allow many services to connect the Metastore, it configures Derby as a network server. This mode is good for unit testing. But it is not good for the practical solutions.

**ii. Local Metastore**

Hive is the data-warehousing framework, so hive does not prefer single session. To overcome this limitation of Embedded Metastore, for **Local Metastore** was introduced. This mode allows us to have many Hive sessions i.e. many users can use the metastore at the same time.

We can achieve by using any JDBC compliant like MySQL which runs in a separate JVM or different machines than that of the Hive service and metastore service which are running in the same JVM.

[![Local Deployment mode for Hive Metastore](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-local-metastore.jpg)](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-local-metastore.jpg)

Local Metastore

This configuration is called as local metastore because metastore service still runs in the same process as the Hive. But it connects to a database running in a separate process, either on the same machine or on a remote machine.

Before starting Apache Hive client, add the JDBC / ODBC driver libraries to the Hive lib folder**.**

MySQL is a popular choice for the standalone metastore. In this case, the ***javax.jdo.option.ConnectionURL*** property is set to ***jdbc:mysql://host/dbname?*** **createDatabaseIfNotExist=true**, and ***javax.jdo.option.ConnectionDriverName*** is set to ***com.mysql.jdbc.Driver.*** The JDBC driver JAR file for MySQL (Connector/J) must be on Hive’s classpath, which is achieved by placing it in Hive’s lib directory.

**iii. Remote Metastore**

Moving further, another metastore configuration called **Remote Metastore**. In this mode, metastore runs on its own separate JVM, not in the Hive service JVM. If other processes want to communicate with the metastore server they can communicate using Thrift Network APIs.

We can also have one more metastore servers in this case to provide more availability. This also brings better manageability/security because the database tier can be completely firewalled off. And the clients no longer need share database credentials with each Hiver user to access the metastore database.

[![Remote deployment mode for Hive Metastore](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-remote-metastore.jpg)](https://data-flair.training/blogs/wp-content/uploads/sites/2/2017/09/hive-remote-metastore.jpg)

Remote Metastore

To use this remote metastore, you should configure Hive service by setting ***hive.metastore.uris*** to the metastore server URI(s). Metastore server URIs are of the form *thrift://host:port,* where the port corresponds to the one set by METASTORE\_PORT when starting the metastore server.

## Databases Supported by Hive

Hive supports 5 backend databases which are as follows:

- Derby
- MySQL
- MS SQL Server
- Oracle
- Postgres

So, this was all in Hive Metastore. Hope you likeour explanation.

## Conclusion – Hive Metastore

In conclusion, we can say that Hive Metadata is a central repository for storing all the Hive metadata information. Metadata includes various types of information like the structure of tables, relations etc. Above we have also discussed all the three metastore modes in detail. you can also Learn the other **big data** technologies like **Apache Hadoop**, **Spark**, **Flink** etc in detail.

**Did we exceed your expectations?**  
If Yes, share your valuable feedback on **[Google](https://g.page/DataFlair/review?kd)**