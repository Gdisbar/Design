# CAP Theorem & Database Selection
===========================================================
          Consistency
          /        \
         /          \
        /            \
Availability -------- Partition Tolerance

# The Trade-offs: Scenario → What Happens → When It Works → When It Fails → Recovery Strategy

**Network Partition** → Nodes can't communicate → System chooses CA → Data inconsistency or downtime → "Implement circuit breakers, retry with backoff"

**Write Conflict** → Two nodes get different writes → System resolves via consensus → Data divergence → Use version vectors or CRDTs

**Node Failure** → One node goes down → System continues→ Loss of availability → Auto-failover

======================================================================
# SQL vs NoSQL Decision Matrix
=====================================
5 Questions to Choose Your Database:

1. **Access Pattern**: Read-heavy → Cache/Replica | Write-heavy → Sharding
2. **Consistency Need**: Payment/Traffic → ACID SQL | Analytics → NoSQL
3. **Failure Mode**: Can you handle stale reads? → NoSQL | Need exact? → SQL
4. **Operational Cost**: Complex vs Simple solutions
5. **10x Traffic**: Will it scale? What breaks first?

======================================================================
# Database Types & Use Cases
===================================================================
┌─────────────────────────────────────────────────────────────────┐
│                        DATABASE TYPES                          │
├───────────────┬──────────────────┬─────────────────────────────┤
│ Type          │ Examples         │ Best For                    │
├───────────────┼──────────────────┼─────────────────────────────┤
│ Document      │ MongoDB          │ JSON data, flexible schema  │
│ Wide-Column   │ CosmosDB         │ Time-series, IoT data       │
│ Key-Value     │ Redis            │ Caching, Sessions           │
│ Graph         │ Neo4j            │ Relationships, Social graphs│
│ Search Engine │ Elasticsearch    │ Full-text search, logs      │
│ OLAP          │ BigQuery         │ Analytics, BI               │
│ Row Store     │ PostgreSQL       │ Transactional workloads     │
└───────────────┴──────────────────┴─────────────────────────────┘

# Elastic Search: When & Why

┌─────────────────────────────────────────────────────────────────┐
│                    SEARCH ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User Query: "LIKE '%middle_trailing_wilcard%'"               │
│                    │                                            │
│                    ▼                                            │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Problem: RDBMS B-Tree CAN'T optimize wildcard       │      │
│  │  Solution: Fallback to full table scan              │      │
│  │  Result: SLOW on large datasets                     │      │
│  └──────────────────────────────────────────────────────┘      │
│                    │                                            │
│                    ▼                                            │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Move to Elastic Search                             │      │
│  │  - Inverted index for fast search                   │      │
│  │  - Handles partial matches efficiently              │      │
│  └──────────────────────────────────────────────────────┘      │
│                    │                                            │
│  If Elastic Search Fails:                                      │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  1. PG scoped search (fallback)                    │      │
│  │  2. Redis trending items (cached results)          │      │
│  │  3. Fallback to basic exact match search           │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  Advanced Search Types:                                       │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Intent-based: Add Vector DB + fallback PG         │      │
│  │  Relation-based: Add Neo4j entity-relation DB      │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# Database Architectures : Scaling

┌─────────────────────────────────────────────────────────────────┐
│                      SCALING STRATEGIES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VERTICAL SCALING (Scale Up)                                   │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  One DB Server                                      │      │
│  │  Increase: CPU, RAM, Disk, Network                  │      │
│  │  Limit: Hardware capacity, cost                     │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  HORIZONTAL SCALING (Scale Out)                                │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Sharding (Write-heavy)                             │      │
│  │  Replication (Read-heavy)                           │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
# Database Architectures : Sharding Types & Failure Scenarios

┌─────────────────────────────────────────────────────────────────┐
│                      SHARDING TYPES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. RANGE-BASED SHARDING                                       │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Shard 1: user_id 1-1000                            │      │
│  │  Shard 2: user_id 1001-2000                         │      │
│  │  Shard 3: user_id 2001-3000                         │      │
│  │                                                     │      │
│  │  ✅ Good: Simple, range queries easy               │      │
│  │  ❌ Bad: Hotspots (new users go to last shard)    │      │
│  │  🔧 Recovery: Rebalance, add new shard             │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  2. DIRECTORY-BASED SHARDING                                   │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Lookup Service → Which DB?                         │      │
│  │                                                     │      │
│  │  ✅ Good: Flexible, can remap                       │      │
│  │  ❌ Bad: Lookup is single point of failure         │      │
│  │  🔧 Recovery: Cache lookup, replicate lookup       │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  3. GEOGRAPHY-BASED SHARDING                                   │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  US users → US DB                                   │      │
│  │  EU users → EU DB                                   │      │
│  │  APAC users → APAC DB                               │      │
│  │                                                     │      │
│  │  ✅ Good: Low latency, compliance                   │      │
│  │  ❌ Bad: Cross-region queries complex              │      │
│  │  🔧 Recovery: Cross-region replication             │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘


# Database Architectures : Replication Types

┌─────────────────────────────────────────────────────────────────┐
│                    REPLICATION TYPES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MASTER-SLAVE REPLICATION                                      │
│  ┌──────────────────────────────────────────────────────┐      │
│  │                    Master                           │      │
│  │                   /      \                          │      │
│  │              Read/Write   \                         │      │
│  │                         Replication                 │      │
│  │                    /               \                │      │
│  │              Slave 1            Slave 2             │      │
│  │           (Read Only)        (Read Only)            │      │
│  │                                                     │      │
│  │  ✅ Good: Read scaling, backup                     │      │
│  │  ❌ Bad: Write bottleneck, replication lag         │      │
│  │  🔧 Recovery: Promote slave to master              │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  MASTER-MASTER REPLICATION                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │         Master 1 ←→ Master 2                        │      │
│  │         (Read/Write)  (Read/Write)                  │      │
│  │                                                     │      │
│  │  ✅ Good: High availability, write scaling         │      │
│  │  ❌ Bad: Conflict resolution, complexity           │      │
│  │  🔧 Recovery: Conflict resolution strategies       │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# Database Architectures : Database Performance Optimization

┌─────────────────────────────────────────────────────────────────┐
│                 PERFORMANCE OPTIMIZATION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CACHING                                                       │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Redis/Memcached for frequent queries             │      │
│  │  - Cache invalidation strategies                    │      │
│  │  - When cache fails: DB fallback + cache warming    │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  INDEXING                                                      │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - B-Tree for equality/range                         │      │
│  │  - Hash for exact lookups                           │      │
│  │  - Covering indexes for faster queries              │      │
│  │  - Watch for: Too many indexes (write overhead)     │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  QUERY OPTIMIZATION                                            │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - EXPLAIN ANALYZE to find bottlenecks              │      │
│  │  - Avoid SELECT * (fetch only needed columns)       │      │
│  │  - Use connection pooling (PG Bouncer)              │      │
│  │  - Keep transaction data and OLAP separate          │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  PG BOUNCER EXAMPLE:                                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  2,500 App Connections                              │      │
│  │         ↓                                           │      │
│  │  PG Bouncer (Connection Pooling)                   │      │
│  │         ↓                                           │      │
│  │  500 Active DB Connections                          │      │
│  │                                                     │      │
│  │  ✅ Saves DB resources, reduces overhead            │      │
│  │  ❌ Connections timeout if pool exhausted           │      │
│  │  🔧 Recovery: Increase pool, add timeout retry     │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

======================================================================
# Caching Strategies & Invalidation
======================================================================

# Cache Types & Use Cases

┌─────────────────────────────────────────────────────────────────┐
│                      CACHE TYPES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT-SIDE CACHING                                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Browser cache (Cache-Control headers)            │      │
│  │  - Mobile app local storage                         │      │
│  │  ✅ Reduces bandwidth, faster load                  │      │
│  │  ❌ Stale data, cache invalidation issues           │      │
│  │  🔧 Recovery: Use ETag, versioned resources        │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  SERVER-SIDE CACHING                                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Redis, Memcached, NGINX                          │      │
│  │  - In-memory caching                                │      │
│  │  ✅ Fast, reduces DB load                           │      │
│  │  ❌ Memory limits, cache churn                     │      │
│  │  🔧 Recovery: TTL, LRU eviction                    │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  CDN (Content Delivery Network)                                │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  1. PULL-BASED: Origin → CDN on first request       │      │
│  │  2. PUSH-BASED: Upload → CDN distribution           │      │
│  │                                                     │      │
│  │  ✅ Global low latency, DDoS protection             │      │
│  │  ❌ Stale content, cost, cache purge delay          │      │
│  │  🔧 Recovery: Purge cache, TTL short for dynamic    │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# Cache Invalidation Strategies

┌─────────────────────────────────────────────────────────────────┐
│                CACHE INVALIDATION                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WRITE-THROUGH CACHE                                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  1. Client writes to cache                          │      │
│  │  2. Cache updates DB                                │      │
│  │  3. Success response                                │      │
│  │                                                     │      │
│  │  ✅ Strong consistency                              │      │
│  │  ❌ Slower writes, if DB fails → write fails       │      │
│  │  🔧 Recovery: Queue writes, retry                   │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  WRITE-BACK CACHE                                              │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  1. Client writes to cache                          │      │
│  │  2. Success response (fast)                         │      │
│  │  3. Cron/async syncs cache to DB                    │      │
│  │                                                     │      │
│  │  ✅ Fast writes, high throughput                    │      │
│  │  ❌ Data loss if cache fails before sync            │      │
│  │  🔧 Recovery: Write-ahead logs, checkpoint          │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  BUFFER CACHE EVICTION:                                        │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Problem: Heavy analytics query evicts all cached   │      │
│  │           transaction data                          │      │
│  │                                                     │      │
│  │  Solution: Separate OLAP and transaction DB         │      │
│  │  Rule of thumb: ALWAYS keep them separate           │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# Cache Failure Recovery

┌─────────────────────────────────────────────────────────────────┐
│              CACHE FAILURE RECOVERY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  When Cache Fails:                                             │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  1. Circuit breaker opens                           │      │
│  │  2. All requests go to DB                           │      │
│  │  3. Monitor DB load                                 │      │
│  │  4. Gradually bring cache back                      │      │
│  │  5. Warm cache with frequent data                   │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  ⚠️  Cache Stampede Protection:                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Multiple clients rebuild same key                │      │
│  │  - Solution: Use mutex/Redis lock                   │      │
│  │  - Only one client recomputes, others wait          │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

======================================================================
# Load Balancing & High Availability
======================================================================

# Load Balancing Algorithms

┌─────────────────────────────────────────────────────────────────┐
│                LOAD BALANCING ALGORITHMS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. ROUND ROBIN                                                │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Requests: 1→Server1, 2→Server2, 3→Server3, 4→Server1│      │
│  │  ✅ Simple, works for identical servers              │      │
│  │  ❌ Uneven load if servers differ                    │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  2. LEAST CONNECTION                                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Sends to server with fewest active connections      │      │
│  │  ✅ Handles long-running requests well               │      │
│  │  ❌ Overhead tracking connections                    │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  3. LEAST RESPONSE TIME                                        │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Measures response time, sends to fastest           │      │
│  │  ✅ Self-tuning                                      │      │
│  │  ❌ Variance in response times                       │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  4. IP HASH                                                    │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  hash(client_ip) % server_count → same server always │      │
│  │  ✅ Session persistence                              │      │
│  │  ❌ Uneven distribution, server changes break       │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  5. WEIGHTED ALGOS                                             │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Server1(weight 5) → 5x more traffic than Server2    │      │
│  │  ✅ Handles different server capacities             │      │
│  │  ❌ Manual configuration overhead                    │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  6. CONSISTENT HASHING                                         │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  virtual nodes distributed on hash ring             │      │
│  │  ✅ Minimal rehashing on add/remove                 │      │
│  │  ❌ Complex to implement                            │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# Load Balancer Types

┌─────────────────────────────────────────────────────────────────┐
│                  LOAD BALANCER TYPES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SOFTWARE:                                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - NGINX: Reverse proxy, caching, SSL termination    │      │
│  │  - HAproxy: High-performance, L4/L7 balancing        │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  HARDWARE:                                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - F5 BIG-IP: Enterprise, WAF, SSL acceleration      │      │
│  │  - Citrix ADC: Load balancing, compression           │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  CLOUD:                                                        │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Azure Load Balancer: L4 balancing                 │      │
│  │  - AWS ELB: Managed LB with scaling                  │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# Single Point of Failure & Recovery

┌─────────────────────────────────────────────────────────────────┐
│              SINGLE POINT OF FAILURE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Architecture:                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │                                                      │      │
│  │    Users → LB → Cache → DB                          │      │
│  │             ↑                                        │      │
│  │         SPOF                                         │      │
│  │                                                      │      │
│  │  If LB fails → Entire system goes down              │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  Recovery Strategies:                                          │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  1. Redundancy                                      │      │
│  │     → Active-Passive: One LB active, one standby    │      │
│  │     → Active-Active: Both LB balance traffic        │      │
│  │                                                     │      │
│  │  2. Health Check & Self-Healing                     │      │
│  │     → LB pings servers, removes unhealthy           │      │
│  │     → Auto-restart failed services                  │      │
│  │     → Kubernetes handles pod restarts               │      │
│  │                                                     │      │
│  │  3. DNS Failover                                    │      │
│  │     → Multiple IPs in DNS record                    │      │
│  │     → Clients retry next IP on failure              │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

