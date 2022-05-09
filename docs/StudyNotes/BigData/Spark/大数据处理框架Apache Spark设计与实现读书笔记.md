# 第一部分

## 第二章 Spark系统部署于应用运行的基本流程

### Spark系统架构

![Spark系统架构](https://images.shrugging.cn/Spark%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84.png)

* 每个Worker进程上存在一个或者多个ExecutorRunner对象。每个ExecutorRunner管理着一个Executor。Executor持有一个线程池。每个线程执行一个task。
* Worker进程通过持有ExecutorRunner对象来控制CoarseGrainedExecutorBackend进程的启停。
* 每个Spark应用启动一个Driver和多个Executor，每个Executor里面运行该的task都属于同一个Spark应用。

# 第二部分

## 第三章 Spark逻辑处理流程

### spark的逻辑处理流程

1. 数据源

2. 数据模型

   RDD是Spark最底层的数据结构

   * RDD只是一个逻辑上的概念，内存中并不会为RDD真正的分配存储空间（除非明确指出缓存该RDD），RDD的数据只会在计算过程中产生，计算完成后就会消失。
   * RDD包含多个数据分区，不同的数据分区可以由不同的任务在不同的节点进行处理。

3. 数据操作

   * 转换算子

     之所以成为transformation是因为转换的过程是一个单向的流程，旧RDD转换为新RDD后，只是生成了一个全新的RDD，旧RDD并不会产生改变。

   * 行动算子

     行动算子用来执行最后的计算

4. 计算结果处理

   * 直接将结果落盘操作
   * 需要传回Driver端进行相关计算

### RDD之间的依赖关系

* 窄依赖（Narrow Dependency）

  childRDD中的每个分区都依赖于parentRDD中的一个或多个分区的全部内容。

* 宽依赖（Shuffle Dependency）

  childRDD中的每个分区不完全都依赖于parentRDD中的一个或多个分区的全部内容。

### 常用的transformation()操作

| Transformation                                               | Meaning                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **map**(*func*)                                              | Return a new distributed dataset formed by passing each element of the source through a function *func*. |
| **filter**(*func*)                                           | Return a new dataset formed by selecting those elements of the source on which *func* returns true. |
| **flatMap**(*func*)                                          | Similar to map, but each input item can be mapped to 0 or more output items (so *func* should return a Seq rather than a single item). |
| **mapPartitions**(*func*)                                    | Similar to map, but runs separately on each partition (block) of the RDD, so *func* must be of type Iterator<T> => Iterator<U> when running on an RDD of type T. |
| **mapPartitionsWithIndex**(*func*)                           | Similar to mapPartitions, but also provides *func* with an integer value representing the index of the partition, so *func* must be of type (Int, Iterator<T>) => Iterator<U> when running on an RDD of type T. |
| **sample**(*withReplacement*, *fraction*, *seed*)            | Sample a fraction *fraction* of the data, with or without replacement, using a given random number generator seed. |
| **union**(*otherDataset*)                                    | Return a new dataset that contains the union of the elements in the source dataset and the argument. |
| **intersection**(*otherDataset*)                             | Return a new RDD that contains the intersection of elements in the source dataset and the argument. |
| **distinct**([*numPartitions*]))                             | Return a new dataset that contains the distinct elements of the source dataset. |
| **groupByKey**([*numPartitions*])                            | When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs. **Note:** If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using `reduceByKey` or `aggregateByKey` will yield much better performance. **Note:** By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional `numPartitions` argument to set a different number of tasks. |
| **reduceByKey**(*func*, [*numPartitions*])                   | When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function *func*, which must be of type (V,V) => V. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. |
| **aggregateByKey**(*zeroValue*)(*seqOp*, *combOp*, [*numPartitions*]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. |
| **sortByKey**([*ascending*], [*numPartitions*])              | When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean `ascending` argument. |
| **join**(*otherDataset*, [*numPartitions*])                  | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are supported through `leftOuterJoin`, `rightOuterJoin`, and `fullOuterJoin`. |
| **cogroup**(*otherDataset*, [*numPartitions*])               | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This operation is also called `groupWith`. |
| **cartesian**(*otherDataset*)                                | When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements). |
| **pipe**(*command*, *[envVars]*)                             | Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the process's stdin and lines output to its stdout are returned as an RDD of strings. |
| **coalesce**(*numPartitions*)                                | Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset. |
| **repartition**(*numPartitions*)                             | Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. This always shuffles all data over the network. |
| **repartitionAndSortWithinPartitions**(*partitioner*)        | Repartition the RDD according to the given partitioner and, within each resulting partition, sort records by their keys. This is more efficient than calling `repartition` and then sorting within each partition because it can push the sorting down into the shuffle machinery. |

### 常用的action()操作

| Action                                             | Meaning                                                      |
| :------------------------------------------------- | :----------------------------------------------------------- |
| **reduce**(*func*)                                 | Aggregate the elements of the dataset using a function *func* (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel. |
| **collect**()                                      | Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data. |
| **count**()                                        | Return the number of elements in the dataset.                |
| **first**()                                        | Return the first element of the dataset (similar to take(1)). |
| **take**(*n*)                                      | Return an array with the first *n* elements of the dataset.  |
| **takeSample**(*withReplacement*, *num*, [*seed*]) | Return an array with a random sample of *num* elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed. |
| **takeOrdered**(*n*, *[ordering]*)                 | Return the first *n* elements of the RDD using either their natural order or a custom comparator. |
| **saveAsTextFile**(*path*)                         | Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark will call toString on each element to convert it to a line of text in the file. |
| **saveAsSequenceFile**(*path*) (Java and Scala)    | Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc). |
| **saveAsObjectFile**(*path*) (Java and Scala)      | Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using `SparkContext.objectFile()`. |
| **countByKey**()                                   | Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key. |
| **foreach**(*func*)                                | Run a function *func* on each element of the dataset. This is usually done for side effects such as updating an [Accumulator](https://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators) or interacting with external storage systems. **Note**: modifyig variables other than Accumulators outside of the `foreach()` may result in undefined behavior. See [Understanding closures ](https://spark.apache.org/docs/latest/rdd-programming-guide.html#understanding-closures-a-nameclosureslinka)for more details. |

## 第四章 Spark物理执行计划

### Spark物理执行计划生成方

* 执行步骤
  1. 根据action()的操作顺序将应用划分为作业(job)
  2. 根据ShuffleDependency依赖关系将job划分为执行阶段(stage)
  3. 根据分区计算将各个stage划分为计算任务(task)

* 几个重要的问题：

  1. job、stage、task的计算顺序

     * job的时间根据action()的执行时间确定
     * stage根据shuffle前后阶段确定执行顺序
     * task可以并行处理 

  2. task内部数据的存储于计算问题（流水线计算）

     “流水线”计算可以减少内存的使用

  3. task之间的数据传输问题

     暂时理解为shuffle上游的stage会根据shuffle的分区提前进行分区，shuffle下游的stage会取走属于自己分区的数据。

# 第四部分

## 第六章 Shuffle机制

## 第七章 数据缓存机制

### 决定那些数据需要被缓存

1. 会被重复使用的数据

   会被多个job重复使用的数据，而且被重复使用的次数越多，缓存的性价比越高。迭代型和交互型应用就是非常适合的场景。

2. 数据不宜过大

   过大的数据会导致与磁盘的I/O，I/O的效率非常低，甚至低过重新计算的效率。

3. 非重复缓存的数据

   OneToOneDependency类型的RDD我们只缓存其child RDD即可。

面向结构化数据结构DataSet、DataFrame与RDD一样可以被缓存。

### 包含数据缓存操作的逻辑处理流程和物理执行计划

首先按照正常的物理执行计划进行规划，再根据加入的被缓存的数据对生成的计划进行调整。

### 缓存级别

* 存储位置

  内存 or 磁盘

* 是否序列化存储

  序列化后可以使用网络传输，同时减少存储空间。但是需要进行反序列化增加时延。

* 是否将缓存数据进行备份

  是否需要将数据复制到多节点

Spark将存储级别分为了12类，作为`persist()`方法的参数

对于MEMORY_ONLY来说，如果内存不够会导致部分分区无法被缓存，那么无法被缓存的分区数据再次被使用是就会重新计算。

### 缓存数据的写入方法

### 缓存数据的读取方法

### 用户接口的设计

* cache() == persist(MEMORY_ONLY)

### 缓存数据的替换与回收方法

几种不同的缓存替换算法：

* 先入先出（FIFO）
* 最近最久未使用（LRU）
* 最近最常被使用（MRU）

#### 自动缓存替换

* 首先RDD是根据action动态生成job的，因此无法预知被缓存的RDD在后面的job中的使用情况，Spark目前采用LRU来进行缓存替换
* Spark每计算一个record就进行缓存，因此在缓存结束之前无法预知目前的RDD需要多少空间，因此当缓存空间不足时就根据LRU替换一个或多个RDD（具体的数目跟一个动态的阈值相关），如果不够就继续替换一个或多个RDD。如果整个空间都不足且存储级别是MEMORY_ONLY就放弃。

#### Spark LRU回收算法的实现细节

* Spark在具体的Executor的JVM进程的内存区中是以LinkedHashMap的数据结构进行存储，存储的基本单位是RDD中的每个分区，标志位blockId，即rddId+partitionId。
* 如果遇到存储空间不足的情况，不会删除同一个RDD之前被存储的partition而去存储新的partition。

#### 用户主动回收缓存数据

* 用户可以使用`unpersist()`方法回收缓存
* `unpersist()`不同于`persist()`的懒执行，其会被立即执行
* `unpersist(blocking=true)`表示同步阻塞，即回收动作完成后才会继续执行下面的步骤，而`unpersist(blocking=false)`表示回收动作是异步执行的。

* 由于`unpersist()`被立即执行的特性，注意合理放置其在用户代码中的位置。

目前数据缓存是分属于不同的应用的，即同一应用的不同job之间可以共享被缓存的数据，而不同的Spark应用无法获取彼此的缓存数据。
