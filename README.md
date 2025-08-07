# System Design Notes

## Table of Contents
1. [Birthday Paradox & Base62](#birthday-paradox--base62)
2. [Caching: In-Memory vs Disk](#caching-in-memory-vs-disk)
3. [CDN with Edge Computing](#cdn-with-edge-computing)
4. [Scaling Strategies](#scaling-strategies)
5. [Uploading Large Files](#uploading-large-files)
6. [File Sync Agents](#file-sync-agents)
7. [File/Data Security](#filedata-security)
8. [Consistency & Transactions](#consistency--transactions)
9. [Pagination](#pagination)
10. [Latency & Caching Strategies](#latency--caching-strategies)

---

## Birthday Paradox & Base62
- Collisions are more likely than expected. Only 23 people are needed for a >50% chance two share a birthday.
- For space size N, expect 50% collision chance after about \( \sqrt{N} \) random picks.

### Base62 Encoding
| Length (power of 62) | Combinations       |
|----------------------|--------------------|
| 6                    | 56 billion         |
| 7                    | 3.5 trillion       |
| 8                    | 218 trillion       |

---

## Caching: In-Memory vs Disk
### Speed
| Storage | Latency               |
|---------|------------------------|
| Memory  | 100 nanoseconds (0.0001 ms) |
| SSD     | 0.1 ms                |
| HDD     | 10 ms                 |

### Throughput
| Storage | IOPS (Reads/sec)      |
|---------|------------------------|
| Memory  | Millions               |
| SSD     | ~100,000              |
| HDD     | 100 - 200             |

---

## CDN with Edge Computing
- CDN = geographically distributed PoPs (Points of Presence).
- Edge Computing = logic execution closer to user via Cloudflare Workers or AWS Lambda@Edge.
- Benefits: Reduced latency, faster TTFB, global reach.
- Cautions: Higher cost with scale; limits on memory, execution time.
- Note: Be mindful of **cache invalidation** and **consistency**. TTL and CDC can help limit staleness.

---

## Scaling Strategies
1. **Database**:
   - Replication
   - Sharding/Partitioning
   - Backups
2. **Services**:
   - Horizontal scaling
   - Auto-scaling groups
3. **Distributed Cache**:
   - Standalone vs Sentinel vs Cluster
4. **Counters**:
   - Counter batching to reduce network overhead
5. **Write Optimization**:
   - Batch writes
6. **Design Patterns**:
   - CQRS (Command Query Responsibility Segregation)
   - Sidecar Pattern
   - Bulkhead Pattern
   - Circuit Breaker

---

## Uploading Large Files
- Use **Blob/Object storage** with **pre-signed URLs** for direct upload/download.
- Use **event triggers** (e.g., S3 triggers Lambda) for post-upload actions.
- Prefer **Chunking** / **Multipart upload** for:
  - Fault tolerance
  - Parallelism
  - Data integrity (via ETag + Part Number)
- Watch out for:
  - Large file timeouts (e.g., 50 GB on 100 Mbps = ~1.1 hrs)
  - Web server limits (e.g., NGINX default 2GB)
- Consider compression: **GZip, Brotli, ZStandard** — better for text files.

---

## File Sync Agents
| Platform | Utility             |
|----------|---------------------|
| Linux    | fswatch, inotify    |
| macOS    | FsEvents            |
| Windows  | FileSystemWatcher   |

---

## File/Data Security
1. **Encryption at Rest**
2. **Encryption in Transit**
3. **Access Control**:
   - RBAC (Role-Based Access Control)
   - IAM roles
   - KMS, Vault

---

## Consistency & Transactions
Write-centric considerations:

1. **Single Database** (best)
2. **SAGA pattern** — with compensating transactions
3. **Distributed Lock + 2PC**
4. **Application-level Lock** (e.g., Mutex)

Other notes:
- Use **Redis SETNX** for distributed locks
- **2PC** for cross-shard transactions (may need pre-locking)
- If race conditions rare: use **optimistic concurrency** (MVCC)
- If possible, **normalize schema to a single store** to avoid complexity

---

## Pagination
### Client-Side
- Use **lazy loading** when dataset is small or partially rendered on demand.

### Server-Side
1. **Offset Pagination**:
   - Simple but problematic under frequent inserts (can cause duplicate/missing rows).
2. **Cursor-Based Pagination**:
   - Use timestamp + unique ID to avoid duplicates.
   - Monotonic counters also work but prevent page jumps (e.g., page 50).
   - GraphQL Relay uses base64 cursors.

---

## Latency & Caching Strategies
Latency mitigation techniques:
1. **Caching**: Redis, Memcached, CDN
2. **TTL**: Reduce staleness window
3. **CDC**: Push updates to cache
4. **Geo-Sharding**: Serve from nearest region
5. **Pooling**: Connection/thread pooling
6. **Prefetching** & **lazy loading**

### Caching Strategies
| Strategy         | Description                                                             |
|------------------|-------------------------------------------------------------------------|
| Cache Aside      | App reads/writes DB, manages cache manually                             |
| Read Through     | Cache auto-loads data on miss                                           |
| Write Through    | Writes go to cache and DB simultaneously                                |
| Write Back       | Write to cache only; DB syncs later (risk of data loss on cache failure) |

Eviction Policies:
- **LRU** (Least Recently Used)
- **LFU** (Least Frequently Used)
- **FIFO** (First In First Out)

---

*More sections to be added: CAP theorem, Rate limiting, Leader election, Observability, etc.*

