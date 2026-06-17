# Leader Election Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/leader-election](../../github-repo/leader-election)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why a distributed system sometimes needs exactly one active coordinator.
2. Second pass: trace leader failure, election start, voting/message exchange, and new leader announcement.
3. Third pass: compare leader election with consensus, distributed locks, and primary-replica failover.

By the end, you should be able to say:

> Leader Election lets a cluster safely choose one node as the coordinator while the other nodes act as followers.

## 1. Technical Definition

Leader Election is a distributed systems pattern where nodes in a cluster run a protocol to choose one node as the leader responsible for coordination tasks.

Core idea:

- Many nodes can participate.
- Only one node should coordinate a specific responsibility at a time.
- Nodes detect leader failure through heartbeats, leases, or membership changes.
- A new leader is elected when the current leader is unavailable.
- Followers must agree on the same leader to avoid split-brain behavior.

### 30-Second Interview Answer

I would use Leader Election when a replicated service needs one active coordinator, such as a scheduler, partition owner, or metadata manager. Nodes detect failure through heartbeats or leases, then run an election or rely on a system like ZooKeeper, etcd, or Kubernetes leases. The key trade-off is correctness versus complexity: we must handle network partitions, stale leaders, fencing tokens, and leader failover latency.

## 2. Layman and Easy to Understand Definition

Leader Election is like a group choosing one captain.

Everyone can work, but only one person coordinates the plan. If the captain leaves, the group chooses a new captain so work can continue.

In code:

- Nodes are candidates.
- One candidate becomes leader.
- Others become followers.
- If the leader fails, the group repeats the election.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Distributed systems often run multiple copies of a service for availability.

But some jobs should not run from every copy:

- Running a scheduled billing job.
- Assigning partitions to workers.
- Managing cluster metadata.
- Performing compaction or cleanup.
- Coordinating failover.

If every node performs the same leader-only job, the system can duplicate work or corrupt shared state.

### 3.2 The Leader Election Solution

Leader Election gives the cluster a controlled way to say:

```text
For this responsibility, node-3 is the leader right now.
```

Other nodes stay ready as followers.

If node-3 fails, the cluster chooses another node.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Candidate | Node that can become leader. |
| Leader | Node currently coordinating work. |
| Follower | Node that accepts the leader and waits. |
| Election protocol | Rules used to choose a leader. |
| Heartbeat or lease | Signal that proves the leader is still active. |
| Fencing token | Monotonic value used to reject stale leaders. |

### 3.4 Common Algorithms

| Algorithm | Core idea |
|---|---|
| Bully | Highest eligible node id wins. |
| Ring | Election message circulates through a logical ring. |
| Raft-style election | Majority vote elects a leader for a term. |
| Lease-based election | Shared store grants a time-limited leadership lease. |

### 3.5 Failure Path

1. Followers stop receiving leader heartbeats.
2. A timeout expires.
3. One or more nodes start an election.
4. The protocol chooses a winner.
5. Followers update their leader reference.
6. Old leaders are fenced if they return late.

## 4. Java Coding Example

This example uses a simple lease store. In production, the lease would live in ZooKeeper, etcd, Consul, a database row with compare-and-set, or Kubernetes `Lease`.

```java
import java.time.Instant;
import java.util.Optional;
import java.util.concurrent.atomic.AtomicReference;

record Lease(String leaderId, long token, Instant expiresAt) {
}

class LeaseStore {
    private final AtomicReference<Lease> current = new AtomicReference<>();

    Optional<Lease> get() {
        return Optional.ofNullable(current.get());
    }

    boolean tryAcquire(String candidateId, Instant now, long ttlSeconds) {
        while (true) {
            Lease existing = current.get();
            boolean expired = existing == null || existing.expiresAt().isBefore(now);

            if (!expired && !existing.leaderId().equals(candidateId)) {
                return false;
            }

            long nextToken = existing == null ? 1 : existing.token() + 1;
            Lease next = new Lease(candidateId, nextToken, now.plusSeconds(ttlSeconds));

            if (current.compareAndSet(existing, next)) {
                return true;
            }
        }
    }
}

class ClusterNode {
    private final String nodeId;
    private final LeaseStore leaseStore;

    ClusterNode(String nodeId, LeaseStore leaseStore) {
        this.nodeId = nodeId;
        this.leaseStore = leaseStore;
    }

    boolean becomeLeaderIfPossible() {
        return leaseStore.tryAcquire(nodeId, Instant.now(), 10);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `Lease` | Leadership is time-limited. |
| `token` | Each leadership term has a monotonic identity. |
| `tryAcquire` | Only one candidate can win the compare-and-set. |
| `ttlSeconds` | A dead leader eventually loses the lease. |
| `ClusterNode` | Nodes compete using the shared store. |

### Java Usage

```java
LeaseStore store = new LeaseStore();

ClusterNode nodeA = new ClusterNode("node-a", store);
ClusterNode nodeB = new ClusterNode("node-b", store);

System.out.println(nodeA.becomeLeaderIfPossible()); // true
System.out.println(nodeB.becomeLeaderIfPossible()); // false while lease is valid
```

## 5. Python Coding Example

```python
import time


class LeaseStore:
    def __init__(self):
        self.lease = None
        self.token = 0

    def try_acquire(self, candidate_id, ttl_seconds):
        now = time.time()

        if self.lease and self.lease["expires_at"] > now:
            return self.lease["leader_id"] == candidate_id

        self.token += 1
        self.lease = {
            "leader_id": candidate_id,
            "token": self.token,
            "expires_at": now + ttl_seconds,
        }
        return True


store = LeaseStore()
print(store.try_acquire("node-a", 10))
print(store.try_acquire("node-b", 10))
```

### Python Usage

Use this shape when explaining the pattern:

- `candidate_id` identifies the node.
- `ttl_seconds` makes leadership temporary.
- `token` prevents stale leaders from writing after failover.

## 6. Where It Comes Handy in Real Life

- Kubernetes controller leader election.
- Kafka partition leadership.
- Database primary selection.
- Distributed schedulers.
- Stream processing coordinators.
- Search cluster master/coordinator nodes.

## 7. Advantages Over Normal Code Without Pattern

Without Leader Election:

```text
every node runs the coordinator job
```

With Leader Election:

```text
only the elected node runs the coordinator job
```

Benefits:

- Avoids duplicate coordination work.
- Improves availability because another node can take over.
- Makes ownership explicit.
- Reduces accidental concurrent writes.
- Supports active-passive failover.

## 8. Where It Excels

- One active coordinator is required.
- Failover must happen automatically.
- The cluster can tolerate some leadership pause.
- Nodes have a reliable membership or lease mechanism.
- Work can be resumed safely by a new leader.

## 9. Where It Fails

- Network partitions are not handled.
- Old leaders can continue writing without fencing.
- Leadership state is stored in an unreliable place.
- Timeouts are too aggressive and cause election storms.
- The leader becomes a throughput bottleneck.

For high-value data mutation, prefer a tested coordination system rather than a homegrown election protocol.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Apache Curator leader election, Spring Integration leader election, Atomix |
| Kubernetes | `coordination.k8s.io` `Lease` objects |
| Infrastructure | ZooKeeper, etcd, Consul |
| Messaging/storage | Kafka controller election, database advisory locks |
| Algorithms | Raft libraries, Paxos-based systems |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Enables single active coordinator. | Adds distributed-systems complexity. |
| Supports automatic failover. | Can suffer split-brain if done poorly. |
| Works well with replicated services. | Requires careful timeout tuning. |
| Makes ownership clear. | Leader can become a bottleneck. |

## 12. Real-World Identification Example

Scenario: You run three scheduler pods, but the nightly invoice job must run once.

Leader Election fit:

- All pods are available.
- One pod holds the scheduler lease.
- Only the leader runs invoice generation.
- If the leader pod dies, another pod takes the lease.

Without it:

- All pods may generate invoices.
- Duplicate emails or charges may happen.
- Manual failover is needed when the scheduler dies.

## 13. MAANG Interview Triggers

Use Leader Election when you hear:

- "Only one node should coordinate this."
- "How do we avoid duplicate scheduled jobs?"
- "What happens if the primary dies?"
- "How do replicas choose a new owner?"
- "How do we prevent split brain?"
- "How do we assign partitions to workers?"

Strong answer keywords:

- heartbeat
- lease
- quorum
- fencing token
- split brain
- failover
- stale leader
- election timeout

## 14. Common Mistakes

### Mistake 1: Ignoring stale leaders

- Why it is wrong: old leaders may keep writing after a new leader is elected.
- Better approach: use fencing tokens or term numbers on leader-owned writes.

### Mistake 2: Depending only on local clocks

- Why it is wrong: clock skew can make leases unsafe.
- Better approach: use a coordination store or quorum protocol with monotonic terms.

### Mistake 3: Using tiny timeouts

- Why it is wrong: normal latency spikes can trigger unnecessary elections.
- Better approach: tune heartbeat and election timeouts based on real latency.

### Mistake 4: Putting all work on the leader

- Why it is wrong: the leader becomes a bottleneck.
- Better approach: let the leader coordinate and let workers execute.

## 15. Leader Election vs Similar Patterns

| Pattern | Difference |
|---|---|
| Leader Election | Chooses one coordinator from many nodes. |
| Distributed Lock | Grants exclusive access to a resource, not necessarily cluster leadership. |
| Consensus | Replicas agree on an ordered log of decisions. |
| Primary-Replica | A topology where the primary handles writes; leader election may choose that primary. |
| Singleton | In-process uniqueness only, not distributed uniqueness. |

## 16. Leader Election Design Checklist

- What responsibility needs one active owner?
- What mechanism detects leader failure?
- What prevents split brain?
- Does every leader term have a fencing token?
- What happens to in-flight work on failover?
- Can followers read safely while not leader?
- How long can the system tolerate no leader?
- How will elections be monitored?

## 17. Quick Revision Notes

- One-line summary: Leader Election chooses one active coordinator in a cluster.
- Three keywords: lease, quorum, fencing.
- Interview trap: choosing a leader without preventing stale leader writes.
- Memory trick: many candidates, one coordinator, automatic replacement.

## 18. Mini Exercise

Design a leader election flow for three scheduler pods.

Answer these:

1. Where is the lease stored?
2. How often does the leader renew it?
3. What happens when the leader stops renewing?
4. How do you prevent the old leader from writing after failover?
5. What metric proves election is healthy?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/leader-election/README.md](../../github-repo/leader-election/README.md)
- [github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/AbstractInstance.java](../../github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/AbstractInstance.java)
- [github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/AbstractMessageManager.java](../../github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/AbstractMessageManager.java)
- [github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/bully/BullyInstance.java](../../github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/bully/BullyInstance.java)
- [github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/bully/BullyMessageManager.java](../../github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/bully/BullyMessageManager.java)
- [github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/ring/RingInstance.java](../../github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/ring/RingInstance.java)
- [github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/ring/RingMessageManager.java](../../github-repo/leader-election/src/main/java/com/iluwatar/leaderelection/ring/RingMessageManager.java)
