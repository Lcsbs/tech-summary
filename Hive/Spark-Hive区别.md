spark和hive的区别

hive定位数据仓库，hive是mapreduce封装；

spark-sql查询引擎

hive和spark对比

**Hive内部表和外部表的区别**

内部表又叫管理表，创建表时不做任何修饰默认创建内部表。想要创建外部表（External Table）需要 **External** 关键字修饰；

|              | 内部表                                                       | 外部表                           |
| ------------ | ------------------------------------------------------------ | -------------------------------- |
| 数据存储位置 | 由 *hive.metastore.warehouse.dir* 参数指定，默认数据存储在HDFS的 ***/user/hive/warehouse/数据库名.db/表名/*** | 由创建表时，*Location*  参数指定 |
| 导入数据     | 数据导入内部表，由hive管理生命周期                           | 改变元数据指定的目录             |
| 删除表       | 删除元数据（metadata）和文件                                 | 删除元数据                       |

https://book.itheima.net/

**Spark处理数据比Hive快的原因**

1. 消除了冗余的HDFS读写

1. 消除了MapReduce阶段，提供丰富的算子，reduce阶段可以cache到内存；
2. JVM的优化；MapReduce是基于进程的，启动一个task就会启动一个jvm；spark是在Excutor中启动一次JVM，每个task启动线程，并在线程池中复用；减少启动花费时间；

**hive列转行**

TODO

Hive内置函数：http://www.cnblogs.com/end/archive/2012/06/18/2553682.html

*collect_set* 就用到了hive行转列知识，需要用到两个内置UDF *collect_set* 、 *conact_ws*

| 返回类型 | 函数                                       | 说明                                           |
| -------- | ------------------------------------------ | ---------------------------------------------- |
| array    | collect_set(col name)                      | 返回无重复记录                                 |
| String   | concat_ws(string SEP, string A, string B…) | 链接多个字符串，字符串之间以指定的分隔符分开。 |
|          |                                            |                                                |

**Flink和Spark Streaming区别**

TODO