# Sharding Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/sharding](../../github-repo/sharding)

## How to Study This Page

Use this page in three passes:

1. First pass: understand sharding as horizontal partitioning of data.
2. Second pass: compare hash, range, and lookup-based routing.
3. Third pass: focus on resharding, hotspots, cross-shard queries, and consistency.

By the end, you should be able to say:

> Sharding splits a large dataset across multiple storage partitions so reads and writes can scale horizontally.

## 1. Technical Definition

Sharding is a data partitioning pattern where rows or records are distributed across multiple independent shards based on a shard key and routing strategy.

Core idea:

- Data is split horizontally.
- Each shard stores a subset of records.
- A shard key decides placement.
- Reads and writes are routed to the correct shard.
- The system scales by adding shards, not only by making one database bigger.

### 30-Second Interview Answer

I would use sharding when one database cannot handle the storage size, write throughput, or read traffic. I would choose a high-cardinality shard key that spreads load evenly and matches common queries. The main trade-offs are cross-shard joins, distributed transactions, rebalancing, hotspot keys, and operational complexity.

## 2. Layman and Easy to Understand Definition

Sharding is like dividing a huge library into multiple buildings.

Books starting with A-F are in building 1, G-M in building 2, and so on. Each building is smaller and easier to manage, but finding books across all buildings is harder.

In software:

- One large table becomes many smaller partitions.
- A routing rule decides where each record lives.
- Queries must know or discover the right shard.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

A single database eventually hits limits:

- Too many rows.
- Too many writes.
- Indexes become large.
- Backups and migrations take too long.
- One machine cannot hold or serve the workload.

### 3.2 The Sharding Solution

Split data by shard key:

```text
user_id 1-1000000 -> shard-1
user_id 1000001-2000000 -> shard-2
user_id 2000001-3000000 -> shard-3
```

Each shard can run on different hardware.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Shard key | Field used to route data. |
| Shard | Storage partition containing some records. |
| Router | Component that maps key to shard. |
| Rebalancer | Moves data when shards are added or removed. |
| Directory | Lookup table for custom shard placement. |

### 3.4 Common Sharding Strategies

| Strategy | Strength | Weakness |
|---|---|---|
| Hash sharding | Good distribution. | Range queries are harder. |
| Range sharding | Easy range scans. | Can create hotspots. |
| Lookup sharding | Flexible routing. | Directory must be reliable. |
| Geo sharding | Lower regional latency. | Cross-region access is harder. |

### 3.5 Failure Path

1. One shard becomes unavailable.
2. Requests for that shard fail or degrade.
3. Other shards may continue working.
4. Replicas or failover restore shard availability.
5. Router updates if shard location changes.

## 4. Java Coding Example

This example routes users to shards by hashing `userId`.

```java
import java.util.List;

record UserRecord(long userId, String name) {
}

class ShardRouter {
    private final List<String> shardNames;

    ShardRouter(List<String> shardNames) {
        this.shardNames = shardNames;
    }

    String shardFor(long userId) {
        int index = Math.floorMod(Long.hashCode(userId), shardNames.size());
        return shardNames.get(index);
    }
}

class UserRepository {
    private final ShardRouter router;

    UserRepository(ShardRouter router) {
        this.router = router;
    }

    void save(UserRecord user) {
        String shard = router.shardFor(user.userId());
        System.out.println("Saving " + user + " into " + shard);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `userId` | Shard key. |
| `ShardRouter` | Routes key to a shard. |
| `floorMod` | Keeps hash index non-negative. |
| `UserRepository` | Application code writes through router. |
| `shardNames` | Represents available partitions. |

### Java Usage

```java
ShardRouter router = new ShardRouter(List.of("shard-a", "shard-b", "shard-c"));
UserRepository users = new UserRepository(router);

users.save(new UserRecord(42, "Asha"));
users.save(new UserRecord(99, "Ravi"));
```

## 5. Python Coding Example

```python
class ShardRouter:
    def __init__(self, shards):
        self.shards = shards

    def shard_for(self, user_id):
        return self.shards[hash(user_id) % len(self.shards)]


class UserRepository:
    def __init__(self, router):
        self.router = router

    def save(self, user_id, name):
        shard = self.router.shard_for(user_id)
        print(f"saving user={user_id} name={name} into {shard}")


router = ShardRouter(["shard-a", "shard-b", "shard-c"])
repo = UserRepository(router)
repo.save(42, "Asha")
```

### Python Usage

Use this shape when explaining:

- The shard key must be present in most queries.
- The router must be deterministic.
- Adding shards changes routing unless consistent hashing or a directory is used.

## 6. Where It Comes Handy in Real Life

- User tables at large scale.
- Chat messages by conversation id.
- Orders by customer id or merchant id.
- Time-series data by tenant and time bucket.
- Search indexes by document id.
- Multi-tenant SaaS data isolation.

## 7. Advantages Over Normal Code Without Pattern

Without sharding:

```text
one database handles all rows and all traffic
```

With sharding:

```text
many databases each handle part of the rows and traffic
```

Benefits:

- Scales write throughput.
- Reduces per-shard data size.
- Allows parallel operations.
- Limits blast radius of some failures.
- Enables geographic placement.

## 8. Where It Excels

- Data is naturally partitionable.
- Most queries include the shard key.
- Workload is too large for one database.
- Tenant or user isolation is useful.
- Cross-shard transactions are rare.

## 9. Where It Fails

- Bad shard key creates hotspots.
- Queries frequently need all shards.
- Joins span shards.
- Rebalancing is not planned.
- The router becomes inconsistent across services.

If one tenant is much larger than others, tenant-id sharding alone may be insufficient.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Databases | MongoDB sharding, Cassandra partitioning, DynamoDB partition keys |
| SQL scaling | Vitess, Citus, CockroachDB, YugabyteDB |
| Java | ShardingSphere, Hibernate multi-tenancy patterns |
| Caching | Redis Cluster hash slots |
| Search | Elasticsearch/OpenSearch shards |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Scales storage horizontally. | Cross-shard queries are expensive. |
| Improves write throughput. | Resharding is operationally hard. |
| Reduces per-node index size. | Bad keys cause hotspots. |
| Can isolate tenants or regions. | Distributed transactions become complex. |

## 12. Real-World Identification Example

Scenario: A messaging system stores billions of messages.

Sharding fit:

- Use `conversation_id` as shard key.
- All messages in one conversation route to the same shard.
- Reads for a conversation hit one shard.
- New shards can be added as traffic grows.

Without it:

- One database grows too large.
- Indexes and backups slow down.
- Write throughput hits a wall.

## 13. MAANG Interview Triggers

Use Sharding when you hear:

- "The table is too large."
- "Write throughput exceeds one database."
- "How do we scale storage horizontally?"
- "How do we partition users/orders/messages?"
- "What shard key would you choose?"
- "What happens during resharding?"

Strong answer keywords:

- shard key
- hash partitioning
- range partitioning
- consistent hashing
- hotspots
- cross-shard query
- rebalancing
- scatter-gather

## 14. Common Mistakes

### Mistake 1: Choosing low-cardinality shard keys

- Why it is wrong: few key values create uneven distribution.
- Better approach: choose high-cardinality keys with predictable access patterns.

### Mistake 2: Ignoring query patterns

- Why it is wrong: every query becomes scatter-gather.
- Better approach: choose a shard key used by common reads and writes.

### Mistake 3: No resharding plan

- Why it is wrong: growth eventually requires moving data.
- Better approach: design routing, backfill, dual writes, and cutover plans.

### Mistake 4: Global secondary indexes without cost awareness

- Why it is wrong: global lookups can require many shards.
- Better approach: maintain lookup tables or denormalized indexes deliberately.

## 15. Sharding vs Similar Patterns

| Pattern | Difference |
|---|---|
| Sharding | Splits rows across storage nodes. |
| Replication | Copies the same data to multiple nodes. |
| Partitioning | General splitting; sharding usually means distributed partitions. |
| Caching | Stores hot data temporarily for faster access. |
| Read Replica | Scales reads but not primary write capacity. |

## 16. Sharding Design Checklist

- What is the shard key?
- Does the shard key appear in common queries?
- Is key distribution even?
- What are the expected hot keys?
- How are cross-shard queries handled?
- How are new shards added?
- How is data migrated during resharding?
- How are backups and restores performed per shard?
- What happens when one shard fails?

## 17. Quick Revision Notes

- One-line summary: Sharding distributes records across multiple storage partitions.
- Three keywords: shard key, routing, rebalancing.
- Interview trap: picking a shard key without considering query patterns.
- Memory trick: split one giant table into many smaller tables by key.

## 18. Mini Exercise

Choose a shard key for a ride-sharing app.

Answer these:

1. Would you shard rides by rider id, driver id, city, or ride id?
2. Which queries become easy?
3. Which queries become hard?
4. What hotspots can happen?
5. How would you reshard after growth?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/sharding/README.md](../../github-repo/sharding/README.md)
- [github-repo/sharding/src/main/java/com/iluwatar/sharding/ShardManager.java](../../github-repo/sharding/src/main/java/com/iluwatar/sharding/ShardManager.java)
- [github-repo/sharding/src/main/java/com/iluwatar/sharding/Shard.java](../../github-repo/sharding/src/main/java/com/iluwatar/sharding/Shard.java)
- [github-repo/sharding/src/main/java/com/iluwatar/sharding/Data.java](../../github-repo/sharding/src/main/java/com/iluwatar/sharding/Data.java)
- [github-repo/sharding/src/main/java/com/iluwatar/sharding/HashShardManager.java](../../github-repo/sharding/src/main/java/com/iluwatar/sharding/HashShardManager.java)
- [github-repo/sharding/src/main/java/com/iluwatar/sharding/RangeShardManager.java](../../github-repo/sharding/src/main/java/com/iluwatar/sharding/RangeShardManager.java)
- [github-repo/sharding/src/main/java/com/iluwatar/sharding/LookupShardManager.java](../../github-repo/sharding/src/main/java/com/iluwatar/sharding/LookupShardManager.java)
- [github-repo/sharding/src/main/java/com/iluwatar/sharding/App.java](../../github-repo/sharding/src/main/java/com/iluwatar/sharding/App.java)
