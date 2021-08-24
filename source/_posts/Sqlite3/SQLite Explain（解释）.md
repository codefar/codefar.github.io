
[TOC]

参考：https://cloud.tencent.com/developer/section/1419999

# SQLite Explain（解释）

可以使用“.eqp on”命令将CLI设置为自动EXPLAIN QUERY PLAN模式


```
.eqp on
```


## 1.1 表和索引扫描

处理SELECT（或其他）语句时，SQLite可以通过各种方式从数据库表中检索数据。它可以扫描表中的所有记录（全表扫描），基于rowid索引扫描表中记录的连续子集，扫描数据库索引中条目的连续子集，或使用组合在单次扫描中的上述策略。SQLite可以从表或索引中检索数据的各种方法在此处详细介绍。


对于查询读取的每个表，EXPLAIN QUERY PLAN的输出包括“详细信息”列中的值以“SCAN”或“SEARCH”开头的记录。“SCAN”用于全表扫描，包括SQLite按索引定义的顺序迭代表中的所有记录的情况。“SEARCH”表示仅访问表行的子集。每个SCAN或SEARCH记录包含以下信息：

- 从中读取表数据的名称。
- 是否使用索引或自动索引。
- 无论是否覆盖索引优化应用。
- WHERE子句的哪些术语用于索引。

例如，以下EXPLAIN QUERY PLAN命令对SELECT语句进行操作，该语句通过对表t1执行全表扫描来实现：

```
sqlite> EXPLAIN QUERY PLAN SELECT a, b FROM t1 WHERE a=1;
QUERY PLAN
`--SCAN TABLE t1
```

上面的示例显示SQLite选择全表扫描将访问表中的所有行。如果查询能够使用索引，则SCAN / SEARCH记录将包括索引的名称，并且对于SEARCH记录，指示如何识别所访问的行的子集。例如：


```
sqlite> CREATE INDEX i1 ON t1(a);
sqlite> EXPLAIN QUERY PLAN SELECT a, b FROM t1 WHERE a=1;
QUERY PLAN
`--SEARCH TABLE t1 USING INDEX i1 (a=?)
```

在前面的例子中，SQLite使用索引“i1”来优化表单的一个WHERE子句（a =？） - 在本例中为“a = 1”。前面的示例无法使用覆盖索引，但以下示例可以，并且该事实反映在输出中：


```
sqlite> CREATE INDEX i2 ON t1(a, b);
sqlite> EXPLAIN QUERY PLAN SELECT a, b FROM t1 WHERE a=1; 
QUERY PLAN
`--SEARCH TABLE t1 USING COVERING INDEX i2 (a=?)
```

SQLite中的所有连接都是使用嵌套扫描实现的。当使用EXPLAIN QUERY PLAN分析以连接为特征的SELECT查询时，将为每个嵌套循环输出一个SCAN或SEARCH记录。例如：


```
sqlite> EXPLAIN QUERY PLAN SELECT t1.*, t2.* FROM t1, t2 WHERE t1.a=1 AND t1.b>2;
QUERY PLAN
|--SEARCH TABLE t1 USING INDEX i2 (a=? AND b>?)
`--SCAN TABLE t2
```

```
sqlite> EXPLAIN QUERY PLAN SELECT t1.*, t2.* FROM t1, t2 WHERE t1.a=1 AND t1.b>2;
0|0|0|SEARCH TABLE t1 USING COVERING INDEX i2 (a=? AND b>?)
0|1|1|SCAN TABLE t2
```

输出的第二列（“order”列）指示嵌套顺序。在这种情况下，使用索引i2扫描表t1是外部循环（顺序= 0），而表t2（order = 1）的全部表扫描是内部循环。

第三列（from“”）表示SELECT语句的FROM子句中与每次扫描相关的表的位置。  
在上面的例子中:  
表t1占据了FROM子句中的第一个位置，所以值列的“from”在第一条记录中为0。  
表t2位于第二个位置，因此相应SCAN记录的“from”列设置为1.在以下示例中，SELECT的FROM子句中的t1和t2的位置相反。查询策略保持不变，但“from”


条目的顺序表示嵌套顺序。在这种情况下，使用索引i2扫描表t1是外循环（因为它首先出现），而表t2的全表扫描是内循环（因为它最后出现）。在以下示例中，SELECT的FROM子句中t1和t2的位置相反。查询策略保持不变。EXPLAIN QUERY PLAN的输出显示了查询的实际评估方式，而不是SQL语句中的指定方式。


```
sqlite> EXPLAIN QUERY PLAN SELECT t1.*, t2.* FROM t2, t1 WHERE t1.a=1 AND t1.b>2;
QUERY PLAN
|--SEARCH TABLE t1 USING INDEX i2 (a=? AND b>?)
`--SCAN TABLE t2
```

如果查询的WHERE子句包含OR表达式，则SQLite可能会使用“OR by union”策略（也称为 OR优化）。在这种情况下，搜索将有单个顶级记录，有两个子记录，每个索引一个：


```
sqlite> CREATE INDEX i3 ON t1(b);
sqlite> EXPLAIN QUERY PLAN SELECT * FROM t1 WHERE a=1 OR b=2;
QUERY PLAN
`--MULTI-INDEX OR
   |--SEARCH TABLE t1 USING COVERING INDEX i2 (a=?)
   `--SEARCH TABLE t1 USING INDEX i3 (b=?)
```

## 1.2 临时排序B树

如果SELECT查询包含ORDER BY，GROUP BY或DISTINCT子句，则SQLite可能需要使用临时b树结构对输出行进行排序。或者，它可能使用索引。使用索引几乎总是比执行排序更有效。如果需要临时b树，则会在EXPLAIN QUERY PLAN输出中添加一条记录，并将“detail”字段设置为“USE TEMP B-TREE FOR xxx”形式的字符串值，其中xxx是“ORDER”之一BY“，”GROUP BY“或”DISTINCT“。例如：


```
sqlite> EXPLAIN QUERY PLAN SELECT c, d FROM t2 ORDER BY c;
QUERY PLAN
|--SCAN TABLE t2
`--USE TEMP B-TREE FOR ORDER BY
```
在这种情况下，可以通过在t2（c）上创建索引来避免使用临时b树，如下所示：


```
sqlite> CREATE INDEX i4 ON t2(c);
sqlite> EXPLAIN QUERY PLAN SELECT c, d FROM t2 ORDER BY c; 
QUERY PLAN
`--SCAN TABLE t2 USING INDEX i4
```


## 1.3 子查询

在上面的所有示例中，第一列（列“selectid”）始终设置为0.如果查询包含子选择（作为FROM子句的一部分或作为SQL表达式的一部分），则EXPLAIN QUERY PLAN还包括每个子选择的报告。每个子选择被分配一个不同的，非零的“选择”值。顶级SELECT语句始终分配有选项ID值0.例如：


```
sqlite> EXPLAIN QUERY PLAN SELECT (SELECT b FROM t1 WHERE a=0), (SELECT a FROM t1 WHERE b=t2.c) FROM t2;
0|0|0|SCAN TABLE t2
0|0|0|EXECUTE SCALAR SUBQUERY 1
1|0|0|SEARCH TABLE t1 USING COVERING INDEX i2 (a=?)
0|0|0|EXECUTE CORRELATED SCALAR SUBQUERY 2
2|0|0|SEARCH TABLE t1 USING INDEX i3 (b=?)
```


在上面的所有示例中，只有一个SELECT语句。如果查询包含子选择，那么它们将显示为外部SELECT的子项。例如：


```
sqlite> EXPLAIN QUERY PLAN SELECT (SELECT b FROM t1 WHERE a=0), (SELECT a FROM t1 WHERE b=t2.c) FROM t2;
|--SCAN TABLE t2 USING COVERING INDEX i4
|--SCALAR SUBQUERY
|  `--SEARCH TABLE t1 USING COVERING INDEX i2 (a=?)
`--CORRELATED SCALAR SUBQUERY
   `--SEARCH TABLE t1 USING INDEX i3 (b=?)
```

上面的示例包含两个“SCALAR”子查询。子查询是SCALAR，因为它们返回单个值 - 一行，一列表。如果实际查询返回的值多于此值，则仅使用第一行的第一列。

上面的第一个子查询相对于外部查询是常量。第一个子查询的值可以计算一次，然后重新用于外部SELECT的每一行。然而，第二个子查询是“相关的”。第二个子查询的值根据外部查询的当前行中的值而更改。因此，必须为外部SELECT中的每个输出行运行一次第二个子查询。

除非应用展平优化，否则如果子查询出现在SELECT语句的FROM子句中，则SQLite可以运行子查询并将结果存储在临时表中，也可以将子查询作为协同例程运行。以下查询是后者的示例。子查询由协同例程运行。只要需要来自子查询的另一行输入，外部查询就会阻塞。控制切换到产生所需输出行的协同程序，然后控制切换回主程序，继续处理。


```
sqlite> EXPLAIN QUERY PLAN SELECT count(*) FROM (SELECT max(b) AS x FROM t1 GROUP BY a) GROUP BY x;
QUERY PLAN
|--CO-ROUTINE 0x20FC3E0
|  `--SCAN TABLE t1 USING COVERING INDEX i2
|--SCAN SUBQUERY 0x20FC3E0
`--USE TEMP B-TREE FOR GROUP BY
```

如果在SELECT语句的FROM子句中的子查询上使用展平优化，则会将子查询有效地合并到外部查询中。EXPLAIN QUERY PLAN的输出反映了这一点，如下例所示：


```
sqlite> EXPLAIN QUERY PLAN SELECT * FROM (SELECT * FROM t2 WHERE c=1), t1;
QUERY PLAN
|--SEARCH TABLE t2 USING INDEX i4 (c=?)
`--SCAN TABLE t1
```

如果可能需要多次访问子查询的内容，则不希望使用协同例程，因为协同例程必须不止一次地计算数据。如果子查询无法展平，则意味着子查询必须表现为临时表。


```
sqlite> SELECT * FROM
      >   (SELECT * FROM t1 WHERE a=1 ORDER BY b LIMIT 2) AS x,
      >   (SELECT * FROM t2 WHERE c=1 ORDER BY d LIMIT 2) AS y;
QUERY PLAN
|--MATERIALIZE 0x18F06F0
|  `--SEARCH TABLE t1 USING COVERING INDEX i2 (a=?)
|--MATERIALIZE 0x18F80D0
|  |--SEARCH TABLE t2 USING INDEX i4 (c=?)
|  `--USE TEMP B-TREE FOR ORDER BY
|--SCAN SUBQUERY 0x18F06F0 AS x
`--SCAN SUBQUERY 0x18F80D0 AS y
```

## 1.4 复合查询

复合查询（UNION，UNION ALL，EXCEPT或INTERSECT）的 每个组件查询都是单独计算的，并在EXPLAIN QUERY PLAN输出中给出了自己的行。


```
sqlite> EXPLAIN QUERY PLAN SELECT a FROM t1 UNION SELECT c FROM t2;
QUERY PLAN
`--COMPOUND QUERY
   |--LEFT-MOST SUBQUERY
   |  `--SCAN TABLE t1 USING COVERING INDEX i1
   `--UNION USING TEMP B-TREE
      `--SCAN TABLE t2 USING COVERING INDEX i4
```


上述输出中的“USING TEMP B-TREE”子句表示临时b树结构用于实现两个子选择结果的UNION。计算复合的另一种方法是将每个子查询作为协同例程运行，安排它们的输出按排序顺序出现，并将结果合并在一起。当查询计划程序选择后一种方法时，EXPLAIN QUERY PLAN输出如下所示：


```
sqlite> EXPLAIN QUERY PLAN SELECT a FROM t1 EXCEPT SELECT d FROM t2 ORDER BY 1;
QUERY PLAN
`--MERGE (EXCEPT)
   |--LEFT
   |  `--SCAN TABLE t1 USING COVERING INDEX i1
   `--RIGHT
      |--SCAN TABLE t2
      `--USE TEMP B-TREE FOR ORDER BY
```

