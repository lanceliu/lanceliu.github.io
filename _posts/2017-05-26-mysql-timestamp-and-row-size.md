---
layout: post
title:  "Mysql - 时间戳和row size"
date:   2017-05-26 10:58:52
categories: mysql
published: true
comments: true
thread: 20170526101155555
---
Mysql - TimeStamp 和 Row size
---

# 1. TimeStamp Default Value
```sql
acquisition_time         TIMESTAMP NOT NULL COMMENT '采集日期',
  create_time              TIMESTAMP DEFAULT CURRENT_TIMESTAMP     NOT NULL COMMENT '创建时间',
  update_time              TIMESTAMP DEFAULT '0000-00-00 00:00:00' NOT NULL COMMENT '更新时间',
```
使用timestamp类型创建表时遇到个问题，上面这段ddl在Mysql5.6上没有问题，在mysql5.5上报报错`Incorrect table definition; there can be only one TIMESTAMP column with CURRENT_TIMESTAMP in DEFAULT or ON UPDATE clause`。度娘一下给出如下解释

    Mysql 5.5

    One TIMESTAMP column in a table can have the current timestamp as the default value for initializing the column,
    as the auto-update value, or both.
    It is not possible to have the current timestamp be the default value for one column and the auto-update value for another column.

    Mysql 5.6
    Previously, at most one TIMESTAMP column per table could be automatically initialized or updated to the current date and time.
    This restriction has been lifted.
    Any TIMESTAMP column definition can have any combination of DEFAULT CURRENT_TIMESTAMP and ON UPDATE CURRENT_TIMESTAMP clauses.
    In addition, these clauses now can be used with DATETIME column definitions.
    For more information, see Automatic Initialization and Updating for TIMESTAMP and DATETIME.

翻译过来就是Mysql5.5 不允许 同一个table中，不允许有两个时间戳类型的默认值都为`timestamp`，但从上面的DDL中没有两个相同, 问题在哪里呢？
- Mysql5.5 如果 `timestamp`不指定默认值的话就为 `CURRENT_TIMESTAMP`

但是Mysql5.6中放开了`不允许有两个时间戳类型的默认值都为timestamp`这个限制。

# 2. Row Size
在创建表时，有一个字段为 varchar(10000),由于表中列过多，创建时报错

    Row size too large. The maximum row size for the used table type,
    not counting BLOBs, is 65535.
    You have to change some columns to TEXT or BLOBs

varchar类型支持的长度为65535，但是innodb引擎的mysql每一row会有大小限制，可以
[参考](https://www.percona.com/blog/2011/04/07/innodb-row-size-limitation/).

> Innodb gives you this error when it can’t store all of the variable-length columns for a given row on a single database page.

> Innodb has a limit for the maximum row size, which is slightly less than half a database page (the actual math is 16k-(page header + page trailer)/2. For the default page size of 16kb. this sets an ~8000 bytes limit on row size (8126 as reported by my test installation). This limit is a consequence of InnoDB storing two rows on a page. If you’re using compression, however, you can store a single row on a page.
If your row has variable length columns and the complete row size exceeds this limit, InnoDB chooses variable length columns for off-page storage.

> In these cases, the first 768 bytes of each variable length column is stored locally, and the rest is stored outside of the page (this behavior is version specific, see https://www.percona.com/blog/2010/02/09/blob-storage-in-innodb/ for further reference).

> It’s worth mentioning that this limit applies to the byte-size of values, not the character-size. This means if you insert a 500 character string with all multi-byte characters into a VARCHAR(500) column, that value will also be chosen for off page storage. This means it is possible that you hit this error after converting from a native language character set to utf8, as it can increase the size dramatically.

> If you have more than 10 variable length columns, and each exceeds 768 bytes, then you’ll have at least 8448 bytes for local storage, not counting other fixed length columns. This exceeds the limit and therefore you get the error.

> You don’t have to get it always, as this is not evaluated at table definition, but at row insertion. You may just have a table with 50 variable length columns, but if their max length is, say 50 bytes, you still have plenty of room for local storage.

> The first thing to highlight here is the need for proper testing. If you just insert dummy data in your database, you may be missing to catch important data-dependent bugs like this one.

> Now, how can you work around this problem?

> Here are a few approaches you can take to solve this:
