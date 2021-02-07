# ContinuousDataSourceRDD

`ContinuousDataSourceRDD` is a specialized `RDD` (`RDD[InternalRow]`) that is used exclusively for the only input RDD (with the input rows) of `DataSourceV2ScanExec` leaf physical operator with a [ContinuousReader](continuous-execution/ContinuousReader.md).

`ContinuousDataSourceRDD` is <<creating-instance, created>> exclusively when `DataSourceV2ScanExec` leaf physical operator is requested for the input RDDs (which there is only one actually).

[[spark.sql.streaming.continuous.executorQueueSize]]
`ContinuousDataSourceRDD` uses [spark.sql.streaming.continuous.executorQueueSize](configuration-properties.md#spark.sql.streaming.continuous.executorQueueSize) configuration property for the <<dataQueueSize, size of the data queue>>.

[[spark.sql.streaming.continuous.executorPollIntervalMs]]
`ContinuousDataSourceRDD` uses [spark.sql.streaming.continuous.executorPollIntervalMs](configuration-properties.md#spark.sql.streaming.continuous.executorPollIntervalMs) configuration property for the <<epochPollIntervalMs, epochPollIntervalMs>>.

[[creating-instance]]
`ContinuousDataSourceRDD` takes the following to be created:

* [[sc]] `SparkContext`
* [[dataQueueSize]] Size of the data queue
* [[epochPollIntervalMs]] `epochPollIntervalMs`
* [[readerInputPartitions]] ``InputPartition[InternalRow]``s

[[getPreferredLocations]]
`ContinuousDataSourceRDD` uses `InputPartition` (of a `ContinuousDataSourceRDDPartition`) for preferred host locations (where the input partition reader can run faster).

=== [[compute]] Computing Partition -- `compute` Method

[source, scala]
----
compute(
  split: Partition,
  context: TaskContext): Iterator[InternalRow]
----

NOTE: `compute` is part of the RDD Contract to compute a given partition.

`compute`...FIXME

=== [[getPartitions]] `getPartitions` Method

[source, scala]
----
getPartitions: Array[Partition]
----

NOTE: `getPartitions` is part of the `RDD` Contract to specify the partitions to <<compute, compute>>.

`getPartitions`...FIXME
