# SparkUI learning

## In the first code, we observe 5 jobs
- Job 0 consists of a single stage, which includes three tasks:
    - Scanning the CSV file with Spark
    - Optimizing SQL into bytecode
    - Applying transformations to all partitions
- Job 1 also consists of a single stage, which includes four tasks:
    - Scanning the CSV file
    - Deserializing it into DataFrame objects (as Rows)
    - Executing the Spark SQL query as an RDD
    - Applying transformations to all partitions
- Job 2 also consists of a single stage, which includes the same three tasks as in Job 0, but they are not as low-level in Java since our Python code is executed.
    - Scanning the CSV file
    - Optimizing SQL into bytecode
    - Applying transformations to all partitions
    - We see that a Shuffle Write occurred for the next job
- Job 3 skips a stage that was already executed in the previous job. We see that the data read from the disk was written by the previous job.
    - Data exchange operation between partitions on different execution nodes
    - Optimization: Instead of executing each operation separately, Spark generates a single optimized Java code block
    - Shuffle process: Moving data between nodes. After this, Spark rearranges the rows so that - all identical categories are in the same partition
    - Again, we see data written to disk for the next job
- Job 4 skips two stages, reading data from the disk that was written by the previous job.
    - Adaptive partition balancing
    - Optimization: Generation of adaptive Java code
    - Internal partition optimization

## In the second code, where an intermediate action collect is added, we observe 8 jobs
Almost 3 additional jobs were created to execute this action:
- Jobs 0 and 1 remain the same as in the previous case.
- Jobs 2, 3, and 4 are the same as in the previous code, but they correspond to executing the added intermediate collect action.
- Jobs 5, 6, and 7 are similar to the previous ones but correspond to the final collect action.

## In the third code, where cache is used, we observe 7 jobs
- Jobs 0 and 1 remain the same as in the previous case.
- Jobs 2, 3, and 4 are also similar to the previous code, but one additional task is added at the end of Job 4:
    - Internal partition optimization with a list of executed code. This is likely a caching process or an optimization step immediately after caching.
Job 5 skips two stages:
    - Exchange - Spark redistributes (shuffles) data between partitions
    - WholeStageCodegen - Generates a single block of Java code
    - MapPartitionInternal - Processes data at the partition level
    - MapPartitionInternal (cache) - Processes cached data
    - InMemoryTableScan (cache) - Reads data from the Spark cache
    - MapPartitionInternal - Re-applies transformations on cached partitions after reading
- Job 6 is similar to the previous one and also skips two stages:
    - Exchange - Spark redistributes (shuffles) data between partitions
    - WholeStageCodegen - Generates a single block of Java code
    - MapPartitionInternal - Processes data at the partition level
    - MapPartitionInternal (cache) - Processes cached data
    - InMemoryTableScan (cache) - Reads data from the Spark cache
    - WholeStageCodegen - Generates a single block of Java code, likely for executing our computations (.where("count>2"))
    - MapPartitionInternal - Re-applies transformations on cached partitions after reading
