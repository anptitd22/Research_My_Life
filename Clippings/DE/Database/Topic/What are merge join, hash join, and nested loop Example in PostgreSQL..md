---
title: "What are merge join, hash join, and nested loop? Example in PostgreSQL."
source: "https://gelovolro.medium.com/what-are-merge-join-hash-join-and-nested-loop-example-in-postgresql-29123ca18fd1"
author:
  - "[[gelovolro]]"
published: 2024-09-18
created: 2026-05-14
description: "In this article, let’s examine three key types of physical joins that PostgreSQL uses, when performing logical outer and inner joins:"
tags:
  - "clippings"
---
## Introduction.

In this article, let’s examine three key types of physical joins that PostgreSQL uses, when performing logical outer and inner joins:

- merge join,
- hash join,
- and nested loop.

**Why did I start by mentioning the distinction between physical and logical joins?** These are the different levels. When you write the SQL queries using LEFT OUTER JOIN or RIGHT JOIN, this is the logical level of JOINs. It is typically used for the Cartesian products, where the rows satisfying the JOIN condition are kept. How such queries are executed “ *under the hood* ” in the RDBMS engine at the physical level? It’s depended on several factors:

- *the size of the tables involved in the join operation,*
- *the amount of memory allocated to “work\_mem” for sort operations and “hash\_mem\_multiplier” for hash operations,*
- *the presence of indexes,*
- *the comparison or equality operations applied during the JOIN.*

Depending on these conditions, PostgreSQL’s planner may choose one method (algorithm) or another one to execute the logical joins.

## Description of each physical JOIN type.

**Merge join**, this is a method used, when both tables are fairly large and there is already sorted data on the fields (the JOIN keys) through indexes. If the tables data is not presorted, PostgreSQL will sort them before performing the JOIN operation, which may increase costs. The key difference from a hash join is that a merge join requires the data used in the JOIN keys be sorted, whereas a hash join doesn’t require sorting at all. It simply builds a hash table and uses it to find the matches. It is also worth adding that a merge join is particularly useful for range joins (e.g., `>=`, `<=`), as after sorting it can efficiently handle such conditions.

**Hash join**, this is a method using hashing, which is used when the data required for the JOIN operation is not sorted. The algorithm works by selecting one of the tables (usually the smaller one by the memory size) and creating a record in a hash table for each of its rows. Then the other table (the larger one) is scanned, and each row is compared against the hash table. If a match is found for the required values, the rows are combined. **It’s also worth emphasizing, that PostgreSQL takes the smaller table to create the hash table in order to minimize memory usage**. This hash table is then used to quickly find the matches in the second larger table. To conclude, a hash join is most commonly used for the large tables, when there are no indexes or when the data is not sorted, and the JOIN condition involves exact matches `=`. Range joins are generally less efficient with a hash join algorithm.

**Nested Loop**, this is a method using the nested loops, where for each value in one table, it searches for a corresponding value in the other table. More specifically, it takes the first value from the first table and sequentially compares it with all values in the second table. If a match is found, the record is included in the final data set. Once the value from the first table has been compared with all values in the second table, the next value from the first smaller table is taken, and the comparison occurs again with all values in the second table. This process continues until every value from the first table has been compared with every value from the second table, which is why the method is called a “nested loop” (since the entire process continues until all values are compared in a nested manner).

**Reading other articles from web, I have seen information, that nested loops are “always” strictly used when one table is small and the other is large. This is NOT necessarily true**, and I’m going to provide the SQL code with an example and an explanation of the query plan using “EXPLAIN ANALYZE”:

```c
DROP TABLE IF EXISTS public.test_nestedloop_tbl1, public.test_nestedloop_tbl2;

CREATE TABLE public.test_nestedloop_tbl1(
    id    INT  PRIMARY KEY,
    value TEXT NOT NULL
);

CREATE TABLE public.test_nestedloop_tbl2(
    id          INT  PRIMARY KEY,
    description TEXT NOT NULL
);

INSERT INTO
    public.test_nestedloop_tbl1(id, value)
SELECT
    i, 'Value ' || i
FROM
    generate_series(1, 1000) AS gs(i);

INSERT INTO
    public.test_nestedloop_tbl2(id, description)
SELECT
    i, 'Description ' || i
FROM
    generate_series(1, 1000) AS gs(i);

EXPLAIN ANALYZE
SELECT
    tbl1.value,
    tbl2.description
FROM
    public.test_nestedloop_tbl1 tbl1
JOIN
    public.test_nestedloop_tbl2 tbl2
ON
    tbl1.id <= tbl2.id;
```

If you execute this query, you will see a nested loop in the query plan output, even though the size of the two tables is the same:

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*B2r3NXq1VXXIrH-c)

nested loop executed query plan

It would be more better to say next detail, that **the nested loops are usually more efficient when one table is smaller than the other. Such words are likely more accurate.** This is because the outer loop iterates over the smaller table, while the inner loop iterates over the larger one, making it less resource-intensive.

## Examples for merge & hash joins

To demonstrate the merge/hash joins work, we will need to prepare a couple of tables with data inserted using random values. Execute the following SQL code:

```c
DROP TABLE IF EXISTS public.test_join_tbl1, public.test_join_tbl2;

CREATE TABLE public.test_join_tbl1(
    id    INT,
    value TEXT
);

CREATE TABLE public.test_join_tbl2(
    id          INT,
    description TEXT
);

INSERT INTO
    public.test_join_tbl1(id, value)
SELECT
    i, 'Value ' || i
FROM
    generate_series(1, 1000000) AS gs(i)
ORDER BY
    random();

INSERT INTO
    public.test_join_tbl2(id, description)
SELECT
    i, 'Description ' || i
FROM
    generate_series(1, 1000000) AS gs(i)
ORDER BY
    random();
```

After creating the tables and inserting data into them, check that the data has appeared:

```c
SELECT * FROM public.test_join_tbl1 LIMIT 10 OFFSET 0;
SELECT * FROM public.test_join_tbl2 LIMIT 10 OFFSET 0;
```

Check the output:

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*i-qGTqmV6vOItemv)

randomly generated values

Now let’s execute a query that will show us the hash join:

```c
EXPLAIN ANALYZE
SELECT
  tbl1.value,
  tbl2.description
FROM
  public.test_join_tbl1 tbl1
JOIN
  public.test_join_tbl2 tbl2
ON
  tbl1.id = tbl2.id;
```

**Important!** If you are testing this SQL code on PostgreSQL instance with a significantly increased “work\_mem” value (e.g.: 128–256 MB) or if the PostgreSQL instance is idle, meaning it’s not being actively used at the moment, the planner may choose a merge join method, because it will have enough free RAM memory to sort the data in advance for the merge join operation. In such a case, set the lower value for the “work\_mem” parameter, e.g.:

```c
SET work_mem = '64kB';
```

and you can increase the amount of generated data using the generates\_series() function, which was used earlier when we’ve created the tables data. Then repeat that SQL JOIN query. **Why is so? It isn’t possible to give a definitive answer here, as it depends ON** the computational resources (CPU, RAM) of your environment where your testing activity will take place, as well as the settings of the PostgreSQL instance itself (such as the aforementioned “work\_mem” parameter), since the planner will determine the best algorithm for the query execution at the current moment, based on the available resources and the amount of data, which is going to be used. If all resources are free and there is no parallel queries activity, the merge join algorithm might be chosen. In a real “production” environment, the RDBMS service usually is under parallel query load, which making the advantages of using indexes for the merge/hash join example more apparent.

## Get gelovolro’s stories in your inbox

Join Medium for free to get updates from this writer.

**Let’s return to the query plan output:**

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*r-x73z7cKf1xdcc4)

hash join plan

## What is needed to get a merge join?

For this, you need to add new indexes:

```c
CREATE INDEX idx_t1_id ON test_join_tbl1(id);
CREATE INDEX idx_t2_id ON test_join_tbl2(id);
```

And repeat the query with the ORDER BY part:

```c
CREATE INDEX idx_t1_id ON test_join_tbl1(id);
CREATE INDEX idx_t2_id ON test_join_tbl2(id);

EXPLAIN ANALYZE
SELECT
  tbl1.value,
  tbl2.description
FROM
  public.test_join_tbl1 tbl1
JOIN
  public.test_join_tbl2 tbl2
ON
  tbl1.id = tbl2.id
ORDER BY
  tbl1.id,
  tbl2.id;
```

As a result, a merge join will appear in the output:

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*V2PE1L23Ugfanuxz)

merge join plan

So, we have achieved the output of all three types of physical joins: merge join, hash join, and nested loop. It’s also worth mentioning that you can disable join types using the following commands:

```c
SET enable_hashjoin  = off;
SET enable_mergejoin = off;
SET enable_nestloop  = off;
```

But in reality, you won’t be able to completely disable the use of, for example, nested loop for joins:) In some cases, nested loop may be the only available join method, especially when the join condition does not imply an exact match. For example, when using comparison operators like `<`, `<=`, `>`, `>=`.

**Some details about hash join**

First, about the algorithm itself…

It’s worth mentioning that the hash join implemented in PostgreSQL is not quite the classic one, which uses the build & probe phases. PostgreSQL uses a hybrid hash join, as described by the PostgreSQL developers themselves: [https://github.com/postgres/postgres/blob/master/src/backend/executor/nodeHashjoin.c](https://github.com/postgres/postgres/blob/master/src/backend/executor/nodeHashjoin.c) "

> This is based on the “hybrid hash join” algorithm described shortly in the [https://en.wikipedia.org/wiki/Hash\_join#Hybrid\_hash\_join](https://dzen.ru/away?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FHash_join%23Hybrid_hash_join) and in detail in the referenced paper: “An Adaptive Hash Join Algorithm for Multiuser Environments” Hansjörg Zeller; Jim Gray (1990). Proceedings of the 16th VLDB conference. Brisbane: 186–197.

In brief, it is a combination of the classic hash join algorithm and the grace hash join.

## Impact of work\_mem

When viewing the plan with a hash join, you might see next line in the planner:

> Buckets: 1048576 Batches: 1 Memory Usage: 58583kB

**Buckets** are the number of slots in the hash table, that PostgreSQL uses to distribute data from one of the tables.

**Batches** are the number of partitions into which PostgreSQL divides the data if the hash table is too large to fit into memory.

**A part of the planner output indicates that all the data of the hash table fit entirely into RAM and there was no need to split the data into multiple batches**. Also remember, that the hash table is built using one of the tables during the JOIN operation, which is smaller in memory size. Therefore, this output shows statistics relative to the smaller table.

But, what happens if we set a smaller value for the “work\_mem” parameter?

```c
SET work_mem = '64kB';
SHOW work_mem;
```

And repeat the query without the ORDER BY part to see the hash join output:

```c
EXPLAIN ANALYZE
SELECT
  tbl1.value,
  tbl2.description
FROM
  public.test_join_tbl1 tbl1
JOIN
  public.test_join_tbl2 tbl2
ON
  tbl1.id = tbl2.id;
```

The output will be next:

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*sq2CX-SpnPfXsAh4)

buckets and batches in hash join plan

The number of buckets and batches has changed. This shows the linear impact of the “work\_mem” parameter. Additionally, it is worth to mention the other PostgreSQL parameter, “hash\_mem\_multiplier”. This parameter determines the maximum amount of memory, that can be allocated for hash operations.

The total amount is calculated as the product of the “work\_mem” value and the “hash\_mem\_multiplier” coefficient. **By default, “hash\_mem\_multiplier” is set to 2.0, meaning that twice the amount of memory is available for hash operations compared to the base “work\_mem” value.**

It is better to increase the “hash\_mem\_multiplier” value, if queries frequently result in data being spilled to disk, and simply increasing “work\_mem” may occur the memory errors in PostgreSQL.

PS: The following version of PostgreSQL was used, when I was writing this article: v16.3

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*sgu6JGN3mlsKybLJZ78OdQ.png)