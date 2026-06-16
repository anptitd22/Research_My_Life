---
title: "MySQL :: MySQL 8.4 Reference Manual :: 26.6 Restrictions and Limitations on Partitioning"
source: "https://dev.mysql.com/doc/refman/8.4/en/partitioning-limitations.html"
author:
published:
created: 2026-06-15
description:
tags:
  - "clippings"
---
## 26.6 Restrictions and Limitations on Partitioning

This section discusses current restrictions and limitations on MySQL partitioning support.

**Prohibited constructs.** The following constructs are not permitted in partitioning expressions:

- Stored procedures, stored functions, loadable functions, or plugins.
- Declared variables or user variables.

For a list of SQL functions which are permitted in partitioning expressions, see [Section 26.6.3, “Partitioning Limitations Relating to Functions”](https://dev.mysql.com/doc/refman/8.4/en/partitioning-limitations-functions.html "26.6.3 Partitioning Limitations Relating to Functions").

**Arithmetic and logical operators.** Use of the arithmetic operators [`+`](https://dev.mysql.com/doc/refman/8.4/en/arithmetic-functions.html#operator_plus), [`-`](https://dev.mysql.com/doc/refman/8.4/en/arithmetic-functions.html#operator_minus), and [`*`](https://dev.mysql.com/doc/refman/8.4/en/arithmetic-functions.html#operator_times) is permitted in partitioning expressions. However, the result must be an integer value or `NULL` (except in the case of `[LINEAR] KEY` partitioning, as discussed elsewhere in this chapter; see [Section 26.2, “Partitioning Types”](https://dev.mysql.com/doc/refman/8.4/en/partitioning-types.html "26.2 Partitioning Types"), for more information).

The [`DIV`](https://dev.mysql.com/doc/refman/8.4/en/arithmetic-functions.html#operator_div) operator is also supported; the [`/`](https://dev.mysql.com/doc/refman/8.4/en/arithmetic-functions.html#operator_divide) operator is not permitted.

The bit operators [`|`](https://dev.mysql.com/doc/refman/8.4/en/bit-functions.html#operator_bitwise-or), [`&`](https://dev.mysql.com/doc/refman/8.4/en/bit-functions.html#operator_bitwise-and), [`^`](https://dev.mysql.com/doc/refman/8.4/en/bit-functions.html#operator_bitwise-xor), [`<<`](https://dev.mysql.com/doc/refman/8.4/en/bit-functions.html#operator_left-shift), [`>>`](https://dev.mysql.com/doc/refman/8.4/en/bit-functions.html#operator_right-shift), and [`~`](https://dev.mysql.com/doc/refman/8.4/en/bit-functions.html#operator_bitwise-invert) are not permitted in partitioning expressions.

**Server SQL mode.** Tables employing user-defined partitioning do not preserve the SQL mode in effect at the time that they were created. As discussed elsewhere in this Manual (see [Section 7.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/8.4/en/sql-mode.html "7.1.11 Server SQL Modes")), the results of many MySQL functions and operators may change according to the server SQL mode. Therefore, a change in the SQL mode at any time after the creation of partitioned tables may lead to major changes in the behavior of such tables, and could easily lead to corruption or loss of data. For these reasons, *it is strongly recommended that you never change the server SQL mode after creating partitioned tables*.

For one such change in the server SQL mode making a partitioned tables unusable, consider the following [`CREATE TABLE`](https://dev.mysql.com/doc/refman/8.4/en/create-table.html "15.1.20 CREATE TABLE Statement") statement, which can be executed successfully only if the [`NO_UNSIGNED_SUBTRACTION`](https://dev.mysql.com/doc/refman/8.4/en/sql-mode.html#sqlmode_no_unsigned_subtraction) mode is in effect:

```sql
Press CTRL+C to copy mysql> SELECT @@sql_mode;
+------------+
| @@sql_mode |
+------------+
|            |
+------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE tu (c1 BIGINT UNSIGNED)
    ->   PARTITION BY RANGE(c1 - 10) (
    ->     PARTITION p0 VALUES LESS THAN (-5),
    ->     PARTITION p1 VALUES LESS THAN (0),
    ->     PARTITION p2 VALUES LESS THAN (5),
    ->     PARTITION p3 VALUES LESS THAN (10),
    ->     PARTITION p4 VALUES LESS THAN (MAXVALUE)
    -> );
ERROR 1563 (HY000): Partition constant is out of partition function domain

mysql> SET sql_mode='NO_UNSIGNED_SUBTRACTION';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@sql_mode;
+-------------------------+
| @@sql_mode              |
+-------------------------+
| NO_UNSIGNED_SUBTRACTION |
+-------------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE tu (c1 BIGINT UNSIGNED)
    ->   PARTITION BY RANGE(c1 - 10) (
    ->     PARTITION p0 VALUES LESS THAN (-5),
    ->     PARTITION p1 VALUES LESS THAN (0),
    ->     PARTITION p2 VALUES LESS THAN (5),
    ->     PARTITION p3 VALUES LESS THAN (10),
    ->     PARTITION p4 VALUES LESS THAN (MAXVALUE)
    -> );
Query OK, 0 rows affected (0.05 sec)
```

If you remove the [`NO_UNSIGNED_SUBTRACTION`](https://dev.mysql.com/doc/refman/8.4/en/sql-mode.html#sqlmode_no_unsigned_subtraction) server SQL mode after creating `tu`, you may no longer be able to access this table:

```sql
Press CTRL+C to copy mysql> SET sql_mode='';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM tu;
ERROR 1563 (HY000): Partition constant is out of partition function domain
mysql> INSERT INTO tu VALUES (20);
ERROR 1563 (HY000): Partition constant is out of partition function domain
```

See also [Section 7.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/8.4/en/sql-mode.html "7.1.11 Server SQL Modes").

Server SQL modes also impact replication of partitioned tables. Disparate SQL modes on source and replica can lead to partitioning expressions being evaluated differently; this can cause the distribution of data among partitions to be different in the source's and replica's copies of a given table, and may even cause inserts into partitioned tables that succeed on the source to fail on the replica. For best results, you should always use the same server SQL mode on the source and on the replica.

**Performance considerations.** Some effects of partitioning operations on performance are given in the following list:

- **File system operations.** Partitioning and repartitioning operations (such as [`ALTER             TABLE`](https://dev.mysql.com/doc/refman/8.4/en/alter-table-partition-operations.html "15.1.9.1 ALTER TABLE Partition Operations") with `PARTITION BY ...`, `REORGANIZE PARTITION`, or `REMOVE             PARTITIONING`) depend on file system operations for their implementation. This means that the speed of these operations is affected by such factors as file system type and characteristics, disk speed, swap space, file handling efficiency of the operating system, and MySQL server options and variables that relate to file handling. In particular, you should make sure that [`large_files_support`](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_large_files_support) is enabled and that [`open_files_limit`](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_open_files_limit) is set properly. Partitioning and repartitioning operations involving `InnoDB` tables may be made more efficient by enabling [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_file_per_table).
	See also [Maximum number of partitions](https://dev.mysql.com/doc/refman/8.4/en/partitioning-limitations.html#partitioning-limitations-max-partitions "Maximum number of partitions").
- **Table locks.** Generally, the process executing a partitioning operation on a table takes a write lock on the table. Reads from such tables are relatively unaffected; pending [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") and [`UPDATE`](https://dev.mysql.com/doc/refman/8.4/en/update.html "15.2.17 UPDATE Statement") operations are performed as soon as the partitioning operation has completed. For `InnoDB` -specific exceptions to this limitation, see [Partitioning Operations](https://dev.mysql.com/doc/refman/8.4/en/innodb-online-ddl-operations.html#online-ddl-partitioning "Partitioning Operations").
- **Indexes; partition pruning.** As with nonpartitioned tables, proper use of indexes can speed up queries on partitioned tables significantly. In addition, designing partitioned tables and queries on these tables to take advantage of partition pruning can improve performance dramatically. See [Section 26.4, “Partition Pruning”](https://dev.mysql.com/doc/refman/8.4/en/partitioning-pruning.html "26.4 Partition Pruning"), for more information.
	Index condition pushdown is supported for partitioned tables. See [Section 10.2.1.6, “Index Condition Pushdown Optimization”](https://dev.mysql.com/doc/refman/8.4/en/index-condition-pushdown-optimization.html "10.2.1.6 Index Condition Pushdown Optimization").
- **Performance with LOAD DATA.** In MySQL 8.4, [`LOAD             DATA`](https://dev.mysql.com/doc/refman/8.4/en/load-data.html "15.2.9 LOAD DATA Statement") uses buffering to improve performance. You should be aware that the buffer uses 130 KB memory per partition to achieve this.

**Maximum number of partitions.** The maximum possible number of partitions for a given table not using the [`NDB`](https://dev.mysql.com/doc/refman/8.4/en/mysql-cluster.html "Chapter 25 MySQL NDB Cluster 8.4") storage engine is 8192. This number includes subpartitions.

The maximum possible number of user-defined partitions for a table using the [`NDB`](https://dev.mysql.com/doc/refman/8.4/en/mysql-cluster.html "Chapter 25 MySQL NDB Cluster 8.4") storage engine is determined according to the version of the NDB Cluster software being used, the number of data nodes, and other factors. See [NDB and user-defined partitioning](https://dev.mysql.com/doc/refman/8.4/en/mysql-cluster-nodes-groups.html#mysql-cluster-nodes-groups-user-partitioning "NDB and user-defined partitioning"), for more information.

If, when creating tables with a large number of partitions (but less than the maximum), you encounter an error message such as Got error... from storage engine: Out of resources when opening file, you may be able to address the issue by increasing the value of the [`open_files_limit`](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_open_files_limit) system variable. However, this is dependent on the operating system, and may not be possible or advisable on all platforms; see [Section B.3.2.16, “File Not Found and Similar Errors”](https://dev.mysql.com/doc/refman/8.4/en/not-enough-file-handles.html "B.3.2.16 File Not Found and Similar Errors"), for more information. In some cases, using large numbers (hundreds) of partitions may also not be advisable due to other concerns, so using more partitions does not automatically lead to better results.

See also [File system operations](https://dev.mysql.com/doc/refman/8.4/en/partitioning-limitations.html#partitioning-limitations-file-system-ops "File system operations").

**Foreign keys not supported for partitioned InnoDB tables.** Partitioned tables using the [`InnoDB`](https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") storage engine do not support foreign keys. More specifically, this means that the following two statements are true:

1. No definition of an `InnoDB` table employing user-defined partitioning may contain foreign key references; no `InnoDB` table whose definition contains foreign key references may be partitioned.
2. No `InnoDB` table definition may contain a foreign key reference to a user-partitioned table; no `InnoDB` table with user-defined partitioning may contain columns referenced by foreign keys.

The scope of the restrictions just listed includes all tables that use the `InnoDB` storage engine. [`CREATE       TABLE`](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html "15.1.20.5 FOREIGN KEY Constraints") and [`ALTER TABLE`](https://dev.mysql.com/doc/refman/8.4/en/alter-table.html "15.1.9 ALTER TABLE Statement") statements that would result in tables violating these restrictions are not allowed.

**ALTER TABLE... ORDER BY.** An ``ALTER TABLE ... ORDER BY         *`column`*`` statement run against a partitioned table causes ordering of rows only within each partition.

**ADD COLUMN... ALGORITHM=INSTANT.** Once you perform [`ALTER TABLE ... ADD         COLUMN ... ALGORITHM=INSTANT`](https://dev.mysql.com/doc/refman/8.4/en/alter-table.html "15.1.9 ALTER TABLE Statement") on a partitioned table, it is no longer possible to exchange partitions with this table.

**Effects on REPLACE statements by modification of primary keys.** It can be desirable in some cases (see [Section 26.6.1, “Partitioning Keys, Primary Keys, and Unique Keys”](https://dev.mysql.com/doc/refman/8.4/en/partitioning-limitations-partitioning-keys-unique-keys.html "26.6.1 Partitioning Keys, Primary Keys, and Unique Keys")) to modify a table's primary key. Be aware that, if your application uses [`REPLACE`](https://dev.mysql.com/doc/refman/8.4/en/replace.html "15.2.12 REPLACE Statement") statements and you do this, the results of these statements can be drastically altered. See [Section 15.2.12, “REPLACE Statement”](https://dev.mysql.com/doc/refman/8.4/en/replace.html "15.2.12 REPLACE Statement"), for more information and an example.

**FULLTEXT indexes.** Partitioned tables do not support `FULLTEXT` indexes or searches.

**Spatial columns.** Columns with spatial data types such as `POINT` or `GEOMETRY` cannot be used in partitioned tables.

**Temporary tables.** Temporary tables cannot be partitioned.

**Log tables.** It is not possible to partition the log tables; an [`ALTER         TABLE ... PARTITION BY ...`](https://dev.mysql.com/doc/refman/8.4/en/alter-table-partition-operations.html "15.1.9.1 ALTER TABLE Partition Operations") statement on such a table fails with an error.

**Data type of partitioning key.** A partitioning key must be either an integer column or an expression that resolves to an integer. Expressions employing [`ENUM`](https://dev.mysql.com/doc/refman/8.4/en/enum.html "13.3.5 The ENUM Type") columns cannot be used. The column or expression value may also be `NULL`; see [Section 26.2.7, “How MySQL Partitioning Handles NULL”](https://dev.mysql.com/doc/refman/8.4/en/partitioning-handling-nulls.html "26.2.7 How MySQL Partitioning Handles NULL").

There are two exceptions to this restriction:

1. When partitioning by \[`LINEAR`\] `KEY`, it is possible to use columns of any valid MySQL data type other than [`TEXT`](https://dev.mysql.com/doc/refman/8.4/en/blob.html "13.3.4 The BLOB and TEXT Types") or [`BLOB`](https://dev.mysql.com/doc/refman/8.4/en/blob.html "13.3.4 The BLOB and TEXT Types") as partitioning keys, because the internal key-hashing functions produce the correct data type from these types. For example, the following two [`CREATE TABLE`](https://dev.mysql.com/doc/refman/8.4/en/create-table.html "15.1.20 CREATE TABLE Statement") statements are valid:
	```sql
	Press CTRL+C to copy CREATE TABLE tkc (c1 CHAR)
	PARTITION BY KEY(c1)
	PARTITIONS 4;
	CREATE TABLE tke
	    ( c1 ENUM('red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet') )
	PARTITION BY LINEAR KEY(c1)
	PARTITIONS 6;
	```
2. When partitioning by `RANGE COLUMNS` or `LIST COLUMNS`, it is possible to use string, [`DATE`](https://dev.mysql.com/doc/refman/8.4/en/datetime.html "13.2.2 The DATE, DATETIME, and TIMESTAMP Types"), and [`DATETIME`](https://dev.mysql.com/doc/refman/8.4/en/datetime.html "13.2.2 The DATE, DATETIME, and TIMESTAMP Types") columns. For example, each of the following [`CREATE           TABLE`](https://dev.mysql.com/doc/refman/8.4/en/create-table.html "15.1.20 CREATE TABLE Statement") statements is valid:
	```sql
	Press CTRL+C to copy CREATE TABLE rc (c1 INT, c2 DATE)
	PARTITION BY RANGE COLUMNS(c2) (
	    PARTITION p0 VALUES LESS THAN('1990-01-01'),
	    PARTITION p1 VALUES LESS THAN('1995-01-01'),
	    PARTITION p2 VALUES LESS THAN('2000-01-01'),
	    PARTITION p3 VALUES LESS THAN('2005-01-01'),
	    PARTITION p4 VALUES LESS THAN(MAXVALUE)
	);
	CREATE TABLE lc (c1 INT, c2 CHAR(1))
	PARTITION BY LIST COLUMNS(c2) (
	    PARTITION p0 VALUES IN('a', 'd', 'g', 'j', 'm', 'p', 's', 'v', 'y'),
	    PARTITION p1 VALUES IN('b', 'e', 'h', 'k', 'n', 'q', 't', 'w', 'z'),
	    PARTITION p2 VALUES IN('c', 'f', 'i', 'l', 'o', 'r', 'u', 'x', NULL)
	);
	```

Neither of the preceding exceptions applies to [`BLOB`](https://dev.mysql.com/doc/refman/8.4/en/blob.html "13.3.4 The BLOB and TEXT Types") or [`TEXT`](https://dev.mysql.com/doc/refman/8.4/en/blob.html "13.3.4 The BLOB and TEXT Types") column types.

**Subqueries.** A partitioning key may not be a subquery, even if that subquery resolves to an integer value or `NULL`.

**Column index prefixes not supported for key partitioning.** When creating a table that is partitioned by key, any columns in the partitioning key which use column prefixes are not allowed in the table's partitioning function. Consider the following [`CREATE TABLE`](https://dev.mysql.com/doc/refman/8.4/en/create-table.html "15.1.20 CREATE TABLE Statement") statement, which has three [`VARCHAR`](https://dev.mysql.com/doc/refman/8.4/en/char.html "13.3.2 The CHAR and VARCHAR Types") columns, and whose primary key uses all three columns and specifies a prefix for one of them. This statement is rejected with an error, as shown here:

```sql
Press CTRL+C to copy mysql> USE d;
Database changed
mysql> CREATE TABLE t1 (
    ->     a VARCHAR(10000),
    ->     b VARCHAR(25),
    ->     c VARCHAR(10),
    ->     PRIMARY KEY (a(10), b, c)
    -> ) PARTITION BY KEY() PARTITIONS 2;
ERROR 6123 (HY000): Column 'd.t1.a' having prefix key part 'a(10)' in the
PARTITION BY KEY() clause is not supported.
```

For general information about partitioning tables by key, see [Section 26.2.5, “KEY Partitioning”](https://dev.mysql.com/doc/refman/8.4/en/partitioning-key.html "26.2.5 KEY Partitioning").

**Issues with subpartitions.** Subpartitions must use `HASH` or `KEY` partitioning. Only `RANGE` and `LIST` partitions may be subpartitioned; `HASH` and `KEY` partitions cannot be subpartitioned.

`SUBPARTITION BY KEY` requires that the subpartitioning column or columns be specified explicitly, unlike the case with `PARTITION BY KEY`, where it can be omitted (in which case the table's primary key column is used by default). Consider the table created by this statement:

```sql
Press CTRL+C to copy CREATE TABLE ts (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30)
);
```

You can create a table having the same columns, partitioned by `KEY`, using a statement such as this one:

```sql
Press CTRL+C to copy CREATE TABLE ts (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30)
)
PARTITION BY KEY()
PARTITIONS 4;
```

The previous statement is treated as though it had been written like this, with the table's primary key column used as the partitioning column:

```sql
Press CTRL+C to copy CREATE TABLE ts (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30)
)
PARTITION BY KEY(id)
PARTITIONS 4;
```

However, the following statement that attempts to create a subpartitioned table using the default column as the subpartitioning column fails, and the column must be specified for the statement to succeed, as shown here:

```sql
mysql> CREATE TABLE ts (
    ->     id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    ->     name VARCHAR(30)
    -> )
    -> PARTITION BY RANGE(id)
    -> SUBPARTITION BY KEY()
    -> SUBPARTITIONS 4
    -> (
    ->     PARTITION p0 VALUES LESS THAN (100),
    ->     PARTITION p1 VALUES LESS THAN (MAXVALUE)
    -> );
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that
corresponds to your MySQL server version for the right syntax to use near ')

mysql> CREATE TABLE ts (
    ->     id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    ->     name VARCHAR(30)
    -> )
    -> PARTITION BY RANGE(id)
    -> SUBPARTITION BY KEY(id)
    -> SUBPARTITIONS 4
    -> (
    ->     PARTITION p0 VALUES LESS THAN (100),
    ->     PARTITION p1 VALUES LESS THAN (MAXVALUE)
    -> );
Query OK, 0 rows affected (0.07 sec)
```

This is a known issue (see Bug #51470).

**DATA DIRECTORY and INDEX DIRECTORY options.** Table-level `DATA DIRECTORY` and `INDEX         DIRECTORY` options are ignored (see Bug #32091). You can employ these options for individual partitions or subpartitions of [`InnoDB`](https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") tables. The directory specified in a `DATA DIRECTORY` clause must be known to [`InnoDB`](https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine"). For more information, see [Using the DATA DIRECTORY Clause](https://dev.mysql.com/doc/refman/8.4/en/innodb-create-table-external.html#innodb-create-table-external-data-directory "Using the DATA DIRECTORY Clause").

**Repairing and rebuilding partitioned tables.** The statements [`CHECK TABLE`](https://dev.mysql.com/doc/refman/8.4/en/check-table.html "15.7.3.2 CHECK TABLE Statement"), [`OPTIMIZE TABLE`](https://dev.mysql.com/doc/refman/8.4/en/optimize-table.html "15.7.3.4 OPTIMIZE TABLE Statement"), [`ANALYZE TABLE`](https://dev.mysql.com/doc/refman/8.4/en/analyze-table.html "15.7.3.1 ANALYZE TABLE Statement"), and [`REPAIR TABLE`](https://dev.mysql.com/doc/refman/8.4/en/repair-table.html "15.7.3.5 REPAIR TABLE Statement") are supported for partitioned tables.

In addition, you can use `ALTER TABLE ... REBUILD       PARTITION` to rebuild one or more partitions of a partitioned table; `ALTER TABLE ... REORGANIZE       PARTITION` also causes partitions to be rebuilt. See [Section 15.1.9, “ALTER TABLE Statement”](https://dev.mysql.com/doc/refman/8.4/en/alter-table.html "15.1.9 ALTER TABLE Statement"), for more information about these two statements.

`ANALYZE`, `CHECK`, `OPTIMIZE`, `REPAIR`, and `TRUNCATE` operations are supported with subpartitions. See [Section 15.1.9.1, “ALTER TABLE Partition Operations”](https://dev.mysql.com/doc/refman/8.4/en/alter-table-partition-operations.html "15.1.9.1 ALTER TABLE Partition Operations").

**File name delimiters for partitions and subpartitions.** Table partition and subpartition file names include generated delimiters such as `#P#` and `#SP#`. The lettercase of such delimiters can vary and should not be depended upon.