# Chapter 6: partitioning

    Clearly, we must break away from the sequential and not limit the computers. We must state definitions and provide for priorities and descriptions of data. We must state relationships, not procedures.

## Introduction

- In chapter 5, we discussed storing the same copy of the whole database
- This will not be suffecient in large data sets
- We will need to break the data up into partitions (Sharding)
- The main reason for wanting to partition data is scalability
- Horizontal scalling is cheaper and has a batter performace than vertical scalling

## Partitioning and Replication

- Partitioning is usually combined with replcation
- Each node has part of the data (partitioning)
- Each node is copied in other nodes (Replication)

## Partitioning of Key-Value data

- If every node take a fair share, then 10 nodes will handle 10 times the data, and handle 10 times reads and writes
- If some node has more data than the others, we call the partition *skewed*, and this partition is called *hot spot*
- The simplest approach is to set the data randomly in the nodes, but its disadvantage is that when you want yo read, you will send requests to all the nodes

### Partitioning by Key-Range

- Suppose the keys are names of persons in books
- Each partition has a range (minimum and maximum) of letters
- The ranges can be chosed manually and automatically
- This can be very useful for timestamps keys
- Because of the access patterns, there may be hotspots, like when you are sending the data to this day, and all the partitions are idle
- You can put the sensor name as a prefix to the timestamp partitioning

### Partitioning by Hash of Key

- Beucase of this riks of skewed partitions, many datastores use a hash function to determine the partition for a given key
- Every partition will store range of hashes instead of range of keys
- We lose the ability to do efficient range queries
- Range queries on the primary key are not supported by Riak, Couchbase, Voldemort
- Cassandra achieves a good compromise between the two approaches, such that if you fixed the primary key, you can range queries the other keys

### Skewed Workloads and Hot Spots

- Hashing the keys is a good idea to reduce hot spots, but it doesn't remove them entirely
- Like the celeberity with millions of follower, when he does an action
- It is a responsiblity of the application developer to fix this issue, for example by setting a [0-100] random key as a prefix of the ID when the user has many followers
- this will force writes to search in all the 100 paritions, so it is a good idea to make this only for the keys that are hot

## Partitioning and Secondary Indexes

- The last partitioning schemes are for primary keys, but when we need to use secondary indexes, the situation becomes more complicated
- HBase and Voldemort have not added the secondary indexes for this complication
- There are two approaches for handling secondary indexes in partitions

### By Document

- Every partition is treated as a separate document by the primary key
- Then each secondary index is a separate index in this document
- This approach is efficient in writing
- But in reading you will send the query to run in all partitions if you look without the primary key
- But it is widely used: MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud and VoltDB all use it

### By Term

- There will be global index for the scondary index and it will be distributed in the partitions
- The write will be fast, but the read will be more compllicated and partitioned
- Its implementation will be discussed in chapter 12

## Rebalancing Partitions

- Overtime databases change
  - You need to add more CPUs
  - You need to add more disks
  - You need to replace other nodes
- Rebalancing minimum requirements:
  - Load should be fairly distributed among nodes after the rebalancing
  - While rebalancing, database should accept reads and writes
  - No more data than necessary should be moved

### Strategies for rebalancing

    There are few ways to assign partitions to nodes

- How not to do it: hash mod N
  - It is a good approach, but it moves most of the keys when `N` changes

- Fixed number of partitions:
  - There is a simple solution to make many more partition than nodes
  - For example a 100 partition per node
  - Then, you can move only some partitions to the created node
  - The same for removing
  - This approach is used in Riak, Elasticsearch, Couchbase, Voldemort

- Dynamic partitioning
  - When a partition grows, it is split up into two partitions like B-Tree
  - It is used in HBase and RethinkDB
  - At first, when the database is small, it start with one node, so it is a hot spot now
  - To solve this, HBase and MongoDB set initial set of partitions on empty database

- Partitioning proportionally to nodes
  - In dynamic partitioning the size of partitions is related to the size of the dataset
  - In Cassandra and Ketama the size of the partitions is proportional to the number of nodes

### Atomic or Manual Rebalancing

- There are automatic rebalancing but it is a good idea to keep a human on the top levels

## Request Routing

- When I want to read or write the key "foo", which partition should I connect to?
- It is an instance of a more larger problem called service discovery
- On high level there are a few differnt approaches:
  - Allow clients to connect to any node, then this node routes
  - Send all request to routing tier first
  - Require the client to be aware of the partition
- In all cases, the key problem is how the dicision is made
- Many distributes systems rely on a separate coordination service like ZooKeeper
- LinkedIn's Expresso uses Helix for cluster management (It relies on ZooKeeper)
- HBase, SolrCloud and Kafka rely on ZooKeeper
- MongoDB has its own architicure
- Cassandra and Riak use a different approach: `gossip protocol` (The first approach)
- Couchbase doesn't rebalance automatically

### Paraller Query Execution

- MPP `Massively parallel processing`
- Large queries can use it to process all the data in the requested partitions in parallel
