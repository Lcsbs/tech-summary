1. Hive哪些查询会执行MapReduce

简单的查询，select，不带count/group/avg函数都不走mr，直接读取HDFS文件进行过滤；好处是很大提高执行效率；

如果要调整修改hive-site.xml，修改hive.fetch.evtion参数为more，简单查询不走mr；设置为minial，任何简单查询都会走mr；

2. Hive的UDF函数

UDF（user-defined-function）：用户定义的普通函数，一进一出；

UDAF(user-defined-aggeration-function)：用户自定义的聚集函数，多进一出

UDTF：用户自定义的表函数，多进多出

hive总结篇及优化

https://blog.csdn.net/yu0_zhang0/article/details/81776459

3. hive和关系型数据库的区别

|                | Hive                                                         | 关系型数据库                       |
| -------------- | ------------------------------------------------------------ | ---------------------------------- |
| 数据查询延迟性 | 延迟性高，适合大数据量的离线批处理                           | 延迟性低，适合在线分析             |
| 数据存储位置   | HDFS                                                         | 本地文件系统                       |
| 查询语言类似   | Hive QL                                                      | SQL                                |
| 事务型         | 0.14版本上支持                                               | 支持                               |
| 数据格式       | hive支持多种数据格式，无需从用户定义数据格式到hive定义数据格式转换，加载数据快 | 数据库存储引擎自己实现，加载数据慢 |
| 任务执行       | MapReduce                                                    | Excetor                            |

4. Hive的优点

简单易上手，免去开发人员学习MapReduce开发成本，统一的元数据管理metestore，包含了数据库表、分区、字段等信息；

共享元数据信息计算引擎可以选MapReduce、Spark、Tez；支持自定义函数；

5. SQL转化为MapReduce的过程

了解了MapReduce实现SQL基本操作之后，我们来看看Hive是如何将SQL转化为MapReduce任务的，整个编译过程分为六个阶段：

Antlr定义SQL的语法规则，完成SQL词法，语法解析，将SQL转化为抽象语法树AST Tree
遍历AST Tree，抽象出查询的基本组成单元QueryBlock
遍历QueryBlock，翻译为执行操作树OperatorTree
逻辑层优化器进行OperatorTree变换，合并不必要的ReduceSinkOperator，减少shuffle数据量
遍历OperatorTree，翻译为MapReduce任务
物理层优化器进行MapReduce任务的变换，生成最终的执行计划

6. **Hive的内部表和外部表的区别**

被external修饰的表是外部表，未被external修饰的表是内部表；

区别：

内部表的数据是由hive自身来管理的，外部表数据是由hdfs管理的；

内部表数据存储位置通过hive-site.xml的hive.metastore.warehouse配置，默认是/user/warehouse/db目录；外部表数据存储位置由创建表时自己指定；

内部表删除时会删除元数据信息和实际存储数据；外部表删除时 会删掉表的元数据信息；

7. **动态分区和静态分区的区别**

动态和静态分区，创建表相同；

静态分区加载数据时需要指定分区字段值；动态分区加载数据时指定分区字段就行；

8. **Hive的性能优化**

Hive的性能优化，参考：https://cloud.tencent.com/developer/article/1453464

- 列裁剪和分区裁剪，查询时指定列和分区，全表扫描数据效率低；
- 提前执行where逻辑，减少下游的数据量；
- 使用Sort by代替 order by；order by会全局数据排序，map端数据全部到reduce端，数据量大时长时间执行不完。使用sort by在每个分区内排序，视情况会启动多个reducer，每个reducer内有序；
- 使用group by代替distinct；当需要统计某一行的去重，使用count(distinct)可以用group by来代替，count(distinct)会产生较少的reducer；
- join基础优化

1. 小表前置，hive在解析带有join的语句时，会尝试将最后的表加载到内存中。如果表太大会导致oom；
2. 多表join时key相同，会将多个join合并为一个MapReduce任务来执行；
3. 利用map join特性，特别适合大小表join的情况。hive在map端完成join操作，节省了reduce端，提高效率；

- 优化SQL处理join数据倾斜

1. 对于空值如果不需要就提前用where条件过滤掉，如果需要则用随机数的方式打散；

- MapReduce优化

1. 调整合理的mapper数，一般如果mapper输入是少量大文件就减少mapper数，如果输入是大量非小文件就增加mapper数；
2. 设置合理的reducer数，通过mapred.reduce.tasks参数来配置；

- 合并小文件

1. 输入端合并小文件；需要改变hive文件的输入，通过hive.input.format参数改为CombineInputFormat；
2. 输出端合并小文件；直接将hive.merge.file和hive.merged.file设置为true；

- 启用压缩

压缩中间结果和输出结果。使用snnapy压缩；

- JVM重用

在MapReduce中，一个task就会启动一个JVM，如果JVM启动时间过长，而且task数量多的话，性能消耗很大。可以配置mapred.job.reuse.jvm.num.tasks数量，来复用JVM；

- 并行执行与本地模式
- 开启严格模式

严格模式就是不允许用户执行3中有风险的语句；

1. 查询分区表时未指定分区列
2. 两表join会产生笛卡儿积
3. 用order by为限制limit条件的

- 采用合适的存储格式

创建表时指定stored as，一般用parquet和text file较多