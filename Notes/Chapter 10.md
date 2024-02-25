# Capter 10: Batch Processing

## Derived Data

- Non Trivial applications require compositions of different systems to work together

### Systems of Records and Derived Data

- systems of records: are the systems where the true input data is present, every record is here exactly once
- Derived data systems: are the systems that take data from other systems to be processed in some way, but if it was lost, we can restore it again from the source like cache

Not all systems have a clear distinctions between system of records and derived data, they are tools, you decide how to use them

## Batch Processing

    A system cannot be successful if it is too strongly influenced by a single person. Once the initial design is complete and fairly robust, the real test begins as people with many different viewpoints undertake their own experiments.

- There are 3 types of systems:
  - Services (online systems): the client is usually a user, he sends a request and waits for a response, here the response time and availability are too important

  - Batch processing systems (offline systems): They use large data as input and compute the result in a long time (minutes to days), then it returns the output, usually the user doesn't wait for it to finish

  - Stream processing systems (near-realtime systems): They are systems between the two, they take large data and process it but it returns the output once it can be returned, it doesn't require the whole set of data to be available to start

For Example MapReduce algorithm which was called "The algorithm that makes google so massively scalable"
It was implemented in various open source data systems like: MongoDB, Hadoop, CouchDB

In this chapter we will look into MapReduce and other batch processing algorithms

## Batch processing with Unix tools

- Surprisingly many data analyses can be done in a few minutes using some combination of awk, sed, grep, sort, uniq, and xargs, and they perform surprisingly well

### Chain of command versus custom programs

- linux tools can handle large data that don't fit in memory, they dot it like `LSM-Trees`

### The Unix Philosophy

Linux was built on these 4 rules in 1978:

- Make each tool do only one job well.
- Expect the input of each tool to be the output of the other
- Design everything to be tried early, then update it
- Use tools instead of unskilled help

These steps looks like the Agile and DevOps movement of today

The sort tool is a great tool that does one thing very well. it is better than the sort function in most programming languages as it uses disk to sort large sets

### A uniform interface

- To make these tools input use other tool's output we must make them all compatible the the same input/output format, which is files in this case

The only downside of linux tools that they work only on one machine, and that's why tools like Hadoop come in

### Separation of logic and writing

- Linux tools works with stdin/stdout, it doesn't matter how would you deal with them, it is guaranteed that the next input/output will be in the same format

### Transparency and experimentaion

- You can stop the pipeline at any point
- The tools doesn't affect the input files and they use a small memory buffer

## Mapreduce and Distributed File Systems

- MapReduce is like the unix tools, except it is distributed across thousand of machines
- It doesn't have side effects on the input, it only produce the output
- Instead of stdin/stdout, MapReduce uses Hadoop distributed file system, HDFS
- There are various filesystems like ClusterFS and Quantcast CFS, object storage services like Amazon S3, Azure, Blob Storage and OpenStack Swift
- In this chapter we will use HDFS but the principles applies to any DFS
- HDFS is based on the Shared-Nothing principle
- HDFS creates one big file system that uses all the disk spaces in all machines
- The biggest HDFS deployments run on tens of thousands of computers

### MapReduce Job execution

- The tasks we did in the linux example were partitioned in 4 steps
  - Read the data and separate it into files
  - map it, in our case `{page $7}`
  - sort it and then send it to step 4
  - reduce it and output the value `uniq -c`

- Actully MapReduce does this, but the user writes custom code for steps 2 and 4 only, MapReduce does the other tasks internally

- The MapReduce framework has 2 functions
  - Mapper: it takes the keys and values from the input
  - Reducer: it takes the key-value pairs and iterate on them

- The role of the mapper is to put the data in suitable way to be sorted, and the role of the reduce it to process the sorted data

### Distributed execution of MapReduce

- The main difference from pipelines of Unix tools is that MapReduce can run the code in many machines without requiring you to worry about the parallelism, you only take one row, you don't need to know from where it comes or where it goes to

- In Hadoop, the mapper and reducer are each a Java class, in MongoDB and CouchDB they are Javascript functions

- The mappers sort their batches of data then they are merged using something like merge sort

- Then each key-value pair knows to which reducer it should go, based on the `Hash of the key` partitioning

### MapRedcue workflows

- A single MapReduce job can do limited tasks, we need to chain a jon to another to get more power, Hadoop doesn't support the workflows, but it does it internally by directory name

- A batch job's output is considered valid only when all subtasks are successfull

- There are many workflows that has been developed for Hadoop, like Oozie, Azkaban, Luigi, Airflow and Pinball

- There are some higher-level tools for Hadoop like: Pig, Hive, Cascading, Crunch and FlumeJava

### Reduce-Side Join and Grouping

- In chapter 2 we discussed data models and query languages but we didn't talk about how joins internally work. We will talk about this thread again
- A naive approach to join tables is to go for events table one by one an query the user table, but its performance is bad
- A good approach is to take a copy of the data using ETL process and use MapReduce to make the join

### Sort-merge joins

- Sorting the data by user id
- Reducing every user id alone
- It only needs the user id to be stored in memory

### Bringing related data together in the same page

- MapReduce separates the application logic from distributing the data to the appropriate devices

### GROUP BY

- In addition to joins, we can implement GROUP BY with MapReduce
- It can be reduced using MapReduce, and the final value will be processed inside the MapReduce

### Handling skew

- It can happen that one node processes many more than the others, imaging a user_id for a celiberity (remember hot spots?)
- There are few algorithms to deal with hot spots in MapReduce

## Map-Side Joins

- In the last section we talked about algorithms to do th joins in the reduce side, but it has sorting, copying and merging processes, but it doesn't assume anything about inputs
- In this section we will see how we can make joins in the map side

### Broadcast hash joins

- It is the simplest way
- It can be implemeneted if the small dataset can fit into memory
- The small dataset is stored in a hash table
- It is so effective
- It is supported by Pig (Replicated Join), Hive (MapJoin), Cascading and Crunch. It is used in a WH like Impala
- There is another approach instead of broadcasting the hash table to all joins, adding the small dataset in a read-only index in the local desk and depend on cache

### Partitioned hash joins

- It partitions the hash table over the partitions
- It assumes the both of the join's inputs have the same number of partitions
- It is implemeted in Hive(bucket map joins)

### Map-Side merge joins

### MapReduce workflows with the map-side joins

## The Output of Batch Workflows

- What we do with the result of the MapReduce workflow?
- It is not an OLTP nor an OLAP
- Here are some applications of it

### Building Search Indexes

- Google origin use of MapReduce was for search index
- It was a workflow of 5-10 MapReduce jobs
- Although Google moved away from using MapReduce, it is useful to understand its advantages
- We sas how full-text search indexes like Lucene works in chapter 3
- If you want a such partitioned search index, MapReduce is a good choice
- If the data doesn't change, then it should be a read-only index
- If the data doesn't change, then it should be a read-only index - If not, then a simple solution is to periodically rerun the indexing process
- Alternative, it is possible to build indexes incrementally like B-Tree

### Key-value stores as a batch process output

- Another common use of batch process outputs is machine learning systems llike classifiers and recommendation systems

### Philosophy of batch process outputs

## Comparing Hadoop to Distributed Systems

### Diversity of storage

### Diversity of processing models

### Designing for frequent faults

## Beyond MapReduce

- There are many tools other than MapReduce
- We start with MapReduce because it is a useful learning tool
- Using MapReduce is actually hard, you will need to implement every join from scratch
- MapReduce also has some performance issues
- We will look into some alternatives of MapReduce in the rest of the chapter

### Materialization of Intermediate State

- MapReduce is separated from the other tasks
- The only connect it the input and the output
- The files created to be moved from one process to the other are in `intermediate state`
- The process of writing these intermediate state of file is called `materialization`
- MapReduce fully materializing intermediate state has downsides compared to Unix pipes:
  - A MapReduce job must wait for all preceding tasks to finish
  - Mappers are often redundant
  - Storing intermediate sates in a distributed file system is an overkill

### Dataflow engines

- Engines developed to solve these issues: Spark, Tez, and Flink
- They have differences but they all handle the entire workflow as one job
- We call the functions of these engines `operators`

### Fault tolerance

### Discussion of materizlization

### Graphs and Iterative processing

- Graph-like models can use batch processing too, like the Pagerank and recommendation system
- Processing graphs with MapReduce is possible but not efficient

### The Pregel processing model

### High-Level APIs and Languages

### The move towards declarative query language
