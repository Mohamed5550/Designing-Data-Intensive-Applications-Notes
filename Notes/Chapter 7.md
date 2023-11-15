# Chapter 7: Transactions

## Introduction

    There can be many situations where the application can go wrong:

- Software or hardware fail
- Some conflict between clients
- The application may crash
- interruptions in the network
- reading partially updated data, which make no sense

You will need to deal with these issues to keep the system reliable

**Transactions** was the best solution for this issue, but we shouldn't take it for granted

- Safety guarantees is the issues that the database takes care of when we use transactions

## The slippery concept of a transaction

    Almost all relational and non-relational databases support transactions in the way that was introduced in 1975 by IBM system R (The first SQL database)

### The meaning of ACID

    The safety guarantees provided by tranactions are often described by ACID

- **Atomicy:** It means that the operations is treated like an atom, it cannot be broken down into smaller parts, it either happens completely or nothing of it will happen
- **Consitency:** It means that the application always sees the database in a consistent state, not partially updated data should be retrieved
- **Isolation:** This means that every transaction is isolated from the other transactions, even if their times conflicts
- **Durability:** Means that any data can't be lost

### Single-Object and Multi-Object Operations

    User should see the updates either before it happens, or after it finished, but not in haf the way, even if there are many things being updated together

- **Single Object:** Atomicity and isolation should apply here too, for large fields, it may take time to update it all

#### Multi object transactions

- For relational databases: you need to check the validation of foreign keys
- For document models: you need to make all the documents synced
- For secondary indexes: You need to sync all indexes together

        These errors can be handles without transactions but will be too complicated

### Handling errors and aborts

    All ACID databases can safely ignore the whole transaction when an error happens, then they retry again

- Some errors that can happen when using the retry option:
  - Transaction is finished but not signed as finished: will do it again
  - It's only worth retrying after transient errors
  - Side effects will be done twice, like sending an email

## Weak Isolation Levels

    Independent transactions can run together, but the problem arises when they depend on each other

- Isolation with serializability lets the user think as if there is no concurrency
- Serializability has it own performance cost
- There are weaker isolation levels, but they have less guarantees
- The weaker levels bugs, happens in real life, not only theoretically

### Read Committed

- It makes two guarantees:
  - When reading from database, you should only read committed data (No dirty read)
  - When you write to database, you should only overwrite committed data (No dirty writes)

- No dirty reads:
  - If the transaction hasn't yet committed or aborted and another transaction sees its data, this is called a dirty read
  - Why there shouldn't be dirty reads:
    - The user may see part of the data before the change and the other part after it, in the same time
    - The user may see changed that will be rolled back later

- No dirty writes:
  - Why there shouldn't bee dirty writes:
    - Two users may buy the same thing, but two updates are required, one may be done by the first user, and the other by the second user

- Implementing read committed:
  - Most databases apply the row-level lock to prevent dirty writes
  - One solution for preventing dirty reads, is to use the same lock for writes, but it is not efficient because many reads will be waiting for one long write
  - A better solution for preventing dirty reads is to keep both the old and new value by current transaction, the query reads the old value only, and when the transaction is finished the new value becomes the old

### Snapshot Isolation and Repeatable Read

    Read committed seems to solve everything, but it is not

- read skew: some write transaction may start and end between the start and end time of other read transaction, which means that the read transaction may read inconsistent values
- Snapshot isolation is the most common solution to this problem
- It's idea is every transaction reads from a snapshot of the database, even if another data has been written before the read is finished

#### Implementation of snapshot isolation

- readers don't block writers and writers don't block readers
- Database uses MVCC (Multi version concurrency control)
- Every write has a transaction id, and every read can read everything before some transaction happened
- The all updated objects are keeped there until the garbage collection removes the unneeded ones

#### Indexes and snapshot isolation

- The different versions of the row can be kept in an object in the index
- Some databases use the append-only variant of B-Tree

#### Repeatable reads and naming confusion

- Nobody really knows what repeatable read means

### Preventing Lost Updates

    We have discussed dirty writes, but we didn't discuss concurrent writes

- The concurrent issue can happen when the user read from database, modify the value, then writes the new value to the database (read-modify-write cycle)

#### solutions

- Atomic write operations

  - Updating the fields directly in the database

  ```SQL
  UPDATE counters SET value = value + 1 WHERE key = 'foo'
  ```

  - ORM ignores this feature many times

- Explicit locking
  - Adding a lock in the application code to some rows on some explicit application log

- Automatically detect lost updates
  - The database retries the transaction again if there is any lost updates
  - This approach is good because it doesn't require the application to do anything

- Compare-and-set
  - The database performs the update only if the value was the same as the one was selected

- Conflict resolution and replication

### Write Skew and Phantoms

    It happens when using snapshot isolation, the selected data is isolation and the updated data depends on a condition of the other selected data

## Serializability

    Testing the concurrency is difficult, because it happens only when you are unlucky with the timing, there are 3 ways to implement serializable isolation level

### Actual Serial Execution

- It seems like an ovious idea, but it came so late, why?
  - Because RAM because more cheap, so the data can be got directly from memory instead of waiting for loading it from desk

  - OLAP can use snapshot isolation because they are readonly

#### Encapsulating transactions in stored prodecures

- Making transaction wait for the user to take all the actions is not a good idea because humans are too slow in making decissions
- The stored proceedures make this request in one request to the database
- Stored procedures has gained a bad reputation because:
  - Each database used a language for stored procedures
  - Code running in a databae is diffiult
  - A database is more performance-sensitive
But modern stored procedures overcame this issues

#### Partitioning

- You can make single thread for each partition and make readonly queries use the snapshot isolation

### Two-Phase Locking (2PL)

- writers blocks both writers and readers and readers block both writers and readers

#### Implementation of two-phase locking

- 