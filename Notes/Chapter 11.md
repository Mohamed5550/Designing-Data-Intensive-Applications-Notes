# Chapter 11: Streaming

    A complex system that works is invariably found to have evolved from a simple system that works. The inverse proposition also appears to be true: A complex system designed from scratch never works and cannot be made to work.

- We saw in chapter 10 how to create batch processes that is helpful in search indexex, analytics and more
- Batch processing assumes that the data is finite, so we can run it after every hour or every day, like sorting, we must know all of the data to puth the smallest item in front of it, but most data is unbounded
- keeping date being processed and returning output continously is stream processing
- in general `stream` refers to data that is incremently made available over time

## Transmitting Event Streams

- The input and output in batch processing world are files
- In streams the input and output are events, containing something that happended at some point of time
- The events might be a user event or a machine event
- The event is stored in JSON or text format or any binary format like discussed in `chapter 4`
- Batch vs Stream
  - in batch processing Files are written and then read by many processes 
  - in stream processing events are produces once by a (sender or publisher) and then processed by multiple consumers (subscribers or recipients)
  - In batch processing files are group of related data
  - In stream processing related data are grouped in a stream or a topic
- In streams there must be notifications to send after the event is received. Databases have traditionaly not supported this king of processing

### Messaging systems

- Common approach for notifying users is to use a messaging system which can be implemented using a direct communication channel like TCP or Unix pipeline
- unix work only when there is one sender and one reciever
- A messaging system allows multiple senders and multiple recievers
- two questions when implementing a messaging system
  - What happens when the producer send more messages than the reciever can recieve?
    - Drop messages
    - Buffer queue
    - backpressure
  - What happens when the node fails

- Direct messaging tools
  - UDP multicast, used in market stocks. But it is unreliable
  - Brokerless messaging libraries suchas ZeroMQ uses TCP or IP multicast
  - StatsD and Brubeck uses UDP
  - Webhooks
- These systems requires the application to be aware of its unreliability

- Message brokers
  - Durability here is the responsibility of brokers
  - They wait for message to be buffered but it doesn't wait for it to be received by consumer

- Message brokers vs Databases
  - Databases keeps data after quering, but brokers delete them after the consumer receives it
  - Message brokers keep small data, but in case of many messages, it splits them into disk
  - databases have secondary indexes, but brokers have channels subscription with other structures
  - brokers notify the consumers when data change, but databases don't

- message brokers standards: `JMS`, `AMQP`
- message brokers softwares: `RabbitMQ`, `ActiveMQ`, `HornetMQ`, `Qpid`, `TIBCO`, `IBM MQ`, `Azure Bus`, `Google Could Pub\Sub`

- Multiple consumers
  - Load Balancing: each message is sent to one consumer
  - Fan-out: each message is sent to all consumers
  - they can be combined
- Acknowledgements
  - each consumer must send back acknowledgement to tell the broker it has received the message, otherwise, the broker will send the message again

### Partitioned Logs

- Messages in brokers are not durable like databases
- We can have a hybrid, combining the two approaches

- Using logs for message storage
  - We can make a log file and check if the end of the file has new record
  - This is the idea begind Unix tool `tail -f`
  - The log can be partitioned across different machines
  - Log-based message brokers: `Apache Kafka`, `Amazon Kinesis Streams`, `Twiter DistributedLog`
  `Google Cloud Sub\Pub`: is the same but it exposes a JMS-style API
  - although desk access, they can achieve a throughout of millions of messages per second
  - this approach doesn't need acknowledgements

- Diska space usage
  - The processed messages are divided into segments and the old segments are deleted periodically
  - If the consumers are too slow and the disk of the producer is full, then the new messages override the old messages

- Log structure is a form of buffering that is limited by disk space

## Database and Streams

    Log-based streams tok some advantages from database and database replication log is a kind of streaming

- We want to sync data between databases and streams

## Keeping Systems in Sync

- Non-trivial data systems use many kinds of databases to satisfy its needs, example:
  - OLTP database for queries
  - Warehouse for analytics
  - Fulltext search index for searching

- To keep data in sync between these databases we may use `ETL` like in warehouses, which is a batch process
- A faster approach is to use `dual writing`, which is to change the data in the second database after you send it to the first one, but it have some problems:
  - Concurrency: concurrency detection
  - Fault tolerance: 2 phace commit
- but it is better if there is only one leader

### Change Data Capture

- Database logs was external implementation of databases not an API
- Recently there was an interest about `CDC`, so they decided to take data from database log and replicate it in the same order to other databases
- Implemenging change data capture
  - Database triggers are used to link table records changes to the other database
  - Examples: `LinkedIn's Databus`, `Facebook Wormhole`, `Yahoo!'s Sherpa` use these idea at large scales
  - WAL API examples: `Bottled Water for PostgreSQL`, `Maxwell and Debezium for MySQL`, `Mongoriver for MongoDB`, `GoldenGate for Oracle`

### Event Sroucing

- It is an approach in the DDD community like the ideas we've discussed
- The only difference it this approach is more abstracted
- Event sourcing works with snaphots so it is faster for restoring data

### States, Streams, and Immutability

- There is a command and an event
- Command is before the transaction is done
- The command can be sent then the transaction fails
- Events are sent after command succeeds
- Events cannot be reverted

## Processing Streams

- There are 3 ways to process data after receiving it into consumer
  - You can put it in a cache or a search index...etc
  - You can push events to users like notifications or emails
  - You can product ouput from each input like MapReduce
- Rest of the chapter is discussion of option 3
- Sorting doesn't make sense here, because the dataset in unbounded

### Uses of Stream Processing

- Examples of using stream processing
  - Fraud detection to tell if the card might be stolen and block it
  - Examining price changes
  - Monitoring machines states in manufacturing
  - Military detecting attacks

- Complex event processing
  - An approach developed in 1990 for analyzing event streams
  - It searches like reqgular expressions
  - In CEP systems the relation between data and queries is reversed
    - in normal databases data is long term and queries are transient
    - in CEP systems queries are long term and data is transient
  - implementations for CEP: `Esper`, `IBM InfoSphere Streams`, `Apama`, `TIBCO StreamBase`, `SQLStream`

- Stream analytics
  - Analysis are like CEP but it is interested in aggregates
    - measuring the rate of an event
    - average of some time period
    - comparing current statistics to previous ones
  - Stream processes are not approximate
  - Open source distributed streams for analytics: `Apache Storm`, `Spark Streaming`, `Flink`, `Concord`, `Samza`, `Kafka Streams`
  - hosted services: `Google Cloud Dataflow`, `Azure Stream analytics`

- Maintaining materialized views
- Search on streams
  - Searching for an event with a complex query
  - Like informing a client when there is a property matching its criteria
  - procolator feature of `Elasticsearch` does this

- Message passing and RPC

### Reasoning About Time

- We need to know when the even happened not when it is processed