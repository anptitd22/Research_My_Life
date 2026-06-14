---
title: "MySQL :: MySQL 9.7 Reference Manual :: 26 Partitioning"
source: "https://dev.mysql.com/doc/refman/9.7/en/partitioning.html"
author:
published:
created: 2026-06-14
description:
tags:
  - "clippings"
---
## Chapter 26 Partitioning

This chapter discusses user-defined partitioning.

Note

Table partitioning differs from partitioning as used by window functions. For information about window functions, see [Section 14.20, “Window Functions”](https://dev.mysql.com/doc/refman/9.7/en/window-functions.html "14.20 Window Functions").

In MySQL 9.7, partitioning support is provided by the [`InnoDB`](https://dev.mysql.com/doc/refman/9.7/en/innodb-storage-engine.html "Chapter 17 The InnoDB Storage Engine") and [`NDB`](https://dev.mysql.com/doc/refman/9.7/en/mysql-cluster.html "Chapter 25 MySQL NDB Cluster 9.7") storage engines.

MySQL 9.7 does not currently support partitioning of tables using any storage engine other than `InnoDB` or `NDB`, such as [`MyISAM`](https://dev.mysql.com/doc/refman/9.7/en/myisam-storage-engine.html "18.2 The MyISAM Storage Engine"). An attempt to create a partitioned tables using a storage engine that does not supply native partitioning support fails with [`ER_CHECK_NOT_IMPLEMENTED`](https://dev.mysql.com/doc/mysql-errors/9.7/en/server-error-reference.html#error_er_check_not_implemented).

If you are compiling MySQL 9.7 from source, configuring the build with `InnoDB` support is sufficient to produce binaries with partition support for `InnoDB` tables. For more information, see [Section 2.8, “Installing MySQL from Source”](https://dev.mysql.com/doc/refman/9.7/en/source-installation.html "2.8 Installing MySQL from Source").

Nothing further needs to be done to enable partitioning support by `InnoDB` (for example, no special entries are required in the `my.cnf` file).

It is not possible to disable partitioning support by the `InnoDB` storage engine.

See [Section 26.1, “Overview of Partitioning in MySQL”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-overview.html "26.1 Overview of Partitioning in MySQL"), for an introduction to partitioning and partitioning concepts.

Several types of partitioning are supported, as well as subpartitioning; see [Section 26.2, “Partitioning Types”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-types.html "26.2 Partitioning Types"), and [Section 26.2.6, “Subpartitioning”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-subpartitions.html "26.2.6 Subpartitioning").

[Section 26.3, “Partition Management”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-management.html "26.3 Partition Management"), covers methods of adding, removing, and altering partitions in existing partitioned tables.

[Section 26.3.4, “Maintenance of Partitions”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-maintenance.html "26.3.4 Maintenance of Partitions"), discusses table maintenance commands for use with partitioned tables.

The [`PARTITIONS`](https://dev.mysql.com/doc/refman/9.7/en/information-schema-partitions-table.html "28.3.26 The INFORMATION_SCHEMA PARTITIONS Table") table in the `INFORMATION_SCHEMA` database provides information about partitions and partitioned tables. See [Section 28.3.26, “The INFORMATION\_SCHEMA PARTITIONS Table”](https://dev.mysql.com/doc/refman/9.7/en/information-schema-partitions-table.html "28.3.26 The INFORMATION_SCHEMA PARTITIONS Table"), for more information; for some examples of queries against this table, see [Section 26.2.7, “How MySQL Partitioning Handles NULL”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-handling-nulls.html "26.2.7 How MySQL Partitioning Handles NULL").

For known issues with partitioning in MySQL 9.7, see [Section 26.6, “Restrictions and Limitations on Partitioning”](https://dev.mysql.com/doc/refman/9.7/en/partitioning-limitations.html "26.6 Restrictions and Limitations on Partitioning").

You may also find the following resources to be useful when working with partitioned tables.

**Additional Resources.** Other sources of information about user-defined partitioning in MySQL include the following:

- [MySQL Partitioning Forum](https://forums.mysql.com/list.php?106)
	This is the official discussion forum for those interested in or experimenting with MySQL Partitioning technology. It features announcements and updates from MySQL developers and others. It is monitored by members of the Partitioning Development and Documentation Teams.
- [PlanetMySQL](http://www.planetmysql.org/)
	A MySQL news site featuring MySQL-related blogs, which should be of interest to anyone using my MySQL. We encourage you to check here for links to blogs kept by those working with MySQL Partitioning, or to have your own blog added to those covered.