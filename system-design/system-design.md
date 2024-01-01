<!-- TOC -->

- System Design - Alex Yu
  - System design - chapter 1: Scaling
    - NoSQL
    - Caching strategies
      - Read through cache
      - Cache-aside (reads)
      - Write through cache
    - Write back
    - Message queues
    - Scaling the database layer
  - Extra notes
    - Availability & Reliability
      - Availability
      - Reliability
      - Availability matrix
  - System design - chapter 5: design consistent hashing
    - Consistent hashing
      - Basic approach: hash space and hash ring
      - Extra notes: Dynamo consistent hashing strategies
        - Dynamo - Strategy 1: T random tokens per node and partition by token value
        - Dynamo - Strategy 2: T random tokens per node and equal sized partitions
        - Dynamo - Strategy 3: Q/S tokens per node, equal-sized partitions
  - System design - chapter 6: design a key value store
    - CAP Theorem
    - Data Partition
    - Data Replication
    - Consistency
    - Reconciliation
    - Handling failures
      - Failure detection
      - Handling temporary failures
      - Handling permanent failures
    - Write path
    - Read path

<!-- /TOC -->

# System Design - Alex Yu
## System design - chapter 1: Scaling

Tiers: web tier, database tier, cache tier

### NoSQL

These databases are grouped into four categories: key-value stores, graph stores, column stores, and document stores

Non-relational databases might be the right choice if:
* Your application requires super-low latency.
* Your data are unstructured, or you do not have any relational data.
* You only need to serialize and deserialize data (JSON, XML, YAML, etc.).
* You need to store a massive amount of data.

Vertical scaling vs horizontal scaling
* Vertical scaling, referred to as “scale up”
* Horizontal scaling, referred to as “scale-out”

Vertical scaling does not have failover and redundancy

Database replication:
* A failover/redundancy for DBs

Master DB generally only supports write operations
Most apps have a high R/W ratio so number of slaves > masters
If master goes offline, a slave database will be promoted to be the new master. In facts, in production systems, it's more complex: replica may not be up to date and need resynch. Other replication methods like multi-masters and circular replication could help.

### Caching strategies
From https://www.prisma.io/dataguide/managing-databases/introduction-database-caching

#### Read through cache
The cache sits between the application and the database, in the middle. 
The application will always speak with the cache for a read. If cache hit, data returned immediately
If cache miss, the cache will populate the missing data from the database and then return it to the application. Cache is populated by a lib or a cache provider.

For any data writes, the application will still go directly to the database.

Good for read-heavy workloads.

Issues: cache starts empty (can be warmed); cache can be inconsistent with database

#### Cache-aside (reads)
Same as above, but the application is responsible for fetching the data and populating the cache.

#### Write through cache
Same as above, but app writes to the cache, and cache will immediately write to DB.

### Write back
Almost exactly the same as the write-through strategy except: the cache does not immediately write to the database, and it instead writes after a delay.

By writing to the database with a delay instead of immediately, the strain on the cache is reduced in a write-heavy workload. 

This makes a write-back, read-through combination good for mixed workloads. This pairing ensures that the most recently written data and accessed data is always present and accessible via the cache.

plus: Can improve overall write performance and if batching is supported then also a reduction in overall writes. 
minus: If cache fails, possible data loss of the data not yet committed to DB.

### Message queues
A durable component, stored in memory, that supports async communication.

### Scaling the database layer
Vertical via hardware, and horizontal via more hosts (sharding)

Sharding allows to split the data in smaller and more managable parts

It is based on a sharding/partition key.

Pitfalls:
- resharding: needed when a single shard exhausts capacity (e.g, uneven distribution). Solution: upgrade the sharding function and move data.
- celebrity problem / hotspot key problem: lots of traffic overloads a specific shard with reads. Solution: allocate a shard per celebrity, and possibly further partition.
- join and de-normalization: sharding complicates join (e.g data over multiple servers). Solution: denormalize data so a given table has enough information locally

Summary of how we scale to support millions of users:
- Keep web tier stateless
- Build redundancy at every tier
- Cache data as much as you can
- Support multiple data centers
- Host static assets in CDN
- Scale your data tier by sharding
- Split tiers into individual services
- Monitor your system and use automation tools


## Extra notes
### Availability & Reliability
https://www.pagerduty.com/resources/learn/reliability/

#### Availability
Percentage of availability = (total elapsed time – sum of downtime)/total elapsed time

#### Reliability
How the service will be available given real-world scenarios — in other words, measuring the frequency and impact of failures. 
Common metrics to measure reliability are:
- Mean time between failure (MTBF) = total time in service/number of failures
- Failure rate = number of failures/total time in service

#### Availability matrix

- Five-by-five mnemonic: 5 nines allows approximately 5 minutes of downtime per year. 
Variants can be derived by multiplying or dividing by 10 (eg 5 nines -> 4 nines: x10)

- "Powers of 10" trick: per day: 4 nines allows 8.64 seconds per day: 8.64 * 10^(4-n) --- with n: number of nines

## System design - chapter 5: design consistent hashing

### Consistent hashing
Quoted from Wikipedia: "Consistent hashing is a special kind of hashing such that when a
hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on
average, where k is the number of keys, and n is the number of slots. In contrast, in most
traditional hash tables, a change in the number of array slots causes nearly all keys to be
remapped [1]”.

#### Basic approach: hash space and hash ring
- Pick a hash function (eg, SHA-1’s output range goes from 0 to 2^160 - 1).
- Collect both ends of the range to create a hash ring
- Place servers on the ring (hash their IP/hostname): s0, s1..sN
- A partition is the hash space between adjacent servers.
- Place keys on the ring (hash their value): k0, k1..kN
- Get key: to pick the server based on the key: go clockwise from the key position on the ring, until a server is reached
- Add server: place the new server on the ring, and remap only the affected keys)
ex: given s1->s2->s3->s0, adding s4 (s1->s2->s3->s4->s0): only keys between s3-s4 hashes are remapped from s0 to s4
- Remove server: same concept

Issues with basic approach:
- server ranges may not have similar size (or become unbalanced after a server removal). To get a more balanced key distribution: have nodes get assigned to multiple positions in the circle (like in Dynamo) ([Cassandra paper](https://www.cs.cornell.edu/Projects/ladis2009/papers/lakshman-ladis2009.pdf))
  - Each server ("node") maps to multiple virtual nodes on the ring ("tokens"): s0 represented by s0_0, s0_1..s0_N
  - Standard deviation by number of virtual nodes:
    - 100 vnodes: 10% std dev
    - 200 vnodes: 5% std dev
  - Trade off of more virtual nodes is memory usage.
- Oblivious to the heterogeneity in the performance of nodes. [Cassandra paper](https://www.cs.cornell.edu/Projects/ladis2009/papers/lakshman-ladis2009.pdf) recommends to analyze load information on the ring and have lightly loaded nodes move on the ring to alleviate heavily loaded nodes.

#### Extra notes: Dynamo consistent hashing strategies
This section details misc strategies from the [Dynamo: Amazon’s Highly Available Key-value Store](https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html) paper (2007).

Variables memo from https://medium.com/omarelgabrys-blog/consistent-hashing-beyond-the-basics-525304a12ba:
- N is the number of replicas for each data item, a common value is 3
- R is the number of replicas to ack / reply to a read request
- W is the number of replicas to ack / presist a write request
- S is the number of nodes in the system

##### Dynamo - Strategy 1: T random tokens per node and partition by token value
Strategy 1 is the same as above *Basic approach*: each node is assigned T random "tokens".

The fundamental issue with this strategy is that the schemes for data partitioning and data placement are intertwined: adding nodes to match trafic spike requires rebalacing:

> For instance, in some cases, it is preferred to add more nodes to the system in 
> order to handle an increase in request load. However, in this scenario, it is not 
> possible to add nodes without affecting data partitioning. ([Dynamo](https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html))


##### Dynamo - Strategy 2: T random tokens per node and equal sized partitions

- Q equally sized partitions/ranges divide the hash space
- Each node is assigned T random tokens. 
- Q is usually set such that Q >> N and Q >> S*T
  - N: number of replicas
  - S: number of nodes in the system

Quotes:
- A partition is placed on the first N unique nodes that are encountered while walking the consistent hashing ring clockwise from the end of the partition
- The tokens are only used to build the function that maps values in the hash space to the ordered lists of nodes and not to decide the partitioning

- The primary advantages of this strategy are: 
  - (i) decoupling of partitioning and partition placement
  - (ii) enabling the possibility of changing the placement scheme at runtime.

> However, the randomness of assigned T tokens, and therefore key ranges, are still a dilemma; Nodes handing data off scan their local persistence store leading to slower bootstrapping; recalculation of Merkle trees; and no easy way of taking snapshots.
(https://medium.com/omarelgabrys-blog/consistent-hashing-beyond-the-basics-525304a12ba)

##### Dynamo - Strategy 3: Q/S tokens per node, equal-sized partitions

Similar to strategy 2:
- Q equally sized partitions/ranges divide the hash space
- placement of partition is decoupled from the partitioning scheme

And specifically:
- each node is assigned a number of tokens T = Q/S
- when a server is added/removed, tokens are stolen/distributed (respect.) from/to the other nodes (to respect Q/S)

Strategy 3 (vs 1 & 2) is advantageous and simpler to deploy for the following reasons:
- (i) Faster bootstrapping/recovery: Since partition ranges are fixed, they can be stored in separate files, meaning a partition can be relocated as a unit by simply transferring the file (avoiding random accesses needed to locate specific items). 
- (ii) Ease of archival: Periodical archiving of the dataset is a mandatory requirement for most of Amazon storage services.

Disadvantage: changing the node membership requires coordination in order to preserve the properties required (Q/S).

## System design - chapter 6: design a key value store
### CAP Theorem
Can only have 2 of:
- Consistency: all clients see the same data, regardless of the node they connect to
- Availability: any client requesting data gets a response, even if some nodes are down
- Partition tolerance: system operates despite network partition (some / all packets lost)

Real distributed systems are CP/AP, because P is a must have as network can fail.

### Data Partition
Use a hash ring
- Auto scaling based on load
- Nodes heterogeneity can be reflected via number of virtual nodes for a given host.

### Data Replication
- Place replicas in different data centers connected via high speed networks 
- Replicas: partition is replicated to the N next unique servers on the hash ring.

### Consistency
- Terminology
  - N: number of replicas
  - W: write qorum size (a write is successful if ACKed by W nodes)
  - R: read qorum size (a read is successful if replied by R nodes)
- Coordinator handles read/writes
- W = 1, R = 1: low latency, but trade-off on consistency (low as well). 
  - ℹ️ It's also possible to trade off durability for latency (ex Dynamo can wait for W-1 acks, W>1, but only 1 replica is instructed to actually persists to disk before replying)
- R = 1, W = N: optimal for fast reads
- R = N, W = 1: optimal for fast writes
- W + R > N: strong consistency
  - Usually N = 3, W = R = 2
- W + R < N: weak consistency, that can be eventual consistency (Dynamo/Cassandra) via reconciliation

### Reconciliation
Via vector clocks:
  - vector of [Si, Vi] (server/process id, counter on this server)
  - Steps
    - init: all counters in the vector are set to 0
    - When a process has an internal event: it increments its own counter in the vector by one
    - When a process sends a message: it increments its own counter in the vector by one + sends a copy of the vector with the message
    - When a process receives a message: merges the receives vector with the local one: via max() element-wise. Then, it increments its own counter in the vector by one.
  - Cases:
    - Cn is ancestor of Cm iff: all counters of Cn <= counters of Cm
      - ex D([s0, 1], [s1, 1])] < D([s0, 1], [s1, 2])
    - Cn is sybling of Cm if there are counters such as: Cn < Cm and Cn > Cm
      - ex D([s0, 2], [s1, 1])] and D([s0, 1], [s1, 2])
      - This is a conflict, it requires semantic resolution (e.g client side) 
  - Issue: grows rapidly in size as number of servers/processes increases. Can be truncated like Dynamo does without issue in prod.

### Handling failures
#### Failure detection
- Detecting failures: via heartbeats, stored in a local table, and exchanged with other nodes.
  - Either multicast (costly)
  - Via Gossip protocol:
    - Each node has a node membership list (nodeId to nodeHeartBeatCounter)
    - Each node periodically increments its own counter
    - Each node periodically sends hearbeats to random nodes, that propagate in turn to other node
    - When heartbeat is received, membership list is updated to latest info
    - Heartbeat timeout? node is considered offline, and info is propagated to other nodes.

#### Handling temporary failures
Once failures are detected via above gossip protocol, deploy measures:

- Sloppy qorum:
  - Ignore qorum requirements: first W/R healthy nodes are picked for write/read (offline nodes are ignored)
- Hinted handoff:
  - While a node A is unavailable, another node B will process its requests
  - Once A is online again, B will provide it with the saved changes to achieve consistency (hinted handoff)
  - ⚠️: can degrade overall system performance if too many nodes need hinted handoff.

#### Handling permanent failures
To keep replicas in sync, use an anti-entropy protocol.

Efficient diff is made via Merkle Trees.

Hash space is divided in buckets that contains keys. 

Keys, and then buckets are hashed as part of a Merkle Tree.

Real world ex: 1M buckets, with 1K keys per bucket.


### Write path
- Write to commit log and persist to disk
- Write to in-memory cache
  - once cache is full, data is flushed to disk as an SSTable (sorted string table)

### Read path
- Check in memory cache: if cache hit, then return data to client
- Else, fetch from disk by doing a lookup in the persisted SSTables. Bloom filters are used to optimize (determines wich SSTables do **not** contain a key)