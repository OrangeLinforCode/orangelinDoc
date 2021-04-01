# Spark复习





## 1.宽依赖、窄依赖

（个人理解）

划分所处环节：逻辑处理流程生成 DAG时

处理RDD之间的数据依赖：RDD之间的数据依赖、RDD多分区之间的依赖

- **为什么有宽依赖和窄依赖？**

解决新生成RDD与父RDD之间的分区间的数据依赖关系

窄依赖：**childRDD每个分区都依赖parentRDD中的一部分分区**

宽依赖：**childRDD每个分区都依赖parentRDD中每个分区的一部分**

两者区别：childRDD的每个分区是否完全依赖parentRDD的一个或者多个分区，如果parentRDD的一个或者多个分区中的数据全部流入childRDD的某一个或者多个分区，则是窄依赖

如果parentRDD分区中数据需要一部分流入childRDD中的某一个分区，另外一部分流入childRDD的另外分区，则是宽依赖

- **划分宽窄依赖有什么好处？**

有利于生成物理执行计划，是否生成shuffle



## 2.如何对RDD数据进行分区

水平划分：record的索引

Hash划分：recird的Hash值

Range划分：排序任务使用，按照元素大小关系





### 3.算子

**reduceByKey、aggregateByKey、combineByKey、foldByKey区别?**



foldByKey比reduceByKey的性能高，产生的临时对象少



cogroup聚合rdd时 可能是宽依赖也可能是窄依赖，也可能都存在

**join算子底层调用cogroup算子**

调用时shuffle的情况需要根据 多个父RDD的分区数、分区规则来确定





**cartesian算子：笛卡尔积**

概算自不是shuffleDependecy



**sortByKey**

如何即对key排序，也对value排序

实现：

- 方法一：使用map，将数据转换成  <(key,value),null>，将（key，value）定义成新的class，并定义compare()函数，最后再使用soryByKey
- 方法二：先使用groupByKey将数据聚合成<key,list(value)>，在使用rdd.mapValues(sort func)对list进行排序



**intersection算子：取交集**

原理：使用cgroup聚合 2个rdd，然后过滤出都存在的record，具体是先将record转换成（K,V）类型，V为固定null，，在聚合数据并过滤掉（）的record



**为什么已经有reduceByKey、AggregateByKey 等操作，还要定义aggregate 和reduce？**

有些场景需要全局聚合，需要对部分聚合结果进行汇总，这个merge汇总的操作就是aggregate、reduce



**大数据量下，merge操作对于单点driver会存在效率和内存问题**

treeAggregate、treeReduce 使用树形聚合的方式

本质上有点类似 归并排序