# Throttling Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/throttling](../../github-repo/throttling)

## How to Study This Page

Use this page in three passes:

1. First pass: understand throttling as slowing or rejecting traffic to protect resources.
2. Second pass: trace tenant counters, time windows, allowed calls, and rejected calls.
3. Third pass: compare Throttling with Rate Limiting, Backpressure, and Queue-Based Load Leveling.

By the end, you should be able to say:

> Throttling controls resource consumption so a service stays stable under high or unfair demand.

## 1. Technical Definition

Throttling is a resilience and resource-management pattern that limits the rate or amount of work a caller, tenant, or service can consume.

Core idea:

- Identify the consumer.
- Track usage.
- Compare usage with allowed quota.
- Allow, delay, degrade, or reject.
- Reset or refill usage over time.

### 30-Second Interview Answer

I would use throttling when a service must protect shared resources from overuse by tenants, users, or operations. It is similar to rate limiting, but I frame throttling more broadly around resource consumption and operational stability. The trade-off is that some requests may be slowed or rejected, so responses should include clear retry guidance and metrics.

## 2. Layman and Easy to Understand Definition

Throttling is like controlling a faucet.

When too much water flows, you narrow the opening so the system behind it is not overwhelmed.

In code:

- Count requests per consumer.
- Compare with allowed quota.
- Serve if under quota.
- Reject or delay if over quota.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without throttling:

- One tenant can consume all capacity.
- Expensive endpoints can starve cheap endpoints.
- Traffic bursts overload dependencies.
- The service misses availability targets.

### 3.2 The Throttling Solution

Add a controlled usage gate:

```text
request -> usage counter -> under quota -> process
                         -> over quota -> throttle response
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Consumer | User, tenant, client, API key, or operation. |
| Quota | Allowed usage level. |
| Counter | Tracks usage in a period. |
| Window | Time period for quota calculation. |
| Throttle response | Delay, reject, or degrade. |
| Reset/refill | Restores available quota. |

### 3.4 Throttling Actions

| Action | Meaning |
|---|---|
| Reject | Return `429` or similar. |
| Delay | Queue or wait before processing. |
| Degrade | Return cheaper partial response. |
| Shed | Drop low-priority work. |

## 4. Java Coding Example

This example throttles per tenant in a fixed window.

```java
import java.time.Instant;
import java.util.HashMap;
import java.util.Map;

class TenantThrottler {
    private final int maxCalls;
    private final Map<String, Integer> calls = new HashMap<>();
    private Instant windowStart = Instant.now();

    TenantThrottler(int maxCalls) {
        this.maxCalls = maxCalls;
    }

    synchronized boolean allow(String tenantId) {
        if (windowStart.plusSeconds(1).isBefore(Instant.now())) {
            calls.clear();
            windowStart = Instant.now();
        }

        int used = calls.getOrDefault(tenantId, 0);
        if (used >= maxCalls) {
            return false;
        }
        calls.put(tenantId, used + 1);
        return true;
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `tenantId` | Throttle key. |
| `maxCalls` | Quota per window. |
| `windowStart` | Window reset time. |
| `calls.clear` | Resets usage. |
| `allow` | Central decision point. |

### Java Usage

```java
TenantThrottler throttler = new TenantThrottler(2);

System.out.println(throttler.allow("tenant-a"));
System.out.println(throttler.allow("tenant-a"));
System.out.println(throttler.allow("tenant-a"));
```

## 5. Python Coding Example

```python
import time


class TenantThrottler:
    def __init__(self, max_calls):
        self.max_calls = max_calls
        self.window_start = time.time()
        self.calls = {}

    def allow(self, tenant_id):
        if time.time() - self.window_start >= 1:
            self.calls.clear()
            self.window_start = time.time()

        used = self.calls.get(tenant_id, 0)
        if used >= self.max_calls:
            return False

        self.calls[tenant_id] = used + 1
        return True


throttler = TenantThrottler(2)
print(throttler.allow("tenant-a"))
```

### Python Usage

Use this shape when explaining:

- Throttling is keyed by consumer.
- Usage resets or refills over time.
- Over-limit requests get controlled handling.

## 6. Where It Comes Handy in Real Life

- API quotas.
- Tenant fairness.
- Expensive report generation.
- Login attempts.
- Background job submission.
- Downstream dependency protection.
- Cloud cost control.

## 7. Advantages Over Normal Code Without Pattern

Without Throttling:

```text
high demand consumes shared capacity until service degrades
```

With Throttling:

```text
usage is controlled before overload spreads
```

Benefits:

- Protects shared resources.
- Improves fairness.
- Maintains service stability.
- Reduces cost spikes.
- Enables tiered usage plans.

## 8. Where It Excels

- Multi-tenant systems.
- Public APIs.
- Expensive operations.
- Overload must be controlled quickly.
- Service-level objectives matter.

## 9. Where It Fails

- Quotas are too low for legitimate usage.
- Throttle key is incorrect.
- Counters are local but deployment is distributed.
- Clients ignore retry guidance.
- Throttling hides deeper capacity problems.

Throttling should be paired with capacity planning and observability.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Bucket4j, Resilience4j RateLimiter, Guava RateLimiter |
| API edge | Kong, NGINX, Envoy, Spring Cloud Gateway |
| Cloud | API Gateway throttling, Cloudflare rules |
| Store | Redis counters, Lua scripts, distributed cache |
| App | Tenant quota services, usage metering systems |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Protects stability. | Can reject valid traffic. |
| Enforces fair usage. | Requires accurate counters. |
| Controls cost. | Distributed throttling is harder. |
| Supports service tiers. | Bad tuning hurts user experience. |

## 12. Real-World Identification Example

Scenario: A report API can consume heavy CPU and database resources.

Throttling fit:

- Tenant gets a maximum number of report requests per minute.
- Extra requests receive `429`.
- Premium tenants get higher limits.
- Metrics show rejected requests and top consumers.

Without it:

- One tenant can slow reports for everyone.
- Database load spikes unpredictably.

## 13. MAANG Interview Triggers

Use Throttling when you hear:

- "Control resource consumption."
- "Tenant fairness."
- "Protect expensive endpoint."
- "Traffic spike overloads service."
- "How do we enforce quotas?"
- "How do we shed load?"

Strong answer keywords:

- quota
- tenant
- fixed window
- token bucket
- `429`
- retry-after
- load shedding
- fairness
- usage metering

## 14. Common Mistakes

### Mistake 1: Confusing business quotas with system protection

- Why it is wrong: billing quotas and overload protection may need different policies.
- Better approach: separate product limits from operational safety limits.

### Mistake 2: Local counters in distributed service

- Why it is wrong: each instance allows its own quota.
- Better approach: use shared counters or define per-instance limits intentionally.

### Mistake 3: No client guidance

- Why it is wrong: clients retry immediately.
- Better approach: return `Retry-After` and clear error body.

### Mistake 4: No high-priority bypass

- Why it is wrong: critical internal calls can be blocked with low-value traffic.
- Better approach: use priority classes or reserved capacity.

## 15. Throttling vs Similar Patterns

| Pattern | Difference |
|---|---|
| Throttling | Controls resource consumption and request flow. |
| Rate Limiting | Specific request-rate control by key/time. |
| Backpressure | Consumer signals producer to slow down. |
| Load Shedding | Drops work to protect core availability. |
| Circuit Breaker | Blocks failing dependency calls. |

## 16. Throttling Design Checklist

- What resource is protected?
- Who is throttled?
- What quota applies?
- What window or refill model is used?
- Is throttling local or distributed?
- What response is returned?
- Are premium tiers supported?
- How are limits changed safely?
- What metrics show fairness and overload?

## 17. Quick Revision Notes

- One-line summary: Throttling controls resource use to keep systems stable.
- Three keywords: quota, tenant, reject.
- Interview trap: using local counters without considering multiple instances.
- Memory trick: narrow the faucet before the pipe bursts.

## 18. Mini Exercise

Design throttling for a report-generation endpoint.

Answer these:

1. What key is throttled?
2. What quota is allowed?
3. What response is returned when over quota?
4. How do premium users differ?
5. What dashboard would you build?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/throttling/README.md](../../github-repo/throttling/README.md)
- [github-repo/throttling/src/main/java/com/iluwatar/throttling/BarCustomer.java](../../github-repo/throttling/src/main/java/com/iluwatar/throttling/BarCustomer.java)
- [github-repo/throttling/src/main/java/com/iluwatar/throttling/CallsCount.java](../../github-repo/throttling/src/main/java/com/iluwatar/throttling/CallsCount.java)
- [github-repo/throttling/src/main/java/com/iluwatar/throttling/Bartender.java](../../github-repo/throttling/src/main/java/com/iluwatar/throttling/Bartender.java)
- [github-repo/throttling/src/main/java/com/iluwatar/throttling/timer/Throttler.java](../../github-repo/throttling/src/main/java/com/iluwatar/throttling/timer/Throttler.java)
- [github-repo/throttling/src/main/java/com/iluwatar/throttling/timer/ThrottleTimerImpl.java](../../github-repo/throttling/src/main/java/com/iluwatar/throttling/timer/ThrottleTimerImpl.java)
- [github-repo/throttling/src/main/java/com/iluwatar/throttling/App.java](../../github-repo/throttling/src/main/java/com/iluwatar/throttling/App.java)
