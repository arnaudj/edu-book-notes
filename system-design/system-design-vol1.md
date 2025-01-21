<!-- TOC -->

- System Design volume 1 - Alex Yu
  - Cheatsheet
    - Steps
    - QPS - Queries per second
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
- Other
  - LMAX Disruptor
    - Problems:
      - 2.4 Cache lines
      - 2.5. The Problems of Queues
      - Write performance tests
    - 3. Design of the LMAX Disruptor
        - 3.1. Memory Allocation
        - 3.3. Sequencing
        - 3.4. Batching Effect
  - System design - chapter 12: design a chat system
    - 1. Establish design scope
    - 2. Propose high-level design, and get buy-in
      - Topology
      - Storage
      - Data model
    - 3. Design deep dive
      - Service discovery
      - Message flow
      - Online presence
        - Online status fanout:
    - 4. Wrap up
  - System design - chapter 14: design Youtube
    - 1. Establish design scope
    - 3. Design deep dive

<!-- /TOC -->

# System Design volume 1 - Alex Yu

## Cheatsheet

### Steps
1. Establish design scope
2. Propose high-level design, and get interviewer buy-in
3. Design deep dive
4. Wrap up

### QPS - Queries per second
Let:
* DAU: daily active users
* MAU: monthly active users

Then:
* Average Concurrent Users: $Average Concurrent Users = DAU * AvgSessionDuration(minutes) / NbMinutesPerDay(1440)$
* Peak Concurrent Users:
  * estimate: $2 * Average Concurrent Users$
  * by peak activity length: $Peak Concurrent Users = DAU * AvgSessionDuration(minutes) / PeakActivityPeriod (minutes)$
* QPS:
  * $Average QPS = Average Concurrent Users * DailyQueriesPerUser$
  * $Peak QPS = Peak Concurrent Users * DailyQueriesPerUser$

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


# Other
## LMAX Disruptor

https://lmax-exchange.github.io/disruptor/disruptor.html

### Problems:

#### 2.4 Cache lines
> Caches are organised into cache-lines that are typically 32-256 bytes (commonly 64 bytes)

> “false sharing”: If two variables are in the same cache line, and they are written to by different threads, then they present the same problems of write contention as if they were a single variable.

Locks are expensive as they can lead to context switch
- starts in user mode (CAS spin): lightweight lock ([JVM Locks](https://www.alibabacloud.com/blog/lets-talk-about-several-of-the-jvm-level-locks-in-java_596090))
- as contention increases: lock get converted to native (heavyweight lock)
  - user to kernel transition is time consuming
  - thread is suspended and cache can get invalidated. 
  > To deal with the write contention a queue often uses locks. But if a lock is used, that can cause a context switch to the kernel. When this happens the processor involved is likely to lose the data in its caches. (https://martinfowler.com/articles/lmax.html)


> When accessing memory in a predictable manner CPUs are able to hide the latency cost of accessing main memory by predicting which memory is likely to be accessed next and pre-fetching it into the cache in the background. 
> Only if the processors can detect a pattern of access such as walking memory with a predictable “stride”.
> Strides typically have to be less than 2048 bytes in either direction to be noticed by the processor. 

> Data structures like linked lists and trees ... are more widely distributed in memory with no predictable stride. 
> Resulting in main memory accesses which can be more than 2 orders of magnitude less efficient.

#### 2.5. The Problems of Queues
> Queue implementations tend to have write contention on the head, tail, and size variables. When in use, queues are typically always close to full or close to empty due to the differences in pace between consumers and producers.

> This propensity to be always full or always empty results in high levels of contention and/or expensive cache coherence. 
> The problem is that even when the head and tail mechanisms are separated using different concurrent objects such as locks or CAS variables, they generally occupy the same cache-line.

> The concerns of managing producers claiming the head of a queue, consumers claiming the tail, and the storage of nodes in between make the designs of concurrent implementations very complex to manage beyond using a single large-grain lock on the queue. Large grain locks on the whole queue for put and take operations are simple to implement but represent a significant bottleneck to throughput. 

> The conclusion they came to was that to get the best caching behavior, you need a design that has only one core writing to any memory location[17]. Multiple readers are fine, processors often use special high-speed links between their caches. But queues fail the one-writer principle. (https://martinfowler.com/articles/lmax.html)

#### Write performance tests
> One particular lesson is the importance of writing tests against null components to ensure the performance test is fast enough to really measure what real components are doing. Writing fast test code is no easier than writing fast production code and it's too easy to get false results because the test isn't as fast as the component it's trying to measure. (https://martinfowler.com/articles/lmax.html)

### 3. Design of the LMAX Disruptor

> Rigorous separation of the concerns that we saw as being conflated in queues.
> Any data should be owned by only one thread for write access, therefore eliminating write contention. That design became known as the “Disruptor”. It was so named because it had elements of similarity for dealing with graphs of dependencies to the concept of “Phasers” [4] in Java 7, introduced to support Fork-Join. https://docs.oracle.com/javase%2F8%2Fdocs%2Fapi%2F%2F/java/util/concurrent/Phaser.html


##### 3.1. Memory Allocation
Ring buffer, with long lived (ie, avoid GC) entries, contiguous in memory (cache striding)

> All memory for the ring buffer is pre-allocated on start up. A ring-buffer can store either an array of pointers to entries or an array of structures representing the entries
> This pre-allocation of entries eliminates issues in languages that support garbage collection, since the entries will be re-used and live for the duration of the Disruptor instance.

> The memory for these entries is allocated at the same time and it is highly likely that it will be laid out contiguously in main memory and so support cache striding. 

> Under heavy load queue-based systems... lead to a reduction in the rate of processing: allocated objects surviving longer than they should, thus being promoted beyond the young generation with generational garbage collectors. 
> 1) the objects have to be copied between generations which cause latency jitter
> 2) these objects have to be collected from the old generation which is typically a much more expensive operation and increases the likelihood of “stop the world” 


##### 3.3. Sequencing

> Producers claim the next slot in sequence when claiming an entry in the ring (simple counter in the case of only one producer) or an atomic counter updated using CAS operations in the case of multiple producers. Once a sequence value is claimed, this entry in the ring buffer is now available to be written to by the claiming producer. When the producer has finished updating the entry it can commit the changes by updating a separate counter which represents the cursor on the ring buffer for the latest entry available to consumers. The ring buffer cursor can be read and written in a busy spin by the producers using memory barrier without requiring a CAS operation as below.

> Consumers wait for a given sequence to become available by using a memory barrier to read the cursor. 

> Consumers each contain their own sequence which they update as they process entries from the ring buffer. These consumer sequences allow the producers to track consumers to prevent the ring from wrapping. 

> Consumer sequences also allow consumers to coordinate work on the same entry in an ordered manner

##### 3.4. Batching Effect

> the consumer finds the ring buffer cursor has advanced a number of steps since it last checked it can process up to that sequence without getting involved in the concurrency mechanisms. This results in the lagging consumer quickly regaining pace with the producers when the producers burst ahead thus balancing the system.


> For most systems, as load and contention increase there is an exponential increase in latency, the characteristic “J” curve. As load increases on the Disruptor, latency remains almost flat until saturation occurs of the memory sub-system.

## System design - chapter 12: design a chat system

### 1. Establish design scope
* chat 1-1 and in group of max 100 users
* Max msg size: 100KB
* mobile and web
* 50M DAU
* Online presence
* Unlimited chat history
* Low latency
* Push notifications


Estimate concurrent users: 1M users using the app 10min during Peak Activity period of 4h: 1M * 10 / (4*60)

### 2. Propose high-level design, and get buy-in
* chat service handles msg send/recv
* relay messages between recipients
* buffers messages until recipient comes back online

How to receive messages?
* Long polling (HTTP)
  * -: inefficient (when app is not used), can't tell if user is disconnected
* selected: *WebSocket* (over HTTP)
  * +: no firewall issues (uses std ports 80/443, after HTTP request is upgraded), bidirectional, less bandwith used (persistent connection) 

Choice: chat servers: websockets for receiving and sending messages.

#### Topology

Stateless services, routed by load balancer based on request path:
- load balancer in front of services:
  - service discovery: returns list of chat servers DNS, based on load & cie
  - authentication service
  - group management
  - user profile
- DB: relational

Stateful services (long lived user connections)
- chat service instances
- Push notification instances

C10k problem: 1M concurrent users, at 10KB RAM/user -> 10GB RAM. Can fit a single node, but do not do it (single point of failure, high availability, etc)

#### Storage
Relational DB to store generic data (user profiles, settings, friends list)

Non-relational DB to store chat messages
- R/W ratio: ~= 1 for 1-1 conversations
- Key-Value store: easy horizontal scaling, read are low latency (relational DB don't handle well long tail random reads)
  - HBase (Fb messenger), Cassandra (Discord)

#### Data model
- Shard group chats based on some id (e.g, channel_id)
- Message ordering: don't rely on timestamp (`created_at` timestamp) since it can be identical, but on message_id.
- message_id requirements: message_id should be unique, and sortable. Can be done via global 64bit seq num generator like *Snowflake*, or local seq num generator (per group; easier to implement)


### 3. Design deep dive

#### Service discovery
Recommends best server based on predefined criteria (geo location, load, etc), e.g via Apache Zookeper

#### Message flow
1-1 chats: cross chat server communication:
- server 1 obtains a message id from the id generator. Pushes the msg to a sync queue, and stores it in KV store.
- server 2: reads the message. Relayed to user if it is online, else a notification is sent (push notification servers)
 
Cross device sync:
Each device of an user maintains a `cur_max_msg_id` (last msg id received) and can get new msgs from the KV store.

Small groups chat flow:
- user A sends a message via server 1.
- server 1 pushes it to each group member's sync msg queue (inbox)
- +: simple flow since each user needs to check only own inbox
- -: doesn't scale if lots of users

#### Online presence
Once client is connected via websocket, it's `status` & `last_active_at` are stored in KV store.

To smooth temp. disconnections, and hearbeat mechanism is used (e.g, if no hbeat received after 30s, mark user as disconnected)

##### Online status fanout:
Done using a publish/subscribe model. Each friends pair maintains a channel.

Given user A with friends B, C, D:
- presence servers maintain channels for friends: A-B, A-C, A-D etc where A status changes are pushed.
- friends subscribe to the right channel.

Effective for small groups. Large ones require different model (e.g, fetch on group join, or manual refreshes)

### 4. Wrap up
Extra points:
- support photo/videos (compression, cloud storage, thumbnails)
- E2E encryption
- improve load time (regions)
- error handling, e.g
  - server going down (Zookeper will provide new valid chat server address)
  - msg resend (retry / queuing)


## System design - chapter 14: design Youtube

### 1. Establish design scope
Focus on video uploading, and video streaming.

Encoding format, terminology:
- container: a bucket to store the video, audio, and meta-data. Ex: avi, mov, mp4
- codec: algorithms to compress/decompress. Ex: VP9, H.264, HEVC

### 3. Design deep dive
Encoding concepts:

- video bitrate: amount of data per unit of time (second)
  - constant: same throughout entire video
  - variable: adjusted based on the complexity of the scene
  - adaptative: dynamically adjusted based on viewer's internet speed and device capabilities (netflix, youtube)
- GOP: Group Of Pictures: used in most modern video encoding algos
  - I Frame: intra codec frame / keyframe: picture coded independently of others
  - P Frame: predictive coded frame: contains motion-compensated difference information relative to previously decoded frames
  - B Frame: bipredictive coded frame: same as P Frame but can reference previous and future frames.
  - How does GOP configuration impact video quality?
    - > The shorter the GOP length, the fewer B and P frames exist between I frames. Remember that B and P frames offer us the most efficient compression, so in a lower bitrate movie, a short GOP length will result in poorer video quality. **A longer GOP length will compress the content more efficiently**, providing higher video quality at lower bit rates, but **at the expense of trading off random access points and error resiliency**. Most encodes typically use GOP lengths in the 1-2 second range.
    (https://aws.amazon.com/blogs/media/part-1-back-to-basics-gops-explained/)

Optimization:
- clients can parallelize upload (aligned on GOP), and the rest of the pipeline can work on these chunks