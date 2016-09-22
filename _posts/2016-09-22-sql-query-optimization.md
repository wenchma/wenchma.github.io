---
layout: post
title: "SQL Query Optimization"
categories: tech
tags: sql
date: 2016-09-22 15:50:21
---

## 1.多where，少having

`where`用来过滤行，`having`用来过滤组

聚合语句：统计分组数据时用，对分组数据再次判断时用having

HAVING子句可以让我们筛选成组后的各组数据．
HAVING子句在聚合后对组记录进行筛选
而WHERE子句在聚合前先筛选记录．也就是说作用在GROUP BY 子句和HAVING子句前.

**Sql代码**:

```sql
SELECT region, SUM(population), SUM(area)  
FROM bbc  
GROUP BY region  
HAVING SUM(area)>1000000  
```

在查询过程中聚合语句(sum,min,max,avg,count)要比`having`子句优先执行. 而`where`子句在查询过程中执行优先级别优先于聚合语句(sum,min,max,avg,count).

 

优先级：where > 聚合语句 > having

## 2.多union all，少union

union删除了重复的行，因此花费了一些时间

union只是将两个结果联结起来一起显示(消除了重复行)，并不是联结两个表

UNION ALL 这个指令的目的也是要将两个 SQL 语句的结果合并在一起.
UNION ALL 和 UNION 不同之处在于 UNION ALL 会将每一笔符合条件的资料都列出来，无论资料值有无重复。


## 3.多Exists，少in

Exists只检查存在性，性能比in强很多，not in 不推荐使用，因为不能应用表的索引，推荐not exists，尽量用exists 代替distinct

有些朋友不会用Exists，就举个例子
例，想要得到有电话号码的人的基本信息，table2有冗余信息

**Sql代码**:

```sql
select * from table1;--(id,name,age)   
select * from table2;--(id,phone)   
in：   
select * from table1 t1 where t1.id in (select t2.id from table2 t2 where t1.id=t2.id);   
Exists：   
select * from table1 t1 where Exists (select 1 from table2 t2 where t1.id=t2.id);   
```

> 用in的朋友注意了，当参数超过1000个，数据库就挂了。（oracle 10g数据库）


### 4.使用绑定变量

Oracle数据库软件会缓存已经执行的sql语句，复用该语句可以减少执行时间。
复用是有条件的，sql语句必须相同
问：怎样算不同？
答：随便什么不同都算不同，不管什么空格啊，大小写什么的，都是不同的
想要复用语句，建议使用PreparedStatement
将语句写成如下形式：

```sql
insert into XXX(pk_id,column1) values(?,?);
update XXX set column1=? where pk_id=?;
delete from XXX where pk_id=?;
select pk_id,column1 from XXX where pk_id=?;
```

### 5.少用*

因为要把`*`解析为列名，这时就查询数据字典，自然就耗费了许多时间。
很多朋友很喜欢用*，比如：select * from XXX;
一般来说，并不需要所有的数据，只需要一些，有的仅仅需要1个2个，
拿5W的数据量，10个属性来测试:
(这里的时间指的是PL/SQL Developer显示所有数据的时间)
使用select * from XXX;平均需要20秒，
使用select column1,column2 from XXX;平均需要12秒
(我的机子不是很好。。。)
对于开发来说，这一条是个灾难，知道是一回事，做就是另一回事了

## 6.分页sql

一般的分页sql如下所示：

```sql
sql1:select * from (select t.*,rownum rn from XXX t)where rn>0 and rn <10;
sql2:select * from (select t.*,rownum rn from XXX t where rownum <10)where rn>0;
```

乍看一下没什么区别，实际上区别很大...125万条数据测试，
sql1平均需要1.25秒(咋这么准呢？ )
sql2平均需要... 0.07秒
原因在于，子查询中，sql2排除了10以外的所有数据
当然了，如果查询最后10条，那效率是一样的 如果有分页需要排序，必须再包一层，结果为

```sql
select * from (select t.*, rownum rn from (select * from XXX order by value desc) t where rownum <= 10 ) where rn > 0;
```

## 7.能用一句sql，千万别用2句sql

不解释, :smile: 