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


###I read something about Conflict Detection... how does it work?

So, there are four implementation kinds of conflict detection:


- Eager detection of conflicts between running transactions
	- conflict is detected between two accesses to X
- Eager detection of conflicts. but only against committed transactions
	- when one transaction tries to commit, the conflict is detected
- Lazy detection of conflicts, between running transactions
	- conflict is detected, but transactions are allowed to continue
- Lazy detection of conflicts, but only against committed transactions
	- One transaction may continue running even through another has committed a conflict update. Once such commit happens, the first transaction is doomed to abort and will be wasting its work.
	
	
So, about this there is something important to retain: there are **read only** transactions and there are **update transactions**. A read only transaction is one which the write set has a size equals to zero ; otherwise is an update transaction.


###And what good is makes for me to know the difference between a read only vs update transaction?


It is usefull when you learn about MultiVersion Software TM.


###So, what is Multiversion Software Transaction Memory?

It is an implemnetation of a STM.

It allows a transaction to read old values of a recently updated object, after which the transaction may serialize **before** transactions that commited earlier in physical time.

This hability to "commit in the past" is particularly appealing for long-running read-only transactions, which may otherwise starve in many STM systems, because short-running peers modify data out from under them before they have a chance to finish.