# RDDs





# Accumulators

## 默认累加器类型

* longAccumulator()
* doubleAccumulator()
* collectionAccumulator()

## 异常情况

### 少加

转换算子中调用累加器，如果没有行动算子的话，那么不会执行

```scala
val mapRDD: RDD[Int] = rdd.map(
       num => {
              sumACC.add(num)
              num
       }
)
```

上述map操作为转换算子，因此输出sum值为0

### 多加

```scala
mapRDD.collect()
mapRDD.collect()
```

如果多次执行行动算子的话，由于累加器是全局共享的，因此输出sum值为20

**一般情况下，累加器会放置在行动算子当中进行操作**

## 自定义累加器（以实现wordcount为例）

```scala
// 自定义实现累加器 wordcount

val rdd = sc.makeRDD(List("hello", "spark", "hello"))

val wcAcc = new MyAccumulator()
sc.register(wcAcc, "wordCountAcc")

rdd.foreach(
    word => {
        wcAcc.add(word)
    }
)

println(wcAcc.value)
```

* 自定义的累加器需要进行注册

```scala
class MyAccumulator extends AccumulatorV2[String, mutable.Map[String, Long]] {

		private var wcMap = mutable.Map[String, Long]()
		
		// 判断是否为初始状态
		override def isZero: Boolean = {
			wcMap.isEmpty
		}

		// 复制一个累加器
		override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] = {
			new MyAccumulator()
		}
		
		// 重置一个累加器
		override def reset(): Unit = {
			wcMap.clear()
		}

		// 获取累加器需要计算的值
		override def add(word: String): Unit = {
			val newCnt = wcMap.getOrElse(word, 0L) + 1
			wcMap.update(word, newCnt)

		}

		// Driver合并多个累加器
		override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]): Unit = {
			val map1 = this.wcMap
			val map2 = other.value

			map2.foreach {
				case (word, count) => {
					val newCount = map1.getOrElse(word, 0L) + count
					map1.update(word, newCount)
				}
			}
		}

		// 获取累加器结果
		override def value: mutable.Map[String, Long] = {
			wcMap
		}
	}
```

# Broadcasts Vars

```scala
val rdd1: RDD[(String, Int)] = sc.makeRDD(List(
       ("a", 1), ("b", 2), ("c", 3)
))
val map = mutable.Map(("a", 4), ("b", 5), ("c", 6))

rdd1.map {
       case (w, c) => {
              val l = map.getOrElse(w, 0)
              (w, (c, l))
       }
}.collect().foreach(println)
```

上述操作会导致向executor发送每个task时都携带了一份map数据，使用广播变量可以直接将map数据放置在每个executor的JVM内存之中，每个task都可以去获取JVM内存中的map，节省了开销，但注意广播变量一定是只读的。

```scala
val rdd1: RDD[(String, Int)] = sc.makeRDD(List(
       ("a", 1), ("b", 2), ("c", 3)
))
val map = mutable.Map(("a", 4), ("b", 5), ("c", 6))
// 封装广播变量
val bc: Broadcast[mutable.Map[String, Int]] = sc.broadcast(map)

// join会导致数据量的几何式增长
//            val joinRDD: RDD[(String, (Int, Int))] = rdd1.join(rdd2)
rdd1.map {
       case (w, c) => {
              val l = bc.value.getOrElse(w, 0)
              (w, (c, l))
       }
}.collect().foreach(println)
```
