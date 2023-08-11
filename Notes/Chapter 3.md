# Chapter 3: Storage and Retrieval

    If you keep things tidily ordered, you’re just too lazy to go searching.

- databases have to do two things:
  - Saving data
  - Getting data

- Why developers need to know how databases handle storage and retrieval internally? you won't write a new engine
  - to be able to select the appropriate one for your needs

## Data Structures that power your database

- The simplest database (log) has O(1) set and O(n) get
- It is saved in CSV files
- Indexes are data structures that are used in databases to implement fast SET/GET operations

## Hash indexes

- Presents In many programming languages
- Allows operations of SET/GET in O(1)
- Are used in Bitcask
- suitable for keys that are updated frequently
- Why don't we run out of disk space? We use segments
  - Compaction (duplicates and deletions)
  - Mergin different segments
  - every segment has its own hash map in memory
  - We find the value by checking the hash tables one by one ordered by the creation date descendingly
  - there are no many hash maps because of the merging process
- Hash indexe practical issues
  - File format: CSV is not the best, binary are faster
  - Deletion: tombstone
  - Cash recovery on restarting: keeping snapshots of the hash maps on disk
  - Partially written: Checksums
  - Concurrency control: ??

- Hash indexes limitations:
  - Hash map must fit in momory
  - It doesn't suppory range queries

## SSTables and LSM-Trees

- the sequence of key-value pairs is sorted by key

### Constructing and maintaing SSTables

- Using in-memory data structures like Red-black tree for each segment
- Appending to memdata until it is nearly full then save it on disk

### Making an LSM-Tree out of SSTable

- Used in LevelDB and RocksDB
- was described by Patrick O’Neil et al. under the name Log-Structured Merge-Tree (or LSM-Tree)

## B-Tree

- The most widely used indexing structure
- Introduced in 1970
- Most common in relational and non-relational databases too

### B-Tree optimization

- Using pointers
- Optimizing keys
- Adding additional pointers to siblings
- Other variations like fractal tree (reducing disk seeks)

## Comparing B-Tree and LSM-Trees

- B-Tree is faster on reads
- LSM-Trees is faster on writes
- There is no fast way to determine the best index, so test

## Other Indexing structures

- 