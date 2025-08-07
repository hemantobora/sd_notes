1. Birthday Paradox - Collisions are far more probable than intuition suggests. Only 23 people are enough for probability of two people sharing the same birthday to exceed 50%. So, we need to increase entropy by increasing the range.
    1. For a space of size N, expect a 50% collision chance after about √N random picks.
    2. Base62 encoding
       
      | Length (power of 62) | Combinations |
      | -------------------- | ------------ |
      | 6                    | 56 billion   |
      | 7                    | 3.5 trillion |
      | 8                    | 218 trillion |

2. Caching. In-Memory vs Disk

     Speed
    | Memory | 100 nano-seconds (0.0001 ms) |
    | ------ | ---------------------------- |
    | SSD    | 0.1 ms                       |
    | HDD    | 10 ms                        |
	
    Throughput
    | Memory | Millions of reads per second |
    | ------ | ---------------------------- |
    | SDD    | 100,000 IOPS                 |
    | HDD    | 100 - 200 IOPS               |

4. CDN with Edge Computing - Point of Presence (PoPs) geographically distributed around the globe. Adding logic to Edge using platforms like Cloudflare workers or AWS Lambda@Edge. With CDN, cost factor would come into play as the traffic increases. Edge computing have limitations in execution time, memory utilization etc.

	Note: Whenever cache comes into play; we need to be careful about cache invalidation and consistency. TTL could be helpful to limit cache staleness. CDC could also help overcome the staleness.

5. For scaling support; go through each of the components in the design one by one.
	1. For Database - Introduce Replication as well as Sharding/Partitioning and periodic backups in some cases.
	2. For Services - Horizontal Scaling. Behind auto scaling group.
	3. For distributed cache - Standalone vs Sentinel vs Cluster set up.
	4. Counter setup - Reduce N/W overhead by "Counter Batching". Say first call get a counter batch of 0 - 1000. Second call, 1001 - 2000 etc.
	5. Always prefer Batch Writes to reduce Database Load on write path.
	6. CQRS - Command Query Request Segregation.
	7. Sidecar Pattern
	8. Bulkhead Pattern
	9. Circuit Breaker


6. Uploading large Files. Always use Blob/Object Storage. Get a pre signed URL and leverage the pre signed URL to upload/download file directly to/from blob/object storage. Leverage some king of event trigger to notify about upload completion to update associated file metadata.
	1. Prefer Chunking and/or Multipart upload. Etag and Part Number work closely with Multipart upload for Integrity, corruption handling and ordering of chunks. We could perform parallel upload/download for improved latency
	2. A 50 GB file over a 100 Mbps n/w takes 4000 seconds to upload/download. i.e. 1.11 hours. This would cause timeout.
	3. Large file means more possibility of network interruptions.
	4. We have to consider server memory limit as well. Amazon API Gw limit the memory to 10MB only. Certain web servers like NGINX and Apache have configured 2GB limit as default.
	5. Consider compression algorithms like GZip, Brotli, ZStandard for efficient storage and improved latency during upload and download. Compression is much more suitable for Text File.

7. System file sync agent
	1. fswatch for Linux, MacOS and BSD
	2. FsEvents on MacOS
	3. FileSystemWatcher for Windows

8. Consider File/Data Security
	1. Encryption at Rest
	2. Encryption at Transit. Leverage Secure Protocols.
	3. Access Control. RBAC - Resource Based access control.


9. When we consider consistency; It involves Write-Centric View.
	1. When two different data stores are involved; We need to leverage some kind of remote lock like Redis SETNX. This provides ordering to transactions. Better yet, if we could normalize the data or data store to use the same database instead. To include atomicity with ordered transaction we could leverage SAGA pattern using compensating transactions.
	2. In cases, where data is spanned across multiple shard or partition and ACID guarantees are needed, we could leverage 2PC as well. We could leverage distributed lock before 2PC to order the transactions to avoid any race condition. However, this step is dependent on use case.
	3. We could leverage application level lock like Mutex Lock, In Memory flags etc. Utilizing this might increase inconsistency and complexity.
 	4. If the race condition is not prevalent, we could leverage Optimistic concurrency control or multi version concurrency control.

    The Preference order should resort to:
    1. Single database (best)
    2. SAGA pattern (good for most distributed cases)
    3. Distributed locks + 2PC (when strict ACID needed)
    4. Application-level locks (last resort)


9. How to support pagination?
	1. Can we perform Lazy Loading at the client side? When the full dataset is small enough to send initially, then load details on-demand as needed. Or the response size is limited with only essential data and client could load additional data based on these prior to becoming available in view port, It should be a good solution.
	2. From server perspective; we could paginate based on offset or cursor.
		1. Offset pagination:- problematic as it looses its position when data keeps coming. So there is a possibility that with offset, client might miss or find duplicate items, though its relatively simple to implement.
		2. Cursor based:- One possibility is that there should be a timestamp element associated with the cursor to maintain ordering. Only timestamp will not help, as multiple item might correspond to the same timestamp leading to data loss. So with a timestamp based cursor, we should also include some identifier to distinguish different items. An alternative solution would be to use monotonically increasing counters. The drawback with this approach is that, we cannot jump to arbitrary page with this approach, say page 50.


10. Latency? How do we handle latency? When it comes to latency we immediately think about Caching. Caching could be a distributed caching such as Redis or Memcached. Additionally, we could also leverage CDN or both. When Caching comes into picture, we should always think about data consistency and invalidation of cache.
	1. TTL to reduce staleness.
	2. CDC to reduce staleness. We could leverage CDC to update the data in the cache. Likewise, add data to the cache.
	3. Geo-Sharding to reduce latency based on geo-proximity server access.
	4. Consider various pooling mechanism such as connection pooling, thread pooling etc.

    Caching Strategy:
    1. Cache aside. Mostly used.
    2. Cache read through. Cache automatically loads data from database on cache miss
    3. Cache write through. Update both source of truth and cache during write path.
    4. Cache write back. Just update the cache, write to data source later.
