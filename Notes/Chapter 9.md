# Chapter 9: Consistency and Consensus

    Is it better to be alive and wrong or right and dead?

## Introduction

- Many things can go wrong in distributed systems, the simplest way is to let the entire service fail, but how to tolerate faults?

- In this chapter we will see algorithms and protocols for fault-tolerance

- The best way is to find some general-purpose abstractions with useful guarantees, code them once, and then let the applications rely on them, like we did with transactions

- We need to know the limits of what can and what cannot be done, this has been researched in depth by theoritical proofs and practical implementations

## Consistency Guarantees

- All replicated systems at least provide the `eventual consistency` guarantee, which means that all replicas will converge to the same value at some time

- But this is a very weak guarantee because it doesn't say when the data will converge

- When working with databases that provides eventual consistency only, it becomes difficult to deal with it, because it is not like the variables in single-threaded program

- The bugs is hard to find by testing because the application will perform well most of the time

- We will discover stornger algorithms, but it doesn't come for free, it comes with performance cost, or less fault-tolerance

- It is like the isolation levels of transactions but they are separated
  - in transaction: it solves the race condition problem
  - in consistency: it solves the replication delays and fualts

## Linearizability

- When working with eventual consistency, the application expects to read different data from two replicas at the same time

- Isn't it better to assume that there is only one replica (one copy of the data)?

### What make a system linearizable?

- The row in relational database or the key in a key-value database is called a register

- After one read reads the value, all preceding values should be the same read

### Relying on linearizability

- Locking and leader election
- Constraints and uniqueness guarantees
- Cross-channel timing dependencies

### Implementing linearizable systems

- In single-leader replication it is possible to make it linearizable by reading from the leader or syncronous nodes, but not all of them can be linearizable because of the design(e.g. using snapshot isolation), or concurrency bugs

- Consensus algorithms can apply linearizability

- Multi-leader replication is generally not linearizable

- Leaderless replication is probably not linearizable because of the unreliable clocks

- Lineaizability and qouroms, it doesn't mean that dynamo style systems are linirizable just because they use quoroms

- as a summary, you can suppose that dynamo-style database are not linearizable

### The cost of linearizability

- The CAP theorem
  - If you require linearizability, it means when the network is disconnected, then you must wait or return an error
  - If you don't require linearizability it mean when the network is disconnected, then you can still process the requests but its behaviour is not linearizable  

- Linearizability and network delays

## Ordering Guarantees

### Ordering and cuasality

- Ordering preserve causality
- The causal order is not a total order
  - totally ordered means it can be compared to see who is greater, numbers are totally ordered but mathematical sets are not
- Linearizability is stronger than causal consistency
  - linearizability is not the only way to preserve causality
  - Researchers are exploring new kinds of databases that preserve causality with performance and availability like evenutual consistency
- Capturing causal dependencies
  - To preserve causality we need to make the operations in a causal order, and the concurrent operations doesn't matter

### Sequence number ordering

- Although causality is an important theoritical concept, keeping track of all the dependencies is impractical
- There is a better way like sequqnce numbers, they are not clock orders, but logical orders
- this way will keep a total order of the operations

- Noncausal sequence number generator
  - Each node can generate its own independent sequence numbers
  - attach timestamps to each operation
  - you can preallocate blocks of sequence numbers
    >The problem here is that they are not consistent with causality

- Lamport timestamps
  - It is a simple method that preserve causality
  - The sequence number consists of the order and the node id
  - the sequence number is always sent with the request
  - when a node receives the request, it increases it

- Timestamp ordering is not sufficient
  - Although Lompart timestamps provides total order but it doesn't solve many problems in distributed systems
  - e.g: when two users want to create the same username in the same time
  - to solve this problem we need to check *total order broadcast*

### Total order broadcast

- It is a protocol for exchaning messages between nodes, which requires that two safety properties to be satisfied
  - reliable delivery: if a message is delivered to one node, it is delivered to all nodes
  - totally ordered delivery: messages is delivered to all nodes in the same order

- using total order broadcast
  - Consensus services like `ZooKeeper` and `etcd` actually implement total order broadcast
  - if every message represents a write to database and every replica processes the same writes in the same order, then the replicas will remain consistent with each other, this is called `state machine replication`, we will return to it in chapter 11
  - the sequence number in ZooKeeper is called `zxid`

- implementing linearizable storage using total order broadcast
  - append a message to the log
  - read the log and wait for your appended message
  - check for any messages claiming the same username, if it is yours, then commit
  - the writes are consistent but the reads are not, but there are some ways to solve this:
    - append read message to log, like in etcd
    - you can sync till the needed message like ZooKeeper
    - you can read from a synced replica
- implementing total order using linearizable storage

## Distributed Transactions and Consensus

- consensus is not an easy task in distributed systems
- many broken systems was built in the mistaken belief that this problem is easy to solve
- There are some situations that we need all the nodes to agree
  - Leader election
  - Atomic commit

### Atomic Commit and Two-Phase Commit (2PC)

- From single node to distributed atomic commit
- The commit is done in two phases (here comes the name)
- The first phase is to check if the commit can be done in all nodes
- The second phase is to commit the commit to all nodes

- A system of promises, the details of the process
  - requesting the transaction ID from coordinator
  - The transactions is done in all nodes with this ID
  - If one failed, the coordinator tells the others to abort
  - If all of them succeeded, the coordinator tells all the nodes to commit
  - The coordinator writes the result in its log first, to deal with crashes
  - if the request fails, it must try forever
  - when the coordinator fails, the nodes must wait also forever, there cannot be a timeout

- Three-Phase Commit
  - 2PC is called blocking commit, because the fact that it waits for the coordinator
  - It is possible to make an atomic commit *nonblocking*
  - 3PC algorithm was proposed, but it assumes a network with bounded delays and nodes with bounded response times, but in practice they are unbounded
  - So, 2PC continues to be used

### Distributed Transactions in Practice

- 2PC is reliable but it has performance issues, so many cloud services decied not to implement distribued transactions
- Distributed transactions in MySQL is reported to be over 10 times slower than the one node transactions
- There are two different types of transactions
  - Database-internal distributed transactions
    - Internal nodes in the same engine like: VoltDB and MySQL
  - Heterogeneous distributed transactions
    - Two or more different technologies

- Exactly-one message processing
- XA transactions
- Holding locks while in doubt
- Recovering from coordinator failure
- Limitations of distributed transactions

### Fault-Tolerant Consensus

- 