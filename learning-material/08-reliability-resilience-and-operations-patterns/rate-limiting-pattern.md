# Rate Limiting Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/rate-limiting-pattern](../../github-repo/rate-limiting-pattern)

## How to Study This Page

Use this page in three passes:

1. First pass: understand rate limiting as controlling how many requests are allowed.
2. Second pass: compare fixed window, sliding window, token bucket, and adaptive limiting.
3. Third pass: focus on distributed counters, fairness, burst control, and `429` responses.

By the end, you should be able to say:

> Rate Limiting protects a system by allowing only a configured amount of traffic per client, key, or operation.

## 1. Technical Definition

Rate Limiting is a resilience and fairness pattern that restricts how many requests a caller can make within a time period or token budget.

Core idea:

- Identify a caller or operation with a limit key.
- Count or budget requests.
- Allow requests under the limit.
- Reject or delay requests above the limit.
- Return clear retry information.

### 30-Second Interview Answer

I would use rate limiting to protect APIs, shared resources, and downstream dependencies from overload or abuse. I would choose a key such as user id, IP, tenant, or API token, then use token bucket for bursts or sliding window for smoother fairness. The trade-off is accuracy versus cost, especially in distributed systems where counters must be shared or approximated.

## 2. Layman and Easy to Understand Definition

Rate limiting is like a turnstile that allows only a certain number of entries per minute.

If too many people arrive, some wait or are rejected so the place does not become overloaded.

In code:

- Request arrives.
- Limiter checks allowance.
- If allowance exists, request continues.
- If not, request gets rejected or delayed.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Traffic is not evenly distributed:

- One client can send too many requests.
- Bots can abuse public endpoints.
- A bug can create request storms.
- A tenant can consume shared capacity.
- Downstream systems can be overloaded.

### 3.2 The Rate Limiting Solution

Add a gate before expensive work:

```text
request -> rate limiter -> allowed -> service
                       -> rejected -> 429
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Limit key | User, IP, tenant, API key, or operation. |
| Limit policy | Allowed requests per time or token budget. |
| Counter/token store | Tracks usage. |
| Decision | Allow, reject, delay, or degrade. |
| Retry metadata | Tells caller when to retry. |

### 3.4 Common Algorithms

| Algorithm | Strength | Weakness |
|---|---|---|
| Fixed window | Simple and cheap. | Allows boundary bursts. |
| Sliding window | Smoother fairness. | More storage or computation. |
| Token bucket | Allows controlled bursts. | Needs token refill logic. |
| Leaky bucket | Smooths output rate. | Can add latency. |
| Adaptive limiter | Responds to system health. | Harder to tune. |

## 4. Java Coding Example

This example implements a small token bucket limiter.

```java
class TokenBucket {
    private final int capacity;
    private final int refillPerSecond;
    private int tokens;
    private long lastRefillMillis;

    TokenBucket(int capacity, int refillPerSecond) {
        this.capacity = capacity;
        this.refillPerSecond = refillPerSecond;
        this.tokens = capacity;
        this.lastRefillMillis = System.currentTimeMillis();
    }

    synchronized boolean allow() {
        refill();
        if (tokens <= 0) {
            return false;
        }
        tokens--;
        return true;
    }

    private void refill() {
        long now = System.currentTimeMillis();
        long elapsedSeconds = (now - lastRefillMillis) / 1000;
        if (elapsedSeconds > 0) {
            tokens = Math.min(capacity, tokens + (int) elapsedSeconds * refillPerSecond);
            lastRefillMillis = now;
        }
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `capacity` | Maximum burst size. |
| `refillPerSecond` | Sustained request rate. |
| `allow` | Decision point before work. |
| `refill` | Restores budget over time. |
| `synchronized` | Protects local counter from races. |

### Java Usage

```java
TokenBucket bucket = new TokenBucket(5, 1);

if (bucket.allow()) {
    System.out.println("request allowed");
} else {
    System.out.println("request rejected with 429");
}
```

## 5. Python Coding Example

```python
import time


class TokenBucket:
    def __init__(self, capacity, refill_per_second):
        self.capacity = capacity
        self.refill_per_second = refill_per_second
        self.tokens = capacity
        self.last_refill = time.time()

    def allow(self):
        now = time.time()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_per_second)
        self.last_refill = now

        if self.tokens < 1:
            return False
        self.tokens -= 1
        return True


bucket = TokenBucket(capacity=5, refill_per_second=1)
print(bucket.allow())
```

### Python Usage

Use this shape when explaining:

- Capacity controls burst.
- Refill controls sustained rate.
- Allow/reject happens before expensive work.

## 6. Where It Comes Handy in Real Life

- Public APIs.
- Login endpoints.
- Search endpoints.
- Payment attempts.
- Webhooks.
- Multi-tenant SaaS.
- Internal service dependencies.
- AI or expensive compute APIs.

## 7. Advantages Over Normal Code Without Pattern

Without Rate Limiting:

```text
any caller can consume unlimited shared capacity
```

With Rate Limiting:

```text
each caller gets controlled capacity
```

Benefits:

- Protects availability.
- Improves fairness.
- Reduces abuse.
- Controls cost.
- Protects downstream dependencies.

## 8. Where It Excels

- Shared systems serve many callers.
- Abuse or accidental spikes are possible.
- You need fairness across tenants.
- Expensive operations must be protected.
- API contracts include quotas.

## 9. Where It Fails

- Limit key is wrong or easy to bypass.
- Distributed counters are inconsistent.
- Legitimate bursts are blocked too aggressively.
- Retry headers are missing.
- Limit is global but users need per-operation fairness.

Rate limits should be observable and adjustable.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Bucket4j, Resilience4j RateLimiter, Guava RateLimiter |
| API gateway | Kong, Envoy, NGINX, Spring Cloud Gateway |
| Cloud | AWS API Gateway usage plans, Cloudflare rate limiting |
| Stores | Redis counters, Redis Lua scripts, DynamoDB conditional writes |
| Service mesh | Envoy local/global rate limit filters |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Protects systems from overload. | Requires correct key design. |
| Improves fairness. | Distributed accuracy is hard. |
| Controls cost and abuse. | Can reject legitimate traffic. |
| Provides predictable quotas. | Adds latency and storage overhead. |

## 12. Real-World Identification Example

Scenario: Public search API is used by thousands of API keys.

Rate Limiting fit:

- Key is API key plus endpoint.
- Token bucket allows short burst.
- Limit is `100 requests/minute`.
- Exceeded calls return `429` and `Retry-After`.

Without it:

- One caller can overload search.
- Other users see high latency.
- Infrastructure cost spikes.

## 13. MAANG Interview Triggers

Use Rate Limiting when you hear:

- "How do we protect public APIs?"
- "Prevent abuse."
- "Fair usage per tenant."
- "How do we handle traffic spikes?"
- "How do we return 429?"
- "Design a rate limiter."

Strong answer keywords:

- token bucket
- fixed window
- sliding window
- limit key
- Redis
- distributed counter
- `429`
- `Retry-After`
- burst

## 14. Common Mistakes

### Mistake 1: Limiting only by IP

- Why it is wrong: NAT, mobile networks, and proxies make IP unfair or bypassable.
- Better approach: use user id, tenant id, API key, or combined keys.

### Mistake 2: No distributed design

- Why it is wrong: each app instance allows its own quota.
- Better approach: use shared counters or accept documented local limits.

### Mistake 3: Missing retry metadata

- Why it is wrong: clients retry immediately and worsen overload.
- Better approach: return `429` with `Retry-After` and quota headers.

### Mistake 4: One limit for all operations

- Why it is wrong: cheap and expensive endpoints consume the same budget.
- Better approach: define operation-specific limits or weighted costs.

## 15. Rate Limiting vs Similar Patterns

| Pattern | Difference |
|---|---|
| Rate Limiting | Controls requests per key over time. |
| Throttling | Broader resource consumption control, often tenant or system focused. |
| Backpressure | Downstream tells upstream to slow production. |
| Circuit Breaker | Stops calls to failing dependencies. |
| Queue Load Leveling | Buffers bursts instead of rejecting immediately. |

## 16. Rate Limiting Design Checklist

- What is the limit key?
- What is the algorithm?
- What is the sustained rate?
- What burst is allowed?
- Is the limit local or distributed?
- What happens when limit is exceeded?
- What headers are returned?
- How are premium tiers handled?
- What metrics show allowed and rejected requests?

## 17. Quick Revision Notes

- One-line summary: Rate Limiting protects capacity by controlling request rate per key.
- Three keywords: key, bucket, `429`.
- Interview trap: ignoring distributed counters.
- Memory trick: request budget refills over time.

## 18. Mini Exercise

Design a rate limiter for login attempts.

Answer these:

1. What key is used?
2. What algorithm is used?
3. What is the limit?
4. What response is returned when exceeded?
5. How do you avoid blocking all users behind one IP?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/rate-limiting-pattern/README.md](../../github-repo/rate-limiting-pattern/README.md)
- [github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/RateLimiter.java](../../github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/RateLimiter.java)
- [github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/TokenBucketRateLimiter.java](../../github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/TokenBucketRateLimiter.java)
- [github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/FixedWindowRateLimiter.java](../../github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/FixedWindowRateLimiter.java)
- [github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/AdaptiveRateLimiter.java](../../github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/AdaptiveRateLimiter.java)
- [github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/FindCustomerRequest.java](../../github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/FindCustomerRequest.java)
- [github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/App.java](../../github-repo/rate-limiting-pattern/src/main/java/com/iluwatar/rate/limiting/pattern/App.java)
