---
title: "MySQL :: MySQL 8.4 Reference Manual :: 17.6.1.6 AUTO_INCREMENT Handling in InnoDB"
source: "https://dev.mysql.com/doc/refman/8.4/en/innodb-auto-increment-handling.html"
author:
published:
created: 2026-06-16
description:
tags:
  - "clippings"
---
[Documentation Home](https://dev.mysql.com/doc/)

---

Related Documentation Download this Manual  
  

#### 17.6.1.6 AUTO\_INCREMENT Handling in InnoDB

`InnoDB` provides a configurable locking mechanism that can significantly improve scalability and performance of SQL statements that add rows to tables with `AUTO_INCREMENT` columns. To use the `AUTO_INCREMENT` mechanism with an `InnoDB` table, an `AUTO_INCREMENT` column must be defined as the first or only column of some index such that it is possible to perform the equivalent of an indexed ``SELECT       MAX(*`ai_col`*)`` lookup on the table to obtain the maximum column value. The index is not required to be a `PRIMARY KEY` or `UNIQUE`, but to avoid duplicate values in the `AUTO_INCREMENT` column, those index types are recommended.

This section describes the `AUTO_INCREMENT` lock modes, usage implications of different `AUTO_INCREMENT` lock mode settings, and how `InnoDB` initializes the `AUTO_INCREMENT` counter.

##### InnoDB AUTO\_INCREMENT Lock Modes

This section describes the `AUTO_INCREMENT` lock modes used to generate auto-increment values, and how each lock mode affects replication. The auto-increment lock mode is configured at startup using the [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) variable.

The following terms are used in describing [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) settings:

- “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statements
	All statements that generate new rows in a table, including [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement"), [`INSERT ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/insert-select.html "15.2.7.1 INSERT ... SELECT Statement"), [`REPLACE`](https://dev.mysql.com/doc/refman/8.4/en/replace.html "15.2.12 REPLACE Statement"), [`REPLACE ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/replace.html "15.2.12 REPLACE Statement"), and [`LOAD             DATA`](https://dev.mysql.com/doc/refman/8.4/en/load-data.html "15.2.9 LOAD DATA Statement"). Includes “ simple-inserts ”, “ bulk-inserts ”, and “ mixed-mode ” inserts.
- “ Simple inserts ”
	Statements for which the number of rows to be inserted can be determined in advance (when the statement is initially processed). This includes single-row and multiple-row [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") and [`REPLACE`](https://dev.mysql.com/doc/refman/8.4/en/replace.html "15.2.12 REPLACE Statement") statements that do not have a nested subquery, but not [`INSERT             ... ON DUPLICATE KEY UPDATE`](https://dev.mysql.com/doc/refman/8.4/en/insert-on-duplicate.html "15.2.7.2 INSERT ... ON DUPLICATE KEY UPDATE Statement").
- “ Bulk inserts ”
	Statements for which the number of rows to be inserted (and the number of required auto-increment values) is not known in advance. This includes [`INSERT ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/insert-select.html "15.2.7.1 INSERT ... SELECT Statement"), [`REPLACE ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/replace.html "15.2.12 REPLACE Statement"), and [`LOAD             DATA`](https://dev.mysql.com/doc/refman/8.4/en/load-data.html "15.2.9 LOAD DATA Statement") statements, but not plain `INSERT`. `InnoDB` assigns new values for the `AUTO_INCREMENT` column one at a time as each row is processed.
- “ Mixed-mode inserts ”
	These are “ simple insert ” statements that specify the auto-increment value for some (but not all) of the new rows. An example follows, where `c1` is an `AUTO_INCREMENT` column of table `t1`:
	```sql
	INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');
	```
	Another type of “ mixed-mode insert ” is [`INSERT             ... ON DUPLICATE KEY UPDATE`](https://dev.mysql.com/doc/refman/8.4/en/insert-on-duplicate.html "15.2.7.2 INSERT ... ON DUPLICATE KEY UPDATE Statement"), which in the worst case is in effect an [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") followed by a [`UPDATE`](https://dev.mysql.com/doc/refman/8.4/en/update.html "15.2.17 UPDATE Statement"), where the allocated value for the `AUTO_INCREMENT` column may or may not be used during the update phase.

There are three possible settings for the [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) variable. The settings are 0, 1, or 2, for “ traditional ”, “ consecutive ”, or “ interleaved ” lock mode, respectively. Interleaved lock mode ([`innodb_autoinc_lock_mode=2`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)) is the default.

The default setting of interleaved lock mode in MySQL 8.4 reflects the change from statement-based replication to row based replication as the default replication type. Statement-based replication requires the consecutive auto-increment lock mode to ensure that auto-increment values are assigned in a predictable and repeatable order for a given sequence of SQL statements, whereas row-based replication is not sensitive to the execution order of SQL statements.

- `innodb_autoinc_lock_mode = 0` (“ traditional ” lock mode)
	The traditional lock mode provides the same behavior that existed before the [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) variable was introduced. The traditional lock mode option is provided for backward compatibility, performance testing, and working around issues with “mixed-mode inserts”, due to possible differences in semantics.
	In this lock mode, all “ INSERT-like ” statements obtain a special table-level `AUTO-INC` lock for inserts into tables with `AUTO_INCREMENT` columns. This lock is normally held to the end of the statement (not to the end of the transaction) to ensure that auto-increment values are assigned in a predictable and repeatable order for a given sequence of [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements, and to ensure that auto-increment values assigned by any given statement are consecutive.
	In the case of statement-based replication, this means that when an SQL statement is replicated on a replica server, the same values are used for the auto-increment column as on the source server. The result of execution of multiple [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements is deterministic, and the replica reproduces the same data as on the source. If auto-increment values generated by multiple [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements were interleaved, the result of two concurrent [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements would be nondeterministic, and could not reliably be propagated to a replica server using statement-based replication.
	To make this clear, consider an example that uses this table:
	```sql
	CREATE TABLE t1 (
	  c1 INT(11) NOT NULL AUTO_INCREMENT,
	  c2 VARCHAR(10) DEFAULT NULL,
	  PRIMARY KEY (c1)
	) ENGINE=InnoDB;
	```
	Suppose that there are two transactions running, each inserting rows into a table with an `AUTO_INCREMENT` column. One transaction is using an [`INSERT ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/insert-select.html "15.2.7.1 INSERT ... SELECT Statement") statement that inserts 1000 rows, and another is using a simple [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statement that inserts one row:
	```sql
	Tx1: INSERT INTO t1 (c2) SELECT 1000 rows from another table ...
	Tx2: INSERT INTO t1 (c2) VALUES ('xxx');
	```
	`InnoDB` cannot tell in advance how many rows are retrieved from the [`SELECT`](https://dev.mysql.com/doc/refman/8.4/en/select.html "15.2.13 SELECT Statement") in the [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statement in Tx1, and it assigns the auto-increment values one at a time as the statement proceeds. With a table-level lock, held to the end of the statement, only one [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statement referring to table `t1` can execute at a time, and the generation of auto-increment numbers by different statements is not interleaved. The auto-increment values generated by the Tx1 [`INSERT ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/insert-select.html "15.2.7.1 INSERT ... SELECT Statement") statement are consecutive, and the (single) auto-increment value used by the [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statement in Tx2 is either smaller or larger than all those used for Tx1, depending on which statement executes first.
	As long as the SQL statements execute in the same order when replayed from the binary log (when using statement-based replication, or in recovery scenarios), the results are the same as they were when Tx1 and Tx2 first ran. Thus, table-level locks held until the end of a statement make [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements using auto-increment safe for use with statement-based replication. However, those table-level locks limit concurrency and scalability when multiple transactions are executing insert statements at the same time.
	In the preceding example, if there were no table-level lock, the value of the auto-increment column used for the [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") in Tx2 depends on precisely when the statement executes. If the [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") of Tx2 executes while the [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") of Tx1 is running (rather than before it starts or after it completes), the specific auto-increment values assigned by the two [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements are nondeterministic, and may vary from run to run.
	Under the [consecutive](https://dev.mysql.com/doc/refman/8.4/en/innodb-auto-increment-handling.html#innodb-auto-increment-lock-mode-consecutive) lock mode, `InnoDB` can avoid using table-level `AUTO-INC` locks for “ simple insert ” statements where the number of rows is known in advance, and still preserve deterministic execution and safety for statement-based replication.
	If you are not using the binary log to replay SQL statements as part of recovery or replication, the [interleaved](https://dev.mysql.com/doc/refman/8.4/en/innodb-auto-increment-handling.html#innodb-auto-increment-lock-mode-interleaved) lock mode can be used to eliminate all use of table-level `AUTO-INC` locks for even greater concurrency and performance, at the cost of permitting gaps in auto-increment numbers assigned by a statement and potentially having the numbers assigned by concurrently executing statements interleaved.
- `innodb_autoinc_lock_mode = 1` (“ consecutive ” lock mode)
	In this mode, “ bulk inserts ” use the special `AUTO-INC` table-level lock and hold it until the end of the statement. This applies to all [`INSERT ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/insert-select.html "15.2.7.1 INSERT ... SELECT Statement"), [`REPLACE ...             SELECT`](https://dev.mysql.com/doc/refman/8.4/en/replace.html "15.2.12 REPLACE Statement"), and [`LOAD             DATA`](https://dev.mysql.com/doc/refman/8.4/en/load-data.html "15.2.9 LOAD DATA Statement") statements. Only one statement holding the `AUTO-INC` lock can execute at a time. If the source table of the bulk insert operation is different from the target table, the `AUTO-INC` lock on the target table is taken after a shared lock is taken on the first row selected from the source table. If the source and target of the bulk insert operation are the same table, the `AUTO-INC` lock is taken after shared locks are taken on all selected rows.
	“ Simple inserts ” (for which the number of rows to be inserted is known in advance) avoid table-level `AUTO-INC` locks by obtaining the required number of auto-increment values under the control of a mutex (a light-weight lock) that is only held for the duration of the allocation process, *not* until the statement completes. No table-level `AUTO-INC` lock is used unless an `AUTO-INC` lock is held by another transaction. If another transaction holds an `AUTO-INC` lock, a “ simple insert ” waits for the `AUTO-INC` lock, as if it were a “ bulk insert ”.
	This lock mode ensures that, in the presence of [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements where the number of rows is not known in advance (and where auto-increment numbers are assigned as the statement progresses), all auto-increment values assigned by any “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statement are consecutive, and operations are safe for statement-based replication.
	Simply put, this lock mode significantly improves scalability while being safe for use with statement-based replication. Further, as with “ traditional ” lock mode, auto-increment numbers assigned by any given statement are *consecutive*. There is *no change* in semantics compared to “ traditional ” mode for any statement that uses auto-increment, with one important exception.
	The exception is for “ mixed-mode inserts ”, where the user provides explicit values for an `AUTO_INCREMENT` column for some, but not all, rows in a multiple-row “ simple insert ”. For such inserts, `InnoDB` allocates more auto-increment values than the number of rows to be inserted. However, all values automatically assigned are consecutively generated (and thus higher than) the auto-increment value generated by the most recently executed previous statement. “ Excess ” numbers are lost.
- `innodb_autoinc_lock_mode = 2` (“ interleaved ” lock mode)
	In this lock mode, no “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statements use the table-level `AUTO-INC` lock, and multiple statements can execute at the same time. This is the fastest and most scalable lock mode, but it is *not safe* when using statement-based replication or recovery scenarios when SQL statements are replayed from the binary log.
	In this lock mode, auto-increment values are guaranteed to be unique and monotonically increasing across all concurrently executing “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statements. However, because multiple statements can be generating numbers at the same time (that is, allocation of numbers is *interleaved* across statements), the values generated for the rows inserted by any given statement may not be consecutive.
	If the only statements executing are “ simple inserts ” where the number of rows to be inserted is known ahead of time, there are no gaps in the numbers generated for a single statement, except for “ mixed-mode inserts ”. However, when “ bulk inserts ” are executed, there may be gaps in the auto-increment values assigned by any given statement.

##### InnoDB AUTO\_INCREMENT Lock Mode Usage Implications

- Using auto-increment with replication
	If you are using statement-based replication, set [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) to 0 or 1 and use the same value on the source and its replicas. Auto-increment values are not ensured to be the same on the replicas as on the source if you use [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) = 2 (“ interleaved ”) or configurations where the source and replicas do not use the same lock mode.
	If you are using row-based or mixed-format replication, all of the auto-increment lock modes are safe, since row-based replication is not sensitive to the order of execution of the SQL statements (and the mixed format uses row-based replication for any statements that are unsafe for statement-based replication).
- “ Lost ” auto-increment values and sequence gaps
	In all lock modes (0, 1, and 2), if a transaction that generated auto-increment values rolls back, those auto-increment values are “ lost ”. Once a value is generated for an auto-increment column, it cannot be rolled back, whether or not the “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statement is completed, and whether or not the containing transaction is rolled back. Such lost values are not reused. Thus, there may be gaps in the values stored in an `AUTO_INCREMENT` column of a table.
- Specifying NULL or 0 for the `AUTO_INCREMENT` column
	In all lock modes (0, 1, and 2), if a user specifies NULL or 0 for the `AUTO_INCREMENT` column in an [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement"), `InnoDB` treats the row as if the value was not specified and generates a new value for it.
- Assigning a negative value to the `AUTO_INCREMENT` column
	In all lock modes (0, 1, and 2), the behavior of the auto-increment mechanism is undefined if you assign a negative value to the `AUTO_INCREMENT` column.
- If the `AUTO_INCREMENT` value becomes larger than the maximum integer for the specified integer type
	In all lock modes (0, 1, and 2), the behavior of the auto-increment mechanism is undefined if the value becomes larger than the maximum integer that can be stored in the specified integer type.
- Gaps in auto-increment values for “ bulk inserts ”
	With [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) set to 0 (“ traditional ”) or 1 (“ consecutive ”), the auto-increment values generated by any given statement are consecutive, without gaps, because the table-level `AUTO-INC` lock is held until the end of the statement, and only one such statement can execute at a time.
	With [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) set to 2 (“ interleaved ”), there may be gaps in the auto-increment values generated by “ bulk inserts,” but only if there are concurrently executing “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statements.
	For lock modes 1 or 2, gaps may occur between successive statements because for bulk inserts the exact number of auto-increment values required by each statement may not be known and overestimation is possible.
- Auto-increment values assigned by “ mixed-mode inserts ”
	Consider a “ mixed-mode insert,” where a “ simple insert ” specifies the auto-increment value for some (but not all) resulting rows. Such a statement behaves differently in lock modes 0, 1, and 2. For example, assume `c1` is an `AUTO_INCREMENT` column of table `t1`, and that the most recent automatically generated sequence number is 100.
	```sql
	mysql> CREATE TABLE t1 (
	    -> c1 INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY, 
	    -> c2 CHAR(1)
	    -> ) ENGINE = INNODB;
	```
	Now, consider the following “ mixed-mode insert ” statement:
	```sql
	mysql> INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');
	```
	With [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) set to 0 (“ traditional ”), the four new rows are:
	```sql
	mysql> SELECT c1, c2 FROM t1 ORDER BY c2;
	+-----+------+
	| c1  | c2   |
	+-----+------+
	|   1 | a    |
	| 101 | b    |
	|   5 | c    |
	| 102 | d    |
	+-----+------+
	```
	The next available auto-increment value is 103 because the auto-increment values are allocated one at a time, not all at once at the beginning of statement execution. This result is true whether or not there are concurrently executing “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statements (of any type).
	With [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) set to 1 (“ consecutive ”), the four new rows are also:
	```sql
	mysql> SELECT c1, c2 FROM t1 ORDER BY c2;
	+-----+------+
	| c1  | c2   |
	+-----+------+
	|   1 | a    |
	| 101 | b    |
	|   5 | c    |
	| 102 | d    |
	+-----+------+
	```
	However, in this case, the next available auto-increment value is 105, not 103 because four auto-increment values are allocated at the time the statement is processed, but only two are used. This result is true whether or not there are concurrently executing “ [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") -like ” statements (of any type).
	With [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) set to 2 (“ interleaved ”), the four new rows are:
	```sql
	mysql> SELECT c1, c2 FROM t1 ORDER BY c2;
	+-----+------+
	| c1  | c2   |
	+-----+------+
	|   1 | a    |
	|   x | b    |
	|   5 | c    |
	|   y | d    |
	+-----+------+
	```
	The values of *`x`* and *`y`* are unique and larger than any previously generated rows. However, the specific values of *`x`* and *`y`* depend on the number of auto-increment values generated by concurrently executing statements.
	Finally, consider the following statement, issued when the most-recently generated sequence number is 100:
	```sql
	mysql> INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (101,'c'), (NULL,'d');
	```
	With any [`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) setting, this statement generates a duplicate-key error 23000 (`Can't write; duplicate key in             table`) because 101 is allocated for the row `(NULL, 'b')` and insertion of the row `(101, 'c')` fails.
- Modifying `AUTO_INCREMENT` column values in the middle of a sequence of [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") statements
	If you modify an `AUTO_INCREMENT` column value to a value larger than the current maximum auto-increment value, the new value is persisted, and subsequent [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") operations allocate auto-increment values starting from the new, larger value. This behavior is demonstrated in the following example:
	```sql
	mysql> CREATE TABLE t1 (
	    -> c1 INT NOT NULL AUTO_INCREMENT,
	    -> PRIMARY KEY (c1)
	    ->  ) ENGINE = InnoDB;
	mysql> INSERT INTO t1 VALUES(0), (0), (3);
	mysql> SELECT c1 FROM t1;
	+----+
	| c1 |
	+----+
	|  1 |
	|  2 |
	|  3 |
	+----+
	mysql> UPDATE t1 SET c1 = 4 WHERE c1 = 1;
	mysql> SELECT c1 FROM t1;
	+----+
	| c1 |
	+----+
	|  2 |
	|  3 |
	|  4 |
	+----+
	mysql> INSERT INTO t1 VALUES(0);
	mysql> SELECT c1 FROM t1;
	+----+
	| c1 |
	+----+
	|  2 |
	|  3 |
	|  4 |
	|  5 |
	+----+
	```

##### InnoDB AUTO\_INCREMENT Counter Initialization

This section describes how `InnoDB` initializes `AUTO_INCREMENT` counters.

If you specify an `AUTO_INCREMENT` column for an `InnoDB` table, the in-memory table object contains a special counter called the auto-increment counter that is used when assigning new values for the column.

The current maximum auto-increment counter value is written to the redo log each time it changes and saved to the data dictionary on each checkpoint; this makes the current maximum auto-increment counter value persistent across server restarts.

On a server restart following a normal shutdown, `InnoDB` initializes the in-memory auto-increment counter using the current maximum auto-increment value stored in the data dictionary.

On a server restart during crash recovery, `InnoDB` initializes the in-memory auto-increment counter using the current maximum auto-increment value stored in the data dictionary and scans the redo log for auto-increment counter values written since the last checkpoint. If a redo-logged value is greater than the in-memory counter value, the redo-logged value is applied. However, in the case of an unexpected server exit, reuse of a previously allocated auto-increment value cannot be guaranteed. Each time the current maximum auto-increment value is changed due to an [`INSERT`](https://dev.mysql.com/doc/refman/8.4/en/insert.html "15.2.7 INSERT Statement") or [`UPDATE`](https://dev.mysql.com/doc/refman/8.4/en/update.html "15.2.17 UPDATE Statement") operation, the new value is written to the redo log, but if the unexpected exit occurs before the redo log is flushed to disk, the previously allocated value could be reused when the auto-increment counter is initialized after the server is restarted.

The only circumstance in which `InnoDB` uses the equivalent of a ``SELECT MAX(ai_col) FROM         *`table_name`* FOR UPDATE`` statement to initialize an auto-increment counter is when [importing a table](https://dev.mysql.com/doc/refman/8.4/en/innodb-table-import.html "17.6.1.3 Importing InnoDB Tables") without a `.cfg` metadata file. Otherwise, the current maximum auto-increment counter value is read from the `.cfg` metadata file if present. Aside from counter value initialization, the equivalent of a ``SELECT MAX(ai_col) FROM         *`table_name`*`` statement is used to determine the current maximum auto-increment counter value of the table when attempting to set the counter value to one that is smaller than or equal to the persisted counter value using an ``ALTER TABLE ... AUTO_INCREMENT =         *`N`*`` statement. For example, you might try to set the counter value to a lesser value after deleting some records. In this case, the table must be searched to ensure that the new counter value is not less than or equal to the actual current maximum counter value.

A server restart does not cancel the effect of the `AUTO_INCREMENT = N` table option. If you initialize the auto-increment counter to a specific value, or if you alter the auto-increment counter value to a larger value, the new value is persisted across server restarts.

[`ALTER TABLE ...           AUTO_INCREMENT = N`](https://dev.mysql.com/doc/refman/8.4/en/alter-table.html "15.1.9 ALTER TABLE Statement") can only change the auto-increment counter value to a value larger than the current maximum.

The current maximum auto-increment value is persisted, preventing the reuse of previously allocated values.

If a [`SHOW TABLE STATUS`](https://dev.mysql.com/doc/refman/8.4/en/show-table-status.html "15.7.7.38 SHOW TABLE STATUS Statement") statement examines a table before the auto-increment counter is initialized, `InnoDB` opens the table and initializes the counter value using the current maximum auto-increment value that is stored in the data dictionary. The value is then stored in memory for use by later inserts or updates. Initialization of the counter value uses a normal exclusive-locking read on the table which lasts to the end of the transaction. `InnoDB` follows the same procedure when initializing the auto-increment counter for a newly created table that has a user-specified auto-increment value greater than 0.

After the auto-increment counter is initialized, if you do not explicitly specify an auto-increment value when inserting a row, `InnoDB` implicitly increments the counter and assigns the new value to the column. If you insert a row that explicitly specifies an auto-increment column value, and the value is greater than the current maximum counter value, the counter is set to the specified value.

`InnoDB` uses the in-memory auto-increment counter as long as the server runs. When the server is stopped and restarted, `InnoDB` reinitializes the auto-increment counter, as described earlier.

The [`auto_increment_offset`](https://dev.mysql.com/doc/refman/8.4/en/replication-options-source.html#sysvar_auto_increment_offset) variable determines the starting point for the `AUTO_INCREMENT` column value. The default setting is 1.

The [`auto_increment_increment`](https://dev.mysql.com/doc/refman/8.4/en/replication-options-source.html#sysvar_auto_increment_increment) variable controls the interval between successive column values. The default setting is 1.

##### Notes

When an `AUTO_INCREMENT` integer column runs out of values, a subsequent `INSERT` operation returns a duplicate-key error. This is general MySQL behavior.