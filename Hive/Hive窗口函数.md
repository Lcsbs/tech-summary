## 什么是窗口函数

窗口函数是用于分析用的一类函数，要理解窗口函数要先从聚合函数说起。 大家都知道聚合函数是将某列中多行的值合并为一行，比如sum、count等。 而窗口函数则可以在本行内做运算，得到多行的结果，即每一行对应一行的值。 通用的窗口函数可以用下面的语法来概括：

```text
Function() Over (Partition By Column1，Column2，Order By Column3)
```

窗口函数又分为以下三类： *聚合型窗口函数* 分析型窗口函数 * 取值型窗口函数

接下来我们将通过几个实际的例子来介绍下窗口函数。

## 准备数据

首先我们准备如下数据：

```sql
CREATE TABLE user_match_temp (
user_name string,
opponent string,
result int,
create_time timestamp);

INSERT INTO TABLE user_match_temp values
('vpspringcloud','vpspringboot',1,'2019-07-18 23:19:00'),
('vpspringboot','vpspringcloud',0,'2019-07-18 23:19:00'),
('vpspringcloud','vpspringdata',0,'2019-07-18 23:20:00'),
('vpspringdata','vpspringcloud',1,'2019-07-18 23:20:00'),
('vpspringcloud','vpspringroo',1,'2019-07-19 22:19:00'),
('vpspringroo','vpspringcloud',0,'2019-07-19 22:19:00'),
('vpspringdata','vpspringboot',0,'2019-07-19 23:19:00'),
('vpspringboot','vpspringdata',1,'2019-07-19 23:19:00');
```

数据包含4列，分别为 user_name，opponent，result，create_time。 我们将基于这些数据来介绍下窗口函数的一些使用场景。

## 聚合型窗口函数：

聚合型即SUM(), MIN(),MAX(),AVG(),COUNT()这些常见的聚合函数。 聚合函数配合窗口函数使用可以使计算更加灵活，例如以下场景： * 至今累计分数

```sql
hive> SELECT *, SUM(result) OVER (PARTITION BY user_name ORDER BY create_time) AS result_sums
hive> FROM user_match_temp;

+----------------+----------------+---------+------------------------+--------------+--+
|   user_name    |    opponent    | result  |      create_time       | result_sums  |
+----------------+----------------+---------+------------------------+--------------+--+
| vpspringdata   | vpspringcloud  | 1       | 2019-07-18 23:20:00.0  | 1            |
| vpspringdata   | vpspringboot   | 0       | 2019-07-19 23:19:00.0  | 1            |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-21 23:20:00.0  | 2            |
| vpspringdata   | vpspringboot   | 0       | 2019-07-23 23:19:00.0  | 2            |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-18 23:19:00.0  | 1            |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-18 23:20:00.0  | 1            |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-19 22:19:00.0  | 2            |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-20 23:19:00.0  | 3            |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-21 23:20:00.0  | 3            |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-22 22:19:00.0  | 4            |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-19 22:19:00.0  | 0            |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-22 22:19:00.0  | 0            |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-18 23:19:00.0  | 0            |
| vpspringboot   | vpspringdata   | 1       | 2019-07-19 23:19:00.0  | 1            |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-20 23:19:00.0  | 1            |
| vpspringboot   | vpspringdata   | 1       | 2019-07-23 23:19:00.0  | 2            |
+----------------+----------------+---------+------------------------+--------------+--+
```

- 之前3场平均胜场

```sql
hive> SELECT *,avg(result) over (partition by user_name order by create_time rows between 3 preceding and current row) as recently_wins
hive> From user_match_temp;
+----------------+----------------+---------+------------------------+---------------------+--+
|   user_name    |    opponent    | result  |      create_time       |    recently_wins    |
+----------------+----------------+---------+------------------------+---------------------+--+
| vpspringdata   | vpspringcloud  | 1       | 2019-07-18 23:20:00.0  | 1.0                 |
| vpspringdata   | vpspringboot   | 0       | 2019-07-19 23:19:00.0  | 0.5                 |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-21 23:20:00.0  | 0.6666666666666666  |
| vpspringdata   | vpspringboot   | 0       | 2019-07-23 23:19:00.0  | 0.5                 |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-18 23:19:00.0  | 1.0                 |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-18 23:20:00.0  | 0.5                 |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-19 22:19:00.0  | 0.6666666666666666  |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-20 23:19:00.0  | 0.75                |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-21 23:20:00.0  | 0.5                 |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-22 22:19:00.0  | 0.75                |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-19 22:19:00.0  | 0.0                 |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-22 22:19:00.0  | 0.0                 |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-18 23:19:00.0  | 0.0                 |
| vpspringboot   | vpspringdata   | 1       | 2019-07-19 23:19:00.0  | 0.5                 |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-20 23:19:00.0  | 0.3333333333333333  |
| vpspringboot   | vpspringdata   | 1       | 2019-07-23 23:19:00.0  | 0.5                 |
+----------------+----------------+---------+------------------------+---------------------+--+
```

 我们通过rows between 即可定义窗口的范围，这里我们定义了窗口的范围为之前3行到该行。

- 累计遇到的对手数量 需要注意的是count(distinct xxx)在窗口函数里是不允许使用的，不过我们也可以用size(collect_set() over(partition by order by))来替代实现我们的需求

```sql
hive> SELECT *,size(collect_set(opponent) over (partition by user_name order by create_time)) as recently_wins
hive> From user_match_temp;

+----------------+----------------+---------+------------------------+------------------+--+
|   user_name    |    opponent    | result  |      create_time       | opponent_counts  |
+----------------+----------------+---------+------------------------+------------------+--+
| vpspringdata   | vpspringcloud  | 1       | 2019-07-18 23:20:00.0  | 1                |
| vpspringdata   | vpspringboot   | 0       | 2019-07-19 23:19:00.0  | 2                |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-21 23:20:00.0  | 2                |
| vpspringdata   | vpspringboot   | 0       | 2019-07-23 23:19:00.0  | 2                |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-18 23:19:00.0  | 1                |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-18 23:20:00.0  | 2                |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-19 22:19:00.0  | 3                |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-20 23:19:00.0  | 3                |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-21 23:20:00.0  | 3                |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-22 22:19:00.0  | 3                |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-19 22:19:00.0  | 1                |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-22 22:19:00.0  | 1                |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-18 23:19:00.0  | 1                |
| vpspringboot   | vpspringdata   | 1       | 2019-07-19 23:19:00.0  | 2                |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-20 23:19:00.0  | 2                |
| vpspringboot   | vpspringdata   | 1       | 2019-07-23 23:19:00.0  | 2                |
+----------------+----------------+---------+------------------------+------------------+--+
```

collect_set()也是一个聚合函数，作用是将多行聚合进一行的某个set内，再用size()统计集合内的元素个数，即可实现我们的需求。

## 分析型窗口函数

分析型即RANk(),ROW_NUMBER(),DENSE_RANK()等常见的排序用的窗口函数，不过他们也是有区别的。

```sql
hive> SELECT *,
hive> rank() over (order by create_time) as user_rank,
hive> row_number() over (order by create_time) as user_row_number,
hive> dense_rank() over (order by create_time) as user_dense_rank
hive> FROM user_match_temp;

+----------------+----------------+---------+------------------------+------------+------------------+------------------+--+
|   user_name    |    opponent    | result  |      create_time       | user_rank  | user_row_number  | user_dense_rank  |
+----------------+----------------+---------+------------------------+------------+------------------+------------------+--+
| vpspringcloud  | vpspringboot   | 1       | 2019-07-18 23:19:00.0  | 1          | 1                | 1                |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-18 23:19:00.0  | 1          | 2                | 1                |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-18 23:20:00.0  | 3          | 3                | 2                |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-18 23:20:00.0  | 3          | 4                | 2                |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-19 22:19:00.0  | 5          | 5                | 3                |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-19 22:19:00.0  | 5          | 6                | 3                |
| vpspringdata   | vpspringboot   | 0       | 2019-07-19 23:19:00.0  | 7          | 7                | 4                |
| vpspringboot   | vpspringdata   | 1       | 2019-07-19 23:19:00.0  | 7          | 8                | 4                |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-20 23:19:00.0  | 9          | 9                | 5                |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-20 23:19:00.0  | 9          | 10               | 5                |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-21 23:20:00.0  | 11         | 11               | 6                |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-21 23:20:00.0  | 11         | 12               | 6                |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-22 22:19:00.0  | 13         | 13               | 7                |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-22 22:19:00.0  | 13         | 14               | 7                |
| vpspringdata   | vpspringboot   | 0       | 2019-07-23 23:19:00.0  | 15         | 15               | 8                |
| vpspringboot   | vpspringdata   | 1       | 2019-07-23 23:19:00.0  | 15         | 16               | 8                |
+----------------+----------------+---------+------------------------+------------+------------------+------------------+--+
```

如上所示： *row_number函数：生成连续的序号（相同元素序号相同）；* rank函数：如两元素排序相同则序号相同，并且会跳过下一个序号； * rank函数：如两元素排序相同则序号相同，不会跳过下一个序号；

除了这三个排序用的函数，还有 *CUME_DIST函数 ：小于等于当前值的行在所有行中的占比* PERCENT_RANK() ：小于当前值的行在所有行中的占比 * NTILE() ：如果把数据按行数分为n份，那么该行所属的份数是第几份 这三种窗口函数 效果如下：

```sql
hive2> SELECT *,
hive2> CUME_DIST() over (order by create_time) as user_CUME_DIST,
hive2> PERCENT_RANK() over (order by create_time) as user_PERCENT_RANK,
hive2> NTILE(3) over (order by create_time) as user_NTILE
hive2> FROM user_match_temp;

+----------------+----------------+---------+------------------------+-----------------+----------------------+-------------+--+
|   user_name    |    opponent    | result  |      create_time       | user_CUME_DIST  |  user_PERCENT_RANK   | user_NTILE  |
+----------------+----------------+---------+------------------------+-----------------+----------------------+-------------+--+
| vpspringcloud  | vpspringboot   | 1       | 2019-07-18 23:19:00.0  | 0.125           | 0.0                  | 1           |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-18 23:19:00.0  | 0.125           | 0.0                  | 1           |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-18 23:20:00.0  | 0.25            | 0.13333333333333333  | 1           |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-18 23:20:00.0  | 0.25            | 0.13333333333333333  | 1           |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-19 22:19:00.0  | 0.375           | 0.26666666666666666  | 1           |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-19 22:19:00.0  | 0.375           | 0.26666666666666666  | 1           |
| vpspringdata   | vpspringboot   | 0       | 2019-07-19 23:19:00.0  | 0.5             | 0.4                  | 2           |
| vpspringboot   | vpspringdata   | 1       | 2019-07-19 23:19:00.0  | 0.5             | 0.4                  | 2           |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-20 23:19:00.0  | 0.625           | 0.5333333333333333   | 2           |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-20 23:19:00.0  | 0.625           | 0.5333333333333333   | 2           |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-21 23:20:00.0  | 0.75            | 0.6666666666666666   | 2           |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-21 23:20:00.0  | 0.75            | 0.6666666666666666   | 3           |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-22 22:19:00.0  | 0.875           | 0.8                  | 3           |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-22 22:19:00.0  | 0.875           | 0.8                  | 3           |
| vpspringdata   | vpspringboot   | 0       | 2019-07-23 23:19:00.0  | 1.0             | 0.9333333333333333   | 3           |
| vpspringboot   | vpspringdata   | 1       | 2019-07-23 23:19:00.0  | 1.0             | 0.9333333333333333   | 3           |
+----------------+----------------+---------+------------------------+-----------------+----------------------+-------------+--+
```

## 取值型窗口函数

这几个函数可以通过字面意思记得，LAG是迟滞的意思，也就是对某一列进行往后错行；LEAD是LAG的反义词，也就是对某一列进行提前几行；FIRST_VALUE是对该列到目前为止的首个值，而LAST_VALUE是到目前行为止的最后一个值。

LAG()和LEAD() 可以带3个参数，第一个是返回的值，第二个是前置或者后置的行数，第三个是默认值。

下一个对手，上一个对手，最近3局的第一个对手及最后一个对手，如下：

```sql
hive> SELECT *,
hive>     lag(opponent,1) 
hive>         over (partition by user_name order by create_time) as lag_opponent,
hive>     lead(opponent,1) over 
hive>         (partition by user_name order by create_time) as lead_opponent,
hive>     first_value(opponent) over (partition by user_name order by create_time rows hive>         between 3 preceding and 3 following) as first_opponent,
hive>     last_value(opponent) over (partition by user_name order by create_time rows hive>         between 3 preceding and 3 following) as last_opponent
hive> From user_match_temp;
+----------------+----------------+---------+------------------------+----------------+----------------+-----------------+----------------+--+
|   user_name    |    opponent    | result  |      create_time       |  lag_opponent  | lead_opponent  | first_opponent  | last_opponent  |
+----------------+----------------+---------+------------------------+----------------+----------------+-----------------+----------------+--+
| vpspringdata   | vpspringcloud  | 1       | 2019-07-18 23:20:00.0  | NULL           | vpspringboot   | vpspringcloud   | vpspringboot   |
| vpspringdata   | vpspringboot   | 0       | 2019-07-19 23:19:00.0  | vpspringcloud  | vpspringcloud  | vpspringcloud   | vpspringboot   |
| vpspringdata   | vpspringcloud  | 1       | 2019-07-21 23:20:00.0  | vpspringboot   | vpspringboot   | vpspringcloud   | vpspringboot   |
| vpspringdata   | vpspringboot   | 0       | 2019-07-23 23:19:00.0  | vpspringcloud  | NULL           | vpspringcloud   | vpspringboot   |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-18 23:19:00.0  | NULL           | vpspringdata   | vpspringboot    | vpspringboot   |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-18 23:20:00.0  | vpspringboot   | vpspringroo    | vpspringboot    | vpspringdata   |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-19 22:19:00.0  | vpspringdata   | vpspringboot   | vpspringboot    | vpspringroo    |
| vpspringcloud  | vpspringboot   | 1       | 2019-07-20 23:19:00.0  | vpspringroo    | vpspringdata   | vpspringboot    | vpspringroo    |
| vpspringcloud  | vpspringdata   | 0       | 2019-07-21 23:20:00.0  | vpspringboot   | vpspringroo    | vpspringdata    | vpspringroo    |
| vpspringcloud  | vpspringroo    | 1       | 2019-07-22 22:19:00.0  | vpspringdata   | NULL           | vpspringroo     | vpspringroo    |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-19 22:19:00.0  | NULL           | vpspringcloud  | vpspringcloud   | vpspringcloud  |
| vpspringroo    | vpspringcloud  | 0       | 2019-07-22 22:19:00.0  | vpspringcloud  | NULL           | vpspringcloud   | vpspringcloud  |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-18 23:19:00.0  | NULL           | vpspringdata   | vpspringcloud   | vpspringdata   |
| vpspringboot   | vpspringdata   | 1       | 2019-07-19 23:19:00.0  | vpspringcloud  | vpspringcloud  | vpspringcloud   | vpspringdata   |
| vpspringboot   | vpspringcloud  | 0       | 2019-07-20 23:19:00.0  | vpspringdata   | vpspringdata   | vpspringcloud   | vpspringdata   |
| vpspringboot   | vpspringdata   | 1       | 2019-07-23 23:19:00.0  | vpspringcloud  | NULL           | vpspringcloud   | vpspringdata   |
+----------------+----------------+---------+------------------------+----------------+----------------+-----------------+----------------+--+
```



发布于 2019-08-12