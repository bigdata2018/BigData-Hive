# Hive 常用 DDL 

<nav>
<a href="#一、创建数据库">一、创建数据库</a><br/>
<a href="#二、查询数据库">二、查询数据库</a><br/>
<a href="#三、修改数据库">三、修改数据库</a><br/>
    <a href="#四、删除数据库">四、删除数据库</a><br/>
    <a href="#五、创建表">五、创建表</a><br/>
    <a href="#六、修改表">六、修改表</a><br/>
    <a href="#七、删除表">七、删除表</a><br/>
</nav>




## 一、创建数据库

```
CREATE DATABASE [IF NOT EXISTS] database_name
[COMMENT database_comment]
[LOCATION hdfs_path]
[WITH DBPROPERTIES (property_name=property_value, ...)];
```

**1.创建一个数据库，数据库在HDFS上的默认存储路径是/user/hive/warehouse/\*.db。**

```
hive (default)> create database db_hive;
```

**2.避免要创建的数据库已经存在错误，增加if not exists判断。（标准写法）**

```
hive (default)> create database db_hive;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Database db_hive already exists
hive (default)> create database if not exists db_hive;
```

**3.创建一个数据库，指定数据库在HDFS上存放的位置**

```
hive (default)> create database db_hive2 location '/db_hive2.db';
```



## 二、查询数据库

### 2.1 显示数据库

**1.显示数据库**

```
hive> show databases;
```

**2.过滤显示查询的数据库**

```
hive> show databases like 'db_hive*';
OK
db_hive
db_hive_1
```



### 2.2 查看数据库详情

**1.显示数据库信息**

```
hive> desc database db_hive;
OK
db_hive		hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db	atguiguUSER	
```

**2.显示数据库详细信息，extended**

```
hive> desc database extended db_hive;
OK
db_hive		hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db	atguiguUSER	
```



### 2.3 切换当前数据库

```
hive (default)> use db_hive;
```



## 三、修改数据库

用户可以使用ALTER DATABASE命令为某个数据库的DBPROPERTIES设置键-值对属性值，来描述这个数据库的属性信息。数据库的其他元数据信息都是不可更改的，包括数据库名和数据库所在的目录位置。

```
hive (default)> alter database db_hive set dbproperties('createtime'='20170830');
```

在hive中查看修改结果

```
hive> desc database extended db_hive;
db_name comment location        owner_name      owner_type      parameters
db_hive         hdfs://hadoop102:8020/user/hive/warehouse/db_hive.db    atguigu USER    {createtime=20170830}
```



## 四、删除数据库

**1.删除空数据库**

```
hive>drop database db_hive2;
```

**2.如果删除的数据库不存在，最好采用 if exists判断数据库是否存在**

```
hive> drop database db_hive;
FAILED: SemanticException [Error 10072]: Database does not exist: db_hive
hive> drop database if exists db_hive2;
```

**3.如果数据库不为空，可以采用cascade命令，强制删除**

```
hive> drop database db_hive;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database db_hive is not empty. One or more tables exist.)
hive> drop database db_hive cascade;
```



## 五、创建表

**1.建表语法**

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] –指定字段名 字段类型【字段描述】
[COMMENT table_comment] –指定对表的描述
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] –分区表
[CLUSTERED BY (col_name, col_name, ...) –分桶表
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] –指定桶内排序字段（几乎不用），及分几个桶
[ROW FORMAT row_format] –-指定表字段，数据的格式
[STORED AS file_format] –-指定表数据的存储格式
[LOCATION hdfs_path] —-指定表在HDFS的映射目录
[TBLPROPERTIES (property_name=property_value, ...)] –指定表的属性
[AS select_statement] –通过查询语句建表
```

**2.字段解释说明** 

（1）CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。

（2）EXTERNAL关键字可以让用户创建一个外部表，在建表的同时可以指定一个指向实际数据的路径（LOCATION），在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。

（3）COMMENT：为表和列添加注释。

（4）PARTITIONED BY创建分区表

（5）CLUSTERED BY创建分桶表

（6）SORTED BY不常用，对桶中的一个或多个列另外排序

（7）ROW FORMAT 

DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char]

​    [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char] 

  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]

用户在建表的时候可以自定义SerDe或者使用自带的SerDe。如果没有指定ROW FORMAT 或者ROW FORMAT DELIMITED，将会使用自带的SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的SerDe，Hive通过SerDe确定表的具体的列的数据。

SerDe是Serialize/Deserilize的简称， hive使用Serde进行行对象的序列与反序列化。

（8）STORED AS指定存储文件类型

常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件）

如果文件数据是纯文本，可以使用STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。

（9）LOCATION ：指定表在HDFS上的存储位置。

（10）AS：后跟查询语句，根据查询结果创建表。

（11）LIKE允许用户复制现有的表结构，但是不复制数据。



### 5.1 管理表

**1.理论**

默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive会（或多或少地）控制着数据的生命周期。Hive默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。 

 当我们删除一个管理表时，Hive也会删除这个表中数据。管理表不适合和其他工具共享数据。

**2.案例实操**

（1）原始数据

```
1001	ss1
1002	ss2
1003	ss3
1004	ss4
1005	ss5
1006	ss6
1007	ss7
1008	ss8
1009	ss9
1010	ss10
1011	ss11
1012	ss12
1013	ss13
1014	ss14
1015	ss15
1016	ss16
```

（2）普通创建表

```
create table if not exists student(
id int, name string
)
row format delimited fields terminated by '\t'
stored as textfile
location '/user/hive/warehouse/student';


或
create table if not exists student(
id int COMMENT “student’ ID”,
name string COMMENT “student’ Name”
)
row format delimited fields terminated by '\t'
stored as textfile –不指定也默认textfile 此行可省略
location '/user/hive/warehouse/student'; --不指定会放在当前所使用的库目录下，使用default 就会放在 /user/hive/warehouse
文件名和表名不用一致
表不能理解成目录
```

（3）根据查询结果创建表（查询的结果会添加到新创建的表中）

```
create table if not exists student2 as select id, name from student;
```

（4）根据已经存在的表结构创建表

```
create table if not exists student3 like student;
```

（5）查询表的类型

```
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE  
```



### 5.2 外部表

**1.理论**

因为表是外部表，所以Hive并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

**2.管理表和外部表的使用场景**

每天将收集到的网站日志定期流入HDFS文本文件。在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT进入内部表。

**3.案例实操**

分别创建部门和员工外部表，并向表中导入数据。

（1）原始数据

dept:

```
10	ACCOUNTING	1700
20	RESEARCH	1800
30	SALES	1900
40	OPERATIONS	1700
```

emp：

```
7369	SMITH	CLERK	7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-2-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-2-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-4-2	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-9-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-5-1	2850.00		30
7782	CLARK	MANAGER	7839	1981-6-9	2450.00		10
7788	SCOTT	ANALYST	7566	1987-4-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-9-8	1500.00	0.00	30
7876	ADAMS	CLERK	7788	1987-5-23	1100.00		20
7900	JAMES	CLERK	7698	1981-12-3	950.00		30
7902	FORD	ANALYST	7566	1981-12-3	3000.00		20
7934	MILLER	CLERK	7782	1982-1-23	1300.00		10
```

（2）上传数据到HDFS

```
hive (default)> dfs -mkdir /student;
hive (default)> dfs -put /opt/module/datas/student.txt /student;
```

（3）建表语句，创建外部表

创建部门表

```
create external table if not exists default.dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

创建员工表

```
create external table if not exists default.emp(
empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields terminated by '\t';
```

（4）查看创建的表

```
hive (default)>show tables;
```

（5）查看表格式化数据

```
hive (default)> desc formatted dept;
Table Type:             EXTERNAL_TABLE
```

（6）删除外部表

```
hive (default)> drop table dept;
```

外部表删除后，hdfs中的数据还在，但是metadata中dept的元数据已被删除



### 5.3 管理表与外部表的互相转换

**1.查询表的类型**

```
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE
```

**2.修改内部表student2为外部表**

```
alter table student2 set tblproperties('EXTERNAL'='TRUE');
```

**3.查询表的类型**

```
hive (default)> desc formatted student2;
Table Type:             EXTERNAL_TABLE
```

**4.修改外部表student2为内部表**

```
alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

**5.查询表的类型**

```
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE
```

注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！



## 六、修改表

### 6.1 重命名表

**1.语法**

```
ALTER TABLE table_name RENAME TO new_table_name
```

**2.实操案例**

```
hive (default)> alter table dept_partition2 rename to dept_partition3;
```



### 6.2 增加、修改和删除表分区

**1.引入分区表（需要根据日期对日志进行管理）**

```
/user/hive/warehouse/log_partition/20170702/20170702.log
/user/hive/warehouse/log_partition/20170703/20170703.log
/user/hive/warehouse/log_partition/20170704/20170704.log
```

**2.创建分区表语法**

```
hive (default)> create table dept_partition(
deptno int, dname string, loc string
)
partitioned by (month string)
row format delimited fields terminated by '\t';
```

注意：分区字段不能是表中已经存在的数据，可以将分区字段看作表的伪列。

**3.加载数据到分区表中**

```
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201709');
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201708');
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201707');
```

注意：分区表加载数据时，必须指定分区

加载数据到分区表

![Hive-DDL-6-2](E:\BigData-Hive\picture\Hive-DDL-6-2.png)

分区表

![Hive-DDL-6-3](E:\BigData-Hive\picture\Hive-DDL-6-3.png)

**4.查询分区表中数据**

单分区查询

```
hive (default)> select * from dept_partition where month='201709';
```

多分区联合查询

```
hive (default)> select * from dept_partition where month='201709'
              union
              select * from dept_partition where month='201708'
              union
              select * from dept_partition where month='201707';

_u3.deptno      _u3.dname       _u3.loc _u3.month
10      ACCOUNTING      NEW YORK        201707
10      ACCOUNTING      NEW YORK        201708
10      ACCOUNTING      NEW YORK        201709
20      RESEARCH        DALLAS  201707
20      RESEARCH        DALLAS  201708
20      RESEARCH        DALLAS  201709
30      SALES   CHICAGO 201707
30      SALES   CHICAGO 201708
30      SALES   CHICAGO 201709
40      OPERATIONS      BOSTON  201707
40      OPERATIONS      BOSTON  201708
40      OPERATIONS      BOSTON  201709
```

**5.增加分区**

创建单个分区

```
hive (default)> alter table dept_partition add partition(month='201706') ;
```

同时创建多个分区

```
hive (default)> alter table dept_partition add partition(month='201705') partition(month='201704');
```

**6.删除分区**

删除单个分区

```
hive (default)> alter table dept_partition drop partition (month='201704');
```

同时删除多个分区

```
hive (default)> alter table dept_partition drop partition (month='201705'), partition (month='201706');
```

**7.查看分区表有多少分区**

```
hive> show partitions dept_partition;
```

**8.查看分区表结构**

```
hive> desc formatted dept_partition;

# Partition Information          
# col_name              data_type               comment             
month                   string    
```



### 6.3 增加、修改、替换列信息

**1.语法**

（1）更新列

```
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
```

（2）增加和替换列

```
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
```

注：ADD是代表新增一字段，字段位置在所有列后面(partition列前)，REPLACE则是表示替换表中所有字段。

**2.实操案例**

（1）查询表结构

```
hive> desc dept;
```

（2）添加列

```
hive (default)> alter table dept add columns(deptdesc string);
```

（3）查询表结构

```
hive> desc dept;
```

（4）更新列

```
hive (default)> alter table dept change column deptdesc desc int;
```

（5）查询表结构

```
hive> desc dept;
```

（6）替换列

```
hive (default)> alter table dept replace columns(deptno string, dname
 string, loc string);
```

（7）查询表结构

```
hive> desc dept;
```



## 七、删除表

```
hive (default)> drop table dept;
```

