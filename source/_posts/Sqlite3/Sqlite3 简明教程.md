# Sqlite3 简明教程

[TOC]

# 命令行基础命令


```
.help：
这个命令可以查看sqlite3的一些命令，读者可以自己运行一下看看。


```


```
sqlite> .help
.auth ON|OFF           Show authorizer callbacks
.backup ?DB? FILE      Backup DB (default "main") to FILE
.bail on|off           Stop after hitting an error.  Default OFF
.binary on|off         Turn binary output on or off.  Default OFF
.changes on|off        Show number of rows changed by SQL
.check GLOB            Fail if output since .testcase does not match
.clone NEWDB           Clone data into NEWDB from the existing database
.databases             List names and files of attached databases
.dbinfo ?DB?           Show status information about the database
.dump ?TABLE? ...      Dump the database in an SQL text format
                         If TABLE specified, only dump tables matching
                         LIKE pattern TABLE.
.echo on|off           Turn command echo on or off
.eqp on|off|full       Enable or disable automatic EXPLAIN QUERY PLAN
.exit                  Exit this program
.explain ?on|off|auto? Turn EXPLAIN output mode on or off or to automatic
.fullschema ?--indent? Show schema and the content of sqlite_stat tables
.headers on|off        Turn display of headers on or off
.help                  Show this message
.import FILE TABLE     Import data from FILE into TABLE
.imposter INDEX TABLE  Create imposter table TABLE on index INDEX
.indexes ?TABLE?       Show names of all indexes
                         If TABLE specified, only show indexes for tables
                         matching LIKE pattern TABLE.
.limit ?LIMIT? ?VAL?   Display or change the value of an SQLITE_LIMIT
.log FILE|off          Turn logging on or off.  FILE can be stderr/stdout
.mode MODE ?TABLE?     Set output mode where MODE is one of:
                         ascii    Columns/rows delimited by 0x1F and 0x1E
                         csv      Comma-separated values
                         column   Left-aligned columns.  (See .width)
                         html     HTML <table> code
                         insert   SQL insert statements for TABLE
                         line     One value per line
                         list     Values delimited by .separator strings
                         quote    Escape answers as for SQL
                         tabs     Tab-separated values
                         tcl      TCL list elements
.nullvalue STRING      Use STRING in place of NULL values
.once FILENAME         Output for the next SQL command only to FILENAME
.open ?--new? ?FILE?   Close existing database and reopen FILE
                         The --new starts with an empty file
.output ?FILENAME?     Send output to FILENAME or stdout
.print STRING...       Print literal STRING
.prompt MAIN CONTINUE  Replace the standard prompts
.quit                  Exit this program
.read FILENAME         Execute SQL in FILENAME
.restore ?DB? FILE     Restore content of DB (default "main") from FILE
.save FILE             Write in-memory database into FILE
.scanstats on|off      Turn sqlite3_stmt_scanstatus() metrics on or off
.schema ?PATTERN?      Show the CREATE statements matching PATTERN
                          Add --indent for pretty-printing
.separator COL ?ROW?   Change the column separator and optionally the row
                         separator for both the output mode and .import
.shell CMD ARGS...     Run CMD ARGS... in a system shell
.show                  Show the current values for various settings
.stats ?on|off?        Show stats or turn stats on or off
.system CMD ARGS...    Run CMD ARGS... in a system shell
.tables ?TABLE?        List names of tables
                         If TABLE specified, only list tables matching
                         LIKE pattern TABLE.
.testcase NAME         Begin redirecting output to 'testcase-out.txt'
.timeout MS            Try opening locked tables for MS milliseconds
.timer on|off          Turn SQL timer on or off
.trace FILE|off        Output each SQL statement as it is run
.vfsinfo ?AUX?         Information about the top-level VFS
.vfslist               List all available VFSes
.vfsname ?AUX?         Print the name of the VFS stack
.width NUM1 NUM2 ...   Set column widths for "column" mode
                         Negative values right-justify
```




```
.show：
　　.show 用来显示sqlite3中的一些设置：

sqlite> .show
        echo: off
         eqp: off
     explain: auto
     headers: on
        mode: column
   nullvalue: ""
      output: stdout
colseparator: "|"
rowseparator: "\n"
       stats: off
       width:
```
# SQLite数据类型

SQLite数据库中存储的值是以下存储类之一：


<html>
<table>
<thead>
<tr>
<th>存储类</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>NULL</td>
<td>表示值为空(<code>null</code>)值。</td>
</tr>
<tr>
<td>INTEGER</td>
<td>表示值是一个有符号整数，根据值的大小存储在<code>1</code>,<code>2</code>,<code>3</code>,<code>4</code>,<code>6</code>或<code>8</code>个字节中。</td>
</tr>
<tr>
<td>REAL</td>
<td>表示值是一个浮点值，存储为<code>8</code>位IEEE浮点数。</td>
</tr>
<tr>
<td>TEXT</td>
<td>表示值是一个文本字符串，使用数据库编码(<code>utf-8</code>，<code>utf-16be</code>或utf-16le)存储</td>
</tr>
<tr>
<td>BLOB</td>
<td>表示值是一个数据块，与输入的数据完全相同。</td>
</tr>
</tbody>
</table>
</html>


#  SQLite 常用子句
1. ORDER BY   
SQLite 的  子句是用来基于一个或多个列按升序或降序顺序排列数据。


```
语法：
　　ASC：表示升序，默认就是升序（可以不写）
　　DESX：表示降序

SELECT column-list 
FROM table_name 
[WHERE condition] 
[ORDER BY column1, column2, .. columnN] [ASC | DESC];

```

```
select * from student order by AGE DESC;
```

2. GROUP BY  
SQLite 的GROUP BY 子句用于与 SELECT 语句一起使用，来对相同的数据进行分组。在 SELECT语句中，GROUP BY 子句放在 WHERE 子句之后，放在 ORDER BY 子句之前。


```
select NAME,  count(NAME) from student group by NAME;
```

3. Having   
SQLite 的having 子句使用指定条件来过滤将出现在最终结果中的分组结果。
在 WHERE 子句在所选列上设置条件，而 HAVING 子句则在由 GROUP BY 子句创建的分组上设置条件。


```
select NAME,  count(NAME) from student group by NAME having count(NAME) > 1;
```

4. Distinct  
SQLite 的distinct 与 SELECT 语句一起使用，来消除所有重复的记录，并只获取唯一一次记录。有可能出现一种情况，在一个表中有多个重复的记录。当提取这样的记录时，DISTINCT 关键字就显得特别有意义，它只获取唯一一次记录，而不是获取重复记录。

```
select distinct NAME from student;
```


#  SQLite进阶-约束、连接、索引
## 约束

约束是在表的数据列上强制执行的规则。可以用来限制可以插入到表中的数据类型。
　　这确保了数据库中数据的准确性和可靠性。
　　约束可以是列级约束或表级约束。列级约束适用于列，表级约束应用到表。
　　
1. NOT NULL 约束：确保某列不能有 NULL 值。
2. DEFAULT 约束：当某列没有指定值时，为该列提供默认值。
3. UNIQUE 约束：确保某列中的所有值是不同的。
4. PRIMARY Key 约束：唯一标识数据库表中的各行/记录。
5. CHECK 约束：CHECK 约束确保某列中的所有值满足一定条件。


NOT NULL 约束：  
默认情况下，列可以保存 NULL 值。如果您不想某列有 NULL 值，那么需要在该列上定义此约束，指定在该列上不允许 NULL 值。
NULL 与没有数据是不一样的，它代表着未知的数据。

DEFAULT 约束：  
DEFAULT 约束在 INSERT INTO 语句没有提供一个特定的值时，为列提供一个默认值。

UNIQUE 约束：  
UNIQUE 约束防止在特定的列存在两个记录具有相同的值。比如，我们可以让NAME不存在相同的。

CHECK 约束：  
CHECK 约束启用输入一条记录要检查值的条件。如果条件值为 false，则记录违反了约束，且不能输入到表。

PRIMARY KEY 约束：  
PRIMARY KEY（主键） 约束唯一标识数据库表中的每个记录。
在一个表中可以有多个UNIQUE 列，但只能有一个主键。在设计数据库表时，主键是很重要的。主键是唯一的 ID。我们使用主键来引用表中的行。可通过把主键设置为其他表的外键，来创建表之间的关系。由于"长期存在编码监督"，在 SQLite 中，主键可以是 NULL，这是与其他数据库不同的地方。  
　　主键是表中的一个字段，唯一标识数据库表中的各行/记录。主键必须包含唯一值。主键列不能有NULL值。  
　　一个表只能有一个主键，它可以由一个或多个字段组成。当多个字段作为主键，它们被称为复合键。  
　　如果一个表在任何字段上定义了一个主键，那么在这些字段上不能有两个记录具有相同的值。
　　

```
create table bound_test( 
ID int primary key not NULL,  --主键约束
NAME TEXT NOT NULL,--not null 约束
AGE INT CHECK(AGE>0), --CHECK约束，AGE要大于0
WEIGHT INT UNIQUE, --UNIQUE 约束，体重不能相等（虽然不恰当哈）
ADDRESS CHAR(50) DEFAULT 'shenzhen' --DEFAULT约束，默认为深圳
);
```

## 连接操作 JOIN

https://zh.wikipedia.org/wiki/%E8%BF%9E%E6%8E%A5_(SQL)#.E5.85.A8.E8.BF.9E.E6.8E.A5

在SQLite中，JOIN子句用于组合数据库中两个或多个表的记录。 它通过使用两个表的公共值来组合来自两个表的字段。

SQLite中主要有三种类型的连接：

1. SQLite内连接
1. SQLite外连接
1. SQLite交叉连接

### 内连接

> 内连接(inner join)是应用程序中用的普遍的"连接"操作，它一般都是默认连接类型。内连接基于连接谓词将两张表(如 A 和 B)的列组合在一起，产生新的结果表。查询会将 A 表的每一行和 B 表的每一行进行比较，并找出满足连接谓词的组合。当连接谓词被满足，A 和 B 中匹配的行会按列组合(并排组合)成结果集中的一行。连接产生的结果集，可以定义为首先对两张表做笛卡尔积(交叉连接) -- 将 A 中的每一行和 B 中的每一行组合，然后返回满足连接谓词的记录。实际上 SQL 产品会尽可能用其他方式去实现连接，笛卡尔积运算是非常没效率的.

SQLite内连接(inner join)是最常见的连接类型。 它用于组合满足连接条件的多个表中的所有行记录。

SQlite内连接是默认的连接类型。

![image](http://www.yiibai.com/uploads/images/201705/2405/368140526_39492.png)

### 外连接

> 外连接并不要求连接的两表的每一条记录在对方表中都一条匹配的记录。要保留所有记录（甚至这条记录没有匹配的记录也要保留）的表称为保留表。 外连接可依据连接表保留左表, 右表或全部表的行而进一步分为左外连接, 右外连接和全连接.

在SQL标准中，有三种类型的外连接：

- 左外连接
- 右外连接
- 全外连接  

但是，**SQLite仅支持左外连接**。

![image](http://www.yiibai.com/uploads/images/201705/2405/231140534_74891.png)


```
SELECT EMP_ID, NAME, DEPT FROM STUDENT LEFT OUTER JOIN DEPARTMENT  
ON STUDENT.ID = DEPARTMENT.EMP_ID;
```

### 交叉连接

> 交叉连接(cross join)，又称笛卡尔连接(cartesian join)或叉乘(Product)，它**是所有类型的内连接的基础**。把表视为行记录的集合，交叉连接即返回这两个集合的笛卡尔积。这其实等价于内连接的链接条件为"永真"，或连接条件不存在.
> 
> 如果 A 和 B 是两个集合，它们的交叉连接就记为: A × B.
> 用于交叉连接的 SQL 代码在 FROM 列出表名，但并**不包含任何过滤的连接谓词**.

SQLite 交叉连接用于将第一个表的每一行与第二个表的每一行进行匹配。 如果第一个表包含x列，而第二个表包含y列，则所得到的交叉连接表的结果将包含x * y列。

![image](http://www.yiibai.com/uploads/images/201705/2405/410140541_80428.png)


**SQL中过滤条件放在on和where中的区别**  

首先两个表做一个笛卡尔积 ，  
ON后面的条件是对这个笛卡尔积做一个过滤形成一张临时表，    
如果没有where就直接返回结果，如果有where就对上一步的临时表再进行过滤。

在CROSS JOIN和INNER JOIN中， ON和where是没区别的，右连接和左连接就不一样了。


# SQLite触发器

SQLite触发器是一种事件驱动的动作或数据库回调函数，它在对指定的表执行INSERT，UPDATE和DELETE语句时自动调用。

## 如何创建触发器？

CREATE TRIGGER语句用于在SQLite中创建一个新的触发器。 此语句也用于向数据库模式添加触发器。

语法

```
CREATE  TRIGGER trigger_name [BEFORE|AFTER] event_name   
ON table_name  
BEGIN  
 -- Trigger logic goes here....  
END;
```

trigger_name是要创建的触发器的名称。  
event_name可以是**INSERT，DELETE和UPDATE**数据库操作。   
table_name是要进行操作的表。


```
CREATE TRIGGER audit_log AFTER INSERT   
ON COMPANY  
BEGIN  
INSERT INTO AUDIT(EMP_ID, ACTION_TYPE ,ENTRY_DATE) VALUES (new.ID, 'AFTER INSERT',datetime('now'));  
END;
```

```
CREATE TRIGGER befor_ins BEFORE INSERT   
ON COMPANY  
BEGIN  
INSERT INTO AUDIT(EMP_ID, ACTION_TYPE ,ENTRY_DATE) VALUES (new.ID, 'BEFORE INSERT', datetime('now'));  
END;
```

demo

```
CREATE TABLE company(  
   ID INT PRIMARY KEY     NOT NULL,  
   NAME           TEXT    NOT NULL,  
   AGE            INT     NOT NULL,  
   ADDRESS        CHAR(50),  
   SALARY         REAL  
);

CREATE TABLE audit(  
    EMP_ID INT NOT NULL,
    ACTION_TYPE TEXT NOT NULL,
    ENTRY_DATE TEXT NOT NULL  
);

CREATE TRIGGER audit_log AFTER INSERT   
ON COMPANY  
BEGIN  
INSERT INTO AUDIT(EMP_ID, ACTION_TYPE ,ENTRY_DATE) VALUES (new.ID, 'AFTER INSERT',datetime('now'));  
END;

CREATE TRIGGER befor_ins BEFORE INSERT   
ON COMPANY  
BEGIN  
INSERT INTO AUDIT(EMP_ID, ACTION_TYPE ,ENTRY_DATE) VALUES (new.ID, 'BEFORE INSERT', datetime('now'));  
END;

INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)  
VALUES (1, 'Maxsu', 22, 'Haikou', 40000.00);

INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)  
VALUES (2, 'Minsu', 28, 'Guangzhou', 35000.00);

```


## 如何列出/查看触发器？

可以使用查询语句从sqlite_master表中来查询列出/查看触发器。

```
SELECT name FROM sqlite_master  
WHERE type = 'trigger';
```

# 高级主题

## SQLite Explain（解释）

在 SQLite 语句之前，可以使用 "EXPLAIN" 关键字或 "EXPLAIN QUERY PLAN" 短语，用于描述表的细节。

如果省略了 EXPLAIN 关键字或短语，任何的修改都会引起 SQLite 语句的查询行为，并返回有关 SQLite 语句如何操作的信息。

- 来自 EXPLAIN 和 EXPLAIN QUERY PLAN 的输出只用于交互式分析和排除故障。
- 输出格式的细节可能会随着 SQLite 版本的不同而有所变化。
- 应用程序不应该使用 EXPLAIN 或 EXPLAIN QUERY PLAN，因为其确切的行为是可变的且只有部分会被记录。

语法
EXPLAIN 的语法如下：

```
EXPLAIN [SQLite Query]
```

EXPLAIN QUERY PLAN 的语法如下：

```
EXPLAIN  QUERY PLAN [SQLite Query]
```

## SQLite PRAGMA

http://www.runoob.com/sqlite/sqlite-pragma.html


```
.eqp on|off|full       Enable or disable automatic EXPLAIN QUERY PLAN
```


SQLite 的 PRAGMA 命令是一个特殊的命令，可以用在 SQLite 环境内控制各种环境变量和状态标志。一个 PRAGMA 值可以被读取，也可以根据需求进行设置。

### 语法

要查询当前的 PRAGMA 值，只需要提供该 pragma 的名字：


```
PRAGMA pragma_name;
```

要为 PRAGMA 设置一个新的值，语法如下：


```
PRAGMA pragma_name = value;
```

设置模式，可以是名称或等值的整数，但返回的值将始终是一个整数。


### auto_vacuum Pragma

auto_vacuum Pragma 获取或设置 auto-vacuum 模式。语法如下：


```
PRAGMA [database.]auto_vacuum;
PRAGMA [database.]auto_vacuum = mode;
```


```
sqlite> .header on
sqlite> EXPLAIN QUERY PLAN SELECT * FROM company;
selectid|order|from|detail
0|0|0|SCAN TABLE company

```

EXPLAIN列的解释：

table：显示这一行的数据是关于哪张表的.

type：这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL

possible_keys：显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句。这个列表是在优化过程的早期创建的，因此有些列出来的索引可能对于后续优化过程是没有用的。

key： 实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE
INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引

