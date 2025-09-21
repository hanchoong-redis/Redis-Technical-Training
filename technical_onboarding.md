# Technical Onboarding - Text-Version

## The Redis (OSS project's) Manifesto

The essential philosophy behind the development of Redis can be found in the Redis Manifesto, by Redis's creator Salvatore Sanfilippo. The latest version can be found in the source code repository.

### A DSL for Abstract Data Types

Redis is a DSL (Domain Specific Language) that manipulates abstract data types and implemented as a TCP daemon. Commands manipulate a key space where keys are binary-safe strings and values are different kinds of abstract data types. Every data type represents an abstract version of a fundamental data structure. For instance Redis Lists are an abstract representation of linked lists. In Redis, the essence of a data type isn't just the kind of operations that the data types support, but also the space and time complexity of the data type and the operations performed upon it.

### Memory storage is #1

The Redis data set, composed of defined key-value pairs, is primarily stored in the computer's memory. The amount of memory in all kinds of computers, including entry-level servers, is increasing significantly each year. Memory is fast, and allows Redis to have very predictable performance. Datasets composed of 10k or 40 millions keys will perform similarly. Complex data types like Redis Sorted Sets are easy to implement and manipulate in memory with good performance, making Redis very simple. Redis will continue to explore alternative options (where data can be optionally stored on disk, say) but the main goal of the project remains the development of an in-memory database.

### Fundamental data structures for a fundamental API

The Redis API is a direct consequence of fundamental data structures. APIs can often be arbitrary but not an API that resembles the nature of fundamental data structures. If we ever meet intelligent life forms from another part of the universe, they'll likely know, understand and recognize the same basic data structures we have in our computer science books. Redis will avoid intermediate layers in API, so that the complexity is obvious and more complex operations can be performed as the sum of the basic operations.

### We believe in code efficiency

Computers get faster and faster, yet we believe that abusing computing capabilities is not wise: the amount of operations you can do for a given amount of energy remains anyway a significant parameter: it allows to do more with less computers and, at the same time, having a smaller environmental impact. Similarly Redis is able to "scale down" to smaller devices. It is perfectly usable in a Raspberry Pi and other small ARM based computers. Faster code having just the layers of abstractions that are really needed will also result, often, in more predictable performances. We think likewise about memory usage, one of the fundamental goals of the Redis project is to incrementally build more and more memory efficient data structures, so that problems that were not approachable in RAM in the past will be perfectly fine to handle in the future.

### Code is like a poem

Code is like a poem; it's not just something we write to reach some practical result. Sometimes people that are far from the Redis philosophy suggest using other code written by other authors (frequently in other languages) in order to implement something Redis currently lacks. But to us this is like if Shakespeare decided to end Enrico IV using the Paradiso from the Divina Commedia. Is using any external code a bad idea? Not at all. Like in "One Thousand and One Nights" smaller self contained stories are embedded in a bigger story, we'll be happy to use beautiful self contained libraries when needed. At the same time, when writing the Redis story we're trying to write smaller stories that will fit in to other code.

### We're against complexity

We believe designing systems is a fight against complexity. We'll accept to fight the complexity when it's worthwhile but we'll try hard to recognize when a small feature is not worth 1000s of lines of code. Most of the time the best way to fight complexity is by not creating it at all. Complexity is also a form of lock-in: code that is very hard to understand cannot be modified by users in an independent way regardless of the license. One of the main Redis goals is to remain understandable, enough for a single programmer to have a clear idea of how it works in detail just reading the source code for a couple of weeks.

### Threading is not a silver bullet

Instead of making Redis threaded we believe on the idea of an efficient (mostly) single threaded Redis core. Multiple of such cores, that may run in the same computer or may run in multiple computers, are abstracted away as a single big system by higher order protocols and features: Redis Cluster and the upcoming Redis Proxy are our main goals. A shared nothing approach is not just much simpler (see the previous point in this document), is also optimal in NUMA systems. In the specific case of Redis it allows for each instance to have a more limited amount of data, making the Redis persist-by-fork approach more sounding. In the future we may explore parallelism only for I/O, which is the low hanging fruit: minimal complexity could provide an improved single process experience.

### Two levels of API

The Redis API has two levels:  
1. a subset of the API fits naturally into a distributed version of Redis and  
2. a more complex API that supports multi-key operations.  

Both are useful if used judiciously but there's no way to make the more complex multi-keys API distributed in an opaque way without violating our other principles. We don't want to provide the illusion of something that will work magically when actually it can't in all cases. Instead we'll provide commands to quickly migrate keys from one instance to another to perform multi-key operations and expose the trade-offs to the user.

### We optimize for joy

We believe writing code is a lot of hard work, and the only way it can be worth is by enjoying it. When there is no longer joy in writing code, the best thing to do is stop. To prevent this, we'll avoid taking paths that will make Redis less of a joy to develop.

All the above points are put together in what we call opportunistic programming: trying to get the most for the user with minimal increases in complexity (hanging fruits). Solve 95% of the problem with 5% of the code when it is acceptable. Avoid a fixed schedule but follow the flow of user requests, inspiration, Redis internal readiness for certain features (sometimes many past changes reach a critical point making a previously complex feature very easy to obtain).

---

## Introduction to Redis Open Source

### Introduction to Redis Server

Redis is an open-source data structure server written in C. It stores multiple data types, such as strings and hashes, which are accessed by key names. For example, a string value "hello world" stored under the key name greeting can be retrieved with the command:

```bash
GET greeting
```

All keys in Redis exist in a flat key space. There is no enforced schema or naming policy, so organization is left to the developer. Redis uses a client–server architecture with a request–response model. Clients connect to the Redis server over TCP using RESP (Redis Serialization Protocol). By default, it runs on port 6379, though this can be configured. RESP is simple and text-based, making it easy to read and implement. With over 200 client libraries in 50+ languages, most developers do not need to build their own client.

Redis is fast because it stores and serves data from RAM instead of disk, and because it is single-threaded, avoiding race conditions and context switching. While it cannot fully use multiple CPU cores, memory or network bottlenecks usually occur first. When needed, multiple Redis server processes can be started, especially in cluster setups.

### Replication

Redis supports master–replica replication. Clients write only to the master, and changes replicate to one or more replicas. Replication is set up by configuring a new Redis instance with `replicaof` pointing to the master. The master generates an RDB snapshot and sends it to the replica, which loads the data into memory. From then on, the replica receives a live stream of updates.

If the link between master and replica breaks, the master buffers updates. On reconnection, the replica attempts partial resynchronization. If too much time has passed, it will require a full resync.

Replication is asynchronous by default, so data loss is possible if the master fails before sending updates to replicas. To avoid this, the `WAIT` command can ensure writes are acknowledged by replicas.

### Persistence

Because Redis stores data in RAM, persistence is needed for durability. Redis supports:

- **Snapshots (RDB):** A compact `.rdb` file stores the dataset at a point in time. Snapshots can be scheduled and are useful for backups, cloning, or restarts. They are created by a forked child process to avoid blocking the main server, but this may still impact performance for large datasets.

- **Append-Only File (AOF):** Every write command is logged to disk and replayed on startup to reconstruct the dataset. AOF provides higher durability. Fsync policies can be tuned:
  - every write: safest, but highest latency.
  - every second (default): balances durability and performance.  

  AOF files are append-only but can be rewritten in the background to reduce size, preserving only the latest state.

### Clients and Networking

Redis can handle many clients at once. Each client sends requests over sockets, and the server responds. Round-trip time (RTT) can become costly when executing many commands sequentially. Redis supports pipelining, where multiple commands are sent without waiting for replies, and the replies are read in one step.

### Additional Features

- **Modules:** Extend Redis with new commands and data structures.
- **Access Control Lists (ACLs):** Control user permissions.
- **SSL/TLS:** Supported since version 6.0.
- **Lua scripting:** Run scripts directly in Redis.
- **Eviction policies:** Free up memory by expiring old or unused keys.
- **Transactions:** Using MULTI and EXEC, groups of commands execute sequentially with isolation.

Redis includes many features and will take time to fully explore, but this overview covers the key fundamentals.

---

## Redis Strings Explained

Redis strings are the most basic and common Redis data type, but they are also powerful. Strings in Redis are binary-safe sequences of bytes, similar to strings in most programming languages. You can store many types of data, including integers, binary values, comma-separated values, serialized JSON, and even larger objects like images, videos, documents, or sound files.

### Practical Uses

The most common use of Redis strings is caching, such as:

- API responses
- Session storage
- HTML pages

Strings are also useful for counters, with built-in support for incrementing and decrementing integers and floating-point values.

### Storing and Retrieving Strings

To create a string, use the `SET` command with a key and value. Example:

```bash
SET user:101:time-zone UTC+8
```

Redis replies with `OK`, confirming the value was stored. To retrieve it, use the `GET` command:

```bash
GET user:101:time-zone
```

### Caching JSON Objects

Strings can store JSON responses, which is helpful when caching results that don’t change often. For example, instead of repeatedly querying a data warehouse, cache the response under a key like `usage:63`.

When caching, you can set an expiration time with the `EX` option:

```bash
SET usage:63 "{...JSON data...}" EX 7200
```

This sets the key to expire in 7,200 seconds (two hours). You can check remaining time with the `TTL` command.

### Incrementing and Decrementing

Redis strings can also be manipulated numerically.

- `INCR key` increments the value by 1.
- `INCRBY key N` increments the value by N.
- Using a negative number decrements the value.
- If the key does not exist, Redis creates it automatically.

Example:

```bash
INCR user:23:visit-count

SET user:23:credit-balance 40
INCRBY user:23:credit-balance 30
INCRBY user:23:credit-balance -50
```

### Key Takeaways

- Redis strings are the fundamental data type.
- They can store text, numbers, JSON, or binary data.
- Strings with expirations are ideal for caching.
- They support built-in increment and decrement operations.
