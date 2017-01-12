---
layout: post
title:  "sql执行计划阅读"
date:   2017-01-11 10:58:52
categories: mysql sql
published: true
comments: true
thread: 20170111101155555
---
sql执行计划阅读
---

能写SQL和写好SQL是初级／高级程序员的界定点。书写SQL时，可以通过Explain命令来辅助写出高性能SQL。

## Explain语法
EXPLAIN  SELECT ……
变体：
1. EXPLAIN EXTENDED SELECT ……
将执行计划“反编译”成SELECT语句，运行SHOW WARNINGS 可得到被MySQL优化器优化后的查询语句
2. EXPLAIN PARTITIONS SELECT ……
用于分区表的EXPLAIN

```sql
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
```

### id
包含一组数字，表示查询中执行select子句或操作表的顺序

- i. id相同，执行顺序由上至下

```sql
mysql> explain SELECT * FROM hr_user, hr_organization;
+----+-------------+-----------------+------+---------------+------+---------+------+------+---------------------------------------+
| id | select_type | table           | type | possible_keys | key  | key_len | ref  | rows | Extra                                 |
+----+-------------+-----------------+------+---------------+------+---------+------+------+---------------------------------------+
|  1 | SIMPLE      | hr_organization | ALL  | NULL          | NULL | NULL    | NULL |   36 | NULL                                  |
|  1 | SIMPLE      | hr_user         | ALL  | NULL          | NULL | NULL    | NULL |  517 | Using join buffer (Block Nested Loop) |
+----+-------------+-----------------+------+---------------+------+---------+------+------+---------------------------------------+
```

- ii. 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
- iii. id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行
- iv. 如果是UNION完毕后的结果时，id=null

```sql
mysql> explain SELECT *
-> FROM hr_loginhistory
-> WHERE uid IN (SELECT id
->               FROM hr_user
->               WHERE hr_user.organizationId = (SELECT id from hr_organization where companyName like '云图信息%'));
+----+-------------+-----------------+--------+---------------+---------+---------+-----------------------------------+------+-------------+
| id | select_type | table           | type   | possible_keys | key     | key_len | ref                               | rows | Extra       |
+----+-------------+-----------------+--------+---------------+---------+---------+-----------------------------------+------+-------------+
|  1 | PRIMARY     | hr_loginhistory | ALL    | NULL          | NULL    | NULL    | NULL                              | 9398 | Using where |
|  1 | PRIMARY     | hr_user         | eq_ref | PRIMARY,Id    | PRIMARY | 8       | yuntujinfu_ll.hr_loginhistory.Uid |    1 | Using where |
|  3 | SUBQUERY    | hr_organization | ALL    | NULL          | NULL    | NULL    | NULL                              |   36 | Using where |
+----+-------------+-----------------+--------+---------------+---------+---------+-----------------------------------+------+-------------+
```

### select_type

```sql
+-------------+-
| select_type |
+-------------+-
| SIMPLE      | 简单SELECT(不使用UNION或子查询等)
+-------------+
| PRIMARY     | 最外面的SELECT
+-------------+
| UNION       | UNION中的第二个或后面的SELECT语句
+-------------+
| DEPENDENT UNION| UNION中的第二个或后面的SELECT语句，取决于外面的查询
+-------------+
| UNION RESULT | UNION的结果。
+-------------+
| SUBQUERY    | 子查询中的第一个SELECT
+-------------+
| DEPENDENT SUBQUERY | 子查询中的第一个SELECT，取决于外面的查询
+-------------+
| DERIVED      | 导出表的SELECT(FROM子句的子查询)
+-------------+
```

- UNION && UNION RESULT && PRIMARY

```sql
mysql> explain (select id from hr_user A WHERE A.DelFlag=1) UNION (SELECT id FROM hr_user B WHERE B.DelFlag=0);
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
| id | select_type  | table      | type | possible_keys | key  | key_len | ref  | rows | Extra           |
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
|  1 | PRIMARY      | A          | ALL  | NULL          | NULL | NULL    | NULL |  517 | Using where     |
|  2 | UNION        | B          | ALL  | NULL          | NULL | NULL    | NULL |  517 | Using where     |
| NULL | UNION RESULT | <union1,2> | ALL  | NULL          | NULL | NULL    | NULL | NULL | Using temporary |
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
```

- DEPENDENT SUBQUERY

```sql
mysql> explain SELECT id
    -> FROM hr_user B
    -> WHERE B.DelFlag = 0 AND exists(SELECT *
    ->                                FROM hr_loginhistory WHERE hr_loginhistory.uid=B.id);
+----+--------------------+-----------------+------+---------------+------+---------+------+------+-------------+
| id | select_type        | table           | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+--------------------+-----------------+------+---------------+------+---------+------+------+-------------+
|  1 | PRIMARY            | B               | ALL  | NULL          | NULL | NULL    | NULL |  517 | Using where |
|  2 | DEPENDENT SUBQUERY | hr_loginhistory | ALL  | NULL          | NULL | NULL    | NULL | 9398 | Using where |
+----+--------------------+-----------------+------+---------------+------+---------+------+------+-------------+
```

- SUBQUERY

```sql
mysql> explain SELECT id, LoginName, (SELECT companyName from hr_organization where hr_organization.id=1) FROM hr_user;
+----+-------------+---------+------+---------------+------+---------+------+------+-----------------------------------------------------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows | Extra                                               |
+----+-------------+---------+------+---------------+------+---------+------+------+-----------------------------------------------------+
|  1 | PRIMARY     | hr_user | ALL  | NULL          | NULL | NULL    | NULL |  517 | NULL                                                |
|  2 | SUBQUERY    | NULL    | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE noticed after reading const tables |
+----+-------------+---------+------+---------------+------+---------+------+------+-----------------------------------------------------+
```

- DERIVED

```sql
mysql> explain SELECT * FROM (SELECT id FROM  hr_organization A WHERE addDate > '2015-01-01') B
    -> ;
+----+-------------+------------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table      | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+------------+------+---------------+------+---------+------+------+-------------+
|  1 | PRIMARY     | <derived2> | ALL  | NULL          | NULL | NULL    | NULL |   36 | NULL        |
|  2 | DERIVED     | A          | ALL  | NULL          | NULL | NULL    | NULL |   36 | Using where |
+----+-------------+------------+------+---------------+------+---------+------+------+-------------+
```

### table
显示这一行的数据是关于哪张表的

### type
显示了连接使用了哪种类别,有无使用索引，是使用Explain命令分析性能瓶颈的关键项之一

结果值从好到坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL


### possible_keys
列指出MySQL能使用哪个索引在该表中找到行

### key
显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL

### key_len
显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好

### ref
显示使用哪个列或常数与key一起从表中选择行。

### rows
显示MySQL认为它执行查询时必须检查的行数。

### Extra
包含MySQL解决查询的详细信息，也是关键参考项之一


    ```
    Distinct
    一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

    Not exists
    MYSQL 优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，

    就不再搜索了

    Range checked for each

    Record（index map:#）
    没有找到理想的索引，因此对于从前面表中来的每一 个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

    Using filesort
    看 到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来 排序全部行

    Using index
    列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表 的全部的请求列都是同一个索引的部分的时候

    Using temporary
    看到这个的时候，查询需要优化了。这 里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

    Using where
    使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index， 这就会发生，或者是查询有问题
    ```

[参考](http://www.cnblogs.com/hailexuexi/archive/2011/11/20/2256020.html)
