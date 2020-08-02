---
title: Spark stateful streaming processing is stuck in StateStoreSave stage!
date: 2020-07-07 20:46:59
tags: [Databricks, Spark, Stream processing, Azure Blob Storage, Azure Data Lake Storage Gen2]
categories: Spark
description: A stateful structured stream processing job is suddenly stuck at the 1st micro-batch job. Here are the notes about that issue, how to debug Spark stateful streaming job, and also how I fix it.

---

# Spark stateful streaming processing is stuck in StateStoreSave stage!

A stateful structured stream processing job is suddenly stuck at the 1st micro-batch job. Here are the notes about that issue, how to debug Spark stateful streaming job, and also how I fix it.

## Stateful Structured Streaming Processing Job

```python
 ## specify data source, read data from Azure Event Hub
(spark.readStream.format("eventhubs") 
 .options(**self.config.ehConfig)
 .load()  
 ## use watermark to control state size
 .withWatermark(processTimeCol, waterMarkTime)
 ## transformations. 
 .withColumn(eventTimeCol, col(eventTimeCol).cast('timestamp'))
 ## aggregation by event time windows
 .groupBy(col(key), window(eventTimeCol, "15 mins"))
 .agg()
 ...
 .select(*(cols+['windowStart', 'firstEventTime', 'lastEventTime', 'count']))
 ## specify data sink, write transformed output to Azure blob storage
 .writeStream
 .format("parquet")
 .option('path', outputPath)
 .outputMode("append")
 ## Processing details--Trigger: when to process data
 .trigger(processingTime="2 seconds")  
 ## Processing details--Checkpoint: for tracking the progress of the query
 .option("checkpointLocation", checkpointPath)
 .start())
```

Spark SQL converts batch-like query to a series of incremental execution plans operating on new micro-batches of data.

![image-20200706163510103](C:\Users\tgttx\Documents\xinyeah.github.io\source\images\image-20200706163510103.png)



## Environment

This Streaming job is running on **Databricks** clusters triggered by **Azure Data Factory** pipeline.

Data source is from **Azure Event Hub**, and this job store the aggregated output to **Azure Blob Storage** mounted on **Databricks**.

**Databricks** instance has **Vnet injections** and have **NSG** associated with the Vnet.

Databricks cluster version is 5.5 LTS which use **Scala** 2.11, **Spark** 2.4.3 and **Python** 3. 

We also use **PySpark** 2.4.4 for this streaming job.



## Issue Symptom

This streaming job is scheduled to run for 4 hours every time. When the current job stops, the next one will start to run. It means the max number of concurrency job is 1. 

It was running well before. Suddenly, when a new streaming job starts, it seems to stuck at the 1st micro-batch like the following picture. You can see the job is stuck for batch=0 and it fails because of time out.

We have 3 regions and this issue happened to every region one by one in a week.

![image-20200706171557385](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200706171557385.png)

![image-20200706171822384](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200706171822384.png)



If you check the checkpoint folder:

![image-20200706172128040](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200706172128040.png)

compare with the normal checkpoint:

![image-20200706172305651](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200706172305651.png)

The difference is obvious: there is no commits in the checkpoint path. It means the streaming job didn't succeed in processing even a single micro-batch.



## Possible Causes

It is really tough to debug this issue because no error message shown up and just several misleading warning messages in the executor's error logs.

Usually, there are several possible reasons to cause streaming processing job stuck, such as:

- **Total size of state per worker** is too large which leads to higher overheads of snapshotting and JVM GC pauses.

- **Number of shuffle partitions** is too high, so the cost of writing state to HDFS will increase which cause the higher latency.

- **NSG rules** added to Databricks Vnet might block some ports and thus infect worker to worker communication.

- **Databricks mounted blobs are expired** and need to rotate the storage connection string and databricks access token.

- **Databricks cluster version** is deprecated and not supported any more.

- **Spark .metadata directory** is messed up. We need to delete the metadata and let the pipeline recreate a new one. but for this one, it would complete micro-batch, and it just do nothing in the process.

  

  But **none of them work** this time. We struggle to figure out the **root cause** is:

- **Azure blob storage has too many files in the checkpoint folder** which slow down the read and write speed. 

I compare the DAG visualization with normal job's, it is shown that the StateStoreSave stage takes much longer (16 hours) than the normal one (21 seconds). StateStoreSave is the stage when spark store current streaming process status in checkpoint. Thus the issue exists in checkpoints. More info for StateStoreSave can be found [here](https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-StateStoreSaveExec.html)

![image-20200706161548770](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200706161548770.png)

From this detailed stage information, we can get:

1. **number of total state rows** is not the concern. we cannot solve the issue by reduce the watermark threshold or recreate checkpoint.

2. **memory used by state total** is lower than the normal state. so it is not a JVM GC pause issue.

3. **time to update total** is the pain point. It takes longer even the number of updated state rows is less, which point the issue to the **write speed** in blob storage. 

   In the checkpoint folder, we stored around 17 million checkpoints for each region. After I delete the whole checkpoint folder and restart the streaming job, the issue is fixed. I am not 100% sure about the reason for it. One possible reason for it is Azure blob storage doesn't support **hierarchical namespace**, and it just mimic hierarchical directory structure by using slashes in the name. 

## Solutions

### Solution 1 - migrate Azure blob storage to Azure Data Lake Gen2 which supports hierarchical namespace.

### Solution 2 - delete checkpoint folder and decrease retention period.

Step1: Stop current streaming job

Step2: Delete .metadata directory and checkpoint folder

Step3: Add a failover mechanism so that the streaming job will resume from where the streaming job stopped in the last successful data persistence.

Here is my failover mechanism if no checkpoint found, so we can delete the checkpoint folder without losing state.

```python
	try:
        # if the checkpoint exists, continue to use it without refreshing
        config.dbutils.fs.ls(config.checkpointPath)
        print('Continue streaming job with checkpoint path, %s' %
              config.checkpointPath)
    except:
        # remove _spark_metadata folder when use a new checkpoint
        config.dbutils.fs.rm(config.outputPath + '_spark_metadata', True)
        print('removed _spark_metadata for last checkpoint')
        print('New streaming job checkpoint path is %s' %
              config.checkpointPath)
        # set the streaming start time to catch up from where the streaming job stoped in last data persistence
        timeKey = 'windowStart'
        try:
            ts = [int(p.path.split(timeKey + "=")[1][:-1])
                  for p in config.dbutils.fs.ls(config.outputPath) if timeKey in p.path]
            if ts:
                lookbackTs = int(dt.datetime.now().replace(minute=0, second=0, microsecond=0).timestamp()) - defualtLookbackTime
                # Set stream start time to the maximum of (15min aggregation output timestamp or one day back from current time)
                streamStartTime = np.max([np.max(ts), lookbackTs])
                # Add 15min to start time
                streamStartTime = dt.datetime.fromtimestamp(
                    streamStartTime) + dt.timedelta(minutes=15)
                streamStartTime = streamStartTime.strftime("%Y-%m-%dT%H:%M:%S.%fZ")
                # Create the positions
                startingEventPosition = {
                    "offset": None,
                    "seqNo": -1,
                    "enqueuedTime": streamStartTime,
                    "isInclusive": True
                }
                config.ehConfig["eventhubs.startingPosition"] = json.dumps(
                    startingEventPosition)
         except:
            pass
```

Step 4: restart the streaming job.

Step 5: Add retention policy to the checkpoint folder to decrease the checkpoints lifetime.