# Hive函数 

<nav>
<a href="#一、常用内置函数">一、常用内置函数</a><br/>
<a href="#二、窗口函数（开窗函数）">二、窗口函数（开窗函数）</a><br/>
<a href="#三、自定义函数">三、自定义函数</a><br/>
    <a href="#四、自定义UDF函数">四、自定义UDF函数</a><br/>
    <a href="#五、自定义UDTF函数">五、自定义UDTF函数</a><br/>
</nav>





## 一、常用内置函数

### 1.系统内置函数

**1.查看系统自带的函数**

```
hive> show functions;
```

**2.显示自带的函数的用法**

```
hive> desc function upper;
```

**3.详细显示自带的函数的用法**

```
hive> desc function extended upper;
```



### 2.常用内置函数

**1.字符串连接函数： concat** 

```
select concat('abc','def’,'gh');
```

**2.带分隔符字符串连接函数： concat_ws** 

```
hive (mydb)> select concat_ws(',','abc','def','gh');
----------------------------------
OK
_c0
abc,def,gh
```

**3.cast类型转换**

```
 select cast(1.5 as int);
```

**4.get_json_object(json 解析函数，用来处理json，必须是json格式)**

```
select get_json_object('{"name":"jack","age":"20"}','$.name');
```

**5.URL解析函数**

```
select parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1', 'HOST');
```

**6.explode**

把map集合中每个键值对或数组中的每个元素都单独生成一行的形式

**7.空字段函数NVL**

```
select comm,nvl(comm, -1) from emp;
```

**8.CASE WHEN**

（1）数据准备

| name | dept_id | sex  |
| ---- | ------- | ---- |
| 悟空 | A       | 男   |
| 大海 | A       | 男   |
| 宋宋 | B       | 男   |
| 凤姐 | A       | 女   |
| 婷姐 | B       | 女   |
| 婷婷 | B       | 女   |

（2）需求

求出不同部门男女各多少人。结果如下：

```
dept_Id     男       女
A     		2       1
B     		1       2
```

（3）创建本地emp_sex.txt，导入数据

```
[nogc@hadoop102 datas]$ vi emp_sex.txt
悟空	A	男
大海	A	男
宋宋	B	男
凤姐	A	女
婷姐	B	女
婷婷	B	女
```

（4）创建hive表并导入数据

```
create table emp_sex(
name string, 
dept_id string, 
sex string) 
row format delimited fields terminated by "\t";
load data local inpath '/opt/module/hive/datas/emp_sex.txt' into table emp_sex;
```

（5）按需求查询数据

```
select 
  dept_id,
  sum(case sex when '男' then 1 else 0 end) male_count,
  sum(case sex when '女' then 1 else 0 end) female_count
from 
  emp_sex
group by
  dept_id;
```

**9.行转列**

(1)相关函数说明

CONCAT(string A/col, string B/col…)：返回输入字符串连接后的结果，支持任意个输入字符串;

CONCAT_WS(separator, str1, str2,...)：它是一个特殊形式的 CONCAT()。第一个参数剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间;

COLLECT_SET(col)：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。

(2)数据准备

| name   | constellation | blood_type |
| ------ | ------------- | ---------- |
| 孙悟空 | 白羊座        | A          |
| 大海   | 射手座        | A          |
| 宋宋   | 白羊座        | B          |
| 猪八戒 | 白羊座        | A          |
| 凤姐   | 射手座        | A          |
| 苍老师 | 白羊座        | B          |

(3)需求

把星座和血型一样的人归类到一起。结果如下：

```
射手座,A            大海|凤姐
白羊座,A            孙悟空|猪八戒
白羊座,B            宋宋|苍老师
```

(4)创建本地constellation.txt，导入数据

```
[nogc@hadoop102 datas]$ vi constellation.txt
孙悟空	白羊座	A
大海	射手座	A
宋宋	白羊座	B
猪八戒	白羊座	A
凤姐	射手座	A
苍老师	白羊座	B
```

(5)创建hive表并导入数据

```
create table person_info(
name string, 
constellation string, 
blood_type string) 
row format delimited fields terminated by "\t";
load data local inpath "/opt/module/hive/datas/constellation.txt" into table person_info;
```

(6)按需求查询数据

```
SELECT t1.c_b , CONCAT_WS("|",collect_set(t1.name))
FROM (
SELECT NAME ,CONCAT_WS(',',constellation,blood_type) c_b
FROM person_info
)t1 
GROUP BY t1.c_b
```

**10.列转行**

(1)函数说明

EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。

LATERAL VIEW

用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias

解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

(2)数据准备

| movie         | category                 |
| ------------- | ------------------------ |
| 《疑犯追踪》  | 悬疑,动作,科幻,剧情      |
| 《Lie to me》 | 悬疑,警匪,动作,心理,剧情 |
| 《战狼2》     | 战争,动作,灾难           |

(3)需求

将电影分类中的数组数据展开。结果如下：

```
《疑犯追踪》      悬疑
《疑犯追踪》      动作
《疑犯追踪》      科幻
《疑犯追踪》      剧情
《Lie to me》   悬疑
《Lie to me》   警匪
《Lie to me》   动作
《Lie to me》   心理
《Lie to me》   剧情
《战狼2》        战争
《战狼2》        动作
《战狼2》        灾难
```

(4)创建本地movie.txt，导入数据

```
[nogc@hadoop102 datas]$ vi movie_info.txt
《疑犯追踪》	悬疑,动作,科幻,剧情
《Lie to me》	悬疑,警匪,动作,心理,剧情
《战狼2》	战争,动作,灾难
```

(5)创建hive表并导入数据

```
create table movie_info(
    movie string, 
    category string) 
row format delimited fields terminated by "\t";
load data local inpath "/opt/module/hive/datas/movie_info.txt" into table movie_info;
```

(6)按需求查询数据

```
SELECT movie,category_name 
FROM movie_info 
lateral VIEW
explode(split(category,",")) movie_info_tmp  AS category_name ;
```

### 3.Rank

**(1)函数说明**

RANK() 排序相同时会重复，总数不会变

DENSE_RANK() 排序相同时会重复，总数会减少

ROW_NUMBER() 会根据顺序计算



**(2)数据准备**

| name   | subject | score |
| ------ | ------- | ----- |
| 孙悟空 | 语文    | 87    |
| 孙悟空 | 数学    | 95    |
| 孙悟空 | 英语    | 68    |
| 大海   | 语文    | 94    |
| 大海   | 数学    | 56    |
| 大海   | 英语    | 84    |
| 宋宋   | 语文    | 64    |
| 宋宋   | 数学    | 86    |
| 宋宋   | 英语    | 84    |
| 婷婷   | 语文    | 65    |
| 婷婷   | 数学    | 85    |
| 婷婷   | 英语    | 78    |

**(3)需求**

计算每门学科成绩排名。

**(4)创建本地score.txt，导入数据**

```
[nogc@hadoop102 datas]$ vi score.txt
```

**(5)创建hive表并导入数据**

```
create table score(
name string,
subject string, 
score int) 
row format delimited fields terminated by "\t";
load data local inpath '/opt/module/hive/datas/score.txt' into table score;
```

**(6)按需求查询数据**

```
select name,
subject,
score,
rank() over(partition by subject order by score desc) rp,
dense_rank() over(partition by subject order by score desc) drp,
row_number() over(partition by subject order by score desc) rmp
from score;

name    subject score   rp      drp     rmp
孙悟空  数学    95      1       1       1
宋宋    数学    86      2       2       2
婷婷    数学    85      3       3       3
大海    数学    56      4       4       4
宋宋    英语    84      1       1       1
大海    英语    84      1       1       2
婷婷    英语    78      3       2       3
孙悟空  英语    68      4       3       4
大海    语文    94      1       1       1
孙悟空  语文    87      2       2       2
婷婷    语文    65      3       3       3
宋宋    语文    64      4       4       4
```



## 二、窗口函数（开窗函数）

### **2.1 相关函数说明**

```
OVER()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化。
CURRENT ROW：当前行
n PRECEDING：往前n行数据
n FOLLOWING：往后n行数据
UNBOUNDED：起点，
	UNBOUNDED PRECEDING 表示从前面的起点， 
    UNBOUNDED FOLLOWING表示到后面的终点
LAG(col,n,default_val)：往前第n行数据
LEAD(col,n, default_val)：往后第n行数据
NTILE(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。
```



### **2.2 数据准备：name，orderdate，cost**

```
jack,2017-01-01,10
tony,2017-01-02,15
jack,2017-02-03,23
tony,2017-01-04,29
jack,2017-01-05,46
jack,2017-04-06,42
tony,2017-01-07,50
jack,2017-01-08,55
mart,2017-04-08,62
mart,2017-04-09,68
neil,2017-05-10,12
mart,2017-04-11,75
neil,2017-06-12,80
mart,2017-04-13,94
```



### 2.3 需求

(1) 查询在2017年4月份购买过的顾客及总人数

(2) 查询顾客的购买明细及月购买总额

(3)上述的场景, 将每个顾客的cost按照日期进行累加

(4)查询每个顾客上次的购买时间

(5) 查询前20%时间的订单信息

**1.创建本地business.txt，导入数据**

```
[nogc@hadoop102 datas]$ vi business.txt
```

**2.创建hive表并导入数据**

```
create table business(
name string, 
orderdate string,
cost int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
load data local inpath "/opt/module/hive/datas/business.txt" into table business;
```

**3.按需求查询数据**

(1) 查询在2017年4月份购买过的顾客及总人数

```
select name,count(*) over () 
from business 
where substring(orderdate,1,7) = '2017-04' 
group by name;
```

(2) 查询顾客的购买明细及月购买总额

```
select name,orderdate,cost,sum(cost) over(partition by month(orderdate)) from
 business;
```

(3) 将每个顾客的cost按照日期进行累加

```
select name,orderdate,cost, 
sum(cost) over() as sample1,--所有行相加 
sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加 
sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，组内数据累加 
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row ) as sample4 ,--和sample3一样,由起点到当前行的聚合 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING and current row) as sample5, --当前行和前面一行做聚合 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING AND 1 FOLLOWING ) as sample6,--当前行和前边一行及后面一行 
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行 
from business;
```

rows必须跟在Order by 子句之后，对排序的结果进行限制，使用固定的行数来限制分区中的数据行数量

(4)查看顾客上次的购买时间

```
select name,orderdate,cost, 
lag(orderdate,1,'1900-01-01') over(partition by name order by orderdate ) as time1, lag(orderdate,2) over (partition by name order by orderdate) as time2 
from business;
```

(5)查询前20%时间的订单信息

```
select * from (
    select name,orderdate,cost, ntile(5) over(order by orderdate) sorted
    from business
) t
where sorted = 1;
```



## 三、自定义函数

1）Hive 自带了一些函数，比如：max/min等，但是数量有限，自己可以通过自定义UDF来方便的扩展。

2）当Hive提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数（UDF：user-defined function）。

3）根据用户自定义函数类别分为以下三种：

（1）UDF（User-Defined-Function）

​     一进一出

（2）UDAF（User-Defined Aggregation Function）

​     聚集函数，多进一出

​     类似于：count/max/min

（3）UDTF（User-Defined Table-Generating Functions）

​     一进多出

​     如lateral view explode()

4）官方文档地址

https://cwiki.apache.org/confluence/display/Hive/HivePlugins

5）编程步骤：

（1）继承Hive提供的类

​        org.apache.hadoop.hive.ql.udf.generic.GenericUDF  

​         org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;

（2）实现类中的抽象方法

（3）在hive的命令行窗口创建函数

添加jar

```
add jar linux_jar_path
```

创建function

```
create [temporary] function [dbname.]function_name AS class_name;
```

（4）在hive的命令行窗口删除函数

```
drop [temporary] function [if exists] [dbname.]function_name;
```



## 四、自定义UDF函数

**0）需求:**

自定义一个UDF实现计算给定字符串的长度，例如：

```
hive(default)> select my_len("abcd");

4
```

**1）创建一个Maven工程Hive**

**2）导入依赖**

```
<dependencies>
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>3.1.2</version>
		</dependency>
</dependencies>
```

**3）创建一个类**

```
package com.nogc.hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentLengthException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

/**
 * 自定义UDF函数，需要继承GenericUDF类
 * 需求: 计算指定字符串的长度
 */
public class MyStringLength extends GenericUDF {
    /**
     *
     * @param arguments 输入参数类型的鉴别器对象
     * @return 返回值类型的鉴别器对象
     * @throws UDFArgumentException
     */
    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
        // 判断输入参数的个数
        if(arguments.length !=1){
            throw new UDFArgumentLengthException("Input Args Length Error!!!");
        }
        // 判断输入参数的类型
        if(!arguments[0].getCategory().equals(ObjectInspector.Category.PRIMITIVE)){
            throw new UDFArgumentTypeException(0,"Input Args Type Error!!!");
        }
        //函数本身返回值为int，需要返回int类型的鉴别器对象
        return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
    }

    /**
     * 函数的逻辑处理
     * @param arguments 输入的参数
     * @return 返回值
     * @throws HiveException
     */
    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
       if(arguments[0].get() == null){
           return 0 ;
       }
       return arguments[0].get().toString().length();
    }

    @Override
    public String getDisplayString(String[] children) {
        return "";
    }
}
```

**4）打成ja包上传到服务器/opt/module/hive/datas/myudf.jar**

**5）将jar包添加到hive的classpath**

```
hive (default)> add jar /opt/module/hive/datas/myudf.jar;
```

**6）创建临时函数与开发好的java class关联, temporary表示临时**

```
hive (default)> create temporary function my_len as "com.nogc.hive. MyStringLength";//
```

**7）即可在hql中使用自定义的函数strip** 

**select my_len("12311");**

```
hive (default)> select ename,my_len(ename) ename_len from emp;
```



## 五、自定义UDTF函数

**0）需求**

自定义一个UDTF实现将一个任意分割符的字符串切割成独立的单词，例如：

```
hive(default)> select myudtf("hello,world,hadoop,hive", ",");

hello
world
hadoop
hive
```

**1）代码实现**

```
package com.nogc.udtf;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

import java.util.ArrayList;
import java.util.List;

public class MyUDTF extends GenericUDTF {

    private ArrayList<String> outList = new ArrayList<>();

    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {


        //1.定义输出数据的列名和类型
        List<String> fieldNames = new ArrayList<>();
        List<ObjectInspector> fieldOIs = new ArrayList<>();

        //2.添加输出数据的列名和类型
        fieldNames.add("lineToWord");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames, fieldOIs);
    }

    @Override
    public void process(Object[] args) throws HiveException {
        
        //1.获取原始数据
        String arg = args[0].toString();

        //2.获取数据传入的第二个参数，此处为分隔符
        String splitKey = args[1].toString();

        //3.将原始数据按照传入的分隔符进行切分
        String[] fields = arg.split(splitKey);

        //4.遍历切分后的结果，并写出
        for (String field : fields) {

            //集合为复用的，首先清空集合
            outList.clear();

            //将每一个单词添加至集合
            outList.add(field);

            //将集合内容写出
            forward(outList);
        }
    }

    @Override
    public void close() throws HiveException {

    }
}
```

**2）打成jar包上传到服务器/opt/module/hive/data/myudtf.jar**

**3）将jar包添加到hive的classpath下**

```
hive (default)> add jar /opt/module/hive/datas/myudtf.jar;
```

**4）创建临时函数与开发好的java class关联**

```
hive (default)> create temporary function myudtf as "com.nogc.hive.MyUDTF";
```

**5）使用自定义的函数**

```
hive (default)> select myudtf("hello,world,hadoop,hive",",") ;
```

