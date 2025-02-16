<!-- TOC -->

- System Design volume 2 - Alex Yu
  - System design - chapter 17: Proximity service
    - Establish design scope
    - Propose high-level design, and get interviewer buy-in
      - Algorithms
      - DB
    - Design deep dive
      - Scaling
      - Caching
  - System design - chapter 18: Nearby friends
  - System design - chapter 19: Google Maps
  - System design - chapter 20: Distributed Message Queue
    - Understand problem and Design scope
    - Propose high-level design, and get interviewer buy-in
    - Design deep dive
      - Data storage:
      - Message structure:
      - Batching:
      - Producer flow:

<!-- /TOC -->
# System Design volume 2 - Alex Yu

## System design - chapter 17: Proximity service
https://github.com/preslavmihaylov/booknotes/tree/master/system-design/system-design-interview/chapter17

### Establish design scope
### Propose high-level design, and get interviewer buy-in
#### Algorithms
* Geohash: evenly divided grid
* Quadtree: same principle, but quadrants are limited to X entries (e.g 100) before sub-dividing.
* S2 (more complex, out of scope): based on Hilbert curves

#### DB
The business DB has 200M entries, has low write volume. Can be a relational DB like MySQL.

Location DB: needs custom data structure (cf algorithms) and caching (cf below).

### Design deep dive
#### Scaling
The business DB (relational, 200M entries) can be sharded by businessID.

The location DB: read-heavy. Does not needed sharding (only 1.7 GB in quadtree in memory format). Should have read replicas to share the read load.

#### Caching
The local DB can benefit from a cache, like Redis, with key geohash to list of businessID.
(geohash smoothes location changes on mobile, and clusters results by quadrant)

Replicate caches should be deployed in different regions/AZs, to spread load, reduce latency, and comply with data privacy laws.


## System design - chapter 18: Nearby friends
https://github.com/preslavmihaylov/booknotes/tree/master/system-design/system-design-interview/chapter18

## System design - chapter 19: Google Maps
https://github.com/preslavmihaylov/booknotes/tree/master/system-design/system-design-interview/chapter19


## System design - chapter 20: Distributed Message Queue
https://github.com/preslavmihaylov/booknotes/blob/master/system-design/system-design-interview/chapter20/README.md

### Understand problem and Design scope

Functional requirements:
* 100 Kb/msg
* Messages can be consumed multiple times
* 2 weeks persistence
* Msgs consumed in same order as produced
* As many publisher/subscribers as possible
* Data delivery semantics: at least once (nice to have: at most once, exactly once)

Non functional requirements:
* High througput, low latency
* Scalability
* Persistent & durable (i.e, with replication)

### Propose high-level design, and get interviewer buy-in
Producer --> Message Queue --> Consumer

Message queue:
- decouples producer and consumers so they scale independently. 
- when consumer ACKs, the message is removed from the queue (nb: in Kafka, consumers track their read offset)

Messaging models:
* point to point: a message M is consumed by exactly one consumer. (Kafka & other message queues)
* pub sub: a message M on topic 1 is consumed by all subscribers of topic 1. (Kafka)
```
                                                          ┌────────────────┐
                                           Message A      │                │
                                                 ┌───────►│   Consumer 1   │
                                                 │        │                │
                                 Message Queue   │        └────────────────┘
┌──────────────┐ Message A   ┌───────────────────┴─┐                        
│              │             │     Topic 1         │                        
│  Producer    ┼────────────►│                     │                        
│              │             │                     │                        
└──────────────┘             └───────────────────┬─┘                        
                                                 │        ┌────────────────┐
                                                 │        │                │
                                                 └───────►│   Consumer 2   │
                                           Message A      │                │
                                                          └────────────────┘
```
* event streamking (Kafka)


Kafka terminology:
* Messages are sent to a topic
* A topic is evenly distributed across partitions
* Each partition is total ordered, FIFO, with offset
* Each msg is dispatched to a given partition, based on the msg's partition key
* A consumer group has N consumers consuming a topic in parallel. Each consumer of the CG reads from a subset of partitions.
* To enforce ordering, in a CG, only 1 consumer can subscribe to a given partition => $Max Consumers Per CG <=> Nb Of Partitions$

### Design deep dive
Design choices to achieve high throughput and retention:
* On disk data structure, leveraging sequential queries and caching strategies of HDD and OS
* Zero copy data structure:
  * the message structure is designed to fit requirements of all layers.
  * usage of zero copy primitives to transfer from disk to network (java `transferTo` method)
* Batching, to avoid small IOs

#### Data storage:
Read and write heavy. Immutable. Prune after expiration delay.

* Database: not an option, since it is hard to scale for both heavy read & heavy write workloads
* Disk: Write ahead log (WAL): on disk files
  * 1 partition is split in N segments (as file grows, older segments become immutable)

#### Message structure:
* key: byte[]: used to pick the partition (e.g, user id)
* value: byte[]: payload
* topic: string
* partition: integer
* offset: long
* timestamp: long
* size: int
* crc: int[5]

#### Batching:

* Applies at producer, queue, and consumer levels
* Batching improves performance by amortizing cost of IOs:
  * Network level: less roundtrips
  * Sequential read/write to WALs, leveraging disk caching and contiguous writes
* Is a tradeoff between throughput and latency:
  * High batch size: high throughput + high latency
  * Lower batch size: lower throughput + lower latency
  * Settings based on intended usage:
    * tuned for low latency: the case of traditional message queues: smaller batch size
    * tuned for high throughput: higher batch size. Broker can scale better by adding more partitions (for higher parallelism), spreading partitions over multiple disks, and more (cf Kafka options)

#### Producer flow:
How to resolve the Broker in charge of a partition? 
* Adding a routing layer, that reads the *replication plan* (from metadata store) and caches locally
* Routing layer is embedded in client to avoid roundtrips and enable efficient batching
* Routing layer sends messages to partition leader
  * leader forwards to replica(s) and waits enough ACKs from replicas (based on config), before commiting data & replying to producer.
* Replicas improve fault tolerance

