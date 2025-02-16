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
