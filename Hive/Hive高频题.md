题目：某张表两个字段分别是用户ID和登录时间。例如记录了用户id为001和002、003的登陆日期，现在问题来了：我们如何统计处两个用户各自连续登陆的天数最大值？也就是001可能会有连续登陆3天，7天，10天的这样记录，但是这其中只有10天是001连续登陆的最长天数。

日志数据

```sql
(001, '2020-04-20'),(001, '2020-04-21'),(001, '2020-04-25'),
(001, '2020-04-26'),(001, '2020-05-10'),(001, '2020-05-11'),
(001, '2020-05-12'),(001, '2020-05-30'),(001, '2020-06-17'),
(001, '2020-06-25'),(002, '2020-10-20'),(002, '2020-10-21'),
(002, '2020-10-22'),(002, '2020-10-23'),(002, '2020-11-10'),
(002, '2020-11-11'),(002, '2020-11-17'),(002, '2020-11-30'),
(002, '2020-12-17'),(002, '2020-12-25'),(003, '2020-10-01'),
(003, '2020-10-02'),(003, '2020-10-11'),(003, '2020-10-31'),
(003, '2020-11-01'),(003, '2020-11-02'),(003, '2020-11-03'),
(003, '2020-11-04'),(003, '2020-12-17'),(003, '2020-12-18');
```

一、实现原理

> 连续登陆，就是连续登录期间后一天减去前一天为1，如果大于1说明连续登录间断；增加一列，如果用户登录日期是连续的，那么这列数也是递增的。用当前日期减去递增列的数是一个固定值；
>
> 比如11-05号登录，对应的序号是1，得到的日期是11-05-1=11-04；
>
> 比如11-06号登录，对应的序号是2，得到的日期是11-06-2=11-04；

二、实现步骤（四部曲）

步骤1：按照用户id，日期新增1列，递增的序号

```sql
select uid,login_date,row_number() over(partition by uid order by login_date) as rn from user_log
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325110233300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1paUUhFTExPMjAxOA==,size_16,color_FFFFFF,t_70)

步骤二：新增列查询出每个用户的登录日期和当前序号差值，如果是连续登录会得到相同的日期；

```sql
select uid,login_date,rn,date_sub(login_date,rn) as diffdate 
from
(select uid,login_date,row_number() over(partition by uid order by login_date) as rn from user_log) as t1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325110319771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1paUUhFTExPMjAxOA==,size_16,color_FFFFFF,t_70)

步骤三：如果是连续的，获取到的diffdate是相同的，统计diffdate字段出现的次数就是连续登录次数

```sql
select t2.uid,count(t2.diffdate) as total_day
from 
(select uid,login_date,rn,date_sub(login_date,rn) as diffdate 
from
(select uid,login_date,row_number() over(partition by uid order by login_date) as rn from user_log) as t1) as t2
group by uid,diffdate
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325110401100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1paUUhFTExPMjAxOA==,size_16,color_FFFFFF,t_70)

步骤四：取出每个用户的最大登录天数

```sql
select uid,max(total_day) as max_login_day
from 
(select t2.uid,count(t2.diffdate) as total_day
from 
(select uid,login_date,rn,date_sub(login_date,rn) as diffdate 
from
(select uid,login_date,row_number() over(partition by uid order by login_date) as rn from user_log) as t1) as t2
group by uid,diffdate) as t3
group by uid;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325110443395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1paUUhFTExPMjAxOA==,size_16,color_FFFFFF,t_70)