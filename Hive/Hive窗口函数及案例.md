> **目录：**
>  **一、hive窗口函数语法**
>  ----over()窗口函数的语法结构
>  ----常与over()一起使用的分析函数
>  **二、hive窗口函数练习28道题**

[**视频录制好了，建议大家边看视频边阅读文章，文章中的窗口函数练习题也有相应的视频。点击蓝色字体可跳转到视频。**](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1Ff4y1S7qu%2F)

前言：我们在学习hive窗口函数的时候，一定要**先了解窗口函数的结构**。而不是直接百度`sum() over()、row_number() over()、`或者`count() over()`的用法，如果这样做，永远也掌握不到窗口函数的核心，当然我刚开始的时候也是这样做的，包括去年自己在接触ORACLE分析函数时也是这样搜索。

还好我比较顽强，在HIVE窗口函数问题上折腾了半个月、看了很多文章后才知道**over()才是窗口函数**，而`sum、row_number、count`只是与`over()`搭配的分析函数，当然除了这三个函数还有其他的函数。

## 一、hive窗口函数语法

在前言中我们已经说了avg()、sum()、max()、min()是分析函数，而over()才是窗口函数，下面我们来看看**over()窗口函数的语法结构、及常与over()一起使用的分析函数**

> 1、over()窗口函数的语法结构
>  2、常与over()一起使用的分析函数
>  3、窗口函数总结

### 1、over()窗口函数的语法结构



```csharp
分析函数 over(partition by 列名 order by 列名 rows between 开始位置 and 结束位置)
```

over()函数中包括三个函数：包括分区`partition by 列名`、排序`order by 列名`、指定窗口范围`rows between 开始位置 and 结束位置`。我们在使用over()窗口函数时，over()函数中的这三个函数可组合使用也可以不使用。

over()函数中如果不使用这三个函数，窗口大小是针对查询产生的所有数据，如果指定了分区，窗口大小是针对每个分区的数据。

##### 1.1、over()函数中的三个函数讲解

**order by**
 order by是排序的意思，是该窗口中的
 **A、partition by**
 `partition by`可理解为group by 分组。`over(partition by 列名)`搭配分析函数时，分析函数按照每一组每一组的数据进行计算的。

**B、rows between 开始位置 and 结束位置**
 是指定窗口范围，比如第一行到当前行。而这个范围是随着数据变化的。`over(rows between 开始位置 and 结束位置)`搭配分析函数时，分析函数按照这个范围进行计算的。
 **窗口范围说明：**
 我们常使用的窗口范围是`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW（表示从起点到当前行）`，常用该窗口来计算累加。



```undefined
PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：起点（一般结合PRECEDING，FOLLOWING使用）
UNBOUNDED PRECEDING 表示该窗口最前面的行（起点）
UNBOUNDED FOLLOWING：表示该窗口最后面的行（终点）
比如说：
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW（表示从起点到当前行）
ROWS BETWEEN 2 PRECEDING AND 1 FOLLOWING（表示往前2行到往后1行）
ROWS BETWEEN 2 PRECEDING AND 1 CURRENT ROW（表示往前2行到当前行）
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING（表示当前行到终点）
```

### 2、常与over()一起使用的分析函数：

##### 2.1、聚合类

**avg()、sum()、max()、min()**

##### 2.1、排名类

**row_number()**按照值排序时产生一个自增编号，不会重复`（如：1、2、3、4、5、6）`
 **rank()** 按照值排序时产生一个自增编号，值相等时会重复，会产生空位`（如：1、2、3、3、3、6）`
 **dense_rank()** 按照值排序时产生一个自增编号，值相等时会重复，不会产生空位`（如：1、2、3、3、3、4）`

##### 2.1、其他类

**lag**(列名,往前的行数,[行数为null时的默认值，不指定为null])，**可**以计算**用户上次购买时间**，或者用户下次购买时间。
 **lead**(列名,往后的行数,[行数为null时的默认值，不指定为null])
 **ntile(n)** 把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，ntile返回此行所属的组的编号

### 3、窗口函数总结：

其实窗口函数逻辑比较绕，我们可以把窗口理解为对表中的数据进行分组，排序等计算。要真正的理解HIVE窗口函数，还是要结合练习题才行。下面我们开始HIVE窗口函数的练习吧！

## 二、hive窗口函数练习28道题

### 第一套练习：hive之简单窗口函数 over()



```undefined
1、使用 over() 函数进行数据统计, 统计每个用户及表中数据的总数
2、求用户明细并统计每天的用户总数
3、计算从第一天到现在的所有 score 大于80分的用户总数
4、计算每个用户到当前日期分数大于80的天数
```

**测试数据**



```undefined
20191020,11111,85
20191020,22222,83
20191020,33333,86
20191021,11111,87
20191021,22222,65
20191021,33333,98
20191022,11111,67
20191022,22222,34
20191022,33333,88
20191023,11111,99
20191023,22222,33
```

**建表并导入数据：**



```csharp
create table test_window
(logday string,    #logday时间
userid string, 
score int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

#加载数据
load data local inpath '/home/xiaowangzi/hive_test_data/test_window.txt' into table test_window;
```

**我们先看下表中的数据：**



```csharp
select  *  from test_window;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-016e37d3cf620ffe.png?imageMogr2/auto-orient/strip|imageView2/2/w/1098/format/webp)

test_window

##### 1、使用 over() 函数进行数据统计, 统计每个用户及表中数据的总数



```csharp
select *, count(userid)over() as total  from  test_window;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-eb3d11b2b8d9ac3c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



这里使用 over() 与 select count(*) 有相同的作用，好处就是，在需要计算总数时不用再进行一次关联。

##### 2、求用户明细并统计每天的用户总数

可以使用 partition by 按日期列对数据进行分区处理，如：over(partition by logday)



```csharp
select  *,count()over(partition by logday)as day_total from  test_window;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-1c0985b757487517.png?imageMogr2/auto-orient/strip|imageView2/2/w/1142/format/webp)

求每天的用户数可以使用`select logday, count(userid) from recommend.test_window group by logday`，但是当想要得到 userid 信息时，这种用法的优势就很明显。

##### 3、计算从第一天到现在的所有 score 大于80分的用户总数

此时简单的分区不能满足需求，需要将 order by 和 窗口定义结合使用。



```csharp
select  *,count()over(order by logday rows between unbounded preceding and current row)as total 
from  test_window
where score > 80;
```



![img](https:////upload-images.jianshu.io/upload_images/2693156-e14d95a3a60f6d40.png?imageMogr2/auto-orient/strip|imageView2/2/w/1107/format/webp)


 通过 over() 计算出按日期的累加值。
 其实自己刚开始的时候就计算我思路是错了，我就想的是不用累加，直接`select *,count()over()as total from test_window where score > 80;`这样计算，如果这样计算的话只会显示表中所有大于80的人数，如果我想看20191021或者看20191022的人数看不见。



##### 4、计算每个用户到当前日期分数大于80的天数



```csharp
select *,
count()over(partition  by userid order by logday rows between unbounded preceding and current row) as total
from test_window
where score > 80  order by logday, userid;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-2d0656e1c65c76f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1104/format/webp)

### 第二套练习



```undefined
1、查询在2017年4月份购买过的顾客及总人数
2、查询顾客的购买明细及月购买总额
3、查询顾客的购买明细及到目前为止每个顾客购买总金额
4、查询顾客上次的购买时间----lag()over()偏移量分析函数的运用
5、查询前20%时间的订单信息
```

**测试数据**



```undefined
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

**建表并加载数据**



```csharp
create table business
(
name string, 
orderdate string,
cost int
)ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

#加载数据
load  data local inpath "/home/xiaowangzi/hive_test_data/business.txt" into table business;
```

**查看表数据**



```csharp
select   *  from  business;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-6e01560061f29970.png?imageMogr2/auto-orient/strip|imageView2/2/w/1073/format/webp)

查看表数据

#### 1、查询在2017年4月份购买过的顾客及总人数

分析：按照日期过滤、分组count求总人数(分组为什么不是用group by？自己思考)



```csharp
select   *,count()over() as total  from  business 
where substr(orderdate,1,7) = '2017-04';
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-00006289430572f4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1060/format/webp)

#### 2、查询顾客的购买明细及月购买总额

是计算每月每个用户的消费金额



```csharp
select 
*,
sum(cost) over(partition by name,substr(orderdate,1,7)) total_amount
from 
business;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-0ecd0b5ab163b432.png?imageMogr2/auto-orient/strip|imageView2/2/w/976/format/webp)

#### 3、查询顾客的购买明细及到目前为止每个顾客购买总金额

分析：按照顾客分组、日期升序排序、组内每条数据将之前的金额累加



```csharp
select 
*,
sum(cost) over(partition by name order  by  orderdate 
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) total_amount
from 
business;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-c047760bddb9e5d4.png?imageMogr2/auto-orient/strip|imageView2/2/w/960/format/webp)

#### 4、查询顾客上次的购买时间----lag()over()偏移量分析函数的运用

lag()over()偏移量分析函数的运用



```csharp
select 
name,
orderdate,
cost,
lag(orderdate,1) over(partition by name order by orderdate) last_date
from 
business;
```



![img](https:////upload-images.jianshu.io/upload_images/2693156-02d413ac859bceb8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1153/format/webp)

image.png


[参考文章：Hive lag()与lead() 函数](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu012637358%2Farticle%2Fdetails%2F84282342)



#### 5、查询前20%时间的订单信息



```csharp
select  *  
from 
(select   *,
ntile(5)over(order  by  cost)sortgroup_num from  business)t 
where t.sortgroup_num = 1;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-0103fb81a85264b0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1154/format/webp)

### 第三套练习：



```undefined
1、每门学科学生成绩排名(是否并列排名、空位排名三种实现)
2、每门学科成绩排名top n的学生
```

**原始数据(学生成绩信息)**



```undefined
name    subject score
孙悟空 语文  87
孙悟空 数学  95
孙悟空 英语  68
大海  语文  94
大海  数学  56
大海  英语  84
宋宋  语文  64
宋宋  数学  86
宋宋  英语  84
婷婷  语文  65
婷婷  数学  85
婷婷  英语  78
```

**建表并加载数据**



```csharp
create table score
(
name string,
subject string, 
score int
) row format delimited fields terminated by "\t";

#加载数据
load data local inpath '/home/xiaowangzi/hive_test_data/score.txt' into table score;
```

**查看数据**



```csharp
select  *  from  score;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-a7bac1849765682d.png?imageMogr2/auto-orient/strip|imageView2/2/w/818/format/webp)

#### 1、每门学科学生成绩排名(是否并列排名、空位排名三种实现)



```csharp
select  *,
row_number()over(partition by subject order by score desc),
rank()over(partition by subject order by score desc),
dense_rank()over(partition by subject order by score desc)
from score
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-700c55173ef1485a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1196/format/webp)

#### 2、每门学科成绩排名top n的学生



```csharp
select 
*
from 
(
select 
*,
row_number() over(partition by subject order by score desc) rmp
from score
) t
where t.rmp<=3;
```

![img](https:////upload-images.jianshu.io/upload_images/2693156-65c230e07704a362.png?imageMogr2/auto-orient/strip|imageView2/2/w/972/format/webp)

[参考文章1：Hive窗口函数案例详解](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fsutao-bigdata%2Fp%2F11609132.html)
 [参考文章2：hive表之简单窗口函数 over()](https://www.jianshu.com/p/088b92a3f853)



作者：夜希辰
链接：https://www.jianshu.com/p/3f3cf58472ca
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。