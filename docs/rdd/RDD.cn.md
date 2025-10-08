# RDD &mdash; 分布式计算的描述

`RDD[T]` 是[容错弹性分布式数据集](#implementations)的[抽象](#contract)，它仅仅是对分布式记录集合（类型为 `T`）上计算的描述。

## Contract

### <span id="compute"> 计算分区

```scala
compute(
  split: Partition,
  context: TaskContext): Iterator[T]
```

计算输入的 [Partition](Partition.md)（使用 [TaskContext](../scheduler/TaskContext.md)）以生成类型为 `T` 的值。

使用场景：

* `RDD` 被请求 [computeOrReadCheckpoint](#computeOrReadCheckpoint)

### <span id="getPartitions"> 分区

```scala
getPartitions: Array[Partition]
```

使用场景：

* `RDD` 被请求获取 [partitions](#partitions)

## Implementations

* [CheckpointRDD](CheckpointRDD.md)
* CoalescedRDD
* [CoGroupedRDD](CoGroupedRDD.md)
* [HadoopRDD](HadoopRDD.md)
* [MapPartitionsRDD](MapPartitionsRDD.md)
* [NewHadoopRDD](NewHadoopRDD.md)
* [ParallelCollectionRDD](ParallelCollectionRDD.md)
* [ReliableCheckpointRDD](ReliableCheckpointRDD.md)
* [ShuffledRDD](ShuffledRDD.md)
* [SubtractedRDD](SubtractedRDD.md)
* _其他_

## 创建实例

创建 `RDD` 需要以下参数：

* <span id="_sc"> [SparkContext](../SparkContext.md)
* <span id="deps"> [Dependencies](Dependency.md) (**父 RDD**，在计算此 RDD 之前必须成功计算的 RDD)

??? note "抽象类"
    `RDD` 是一个抽象类，不能直接创建。它是为[具体的 RDD](#implementations) 间接创建的。

## <span id="partitions"> partitions

```scala
partitions: Array[Partition]
```

`partitions`...待完善

`partitions` 使用场景：

* `DAGScheduler` 被请求 [getPreferredLocsInternal](../scheduler/DAGScheduler.md#getPreferredLocsInternal)
* `SparkContext` 被请求 [runJob](../SparkContext.md#runJob)
* `Stage` 被[创建](../scheduler/Stage.md#numPartitions)
* _其他_

## <span id="toDebugString"> 递归依赖

```scala
toDebugString: String
```

`toDebugString`...待完善

## <span id="doCheckpoint"> doCheckpoint

```scala
doCheckpoint(): Unit
```

`doCheckpoint`...待完善

`doCheckpoint` 使用场景：

* `SparkContext` 被请求[同步运行作业](../SparkContext.md#runJob)

## <span id="iterator"> iterator

```scala
iterator(
  split: Partition,
  context: TaskContext): Iterator[T]
```

`iterator`...待完善

!!! note "Final 方法"
    `iterator` 是一个 `final` 方法，子类不能覆盖它。参见 [Scala 语言规范]({{ scala.spec }})中的 [5.2.6 final]({{ scala.spec }}/05-classes-and-objects.html)。

### <span id="getOrCompute"> getOrCompute

```scala
getOrCompute(
  partition: Partition,
  context: TaskContext): Iterator[T]
```

`getOrCompute`...待完善

### <span id="computeOrReadCheckpoint"> computeOrReadCheckpoint

```scala
computeOrReadCheckpoint(
  split: Partition,
  context: TaskContext): Iterator[T]
```

`computeOrReadCheckpoint`...待完善

## 隐式方法

### <span id="rddToOrderedRDDFunctions"> rddToOrderedRDDFunctions

```scala
rddToOrderedRDDFunctions[K : Ordering : ClassTag, V: ClassTag](
  rdd: RDD[(K, V)]): OrderedRDDFunctions[K, V, (K, V)]
```

`rddToOrderedRDDFunctions` 是一个 Scala 隐式方法，用于创建 [OrderedRDDFunctions](OrderedRDDFunctions.md)。

`rddToOrderedRDDFunctions` 使用场景（隐式）：

* [RDD.sortBy](spark-rdd-transformations.md#sortBy)
* [PairRDDFunctions.combineByKey](PairRDDFunctions.md#combineByKey)

## Review Me

== [[extensions]][[implementations]] 可用 RDD（子集）

[cols="30,70",options="header",width="100%"]
|===
| RDD
| 描述

| [CoGroupedRDD](CoGroupedRDD.md)
| [[CoGroupedRDD]]

| CoalescedRDD
| [[CoalescedRDD]] spark-rdd-partitions.md#repartition[repartition] 或 spark-rdd-partitions.md#coalesce[coalesce] 转换的结果

| HadoopRDD.md[HadoopRDD]
| [[HadoopRDD]] 允许使用旧的 MapReduce API 从 HDFS 中读取存储的数据。最著名的用例是 `SparkContext.textFile` 返回的 RDD。

| MapPartitionsRDD.md[MapPartitionsRDD]
| [[MapPartitionsRDD]] 调用类似 map 的操作的结果（例如 `map`、`flatMap`、`filter`、spark-rdd-transformations.md#mapPartitions[mapPartitions]）

| ParallelCollectionRDD.md[ParallelCollectionRDD]
| [[ParallelCollectionRDD]]

| ShuffledRDD.md[ShuffledRDD]
| [[ShuffledRDD]] "shuffle" 操作的结果（例如 spark-rdd-partitions.md#repartition[repartition] 或 spark-rdd-partitions.md#coalesce[coalesce]）

|===

== [[storageLevel]][[getStorageLevel]] StorageLevel

RDD 可以指定一个 storage:StorageLevel.md[StorageLevel]。默认的 StorageLevel 是 storage:StorageLevel.md#NONE[NONE]。

storageLevel 可以使用 <<persist, persist>> 方法指定。

在 <<unpersist, 取消持久化>>之后，storageLevel 再次变为 NONE。

当前的 StorageLevel 可以通过 `getStorageLevel` 方法获得。

[source, scala]
----
getStorageLevel: StorageLevel
----

== [[id]] 唯一标识符

[source, scala]
----
id: Int
----

id 是给定 <<_sc, SparkContext>> 中的*唯一标识符*（又名 *RDD ID*）。

在创建 RDD 时，id 会向 <<sc, SparkContext>> 请求 SparkContext.md#newRddId[newRddId]。

== [[isBarrier_]][[isBarrier]] Barrier Stage

RDD 可以是 spark-barrier-execution-mode.md#barrier-stage[barrier stage] 的一部分。默认情况下，当以下条件满足时，`isBarrier` 标志被启用（`true`）：

.. 在 <<dependencies, RDD 依赖>>中没有 [ShuffleDependencies](ShuffleDependency.md)

.. 至少有一个[父 RDD](Dependency.md#rdd) 启用了该标志

ShuffledRDD.md[ShuffledRDD] 的标志总是禁用的。

MapPartitionsRDD.md[MapPartitionsRDD] 是唯一可以启用该标志的 RDD。

== [[getOrCompute]] 获取或计算 RDD 分区

[source, scala]
----
getOrCompute(
  partition: Partition,
  context: TaskContext): Iterator[T]
----

`getOrCompute` 为 <<id, RDD id>> 和 [partition index](Partition.md#index) 创建一个 storage:BlockId.md#RDDBlockId[RDDBlockId]。

`getOrCompute` 请求 `BlockManager` 进行 storage:BlockManager.md#getOrElseUpdate[getOrElseUpdate] 以获取块 ID（使用 <<storageLevel, storage level>> 和 `makeIterator` 函数）。

NOTE: `getOrCompute` 使用 core:SparkEnv.md#get[SparkEnv] 访问当前的 core:SparkEnv.md#blockManager[BlockManager]。

[[getOrCompute-readCachedBlock]]
`getOrCompute` 记录是否...待完善（readCachedBlock）

`getOrCompute` 根据 storage:BlockManager.md#getOrElseUpdate[BlockManager] 的响应以及内部 `readCachedBlock` 标志现在是打开还是仍然关闭来分支。在任何情况下，`getOrCompute` 都会创建一个 spark-InterruptibleIterator.md[InterruptibleIterator]。

NOTE: spark-InterruptibleIterator.md[InterruptibleIterator] 只是简单地委托给包装的内部 `Iterator`，但允许[任务终止功能](../scheduler/TaskContext.md#isInterrupted)。

对于可用的 `BlockResult` 和 `readCachedBlock` 标志打开，`getOrCompute`...待完善

对于可用的 `BlockResult` 和 `readCachedBlock` 标志关闭，`getOrCompute`...待完善

NOTE: `BlockResult` 可以在本地块管理器中找到，或者从远程块管理器获取。它也可能刚刚被存储（持久化）。无论哪种情况，`BlockResult` 都是可用的（storage:BlockManager.md#getOrElseUpdate[BlockManager.getOrElseUpdate] 给出带有 `BlockResult` 的 `Left` 值）。

对于 `Right(iter)`（无论 `readCachedBlock` 标志的值如何，因为...待完善），`getOrCompute`...待完善

NOTE: storage:BlockManager.md#getOrElseUpdate[BlockManager.getOrElseUpdate] 给出 `Right(iter)` 值以指示块有错误。

NOTE: `getOrCompute` 在 Spark 执行器上使用。

NOTE: `getOrCompute` 专门在 RDD 被请求<<iterator, 分区中值的迭代器>>时使用。

== [[dependencies]] RDD 依赖

[source, scala]
----
dependencies: Seq[Dependency[_]]
----

`dependencies` 返回 RDD 的[依赖关系](Dependency.md)。

NOTE: `dependencies` 是一个 final 方法，Spark 中的任何类都不能覆盖它。

在内部，`dependencies` 检查 RDD 是否[被检查点](checkpointing.md)并相应地采取行动。

对于正在检查点的 RDD，`dependencies` 返回一个包含 [OneToOneDependency](NarrowDependency.md#OneToOneDependency) 的单元素集合。

对于未检查点的 RDD，使用 <<contract, `getDependencies` 方法>>计算 `dependencies` 集合。

NOTE: `getDependencies` 方法是一个抽象方法，自定义 RDD 必须提供。

== [[checkpointRDD]] 获取 CheckpointRDD

[source, scala]
----
checkpoint Option[CheckpointRDD[T]]
----

如果可用（如果 RDD 已检查点），checkpointRDD 从 <<checkpointData, checkpointData>> 内部注册表给出 CheckpointRDD。

checkpointRDD 在 RDD 被请求 <<dependencies, dependencies>>、<<partitions, partitions>> 和 <<preferredLocations, preferredLocations>> 时使用。

== [[isCheckpointedAndMaterialized]] isCheckpointedAndMaterialized 方法

[source, scala]
----
isCheckpointedAndMaterialized: Boolean
----

isCheckpointedAndMaterialized...待完善

isCheckpointedAndMaterialized 在 RDD 被请求 <<computeOrReadCheckpoint, computeOrReadCheckpoint>>、<<localCheckpoint, localCheckpoint>> 和 <<isCheckpointed, isCheckpointed>> 时使用。

== [[getNarrowAncestors]] getNarrowAncestors 方法

[source, scala]
----
getNarrowAncestors: Seq[RDD[_]]
----

getNarrowAncestors...待完善

getNarrowAncestors 在 StageInfo 被请求 [fromStage](../scheduler/StageInfo.md#fromStage) 时使用。

== [[persist]] 持久化 RDD

[source, scala]
----
persist(): this.type
persist(
  newLevel: StorageLevel): this.type
----

参考 spark-rdd-caching.md#persist[持久化 RDD]。

== [[persist-internal]] persist 内部方法

[source, scala]
----
persist(
  newLevel: StorageLevel,
  allowOverride: Boolean): this.type
----

persist...待完善

persist（私有）在 RDD 被请求 <<persist, persist>> 和 <<localCheckpoint, localCheckpoint>> 时使用。

== [[computeOrReadCheckpoint]] 计算分区或从检查点读取

[source, scala]
----
computeOrReadCheckpoint(
  split: Partition,
  context: TaskContext): Iterator[T]
----

computeOrReadCheckpoint 从检查点读取 `split` 分区（<<isCheckpointedAndMaterialized, 如果已经可用>>）或 <<compute, 自己计算它>>。

computeOrReadCheckpoint 在 RDD 被请求<<iterator, 计算分区的记录>>或 <<getOrCompute, getOrCompute>> 时使用。

== [[preferredLocations]] 定义 RDD 分区的放置偏好

[source, scala]
----
preferredLocations(
  split: Partition): Seq[String]
----

preferredLocations 请求 CheckpointRDD 获取 <<checkpointRDD, 放置偏好>>（如果 RDD 已检查点）或 <<getPreferredLocations, 自己计算它们>>。

preferredLocations 是一个模板方法，使用 <<getPreferredLocations, getPreferredLocations>>，自定义 RDD 可以覆盖它以指定分区的放置偏好。getPreferredLocations 默认不定义放置偏好。

preferredLocations 主要在 DAGScheduler 被请求 scheduler:DAGScheduler.md#getPreferredLocs[计算缺失分区的首选位置]时使用。

== [[partitions]] 访问 RDD 分区

[source, scala]
----
partitions: Array[Partition]
----

partitions 返回 `RDD` 的 spark-rdd-partitions.md[Partitions]。

partitions 请求 CheckpointRDD 获取 <<checkpointRDD, partitions>>（如果 RDD 已检查点）或 <<getPartitions, 自己查找它们>>并缓存（在 <<partitions_, partitions_>> 内部注册表中，下次使用）。

分区具有这样的属性：它们的内部索引应该等于它们在所属 RDD 中的位置。

== [[checkpoint]] 可靠检查点 -- checkpoint 方法

[source, scala]
----
checkpoint(): Unit
----

checkpoint...待完善

checkpoint 使用场景...待完善
