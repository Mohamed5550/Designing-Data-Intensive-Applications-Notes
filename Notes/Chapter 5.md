# Chapter 5: Replication

    The major difference between a thing that might go wrong and a thing that cannot possibly go wrong is that when a thing that cannot possibly go wrong goes wrong it usually turns out to be impossible to get at or repair.

- Reasons of using replication
  - putting data geographically near to request source
  - Having another copy of the data to restore on failure
  - Scaling the data into multiple machines

## Leaders and Followers

- Each node storing a copy of the database is called replica
- Each write goes to leader then it tells followers the new changes
- Each read can be done on leader or on any follower
- Follower are read-only from the view point of clients

### Syncronous and Asyncronous Replication

- First follower is syncronous but the others are asyncronous
- Replications is fast
- When you turn on syncronous it means one follower will be syncronous
- If the syncronous becomes unavailable or slow, one of the asyncronous becomes syncronous
- This configuration is called semi-syncronous

### Setting Up New Followers

- Copying one node will produce different data at different points of time
- Locking the database until the process is finished is not a good solution for availability
- The best approach
  - Taking a snapshot of the leader
  - Copying snapshot to the new node
  - The new node requests from the leader the changes that happended since the snapshot was taken
  - it is now caught up and can continue working

### Handling Node Outages

- It is a good advantage to be able to recover the node when it fails or when we restart the server
- For a follower, we depend on log to restore and catch up
- For a leader, we tell one of the followers to be leader
  - It can be done manually or automatically
  - The new leader is chosen by election
  - The old leader beomes a follower when it returns back
  - there are some problems in the failover process
  - operations team prefers doing it manually

### Implementation Of Replication Logs

- Statement replication
  - Sending the full statment to the follower
  - There are some problems like:
    - the methods that return standalone values like RAND(), NOW()
    - The conditions on the queries may get different data if the replications is not written in the same order
    - the triggers may produce different side effects
  - There can be work arounds for these issues but there are many edge cases
  - So this method is not prefered

- Write-ahead log (WAL) shipping
  - We save logs of the bytes to write to the follower
  - this requires the same datastructure to berebuilt
  - this needs too much low-level implementation

- Logaical (row-based) log replication
  - this sends the rows to be changes
  - on insert it sends the new row
  - on delete it sends the primmary key
  - on update it sends the primary key and the new updated columns

- Trigger-based replication

### Problems with Replications Lag

- asyncronous is fast but may read outdated information
- syncronous is reliable but may be slow, and stops if one node fails

#### Reading Your Own Writes

- The user needs to read his own writes at many applications like the comment post action
- If there is a lag he may not see his data
- There are some techniques for solving this issue:
  - Making some requests always read from the leader, like reading own profile data
  - Reading from the leader for 1 minute after the last update
  - Adding a logical timestamp
- There will be other issues if the user has more than one device
  - May be solved by making all the user devices requests go to the same datacenter

#### Monotonic reads

- It can be happened that the user reads from more than one replica with different lag time
- If he reads from one with lower lag then from one with heigher lag, he may see things go backwards in time
- This can be solved by making every user always read from one replica

#### Consistent Prefix Reads

- This guarantee says if the date was written in some order, the user must read them in the same order

### Solutions for Reolication Lag

- More in next chapters

## Multi-Leader Replication

- Each datacenter has only one leader
- Every leader acts as a follower for the other leaders
- Some databases implements multi-leader with external tools:
  - Tungsten Replicator for MySQL
  - BDR for PostgreSQL
  - GoldenGate for Oracle
- It is effecient but there is a downside, there may be conflicts when editing the same thing
- multi-leader configuration is considered dangerous and should be avoided if possible

### Offline operations

- There are apps that needs queries to be done witout internet connections
- There will be a local database
- Each device will act as a datacenter in the Multi-Leader model
- There are tools to handle this mode like CouchDB

### Collaborative editing

- You can put a lock until each user finishes his edits
- Or allow small edits to be replicated but this needs a conflict resolution

### Handling write conflicts

- If in a single-leader mode, you can put a lock
- In a multi-leader it may be late to ask the user to solve the conflict, as the two users wrote successfully
- You can make conflict-detection syncronous but by applying this we will lose the advantage of the multi-leader setup

### Conflict avoidance

- The best way of dealing with conflicts is to avoid them

### Convergent conflict resolution

- LWW (Last write wins), add UUID to records and accept the heigher
- Give priorities to replicas
- The both implies data loss
- Merge them and let the application code resolve them later

### Custom conflict resolution logic

- Most Mutli-Leader replication tools allows the application to solve the conflicts
- the app solves the conflict auto, or prompt the user to solve it
- The conflicts are resolved on read, like CouchDB
- Automatically resolving conflicts are a research concern
  - Conflict-free replicated datatype like in Riak
  - Track history and do three-way merge
  - Operational transformation algorithm, like in Google Docs

### Multi-leader replication topologies

- Circular: like MySQL
- Star
- All-to-all

- Multi-Leader conflict detection is poorly implemented
- PostgreSQL BDR doesn't provide a consisten ordering
- Tungsten Replicator of MySQL doesn't detect conflicts at all

## Leaderless Replication

- The idea was there at first but then forgotten
- It became popular again after Amazon used it in its Dynamo-style datastores

### Writing to the database when a node is down

- The write process is considered successful if a specified number of replicas return ok

### Read repair and anti-entropy

- Two mechanisms are used in the Dynamo-style datastores
  - Read repair: determines the stale value and update it on reading it
  - Anti-entropy: a background process that fixes the stale replicas
- Not all systems applies the both, Voldmort doesn' have anti-entropy

### Quoroms for reading and writing

- generalization of the detecting-stale replicas strategy for n replicas
- W => successful writes
- R => replicas queried on read
- R + W > n
- In Dynamo-style databases, n, w, and r are configurable

### Limitation of Quorom consistency

### Monitoring staleness

### Sloppy Quoroms and Hinted Handoff

- When the `W` writes fails there are two solutions:
  - 1- Return errors to try again
  - 2- Complete the `W` from other nodes that aren't among the `n` nondes
- The second solution is called `sloppy-qourom`
- When the failed nodes are fixes, the neighbors asks the data to go home again (`Hinted Handoff`)
- Sloppy qouroms are optimal in Dynamo implementations
  - Riak: enabled by default
  - Cassandra and Voldmort: disabled by default

### Multi-datacenter operaion

### Detecting concurrent writes

- LWW (Last write wins): by timestamp (The only in Cassandra, Optional in Riak)
- LWW is a poor choice for conflict resolution
- The only safe way is to make sure every key is written once (With a UUID as the key)
- The heppened-before relationship