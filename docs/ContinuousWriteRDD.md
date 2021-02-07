# ContinuousWriteRDD -- RDD of WriteToContinuousDataSourceExec Unary Physical Operator

`ContinuousWriteRDD` is a specialized `RDD` (`RDD[Unit]`) that is used exclusively as the underlying RDD of `WriteToContinuousDataSourceExec` unary physical operator to <<compute, write records continuously>>.

`ContinuousWriteRDD` is <<creating-instance, created>> exclusively when `WriteToContinuousDataSourceExec` unary physical operator is requested to <<WriteToContinuousDataSourceExec.md#doExecute, execute and generate a recipe for a distributed computation (as an RDD[InternalRow])>>.

[[partitioner]]
[[getPartitions]]
`ContinuousWriteRDD` uses the <<prev, parent RDD>> for the partitions and the partitioner.

[[creating-instance]]
`ContinuousWriteRDD` takes the following to be created:

* [[prev]] Parent RDD (`RDD[InternalRow]`)
* [[writeTask]] Write task (`DataWriterFactory[InternalRow]`)

=== [[compute]] Computing Partition -- `compute` Method

[source, scala]
----
compute(
  split: Partition,
  context: TaskContext): Iterator[Unit]
----

NOTE: `compute` is part of the `RDD` Contract to compute a partition.

`compute` requests the `EpochCoordinatorRef` helper for a <<EpochCoordinatorRef.md#get, remote reference to the EpochCoordinator RPC endpoint>> (using the <<ContinuousExecution.md#EPOCH_COORDINATOR_ID_KEY, __epoch_coordinator_id local property>>).

NOTE: The <<EpochCoordinator.md#, EpochCoordinator RPC endpoint>> runs on the driver as the single point to coordinate epochs across partition tasks.

`compute` uses the `EpochTracker` helper to <<EpochTracker.md#initializeCurrentEpoch, initializeCurrentEpoch>> (using the <<ContinuousExecution.md#START_EPOCH_KEY, __continuous_start_epoch>> local property).

[[compute-loop]]
`compute` then executes the following steps (in a loop) until the task (as the given `TaskContext`) is killed or completed.

`compute` requests the <<prev, parent RDD>> to compute the given partition (that gives an `Iterator[InternalRow]`).

`compute` requests the <<writeTask, DataWriterFactory>> to create a `DataWriter` (for the partition and the task attempt IDs from the given `TaskContext` and the <<EpochTracker.md#getCurrentEpoch, current epoch>> from the `EpochTracker` helper) and requests it to write all records (from the `Iterator[InternalRow]`).

`compute` prints out the following INFO message to the logs:

```
Writer for partition [partitionId] in epoch [epoch] is committing.
```

`compute` requests the `DataWriter` to commit (that gives a `WriterCommitMessage`).

`compute` requests the EpochCoordinator RPC endpoint reference to send out a <<EpochCoordinator.md#CommitPartitionEpoch, CommitPartitionEpoch>> message (with the `WriterCommitMessage`).

`compute` prints out the following INFO message to the logs:

```
Writer for partition [partitionId] in epoch [epoch] is committed.
```

In the end (of the loop), `compute` uses the `EpochTracker` helper to <<EpochTracker.md#incrementCurrentEpoch, incrementCurrentEpoch>>.

In case of an error, `compute` prints out the following ERROR message to the logs and requests the `DataWriter` to abort.

```
Writer for partition [partitionId] is aborting.
```

In the end, `compute` prints out the following ERROR message to the logs:

```
Writer for partition [partitionId] aborted.
```
