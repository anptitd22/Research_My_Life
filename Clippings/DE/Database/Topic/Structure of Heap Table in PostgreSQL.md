---
title: "Structure of Heap Table in PostgreSQL"
source: "https://medium.com/quadcode-life/structure-of-heap-table-in-postgresql-d44c94332052"
author:
  - "[[Quadcode Team]]"
published: 2022-12-06
created: 2026-06-29
description: "Metadata and how it’s arranged in Postgres. What is Table page and its physical representation on disk. Fillfactor parameters that affect system performance. Object identifiers."
tags:
  - "clippings"
---
[Sitemap](https://medium.com/sitemap/sitemap.xml)## [Quadcode](https://medium.com/quadcode-life?source=post_page---publication_nav-526934940ae0-d44c94332052---------------------------------------)

[![Quadcode](https://miro.medium.com/v2/resize:fill:38:38/1*5q8mdbyZh2rpkIsHDNG8xg.jpeg)](https://medium.com/quadcode-life?source=post_page---post_publication_sidebar-526934940ae0-d44c94332052---------------------------------------)

Quadcode is an international IT company. We develop trading SaaS platform, banking and internal solutions. Over 50 million traders in 150+ countries appreciate the benefits of our platform.

My name is Azat Yakupov. I’m a Quadcode Data Architect. In parallel with my work, I give lectures at the university and conduct courses in data engineering, data architecture and data analytics. I’ve been in the IT industry for more than 20 years, more than 6 of them in architecture. I want to share some of the extensive experience that I’ve accumulated during this time.

This article is about Heap tables, aka sandbox tables. In it we’ll look at:

- Metadata and how it’s arranged in Postgres.
- What is Table page and its physical representation on disk.
- Fillfactor parameters that affect system performance.
- Object identifiers.
![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*FFLThKGcVoqEA5ccpcItRg.png)

## Heap Tables

All data engineering begins at the moment when we create a table, describe attributes and add content there. Therefore, let’s start our analysis with a regular table. How is it stored and logically represented on disk?

Many people think that it’s a real structure that is stored as a set of files, and of ordered strings. But actually, PostgreSQL has a little randomness that represents our table in an order where data can be stored on different pages and different places within the page itself:

![Data in Heap tables can be stored randomly on different pages and different places within the page](https://miro.medium.com/v2/resize:fit:640/format:webp/1*j5g-GwepekrpQ0vUPkvPiw.png)

Figure 1

It happens because there is a `*VACUUM*` mechanism in Postgres. It redistributes data, cleaning out dead records and bringing a little chaos to the rows.

The useful `*ORDER BY*` construction helps to prevent such a random data set. If you want to sort the data so that it doesn’t appear randomly, it’s better to use `*ORDER BY*`, knowing that you need accurate sorting by certain attributes and data. For example, it helps if there was one data snapshot without `ORDER BY` an hour ago and now another snapshot appears, because some internal Postgres process started as `*VACUUM*` and made some transformations in your pages.

Let’s look at the usual table syntax. Here I create a table with the name `*t*`. The table has three columns: A, B, C.

```c
CREATE TABLE public.t
(
 A INTEGER,
 B INTEGER,
 C VARCHAR
);
```

Having created the table, we can access its metadata. The request seems scary, but in fact it just accesses the Postgres metadata, getting the necessary information for further investigation:

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*_IjCbR6bzrODYOgvbV3u2w.png)

Figure 2

The metadata contains the following information:

- **OID** — the object identifier that’s created inside Postgres. It is based on a systemic sequence. This sequence returns something unique each time for a new object that we’re creating: a column, a function, a trigger, a virtual table, and so on. By the OID we can refer to the object.
- **Relation name** — the name of the object.
- **Schema** — public.
- **Object owner** — the person who created the table and is responsible for distributing accesses to it.
- **Tablespace** — the default for Postgres is pg\_default.
- **Amount pages, amount tuples** — columns responsible for the number of pages and rows.
- **TOAST tables** — satellite tables. They allow the main table to function and allow you to split long rows if they don’t fit on the basis of a strategy or a selected policy on a particular page.
- **Type** — table type. We’re now considering a regular logged table, which is created based on the `*CREATE TABLE*` command.
- **File path** — the path that is mapped at the operating system level in pg\_home, where your Postgres instance is located.
- **Relation size** — the total size of the table in bytes. Since we haven’t inserted any data into the table, Relation size = 0 bytes.

In the path line, you can see that we have a 16387 file. This is a binary file that will contain the content of our table if we do the filling using INSERT, and then do UPDATEs or DELETEs. Note that this file matches the OID of the table.

Let’s conduct a little experiment:

1. Insert data into the table.
2. Generate a row.
3. Recalculate the table statistics.
4. Look at the contents of the table, file and meta layer.

As a result, we have changes in the metadata. In Figure 3, these changes are highlighted in turquoise:

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*wHf7LfkavJ9fgXl1PFoGHQ.png)

Figure 3

The number of pages is now 1, the number of records is 1, and the volume of relationships has dramatically increased to 8 kilobytes. What’s the reason for this? The fact is that when we insert 1 row, the minimum atomic element is immediately allocated. Within Postgres storage, it’s a page, and it’s always 8 kilobytes. The exception is when you’ve completely rebuilt Postgres and increased the size of the page or decreased it by a factor of two.

How do Heap tables work? Imagine that we have a set of tables: Table 1 and Table 2 are smaller than a gigabyte and Postgres understands through its metadata which files these tables refer to through the logical layer. This strongly resembles the ANSI-SPARC architecture, when it’s assumed that we have an external and internal conceptual data layer for working with structures. There’s also an inner layer; in Figure 4 it’s highlighted with a green outline, and the outer layer is tables.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*xvPSbuTJyTwLp4H52QBC8g.png)

Figure 4

Table 1 corresponds to the red file, with the same name as its internal identifier. Table 2 corresponds to the blue square. But there’s a nuance. If the table is larger than a gigabyte, then Postgres begins to split it into separate files with a size of 1 gigabyte. Thus, suffixes appear, indicating at the operating system level in which order to read parts of our table. One table at the metadata level corresponds to several files at the operating system level, and the suffix helps to organize the assignment of the content.

## Table Page

Let’s now consider the page itself.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*l76tI6zjeh0A6AZKzqqJjw.png)

Figure 5

The Postgres page structure strongly resembles that used in Oracle and MySQL. These structures simultaneously grow from top to bottom and from bottom to top. When you do INSERTs, T1, T2, T3 or tuples are born, which are written from bottom to left. They grow until they meet somewhere closer to the beginning of the page with the values of pointers (I1, I2, I3).

Pointers point to specific tuples that are stored below in the file. This structure has gone through fire, water and copper pipes in terms of optimization and proper storage of information. It helps the Postgres optimizer to use its advantages in terms of working with offsets and pointers.

The page has **a header**, which is a meta layer that stores an interesting structure showing the total amount of space in the page. Also in the page structure there are **transaction pointers, and a meta layer for each tuple, for each row**. There’s a special zone, but it’s not used for standard tables, since it’s not necessary for index structures. Why? Because this page is an atomic element not only for tables, but also for indexes. One common structure is summed up for all kinds of storage.

The page has **Fillfactor**, which we’ll talk more about in the next chapter. By default, Fillfactor is set to 100%. This means that Postgres will fill the page as much as possible. What does it mean to fill it as much as possible? Roughly speaking, until the lower zone meets the upper one. As soon as they touch, a new page will be born. The situation will repeat until 8 kilobytes are filled, then the next 8 kilobytes will be born, and so on. This is how the table grows.

Let’s look at examples.

```c
CREATE EXTENSION pgstattuple;

SELECT *
FROM pgstattuple(16387);
```

I used the extension `*pgstattuple*`. It allows you to view statistics without resorting to massive metadata selectors. Using the extension API, I transmit the internal ID of my table. The result is a cross-section of statistics: by table length, number of live records, dead records — or zombies, as I call them — and so on.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*egVwCM_e_dr4AWCPLGueZA.png)

Figure 6

I also want to study the page itself and see what’s inside. I’m interested to see if the page matches what we’ll see through `*pageinspect*`. For example, examine the header. I send the table name `t`, and give the number of the page — `0`, because we only have one page.

```c
CREATE EXTENSION pageinspect;

SELECT *
FROM page_header(get_raw_page('public.t',0));
```
![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*Ycq7tIypw5FXn4AndnXHlQ.png)

Figure 7

**The lsn number** is a unique sequence number that was allocated to my change when it was inserted. This number is used both for data recovery and replication.

If we consider the page header, which is 24 bytes, then what structures are there?

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*fME9LCSCBZM9Cu9KkurtCQ.png)

Figure 8

The structures are presented and marked up byte by byte: how much and what it occupies. The header is clearly described in terms of what elements are included in it, so that the Postgres optimizer can go into the header before reading the entire page and understand what it’s dealing with. And when you first start reading, you can tell right away how much room there is in that page.

In Figure 9, you can look at our header, and pointer (I1), which points to our tuple (T1), which starts with an offset of 8144 bytes. This T1 tuple contains the string that we generated at the insert level.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*y1Vxt4RS1LX4VPCCsqJ4OQ.png)

Figure 9

If you look not at the metadata, but at the string itself, we can see interesting attributes regarding not only the length of the tuple itself, but also by the transaction number that was applied.

Our string (T1) is 42 bytes. How’s it all stored? The fact is that each string also stores metadata. And these 42 bytes are divided into two parts:

1. The structure drawn on the left in Figure 10. It determines the byte size for storing each field of this metadata.
2. Tuple data, and they’re 18 bytes.

It turns out that we have 18 out of 42 bytes — this is user data. Note that metadata is so important that it takes up more than half of the space.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*h0WvlC8-K9xlkvH7a-sFSw.png)

Figure 10

Another metadata slice is **the CTID pointer**. It contains the address of the page plus the address of the tuple inside this page.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*Laml56CiORbi6m_zNb4dqA.png)

Figure 11

This structure is very similar to an array, a record, which indicates that this string *“1 1 string #1”* is stored in the null page in the first tuple. If we add OID relations here, we’ll essentially get the coordinates of the information search:

1. a data file;
2. a page in this data file;
3. a tuple where we need to get this information from a specific page.

B-Tree indexes are built on this principle, when the leaves in this B-Tree index point to a particular CTID specified in a particular file.

## Fillfactor

Fillfactor is the ability of a page to store information only up to a certain level. As I’ve already said, its default value of 100% means that the page will be filled to the end.

In the example in Figure 12, I set Fillfactor = 50%. It means that by bringing the `*ALTER TABLE*` type of table to 50%, new pages will be born so that 50% can’t be touched with inserts. That is, when you insert a new tupple, it won’t cross the red borders in the figure: a new page will be born every time the tuples reach the thin red line.

```c
ALTER TABLE public.t SET (FILLFACTOR = 50);
```
![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*TJzs7uthdchmn-2_U0_0GQ.png)

Figure 12

Fillfactor is needed for future UPDATEs of the table. The fact is that when you make an UPDATE, Postgres tries to save it as much as possible in the same page where the original string was stored, and not mutate it somewhere else. This is a good practice for Postgres.

To help Postgres and free up space within the page, you can leave it a space for each page and say: “Let’s install Fillfactor 90%; it will be filled 90%, and 10% will remain for future possible UPDATEs.” But we need to understand that if these are operational tables that are updated and changed very often, then yes, it’s worth trying with a threshold of 90%. If these are static tables like directories or dimensions, then you should immediately set Fillfactor to 100%, because the data is first cleared and then refilled. But if they’re updated, then you need to immediately think about the correct policy for Fillfactor.

Fillfactor can be set as for a table that already exists:

```c
ALTER TABLE public.t SET (FILLFACTOR = 100);
```

And for a newly compiled table, explicitly specifying in the options that Fillfactor is equal to such and such a value:

```c
CREATE TABLE public.t
(
 A INTEGER,
 B INTEGER,
 C VARCHAR
) WITH (FILLFACTOR = 100);
```

But once I set the Fillfactor on an existing table, and I want to recalculate the historical pages too, then I have to do a `*VACUUM FULL*` operation. In essence, this operation will copy data from one file to another:

```c
VACUUM FULL public.t
```

`*VACUUM FULL*` is an operation that is quite critical for highload. It blocks read and write traffic for the table until the operation itself is finished. I advise you to be careful with this and [read the documentation](https://postgrespro.com/docs/postgresql/14/sql-vacuum) that VACUUM can lead to blockages. Blocking the table in turn causes “connection refused” and “connection lost”, if you set the timeout to a small enough value.

## Object identifier

In the end of the article I want to pay special attention to one property of the OID. Using the OID for the indications in the table is bad.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*pMUu1ed3ZRPxTgtzoK1W3w.png)

Figure 13

OID is a system sequence that generates a number every time you create a new object: a table, column, function, procedure, trigger, etc. But if you specify the OID in the table, it will mean that each row insertion will get the ID value from the system sequence.

It would seem that there’s nothing terrible in that. You could say, “Why not use an internal system identifier, essentially a primary key, a primary key that’s generated each time for each line?” But there’s a problem. Our OID is a particular type. And the problem comes when the system sequence ends and the counter has no choice but to start from scratch.

The thing is that you already have objects starting from scratch in your database. In that case Postgres will collapse and everything will fold up and close. And that’s where we have to decide:

1. How and by what approaches to restore this data.
2. How to deal with such a cluster.

The OID is used only to generate the structure object number, but not for the content.

## A summary and what’s next

1. Data in Heap tables can be stored randomly on different pages and different places within the page. If you want to sort the data, use `ORDER BY`.
2. Each table has its own metadata with lots of information.
3. If the table is larger than a gigabyte, then Postgres begins to split it into separate files with a size of 1 gigabyte.
4. PostgreSQL uses a fixed page size, commonly 8 kB. The page structure simultaneously grows from top to bottom and from bottom to top.
5. Each page has a header, transaction pointers, a meta layer for each tuple and for each row.
6. The page has Fillfactor — the ability to store information only up to a certain level.
7. The object identifier is used only to generate the structure object number, but not for the content.

In the next article we’ll look at TOAST tables or so-called satellite tables. They help to split the data in terms of the length of rows.

[![Quadcode](https://miro.medium.com/v2/resize:fill:48:48/1*5q8mdbyZh2rpkIsHDNG8xg.jpeg)](https://medium.com/quadcode-life?source=post_page---post_publication_info--d44c94332052---------------------------------------)

[![Quadcode](https://miro.medium.com/v2/resize:fill:64:64/1*5q8mdbyZh2rpkIsHDNG8xg.jpeg)](https://medium.com/quadcode-life?source=post_page---post_publication_info--d44c94332052---------------------------------------)

[Last published Feb 12, 2025](https://medium.com/quadcode-life/fully-typed-forms-based-on-api-schema-with-react-hook-form-openapi-typescript-and-yup-validation-93ba1321368b?source=post_page---post_publication_info--d44c94332052---------------------------------------)

Quadcode is an international IT company. We develop trading SaaS platform, banking and internal solutions. Over 50 million traders in 150+ countries appreciate the benefits of our platform.

Next level of fintech expertise.