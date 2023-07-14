# Chapter 1: Reliable, Scalable and Maintainable applications

## Introduction

- Many applications today are data intensive
- A data intensive app is an app which needs these terms:
  - Database: saving data to use later
  - Search Indexes: searching over these data
  - Caches: remembering the result of expensive operations
  - Streams: sending messages to other systems asynchronously
  - Batches: Proccessing large amounts of data

- No one needs to build these things from scrach because databases are perfect tools for these tasks

## Thinking about data systems

- Databases, queues, etc... are getting merged together
- They are called data systems
- The developer uses the database api without knowing the implementation
- combines more than data system to achieve a certain purpose

## Reliability

- the system returns the expected output
- the system can tolerate the user mistakes
- the system has a good performance for the current use case
- the system prevents unauthorized access

- mistakes are called faults
- systems that can tolerate faults are called fault-tolerant or resilient
- not every fault can be handled: imagine the eniter planed is swallowed by a black hole

- some faults cannot be cured: like security faults
- in the book we will focus of faults that can be cured

### Hardware faults

- server parts gets courrupted all the time
- redundancy approach

### Software errors

- hardware errors happens separately
- it's unlikely that large number of hardware components will fail at the same time
- errors type:
  - when a bug occures in the core due to bad input, like linux kernel bug 2012
  - one core process because unresponsive or became invalid
  - one process takes all the shared resources
  - one errors triggers another error

- no fast solutions, here are some tips:
  - testing
  - allowing restarting
  - checking if sometings guaranteed are became invalid

### Human errors

- humans are unreliable
- one study says that most faults are caused by humans and 10-20% are hardware and software faults

- some good approaches to handle this
  - abstraction and goold interfaces
  - making sandboxes
  - testing
  - rollback
  - monitoring

## Scalability

- if it is reliable now, it doesn't mean it will be reliable in the future
- scalability is not one-dimensional

### Describing load

- it is situational, depends on many things like:
  - number of requests per second
  - number of active users in a chat room
  - number of read/write operations per second

- twitter problems
  - the fanout solution
  - the hybrid approach

### Describing performace

- two ways
  - when you increase load, with the same resources, how performance is affected
  - when you increase load, what is the cost of resources you need to increase

- mean vs percentiles

### Approaches for coping with load

- A good archeticture for one level of load, is not likely to cope with 10 times this load
- elastic systems adds more resources automatically when load increases

## Maintainability

- Most cost of the software is after deployment
- design prenciples for software engineers:
  - Operability
  - Simplicity
  - Evolvability

### Operability

- operations is important
- Monitoring
- Tracking the cause of problems
- Keeping software up to date
- Predecting future problems
..etc

### Simplicity

- complex systems is like a big ball of mud
- making the system simple doesn'y mean reducing its functionality

### Evolvability

- Making change easy
- TDD is good for this