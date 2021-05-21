前言：我们在学习hive窗口函数的时候，一定要**先了解窗口函数的结构**。而不是直接百度`sum() over()、row_number() over()、`或者`count() over()`的用法，如果这样做，永远也掌握不到窗口函数的核心，当然我刚开始的时候也是这样做的，包括去年自己在接触ORACLE分析函数时也是这样搜索。

还好我比较顽强，在HIVE窗口函数问题上折腾了半个月、看了很多文章后才知道**over()才是窗口函数**，而`sum、row_number、count`只是与`over()`搭配的分析函数，当然除了这三个函数还有其他的函数。

## 一、hive窗口函数语法

在前言中我们已经说了avg()、sum()、max()、min()是分析函数，而over()才是窗口函数，下面我们来看看**over()窗口函数的语法结构、及常与over()一起使用的分析函数**

> 1. over()窗口函数的语法结构
> 2. 常与over()一起使用的分析函数
> 3. 窗口函数总结

### 1. over()窗口函数的语法结构

```text
分析函数 over(partition by 列名 order by 列名 rows between 开始位置 and 结束位置)
```

over()函数中包括三个函数：包括分区`partition by 列名`、排序`order by 列名`、指定窗口范围`rows between 开始位置 and 结束位置`。我们在使用over()窗口函数时，over()函数中的这三个函数可组合使用也可以不使用。

over()函数中如果不使用这三个函数，窗口大小是针对查询产生的所有数据，如果指定了分区，窗口大小是针对每个分区的数据。

### 1.1 over()函数中的三个函数讲解

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

### 2. 常与over()一起使用的分析函数

#### 2.1 聚合类

**avg()、sum()、max()、min()**

#### 2.2 排名类

**row_number()**按照值排序时产生一个自增编号，不会重复`（如：1、2、3、4、5、6）`
 **rank()** 按照值排序时产生一个自增编号，值相等时会重复，会产生空位`（如：1、2、3、3、3、6）`
 **dense_rank()** 按照值排序时产生一个自增编号，值相等时会重复，不会产生空位`（如：1、2、3、3、3、4）`

#### 2.3 其他类

**lag**(列名,往前的行数,[行数为null时的默认值，不指定为null])，**可**以计算**用户上次购买时间**，或者用户下次购买时间。
 **lead**(列名,往后的行数,[行数为null时的默认值，不指定为null])
 **ntile(n)** 把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，ntile返回此行所属的组的编号

### 3. 窗口函数总结

其实窗口函数逻辑比较绕，我们可以把窗口理解为对表中的数据进行分组，排序等计算。要真正的理解HIVE窗口函数，还是要结合练习题才行。下面我们开始HIVE窗口函数的练习吧！









