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
