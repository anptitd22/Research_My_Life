---
title: "MySQL :: MySQL 9.7 Reference Manual :: 17.5.2 Change Buffer"
source: "https://dev.mysql.com/doc/refman/9.7/en/innodb-change-buffer.html"
author:
published:
created: 2026-05-13
description:
tags:
  - "clippings"
---
### 17.5.2 Change Buffer

The change buffer is a special data structure that caches changes to [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index "secondary index") pages when those pages are not in the [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool "buffer pool"). The buffered changes, which may result from [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement"), [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement"), or [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement") operations (DML), are merged later when the pages are loaded into the buffer pool by other read operations.

**Figure 17.3 Change Buffer**

![Content is described in the surrounding text.](https://dev.mysql.com/doc/refman/9.7/en/images/innodb-change-buffer.png)  

Unlike [clustered indexes](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clustered_index "clustered index"), secondary indexes are usually nonunique, and inserts into secondary indexes happen in a relatively random order. Similarly, deletes and updates may affect secondary index pages that are not adjacently located in an index tree. Merging cached changes at a later time, when affected pages are read into the buffer pool by other operations, avoids substantial random access I/O that would be required to read secondary index pages into the buffer pool from disk.

Periodically, the purge operation that runs when the system is mostly idle, or during a slow shutdown, writes the updated index pages to disk. The purge operation can write disk blocks for a series of index values more efficiently than if each value were written to disk immediately.

Change buffer merging may take several hours when there are many affected rows and numerous secondary indexes to update. During this time, disk I/O is increased, which can cause a significant slowdown for disk-bound queries. Change buffer merging may also continue to occur after a transaction is committed, and even after a server shutdown and restart (see [Section 17.20.3, “Forcing InnoDB Recovery”](https://dev.mysql.com/doc/refman/9.7/en/forcing-innodb-recovery.html "17.20.3 Forcing InnoDB Recovery") for more information).

In memory, the change buffer occupies part of the buffer pool. On disk, the change buffer is part of the system tablespace, where index changes are buffered when the database server is shut down.

The type of data cached in the change buffer is governed by the [`innodb_change_buffering`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffering) variable. For more information, see [Configuring Change Buffering](https://dev.mysql.com/doc/refman/9.7/en/innodb-change-buffer.html#innodb-change-buffer-configuration "Configuring Change Buffering"). You can also configure the maximum change buffer size. For more information, see [Configuring the Change Buffer Maximum Size](https://dev.mysql.com/doc/refman/9.7/en/innodb-change-buffer.html#innodb-change-buffer-maximum-size "Configuring the Change Buffer Maximum Size").

Change buffering is not supported for a secondary index if the index contains a descending index column or if the primary key includes a descending index column.

For answers to frequently asked questions about the change buffer, see [Section A.16, “MySQL 9.7 FAQ: InnoDB Change Buffer”](https://dev.mysql.com/doc/refman/9.7/en/faqs-innodb-change-buffer.html "A.16 MySQL 9.7 FAQ: InnoDB Change Buffer").

#### Configuring Change Buffering

When [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement"), [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement"), and [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement") operations are performed on a table, the values of indexed columns (particularly the values of secondary keys) are often in an unsorted order, requiring substantial I/O to bring secondary indexes up to date. The [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer "change buffer") caches changes to secondary index entries when the relevant [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page "page") is not in the [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool "buffer pool"), thus avoiding expensive I/O operations by not immediately reading in the page from disk. The buffered changes are merged when the page is loaded into the buffer pool, and the updated page is later flushed to disk. The `InnoDB` main thread merges buffered changes when the server is nearly idle, and during a [slow shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_slow_shutdown "slow shutdown").

Because it can result in fewer disk reads and writes, change buffering is most valuable for workloads that are I/O-bound; for example, applications with a high volume of DML operations such as bulk inserts benefit from change buffering.

However, the change buffer occupies a part of the buffer pool, reducing the memory available to cache data pages. If the working set almost fits in the buffer pool, or if your tables have relatively few secondary indexes, it may be useful to disable change buffering. If the working data set fits entirely within the buffer pool, change buffering does not impose extra overhead, because it only applies to pages that are not in the buffer pool.

The [`innodb_change_buffering`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffering) variable controls the extent to which `InnoDB` performs change buffering. You can enable or disable buffering for inserts, delete operations (when index records are initially marked for deletion) and purge operations (when index records are physically deleted). An update operation is a combination of an insert and a delete. The default [`innodb_change_buffering`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffering) value is `none`, and permitted values are described within the [`innodb_change_buffering`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffering) documentation.

You can set the [`innodb_change_buffering`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffering) variable in the MySQL option file (`my.cnf` or `my.ini`) or change it dynamically with the [`SET GLOBAL`](https://dev.mysql.com/doc/refman/9.7/en/set-variable.html "15.7.6.1 SET Syntax for Variable Assignment") statement, which requires privileges sufficient to set global system variables. See [Section 7.1.9.1, “System Variable Privileges”](https://dev.mysql.com/doc/refman/9.7/en/system-variable-privileges.html "7.1.9.1 System Variable Privileges"). Changing the setting affects the buffering of new operations; the merging of existing buffered entries is not affected.

#### Configuring the Change Buffer Maximum Size

The [`innodb_change_buffer_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffer_max_size) variable permits configuring the maximum size of the change buffer as a percentage of the total size of the buffer pool. By default, [`innodb_change_buffer_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffer_max_size) is set to 5. The maximum setting is 50.

Consider increasing [`innodb_change_buffer_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffer_max_size) on a MySQL server with heavy insert, update, and delete activity, where change buffer merging does not keep pace with new change buffer entries, causing the change buffer to reach its maximum size limit.

Consider decreasing [`innodb_change_buffer_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffer_max_size) on a MySQL server with static data used for reporting, or if the change buffer consumes too much of the memory space shared with the buffer pool, causing pages to age out of the buffer pool sooner than desired.

Test different settings with a representative workload to determine an optimal configuration. The [`innodb_change_buffer_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffer_max_size) variable is dynamic, which permits modifying the setting without restarting the server.

#### Monitoring the Change Buffer

The following options are available for change buffer monitoring:

- `InnoDB` Standard Monitor output includes change buffer status information. To view monitor data, issue the `SHOW ENGINE INNODB STATUS` statement.
	```sql
	Press CTRL+C to copy mysql> SHOW ENGINE INNODB STATUS\G
	```
	Change buffer status information is located under the `INSERT BUFFER AND ADAPTIVE HASH INDEX` heading and appears similar to the following:
	```sql
	Press CTRL+C to copy -------------------------------------
	INSERT BUFFER AND ADAPTIVE HASH INDEX
	-------------------------------------
	Ibuf: size 1, free list len 0, seg size 2, 0 merges
	merged operations:
	 insert 0, delete mark 0, delete 0
	discarded operations:
	 insert 0, delete mark 0, delete 0
	Hash table size 4425293, used cells 32, node heap has 1 buffer(s)
	13577.57 hash searches/s, 202.47 non-hash searches/s
	```
	For more information, see [Section 17.17.3, “InnoDB Standard Monitor and Lock Monitor Output”](https://dev.mysql.com/doc/refman/9.7/en/innodb-standard-monitor.html "17.17.3 InnoDB Standard Monitor and Lock Monitor Output").
- The Information Schema [`INNODB_METRICS`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-metrics-table.html "28.4.21 The INFORMATION_SCHEMA INNODB_METRICS Table") table provides most of the data points found in `InnoDB` Standard Monitor output plus other data points. To view change buffer metrics and a description of each, issue the following query:
	```sql
	Press CTRL+C to copy mysql> SELECT NAME, COMMENT FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME LIKE '%ibuf%'\G
	```
	See [Section 17.15.6, “InnoDB INFORMATION\_SCHEMA Metrics Table”](https://dev.mysql.com/doc/refman/9.7/en/innodb-information-schema-metrics-table.html "17.15.6 InnoDB INFORMATION_SCHEMA Metrics Table").
- The Information Schema [`INNODB_BUFFER_PAGE`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-buffer-page-table.html "28.4.2 The INFORMATION_SCHEMA INNODB_BUFFER_PAGE Table") table provides metadata about each page in the buffer pool, including change buffer index and change buffer bitmap pages. Change buffer pages are identified by `PAGE_TYPE`. `IBUF_INDEX` is the page type for change buffer index pages, and `IBUF_BITMAP` is the page type for change buffer bitmap pages.
	Warning
	Querying the [`INNODB_BUFFER_PAGE`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-buffer-page-table.html "28.4.2 The INFORMATION_SCHEMA INNODB_BUFFER_PAGE Table") table can introduce significant performance overhead. To avoid impacting performance, reproduce the issue you want to investigate on a test instance and run your queries on the test instance.
	For example, you can query the [`INNODB_BUFFER_PAGE`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-buffer-page-table.html "28.4.2 The INFORMATION_SCHEMA INNODB_BUFFER_PAGE Table") table to determine the approximate number of `IBUF_INDEX` and `IBUF_BITMAP` pages as a percentage of total buffer pool pages.
	```sql
	Press CTRL+C to copy mysql> SELECT (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
	       WHERE PAGE_TYPE LIKE 'IBUF%') AS change_buffer_pages,
	       (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE) AS total_pages,
	       (SELECT ((change_buffer_pages/total_pages)*100))
	       AS change_buffer_page_percentage;
	+---------------------+-------------+-------------------------------+
	| change_buffer_pages | total_pages | change_buffer_page_percentage |
	+---------------------+-------------+-------------------------------+
	|                  25 |        8192 |                        0.3052 |
	+---------------------+-------------+-------------------------------+
	```
	For information about other data provided by the [`INNODB_BUFFER_PAGE`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-buffer-page-table.html "28.4.2 The INFORMATION_SCHEMA INNODB_BUFFER_PAGE Table") table, see [Section 28.4.2, “The INFORMATION\_SCHEMA INNODB\_BUFFER\_PAGE Table”](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-buffer-page-table.html "28.4.2 The INFORMATION_SCHEMA INNODB_BUFFER_PAGE Table"). For related usage information, see [Section 17.15.5, “InnoDB INFORMATION\_SCHEMA Buffer Pool Tables”](https://dev.mysql.com/doc/refman/9.7/en/innodb-information-schema-buffer-pool-tables.html "17.15.5 InnoDB INFORMATION_SCHEMA Buffer Pool Tables").
- [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/performance-schema.html "Chapter 29 MySQL Performance Schema") provides change buffer mutex wait instrumentation for advanced performance monitoring. To view change buffer instrumentation, issue the following query:
	```sql
	Press CTRL+C to copy mysql> SELECT * FROM performance_schema.setup_instruments
	       WHERE NAME LIKE '%wait/synch/mutex/innodb/ibuf%';
	+-------------------------------------------------------+---------+-------+
	| NAME                                                  | ENABLED | TIMED |
	+-------------------------------------------------------+---------+-------+
	| wait/synch/mutex/innodb/ibuf_bitmap_mutex             | YES     | YES   |
	| wait/synch/mutex/innodb/ibuf_mutex                    | YES     | YES   |
	| wait/synch/mutex/innodb/ibuf_pessimistic_insert_mutex | YES     | YES   |
	+-------------------------------------------------------+---------+-------+
	```
	For information about monitoring `InnoDB` mutex waits, see [Section 17.16.2, “Monitoring InnoDB Mutex Waits Using Performance Schema”](https://dev.mysql.com/doc/refman/9.7/en/monitor-innodb-mutex-waits-performance-schema.html "17.16.2 Monitoring InnoDB Mutex Waits Using Performance Schema").