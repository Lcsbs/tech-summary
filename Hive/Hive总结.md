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

- 列裁剪和分区裁剪，查询时指定列和分区
- 使用Sort by代替 order by
- 使用group by代替distinct
- 开启严格模式
- 采取合适的存储格式
- MapReduce优化
- 优化SQL处理数据倾斜
- 本地模式与开启并行执行

