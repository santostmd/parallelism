#Transactional Memory 

**TL;DR version**

##Intro

###What is a Transaction?

A transaction is a sequence of actions that appears indivisible and instantaneously

Has four specific attributes:

+ Failure atomicity: Executes it all, or execute nothing
+ Consistency: It goes from one consistence state to anoter consistence state (assuming a bug free program)
+ Isolation: Transactions do not interfere with each other while they are running
+ Durability: The result of a commit is permanent and available to all subsequent transactions


###So, what is exactly Transactional Memory?

- Is and abstraction for simplifying concurrent programming
- It follows and optimistic approach:
    + It relies on the conflict detection: abort and retry when necessary
- Software or Hardware runtime manages the conflicting accesses

**Support for TM runtime**

|      **HW**         |       **SW**        |
|-------------------------------|----|
|Sun Rock Processor|Intel c++ compiler/GNN GCC 4.7|
|AMD HTM specification|Intel c++ compiler/GNN GCC 4.7|
|IBM BlueGene/Q|Java (annotations)|
|Intel Haswell Processor| |


###Hm... Nice. I guess there is some advantages and some disavantages, aight?
Life is made of commitments - so yes, of course there is.

|**Advantages**|**Disavantages**|
|--------------|----------------|
|Code blocks are simply marked as atomic/isolated|Not always ideal|
|Less concerns about parallelism|Successive collisions -> Repetitive restarts -> livelocks|
|Runtime ensures the ACI properties|Memory Transactions cannot contain certain operation (e.g. I/O)|
|Hide away synchronization issues from the programmer|Incompatibility with legacy code|


While increasing the number of locks, it gets harder to understang a program.


And about the correctness issues, we can say that in TM :

- There are no dead-locks
- There are Weak-isolation Data Races problems
- There are High level Data Races Problems




###Yeah, with that we sure are getting good things... But one does not simply implement a thing like this, does it?

Calm down big boy. Let's see some approaches first.

**Pessimist Approach**

- Each item (data shared) has a read/write lock
- When an object is read, get the read lock
    + Blocks if write lock is taken
- When an object is written, get the write lock
- Uopn commit, release all locks

**Optimistic approach**

- Each item has a version number
- Read items and store read versions
- Write local copy of items
- Upon commit do atomically:
    + If all read items still have the read version
        * Then no other concurrent transaction updated the items
    + Apply all writes and increase the version number
    + Else, abort
