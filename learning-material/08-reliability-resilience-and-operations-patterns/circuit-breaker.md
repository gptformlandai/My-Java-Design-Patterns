# Circuit Breaker Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/circuit-breaker](../../github-repo/circuit-breaker)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why repeated calls to a failing dependency make failures worse.
2. Second pass: trace closed, open, and half-open states.
3. Third pass: compare Circuit Breaker with Retry, Timeout, Bulkhead, and Rate Limiting.

By the end, you should be able to say:

> Circuit Breaker stops calling a failing dependency temporarily so the system can fail fast and recover safely.

## 1. Technical Definition

Circuit Breaker is a resilience pattern that monitors calls to a dependency and temporarily blocks calls when failures exceed a threshold.

Core idea:

- Closed state allows calls.
- Failure count or error rate is tracked.
- Open state fails fast without calling dependency.
- Half-open state probes recovery.
- Success closes the circuit; failure opens it again.

### 30-Second Interview Answer

I would use Circuit Breaker around remote calls that can fail, hang, or overload callers. It starts closed, opens when failures cross a threshold, and later moves to half-open to test recovery. The trade-off is temporary degraded functionality, but it prevents cascading failures, thread exhaustion, and retry storms.

## 2. Layman and Easy to Understand Definition

Circuit Breaker is like an electrical breaker.

When a downstream system is failing, the breaker trips so the current system does not keep sending more load into danger. After a cooling period, it tests whether things are safe again.

In code:

- Call dependency while healthy.
- Count failures.
- Stop calls when failures are too high.
- Try a small test call later.
- Resume when healthy.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Remote dependencies can fail slowly:

```text
service A -> service B timeout
service A -> service B timeout
service A -> service B timeout
```

Repeated calls consume threads, connection pools, and queues.

### 3.2 The Circuit Breaker Solution

Add a stateful protection layer:

```text
closed -> calls allowed
open -> calls blocked
half-open -> limited probe calls
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Protected dependency | Remote service, database, or API. |
| Circuit breaker | Wrapper that tracks outcomes. |
| Failure threshold | Count or rate that opens the circuit. |
| Open duration | Cooldown before probe. |
| Fallback | Safe degraded response. |
| Metrics | Error rate, latency, state changes. |

### 3.4 State Flow

1. Closed: calls pass through.
2. Failures increase.
3. Threshold is crossed.
4. Circuit opens and fails fast.
5. Cooldown expires.
6. Half-open sends limited probe.
7. Probe success closes; probe failure reopens.

## 4. Java Coding Example

This example shows a small breaker around a remote call.

```java
enum CircuitState {
    CLOSED, OPEN, HALF_OPEN
}

class SimpleCircuitBreaker {
    private CircuitState state = CircuitState.CLOSED;
    private int failures = 0;
    private long openedAt = 0;

    String call(RemoteCall remote) {
        if (state == CircuitState.OPEN) {
            if (System.currentTimeMillis() - openedAt < 5000) {
                return "fallback";
            }
            state = CircuitState.HALF_OPEN;
        }

        try {
            String result = remote.execute();
            failures = 0;
            state = CircuitState.CLOSED;
            return result;
        } catch (RuntimeException failure) {
            failures++;
            if (failures >= 3 || state == CircuitState.HALF_OPEN) {
                state = CircuitState.OPEN;
                openedAt = System.currentTimeMillis();
            }
            return "fallback";
        }
    }
}

interface RemoteCall {
    String execute();
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `CircuitState` | Breaker state machine. |
| `failures` | Failure tracking. |
| `openedAt` | Cooldown timing. |
| `OPEN` branch | Fails fast without remote call. |
| `HALF_OPEN` | Tests recovery. |

### Java Usage

```java
SimpleCircuitBreaker breaker = new SimpleCircuitBreaker();
String response = breaker.call(() -> {
    throw new RuntimeException("remote down");
});

System.out.println(response);
```

## 5. Python Coding Example

```python
import time


class CircuitBreaker:
    def __init__(self, threshold=3, cooldown=5):
        self.threshold = threshold
        self.cooldown = cooldown
        self.state = "closed"
        self.failures = 0
        self.opened_at = 0

    def call(self, func):
        if self.state == "open":
            if time.time() - self.opened_at < self.cooldown:
                return "fallback"
            self.state = "half-open"

        try:
            value = func()
            self.failures = 0
            self.state = "closed"
            return value
        except Exception:
            self.failures += 1
            if self.failures >= self.threshold or self.state == "half-open":
                self.state = "open"
                self.opened_at = time.time()
            return "fallback"
```

### Python Usage

Use this shape when explaining:

- Open means fail fast.
- Half-open means controlled recovery probe.
- Fallback protects user experience.

## 6. Where It Comes Handy in Real Life

- Payment provider calls.
- Inventory service calls.
- Search dependency.
- External API integration.
- Database dependency under incident.
- API Gateway downstream calls.

## 7. Advantages Over Normal Code Without Pattern

Without Circuit Breaker:

```text
every request keeps waiting on failing dependency
```

With Circuit Breaker:

```text
calls fail fast and dependency gets recovery time
```

Benefits:

- Prevents cascading failure.
- Reduces thread and connection exhaustion.
- Gives dependency recovery time.
- Enables graceful fallback.
- Makes dependency health visible.

## 8. Where It Excels

- Remote dependency can fail or slow down.
- Caller has fallback behavior.
- Timeouts alone are not enough.
- High traffic can amplify dependency failures.
- You need automatic recovery probing.

## 9. Where It Fails

- No fallback exists and all calls are mandatory.
- Failure threshold is too sensitive.
- Half-open allows too many probe calls.
- It wraps local deterministic code.
- It is combined with aggressive retry without limits.

Circuit Breaker needs timeout, metrics, and sensible fallback.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Resilience4j, Failsafe, Spring Cloud CircuitBreaker |
| Legacy | Netflix Hystrix |
| Service mesh | Envoy, Istio outlier detection |
| Observability | Prometheus, Micrometer, OpenTelemetry |
| Cloud | AWS App Mesh, API Gateway integrations |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Prevents cascading failures. | Adds tuning complexity. |
| Fails fast during incidents. | Can block calls during partial recovery. |
| Supports fallback behavior. | Requires good metrics. |
| Protects caller resources. | Poor thresholds can create false opens. |

## 12. Real-World Identification Example

Scenario: Checkout depends on payment provider. Provider starts timing out.

Circuit Breaker fit:

- Payment calls timeout and count as failures.
- Breaker opens after threshold.
- Checkout returns "payment temporarily unavailable."
- Half-open probes later.
- Breaker closes when provider recovers.

Without it:

- Checkout threads wait on every call.
- Queue grows.
- Other endpoints become slow.

## 13. MAANG Interview Triggers

Use Circuit Breaker when you hear:

- "Downstream service is failing."
- "Avoid cascading failures."
- "Remote dependency timeout."
- "How do we fail fast?"
- "How does service recover automatically?"
- "How do we protect thread pools?"

Strong answer keywords:

- closed
- open
- half-open
- fallback
- timeout
- failure threshold
- recovery window
- cascading failure

## 14. Common Mistakes

### Mistake 1: No timeout before breaker

- Why it is wrong: calls can hang before failure is counted.
- Better approach: always pair breaker with strict timeouts.

### Mistake 2: Retrying through an open breaker

- Why it is wrong: callers create retry storms and waste resources.
- Better approach: fail fast and respect breaker state.

### Mistake 3: No fallback design

- Why it is wrong: users get raw technical failure.
- Better approach: return cached, partial, queued, or user-friendly degraded response.

### Mistake 4: One shared breaker for unrelated calls

- Why it is wrong: one failing dependency blocks healthy paths.
- Better approach: isolate breakers by dependency and operation.

## 15. Circuit Breaker vs Similar Patterns

| Pattern | Difference |
|---|---|
| Circuit Breaker | Stops calls to a failing dependency temporarily. |
| Retry | Repeats transient failures. |
| Timeout | Limits duration of one call. |
| Bulkhead | Isolates resource pools. |
| Rate Limiting | Limits request rate before overload. |

## 16. Circuit Breaker Design Checklist

- Which dependency is protected?
- What timeout applies?
- What failures count?
- What threshold opens the circuit?
- How long does it stay open?
- How many half-open probes are allowed?
- What fallback is returned?
- What metrics and alerts show breaker state?

## 17. Quick Revision Notes

- One-line summary: Circuit Breaker prevents repeated calls to failing dependencies.
- Three keywords: open, half-open, fallback.
- Interview trap: using retry without circuit breaker during dependency outages.
- Memory trick: trip, cool down, test, reset.

## 18. Mini Exercise

Design a circuit breaker for a recommendation service.

Answer these:

1. What timeout applies?
2. What failure threshold opens the circuit?
3. What fallback response is returned?
4. How long before half-open probe?
5. What metric alerts the team?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/circuit-breaker/README.md](../../github-repo/circuit-breaker/README.md)
- [github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/CircuitBreaker.java](../../github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/CircuitBreaker.java)
- [github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/DefaultCircuitBreaker.java](../../github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/DefaultCircuitBreaker.java)
- [github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/State.java](../../github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/State.java)
- [github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/RemoteService.java](../../github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/RemoteService.java)
- [github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/MonitoringService.java](../../github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/MonitoringService.java)
- [github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/App.java](../../github-repo/circuit-breaker/src/main/java/com/iluwatar/circuitbreaker/App.java)
