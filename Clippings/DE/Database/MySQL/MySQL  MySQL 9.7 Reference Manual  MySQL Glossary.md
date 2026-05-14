---
title: "MySQL :: MySQL 9.7 Reference Manual :: MySQL Glossary"
source: "https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool"
author:
published:
created: 2026-05-13
description:
tags:
  - "clippings"
---
## MySQL Glossary

These terms are commonly used in information about the MySQL database server.

### A

.ARM file

Metadata for `ARCHIVE` tables. Contrast with **.ARZ file**. Files with this extension are always included in backups produced by the `mysqlbackup` command of the **MySQL Enterprise Backup** product.

See Also [.ARZ file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_arz_file), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command).

.ARZ file

Data for ARCHIVE tables. Contrast with **.ARM file**. Files with this extension are always included in backups produced by the `mysqlbackup` command of the **MySQL Enterprise Backup** product.

See Also [.ARM file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_arm_file), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command).

ACID

An acronym standing for atomicity, consistency, isolation, and durability. These properties are all desirable in a database system, and are all closely tied to the notion of a **transaction**. The transactional features of `InnoDB` adhere to the ACID principles.

Transactions are **atomic** units of work that can be **committed** or **rolled back**. When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is committed, or all the changes are undone when the transaction is rolled back.

The database remains in a consistent state at all times — after each commit or rollback, and while transactions are in progress. If related data is being updated across multiple tables, queries see either all old values or all new values, not a mix of old and new values.

Transactions are protected (isolated) from each other while they are in progress; they cannot interfere with each other or see each other's uncommitted data. This isolation is achieved through the **locking** mechanism. Experienced users can adjust the **isolation level**, trading off less protection in favor of increased performance and **concurrency**, when they can be sure that the transactions really do not interfere with each other.

The results of transactions are durable: once a commit operation succeeds, the changes made by that transaction are safe from power failures, system crashes, race conditions, or other potential dangers that many non-database applications are vulnerable to. Durability typically involves writing to disk storage, with a certain amount of redundancy to protect against power failures or software crashes during write operations. (In `InnoDB`, the **doublewrite buffer** assists with durability.)

See Also [atomic](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_atomic), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [doublewrite buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_doublewrite_buffer), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

adaptive flushing

An algorithm for **InnoDB** tables that smooths out the I/O overhead introduced by **checkpoints**. Instead of **flushing** all modified **pages** from the **buffer pool** to the **data files** at once, MySQL periodically flushes small sets of modified pages. The adaptive flushing algorithm extends this process by estimating the optimal rate to perform these periodic flushes, based on the rate of flushing and how fast **redo** information is generated.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [checkpoint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_checkpoint), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log).

adaptive hash index

An optimization for `InnoDB` tables that can speed up lookups using `=` and `IN` operators, by constructing a **hash index** in memory. MySQL monitors index searches for `InnoDB` tables, and if queries could benefit from a hash index, it builds one automatically for index **pages** that are frequently accessed. In a sense, the adaptive hash index configures MySQL at runtime to take advantage of ample main memory, coming closer to the architecture of main-memory databases. This feature is controlled by the [`innodb_adaptive_hash_index`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_adaptive_hash_index) configuration option. Because this feature benefits some workloads and not others, and the memory used for the hash index is reserved in the **buffer pool**, typically you should benchmark with this feature both enabled and disabled.

The hash index is always built based on an existing **B-tree** index on the table. MySQL can build a hash index on a prefix of any length of the key defined for the B-tree, depending on the pattern of searches against the index. A hash index can be partial; the whole B-tree index does not need to be cached in the buffer pool.

See Also [B-tree](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_b_tree), [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [hash index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hash_index), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index).

ADO.NET

An object-relational mapping (ORM) framework for applications built using.NET technologies such as **ASP.NET**. Such applications can interface with MySQL through the **Connector/NET** component.

See Also [.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos__net), [ASP.net](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_asp_net), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [Mono](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mono), [Visual Studio](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_visual_studio).

AIO

Acronym for **asynchronous I/O**. You might see this acronym in `InnoDB` messages or keywords.

See Also [asynchronous I/O](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_asynchronous_io).

ANSI

In **ODBC**, an alternative method of supporting character sets and other internationalization aspects. Contrast with **Unicode**. **Connector/ODBC** 3.51 is an ANSI driver, while Connector/ODBC 5.1 is a Unicode driver.

See Also [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc), [ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_odbc), [Unicode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unicode).

API

APIs provide low-level access to the MySQL protocol and MySQL resources from **client** programs. Contrast with the higher-level access provided by a **Connector**.

See Also [C API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_c_api), [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [native C API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_native_c_api), [Perl API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_perl_api), [PHP API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_php_api), [Python API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_python_api), [Ruby API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ruby_api).

application programming interface (API)

A set of functions or procedures. An API provides a stable set of names and types for functions, procedures, parameters, and return values.

apply

When a backup produced by the **MySQL Enterprise Backup** product does not include the most recent changes that occurred while the backup was underway, the process of updating the backup files to include those changes is known as the **apply** step. It is specified by the `apply-log` option of the `mysqlbackup` command.

Before the changes are applied, we refer to the files as a **raw backup**. After the changes are applied, we refer to the files as a **prepared backup**. The changes are recorded in the **ibbackup\_logfile** file; once the apply step is finished, this file is no longer necessary.

See Also [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [ibbackup\_logfile](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ibbackup_logfile), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [prepared backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_backup), [raw backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_raw_backup).

ASP.net

A framework for developing web-based applications using **.NET** technologies and languages. Such applications can interface with MySQL through the **Connector/NET** component.

Another technology for writing server-side web pages with MySQL is **PHP**.

See Also [.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos__net), [ADO.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ado_net), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [Mono](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mono), [PHP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_php), [Visual Studio](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_visual_studio).

assembly

A library of compiled code in a **.NET** system, accessed through **Connector/NET**. Stored in the **GAC** to allow versioning without naming conflicts.

See Also [.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos__net), [GAC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gac).

asynchronous I/O

A type of I/O operation that allows other processing to proceed before the I/O is completed. Also known as **nonblocking I/O** and abbreviated as **AIO**. `InnoDB` uses this type of I/O for certain operations that can run in parallel without affecting the reliability of the database, such as reading pages into the **buffer pool** that have not actually been requested, but might be needed soon.

Historically, `InnoDB` used asynchronous I/O on Windows systems only. Starting with the InnoDB Plugin 1.1 and MySQL 5.5, `InnoDB ` uses asynchronous I/O on Linux systems. This change introduces a dependency on `libaio`. Asynchronous I/O on Linux systems is configured using the [`innodb_use_native_aio`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_use_native_aio) option, which is enabled by default. On other Unix-like systems, InnoDB uses synchronous I/O only.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [nonblocking I/O](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_non_blocking_io).

atomic

In the SQL context, **transactions** are units of work that either succeed entirely (when **committed**) or have no effect at all (when **rolled back**). The indivisible ("atomic") property of transactions is the “A” in the acronym **ACID**.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

atomic DDL

An atomic *DDL* statement is one that combines the *data dictionary* updates, *storage engine* operations, and *binary log* writes associated with a DDL operation into a single, atomic transaction. The transaction is either fully committed or rolled back, even if the server halts during the operation. Atomic DDL support was added in MySQL 8.0. For more information, see [Section 15.1.1, “Atomic Data Definition Statement Support”](https://dev.mysql.com/doc/refman/9.7/en/atomic-ddl.html "15.1.1 Atomic Data Definition Statement Support").

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [storage engine](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_storage_engine).

atomic instruction

Special instructions provided by the CPU, to ensure that critical low-level operations cannot be interrupted.

auto-increment

A property of a table column (specified by the `AUTO_INCREMENT` keyword) that automatically adds an ascending sequence of values in the column.

It saves work for the developer, not to have to produce new unique values when inserting new rows. It provides useful information for the query optimizer, because the column is known to be not null and with unique values. The values from such a column can be used as lookup keys in various contexts, and because they are auto-generated there is no reason to ever change them; for this reason, primary key columns are often specified as auto-incrementing.

Auto-increment columns can be problematic with statement-based replication, because replaying the statements on a replica might not produce the same set of column values as on the source, due to timing issues. When you have an auto-incrementing primary key, you can use statement-based replication only with the setting [`innodb_autoinc_lock_mode=1`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode). If you have [`innodb_autoinc_lock_mode=2`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode), which allows higher concurrency for insert operations, use **row-based replication** rather than **statement-based replication**. The setting [`innodb_autoinc_lock_mode=0`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) should not be used except for compatibility purposes.

Consecutive lock mode ([`innodb_autoinc_lock_mode=1`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)) is the default setting prior to MySQL 8.0.3. As of MySQL 8.0.3, interleaved lock mode ([`innodb_autoinc_lock_mode=2`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)) is the default, which reflects the change from statement-based to row-based replication as the default replication type.

See Also [auto-increment locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment_locking), [innodb\_autoinc\_lock\_mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_autoinc_lock_mode), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [row-based replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_based_replication), [statement-based replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_statement_based_replication).

auto-increment locking

The convenience of an **auto-increment** primary key involves some tradeoff with concurrency. In the simplest case, if one transaction is inserting values into the table, any other transactions must wait to do their own inserts into that table, so that rows inserted by the first transaction receive consecutive primary key values. `InnoDB` includes optimizations and the [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) option so that you can configure and optimal balance between predictable sequences of auto-increment values and maximum **concurrency** for insert operations.

See Also [auto-increment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment), [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [innodb\_autoinc\_lock\_mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_autoinc_lock_mode).

autocommit

A setting that causes a **commit** operation after each **SQL** statement. This mode is not recommended for working with `InnoDB` tables with **transactions** that span several statements. It can help performance for **read-only transactions** on `InnoDB` tables, where it minimizes overhead from **locking** and generation of **undo** data, especially in MySQL 5.6.4 and up. It is also appropriate for working with [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") tables, where transactions are not applicable.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [read-only transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_only_transaction), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction), [undo](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo).

availability

The ability to cope with, and if necessary recover from, failures on the host, including failures of MySQL, the operating system, or the hardware and maintenance activity that may otherwise cause downtime. Often paired with **scalability** as critical aspects of a large-scale deployment.

See Also [scalability](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scalability).

MySQL Enterprise Backup

A licensed product that performs **hot backups** of MySQL databases. It offers the most efficiency and flexibility when backing up [`InnoDB`](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") tables, but can also back up [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") and other kinds of tables.

See Also [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb).

### B

B-tree

A tree data structure that is popular for use in database indexes. The structure is kept sorted at all times, enabling fast lookup for exact matches (equals operator) and ranges (for example, greater than, less than, and `BETWEEN` operators). This type of index is available for most storage engines, such as [`InnoDB`](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") and [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine").

Because B-tree nodes can have many children, a B-tree is not the same as a binary tree, which is limited to 2 children per node.

Contrast with **hash index**, which is only available in the [`MEMORY`](https://dev.mysql.com/doc/refman/9.7/en/memory-storage-engine.html "18.3 The MEMORY Storage Engine") storage engine. The `MEMORY` storage engine can also use B-tree indexes, and you should choose B-tree indexes for `MEMORY` tables if some queries use range operators.

The use of the term B-tree is intended as a reference to the general class of index design. B-tree structures used by MySQL storage engines may be regarded as variants due to sophistications not present in a classic B-tree design. For related information, refer to the `InnoDB` Page Structure [Fil Header](https://dev.mysql.com/doc/internals/en/innodb-fil-header.html) section of the [MySQL Internals Manual](https://dev.mysql.com/doc/internals/en/index.html).

See Also [hash index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hash_index).

backticks

Identifiers within MySQL SQL statements must be quoted using the backtick character (`` ` ``) if they contain special characters or reserved words. For example, to refer to a table named `FOO#BAR` or a column named `SELECT`, you would specify the identifiers as `` `FOO#BAR` `` and `` `SELECT` ``. Since the backticks provide an extra level of safety, they are used extensively in program-generated SQL statements, where the identifier names might not be known in advance.

Many other database systems use double quotation marks (`"`) around such special names. For portability, you can enable `ANSI_QUOTES` mode in MySQL and use double quotation marks instead of backticks to qualify identifier names.

See Also [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

backup

The process of copying some or all table data and metadata from a MySQL instance, for safekeeping. Can also refer to the set of copied files. This is a crucial task for DBAs. The reverse of this process is the **restore** operation.

With MySQL, **physical backups** are performed by the **MySQL Enterprise Backup** product, and **logical backups** are performed by the `mysqldump` command. These techniques have different characteristics in terms of size and representation of the backup data, and speed (especially speed of the restore operation).

Backups are further classified as **hot**, **warm**, or **cold** depending on how much they interfere with normal database operation. (Hot backups have the least interference, cold backups the most.)

See Also [cold backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cold_backup), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [logical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_logical_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqldump](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqldump), [physical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_physical_backup), [warm backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_warm_backup).

beta

An early stage in the life of a software product, when it is available only for evaluation, typically without a definite release number or a number less than 1. `InnoDB` does not use the beta designation, preferring an **early adopter** phase that can extend over several point releases, leading to a **GA** release.

See Also [early adopter](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_early_adopter), [GA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ga).

binary log

A file containing a record of all statements or row changes that attempt to change table data. The contents of the binary log can be replayed to bring replicas up to date in a **replication** scenario, or to bring a database up to date after restoring table data from a backup. The binary logging feature can be turned on and off, although Oracle recommends always enabling it if you use replication or perform backups.

You can examine the contents of the binary log, or replay it during replication or recovery, by using the [**mysqlbinlog**](https://dev.mysql.com/doc/refman/9.7/en/mysqlbinlog.html "6.6.9 mysqlbinlog — Utility for Processing Binary Log Files") command. For full information about the binary log, see [Section 7.4.4, “The Binary Log”](https://dev.mysql.com/doc/refman/9.7/en/binary-log.html "7.4.4 The Binary Log"). For MySQL configuration options related to the binary log, see [Section 19.1.6.4, “Binary Logging Options and Variables”](https://dev.mysql.com/doc/refman/9.7/en/replication-options-binary-log.html "19.1.6.4 Binary Logging Options and Variables").

For the **MySQL Enterprise Backup** product, the file name of the binary log and the current position within the file are important details. To record this information for the source when taking a backup in a replication context, you can specify the `--slave-info` option.

Prior to MySQL 5.0, a similar capability was available, known as the update log. In MySQL 5.0 and higher, the binary log replaces the update log.

See Also [binlog](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binlog), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication).

binlog

An informal name for the **binary log** file. For example, you might see this abbreviation used in e-mail messages or forum discussions.

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log).

blind query expansion

A special mode of **full-text search** enabled by the `WITH QUERY         EXPANSION` clause. It performs the search twice, where the search phrase for the second search is the original search phrase concatenated with the few most highly relevant documents from the first search. This technique is mainly applicable for short search phrases, perhaps only a single word. It can uncover relevant matches where the precise search term does not occur in the document.

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search).

BLOB

An SQL data type ([`TINYBLOB`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types"), [`BLOB`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types"), [`MEDIUMBLOB`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types"), and [`LONGBLOB`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types")) for objects containing any kind of binary data, of arbitrary size. Used for storing documents, images, sound files, and other kinds of information that cannot easily be decomposed to rows and columns within a MySQL table. The techniques for handling BLOBs within a MySQL application vary with each **Connector** and **API**. MySQL `Connector/ODBC` defines `BLOB` values as `LONGVARBINARY`. For large, free-form collections of character data, the industry term is **CLOB**, represented by the MySQL `TEXT` data types.

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [CLOB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clob), [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc).

bottleneck

A portion of a system that is constrained in size or capacity, that has the effect of limiting overall throughput. For example, a memory area might be smaller than necessary; access to a single required resource might prevent multiple CPU cores from running simultaneously; or waiting for disk I/O to complete might prevent the CPU from running at full capacity. Removing bottlenecks tends to improve **concurrency**. For example, the ability to have multiple `InnoDB` **buffer pool** instances reduces contention when multiple sessions read from and write to the buffer pool simultaneously.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency).

bounce

A **shutdown** operation immediately followed by a restart. Ideally with a relatively short **warmup** period so that performance and throughput quickly return to a high level.

See Also [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown).

buddy allocator

A mechanism for managing different-sized **pages** in the InnoDB **buffer pool**.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [page size](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page_size).

buffer

A memory or disk area used for temporary storage. Data is buffered in memory so that it can be written to disk efficiently, with a few large I/O operations rather than many small ones. Data is buffered on disk for greater reliability, so that it can be recovered even when a **crash** or other failure occurs at the worst possible time. The main types of buffers used by InnoDB are the **buffer pool**, the **doublewrite buffer**, and the **change buffer**.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [crash](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash), [doublewrite buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_doublewrite_buffer).

buffer pool

The memory area that holds cached `InnoDB` data for both tables and indexes. For efficiency of high-volume read operations, the buffer pool is divided into **pages** that can potentially hold multiple rows. For efficiency of cache management, the buffer pool is implemented as a linked list of pages; data that is rarely used is aged out of the cache, using a variation of the **LRU** algorithm. On systems with large memory, you can improve concurrency by dividing the buffer pool into multiple **buffer pool instances**.

Several `InnoDB` status variables, `INFORMATION_SCHEMA` tables, and `performance_schema` tables help to monitor the internal workings of the buffer pool. Starting in MySQL 5.6, you can avoid a lengthy warmup period after restarting the server, particularly for instances with large buffer pools, by saving the buffer pool state at server shutdown and restoring the buffer pool to the same state at server startup. See [Section 17.8.3.6, “Saving and Restoring the Buffer Pool State”](https://dev.mysql.com/doc/refman/9.7/en/innodb-preload-buffer-pool.html "17.8.3.6 Saving and Restoring the Buffer Pool State").

See Also [buffer pool instance](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool_instance), [LRU](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lru), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [warm up](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_warm_up).

buffer pool instance

Any of the multiple regions into which the **buffer pool** can be divided, controlled by the [`innodb_buffer_pool_instances`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_instances) configuration option. The total memory size specified by [`innodb_buffer_pool_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_size) is divided among all buffer pool instances. Typically, having multiple buffer pool instances is appropriate for systems that allocate multiple gigabytes to the `InnoDB` buffer pool, with each instance being one gigabyte or larger. On systems loading or looking up large amounts of data in the buffer pool from many concurrent sessions, having multiple buffer pool instances reduces contention for exclusive access to data structures that manage the buffer pool.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool).

built-in

The built-in `InnoDB` storage engine within MySQL is the original form of distribution for the storage engine. Contrast with the **InnoDB Plugin**. Starting with MySQL 5.5, the InnoDB Plugin is merged back into the MySQL code base as the built-in `InnoDB` storage engine (known as InnoDB 1.1).

This distinction is important mainly in MySQL 5.1, where a feature or bug fix might apply to the InnoDB Plugin but not the built-in `InnoDB`, or vice versa.

See Also [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb).

business rules

The relationships and sequences of actions that form the basis of business software, used to run a commercial company. Sometimes these rules are dictated by law, other times by company policy. Careful planning ensures that the relationships encoded and enforced by the database, and the actions performed through application logic, accurately reflect the real policies of the company and can handle real-life situations.

For example, an employee leaving a company might trigger a sequence of actions from the human resources department. The human resources database might also need the flexibility to represent data about a person who has been hired, but not yet started work. Closing an account at an online service might result in data being removed from a database, or the data might be moved or flagged so that it could be recovered if the account is re-opened. A company might establish policies regarding salary maximums, minimums, and adjustments, in addition to basic sanity checks such as the salary not being a negative number. A retail database might not allow a purchase with the same serial number to be returned more than once, or might not allow credit card purchases above a certain value, while a database used to detect fraud might allow these kinds of things.

See Also [relational](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_relational).

### C

.cfg file

A metadata file used with the `InnoDB` **transportable tablespace** feature. It is produced by the command `FLUSH TABLES ...         FOR EXPORT`, puts one or more tables in a consistent state that can be copied to another server. The `.cfg` file is copied along with the corresponding **.ibd file**, and used to adjust the internal values of the `.ibd` file, such as the **space ID**, during the `ALTER TABLE ... IMPORT TABLESPACE` step.

See Also [space ID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_space_id), [transportable tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transportable_tablespace).

C

A programming language that combines portability with performance and access to low-level hardware features, making it a popular choice for writing operating systems, drivers, and other kinds of system software. Many complex applications, languages, and reusable modules feature pieces written in C, tied together with high-level components written in other languages. Its core syntax is familiar to **C++**, **Java**, and **C#** developers.

See Also [C API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_c_api), [C++](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cplusplus), [C#](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_csharp), [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java).

C API

The C **API** code is distributed with MySQL. It is included in the **libmysqlclient** library and enables **C** programs to access a database.

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [C](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_c), [libmysqlclient](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysqlclient).

C#

A programming language combining strong typing and object-oriented features, running within the Microsoft **.NET** framework or its open-source counterpart **Mono**. Often used for creating applications with the **ASP.net** framework. Its syntax is familiar to **C**, **C++** and **Java** developers.

See Also [.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos__net), [ASP.net](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_asp_net), [C](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_c), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [C++](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cplusplus), [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java), [Mono](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mono).

C++

A programming language with core syntax familiar to **C** developers. Provides access to low-level operations for performance, combined with higher-level data types, object-oriented features, and garbage collection. To write C++ applications for MySQL, you use the **Connector/C++** component.

See Also [C](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_c), [Connector/C++](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_c__).

cache

The general term for any memory area that stores copies of data for frequent or high-speed retrieval. In `InnoDB`, the primary kind of cache structure is the **buffer pool**.

See Also [buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer), [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool).

cardinality

The number of different values in a table **column**. When queries refer to columns that have an associated **index**, the cardinality of each column influences which access method is most efficient. For example, for a column with a **unique constraint**, the number of different values is equal to the number of rows in the table. If a table has a million rows but only 10 different values for a particular column, each value occurs (on average) 100,000 times. A query such as `SELECT c1 FROM t1 WHERE c1 = 50;` thus might return 1 row or a huge number of rows, and the database server might process the query differently depending on the cardinality of `c1`.

If the values in a column have a very uneven distribution, the cardinality might not be a good way to determine the best query plan. For example, `SELECT c1 FROM t1 WHERE c1 =         x;` might return 1 row when `x=50` and a million rows when `x=30`. In such a case, you might need to use **index hints** to pass along advice about which lookup method is more efficient for a particular query.

Cardinality can also apply to the number of distinct values present in multiple columns, as in a **composite index**.

See Also [column](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_column), [composite index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_composite_index), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [index hint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index_hint), [random dive](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_random_dive), [selectivity](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_selectivity), [unique constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_constraint).

change buffer

A special data structure that records changes to **pages** in **secondary indexes**. These values could result from SQL [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement"), [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement"), or [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement") statements (**DML**). The set of features involving the change buffer is known collectively as **change buffering**, consisting of **insert buffering**, **delete buffering**, and **purge buffering**.

Changes are only recorded in the change buffer when the relevant page from the secondary index is not in the **buffer pool**. When the relevant index page is brought into the buffer pool while associated changes are still in the change buffer, the changes for that page are applied in the buffer pool (**merged**) using the data from the change buffer. Periodically, the **purge** operation that runs during times when the system is mostly idle, or during a slow shutdown, writes the new index pages to disk. The purge operation can write the disk blocks for a series of index values more efficiently than if each value were written to disk immediately.

Physically, the change buffer is part of the **system tablespace**, so that the index changes remain buffered across database restarts. The changes are only applied (**merged**) when the pages are brought into the buffer pool due to some other read operation.

The kinds and amount of data stored in the change buffer are governed by the [`innodb_change_buffering`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffering) and [`innodb_change_buffer_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffer_max_size) configuration options. To see information about the current data in the change buffer, issue the [`SHOW ENGINE INNODB         STATUS`](https://dev.mysql.com/doc/refman/9.7/en/show-engine.html "15.7.7.17 SHOW ENGINE Statement") command.

Formerly known as the **insert buffer**.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [change buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffering), [delete buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_delete_buffering), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [insert buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffer), [insert buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffering), [merge](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_merge), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [purge buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_purge_buffering), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index).

change buffering

The general term for the features involving the **change buffer**, consisting of **insert buffering**, **delete buffering**, and **purge buffering**. Index changes resulting from SQL statements, which could normally involve random I/O operations, are held back and performed periodically by a background **thread**. This sequence of operations can write the disk blocks for a series of index values more efficiently than if each value were written to disk immediately. Controlled by the [`innodb_change_buffering`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffering) and [`innodb_change_buffer_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_change_buffer_max_size) configuration options.

See Also [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [delete buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_delete_buffering), [insert buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffering), [purge buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_purge_buffering).

checkpoint

As changes are made to data pages that are cached in the **buffer pool**, those changes are written to the **data files** sometime later, a process known as **flushing**. The checkpoint is a record of the latest changes (represented by an **LSN** value) that have been successfully written to the data files.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [fuzzy checkpointing](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fuzzy_checkpointing), [LSN](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lsn).

checksum

In `InnoDB`, a validation mechanism to detect corruption when a **page** in a **tablespace** is read from disk into the `InnoDB` **buffer pool**. This feature is controlled by the `innodb_checksums` configuration option in MySQL 5.5. `innodb_checksums` is deprecated in MySQL 5.6.3, replaced by [`innodb_checksum_algorithm`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_checksum_algorithm).

The [**innochecksum**](https://dev.mysql.com/doc/refman/9.7/en/innochecksum.html "6.6.2 innochecksum — Offline InnoDB File Checksum Utility") command helps diagnose corruption problems by testing the checksum values for a specified **tablespace** file while the MySQL server is shut down.

MySQL also uses checksums for replication purposes. For details, see the configuration options [`binlog_checksum`](https://dev.mysql.com/doc/refman/9.7/en/replication-options-binary-log.html#sysvar_binlog_checksum), [`source_verify_checksum`](https://dev.mysql.com/doc/refman/9.7/en/replication-options-binary-log.html#sysvar_source_verify_checksum) or [`master_verify_checksum`](https://dev.mysql.com/doc/refman/9.7/en/replication-options-binary-log.html#sysvar_master_verify_checksum), and [`replica_sql_verify_checksum`](https://dev.mysql.com/doc/refman/9.7/en/replication-options-replica.html#sysvar_replica_sql_verify_checksum) or [`slave_sql_verify_checksum`](https://dev.mysql.com/doc/refman/9.7/en/replication-options-replica.html#sysvar_slave_sql_verify_checksum).

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

child table

In a **foreign key** relationship, a child table is one whose rows refer (or point) to rows in another table with an identical value for a specific column. This is the table that contains the `FOREIGN KEY ...         REFERENCES` clause and optionally `ON         UPDATE` and `ON DELETE` clauses. The corresponding row in the **parent table** must exist before the row can be created in the child table. The values in the child table can prevent delete or update operations on the parent table, or can cause automatic deletion or updates in the child table, based on the `ON         CASCADE` option used when creating the foreign key.

See Also [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [parent table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_parent_table).

clean page

A **page** in the `InnoDB` **buffer pool** where all changes made in memory have also been written (**flushed**) to the [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files "data files"). The opposite of a **dirty page**.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

clean shutdown

A **shutdown** that completes without errors and applies all changes to `InnoDB` tables before finishing, as opposed to a **crash** or a **fast shutdown**. Synonym for **slow shutdown**.

See Also [crash](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash), [fast shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fast_shutdown), [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown), [slow shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_slow_shutdown).

client

A program that runs outside the database server, communicating with the database by sending requests through a **Connector**, or an **API** made available through **client libraries**. It can run on the same physical machine as the database server, or on a remote machine connected over a network. It can be a special-purpose database application, or a general-purpose program like the [**mysql**](https://dev.mysql.com/doc/refman/9.7/en/mysql.html "6.5.1 mysql — The MySQL Command-Line Client") command-line processor.

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [client libraries](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client_libraries), [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [mysql](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql), [server](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_server).

client libraries

Files containing collections of functions for working with databases. By compiling your program with these libraries, or installing them on the same system as your application, you can run a database application (known as a **client**) on a machine that does not have the MySQL server installed; the application accesses the database over a network. With MySQL, you can use the **libmysqlclient** library from the MySQL server itself.

See Also [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [libmysqlclient](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysqlclient).

client-side prepared statement

A type of **prepared statement** where the caching and reuse are managed locally, emulating the functionality of **server-side prepared statements**. Historically, used by some **Connector/J**, **Connector/ODBC**, and **Connector/PHP** developers to work around issues with server-side stored procedures. With modern MySQL server versions, server-side prepared statements are recommended for performance, scalability, and memory efficiency.

See Also [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc), [Connector/PHP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_php), [prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_statement).

CLOB

An SQL data type ([`TINYTEXT`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types"), [`TEXT`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types"), [`MEDIUMTEXT`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types"), or [`LONGTEXT`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types")) for objects containing any kind of character data, of arbitrary size. Used for storing text-based documents, with associated character set and collation order. The techniques for handling CLOBs within a MySQL application vary with each **Connector** and **API**. MySQL Connector/ODBC defines `TEXT` values as `LONGVARCHAR`. For storing binary data, the equivalent is the **BLOB** type.

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [BLOB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_blob), [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc).

clustered index

The `InnoDB` term for a **primary key** index. `InnoDB` table storage is organized based on the values of the primary key columns, to speed up queries and sorts involving the primary key columns. For best performance, choose the primary key columns carefully based on the most performance-critical queries. Because modifying the columns of the clustered index is an expensive operation, choose primary columns that are rarely or never updated.

In the Oracle Database product, this type of table is known as an **index-organized table**.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index).

cold backup

A **backup** taken while the database is shut down. For busy applications and websites, this might not be practical, and you might prefer a **warm backup** or a **hot backup**.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [warm backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_warm_backup).

column

A data item within a **row**, whose storage and semantics are defined by a data type. Each **table** and **index** is largely defined by the set of columns it contains.

Each column has a **cardinality** value. A column can be the **primary key** for its table, or part of the primary key. A column can be subject to a **unique constraint**, a **NOT NULL constraint**, or both. Values in different columns, even across different tables, can be linked by a **foreign key** relationship.

In discussions of MySQL internal operations, sometimes **field** is used as a synonym.

See Also [cardinality](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cardinality), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [NOT NULL constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_not_null_constraint), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [row](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row), [unique constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_constraint).

column index

An **index** on a single column.

See Also [composite index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_composite_index), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index).

column prefix

When an **index** is created with a length specification, such as `CREATE INDEX idx ON t1         (c1(N))`, only the first N characters of the column value are stored in the index. Keeping the index prefix small makes the index compact, and the memory and disk I/O savings help performance. (Although making the index prefix too small can hinder query optimization by making rows with different values appear to the query optimizer to be duplicates.)

For columns containing binary values or long text strings, where sorting is not a major consideration and storing the entire value in the index would waste space, the index automatically uses the first N (typically 768) characters of the value to do lookups and sorts.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index).

command interceptor

Synonym for **statement interceptor**. One aspect of the **interceptor** design pattern available for both **Connector/NET** and **Connector/J**. What Connector/NET calls a command, Connector/J refers to as a statement. Contrast with **exception interceptor**.

See Also [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [exception interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_exception_interceptor), [interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_interceptor), [statement interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_statement_interceptor).

commit

A **SQL** statement that ends a **transaction**, making permanent any changes made by the transaction. It is the opposite of **rollback**, which undoes any changes made in the transaction.

`InnoDB` uses an **optimistic** mechanism for commits, so that changes can be written to the data files before the commit actually occurs. This technique makes the commit itself faster, with the tradeoff that more work is required in case of a rollback.

By default, MySQL uses the **autocommit** setting, which automatically issues a commit following each SQL statement.

See Also [autocommit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_autocommit), [optimistic](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_optimistic), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

composite index

An **index** that includes multiple columns.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index).

compressed backup

The compression feature of the **MySQL Enterprise Backup** product makes a compressed copy of each tablespace, changing the extension from `.ibd` to `.ibz`. Compressing backup data allows you to keep more backups on hand, and reduces the time to transfer backups to a different server. The data is uncompressed during the restore operation. When a compressed backup operation processes a table that is already compressed, it skips the compression step for that table, because compressing again would result in little or no space savings.

A set of files produced by the **MySQL Enterprise Backup** product, where each **tablespace** is compressed. The compressed files are renamed with a `.ibz` file extension.

Applying **compression** at the start of the backup process helps to avoid storage overhead during the compression process, and to avoid network overhead when transferring the backup files to another server. The process of **applying** the **binary log** takes longer, and requires uncompressing the backup files.

See Also [apply](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_apply), [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

compressed table

A table for which the data is stored in compressed form. For `InnoDB`, it is a table created with `ROW_FORMAT=COMPRESSED`. See [Section 17.9, “InnoDB Table and Page Compression”](https://dev.mysql.com/doc/refman/9.7/en/innodb-compression.html "17.9 InnoDB Table and Page Compression") for more information.

compression failure

Not actually an error, rather an expensive operation that can occur when using **compression** in combination with **DML** operations. It occurs when: updates to a compressed **page** overflow the area on the page reserved for recording modifications; the page is compressed again, with all changes applied to the table data; the re-compressed data does not fit on the original page, requiring MySQL to split the data into two new pages and compress each one separately. To check the frequency of this condition, query the [`INFORMATION_SCHEMA.INNODB_CMP`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-cmp-table.html "28.4.6 The INFORMATION_SCHEMA INNODB_CMP and INNODB_CMP_RESET Tables") table and check how much the value of the `COMPRESS_OPS` column exceeds the value of the `COMPRESS_OPS_OK` column. Ideally, compression failures do not occur often; when they do, you can adjust the [`innodb_compression_level`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_compression_level), [`innodb_compression_failure_threshold_pct`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_compression_failure_threshold_pct), and [`innodb_compression_pad_pct_max`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_compression_pad_pct_max) configuration options.

See Also [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

concatenated index

See [composite index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_composite_index).

concurrency

The ability of multiple operations (in database terminology, **transactions**) to run simultaneously, without interfering with each other. Concurrency is also involved with performance, because ideally the protection for multiple simultaneous transactions works with a minimum of performance overhead, using efficient mechanisms for **locking**.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

configuration file

The file that holds the **option** values used by MySQL at startup. Traditionally, on Linux and Unix this file is named `my.cnf`, and on Windows it is named `my.ini`. You can set a number of options related to InnoDB under the `[mysqld]` section of the file.

See [Section 6.2.2.2, “Using Option Files”](https://dev.mysql.com/doc/refman/9.7/en/option-files.html "6.2.2.2 Using Option Files") for information about where MySQL searches for configuration files.

When you use the **MySQL Enterprise Backup** product, you typically use two configuration files: one that specifies where the data comes from and how it is structured (which could be the original configuration file for your server), and a stripped-down one containing only a small set of options that specify where the backup data goes and how it is structured. The configuration files used with the **MySQL Enterprise Backup** product must contain certain options that are typically left out of regular configuration files, so you might need to add options to your existing configuration file for use with **MySQL Enterprise Backup**.

See Also [my.cnf](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_my_cnf), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [option](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_option), [option file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_option_file).

connection

The communication channel between an application and a MySQL server. The performance and scalability of a database applications is influenced by on how quickly a database connection can be established, how many can be made simultaneously, and how long they persist. The parameters such as **host**, **port**, and so on are represented as a **connection string** in **Connector/NET**, and as a **DSN** in **Connector/ODBC**. High-traffic systems make use of an optimization known as the **connection pool**.

See Also [connection pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection_pool), [connection string](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection_string), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc), [DSN](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dsn), [host](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_host), [port](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_port).

connection pool

A cache area that allows database **connections** to be reused within the same application or across different applications, rather than setting up and tearing down a new connection for every database operation. This technique is common with **J2EE** application servers. **Java** applications using **Connector/J** can use the connection pool features of **Tomcat** and other application servers. The reuse is transparent to applications; the application still opens and closes the connection as usual.

See Also [connection](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection), [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee), [Tomcat](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tomcat).

connection string

A representation of the parameters for a database **connection**, encoded as a string literal so that it can be used in program code. The parts of the string represent connection parameters such as **host** and **port**. A connection string contains several key-value pairs, separated by semicolons. Each key-value pair is joined with an equal sign. Frequently used with **Connector/NET** applications; see [Creating a Connector/NET Connection String](https://dev.mysql.com/doc/connector-net/en/connector-net-connections-string.html) for details.

See Also [connection](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [host](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_host), [port](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_port).

connector

MySQL Connectors provide connectivity to the MySQL server for **client** programs. Several programming languages and frameworks each have their own associated Connector. Contrast with the lower-level access provided by an **API**.

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [Connector/C++](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_c__), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc).

Connector/C++

Connector/C++ 8.0 can be used to access MySQL servers that implement a [document store](https://dev.mysql.com/doc/refman/9.7/en/document-store.html "Chapter 22 Using MySQL as a Document Store"), or in a traditional way using SQL queries. It enables development of C++ applications using X DevAPI, or plain C applications using X DevAPI for C. It also enables development of C++ applications that use the legacy JDBC-based API from Connector/C++ 1.1. For more information, see [MySQL Connector/C++ 9.7 Developer Guide](https://dev.mysql.com/doc/connector-cpp/9.7/en/).

See Also [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [JDBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_jdbc).

Connector/NET

A MySQL **connector** for developers writing applications using languages, technologies, and frameworks such as **C#**, **.NET**, **Mono**, **Visual Studio**, **ASP.net**, and **ADO.net**.

See Also [ADO.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ado_net), [ASP.net](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_asp_net), [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [C#](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_csharp), [Mono](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mono), [Visual Studio](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_visual_studio).

Connector/ODBC

The family of MySQL ODBC drivers that provide access to a MySQL database using the industry standard Open Database Connectivity (**ODBC**) API. Formerly called MyODBC drivers. For full details, see [MySQL Connector/ODBC Developer Guide](https://dev.mysql.com/doc/connector-odbc/en/).

See Also [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_odbc).

Connector/PHP

A version of the `mysql` and `mysqli` **APIs** for **PHP** optimized for the Windows operating system.

See Also [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [PHP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_php), [PHP API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_php_api).

consistent read

A read operation that uses **snapshot** information to present query results based on a point in time, regardless of changes performed by other transactions running at the same time. If queried data has been changed by another transaction, the original data is reconstructed based on the contents of the **undo log**. This technique avoids some of the **locking** issues that can reduce **concurrency** by forcing transactions to wait for other transactions to finish.

With **REPEATABLE READ** **isolation level**, the snapshot is based on the time when the first read operation is performed. With **READ COMMITTED** isolation level, the snapshot is reset to the time of each consistent read operation.

Consistent read is the default mode in which `InnoDB` processes `SELECT` statements in **READ COMMITTED** and **REPEATABLE READ** isolation levels. Because a consistent read does not set any locks on the tables it accesses, other sessions are free to modify those tables while a consistent read is being performed on the table.

For technical details about the applicable isolation levels, see [Section 17.7.2.3, “Consistent Nonlocking Reads”](https://dev.mysql.com/doc/refman/9.7/en/innodb-consistent-read.html "17.7.2.3 Consistent Nonlocking Reads").

See Also [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [READ COMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_committed), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [snapshot](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_snapshot), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

constraint

An automatic test that can block database changes to prevent data from becoming inconsistent. (In computer science terms, a kind of assertion related to an invariant condition.) Constraints are a crucial component of the **ACID** philosophy, to maintain data consistency. Constraints supported by MySQL include **FOREIGN KEY constraints** and **unique constraints**.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [unique constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_constraint).

counter

A value that is incremented by a particular kind of `InnoDB` operation. Useful for measuring how busy a server is, troubleshooting the sources of performance issues, and testing whether changes (for example, to configuration settings or indexes used by queries) have the desired low-level effects. Different kinds of counters are available through **Performance Schema** tables and **INFORMATION\_SCHEMA** tables, particularly `INFORMATION_SCHEMA.INNODB_METRICS`.

See Also [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_information_schema), [metrics counter](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_metrics_counter), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema).

covering index

An **index** that includes all the columns retrieved by a query. Instead of using the index values as pointers to find the full table rows, the query returns values from the index structure, saving disk I/O. `InnoDB` can apply this optimization technique to more indexes than MyISAM can, because `InnoDB` **secondary indexes** also include the **primary key** columns. `InnoDB` cannot apply this technique for queries against tables modified by a transaction, until that transaction ends.

Any **column index** or **composite index** could act as a covering index, given the right query. Design your indexes and queries to take advantage of this optimization technique wherever possible.

See Also [column index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_column_index), [composite index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_composite_index), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index).

CPU-bound

A type of **workload** where the primary **bottleneck** is CPU operations in memory. Typically involves read-intensive operations where the results can all be cached in the **buffer pool**.

See Also [bottleneck](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_bottleneck), [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [workload](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_workload).

crash

MySQL uses the term “crash” to refer generally to any unexpected **shutdown** operation where the server cannot do its normal cleanup. For example, a crash could happen due to a hardware fault on the database server machine or storage device; a power failure; a potential data mismatch that causes the MySQL server to halt; a **fast shutdown** initiated by the DBA; or many other reasons. The robust, automatic **crash recovery** for **InnoDB** tables ensures that data is made consistent when the server is restarted, without any extra work for the DBA.

See Also [crash recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash_recovery), [fast shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fast_shutdown), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown).

crash recovery

The cleanup activities that occur when MySQL is started again after a **crash**. For **InnoDB** tables, changes from incomplete transactions are replayed using data from the **redo log**. Changes that were **committed** before the crash, but not yet written into the **data files**, are reconstructed from the **doublewrite buffer**. When the database is shut down normally, this type of activity is performed during shutdown by the **purge** operation.

During normal operation, committed data can be stored in the **change buffer** for a period of time before being written to the data files. There is always a tradeoff between keeping the data files up-to-date, which introduces performance overhead during normal operation, and buffering the data, which can make shutdown and crash recovery take longer.

See Also [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [crash](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [doublewrite buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_doublewrite_buffer), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log).

CRUD

Acronym for “create, read, update, delete”, a common sequence of operations in database applications. Often denotes a class of applications with relatively simple database usage (basic **DDL**, **DML** and **query** statements in **SQL**) that can be implemented quickly in any language.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

cursor

An internal MySQL data structure that represents the result set of an SQL statement. Often used with **prepared statements** and **dynamic SQL**. It works like an iterator in other high-level languages, producing each value from the result set as requested.

Although SQL usually handles the processing of cursors for you, you might delve into the inner workings when dealing with performance-critical code.

See Also [dynamic SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dynamic_sql), [prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_statement), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query).

### D

data definition language

See [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl).

data directory

The directory under which each MySQL **instance** keeps the **data files** for `InnoDB` and the directories representing individual databases. Controlled by the [`datadir`](https://dev.mysql.com/doc/refman/9.7/en/server-system-variables.html#sysvar_datadir) configuration option.

See Also [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [instance](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_instance).

data files

The files that physically contain **table** and **index** data.

The `InnoDB` **system tablespace**, which holds the `InnoDB` **data dictionary** and is capable of holding data for multiple `InnoDB` tables, is represented by one or more `.ibdata` data files.

File-per-table tablespaces, which hold data for a single `InnoDB` table, are represented by a `.ibd` data file.

General tablespaces (introduced in MySQL 5.7.6), which can hold data for multiple `InnoDB` tables, are also represented by a `.ibd` data file.

See Also [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [general tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_tablespace), [ibdata file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ibdata_file), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

data manipulation language

See [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml).

data warehouse

A database system or application that primarily runs large **queries**. The read-only or read-mostly data might be organized in **denormalized** form for query efficiency. Can benefit from the optimizations for **read-only transactions** in MySQL 5.6 and higher. Contrast with **OLTP**.

See Also [denormalized](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_denormalized), [OLTP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_oltp), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [read-only transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_only_transaction).

database

Within the MySQL **data directory**, each database is represented by a separate directory. The InnoDB **system tablespace**, which can hold table data from multiple databases within a MySQL **instance**, is kept in **data files** that reside outside of individual database directories. When **file-per-table** mode is enabled, the **.ibd files** representing individual InnoDB tables are stored inside the database directories unless created elsewhere using the `DATA         DIRECTORY` clause. General tablespaces, introduced in MySQL 5.7.6, also hold table data in **.ibd files**. Unlike file-per-table **.ibd files**, general tablespace **.ibd files** can hold table data from multiple databases within a MySQL **instance**, and can be assigned to directories relative to or independent of the MySQL data directory.

For long-time MySQL users, a database is a familiar notion. Users coming from an Oracle Database background may find that the MySQL meaning of a database is closer to what Oracle Database calls a **schema**.

See Also [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [instance](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_instance), [schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_schema).

DCL

Data control language, a set of **SQL** statements for managing privileges. In MySQL, consists of the [`GRANT`](https://dev.mysql.com/doc/refman/9.7/en/grant.html "15.7.1.6 GRANT Statement") and [`REVOKE`](https://dev.mysql.com/doc/refman/9.7/en/revoke.html "15.7.1.8 REVOKE Statement") statements. Contrast with **DDL** and **DML**.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

DDEX provider

A feature that lets you use the data design tools within **Visual Studio** to manipulate the schema and objects within a MySQL database. For MySQL applications using **Connector/NET**, the MySQL Visual Studio Plugin acts as a DDEX provider with MySQL 5.0 and later.

See Also [Visual Studio](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_visual_studio).

DDL

Data definition language, a set of **SQL** statements for manipulating the database itself rather than individual table rows. Includes all forms of the `CREATE`, `ALTER`, and `DROP` statements. Also includes the `TRUNCATE` statement, because it works differently than a ``DELETE         FROM *`table_name`*`` statement, even though the ultimate effect is similar.

DDL statements automatically **commit** the current **transaction**; they cannot be **rolled back**.

The `InnoDB` [online DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_online_ddl "online DDL") feature enhances performance for [`CREATE         INDEX`](https://dev.mysql.com/doc/refman/9.7/en/create-index.html "15.1.18 CREATE INDEX Statement"), [`DROP INDEX`](https://dev.mysql.com/doc/refman/9.7/en/drop-index.html "15.1.31 DROP INDEX Statement"), and many types of [`ALTER TABLE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement") operations. See [Section 17.12, “InnoDB and Online DDL”](https://dev.mysql.com/doc/refman/9.7/en/innodb-online-ddl.html "17.12 InnoDB and Online DDL") for more information. Also, the `InnoDB` [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table "file-per-table") setting can affect the behavior of [`DROP         TABLE`](https://dev.mysql.com/doc/refman/9.7/en/drop-table.html "15.1.37 DROP TABLE Statement") and [`TRUNCATE         TABLE`](https://dev.mysql.com/doc/refman/9.7/en/truncate-table.html "15.1.42 TRUNCATE TABLE Statement") operations.

Contrast with **DML** and **DCL**.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [DCL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dcl), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

deadlock

A situation where different **transactions** are unable to proceed, because each holds a **lock** that the other needs. Because both transactions are waiting for a resource to become available, neither one ever releases the locks it holds.

A deadlock can occur when the transactions lock rows in multiple tables (through statements such as `UPDATE` or `SELECT ... FOR UPDATE`), but in the opposite order. A deadlock can also occur when such statements lock ranges of index records and **gaps**, with each transaction acquiring some locks but not others due to a timing issue.

For background information on how deadlocks are automatically detected and handled, see [Section 17.7.5.2, “Deadlock Detection”](https://dev.mysql.com/doc/refman/9.7/en/innodb-deadlock-detection.html "17.7.5.2 Deadlock Detection"). For tips on avoiding and recovering from deadlock conditions, see [Section 17.7.5.3, “How to Minimize and Handle Deadlocks”](https://dev.mysql.com/doc/refman/9.7/en/innodb-deadlocks-handling.html "17.7.5.3 How to Minimize and Handle Deadlocks").

See Also [gap](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

delete

When `InnoDB` processes a `DELETE` statement, the rows are immediately marked for deletion and no longer are returned by queries. The storage is reclaimed sometime later, during the periodic garbage collection known as the **purge** operation. For removing large quantities of data, related operations with their own performance characteristics are **TRUNCATE** and **DROP**.

See Also [drop](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_drop), [truncate](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_truncate).

delete buffering

The technique of storing changes to secondary index pages, resulting from `DELETE` operations, in the **change buffer** rather than writing the changes immediately, so that the physical writes can be performed to minimize random I/O. (Because delete operations are a two-step process, this operation buffers the write that normally marks an index record for deletion.) It is one of the types of **change buffering**; the others are **insert buffering** and **purge buffering**.

See Also [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [change buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffering), [insert buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffer), [insert buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffering), [purge buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_purge_buffering).

denormalized

A data storage strategy that duplicates data across different tables, rather than linking the tables with **foreign keys** and **join** queries. Typically used in **data warehouse** applications, where the data is not updated after loading. In such applications, query performance is more important than making it simple to maintain consistent data during updates. Contrast with **normalized**.

See Also [data warehouse](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_warehouse), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [join](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_join), [normalized](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_normalized).

dirty page

A **page** in the `InnoDB` **buffer pool** that has been updated in memory, where the changes are not yet written (**flushed**) to the **data files**. The opposite of a **clean page**.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [clean page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clean_page), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

dirty read

An operation that retrieves unreliable data, data that was updated by another transaction but not yet **committed**. It is only possible with the **isolation level** known as **read uncommitted**.

This kind of operation does not adhere to the **ACID** principle of database design. It is considered very risky, because the data could be **rolled back**, or updated further before being committed; then, the transaction doing the dirty read would be using data that was never confirmed as accurate.

Its opposite is **consistent read**, where `InnoDB` ensures that a transaction does not read information updated by another transaction, even if the other transaction commits in the meantime.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [READ UNCOMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_uncommitted), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback).

disk-based

A kind of database that primarily organizes data on disk storage (hard drives or equivalent). Data is brought back and forth between disk and memory to be operated upon. It is the opposite of an **in-memory database**. Although `InnoDB` is disk-based, it also contains features such as he **buffer pool**, multiple buffer pool instances, and the **adaptive hash index** that allow certain kinds of workloads to work primarily from memory.

See Also [adaptive hash index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_adaptive_hash_index), [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [in-memory database](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_in_memory_database).

disk-bound

A type of **workload** where the primary **bottleneck** is disk I/O. (Also known as **I/O-bound**.) Typically involves frequent writes to disk, or random reads of more data than can fit into the **buffer pool**.

See Also [bottleneck](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_bottleneck), [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [workload](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_workload).

DML

Data manipulation language, a set of **SQL** statements for performing [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement"), [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement"), and [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement") operations. The [`SELECT`](https://dev.mysql.com/doc/refman/9.7/en/select.html "15.2.13 SELECT Statement") statement is sometimes considered as a DML statement, because the `SELECT ...         FOR UPDATE` form is subject to the same considerations for **locking** as [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement"), [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement"), and [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement").

DML statements for an `InnoDB` table operate in the context of a **transaction**, so their effects can be **committed** or **rolled back** as a single unit.

Contrast with **DDL** and **DCL**.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [DCL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dcl), [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

document id

In the `InnoDB` **full-text search** feature, a special column in the table containing the **FULLTEXT index**, to uniquely identify the document associated with each **ilist** value. Its name is `FTS_DOC_ID` (uppercase required). The column itself must be of `BIGINT UNSIGNED NOT NULL` type, with a unique index named `FTS_DOC_ID_INDEX`. Preferably, you define this column when creating the table. If `InnoDB` must add the column to the table while creating a `FULLTEXT` index, the indexing operation is considerably more expensive.

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search), [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index), [ilist](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ilist).

doublewrite buffer

`InnoDB` uses a file flush technique called doublewrite. Before writing **pages** to the **data files**, `InnoDB` first writes them to a storage area called the doublewrite buffer. Only after the write and the flush to the doublewrite buffer have completed, does `InnoDB` write the pages to their proper positions in the data file. If there is an operating system, storage subsystem or [**mysqld**](https://dev.mysql.com/doc/refman/9.7/en/mysqld.html "6.3.1 mysqld — The MySQL Server") process crash in the middle of a page write, `InnoDB` can find a good copy of the page from the doublewrite buffer during **crash recovery**.

Although data is always written twice, the doublewrite buffer does not require twice as much I/O overhead or twice as many I/O operations. Data is written to the buffer itself as a large sequential chunk, with a single `fsync()` call to the operating system.

See Also [crash recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash_recovery), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

drop

A kind of **DDL** operation that removes a schema object, through a statement such as [`DROP TABLE`](https://dev.mysql.com/doc/refman/9.7/en/drop-table.html "15.1.37 DROP TABLE Statement") or [`DROP INDEX`](https://dev.mysql.com/doc/refman/9.7/en/drop-index.html "15.1.31 DROP INDEX Statement"). It maps internally to an [`ALTER TABLE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement") statement. From an `InnoDB` perspective, the performance considerations of such operations involve the time that the **data dictionary** is locked to ensure that interrelated objects are all updated, and the time to update memory structures such as the **buffer pool**. For a **table**, the drop operation has somewhat different characteristics than a **truncate** operation ([`TRUNCATE TABLE`](https://dev.mysql.com/doc/refman/9.7/en/truncate-table.html "15.1.42 TRUNCATE TABLE Statement") statement).

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [truncate](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_truncate).

DSN

Acronym for “Database Source Name”. It is the encoding for **connection** information within **Connector/ODBC**. See [Configuring a Connector/ODBC DSN on Windows](https://dev.mysql.com/doc/connector-odbc/en/connector-odbc-configuration-dsn-windows.html) for full details. It is the equivalent of the **connection string** used by **Connector/NET**.

See Also [connection](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection), [connection string](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection_string), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc).

dynamic cursor

A type of **cursor** supported by **ODBC** that can pick up new and changed results when the rows are read again. Whether and how quickly the changes are visible to the cursor depends on the type of table involved (transactional or non-transactional) and the isolation level for transactional tables. Support for dynamic cursors must be explicitly enabled.

See Also [cursor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cursor), [ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_odbc).

dynamic SQL

A feature that lets you create and execute **prepared statements** using more robust, secure, and efficient methods to substitute parameter values than the naive technique of concatenating the parts of the statement into a string variable.

See Also [prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_statement).

dynamic statement

A **prepared statement** created and executed through **dynamic SQL**.

See Also [dynamic SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dynamic_sql), [prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_statement).

### E

early adopter

A stage similar to **beta**, when a software product is typically evaluated for performance, functionality, and compatibility in a non-mission-critical setting.

See Also [beta](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_beta).

Eiffel

A programming language including many object-oriented features. Some of its concepts are familiar to **Java** and **C#** developers. For the open-source Eiffel **API** for MySQL, see [Section 31.13, “MySQL Eiffel Wrapper”](https://dev.mysql.com/doc/refman/9.7/en/apis-eiffel.html "31.13 MySQL Eiffel Wrapper").

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [C#](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_csharp), [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java).

embedded

The embedded MySQL server library (**libmysqld**) makes it possible to run a full-featured MySQL server inside a **client** application. The main benefits are increased speed and more simple management for embedded applications.

See Also [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [libmysqld](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysqld).

error log

A type of **log** showing information about MySQL startup and critical runtime errors and **crash** information. For details, see [Section 7.4.2, “The Error Log”](https://dev.mysql.com/doc/refman/9.7/en/error-log.html "7.4.2 The Error Log").

See Also [crash](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash), [log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_log).

eviction

The process of removing an item from a cache or other temporary storage area, such as the `InnoDB` **buffer pool**. Often, but not always, uses the **LRU** algorithm to determine which item to remove. When a **dirty page** is evicted, its contents are **flushed** to disk, and any dirty **neighbor pages** might be flushed also.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [LRU](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lru), [neighbor page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_neighbor_page).

exception interceptor

A type of **interceptor** for tracing, debugging, or augmenting SQL errors encountered by a database application. For example, the interceptor code could issue a `SHOW WARNINGS` statement to retrieve additional information, and add descriptive text or even change the type of the exception returned to the application. Because the interceptor code is only called when SQL statements return errors, it does not impose any performance penalty on the application during normal (error-free) operation.

In **Java** applications using **Connector/J**, setting up this type of interceptor involves implementing the `com.mysql.jdbc.ExceptionInterceptor` interface, and adding a `exceptionInterceptors` property to the **connection string**.

In **Visual Studio** applications using **Connector/NET**, setting up this type of interceptor involves defining a class that inherits from the `BaseExceptionInterceptor` class and specifying that class name as part of the connection string.

See Also [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_interceptor), [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java), [Visual Studio](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_visual_studio).

exclusive lock

A kind of **lock** that prevents any other **transaction** from locking the same row. Depending on the transaction **isolation level**, this kind of lock might block other transactions from writing to the same row, or might also block other transactions from reading the same row. The default `InnoDB` isolation level, **REPEATABLE READ**, enables higher **concurrency** by allowing transactions to read rows that have exclusive locks, a technique known as **consistent read**.

See Also [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [shared lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shared_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

extent

A group of **pages** within a **tablespace**. For the default **page size** of 16KB, an extent contains 64 pages. In MySQL 5.6, the page size for an `InnoDB` instance can be 4KB, 8KB, or 16KB, controlled by the [`innodb_page_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_page_size) configuration option. For 4KB, 8KB, and 16KB pages sizes, the extent size is always 1MB (or 1048576 bytes).

Support for 32KB and 64KB `InnoDB` page sizes was added in MySQL 5.7.6. For a 32KB page size, the extent size is 2MB. For a 64KB page size, the extent size is 4MB.

`InnoDB` features such as **segments**, **read-ahead** requests and the **doublewrite buffer** use I/O operations that read, write, allocate, or free data one extent at a time.

See Also [doublewrite buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_doublewrite_buffer), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [page size](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page_size), [read-ahead](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_ahead), [segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_segment), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

### F

failover

The ability to automatically switch to a standby server in the event of a failure. In the MySQL context, failover involves a standby database server. Often supported within **J2EE** environments by the application server or framework.

See Also [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee).

Fast Index Creation

A capability first introduced in the InnoDB Plugin, now part of MySQL in 5.5 and higher, that speeds up creation of `InnoDB` **secondary indexes** by avoiding the need to completely rewrite the associated table. The speedup applies to dropping secondary indexes also.

Because index maintenance can add performance overhead to many data transfer operations, consider doing operations such as `ALTER TABLE ... ENGINE=INNODB` or `INSERT INTO ... SELECT * FROM ...` without any secondary indexes in place, and creating the indexes afterward.

In MySQL 5.6, this feature becomes more general. You can read and write to tables while an index is being created, and many more kinds of [`ALTER TABLE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement") operations can be performed without copying the table, without blocking **DML** operations, or both. Thus in MySQL 5.6 and higher, this set of features is referred to as **online DDL** rather than Fast Index Creation.

For related information, see [Section 17.12, “InnoDB and Online DDL”](https://dev.mysql.com/doc/refman/9.7/en/innodb-online-ddl.html "17.12 InnoDB and Online DDL").

See Also [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [online DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_online_ddl), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index).

fast shutdown

The default **shutdown** procedure for `InnoDB`, based on the configuration setting [`innodb_fast_shutdown=1`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_fast_shutdown). To save time, certain **flush** operations are skipped. This type of shutdown is safe during normal usage, because the flush operations are performed during the next startup, using the same mechanism as in **crash recovery**. In cases where the database is being shut down for an upgrade or downgrade, do a **slow shutdown** instead to ensure that all relevant changes are applied to the **data files** during the shutdown.

See Also [crash recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash_recovery), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown), [slow shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_slow_shutdown).

file-per-table

A general name for the setting controlled by the [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) option, which is an important configuration option that affects aspects of `InnoDB` file storage, availability of features, and I/O characteristics. As of MySQL 5.6.7, [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) is enabled by default.

With the [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) option enabled, you can create a table in its own **.ibd file** rather than in the shared **ibdata files** of the **system tablespace**. When table data is stored in an individual **.ibd file**, you have more flexibility to choose **row formats** required for features such as data **compression**. The `TRUNCATE TABLE` operation is also faster, and reclaimed space can be used by the operating system rather than remaining reserved for `InnoDB`.

The **MySQL Enterprise Backup** product is more flexible for tables that are in their own files. For example, tables can be excluded from a backup, but only if they are in separate files. Thus, this setting is suitable for tables that are backed up less frequently or on a different schedule.

See Also [ibdata file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ibdata_file), [innodb\_file\_per\_table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_file_per_table), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [row format](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_format).

fill factor

In an `InnoDB` **index**, the proportion of a **page** that is taken up by index data before the page is split. The unused space when index data is first divided between pages allows for rows to be updated with longer string values without requiring expensive index maintenance operations. If the fill factor is too low, the index consumes more space than needed, causing extra I/O overhead when reading the index. If the fill factor is too high, any update that increases the length of column values can cause extra I/O overhead for index maintenance. See [Section 17.6.2.2, “The Physical Structure of an InnoDB Index”](https://dev.mysql.com/doc/refman/9.7/en/innodb-physical-structure.html "17.6.2.2 The Physical Structure of an InnoDB Index") for more information.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

fixed row format

This row format is used by the `MyISAM` storage engine, not by `InnoDB`. If you create an `InnoDB` table with the option `ROW_FORMAT=FIXED` in MySQL 5.7.6 or earlier, `InnoDB` uses the **compact row format** instead, although the `FIXED` value might still show up in output such as `SHOW TABLE STATUS` reports. As of MySQL 5.7.7, `InnoDB` returns an error if `ROW_FORMAT=FIXED` is specified.

See Also [row format](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_format).

flush

To write changes to the database files, that had been buffered in a memory area or a temporary disk storage area. The `InnoDB` storage structures that are periodically flushed include the **redo log**, the **undo log**, and the **buffer pool**.

Flushing can happen because a memory area becomes full and the system needs to free some space, because a **commit** operation means the changes from a transaction can be finalized, or because a **slow shutdown** operation means that all outstanding work should be finalized. When it is not critical to flush all the buffered data at once, `InnoDB` can use a technique called **fuzzy checkpointing** to flush small batches of pages to spread out the I/O overhead.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [fuzzy checkpointing](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fuzzy_checkpointing), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log), [slow shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_slow_shutdown), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

flush list

An internal `InnoDB` data structure that tracks **dirty pages** in the **buffer pool**: that is, **pages** that have been changed and need to be written back out to disk. This data structure is updated frequently by `InnoDB` internal **mini-transactions**, and so is protected by its own **mutex** to allow concurrent access to the buffer pool.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [LRU](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lru), [mini-transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mini_transaction), [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [page cleaner](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page_cleaner).

foreign key

A type of pointer relationship, between rows in separate `InnoDB` tables. The foreign key relationship is defined on one column in both the **parent table** and the **child table**.

In addition to enabling fast lookup of related information, foreign keys help to enforce **referential integrity**, by preventing any of these pointers from becoming invalid as data is inserted, updated, and deleted. This enforcement mechanism is a type of **constraint**. A row that points to another table cannot be inserted if the associated foreign key value does not exist in the other table. If a row is deleted or its foreign key value changed, and rows in another table point to that foreign key value, the foreign key can be set up to prevent the deletion, cause the corresponding column values in the other table to become **null**, or automatically delete the corresponding rows in the other table.

One of the stages in designing a **normalized** database is to identify data that is duplicated, separate that data into a new table, and set up a foreign key relationship so that the multiple tables can be queried like a single table, using a **join** operation.

See Also [child table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_child_table), [FOREIGN KEY constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key_constraint), [join](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_join), [normalized](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_normalized), [NULL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_null), [parent table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_parent_table), [referential integrity](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_referential_integrity), [relational](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_relational).

FOREIGN KEY constraint

The type of **constraint** that maintains database consistency through a **foreign key** relationship. Like other kinds of constraints, it can prevent data from being inserted or updated if data would become inconsistent; in this case, the inconsistency being prevented is between data in multiple tables. Alternatively, when a **DML** operation is performed, `FOREIGN KEY` constraints can cause data in **child rows** to be deleted, changed to different values, or set to **null**, based on the `ON         CASCADE` option specified when creating the foreign key.

See Also [child table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_child_table), [constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_constraint), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [NULL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_null).

FTS

In most contexts, an acronym for **full-text search**. Sometimes in performance discussions, an acronym for **full table scan**.

See Also [full table scan](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_table_scan), [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search).

full backup

A **backup** that includes all the **tables** in each MySQL **database**, and all the databases in a MySQL **instance**. Contrast with **partial backup**.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [database](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_database), [instance](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_instance), [partial backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_partial_backup).

full table scan

An operation that requires reading the entire contents of a table, rather than just selected portions using an **index**. Typically performed either with small lookup tables, or in data warehousing situations with large tables where all available data is aggregated and analyzed. How frequently these operations occur, and the sizes of the tables relative to available memory, have implications for the algorithms used in query optimization and managing the **buffer pool**.

The purpose of indexes is to allow lookups for specific values or ranges of values within a large table, thus avoiding full table scans when practical.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index).

full-text search

The MySQL feature for finding words, phrases, Boolean combinations of words, and so on within table data, in a faster, more convenient, and more flexible way than using the SQL `LIKE` operator or writing your own application-level search algorithm. It uses the SQL function [`MATCH()`](https://dev.mysql.com/doc/refman/9.7/en/fulltext-search.html#function_match) and **FULLTEXT indexes**.

See Also [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index).

FULLTEXT index

The special kind of **index** that holds the **search index** in the MySQL **full-text search** mechanism. Represents the words from values of a column, omitting any that are specified as **stopwords**. Originally, only available for `MyISAM` tables. Starting in MySQL 5.6.4, it is also available for **InnoDB** tables.

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [search index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_search_index), [stopword](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_stopword).

fuzzy checkpointing

A technique that **flushes** small batches of **dirty pages** from the **buffer pool**, rather than flushing all dirty pages at once which would disrupt database processing.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush).

### G

GA

“Generally available”, the stage when a software product leaves **beta** and is available for sale, official support, and production use.

See Also [beta](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_beta).

GAC

Acronym for “Global Assembly Cache”. A central area for storing libraries (**assemblies**) on a **.NET** system. Physically consists of nested folders, treated as a single virtual folder by the **.NET** CLR.

See Also [.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos__net), [assembly](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_assembly).

gap

A place in an `InnoDB` **index** data structure where new values could be inserted. When you lock a set of rows with a statement such as `SELECT ... FOR UPDATE`, `InnoDB` can create locks that apply to the gaps as well as the actual values in the index. For example, if you select all values greater than 10 for update, a gap lock prevents another transaction from inserting a new value that is greater than 10. The **supremum record** and **infimum record** represent the gaps containing all values greater than or less than all the current index values.

See Also [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [gap lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap_lock), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [infimum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_infimum_record), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [supremum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_supremum_record).

gap lock

A **lock** on a **gap** between index records, or a lock on the gap before the first or after the last index record. For example, `SELECT c1 FROM t WHERE c1 BETWEEN 10 and         20 FOR UPDATE;` prevents other transactions from inserting a value of 15 into the column `t.c1`, whether or not there was already any such value in the column, because the gaps between all existing values in the range are locked. Contrast with **record lock** and **next-key lock**.

Gap locks are part of the tradeoff between performance and **concurrency**, and are used in some transaction **isolation levels** and not others.

See Also [gap](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap), [infimum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_infimum_record), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [next-key lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_next_key_lock), [record lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_record_lock), [supremum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_supremum_record).

general log

See [general query log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_query_log).

general query log

A type of **log** used for diagnosis and troubleshooting of SQL statements processed by the MySQL server. Can be stored in a file or in a database table. You must enable this feature through the [`general_log`](https://dev.mysql.com/doc/refman/9.7/en/server-system-variables.html#sysvar_general_log) configuration option to use it. You can disable it for a specific connection through the [`sql_log_off`](https://dev.mysql.com/doc/refman/9.7/en/server-system-variables.html#sysvar_sql_log_off) configuration option.

Records a broader range of queries than the **slow query log**. Unlike the **binary log**, which is used for replication, the general query log contains [`SELECT`](https://dev.mysql.com/doc/refman/9.7/en/select.html "15.2.13 SELECT Statement") statements and does not maintain strict ordering. For more information, see [Section 7.4.3, “The General Query Log”](https://dev.mysql.com/doc/refman/9.7/en/query-log.html "7.4.3 The General Query Log").

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_log), [slow query log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_slow_query_log).

general tablespace

A shared `InnoDB` **tablespace** created using [`CREATE TABLESPACE`](https://dev.mysql.com/doc/refman/9.7/en/create-tablespace.html "15.1.25 CREATE TABLESPACE Statement") syntax. General tablespaces can be created outside of the MySQL data directory, are capable of holding multiple **tables**, and support tables of all row formats. General tablespaces were introduced in MySQL 5.7.6.

Tables are added to a general tablespace using [``CREATE TABLE         *`tbl_name`* ... TABLESPACE [=]         *`tablespace_name`*``](https://dev.mysql.com/doc/refman/9.7/en/create-table.html "15.1.24 CREATE TABLE Statement") or [``ALTER TABLE         *`tbl_name`* TABLESPACE [=]         *`tablespace_name`*``](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement") syntax.

Contrast with **system tablespace** and **file-per-table** tablespace.

For more information, see [Section 17.6.3.3, “General Tablespaces”](https://dev.mysql.com/doc/refman/9.7/en/general-tablespaces.html "17.6.3.3 General Tablespaces").

See Also [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

Glassfish

See Also [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee).

global transaction

A type of **transaction** involved in **XA** operations. It consists of several actions that are transactional in themselves, but that all must either complete successfully as a group, or all be rolled back as a group. In essence, this extends **ACID** properties “up a level” so that multiple ACID transactions can be executed in concert as components of a global operation that also has ACID properties.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

group commit

An `InnoDB` optimization that performs some low-level I/O operations (log write) once for a set of **commit** operations, rather than flushing and syncing separately for each commit.

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit).

GUID

Acronym for “globally unique identifier”, an ID value that can be used to associate data across different databases, languages, operating systems, and so on. (As an alternative to using sequential integers, where the same values could appear in different tables, databases, and so on referring to different data.) Older MySQL versions represented it as `BINARY(16)`. Currently, it is represented as `CHAR(36)`. MySQL has a `UUID()` function that returns GUID values in character format, and a `UUID_SHORT()` function that returns GUID values in integer format. Because successive GUID values are not necessarily in ascending sort order, it is not an efficient value to use as a primary key for large InnoDB tables.

### H

hash index

A type of **index** intended for queries that use equality operators, rather than range operators such as greater-than or `BETWEEN`. It is available for [`MEMORY`](https://dev.mysql.com/doc/refman/9.7/en/memory-storage-engine.html "18.3 The MEMORY Storage Engine") tables. Although hash indexes are the default for [`MEMORY`](https://dev.mysql.com/doc/refman/9.7/en/memory-storage-engine.html "18.3 The MEMORY Storage Engine") tables for historic reasons, that storage engine also supports **B-tree** indexes, which are often a better choice for general-purpose queries.

MySQL includes a variant of this index type, the **adaptive hash index**, that is constructed automatically for [`InnoDB`](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") tables if needed based on runtime conditions.

See Also [adaptive hash index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_adaptive_hash_index), [B-tree](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_b_tree), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb).

HDD

Acronym for “hard disk drive”. Refers to storage media using spinning platters, usually when comparing and contrasting with **SSD**. Its performance characteristics can influence the throughput of a **disk-based** workload.

See Also [disk-based](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_based), [SSD](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ssd).

heartbeat

A periodic message that is sent to indicate that a system is functioning properly. In a **replication** context, if the **source** stops sending such messages, one of the **replicas** can take its place. Similar techniques can be used between the servers in a cluster environment, to confirm that all of them are operating properly.

See Also [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication), [source](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_source).

high-water mark

A value representing an upper limit, either a hard limit that should not be exceeded at runtime, or a record of the maximum value that was actually reached. Contrast with **low-water mark**.

See Also [low-water mark](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_low_water_mark).

history list

A list of **transactions** with delete-marked records scheduled to be processed by the `InnoDB` **purge** operation. Recorded in the **undo log**. The length of the history list is reported by the command `SHOW ENGINE INNODB STATUS`. If the history list grows longer than the value of the [`innodb_max_purge_lag`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_max_purge_lag) configuration option, each **DML** operation is delayed slightly to allow the purge operation to finish **flushing** the deleted records.

Also known as **purge lag**.

See Also [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [purge lag](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_purge_lag), [rollback segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback_segment), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

hole punching

Releasing empty blocks from a page. The `InnoDB` **transparent page compression** feature relies on hole punching support. For more information, see [Section 17.9.2, “InnoDB Page Compression”](https://dev.mysql.com/doc/refman/9.7/en/innodb-page-compression.html "17.9.2 InnoDB Page Compression").

See Also [sparse file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sparse_file), [transparent page compression](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transparent_page_compression).

host

The network name of a database server, used to establish a **connection**. Often specified in conjunction with a **port**. In some contexts, the IP address `127.0.0.1` works better than the special name `localhost` for accessing a database on the same server as the application.

See Also [connection](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection), [localhost](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_localhost), [port](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_port).

hot

A condition where a row, table, or internal data structure is accessed so frequently, requiring some form of locking or mutual exclusion, that it results in a performance or scalability issue.

Although “hot” typically indicates an undesirable condition, a **hot backup** is the preferred type of backup.

See Also [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup).

hot backup

A backup taken while the database is running and applications are reading and writing to it. The backup involves more than simply copying data files: it must include any data that was inserted or updated while the backup was in process; it must exclude any data that was deleted while the backup was in process; and it must ignore any changes that were not committed.

The Oracle product that performs hot backups, of `InnoDB` tables especially but also tables from `MyISAM` and other storage engines, is known as **MySQL Enterprise Backup**.

The hot backup process consists of two stages. The initial copying of the data files produces a **raw backup**. The **apply** step incorporates any changes to the database that happened while the backup was running. Applying the changes produces a **prepared** backup; these files are ready to be restored whenever necessary.

See Also [apply](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_apply), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [prepared backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_backup), [raw backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_raw_backup).

### I

.ibz file

When the **MySQL Enterprise Backup** product performs a **compressed backup**, it transforms each **tablespace** file that is created using the **file-per-table** setting from a `.ibd` extension to a `.ibz` extension.

The compression applied during backup is distinct from the **compressed row format** that keeps table data compressed during normal operation. A compressed backup operation skips the compression step for a tablespace that is already in compressed row format, as compressing a second time would slow down the backup but produce little or no space savings.

See Also [compressed backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_compressed_backup), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

I/O-bound

See [disk-bound](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_bound).

ib-file set

The set of files managed by `InnoDB` within a MySQL database: the **system tablespace**, **file-per-table** tablespace files, and **redo log** files. Depending on MySQL version and `InnoDB` configuration, may also include **general tablespace**, **temporary tablespace**, and **undo tablespace** files. This term is sometimes used in detailed discussions of `InnoDB` file structures and formats to refer to the set of files managed by `InnoDB` within a MySQL database.

See Also [database](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_database), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [general tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_tablespace), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log), [undo tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_tablespace).

ibbackup\_logfile

A supplemental backup file created by the **MySQL Enterprise Backup** product during a **hot backup** operation. It contains information about any data changes that occurred while the backup was running. The initial backup files, including `ibbackup_logfile`, are known as a **raw backup**, because the changes that occurred during the backup operation are not yet incorporated. After you perform the **apply** step to the raw backup files, the resulting files do include those final data changes, and are known as a **prepared backup**. At this stage, the `ibbackup_logfile` file is no longer necessary.

See Also [apply](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_apply), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [prepared backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_backup), [raw backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_raw_backup).

ibdata file

A set of files with names such as `ibdata1`, `ibdata2`, and so on, that make up the `InnoDB` **system tablespace**. For information about the structures and data that reside in the system tablespace `ibdata` files, see [Section 17.6.3.1, “The System Tablespace”](https://dev.mysql.com/doc/refman/9.7/en/innodb-system-tablespace.html "17.6.3.1 The System Tablespace").

Growth of the `ibdata` files is influenced by the [`innodb_autoextend_increment`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoextend_increment) configuration option.

See Also [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [doublewrite buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_doublewrite_buffer), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [innodb\_file\_per\_table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_file_per_table), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

ibtmp file

The `InnoDB` **temporary tablespace** **data file** for non-compressed `InnoDB` **temporary tables** and related objects. The configuration file option, [`innodb_temp_data_file_path`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_temp_data_file_path), allows users to define a relative path for the temporary tablespace data file. If [`innodb_temp_data_file_path`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_temp_data_file_path) is not specified, the default behavior is to create a single auto-extending 12MB data file named `ibtmp1` in the data directory, alongside `ibdata1`.

See Also [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [temporary table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_temporary_table).

ib\_logfile

A set of files, typically named `ib_logfile0` and `ib_logfile1`, that form the **redo log**. Also sometimes referred to as the **log group**. These files record statements that attempt to change data in `InnoDB` tables. These statements are replayed automatically to correct data written by incomplete transactions, on startup following a crash.

This data cannot be used for manual recovery; for that type of operation, use the **binary log**.

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [log group](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_log_group), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log).

ilist

Within an `InnoDB` **FULLTEXT index**, the data structure consisting of a document ID and positional information for a token (that is, a particular word).

See Also [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index).

implicit row lock

A row lock that `InnoDB` acquires to ensure consistency, without you specifically requesting it.

See Also [row lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_lock).

in-memory database

A type of database system that maintains data in memory, to avoid overhead due to disk I/O and translation between disk blocks and memory areas. Some in-memory databases sacrifice durability (the “D” in the **ACID** design philosophy) and are vulnerable to hardware, power, and other types of failures, making them more suitable for read-only operations. Other in-memory databases do use durability mechanisms such as logging changes to disk or using non-volatile memory.

MySQL features that address the same kinds of memory-intensive processing include the `InnoDB` **buffer pool**, **adaptive hash index**, and **read-only transaction** optimization, the [`MEMORY`](https://dev.mysql.com/doc/refman/9.7/en/memory-storage-engine.html "18.3 The MEMORY Storage Engine") storage engine, the `MyISAM` key cache, and the MySQL query cache.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [adaptive hash index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_adaptive_hash_index), [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [disk-based](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_based), [read-only transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_only_transaction).

incremental backup

A type of **hot backup**, performed by the **MySQL Enterprise Backup** product, that only saves data changed since some point in time. Having a full backup and a succession of incremental backups lets you reconstruct backup data over a long period, without the storage overhead of keeping several full backups on hand. You can restore the full backup and then apply each of the incremental backups in succession, or you can keep the full backup up-to-date by applying each incremental backup to it, then perform a single restore operation.

The granularity of changed data is at the **page** level. A page might actually cover more than one row. Each changed page is included in the backup.

See Also [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

index

A data structure that provides a fast lookup capability for **rows** of a **table**, typically by forming a tree structure (**B-tree)** representing all the values of a particular **column** or set of columns.

`InnoDB` tables always have a **clustered index** representing the **primary key**. They can also have one or more **secondary indexes** defined on one or more columns. Depending on their structure, secondary indexes can be classified as **partial**, **column**, or **composite** indexes.

Indexes are a crucial aspect of **query** performance. Database architects design tables, queries, and indexes to allow fast lookups for data needed by applications. The ideal database design uses a **covering index** where practical; the query results are computed entirely from the index, without reading the actual table data. Each **foreign key** constraint also requires an index, to efficiently check whether values exist in both the **parent** and **child** tables.

Although a B-tree index is the most common, a different kind of data structure is used for **hash indexes**, as in the `MEMORY` storage engine and the `InnoDB` **adaptive hash index**. **R-tree** indexes are used for spatial indexing of multi-dimensional information.

See Also [adaptive hash index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_adaptive_hash_index), [B-tree](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_b_tree), [child table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_child_table), [clustered index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clustered_index), [column index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_column_index), [composite index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_composite_index), [covering index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_covering_index), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [hash index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hash_index), [parent table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_parent_table), [partial index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_partial_index), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [R-tree](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_r_tree), [row](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index).

index cache

A memory area that holds the token data for `InnoDB` **full-text search**. It buffers the data to minimize disk I/O when data is inserted or updated in columns that are part of a **FULLTEXT index**. The token data is written to disk when the index cache becomes full. Each `InnoDB` `FULLTEXT` index has its own separate index cache, whose size is controlled by the configuration option [`innodb_ft_cache_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_ft_cache_size).

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search), [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index).

index condition pushdown

Index condition pushdown (ICP) is an optimization that pushes part of a `WHERE` condition down to the storage engine if parts of the condition can be evaluated using fields from the **index**. ICP can reduce the number of times the **storage engine** must access the base table and the number of times the MySQL server must access the storage engine. For more information, see [Section 10.2.1.6, “Index Condition Pushdown Optimization”](https://dev.mysql.com/doc/refman/9.7/en/index-condition-pushdown-optimization.html "10.2.1.6 Index Condition Pushdown Optimization").

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [storage engine](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_storage_engine).

index hint

Extended SQL syntax for overriding the **indexes** recommended by the optimizer. For example, the `FORCE INDEX`, `USE INDEX`, and `IGNORE         INDEX` clauses. Typically used when indexed columns have unevenly distributed values, resulting in inaccurate **cardinality** estimates.

See Also [cardinality](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cardinality), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index).

index prefix

In an **index** that applies to multiple columns (known as a **composite index**), the initial or leading columns of the index. A query that references the first 1, 2, 3, and so on columns of a composite index can use the index, even if the query does not reference all the columns in the index.

See Also [composite index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_composite_index), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index).

index statistics

See [statistics](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_statistics).

infimum record

A **pseudo-record** in an **index**, representing the **gap** below the smallest value in that index. If a transaction has a statement such as `SELECT ... FROM ... WHERE col < 10 FOR         UPDATE;`, and the smallest value in the column is 5, it is a lock on the infimum record that prevents other transactions from inserting even smaller values such as 0, -10, and so on.

See Also [gap](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [pseudo-record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_pseudo_record), [supremum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_supremum_record).

INFORMATION\_SCHEMA

The name of the **database** that provides a query interface to the MySQL **data dictionary**. (This name is defined by the ANSI SQL standard.) To examine information (metadata) about the database, you can query tables such as `INFORMATION_SCHEMA.TABLES` and `INFORMATION_SCHEMA.COLUMNS`, rather than using `SHOW` commands that produce unstructured output.

The `INFORMATION_SCHEMA` database also contains tables specific to **InnoDB** that provide a query interface to the `InnoDB` data dictionary. You use these tables not to see how the database is structured, but to get real-time information about the workings of `InnoDB` tables to help with performance monitoring, tuning, and troubleshooting.

See Also [database](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_database), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb).

InnoDB

A MySQL component that combines high performance with **transactional** capability for reliability, robustness, and concurrent access. It embodies the **ACID** design philosophy. Represented as a **storage engine**; it handles tables created or altered with the `ENGINE=INNODB` clause. See [Chapter 17, *The InnoDB Storage Engine*](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") for architectural details and administration procedures, and [Section 10.5, “Optimizing for InnoDB Tables”](https://dev.mysql.com/doc/refman/9.7/en/optimizing-innodb.html "10.5 Optimizing for InnoDB Tables") for performance advice.

In MySQL 5.5 and higher, `InnoDB` is the default storage engine for new tables and the `ENGINE=INNODB` clause is not required.

`InnoDB` tables are ideally suited for **hot backups**. See [Section 32.1, “MySQL Enterprise Backup Overview”](https://dev.mysql.com/doc/refman/9.7/en/mysql-enterprise-backup.html "32.1 MySQL Enterprise Backup Overview") for information about the **MySQL Enterprise Backup** product for backing up MySQL servers without interrupting normal processing.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [storage engine](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_storage_engine), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

innodb\_autoinc\_lock\_mode

The [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) option controls the algorithm used for **auto-increment locking**. When you have an auto-incrementing **primary key**, you can use statement-based replication only with the setting [`innodb_autoinc_lock_mode=1`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode). This setting is known as *consecutive* lock mode, because multi-row inserts within a transaction receive consecutive auto-increment values. If you have [`innodb_autoinc_lock_mode=2`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode), which allows higher concurrency for insert operations, use row-based replication rather than statement-based replication. This setting is known as *interleaved* lock mode, because multiple multi-row insert statements running at the same time can receive **auto-increment** values that are interleaved. The setting [`innodb_autoinc_lock_mode=0`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) should not be used except for compatibility purposes.

Consecutive lock mode ([`innodb_autoinc_lock_mode=1`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)) is the default setting prior to MySQL 8.0.3. As of MySQL 8.0.3, interleaved lock mode ([`innodb_autoinc_lock_mode=2`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)) is the default, which reflects the change from statement-based to row-based replication as the default replication type.

See Also [auto-increment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment), [auto-increment locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment_locking), [mixed-mode insert](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mixed_mode_insert), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key).

innodb\_file\_per\_table

An important configuration option that affects many aspects of `InnoDB` file storage, availability of features, and I/O characteristics. In MySQL 5.6.7 and higher, it is enabled by default. The [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) option turns on **file-per-table** mode. With this mode enabled, a newly created `InnoDB` table and associated indexes can be stored in a file-per-table **.ibd file**, outside the **system tablespace**.

This option affects the performance and storage considerations for a number of SQL statements, such as [`DROP TABLE`](https://dev.mysql.com/doc/refman/9.7/en/drop-table.html "15.1.37 DROP TABLE Statement") and [`TRUNCATE TABLE`](https://dev.mysql.com/doc/refman/9.7/en/truncate-table.html "15.1.42 TRUNCATE TABLE Statement").

Enabling the [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) option allows you to take advantage of features such as table **compression** and named-table backups in **MySQL Enterprise Backup**.

For more information, see [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_file_per_table), and [Section 17.6.3.2, “File-Per-Table Tablespaces”](https://dev.mysql.com/doc/refman/9.7/en/innodb-file-per-table-tablespaces.html "17.6.3.2 File-Per-Table Tablespaces").

See Also [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup).

innodb\_lock\_wait\_timeout

The [`innodb_lock_wait_timeout`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_lock_wait_timeout) option sets the balance between **waiting** for shared resources to become available, or giving up and handling the error, retrying, or doing alternative processing in your application. Rolls back any `InnoDB` transaction that waits more than a specified time to acquire a **lock**. Especially useful if **deadlocks** are caused by updates to multiple tables controlled by different storage engines; such deadlocks are not **detected** automatically.

See Also [deadlock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_deadlock), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [wait](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_wait).

innodb\_strict\_mode

The [`innodb_strict_mode`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_strict_mode) option controls whether `InnoDB` operates in **strict mode**, where conditions that are normally treated as warnings, cause errors instead (and the underlying statements fail).

See Also [strict mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_strict_mode).

Innovation Series

Innovation releases with the same major version form an Innovation series. For example, MySQL 8.1 through 8.3 form the MySQL 8 Innovation series.

See Also [LTS Series](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lts_series).

insert

One of the primary **DML** operations in **SQL**. The performance of inserts is a key factor in **data warehouse** systems that load millions of rows into tables, and **OLTP** systems where many concurrent connections might insert rows into the same table, in arbitrary order. If insert performance is important to you, you should learn about **InnoDB** features such as the **insert buffer** used in **change buffering**, and **auto-increment** columns.

See Also [auto-increment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment), [change buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffering), [data warehouse](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_warehouse), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [insert buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffer), [OLTP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_oltp), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

insert buffer

The former name of the **change buffer**. In MySQL 5.5, support was added for buffering changes to secondary index pages for [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement") and [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement") operations. Previously, only changes resulting from [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement") operations were buffered. The preferred term is now *change buffer*.

See Also [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [change buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffering).

insert buffering

The technique of storing changes to secondary index pages, resulting from [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement") operations, in the **change buffer** rather than writing the changes immediately, so that the physical writes can be performed to minimize random I/O. It is one of the types of **change buffering**; the others are **delete buffering** and **purge buffering**.

Insert buffering is not used if the secondary index is **unique**, because the uniqueness of new values cannot be verified before the new entries are written out. Other kinds of change buffering do work for unique indexes.

See Also [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [change buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffering), [delete buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_delete_buffering), [insert buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffer), [purge buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_purge_buffering), [unique index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_index).

insert intention lock

A type of **gap lock** that is set by [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement") operations prior to row insertion. This type of **lock** signals the intent to insert in such a way that multiple transactions inserting into the same index gap need not wait for each other if they are not inserting at the same position within the gap. For more information, see [Section 17.7.1, “InnoDB Locking”](https://dev.mysql.com/doc/refman/9.7/en/innodb-locking.html "17.7.1 InnoDB Locking").

See Also [gap lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap_lock), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [next-key lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_next_key_lock).

instance

A single **mysqld** daemon managing a **data directory** representing one or more **databases** with a set of **tables**. It is common in development, testing, and some **replication** scenarios to have multiple instances on the same **server** machine, each managing its own data directory and listening on its own port or socket. With one instance running a **disk-bound** workload, the server might still have extra CPU and memory capacity to run additional instances.

See Also [data directory](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_directory), [database](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_database), [disk-bound](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_bound), [mysqld](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqld), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication), [server](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_server).

instrumentation

Modifications at the source code level to collect performance data for tuning and debugging. In MySQL, data collected by instrumentation is exposed through an SQL interface using the `INFORMATION_SCHEMA` and `PERFORMANCE_SCHEMA` databases.

See Also [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_information_schema), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema).

intention exclusive lock

See [intention lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_intention_lock).

intention lock

A kind of **lock** that applies to the table, used to indicate the kind of lock the **transaction** intends to acquire on rows in the table. Different transactions can acquire different kinds of intention locks on the same table, but the first transaction to acquire an *intention exclusive* (IX) lock on a table prevents other transactions from acquiring any S or X locks on the table. Conversely, the first transaction to acquire an *intention shared* (IS) lock on a table prevents other transactions from acquiring any X locks on the table. The two-phase process allows the lock requests to be resolved in order, without blocking locks and corresponding operations that are compatible. For more information about this locking mechanism, see [Section 17.7.1, “InnoDB Locking”](https://dev.mysql.com/doc/refman/9.7/en/innodb-locking.html "17.7.1 InnoDB Locking").

See Also [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [lock mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock_mode), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

intention shared lock

See [intention lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_intention_lock).

interceptor

Code for instrumenting or debugging some aspect of an application, which can be enabled without recompiling or changing the source of the application itself.

See Also [command interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_command_interceptor), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [exception interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_exception_interceptor).

inverted index

A data structure optimized for document retrieval systems, used in the implementation of `InnoDB` **full-text search**. The `InnoDB` **FULLTEXT index**, implemented as an inverted index, records the position of each word within a document, rather than the location of a table row. A single column value (a document stored as a text string) is represented by many entries in the inverted index.

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search), [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index), [ilist](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ilist).

IOPS

Acronym for **I/O operations per second**. A common measurement for busy systems, particularly **OLTP** applications. If this value is near the maximum that the storage devices can handle, the application can become **disk-bound**, limiting **scalability**.

See Also [disk-bound](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_bound), [OLTP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_oltp), [scalability](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scalability).

isolation level

One of the foundations of database processing. Isolation is the **I** in the acronym **ACID**; the isolation level is the setting that fine-tunes the balance between performance and reliability, consistency, and reproducibility of results when multiple **transactions** are making changes and performing queries at the same time.

From highest amount of consistency and protection to the least, the isolation levels supported by InnoDB are: **SERIALIZABLE**, **REPEATABLE READ**, **READ COMMITTED**, and **READ UNCOMMITTED**.

With `InnoDB` tables, many users can keep the default isolation level (*REPEATABLE READ*) for all operations. Expert users might choose the **READ COMMITTED** level as they push the boundaries of scalability with **OLTP** processing, or during data warehousing operations where minor inconsistencies do not affect the aggregate results of large amounts of data. The levels on the edges (**SERIALIZABLE** and **READ UNCOMMITTED**) change the processing behavior to such an extent that they are rarely used.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [OLTP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_oltp), [READ COMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_committed), [READ UNCOMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_uncommitted), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [SERIALIZABLE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_serializable), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

### J

J2EE

Java Platform, Enterprise Edition: Oracle's enterprise Java platform. It consists of an API and a runtime environment for enterprise-class Java applications. For full details, see [http://www.oracle.com/technetwork/java/javaee/overview/index.html](http://www.oracle.com/technetwork/java/javaee/overview/index.html). With MySQL applications, you typically use **Connector/J** for database access, and an application server such as **Tomcat** or **JBoss** to handle the middle-tier work, and optionally a framework such as **Spring**. Database-related features often offered within a J2EE stack include a **connection pool** and **failover** support.

See Also [connection pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection_pool), [failover](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_failover), [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java), [JBoss](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_jboss), [Spring](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_spring), [Tomcat](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tomcat).

Java

A programming language combining high performance, rich built-in features and data types, object-oriented mechanisms, extensive standard library, and wide range of reusable third-party modules. Enterprise development is supported by many frameworks, application servers, and other technologies. Much of its syntax is familiar to **C** and **C++** developers. To write Java applications with MySQL, you use the **JDBC** driver known as **Connector/J**.

See Also [C](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_c), [C++](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cplusplus), [JDBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_jdbc).

JBoss

See Also [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee).

JDBC

Abbreviation for “Java Database Connectivity”, an **API** for database access from **Java** applications. Java developers writing MySQL applications use the **Connector/J** component as their JDBC driver.

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee), [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java).

JNDI

See Also [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java).

join

A **query** that retrieves data from more than one table, by referencing columns in the tables that hold identical values. Ideally, these columns are part of an `InnoDB` **foreign key** relationship, which ensures **referential integrity** and that the join columns are **indexed**. Often used to save space and improve query performance by replacing repeated strings with numeric IDs, in a **normalized** data design.

See Also [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [normalized](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_normalized), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [referential integrity](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_referential_integrity).

### K

keystore

See Also [SSL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ssl).

KEY\_BLOCK\_SIZE

An option to specify the size of data pages within an `InnoDB` table that uses **compressed row format**. The default is 8 kilobytes. Lower values risk hitting internal limits that depend on the combination of row size and compression percentage.

For [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") tables, `KEY_BLOCK_SIZE` optionally specifies the size in bytes to use for index key blocks. The value is treated as a hint; a different size could be used if necessary. A `KEY_BLOCK_SIZE` value specified for an individual index definition overrides a table-level `KEY_BLOCK_SIZE` value.

### L

latch

A lightweight structure used by `InnoDB` to implement a **lock** for its own internal memory structures, typically held for a brief time measured in milliseconds or microseconds. A general term that includes both **mutexes** (for exclusive access) and **rw-locks** (for shared access). Certain latches are the focus of `InnoDB` performance tuning. Statistics about latch use and contention are available through the **Performance Schema** interface.

See Also [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema), [rw-lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rw_lock).

libmysql

Informal name for the **libmysqlclient** library.

See Also [libmysqlclient](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysqlclient).

libmysqlclient

The library file, named `libmysqlclient.a` or `libmysqlclient.so`, that is typically linked into **client** programs written in **C**. Sometimes known informally as **libmysql** or the **mysqlclient** library.

See Also [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [libmysql](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysql), [mysqlclient](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlclient).

libmysqld

This **embedded** MySQL server library makes it possible to run a full-featured MySQL server inside a **client** application. The main benefits are increased speed and more simple management for embedded applications. You link with the `libmysqld` library rather than **libmysqlclient**. The API is identical between all three of these libraries.

See Also [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [embedded](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_embedded), [libmysql](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysql), [libmysqlclient](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysqlclient).

lifecycle interceptor

A type of **interceptor** supported by **Connector/J**. It involves implementing the interface `com.mysql.jdbc.ConnectionLifecycleInterceptor`.

See Also [interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_interceptor).

list

The `InnoDB` **buffer pool** is represented as a list of memory **pages**. The list is reordered as new pages are accessed and enter the buffer pool, as pages within the buffer pool are accessed again and are considered newer, and as pages that are not accessed for a long time are **evicted** from the buffer pool. The buffer pool is divided into **sublists**, and the replacement policy is a variation of the familiar **LRU** technique.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [eviction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_eviction), [LRU](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lru), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [sublist](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sublist).

load balancing

A technique for scaling read-only connections by sending query requests to different slave servers in a replication or Cluster configuration. With **Connector/J**, load balancing is enabled through the `com.mysql.jdbc.ReplicationDriver` class and controlled by the configuration property `loadBalanceStrategy`.

See Also [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee).

localhost

See Also [connection](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection).

lock

The high-level notion of an object that controls access to a resource, such as a table, row, or internal data structure, as part of a **locking** strategy. For intensive performance tuning, you might delve into the actual structures that implement locks, such as **mutexes** and **latches**.

See Also [latch](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_latch), [lock mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock_mode), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex).

lock escalation

An operation used in some database systems that converts many **row locks** into a single **table lock**, saving memory space but reducing concurrent access to the table. `InnoDB` uses a space-efficient representation for row locks, so that **lock** escalation is not needed.

See Also [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [row lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_lock), [table lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_table_lock).

lock mode

A shared (S) **lock** allows a **transaction** to read a row. Multiple transactions can acquire an S lock on that same row at the same time.

An exclusive (X) lock allows a transaction to update or delete a row. No other transaction can acquire any kind of lock on that same row at the same time.

**Intention locks** apply to the table, and are used to indicate what kind of lock the transaction intends to acquire on rows in the table. Different transactions can acquire different kinds of intention locks on the same table, but the first transaction to acquire an intention exclusive (IX) lock on a table prevents other transactions from acquiring any S or X locks on the table. Conversely, the first transaction to acquire an intention shared (IS) lock on a table prevents other transactions from acquiring any X locks on the table. The two-phase process allows the lock requests to be resolved in order, without blocking locks and corresponding operations that are compatible.

See Also [intention lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_intention_lock), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

locking

The system of protecting a **transaction** from seeing or changing data that is being queried or changed by other transactions. The **locking** strategy must balance reliability and consistency of database operations (the principles of the **ACID** philosophy) against the performance needed for good **concurrency**. Fine-tuning the locking strategy often involves choosing an **isolation level** and ensuring all your database operations are safe and reliable for that isolation level.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

locking read

A [`SELECT`](https://dev.mysql.com/doc/refman/9.7/en/select.html "15.2.13 SELECT Statement") statement that also performs a **locking** operation on an `InnoDB` table. Either `SELECT ...         FOR UPDATE` or `SELECT ...         LOCK IN SHARE MODE`. It has the potential to produce a **deadlock**, depending on the **isolation level** of the transaction. The opposite of a **non-locking read**. Not allowed for global tables in a **read-only transaction**.

`SELECT ... FOR SHARE` replaces `SELECT         ... LOCK IN SHARE MODE` in MySQL 8.0.1, but `LOCK IN SHARE MODE` remains available for backward compatibility.

See [Section 17.7.2.4, “Locking Reads”](https://dev.mysql.com/doc/refman/9.7/en/innodb-locking-reads.html "17.7.2.4 Locking Reads").

See Also [deadlock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_deadlock), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [non-locking read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_non_locking_read), [read-only transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_only_transaction).

log

In the `InnoDB` context, “log” or “log files” typically refers to the **redo log** represented by the **ib\_logfile *`N`*** files. Another type of `InnoDB` log is the **undo log**, which is a storage area that holds copies of data modified by active transactions.

Other kinds of logs that are important in MySQL are the **error log** (for diagnosing startup and runtime problems), **binary log** (for working with replication and performing point-in-time restores), the **general query log** (for diagnosing application problems), and the **slow query log** (for diagnosing performance problems).

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [error log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_error_log), [general query log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_query_log), [ib\_logfile](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ib_logfile), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log), [slow query log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_slow_query_log), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

log buffer

The memory area that holds data to be written to the **log files** that make up the **redo log**. It is controlled by the [`innodb_log_buffer_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_log_buffer_size) configuration option.

See Also [log file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_log_file), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log).

log file

One of the **ib\_logfile *`N`*** files that make up the **redo log**. Data is written to these files from the **log buffer** memory area.

See Also [ib\_logfile](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ib_logfile), [log buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_log_buffer), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log).

log group

The set of files that make up the **redo log**, typically named `ib_logfile0` and `ib_logfile1`. (For that reason, sometimes referred to collectively as **ib\_logfile**.)

See Also [ib\_logfile](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ib_logfile), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log).

logical

A type of operation that involves high-level, abstract aspects such as tables, queries, indexes, and other SQL concepts. Typically, logical aspects are important to make database administration and application development convenient and usable. Contrast with **physical**.

See Also [logical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_logical_backup), [physical](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_physical).

logical backup

A **backup** that reproduces table structure and data, without copying the actual data files. For example, the **`mysqldump`** command produces a logical backup, because its output contains statements such as [`CREATE TABLE`](https://dev.mysql.com/doc/refman/9.7/en/create-table.html "15.1.24 CREATE TABLE Statement") and [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement") that can re-create the data. Contrast with **physical backup**. A logical backup offers flexibility (for example, you could edit table definitions or insert statements before restoring), but can take substantially longer to **restore** than a physical backup.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [mysqldump](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqldump), [physical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_physical_backup), [restore](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_restore).

loose\_

A prefix added to `InnoDB` configuration options after server **startup**, so any new configuration options not recognized by the current level of MySQL do not cause a startup failure. MySQL processes configuration options that start with this prefix, but gives a warning rather than a failure if the part after the prefix is not a recognized option.

See Also [startup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_startup).

low-water mark

A value representing a lower limit, typically a threshold value at which some corrective action begins or becomes more aggressive. Contrast with **high-water mark**.

See Also [high-water mark](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_high_water_mark).

LRU

An acronym for “least recently used”, a common method for managing storage areas. The items that have not been used recently are **evicted** when space is needed to cache newer items. `InnoDB` uses the LRU mechanism by default to manage the **pages** within the **buffer pool**, but makes exceptions in cases where a page might be read only a single time, such as during a **full table scan**. This variation of the LRU algorithm is called the **midpoint insertion strategy**. For more information, see [Section 17.5.1, “Buffer Pool”](https://dev.mysql.com/doc/refman/9.7/en/innodb-buffer-pool.html "17.5.1 Buffer Pool").

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [eviction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_eviction), [full table scan](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_table_scan), [midpoint insertion strategy](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_midpoint_insertion_strategy), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

LSN

Acronym for “log sequence number”. This arbitrary, ever-increasing value represents a point in time corresponding to operations recorded in the **redo log**. (This point in time is regardless of **transaction** boundaries; it can fall in the middle of one or more transactions.) It is used internally by `InnoDB` during **crash recovery** and for managing the **buffer pool**.

Prior to MySQL 5.6.3, the LSN was a 4-byte unsigned integer. The LSN became an 8-byte unsigned integer in MySQL 5.6.3 when the redo log file size limit increased from 4GB to 512GB, as additional bytes were required to store extra size information. Applications built on MySQL 5.6.3 or later that use LSN values should use 64-bit rather than 32-bit variables to store and compare LSN values.

In the **MySQL Enterprise Backup** product, you can specify an LSN to represent the point in time from which to take an **incremental backup**. The relevant LSN is displayed by the output of the **mysqlbackup** command. Once you have the LSN corresponding to the time of a full backup, you can specify that value to take a subsequent incremental backup, whose output contains another LSN for the next incremental backup.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [crash recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash_recovery), [incremental backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_incremental_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

LTS Series

LTS releases with the same major version number form an LTS series. For example, all MySQL 8.4.x releases form the MySQL 8.4 LTS series.

Note: MySQL 8.0 is a Bugfix series that preceded the LTS release model.

See Also [Innovation Series](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innovation_series).

### M

.MRG file

A file containing references to other tables, used by the `MERGE` storage engine. Files with this extension are always included in backups produced by the **mysqlbackup** command of the **MySQL Enterprise Backup** product.

See Also [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command).

.MYD file

A file that MySQL uses to store data for a [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") table.

See Also [.MYI file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_myi_file), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command).

.MYI file

A file that MySQL uses to store indexes for a [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") table.

See Also [.MYD file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_myd_file), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command).

master

See [source](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_source).

master thread

An `InnoDB` **thread** that performs various tasks in the background. Most of these tasks are I/O related, such as writing changes from the **change buffer** to the appropriate secondary indexes.

To improve **concurrency**, sometimes actions are moved from the master thread to separate background threads. For example, in MySQL 5.6 and higher, **dirty pages** are **flushed** from the **buffer pool** by the **page cleaner** thread rather than the master thread.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [page cleaner](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page_cleaner), [thread](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_thread).

MDL

Acronym for “metadata lock”.

See Also [metadata lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_metadata_lock).

medium trust

Synonym for **partial trust**. Because the range of trust settings is so broad, “partial trust” is preferred, to avoid the implication that there are only three levels (low, medium, and full).

See Also [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [partial trust](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_partial_trust).

merge

To apply changes to data cached in memory, such as when a page is brought into the **buffer pool**, and any applicable changes recorded in the **change buffer** are incorporated into the page in the buffer pool. The updated data is eventually written to the **tablespace** by the **flush** mechanism.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

metadata lock

A type of **lock** that prevents **DDL** operations on a table that is being used at the same time by another **transaction**. For details, see [Section 10.11.4, “Metadata Locking”](https://dev.mysql.com/doc/refman/9.7/en/metadata-locking.html "10.11.4 Metadata Locking").

Enhancements to **online** operations, particularly in MySQL 5.6 and higher, are focused on reducing the amount of metadata locking. The objective is for DDL operations that do not change the table structure (such as [`CREATE INDEX`](https://dev.mysql.com/doc/refman/9.7/en/create-index.html "15.1.18 CREATE INDEX Statement") and [`DROP INDEX`](https://dev.mysql.com/doc/refman/9.7/en/drop-index.html "15.1.31 DROP INDEX Statement") for `InnoDB` tables) to proceed while the table is being queried, updated, and so on by other transactions.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [online](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_online), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

metrics counter

A feature implemented by the [`INNODB_METRICS`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-innodb-metrics-table.html "28.4.21 The INFORMATION_SCHEMA INNODB_METRICS Table") table in the **INFORMATION\_SCHEMA**, in MySQL 5.6 and higher. You can query **counts** and totals for low-level `InnoDB` operations, and use the results for performance tuning in combination with data from the **Performance Schema**.

See Also [counter](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_counter), [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_information_schema), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema).

midpoint insertion strategy

The technique of initially bringing **pages** into the `InnoDB` **buffer pool** not at the “newest” end of the list, but instead somewhere in the middle. The exact location of this point can vary, based on the setting of the [`innodb_old_blocks_pct`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_old_blocks_pct) option. The intent is that pages that are only read once, such as during a **full table scan**, can be aged out of the buffer pool sooner than with a strict **LRU** algorithm. For more information, see [Section 17.5.1, “Buffer Pool”](https://dev.mysql.com/doc/refman/9.7/en/innodb-buffer-pool.html "17.5.1 Buffer Pool").

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [full table scan](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_table_scan), [LRU](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lru), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

mini-transaction

An internal phase of `InnoDB` processing, when making changes at the **physical** level to internal data structures during **DML** operations. A mini-transaction (mtr) has no notion of **rollback**; multiple mini-transactions can occur within a single **transaction**. Mini-transactions write information to the **redo log** that is used during **crash recovery**. A mini-transaction can also happen outside the context of a regular transaction, for example during **purge** processing by background threads.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [crash recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash_recovery), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [physical](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_physical), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

mixed-mode insert

An [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement") statement where **auto-increment** values are specified for some but not all of the new rows. For example, a multi-value `INSERT` could specify a value for the auto-increment column in some cases and `NULL` in other cases. `InnoDB` generates auto-increment values for the rows where the column value was specified as `NULL`. Another example is an [`INSERT ...         ON DUPLICATE KEY UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/insert-on-duplicate.html "15.2.7.2 INSERT ... ON DUPLICATE KEY UPDATE Statement") statement, where auto-increment values might be generated but not used, for any duplicate rows that are processed as `UPDATE` rather than `INSERT` statements.

Can cause consistency issues between **source** and **replica** servers in a **replication** configuration. Can require adjusting the value of the **innodb\_autoinc\_lock\_mode** configuration option.

See Also [auto-increment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment), [innodb\_autoinc\_lock\_mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_autoinc_lock_mode), [replica](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replica), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication), [source](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_source).

MM.MySQL

An older JDBC driver for MySQL that evolved into **Connector/J** when it was integrated with the MySQL product.

Mono

An Open Source framework developed by Novell, that works with **Connector/NET** and **C#** applications on Linux platforms.

See Also [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [C#](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_csharp).

mtr

See [mini-transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mini_transaction).

multi-core

A type of processor that can take advantage of multithreaded programs, such as the MySQL server.

multiversion concurrency control

See [MVCC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mvcc).

mutex

Informal abbreviation for “mutex variable”. (Mutex itself is short for “mutual exclusion”.) The low-level object that `InnoDB` uses to represent and enforce exclusive-access **locks** to internal in-memory data structures. Once the lock is acquired, any other process, thread, and so on is prevented from acquiring the same lock. Contrast with **rw-locks**, which `InnoDB` uses to represent and enforce shared-access **locks** to internal in-memory data structures. Mutexes and rw-locks are known collectively as **latches**.

See Also [latch](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_latch), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema), [Pthreads](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_pthreads), [rw-lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rw_lock).

MVCC

Acronym for “multiversion concurrency control”. This technique lets `InnoDB` **transactions** with certain **isolation levels** perform **consistent read** operations; that is, to query rows that are being updated by other transactions, and see the values from before those updates occurred. This is a powerful technique to increase **concurrency**, by allowing queries to proceed without waiting due to **locks** held by the other transactions.

This technique is not universal in the database world. Some other database products, and some other MySQL storage engines, do not support it.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

my.cnf

The name, on Unix or Linux systems, of the MySQL **option file**.

See Also [my.ini](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_my_ini), [option file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_option_file).

my.ini

The name, on Windows systems, of the MySQL **option file**.

See Also [my.cnf](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_my_cnf), [option file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_option_file).

MyODBC drivers

Obsolete name for **Connector/ODBC**.

See Also [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc).

mysql

The [**mysql**](https://dev.mysql.com/doc/refman/9.7/en/mysql.html "6.5.1 mysql — The MySQL Command-Line Client") program is the command-line interpreter for the MySQL database. It processes **SQL** statements, and also MySQL-specific commands such as [`SHOW         TABLES`](https://dev.mysql.com/doc/refman/9.7/en/show-tables.html "15.7.7.40 SHOW TABLES Statement"), by passing requests to the [**mysqld**](https://dev.mysql.com/doc/refman/9.7/en/mysqld.html "6.3.1 mysqld — The MySQL Server") daemon.

See Also [mysqld](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqld), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

mysqlbackup command

A command-line tool of the **MySQL Enterprise Backup** product. It performs a **hot backup** operation for [`InnoDB`](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") tables, and a [warm backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_warm_backup "warm backup") for [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") and other kinds of tables. See [Section 32.1, “MySQL Enterprise Backup Overview”](https://dev.mysql.com/doc/refman/9.7/en/mysql-enterprise-backup.html "32.1 MySQL Enterprise Backup Overview") for more information about this command.

See Also [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [warm backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_warm_backup).

mysqlclient

The informal name for the library that is implemented by the file **libmysqlclient**, with extension `.a` or `.so`.

See Also [libmysqlclient](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysqlclient).

mysqld

[**mysqld**](https://dev.mysql.com/doc/refman/9.7/en/mysqld.html "6.3.1 mysqld — The MySQL Server"), also known as MySQL Server, is a single multithreaded program that does most of the work in a MySQL installation. It does not spawn additional processes. MySQL Server manages access to the MySQL data directory that contains databases, tables, and other information such as log files and status files.

[**mysqld**](https://dev.mysql.com/doc/refman/9.7/en/mysqld.html "6.3.1 mysqld — The MySQL Server") runs as a Unix daemon or Windows service, constantly waiting for requests and performing maintenance work in the background.

See Also [instance](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_instance), [mysql](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql).

MySQLdb

The name of the open-source **Python** module that forms the basis of the MySQL **Python API**.

See Also [Python](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_python), [Python API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_python_api).

mysqldump

A command that performs a **logical backup** of some combination of databases, tables, and table data. The results are SQL statements that reproduce the original schema objects, data, or both. For substantial amounts of data, a **physical backup** solution such as **MySQL Enterprise Backup** is faster, particularly for the **restore** operation.

See Also [logical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_logical_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [physical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_physical_backup), [restore](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_restore).

### N

.NET

See Also [ADO.NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ado_net), [ASP.net](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_asp_net), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [Mono](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mono), [Visual Studio](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_visual_studio).

native C API

Synonym for **libmysqlclient**.

See Also [libmysql](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysql).

natural key

An indexed column, typically a **primary key**, where the values have some real-world significance. Usually advised against because:

- If the value should ever change, there is potentially a lot of index maintenance to re-sort the **clustered index** and update the copies of the primary key value that are repeated in each **secondary index**.
- Even seemingly stable values can change in unpredictable ways that are difficult to represent correctly in the database. For example, one country can change into two or several, making the original country code obsolete. Or, rules about unique values might have exceptions. For example, even if taxpayer IDs are intended to be unique to a single person, a database might have to handle records that violate that rule, such as in cases of identity theft. Taxpayer IDs and other sensitive ID numbers also make poor primary keys, because they may need to be secured, encrypted, and otherwise treated differently than other columns.

Thus, it is typically better to use arbitrary numeric values to form a **synthetic key**, for example using an **auto-increment** column.

See Also [auto-increment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment), [clustered index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clustered_index), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index), [synthetic key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_synthetic_key).

neighbor page

Any **page** in the same **extent** as a particular page. When a page is selected to be **flushed**, any neighbor pages that are **dirty** are typically flushed as well, as an I/O optimization for traditional hard disks. In MySQL 5.6 and up, this behavior can be controlled by the configuration variable [`innodb_flush_neighbors`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_flush_neighbors); you might turn that setting off for SSD drives, which do not have the same overhead for writing smaller batches of data at random locations.

See Also [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [extent](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_extent), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

next-key lock

A combination of a **record lock** on the index record and a [gap lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap_lock "gap lock") on the gap before the index record.

See Also [gap lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap_lock), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [record lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_record_lock).

non-locking read

A **query** that does not use the `SELECT ... FOR UPDATE` or `SELECT ...         LOCK IN SHARE MODE` clauses. The only kind of query allowed for global tables in a **read-only transaction**. The opposite of a **locking read**. See [Section 17.7.2.3, “Consistent Nonlocking Reads”](https://dev.mysql.com/doc/refman/9.7/en/innodb-consistent-read.html "17.7.2.3 Consistent Nonlocking Reads").

`SELECT ... FOR SHARE` replaces `SELECT         ... LOCK IN SHARE MODE` in MySQL 8.0.1, but `LOCK IN SHARE MODE` remains available for backward compatibility.

See Also [locking read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking_read), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [read-only transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_only_transaction).

non-repeatable read

The situation when a query retrieves data, and a later query within the same **transaction** retrieves what should be the same data, but the queries return different results (changed by another transaction committing in the meantime).

This kind of operation goes against the **ACID** principle of database design. Within a transaction, data should be consistent, with predictable and stable relationships.

Among different **isolation levels**, non-repeatable reads are prevented by the **serializable read** and **repeatable read** levels, and allowed by the **consistent read**, and **read uncommitted** levels.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [READ UNCOMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_uncommitted), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [SERIALIZABLE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_serializable), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

nonblocking I/O

An industry term that means the same as **asynchronous I/O**.

See Also [asynchronous I/O](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_asynchronous_io).

normalized

A database design strategy where data is split into multiple tables, and duplicate values condensed into single rows represented by an ID, to avoid storing, querying, and updating redundant or lengthy values. It is typically used in **OLTP** applications.

For example, an address might be given a unique ID, so that a census database could represent the relationship **lives at this address** by associating that ID with each member of a family, rather than storing multiple copies of a complex value such as **123 Main Street, Anytown, USA**.

For another example, although a simple address book application might store each phone number in the same table as a person's name and address, a phone company database might give each phone number a special ID, and store the numbers and IDs in a separate table. This normalized representation could simplify large-scale updates when area codes split apart.

Normalization is not always recommended. Data that is primarily queried, and only updated by deleting entirely and reloading, is often kept in fewer, larger tables with redundant copies of duplicate values. This data representation is referred to as **denormalized**, and is frequently found in data warehousing applications.

See Also [denormalized](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_denormalized), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [OLTP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_oltp), [relational](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_relational).

NoSQL

A broad term for a set of data access technologies that do not use the **SQL** language as their primary mechanism for reading and writing data. Some NoSQL technologies act as key-value stores, only accepting single-value reads and writes; some relax the restrictions of the **ACID** methodology; still others do not require a pre-planned **schema**. MySQL users can combine NoSQL-style processing for speed and simplicity with SQL operations for flexibility and convenience, by using the **memcached** API to directly access some kinds of MySQL tables.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_schema), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

NOT NULL constraint

A type of **constraint** that specifies that a **column** cannot contain any **NULL** values. It helps to preserve **referential integrity**, as the database server can identify data with erroneous missing values. It also helps in the arithmetic involved in query optimization, allowing the optimizer to predict the number of entries in an index on that column.

See Also [column](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_column), [constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_constraint), [NULL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_null), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [referential integrity](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_referential_integrity).

NULL

A special value in **SQL**, indicating the absence of data. Any arithmetic operation or equality test involving a `NULL` value, in turn produces a `NULL` result. (Thus it is similar to the IEEE floating-point concept of NaN, “not a number”.) Any aggregate calculation such as `AVG()` ignores rows with `NULL` values, when determining how many rows to divide by. The only test that works with `NULL` values uses the SQL idioms `IS         NULL` or `IS NOT NULL`.

`NULL` values play a part in **index** operations, because for performance a database must minimize the overhead of keeping track of missing data values. Typically, `NULL` values are not stored in an index, because a query that tests an indexed column using a standard comparison operator could never match a row with a `NULL` value for that column. For the same reason, unique indexes do not prevent `NULL` values; those values simply are not represented in the index. Declaring a `NOT         NULL` constraint on a column provides reassurance that there are no rows left out of the index, allowing for better query optimization (accurate counting of rows and estimation of whether to use the index).

Because the **primary key** must be able to uniquely identify every row in the table, a single-column primary key cannot contain any `NULL` values, and a multi-column primary key cannot contain any rows with `NULL` values in all columns.

Although the Oracle database allows a `NULL` value to be concatenated with a string, `InnoDB` treats the result of such an operation as `NULL`.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

### O

.OPT file

A file containing database configuration information. Files with this extension are included in backups produced by the **mysqlbackup** command of the **MySQL Enterprise Backup** product.

See Also [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command).

ODBC

Acronym for Open Database Connectivity, an industry-standard API. Typically used with Windows-based servers, or applications that require ODBC to communicate with MySQL. The MySQL ODBC driver is called **Connector/ODBC**.

See Also [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc).

off-page column

A column containing variable-length data (such as [`BLOB`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types") and [`VARCHAR`](https://dev.mysql.com/doc/refman/9.7/en/char.html "13.3.2 The CHAR and VARCHAR Types")) that is too long to fit on a **B-tree** page. The data is stored in **overflow pages**. The **DYNAMIC** row format is more efficient for such storage than the older **COMPACT** row format.

See Also [B-tree](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_b_tree), [overflow page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_overflow_page).

OLTP

Acronym for “Online Transaction Processing”. A database system, or a database application, that runs a workload with many **transactions**, with frequent writes as well as reads, typically affecting small amounts of data at a time. For example, an airline reservation system or an application that processes bank deposits. The data might be organized in **normalized** form for a balance between **DML** (insert/update/delete) efficiency and **query** efficiency. Contrast with **data warehouse**.

With its **row-level locking** and **transactional** capability, **InnoDB** is the ideal storage engine for MySQL tables used in OLTP applications.

See Also [data warehouse](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_warehouse), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [row lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

online

A type of operation that involves no downtime, blocking, or restricted operation for the database. Typically applied to **DDL**. Operations that shorten the periods of restricted operation, such as **fast index creation**, have evolved into a wider set of **online DDL** operations in MySQL 5.6.

In the context of backups, a **hot backup** is an online operation and a **warm backup** is partially an online operation.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [Fast Index Creation](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fast_index_creation), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [online DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_online_ddl), [warm backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_warm_backup).

online DDL

A feature that improves the performance, concurrency, and availability of `InnoDB` tables during **DDL** (primarily [`ALTER TABLE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement")) operations. See [Section 17.12, “InnoDB and Online DDL”](https://dev.mysql.com/doc/refman/9.7/en/innodb-online-ddl.html "17.12 InnoDB and Online DDL") for details.

The details vary according to the type of operation. In some cases, the table can be modified concurrently while the [`ALTER TABLE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement") is in progress. The operation might be able to be performed without a table copy, or using a specially optimized type of table copy. DML log space usage for in-place operations is controlled by the [`innodb_online_alter_log_max_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_online_alter_log_max_size) configuration option.

This feature is an enhancement of the **Fast Index Creation** feature in MySQL 5.5.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [Fast Index Creation](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fast_index_creation), [online](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_online).

optimistic

A methodology that guides low-level implementation decisions for a relational database system. The requirements of performance and **concurrency** in a relational database mean that operations must be started or dispatched quickly. The requirements of consistency and **referential integrity** mean that any operation could fail: a transaction might be rolled back, a **DML** operation could violate a constraint, a request for a lock could cause a deadlock, a network error could cause a timeout. An optimistic strategy is one that assumes most requests or attempts succeed, so that relatively little work is done to prepare for the failure case. When this assumption is true, the database does little unnecessary work; when requests do fail, extra work must be done to clean up and undo changes.

`InnoDB` uses optimistic strategies for operations such as **locking** and **commits**. For example, data changed by a transaction can be written to the data files before the commit occurs, making the commit itself very fast, but requiring more work to undo the changes if the transaction is rolled back.

The opposite of an optimistic strategy is a **pessimistic** one, where a system is optimized to deal with operations that are unreliable and frequently unsuccessful. This methodology is rare in a database system, because so much care goes into choosing reliable hardware, networks, and algorithms.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [pessimistic](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_pessimistic), [referential integrity](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_referential_integrity).

optimizer

The MySQL component that determines the best **indexes** and **join** order to use for a **query**, based on characteristics and data distribution of the relevant **tables**.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [join](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_join), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query).

option

A configuration parameter for MySQL, either stored in the **option file** or passed on the command line.

For the **options** that apply to **InnoDB** tables, each option name starts with the prefix `innodb_`.

See Also [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [option](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_option), [option file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_option_file).

option file

The file that holds the configuration **options** for the MySQL instance. Traditionally, on Linux and Unix this file is named `my.cnf`, and on Windows it is named `my.ini`.

See Also [configuration file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_configuration_file), [my.cnf](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_my_cnf), [my.ini](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_my_ini), [option](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_option).

overflow page

Separately allocated disk **pages** that hold variable-length columns (such as `BLOB` and `VARCHAR`) that are too long to fit on a **B-tree** page. The associated columns are known as **off-page columns**.

See Also [B-tree](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_b_tree), [off-page column](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_off_page_column), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

### P

.par file

A file containing partition definitions. Files with this extension are included in backups produced by the `mysqlbackup` command of the **MySQL Enterprise Backup** product.

With the introduction of native partitioning support for `InnoDB` tables in MySQL 5.7.6, `.par` files are no longer created for partitioned `InnoDB` tables. Partitioned `MyISAM` tables continue to use `.par` files in MySQL 5.7. In MySQL 8.0, partitioning support is only provided by the `InnoDB` storage engine. As such, `.par` files are no longer used as of MySQL 8.0.

See Also [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command).

page

A unit representing how much data `InnoDB` transfers at any one time between disk (the **data files**) and memory (the **buffer pool**). A page can contain one or more **rows**, depending on how much data is in each row. If a row does not fit entirely into a single page, `InnoDB` sets up additional pointer-style data structures so that the information about the row can be stored in one page.

One way to fit more data in each page is to use **compressed row format**. For tables that use BLOBs or large text fields, **compact row format** allows those large columns to be stored separately from the rest of the row, reducing I/O overhead and memory usage for queries that do not reference those columns.

When `InnoDB` reads or writes sets of pages as a batch to increase I/O throughput, it reads or writes an **extent** at a time.

All the `InnoDB` disk data structures within a MySQL instance share the same **page size**.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [extent](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_extent), [page size](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page_size), [row](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row).

page cleaner

An `InnoDB` background **thread** that **flushes** **dirty pages** from the **buffer pool**. Prior to MySQL 5.6, this activity was performed by the **master thread**. The number of page cleaner threads is controlled by the [`innodb_page_cleaners`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_page_cleaners) configuration option, introduced in MySQL 5.7.4.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [master thread](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_master_thread), [thread](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_thread).

page size

For releases up to and including MySQL 5.5, the size of each `InnoDB` **page** is fixed at 16 kilobytes. This value represents a balance: large enough to hold the data for most rows, yet small enough to minimize the performance overhead of transferring unneeded data to memory. Other values are not tested or supported.

Starting in MySQL 5.6, the page size for an `InnoDB` **instance** can be either 4KB, 8KB, or 16KB, controlled by the [`innodb_page_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_page_size) configuration option. As of MySQL 5.7.6, `InnoDB` also supports 32KB and 64KB page sizes. For 32KB and 64KB page sizes, `ROW_FORMAT=COMPRESSED` is not supported and the maximum record size is 16KB.

Page size is set when creating the MySQL instance, and it remains constant afterward. The same page size applies to all `InnoDB` **tablespaces**, including the **system tablespace**, **file-per-table** tablespaces, and **general tablespaces**.

Smaller page sizes can help performance with storage devices that use small block sizes, particularly for **SSD** devices in **disk-bound** workloads, such as for **OLTP** applications. As individual rows are updated, less data is copied into memory, written to disk, reorganized, locked, and so on.

See Also [disk-bound](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_bound), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [general tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_tablespace), [instance](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_instance), [OLTP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_oltp), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [SSD](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ssd), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

parent table

The table in a **foreign key** relationship that holds the initial column values pointed to from the **child table**. The consequences of deleting, or updating rows in the parent table depend on the `ON UPDATE` and `ON         DELETE` clauses in the foreign key definition. Rows with corresponding values in the child table could be automatically deleted or updated in turn, or those columns could be set to `NULL`, or the operation could be prevented.

See Also [child table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_child_table), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key).

partial backup

A **backup** that contains some of the **tables** in a MySQL database, or some of the databases in a MySQL instance. Contrast with **full backup**.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [full backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_backup).

partial index

An **index** that represents only part of a column value, typically the first N characters (the **prefix**) of a long `VARCHAR` value.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [index prefix](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index_prefix).

partial trust

An execution environment typically used by hosting providers, where applications have some permissions but not others. For example, applications might be able to access a database server over a network, but be “sandboxed” with regard to reading and writing local files.

See Also [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net).

Performance Schema

The `performance_schema` schema, in MySQL 5.5 and up, presents a set of tables that you can query to get detailed information about the performance characteristics of many internal parts of the MySQL server. See [Chapter 29, *MySQL Performance Schema*](https://dev.mysql.com/doc/refman/9.7/en/performance-schema.html "Chapter 29 MySQL Performance Schema").

See Also [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_information_schema), [latch](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_latch), [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex), [rw-lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rw_lock).

Perl

A programming language with roots in Unix scripting and report generation. Incorporates high-performance regular expressions and file I/O. Large collection of reusable modules available through repositories such as CPAN.

See Also [Perl API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_perl_api).

Perl API

An open-source **API** for MySQL applications written in the **Perl** language. Implemented through the `DBI` and `DBD::mysql` modules. For details, see [Section 31.9, “MySQL Perl API”](https://dev.mysql.com/doc/refman/9.7/en/apis-perl.html "31.9 MySQL Perl API").

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [Perl](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_perl).

pessimistic

A methodology that sacrifices performance or concurrency in favor of safety. It is appropriate if a high proportion of requests or attempts might fail, or if the consequences of a failed request are severe. `InnoDB` uses what is known as a pessimistic **locking** strategy, to minimize the chance of **deadlocks**. At the application level, you might avoid deadlocks by using a pessimistic strategy of acquiring all locks needed by a transaction at the very beginning.

Many built-in database mechanisms use the opposite **optimistic** methodology.

See Also [deadlock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_deadlock), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [optimistic](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_optimistic).

phantom

A row that appears in the result set of a query, but not in the result set of an earlier query. For example, if a query is run twice within a **transaction**, and in the meantime, another transaction commits after inserting a new row or updating a row so that it matches the `WHERE` clause of the query.

This occurrence is known as a phantom read. It is harder to guard against than a **non-repeatable read**, because locking all the rows from the first query result set does not prevent the changes that cause the phantom to appear.

Among different **isolation levels**, phantom reads are prevented by the **serializable read** level, and allowed by the **repeatable read**, **consistent read**, and **read uncommitted** levels.

See Also [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [non-repeatable read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_non_repeatable_read), [READ UNCOMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_uncommitted), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [SERIALIZABLE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_serializable), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

PHP

A programming language originating with web applications. The code is typically embedded as blocks within the source of a web page, with the output substituted into the page as it is transmitted by the web server. This is in contrast to applications such as CGI scripts that print output in the form of an entire web page. The PHP style of coding is used for highly interactive and dynamic web pages. Modern PHP programs can also be run as command-line or GUI applications.

MySQL applications are written using one of the **PHP APIs**. Reusable modules can be written in **C** and called from PHP.

Another technology for writing server-side web pages with MySQL is **ASP.net**.

See Also [ASP.net](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_asp_net), [C](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_c), [PHP API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_php_api).

PHP API

Several **APIs** are available for writing MySQL applications in the **PHP** language: the original MySQL API (`Mysql`) the MySQL Improved Extension (`Mysqli`) the MySQL Native Driver (`Mysqlnd`) the MySQL functions (`PDO_MYSQL`), and Connector/PHP. For details, see [MySQL and PHP](https://dev.mysql.com/doc/apis-php/en/).

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [PHP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_php).

physical

A type of operation that involves hardware-related aspects such as disk blocks, memory pages, files, bits, disk reads, and so on. Typically, physical aspects are important during expert-level performance tuning and problem diagnosis. Contrast with **logical**.

See Also [logical](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_logical), [physical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_physical_backup).

physical backup

A **backup** that copies the actual data files. For example, the **`mysqlbackup`** command of the **MySQL Enterprise Backup** product produces a physical backup, because its output contains data files that can be used directly by the `mysqld` server, resulting in a faster **restore** operation. Contrast with **logical backup**.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [logical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_logical_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [restore](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_restore).

PITR

Acronym for **point-in-time recovery**.

See Also [point-in-time recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_point_in_time_recovery).

plan stability

A property of a **query execution plan**, where the optimizer makes the same choices each time for a given **query**, so that performance is consistent and predictable.

See Also [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [query execution plan](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query_execution_plan).

point-in-time recovery

The process of restoring a **backup** to recreate the state of the database at a specific date and time. Commonly abbreviated “PITR”. Because it is unlikely that the specified time corresponds exactly to the time of a backup, this technique usually requires a combination of a **physical backup** and a **logical backup**. For example, with the **MySQL Enterprise Backup** product, you restore the last backup that you took before the specified point in time, then replay changes from the **binary log** between the time of the backup and the PITR time.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [logical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_logical_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [physical backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_physical_backup).

port

The number of the TCP/IP socket the database server listens on, used to establish a **connection**. Often specified in conjunction with a **host**. Depending on your use of network encryption, there might be one port for unencrypted traffic and another port for **SSL** connections.

See Also [connection](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection), [host](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_host), [SSL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ssl).

prefix

See [index prefix](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index_prefix).

prepared backup

A set of backup files, produced by the **MySQL Enterprise Backup** product, after all the stages of applying **binary logs** and **incremental backups** are finished. The resulting files are ready to be **restored**. Prior to the apply steps, the files are known as a **raw backup**.

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [incremental backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_incremental_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [raw backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_raw_backup), [restore](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_restore).

prepared statement

An SQL statement that is analyzed in advance to determine an efficient execution plan. It can be executed multiple times, without the overhead for parsing and analysis each time. Different values can be substituted for literals in the `WHERE` clause each time, through the use of placeholders. This substitution technique improves security, protecting against some kinds of SQL injection attacks. You can also reduce the overhead for converting and copying return values to program variables.

Although you can use prepared statements directly through SQL syntax, the various **Connectors** have programming interfaces for manipulating prepared statements, and these APIs are more efficient than going through SQL.

See Also [client-side prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client_side_prepared_statement), [connector](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector), [server-side prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_server_side_prepared_statement).

primary key

A set of columns—and by implication, the index based on this set of columns—that can uniquely identify every row in a table. As such, it must be a unique index that does not contain any `NULL` values.

`InnoDB` requires that every table has such an index (also called the **clustered index** or **cluster index**), and organizes the table storage based on the column values of the primary key.

When choosing primary key values, consider using arbitrary values (a **synthetic key**) rather than relying on values derived from some other source (a **natural key**).

See Also [clustered index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clustered_index), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [natural key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_natural_key), [synthetic key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_synthetic_key).

process

An instance of an executing program. The operating system switches between multiple running processes, allowing for a certain degree of **concurrency**. On most operating systems, processes can contain multiple **threads** of execution that share resources. Context-switching between threads is faster than the equivalent switching between processes.

See Also [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [thread](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_thread).

pseudo-record

An artificial record in an index, used for **locking** key values or ranges that do not currently exist.

See Also [infimum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_infimum_record), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [supremum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_supremum_record).

Pthreads

The POSIX threads standard, which defines an API for threading and locking operations on Unix and Linux systems. On Unix and Linux systems, `InnoDB` uses this implementation for **mutexes**.

See Also [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex).

purge buffering

The technique of storing changes to secondary index pages, resulting from `DELETE` operations, in the **change buffer** rather than writing the changes immediately, so that the physical writes can be performed to minimize random I/O. (Because delete operations are a two-step process, this operation buffers the write that normally purges an index record that was previously marked for deletion.) It is one of the types of **change buffering**; the others are **insert buffering** and **delete buffering**.

See Also [change buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffer), [change buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffering), [delete buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_delete_buffering), [insert buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffer), [insert buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_insert_buffering).

purge lag

Another name for the `InnoDB` **history list**. Related to the [`innodb_max_purge_lag`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_max_purge_lag) configuration option.

See Also [history list](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_history_list).

purge thread

A **thread** within the `InnoDB` process that is dedicated to performing the periodic **purge** operation. In MySQL 5.6 and higher, multiple purge threads are enabled by the [`innodb_purge_threads`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_purge_threads) configuration option.

See Also [thread](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_thread).

Python

A programming language used in a broad range of fields, from Unix scripting to large-scale applications. Includes runtime typing, built-in high-level data types, object-oriented features, and an extensive standard library. Often used as a “glue” language between components written in other languages. The MySQL **Python API** is the open-source **MySQLdb** module.

See Also [MySQLdb](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqldb), [Python API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_python_api).

Python API

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [Python](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_python).

### Q

query

In **SQL**, an operation that reads information from one or more **tables**. Depending on the organization of data and the parameters of the query, the lookup might be optimized by consulting an **index**. If multiple tables are involved, the query is known as a **join**.

For historical reasons, sometimes discussions of internal processing for statements use “query” in a broader sense, including other types of MySQL statements such as **DDL** and **DML** statements.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [join](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_join), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

query execution plan

The set of decisions made by the optimizer about how to perform a **query** most efficiently, including which **index** or indexes to use, and the order in which to **join** tables. **Plan stability** involves the same choices being made consistently for a given query.

See Also [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [join](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_join), [plan stability](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_plan_stability), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query).

query log

See [general query log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_query_log).

quiesce

To reduce the amount of database activity, often in preparation for an operation such as an [`ALTER         TABLE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement"), a **backup**, or a **shutdown**. Might or might not involve doing as much **flushing** as possible, so that **InnoDB** does not continue doing background I/O.

In MySQL 5.6 and higher, the syntax `FLUSH TABLES ...         FOR EXPORT` writes some data to disk for `InnoDB` tables that make it simpler to back up those tables by copying the data files.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown).

### R

R-tree

A tree data structure used for spatial indexing of multi-dimensional data such as geographical coordinates, rectangles or polygons.

See Also [B-tree](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_b_tree).

RAID

Acronym for “Redundant Array of Inexpensive Drives”. Spreading I/O operations across multiple drives enables greater **concurrency** at the hardware level, and improves the efficiency of low-level write operations that otherwise would be performed in sequence.

See Also [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency).

random dive

A technique for quickly estimating the number of different values in a column (the column's **cardinality**). `InnoDB` samples pages at random from the index and uses that data to estimate the number of different values.

See Also [cardinality](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cardinality).

raw backup

The initial set of backup files produced by the **MySQL Enterprise Backup** product, before the changes reflected in the **binary log** and any **incremental backups** are applied. At this stage, the files are not ready to **restore**. After these changes are applied, the files are known as a **prepared backup**.

See Also [binary log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_binary_log), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [ibbackup\_logfile](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ibbackup_logfile), [incremental backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_incremental_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [prepared backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_backup), [restore](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_restore).

READ COMMITTED

An **isolation level** that uses a **locking** strategy that relaxes some of the protection between **transactions**, in the interest of performance. Transactions cannot see uncommitted data from other transactions, but they can see data that is committed by another transaction after the current transaction started. Thus, a transaction never sees any bad data, but the data that it does see may depend to some extent on the timing of other transactions.

When a transaction with this isolation level performs `UPDATE ... WHERE` or `DELETE ...         WHERE` operations, other transactions might have to wait. The transaction can perform `SELECT ... FOR         UPDATE`, and `LOCK IN SHARE MODE` operations without making other transactions wait.

`SELECT ... FOR SHARE` replaces `SELECT         ... LOCK IN SHARE MODE` in MySQL 8.0.1, but `LOCK IN SHARE MODE` remains available for backward compatibility.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [SERIALIZABLE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_serializable), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

read phenomena

Phenomena such as **dirty reads**, **non-repeatable reads**, and **phantom** reads which can occur when a transaction reads data that another transaction has modified.

See Also [dirty read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_read), [non-repeatable read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_non_repeatable_read), [phantom](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_phantom).

READ UNCOMMITTED

The **isolation level** that provides the least amount of protection between transactions. Queries employ a **locking** strategy that allows them to proceed in situations where they would normally wait for another transaction. However, this extra performance comes at the cost of less reliable results, including data that has been changed by other transactions and not committed yet (known as **dirty read**). Use this isolation level with great caution, and be aware that the results might not be consistent or reproducible, depending on what other transactions are doing at the same time. Typically, transactions with this isolation level only do queries, not insert, update, or delete operations.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [dirty read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

read view

An internal snapshot used by the **MVCC** mechanism of `InnoDB`. Certain **transactions**, depending on their **isolation level**, see the data values as they were at the time the transaction (or in some cases, the statement) started. Isolation levels that use a read view are **REPEATABLE READ**, **READ COMMITTED**, and **READ UNCOMMITTED**.

See Also [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [MVCC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mvcc), [READ COMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_committed), [READ UNCOMMITTED](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_uncommitted), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

read-ahead

A type of I/O request that prefetches a group of **pages** (an entire **extent**) into the **buffer pool** asynchronously, in case these pages are needed soon. The linear read-ahead technique prefetches all the pages of one extent based on access patterns for pages in the preceding extent. The random read-ahead technique prefetches all the pages for an extent once a certain number of pages from the same extent are in the buffer pool. Random read-ahead is not part of MySQL 5.5, but is re-introduced in MySQL 5.6 under the control of the [`innodb_random_read_ahead`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_random_read_ahead) configuration option.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [extent](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_extent), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).

read-only transaction

A type of **transaction** that can be optimized for `InnoDB` tables by eliminating some of the bookkeeping involved with creating a **read view** for each transaction. Can only perform **non-locking read** queries. It can be started explicitly with the syntax [`START TRANSACTION         READ ONLY`](https://dev.mysql.com/doc/refman/9.7/en/commit.html "15.3.1 START TRANSACTION, COMMIT, and ROLLBACK Statements"), or automatically under certain conditions. See [Section 10.5.3, “Optimizing InnoDB Read-Only Transactions”](https://dev.mysql.com/doc/refman/9.7/en/innodb-performance-ro-txn.html "10.5.3 Optimizing InnoDB Read-Only Transactions") for details.

See Also [non-locking read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_non_locking_read), [read view](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_read_view), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

record lock

A [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock "lock") on an index record. For example, `SELECT c1 FROM t WHERE c1 = 10 FOR         UPDATE;` prevents any other transaction from inserting, updating, or deleting rows where the value of `t.c1` is 10. Contrast with **gap lock** and **next-key lock**.

See Also [gap lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap_lock), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [next-key lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_next_key_lock).

redo

The data, in units of records, recorded in the **redo log** when [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml "DML") statements make changes to `InnoDB` tables. It is used during **crash recovery** to correct data written by incomplete **transactions**. The ever-increasing **LSN** value represents the cumulative amount of redo data that has passed through the redo log.

See Also [crash recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash_recovery), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [LSN](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lsn), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

redo log

A disk-based data structure used during **crash recovery**, to correct data written by incomplete **transactions**. During normal operation, it encodes requests to change `InnoDB` table data, which result from SQL statements or low-level API calls. Modifications that did not finish updating the **data files** before an unexpected **shutdown** are replayed automatically.

The redo log is physically represented on disk as a set of redo log files. Redo log data is encoded in terms of records affected; this data is collectively referred to as **redo**. The passage of data through the redo log is represented by an ever-increasing **LSN** value.

For more information, see [Section 17.6.5, “Redo Log”](https://dev.mysql.com/doc/refman/9.7/en/innodb-redo-log.html "17.6.5 Redo Log")

See Also [crash recovery](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_crash_recovery), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [ib\_logfile](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ib_logfile), [log buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_log_buffer), [LSN](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lsn), [redo](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo), [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

redo log archiving

An `InnoDB` feature that, when enabled, sequentially writes redo log records to an archive file to avoid potential loss of data than can occur when a backup utility fails to keep pace with redo log generation while a backup operation is in progress. For more information, see [Redo Log Archiving](https://dev.mysql.com/doc/refman/9.7/en/innodb-redo-log.html#innodb-redo-log-archiving "Redo Log Archiving").

See Also [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log).

redundant row format

The oldest `InnoDB` **row format**. Prior to MySQL 5.0.3, it was the only row format available in `InnoDB`. From MySQL 5.0.3 to MySQL 5.7.8, the default row format is **COMPACT**. As of MySQL 5.7.9, the default row format is defined by the [`innodb_default_row_format`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_default_row_format) configuration option, which has a default setting of **DYNAMIC**. You can still specify the **REDUNDANT** row format for compatibility with older `InnoDB` tables.

For more information, see [Section 17.10, “InnoDB Row Formats”](https://dev.mysql.com/doc/refman/9.7/en/innodb-row-format.html "17.10 InnoDB Row Formats").

See Also [row format](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_format).

referential integrity

The technique of maintaining data always in a consistent format, part of the **ACID** philosophy. In particular, data in different tables is kept consistent through the use of **foreign key constraints**, which can prevent changes from happening or automatically propagate those changes to all related tables. Related mechanisms include the **unique constraint**, which prevents duplicate values from being inserted by mistake, and the **NOT NULL constraint**, which prevents blank values from being inserted by mistake.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [FOREIGN KEY constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key_constraint), [NOT NULL constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_not_null_constraint), [unique constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_constraint).

relational

An important aspect of modern database systems. The database server encodes and enforces relationships such as one-to-one, one-to-many, many-to-one, and uniqueness. For example, a person might have zero, one, or many phone numbers in an address database; a single phone number might be associated with several family members. In a financial database, a person might be required to have exactly one taxpayer ID, and any taxpayer ID could only be associated with one person.

The database server can use these relationships to prevent bad data from being inserted, and to find efficient ways to look up information. For example, if a value is declared to be unique, the server can stop searching as soon as the first match is found, and it can reject attempts to insert a second copy of the same value.

At the database level, these relationships are expressed through SQL features such as **columns** within a table, unique and `NOT NULL` **constraints**, **foreign keys**, and different kinds of join operations. Complex relationships typically involve data split between more than one table. Often, the data is **normalized**, so that duplicate values in one-to-many relationships are stored only once.

In a mathematical context, the relations within a database are derived from set theory. For example, the `OR` and `AND` operators of a `WHERE` clause represent the notions of union and intersection.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [column](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_column), [constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_constraint), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [normalized](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_normalized).

relevance

In the **full-text search** feature, a number signifying the similarity between the search string and the data in the **FULLTEXT index**. For example, when you search for a single word, that word is typically more relevant for a row where it occurs several times in the text than a row where it appears only once.

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search), [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index).

REPEATABLE READ

The default **isolation level** for `InnoDB`. It prevents any rows that are queried from being changed by other **transactions**, thus blocking **non-repeatable reads** but not **phantom** reads. It uses a moderately strict **locking** strategy so that all queries within a transaction see data from the same snapshot, that is, the data as it was at the time the transaction started.

When a transaction with this isolation level performs `UPDATE ... WHERE`, `DELETE ...         WHERE`, `SELECT ... FOR UPDATE`, and `LOCK IN SHARE MODE` operations, other transactions might have to wait.

`SELECT ... FOR SHARE` replaces `SELECT         ... LOCK IN SHARE MODE` in MySQL 8.0.1, but `LOCK IN SHARE MODE` remains available for backward compatibility.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [phantom](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_phantom), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

repertoire

Repertoire is a term applied to character sets. A character set repertoire is the collection of characters in the set. See [Section 12.2.1, “Character Set Repertoire”](https://dev.mysql.com/doc/refman/9.7/en/charset-repertoire.html "12.2.1 Character Set Repertoire").

replica

A database **server** machine in a **replication** topology that receives changes from another server (the **source**) and applies those same changes. Thus it maintains the same contents as the source, although it might lag somewhat behind.

In MySQL, replicas are commonly used in disaster recovery, to take the place of a source that fails. They are also commonly used for testing software upgrades and new settings, to ensure that database configuration changes do not cause problems with performance or reliability.

Replicas typically have high workloads, because they process all the **DML** (write) operations relayed from the source, as well as user queries. To ensure that replicas can apply changes from the source fast enough, they frequently have fast I/O devices and sufficient CPU and memory to run multiple database instances on the same server. For example, the source might use hard drive storage while the replicas use **SSD** s.

See Also [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication), [server](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_server), [source](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_source), [SSD](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ssd).

replication

The practice of sending changes from a **source**, to one or more **replicas**, so that all databases have the same data. This technique has a wide range of uses, such as load-balancing for better scalability, disaster recovery, and testing software upgrades and configuration changes. The changes can be sent between the databases by methods called **row-based replication** and **statement-based replication**.

See Also [replica](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replica), [row-based replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_based_replication), [source](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_source), [statement-based replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_statement_based_replication).

restore

The process of putting a set of backup files from the **MySQL Enterprise Backup** product in place for use by MySQL. This operation can be performed to fix a corrupted database, to return to some earlier point in time, or (in a **replication** context) to set up a new **replica**. In the **MySQL Enterprise Backup** product, this operation is performed by the `copy-back` option of the `mysqlbackup` command.

See Also [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [mysqlbackup command](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqlbackup_command), [prepared backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_backup), [replica](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replica), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication).

rollback

A **SQL** statement that ends a **transaction**, undoing any changes made by the transaction. It is the opposite of **commit**, which makes permanent any changes made in the transaction.

By default, MySQL uses the **autocommit** setting, which automatically issues a commit following each SQL statement. You must change this setting before you can use the rollback technique.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [autocommit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_autocommit), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

rollback segment

The storage area containing the **undo logs**. Rollback segments have traditionally resided in the **system tablespace**. As of MySQL 5.6, rollback segments can reside in **undo tablespaces**. As of MySQL 5.7, rollback segments are also allocated to the *global temporary tablespace*.

See Also [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log), [undo tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_tablespace).

row

The logical data structure defined by a set of **columns**. A set of rows makes up a **table**. Within `InnoDB` **data files**, each **page** can contain one or more rows.

Although `InnoDB` uses the term **row format** for consistency with MySQL syntax, the row format is a property of each table and applies to all rows in that table.

See Also [column](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_column), [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page), [row format](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_format).

row format

The disk storage format for **rows** of an `InnoDB` **table**. As `InnoDB` gains new capabilities such as **compression**, new row formats are introduced to support the resulting improvements in storage efficiency and performance.

The row format of an `InnoDB` table is specified by the `ROW_FORMAT` option or by the [`innodb_default_row_format`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_default_row_format) configuration option (introduced in MySQL 5.7.9). Row formats include `REDUNDANT`, `COMPACT`, `COMPRESSED`, and `DYNAMIC`. To view the row format of an `InnoDB` table, issue the [`SHOW TABLE STATUS`](https://dev.mysql.com/doc/refman/9.7/en/show-table-status.html "15.7.7.39 SHOW TABLE STATUS Statement") statement or query `InnoDB` table metadata in the [`INFORMATION_SCHEMA`](https://dev.mysql.com/doc/refman/9.7/en/information-schema.html "Chapter 28 INFORMATION_SCHEMA Tables").

See Also [redundant row format](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redundant_row_format), [row](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row).

row lock

A **lock** that prevents a row from being accessed in an incompatible way by another **transaction**. Other rows in the same table can be freely written to by other transactions. This is the type of **locking** done by **DML** operations on **InnoDB** tables.

Contrast with **table locks** used by [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine"), or during **DDL** operations on `InnoDB` tables that cannot be done with **online DDL**; those locks block concurrent access to the table.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [online DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_online_ddl), [table lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_table_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

row-based replication

A form of **replication** where events are propagated from the **source** specifying how to change individual rows on the **replica**. It is safe to use for all settings of the [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) option.

See Also [auto-increment locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment_locking), [innodb\_autoinc\_lock\_mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_autoinc_lock_mode), [replica](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replica), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication), [source](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_source), [statement-based replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_statement_based_replication).

row-level locking

The **locking** mechanism used for **InnoDB** tables, relying on **row locks** rather than **table locks**. Multiple **transactions** can modify the same table concurrently. Only if two transactions try to modify the same row does one of the transactions wait for the other to complete (and release its row locks).

See Also [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [row lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_lock), [table lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_table_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

Ruby

A programming language that emphasizes dynamic typing and object-oriented programming. Some syntax is familiar to **Perl** developers.

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api), [Perl](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_perl), [Ruby API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ruby_api).

Ruby API

`mysql2`, based based on the **libmysqlclient** API library, is available for Ruby programmers developing MySQL applications. For more information, see [Section 31.11, “MySQL Ruby APIs”](https://dev.mysql.com/doc/refman/9.7/en/apis-ruby.html "31.11 MySQL Ruby APIs").

See Also [libmysql](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_libmysql), [Ruby](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ruby).

rw-lock

The low-level object that `InnoDB` uses to represent and enforce shared-access **locks** to internal in-memory data structures following certain rules. Contrast with **mutexes**, which `InnoDB` uses to represent and enforce exclusive access to internal in-memory data structures. Mutexes and rw-locks are known collectively as **latches**.

`rw-lock` types include `s-locks` (shared locks), `x-locks` (exclusive locks), and `sx-locks` (shared-exclusive locks).

- An `s-lock` provides read access to a common resource.
- An `x-lock` provides write access to a common resource while not permitting inconsistent reads by other threads.
- An `sx-lock` provides write access to a common resource while permitting inconsistent reads by other threads. `sx-locks` were introduced in MySQL 5.7 to optimize concurrency and improve scalability for read-write workloads.

The following matrix summarizes rw-lock type compatibility.

|  | *`S`* | *`SX`* | *`X`* |
| --- | --- | --- | --- |
| *`S`* | Compatible | Compatible | Conflict |
| *`SX`* | Compatible | Conflict | Conflict |
| *`X`* | Conflict | Conflict | Conflict |

See Also [latch](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_latch), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema).

### S

savepoint

Savepoints help to implement nested **transactions**. They can be used to provide scope to operations on tables that are part of a larger transaction. For example, scheduling a trip in a reservation system might involve booking several different flights; if a desired flight is unavailable, you might **roll back** the changes involved in booking that one leg, without rolling back the earlier flights that were successfully booked.

See Also [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

scalability

The ability to add more work and issue more simultaneous requests to a system, without a sudden drop in performance due to exceeding the limits of system capacity. Software architecture, hardware configuration, application coding, and type of workload all play a part in scalability. When the system reaches its maximum capacity, popular techniques for increasing scalability are **scale up** (increasing the capacity of existing hardware or software) and **scale out** (adding new servers and more instances of MySQL). Often paired with **availability** as critical aspects of a large-scale deployment.

See Also [availability](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_availability), [scale out](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scale_out), [scale up](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scale_up).

scale out

A technique for increasing **scalability** by adding new servers and more instances of MySQL. For example, setting up replication, NDB Cluster, connection pooling, or other features that spread work across a group of servers. Contrast with **scale up**.

See Also [scalability](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scalability), [scale up](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scale_up).

scale up

A technique for increasing **scalability** by increasing the capacity of existing hardware or software. For example, increasing the memory on a server and adjusting memory-related parameters such as [`innodb_buffer_pool_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_size) and [`innodb_buffer_pool_instances`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_instances). Contrast with **scale out**.

See Also [scalability](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scalability), [scale out](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_scale_out).

schema

Conceptually, a schema is a set of interrelated database objects, such as tables, table columns, data types of the columns, indexes, foreign keys, and so on. These objects are connected through SQL syntax, because the columns make up the tables, the foreign keys refer to tables and columns, and so on. Ideally, they are also connected logically, working together as part of a unified application or flexible framework. For example, the **INFORMATION\_SCHEMA** and **performance\_schema** databases use “schema” in their names to emphasize the close relationships between the tables and columns they contain.

In MySQL, physically, a **schema** is synonymous with a **database**. You can substitute the keyword `SCHEMA` instead of `DATABASE` in MySQL SQL syntax, for example using `CREATE SCHEMA` instead of `CREATE DATABASE`.

Some other database products draw a distinction. For example, in the Oracle Database product, a **schema** represents only a part of a database: the tables and other objects owned by a single user.

See Also [database](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_database), [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_information_schema), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema).

search index

In MySQL, **full-text search** queries use a special kind of index, the **FULLTEXT index**. In MySQL 5.6.4 and up, `InnoDB` and `MyISAM` tables both support `FULLTEXT` indexes; formerly, these indexes were only available for `MyISAM` tables.

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search), [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index).

secondary index

A type of `InnoDB` **index** that represents a subset of table columns. An `InnoDB` table can have zero, one, or many secondary indexes. (Contrast with the **clustered index**, which is required for each `InnoDB` table, and stores the data for all the table columns.)

A secondary index can be used to satisfy queries that only require values from the indexed columns. For more complex queries, it can be used to identify the relevant rows in the table, which are then retrieved through lookups using the clustered index.

Creating and dropping secondary indexes has traditionally involved significant overhead from copying all the data in the `InnoDB` table. The **fast index creation** feature makes both `CREATE         INDEX` and `DROP INDEX` statements much faster for `InnoDB` secondary indexes.

See Also [clustered index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clustered_index), [Fast Index Creation](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fast_index_creation), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index).

segment

A division within an `InnoDB` **tablespace**. If a tablespace is analogous to a directory, the segments are analogous to files within that directory. A segment can grow. New segments can be created.

For example, within a **file-per-table** tablespace, table data is in one segment and each associated index is in its own segment. The **system tablespace** contains many different segments, because it can hold many tables and their associated indexes. Prior to MySQL 8.0, the system tablespace also includes one or more **rollback segments** used for **undo logs**.

Segments grow and shrink as data is inserted and deleted. When a segment needs more room, it is extended by one **extent** (1 megabyte) at a time. Similarly, a segment releases one extent's worth of space when all the data in that extent is no longer needed.

See Also [extent](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_extent), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [rollback segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback_segment), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

selectivity

A property of data distribution, the number of distinct values in a column (its **cardinality**) divided by the number of records in the table. High selectivity means that the column values are relatively unique, and can retrieved efficiently through an index. If you (or the query optimizer) can predict that a test in a `WHERE` clause only matches a small number (or proportion) of rows in a table, the overall **query** tends to be efficient if it evaluates that test first, using an index.

See Also [cardinality](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cardinality), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query).

SERIALIZABLE

The **isolation level** that uses the most conservative locking strategy, to prevent any other **transactions** from inserting or changing data that was read by this transaction, until it is finished. This way, the same query can be run over and over within a transaction, and be certain to retrieve the same set of results each time. Any attempt to change data that was committed by another transaction since the start of the current transaction, cause the current transaction to wait.

This is the default isolation level specified by the SQL standard. In practice, this degree of strictness is rarely needed, so the default isolation level for `InnoDB` is the next most strict, **REPEATABLE READ**.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [REPEATABLE READ](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_repeatable_read), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

server

A type of program that runs continuously, waiting to receive and act upon requests from another program (the **client**). Because often an entire computer is dedicated to running one or more server programs (such as a database server, a web server, an application server, or some combination of these), the term **server** can also refer to the computer that runs the server software.

See Also [client](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client), [mysqld](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysqld).

server-side prepared statement

A **prepared statement** managed by the MySQL server. Historically, issues with server-side prepared statements led **Connector/J** and **Connector/PHP** developers to sometimes use **client-side prepared statements** instead. With modern MySQL server versions, server-side prepared statements are recommended for performance, scalability, and memory efficiency.

See Also [client-side prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_client_side_prepared_statement), [Connector/PHP](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_php), [prepared statement](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_prepared_statement).

servlet

shared lock

A kind of **lock** that allows other **transactions** to read the locked object, and to also acquire other shared locks on it, but not to write to it. The opposite of **exclusive lock**.

See Also [exclusive lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_exclusive_lock), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

shared tablespace

Another way of referring to the **system tablespace** or a **general tablespace**. General tablespaces were introduced in MySQL 5.7. More than one table can reside in a shared tablespace. Only a single table can reside in a *file-per-table* tablespace.

See Also [general tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_tablespace).

sharp checkpoint

The process of **flushing** to disk all **dirty** buffer pool pages whose redo entries are contained in certain portion of the **redo log**. Occurs before `InnoDB` reuses a portion of a log file; the log files are used in a circular fashion. Typically occurs with write-intensive **workloads**.

See Also [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [redo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_redo_log), [workload](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_workload).

shutdown

The process of stopping the MySQL server. By default, this process cleans up operations for **InnoDB** tables, so `InnoDB` can be **slow** to shut down, but fast to start up later. If you skip the cleanup operations, it is **fast** to shut down but the cleanup must be performed during the next restart.

The shutdown mode for `InnoDB` is controlled by the [`innodb_fast_shutdown`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_fast_shutdown) option.

See Also [fast shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fast_shutdown), [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [slow shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_slow_shutdown), [startup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_startup).

slave

See [replica](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replica).

slow query log

A type of **log** used for performance tuning of SQL statements processed by the MySQL server. The log information is stored in a file. You must enable this feature to use it. You control which categories of “slow” SQL statements are logged. For more information, see [Section 7.4.5, “The Slow Query Log”](https://dev.mysql.com/doc/refman/9.7/en/slow-query-log.html "7.4.5 The Slow Query Log").

See Also [general query log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_query_log), [log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_log).

slow shutdown

A type of **shutdown** that does additional `InnoDB` flushing operations before completing. Also known as a **clean shutdown**. Specified by the configuration parameter [`innodb_fast_shutdown=0`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_fast_shutdown) or the command `SET GLOBAL innodb_fast_shutdown=0;`. Although the shutdown itself can take longer, that time should be saved on the subsequent startup.

See Also [clean shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_clean_shutdown), [fast shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fast_shutdown), [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown).

snapshot

A representation of data at a particular time, which remains the same even as changes are **committed** by other **transactions**. Used by certain **isolation levels** to allow **consistent reads**.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

sort buffer

The buffer used for sorting data during creation of an `InnoDB` index. Sort buffer size is configured using the [`innodb_sort_buffer_size`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_sort_buffer_size) configuration option.

source

A database server machine in a **replication** scenario that processes the initial insert, update, and delete requests for data. These changes are propagated to, and repeated on, other servers known as **replicas**.

See Also [replica](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replica), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication).

space ID

An identifier used to uniquely identify an `InnoDB` **tablespace** within a MySQL instance. The space ID for the **system tablespace** is always zero; this same ID applies to all tables within the system tablespace or within a general tablespace. Each **file-per-table** tablespace and **general tablespace** has its own space ID.

Prior to MySQL 5.6, this hardcoded value presented difficulties in moving `InnoDB` tablespace files between MySQL instances. Starting in MySQL 5.6, you can copy tablespace files between instances by using the **transportable tablespace** feature involving the statements `FLUSH TABLES ... FOR         EXPORT`, `ALTER TABLE ... DISCARD         TABLESPACE`, and `ALTER TABLE ... IMPORT         TABLESPACE`. The information needed to adjust the space ID is conveyed in the **.cfg file** which you copy along with the tablespace. See [Section 17.6.1.3, “Importing InnoDB Tables”](https://dev.mysql.com/doc/refman/9.7/en/innodb-table-import.html "17.6.1.3 Importing InnoDB Tables") for details.

See Also [.cfg file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cfg_file), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [general tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_tablespace), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace), [transportable tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transportable_tablespace).

sparse file

A type of file that uses file system space more efficiently by writing metadata representing empty blocks to disk instead of writing the actual empty space. The `InnoDB` **transparent page compression** feature relies on sparse file support. For more information, see [Section 17.9.2, “InnoDB Page Compression”](https://dev.mysql.com/doc/refman/9.7/en/innodb-page-compression.html "17.9.2 InnoDB Page Compression").

See Also [hole punching](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hole_punching), [transparent page compression](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transparent_page_compression).

spin

A type of **wait** operation that continuously tests whether a resource becomes available. This technique is used for resources that are typically held only for brief periods, where it is more efficient to wait in a “busy loop” than to put the thread to sleep and perform a context switch. If the resource does not become available within a short time, the spin loop ceases and another wait technique is used.

See Also [latch](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_latch), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex), [wait](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_wait).

Spring

A Java-based application framework designed for assisting in application design by providing a way to configure components.

See Also [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee).

SQL

The Structured Query Language that is standard for performing database operations. Often divided into the categories **DDL**, **DML**, and **queries**. MySQL includes some additional statement categories such as **replication**. See [Chapter 11, *Language Structure*](https://dev.mysql.com/doc/refman/9.7/en/language-structure.html "Chapter 11 Language Structure") for the building blocks of SQL syntax, [Chapter 13, *Data Types*](https://dev.mysql.com/doc/refman/9.7/en/data-types.html "Chapter 13 Data Types") for the data types to use for MySQL table columns, [Chapter 15, *SQL Statements*](https://dev.mysql.com/doc/refman/9.7/en/sql-statements.html "Chapter 15 SQL Statements") for details about SQL statements and their associated categories, and [Chapter 14, *Functions and Operators*](https://dev.mysql.com/doc/refman/9.7/en/functions.html "Chapter 14 Functions and Operators") for standard and MySQL-specific functions to use in queries.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication).

SQLState

An error code defined by the **JDBC** standard, for exception handling by applications using **Connector/J**.

See Also [JDBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_jdbc).

SSD

Acronym for “solid-state drive”. A type of storage device with different performance characteristics than a traditional hard disk drive (**HDD**): smaller storage capacity, faster for random reads, no moving parts, and with a number of considerations affecting write performance. Its performance characteristics can influence the throughput of a **disk-bound** workload.

See Also [disk-bound](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_bound), [HDD](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hdd).

SSL

Acronym for “secure sockets layer”. Provides the encryption layer for network communication between an application and a MySQL database server.

See Also [keystore](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_keystore), [truststore](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_truststore).

startup

The process of starting the MySQL server. Typically done by one of the programs listed in [Section 6.3, “Server and Server-Startup Programs”](https://dev.mysql.com/doc/refman/9.7/en/programs-server.html "6.3 Server and Server-Startup Programs"). The opposite of **shutdown**.

See Also [shutdown](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_shutdown).

statement interceptor

A type of **interceptor** for tracing, debugging, or augmenting SQL statements issued by a database application. Sometimes also known as a **command interceptor**.

In **Java** applications using **Connector/J**, setting up this type of interceptor involves implementing the `com.mysql.jdbc.StatementInterceptorV2` interface, and adding a `statementInterceptors` property to the **connection string**.

In **Visual Studio** applications using **Connector/NET**, setting up this type of interceptor involves defining a class that inherits from the `BaseCommandInterceptor` class and specifying that class name as part of the connection string.

See Also [command interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_command_interceptor), [connection string](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connection_string), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net), [interceptor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_interceptor), [Java](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_java), [Visual Studio](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_visual_studio).

statement-based replication

A form of **replication** where SQL statements are sent from the **source** and replayed on the **replica**. It requires some care with the setting for the [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) option, to avoid potential timing problems with **auto-increment locking**.

See Also [auto-increment locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment_locking), [innodb\_autoinc\_lock\_mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_autoinc_lock_mode), [replica](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replica), [replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_replication), [row-based replication](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_based_replication), [source](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_source).

statistics

Estimated values relating to each `InnoDB` **table** and **index**, used to construct an efficient **query execution plan**. The main values are the **cardinality** (number of distinct values) and the total number of table rows or index entries. The statistics for the table represent the data in its **primary key** index. The statistics for a **secondary index** represent the rows covered by that index.

The values are estimated rather than counted precisely because at any moment, different **transactions** can be inserting and deleting rows from the same table. To keep the values from being recalculated frequently, you can enable **persistent statistics**, where the values are stored in `InnoDB` system tables, and refreshed only when you issue an [`ANALYZE TABLE`](https://dev.mysql.com/doc/refman/9.7/en/analyze-table.html "15.7.3.1 ANALYZE TABLE Statement") statement.

You can control how **NULL** values are treated when calculating statistics through the [`innodb_stats_method`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_stats_method) configuration option.

Other types of statistics are available for database objects and database activity through the **INFORMATION\_SCHEMA** and **PERFORMANCE\_SCHEMA** tables.

See Also [cardinality](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cardinality), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_information_schema), [NULL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_null), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [query execution plan](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query_execution_plan), [secondary index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_secondary_index), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

stemming

The ability to search for different variations of a word based on a common root word, such as singular and plural, or past, present, and future verb tense. This feature is currently supported in `MyISAM` **full-text search** feature but not in **FULLTEXT indexes** for `InnoDB` tables.

See Also [full-text search](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_text_search), [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index).

stopword

In a **FULLTEXT index**, a word that is considered common or trivial enough that it is omitted from the **search index** and ignored in search queries. Different configuration settings control stopword processing for `InnoDB` and `MyISAM` tables. See [Section 14.9.4, “Full-Text Stopwords”](https://dev.mysql.com/doc/refman/9.7/en/fulltext-stopwords.html "14.9.4 Full-Text Stopwords") for details.

See Also [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index), [search index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_search_index).

storage engine

A component of the MySQL database that performs the low-level work of storing, updating, and querying data. In MySQL 5.5 and higher, **InnoDB** is the default storage engine for new tables, superceding `MyISAM`. Different storage engines are designed with different tradeoffs between factors such as memory usage versus disk usage, read speed versus write speed, and speed versus robustness. Each storage engine manages specific tables, so we refer to [`InnoDB`](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") tables, [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") tables, and so on.

The **MySQL Enterprise Backup** product is optimized for backing up `InnoDB` tables. It can also back up tables handled by `MyISAM` and other storage engines.

See Also [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [MySQL Enterprise Backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mysql_enterprise_backup), [table type](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_table_type).

stored object

A stored program or view.

stored program

A stored routine (procedure or function), trigger, or Event Scheduler event.

stored routine

A stored procedure or function.

strict mode

The general name for the setting controlled by the [`innodb_strict_mode`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_strict_mode) option. Turning on this setting causes certain conditions that are normally treated as warnings, to be considered errors. For example, certain invalid combinations of options related to **file format** and **row format**, that normally produce a warning and continue with default values, now cause the `CREATE TABLE` operation to fail. [`innodb_strict_mode`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_strict_mode) is enabled by default in MySQL 5.7.

MySQL also has something called strict mode. See [Section 7.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/9.7/en/sql-mode.html "7.1.11 Server SQL Modes").

See Also [innodb\_strict\_mode](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_strict_mode), [row format](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_format).

sublist

Within the list structure that represents the **buffer pool**, pages that are relatively old and relatively new are represented by different portions of the **list**. A set of parameters control the size of these portions and the dividing point between the new and old pages.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [eviction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_eviction), [list](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_list), [LRU](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lru).

supremum record

A **pseudo-record** in an index, representing the **gap** above the largest value in that index. If a transaction has a statement such as `SELECT ... FROM ... WHERE col > 10 FOR         UPDATE;`, and the largest value in the column is 20, it is a lock on the supremum record that prevents other transactions from inserting even larger values such as 50, 100, and so on.

See Also [gap](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_gap), [infimum record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_infimum_record), [pseudo-record](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_pseudo_record).

surrogate key

Synonym name for **synthetic key**.

See Also [synthetic key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_synthetic_key).

synthetic key

An indexed column, typically a **primary key**, where the values are assigned arbitrarily. Often done using an **auto-increment** column. By treating the value as completely arbitrary, you can avoid overly restrictive rules and faulty application assumptions. For example, a numeric sequence representing employee numbers might have a gap if an employee was approved for hiring but never actually joined. Or employee number 100 might have a later hiring date than employee number 500, if they left the company and later rejoined. Numeric values also produce shorter values of predictable length. For example, storing numeric codes meaning “Road”, “Boulevard”, “Expressway”, and so on is more space-efficient than repeating those strings over and over.

Also known as a **surrogate key**. Contrast with **natural key**.

See Also [auto-increment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_auto_increment), [natural key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_natural_key), [primary key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_primary_key), [surrogate key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_surrogate_key).

### T

table lock

A lock that prevents any other **transaction** from accessing a table. `InnoDB` makes considerable effort to make such locks unnecessary, by using techniques such as **online DDL**, **row locks** and **consistent reads** for processing **DML** statements and **queries**. You can create such a lock through SQL using the `LOCK TABLE` statement; one of the steps in migrating from other database systems or MySQL storage engines is to remove such statements wherever practical.

See Also [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [DML](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dml), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [locking](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_locking), [online DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_online_ddl), [query](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_query), [row lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row_lock), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

table scan

See [full table scan](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_full_table_scan).

table statistics

See [statistics](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_statistics).

table type

Obsolete synonym for **storage engine**. We refer to [`InnoDB`](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") tables, [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine") tables, and so on.

See Also [InnoDB](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb), [storage engine](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_storage_engine).

tablespace

A data file that can hold data for one or more `InnoDB` **tables** and associated **indexes**.

The **system tablespace** contains the `InnoDB` **data dictionary**, and prior to MySQL 5.6 holds all other `InnoDB` tables by default.

The [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) option, enabled by default in MySQL 5.6 and higher, allows tables to be created in their own tablespaces. File-per-table tablespaces support features such as efficient storage of **off-page columns**, table compression, and transportable tablespaces. See [Section 17.6.3.2, “File-Per-Table Tablespaces”](https://dev.mysql.com/doc/refman/9.7/en/innodb-file-per-table-tablespaces.html "17.6.3.2 File-Per-Table Tablespaces") for details.

`InnoDB` introduced general tablespaces in MySQL 5.7.6. General tablespaces are shared tablespaces created using [`CREATE TABLESPACE`](https://dev.mysql.com/doc/refman/9.7/en/create-tablespace.html "15.1.25 CREATE TABLESPACE Statement") syntax. They can be created outside of the MySQL data directory, are capable of holding multiple tables, and support tables of all row formats.

MySQL NDB Cluster also groups its tables into tablespaces. See [Section 25.6.11.1, “NDB Cluster Disk Data Objects”](https://dev.mysql.com/doc/refman/9.7/en/mysql-cluster-disk-data-objects.html "25.6.11.1 NDB Cluster Disk Data Objects") for details.

See Also [data files](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_data_files), [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [general tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_general_tablespace), [index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_index), [innodb\_file\_per\_table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_file_per_table).

Tcl

A programming language originating in the Unix scripting world. Sometimes extended by code written in **C**, **C++**, or **Java**. For the open-source Tcl **API** for MySQL, see [Section 31.12, “MySQL Tcl API”](https://dev.mysql.com/doc/refman/9.7/en/apis-tcl.html "31.12 MySQL Tcl API").

See Also [API](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_api).

temporary table

A **table** whose data does not need to be truly permanent. For example, temporary tables might be used as storage areas for intermediate results in complicated calculations or transformations; this intermediate data would not need to be recovered after a crash. Database products can take various shortcuts to improve the performance of operations on temporary tables, by being less scrupulous about writing data to disk and other measures to protect the data across restarts.

Sometimes, the data itself is removed automatically at a set time, such as when the transaction ends or when the session ends. With some database products, the table itself is removed automatically too.

text collection

The set of columns included in a **FULLTEXT index**.

See Also [FULLTEXT index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_fulltext_index).

thread

A unit of processing that is typically more lightweight than a **process**, allowing for greater **concurrency**.

See Also [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [master thread](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_master_thread), [process](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_process), [Pthreads](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_pthreads).

Tomcat

An open source **J2EE** application server, implementing the Java Servlet and JavaServer Pages programming technologies. Consists of a web server and Java servlet container. With MySQL, typically used in conjunction with **Connector/J**.

See Also [J2EE](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_j2ee).

torn page

An error condition that can occur due to a combination of I/O device configuration and hardware failure. If data is written out in chunks smaller than the `InnoDB` **page size** (by default, 16KB), a hardware failure while writing could result in only part of a page being stored to disk. The `InnoDB` **doublewrite buffer** guards against this possibility.

See Also [doublewrite buffer](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_doublewrite_buffer).

TPS

Acronym for “ **transactions** per second”, a unit of measurement sometimes used in benchmarks. Its value depends on the **workload** represented by a particular benchmark test, combined with factors that you control such as the hardware capacity and database configuration.

See Also [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction), [workload](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_workload).

transaction

Transactions are atomic units of work that can be **committed** or **rolled back**. When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is committed, or all the changes are undone when the transaction is rolled back.

Database transactions, as implemented by `InnoDB`, have properties that are collectively known by the acronym **ACID**, for atomicity, consistency, isolation, and durability.

See Also [ACID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_acid), [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [isolation level](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_isolation_level), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback).

transaction ID

An internal field associated with each **row**. This field is physically changed by [`INSERT`](https://dev.mysql.com/doc/refman/9.7/en/insert.html "15.2.7 INSERT Statement"), [`UPDATE`](https://dev.mysql.com/doc/refman/9.7/en/update.html "15.2.17 UPDATE Statement"), and [`DELETE`](https://dev.mysql.com/doc/refman/9.7/en/delete.html "15.2.2 DELETE Statement") operations to record which **transaction** has locked the row.

See Also [implicit row lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_implicit_row_lock), [row](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_row), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

transparent page compression

A feature added in MySQL 5.7.8 that permits page-level compression for `InnoDB` tables that reside in **file-per-table** tablespaces. Page compression is enabled by specifying the `COMPRESSION` attribute with [`CREATE TABLE`](https://dev.mysql.com/doc/refman/9.7/en/create-table.html "15.1.24 CREATE TABLE Statement") or [`ALTER TABLE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement"). For more information, see [Section 17.9.2, “InnoDB Page Compression”](https://dev.mysql.com/doc/refman/9.7/en/innodb-page-compression.html "17.9.2 InnoDB Page Compression").

See Also [file-per-table](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_file_per_table), [hole punching](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hole_punching), [sparse file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sparse_file).

transportable tablespace

A feature that allows a **tablespace** to be moved from one instance to another. Traditionally, this has not been possible for `InnoDB` tablespaces because all table data was part of the **system tablespace**. In MySQL 5.6 and higher, the [`FLUSH         TABLES ... FOR EXPORT`](https://dev.mysql.com/doc/refman/9.7/en/flush.html#flush-tables-for-export-with-list) syntax prepares an `InnoDB` table for copying to another server; running [`ALTER TABLE         ... DISCARD TABLESPACE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement") and [`ALTER TABLE ...         IMPORT TABLESPACE`](https://dev.mysql.com/doc/refman/9.7/en/alter-table.html "15.1.11 ALTER TABLE Statement") on the other server brings the copied data file into the other instance. A separate **.cfg file**, copied along with the **.ibd file**, is used to update the table metadata (for example the **space ID**) as the tablespace is imported. See [Section 17.6.1.3, “Importing InnoDB Tables”](https://dev.mysql.com/doc/refman/9.7/en/innodb-table-import.html "17.6.1.3 Importing InnoDB Tables") for usage information.

See Also [.cfg file](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cfg_file), [space ID](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_space_id), [tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_tablespace).

troubleshooting

The process of determining the source of a problem. Some of the resources for troubleshooting MySQL problems include:

truncate

A **DDL** operation that removes the entire contents of a table, while leaving the table and related indexes intact. Contrast with **drop**. Although conceptually it has the same result as a `DELETE` statement with no `WHERE` clause, it operates differently behind the scenes: `InnoDB` creates a new empty table, drops the old table, then renames the new table to take the place of the old one. Because this is a DDL operation, it cannot be **rolled back**.

If the table being truncated contains **foreign keys** that reference another table, the truncation operation uses a slower method of operation, deleting one row at a time so that corresponding rows in the referenced table can be deleted as needed by any `ON DELETE CASCADE` clause. (MySQL 5.5 and higher do not allow this slower form of truncate, and return an error instead if foreign keys are involved. In this case, use a `DELETE` statement instead.

See Also [DDL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ddl), [drop](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_drop), [foreign key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_foreign_key), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback).

truststore

See Also [SSL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ssl).

tuple

A technical term designating an ordered set of elements. It is an abstract notion, used in formal discussions of database theory. In the database field, tuples are usually represented by the columns of a table row. They could also be represented by the result sets of queries, for example, queries that retrieved only some columns of a table, or columns from joined tables.

See Also [cursor](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cursor).

two-phase commit

An operation that is part of a distributed **transaction**, under the **XA** specification. (Sometimes abbreviated as 2PC.) When multiple databases participate in the transaction, either all databases **commit** the changes, or all databases **roll back** the changes.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

### U

undo

Data that is maintained throughout the life of a **transaction**, recording all changes so that they can be undone in case of a **rollback** operation. It is stored in **undo logs** either within the **system tablespace** (in MySQL 5.7 or earlier) or in separate **undo tablespaces**. As of MySQL 8.0, undo logs reside in undo tablespaces by default.

See Also [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [rollback segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback_segment), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log), [undo tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_tablespace).

undo buffer

See [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

undo log

A storage area that holds copies of data modified by active **transactions**. If another transaction needs to see the original data (as part of a **consistent read** operation), the unmodified data is retrieved from this storage area.

In MySQL 5.6 and MySQL 5.7, you can use the [`innodb_undo_tablespaces`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_undo_tablespaces) variable have undo logs reside in **undo tablespaces**, which can be placed on another storage device such as an **SSD**. In MySQL 8.0, undo logs reside in two default undo tablespaces that are created when MySQL is initialized, and additional undo tablespaces can be created using [`CREATE UNDO         TABLESPACE`](https://dev.mysql.com/doc/refman/9.7/en/create-tablespace.html "15.1.25 CREATE TABLESPACE Statement") syntax.

The undo log is split into separate portions, the **insert undo buffer** and the **update undo buffer**.

See Also [consistent read](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_consistent_read), [rollback segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback_segment), [SSD](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ssd), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction), [undo tablespace](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_tablespace).

undo log segment

A collection of **undo logs**. Undo log segments exists within **rollback segments**. An undo log segment might contain undo logs from multiple transactions. An undo log segment can only be used by one transaction at a time but can be reused after it is released at transaction **commit** or **rollback**. May also be referred to as an “undo segment”.

See Also [commit](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_commit), [rollback](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback), [rollback segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback_segment), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log).

undo tablespace

An undo tablespace contains **undo logs**. Undo logs exist within **undo log segments**, which are contained within **rollback segments**. Rollback segments have traditionally resided in the system tablespace. As of MySQL 5.6, rollback segments can reside in undo tablespaces. In MySQL 5.6 and MySQL 5.7, the number of undo tablespaces is controlled by the [`innodb_undo_tablespaces`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_undo_tablespaces) configuration option. In MySQL 8.0, two default undo tablespaces are created when the MySQL instance is initialized, and additional undo tablespaces can be created using [`CREATE UNDO         TABLESPACE`](https://dev.mysql.com/doc/refman/9.7/en/create-tablespace.html "15.1.25 CREATE TABLESPACE Statement") syntax.

For more information, see [Section 17.6.3.4, “Undo Tablespaces”](https://dev.mysql.com/doc/refman/9.7/en/innodb-undo-tablespaces.html "17.6.3.4 Undo Tablespaces").

See Also [rollback segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_rollback_segment), [undo log](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log), [undo log segment](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_undo_log_segment).

Unicode

A system for supporting national characters, character sets, code pages, and other internationalization aspects in a flexible and standardized way.

Unicode support is an important aspect of the **ODBC** standard. **Connector/ODBC** 5.1 is a Unicode driver, as opposed to Connector/ODBC 3.51, which is an **ANSI** driver.

See Also [ANSI](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_ansi), [Connector/ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_odbc), [ODBC](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_odbc).

unique constraint

A kind of **constraint** that asserts that a column cannot contain any duplicate values. In terms of **relational** algebra, it is used to specify 1-to-1 relationships. For efficiency in checking whether a value can be inserted (that is, the value does not already exist in the column), a unique constraint is supported by an underlying **unique index**.

See Also [constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_constraint), [relational](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_relational), [unique index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_index).

unique index

An index on a column or set of columns that have a **unique constraint**. Because the index is known not to contain any duplicate values, certain kinds of lookups and count operations are more efficient than in the normal kind of index. Most of the lookups against this type of index are simply to determine if a certain value exists or not. The number of values in the index is the same as the number of rows in the table, or at least the number of rows with non-null values for the associated columns.

**Change buffering** optimization does not apply to unique indexes. As a workaround, you can temporarily set `unique_checks=0` while doing a bulk data load into an `InnoDB` table.

See Also [cardinality](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cardinality), [change buffering](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_change_buffering), [unique constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_constraint), [unique key](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_key).

unique key

The set of columns (one or more) comprising a **unique index**. When you can define a `WHERE` condition that matches exactly one row, and the query can use an associated unique index, the lookup and error handling can be performed very efficiently.

See Also [cardinality](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cardinality), [unique constraint](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_constraint), [unique index](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_unique_index).

### V

variable-length type

A data type of variable length. [`VARCHAR`](https://dev.mysql.com/doc/refman/9.7/en/char.html "13.3.2 The CHAR and VARCHAR Types"), [`VARBINARY`](https://dev.mysql.com/doc/refman/9.7/en/binary-varbinary.html "13.3.3 The BINARY and VARBINARY Types"), and [`BLOB`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types") and [`TEXT`](https://dev.mysql.com/doc/refman/9.7/en/blob.html "13.3.4 The BLOB and TEXT Types") types are variable-length types.

`InnoDB` treats fixed-length fields greater than or equal to 768 bytes in length as variable-length fields, which can be stored **off-page**. For example, a `CHAR(255)` column can exceed 768 bytes if the maximum byte length of the character set is greater than 3, as it is with `utf8mb4`.

See Also [off-page column](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_off_page_column), [overflow page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_overflow_page).

victim

The **transaction** that is automatically chosen to be **rolled back** when a **deadlock** is detected. `InnoDB` rolls back the transaction that has updated the fewest rows.

**Deadlock detection** can be disabled using the [`innodb_deadlock_detect`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_deadlock_detect) configuration option.

See Also [deadlock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_deadlock), [innodb\_lock\_wait\_timeout](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_innodb_lock_wait_timeout), [transaction](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_transaction).

view

A stored query that when invoked produces a result set. A view acts as a virtual table.

Visual Studio

For supported versions of Visual Studio, see the following references:

- Connector/NET: [Connector/NET Versions](https://dev.mysql.com/doc/connector-net/en/connector-net-versions.html)
- Connector/C++ 8.0: [Platform Support and Prerequisites](https://dev.mysql.com/doc/connector-cpp/9.7/en/connector-cpp-introduction.html#connector-cpp-prerequisites)

See Also [Connector/C++](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_c__), [Connector/NET](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_connector_net).

### W

wait

When an operation, such as acquiring a **lock**, **mutex**, or **latch**, cannot be completed immediately, `InnoDB` pauses and tries again. The mechanism for pausing is elaborate enough that this operation has its own name, the **wait**. Individual threads are paused using a combination of internal `InnoDB` scheduling, operating system `wait()` calls, and short-duration **spin** loops.

On systems with heavy load and many transactions, you might use the output from the `SHOW INNODB STATUS` command or **Performance Schema** to determine whether threads are spending too much time waiting, and if so, how you can improve **concurrency**.

See Also [concurrency](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_concurrency), [latch](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_latch), [lock](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lock), [mutex](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_mutex), [Performance Schema](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_performance_schema), [spin](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_spin).

warm backup

A **backup** taken while the database is running, but that restricts some database operations during the backup process. For example, tables might become read-only. For busy applications and websites, you might prefer a **hot backup**.

See Also [backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_backup), [cold backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cold_backup), [hot backup](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_hot_backup).

warm up

To run a system under a typical **workload** for some time after startup, so that the **buffer pool** and other memory regions are filled as they would be under normal conditions. This process happens naturally over time when a MySQL server is restarted or subjected to a new workload.

Typically, you run a workload for some time to warm up the buffer pool before running performance tests, to ensure consistent results across multiple runs; otherwise, performance might be artificially low during the first run.

In MySQL 5.6, you can speed up the warmup process by enabling the [`innodb_buffer_pool_dump_at_shutdown`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_dump_at_shutdown) and [`innodb_buffer_pool_load_at_startup`](https://dev.mysql.com/doc/refman/9.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_load_at_startup) configuration options, to bring the contents of the buffer pool back into memory after a restart. These options are enabled by default in MySQL 5.7. See [Section 17.8.3.6, “Saving and Restoring the Buffer Pool State”](https://dev.mysql.com/doc/refman/9.7/en/innodb-preload-buffer-pool.html "17.8.3.6 Saving and Restoring the Buffer Pool State").

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [workload](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_workload).

workload

The combination and volume of **SQL** and other database operations, performed by a database application during typical or peak usage. You can subject the database to a particular workload during performance testing to identify **bottlenecks**, or during capacity planning.

See Also [bottleneck](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_bottleneck), [CPU-bound](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_cpu_bound), [disk-bound](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_disk_bound), [SQL](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_sql).

write combining

An optimization technique that reduces write operations when **dirty pages** are **flushed** from the `InnoDB` **buffer pool**. If a row in a page is updated multiple times, or multiple rows on the same page are updated, all of those changes are stored to the data files in a single write operation rather than one write for each change.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [dirty page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_dirty_page), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush).

### Y

young

A characteristic of a **page** in the `InnoDB` **buffer pool** meaning that it has been accessed recently, and so is moved within the buffer pool data structure, so that it is not **flushed** too soon by the **LRU** algorithm. This term is used in some **INFORMATION\_SCHEMA** column names of tables related to the buffer pool.

See Also [buffer pool](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_buffer_pool), [flush](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_flush), [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_information_schema), [LRU](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_lru), [page](https://dev.mysql.com/doc/refman/9.7/en/glossary.html#glos_page).