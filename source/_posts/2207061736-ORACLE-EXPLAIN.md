---
title: 2207061736@ORACLE-EXPLAIN
categories:
  - null
tags:
  - null
date: 2022-07-06 17:36:52
---
在PL/SQL中 执行explain plain for  select ...

查看Description 数据参考：

Oracle常见的执行计划步骤

这里我们介绍一些常见的执行计划中的步骤及算法。

## 1、表访问路径

### (1)TABLE ACCESS

FULL：全表扫描。它会访问表中的每一条记录(读取高水位线以内的每一个数据块)。

CLUSTER：通过索引簇的键来访问表。

BY INDEX ROWID：通过指定ROWID来访问表中的单条记录。ROWID是访问记录的最快方式，通常由索引访问得到。

BY USER ROWID：提供一个绑定变量、字面变量或WHERE CURRENT OF CURSOR子句来通过ROWID进行访问。

BY GLOBAL INDEX ROWID：通过由全局分区索引来获得ROWID，然后进行表访问。该访问出现在分区表中。

BY LOCAL INDEX ROWID：通过由局部分区索引获得ROWID，然后进行表访问。该访问出现在分区表中。

### (2)EXTERNAL TABLE ACCESS：访问外部表。

### (3)RESULT CACHE：这个SQL结果集可能来自结果集缓存。

### (4)MAT_VIEW REWRITE ACCESS：SQL被重写以利用物化视图。

## 2、索引操作

### (1)AND-EQUAL：合并来自一个或多个索引的结果集。

### (2)INDEX

UNIQUE SCAN：只返回一条记录的地址(ROWID)的索引扫描。

RANGE SCAN：返回多条记录的ROWID的索引检索。一般出现这样的访问，是因为出现了区间操作符。

FULL SCAN：按照索引键的顺序扫描整个索引。

SKIP SCAN：组合索引键中非前导列索引检索。

FULL SCAN(MAX/MIN)：检索索引中的最高或最低的索引条目。

FAST FULL SCAN：按照块顺序扫描每一个索引的条目，可能会使用多块读取。

### (3)DOMAIN INDEX：应用域索引。

## 3、位图索引操作

### (1)BITMAP

CONVERSION：将位转换为ROWID或相反。

INDEX：从位图中撮一个值或一个范围的值。

MERGE：合并位图。

MINUS：从一个位图减去另一个位图。

OR：对两个位图进行OR操作。

## 4、表连接操作

### (1)CONNECT BY：对前一个步骤的输出结果执行一个层次化的自连接操作。

### (2)MERGE JOIN：对前一个步骤的输出结果执行一个合并连接。

### (3)NESTED LOOP：对前一个步骤执行嵌套循环连接。对上一层结果集的每一行，都会扫描下一层结果集以找到匹配记录。

### (4)HASH JOIN：对两个记录进行散列连接。

### (5)任意连接操作

OUTER：外连接

ANTI：反连接

SEMI：反连接

CARTESIAN：一个结果集中的每一条记录都与另一个结果集中的每一条记录进行连接。

## 5、集合操作

CONCATENATION：与显式指定一个UNION子句一样，多个结果集被按照同样的方式做合并。它通常发生在对索引列使用OR运算时。

INTERSECTION：对两个结果进行比较，只返回两个结果集中都存在的记录。

MINUS：除了在第二个结果集中出现的记录外，返回第一个结果集中的所有记录。

UNION-ALL：对两个结果集进行合并，并返回两个结果集中的所有记录。

UNION：与UNION-ALL相同，但是它不返回重复记录。

VIEW：访问一个视图定义或或创建一个临时表用于存放结果集。

## 6、分区操作

### (1)PARTITION

SINGLE：访问单个分区。

ITERATOR：访问多个分区。

ALL：访问所有分区。

INLIST：基于IN列表中的值来访问分区。

## 7、汇总操作

### (1)COUNT：使用COUNT函数进计算。

STOPKEY：计算结果中的记录，当达到一定数量时，就停止计算。这通常发生在使用了WHERE子句且指定了ROWNUM，如：WHERE ROWNUM <= 10。

### (2)BUFFER SORT：对临时结果集做一次内在排序。

### (3)HASH GROUP BY：使用散列进行分组操作。

### (4)INLIST ITERATOR：对IN中的每一个值都实现一次子操作。

### (5)SORT

GROUP BY：为了满足GROUP BY而对结果集进行排序。

AGGREGATE：当在已分组好的数据上使用分组函数时，会出现此操作。

JOIN：了进行合并连接而对记录进行排序。

UNIQUE：排除重复记录的排序操作，通常是使用DISTINCT子句。

GROUP BY：为满足GROUP BY子句，对结果集进行排序分组。

## 8、其他操作

### (1)FOR UPDATE：使用了FOR UPDATE子句。

### (2)COLLECTION ITERATOR：使用了表函数提取记录。

### (3)FAST DUAL：访问DUAL表。

### (4)FILTER：从结果集中排除掉不匹配选取条件的记录。

### (5)REMOTE：通过数据库连接，访问一个外部的数据库。

### (6)FIRST ROW：获取查询的第一条记录。

### (7)SEQUENCE：使用了Oracle序列。

### (8)LOAD AS SELECT：使用SELECT进行直接路径的INSERT操作。

### (9)FIXED TABLE：访问固定的(X$/V$)表。

### (10)FIXED INDEX：访问固定的索引。

### (11)WINDOW BUFFER：支持分析函数的内部操作。