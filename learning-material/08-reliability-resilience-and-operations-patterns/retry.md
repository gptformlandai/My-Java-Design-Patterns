# Retry Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/retry](../../github-repo/retry)

## How to Study This Page

Use this page in three passes:

1. First pass: understand retry as a response to transient failure.
2. Second pass: trace max attempts, delay, backoff, jitter, and retryable exceptions.
3. Third pass: compare Retry with Circuit Breaker, Timeout, Queue, and Idempotency.

By the end, you should be able to say:

> Retry repeats an operation that failed due to a temporary condition, but only with limits and safe retry rules.

## 1. Technical Definition

Retry is a resilience pattern that re-attempts failed operations when failures are likely transient and the operation is safe to repeat.

Core idea:

- Detect retryable failures.
- Retry only a limited number of times.
- Wait between attempts.
- Use exponential backoff and jitter.
- Stop on non-retryable failures.

### 30-Second Interview Answer

I would use Retry for transient failures like temporary network errors, timeouts, or `503` responses. I would cap attempts, use exponential backoff with jitter, retry only idempotent or idempotency-key-protected operations, and combine it with timeouts and circuit breakers. The trade-off is that retries can amplify load if done blindly.

## 2. Layman and Easy to Understand Definition

Retry is trying again when something probably failed temporarily.

If a network call fails once, trying again may succeed. But trying forever can make the system worse.

In code:

- Try operation.
- If retryable failure happens, wait.
- Try again.
- Stop after max attempts.
- Return success or final failure.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Distributed systems fail transiently:

- Packet loss.
- Temporary DNS issue.
- Brief database failover.
- Short dependency overload.
- Connection reset.

Immediate failure may be unnecessary if the next attempt would succeed.

### 3.2 The Retry Solution

Add controlled repeated attempts:

```text
attempt 1 fails
wait 100 ms
attempt 2 fails
wait 200 ms
attempt 3 succeeds
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Operation | Work being attempted. |
| Retry policy | Max attempts, delays, and exception rules. |
| Backoff | Increasing delay between attempts. |
| Jitter | Randomness to avoid synchronized retries. |
| Idempotency | Safety property for repeated operations. |
| Timeout budget | Total time allowed for all attempts. |

### 3.4 Failure Classification

| Retry? | Example |
|---|---|
| Yes | Timeout, connection reset, `429`, `503`. |
| Maybe | `500`, depending on operation and API contract. |
| No | Validation error, authorization failure, not found for stable resource. |

## 4. Java Coding Example

This example retries a callable with exponential backoff.

```java
import java.util.concurrent.Callable;

class RetryPolicy {
    <T> T run(Callable<T> operation, int maxAttempts, long initialDelayMillis) throws Exception {
        long delay = initialDelayMillis;

        for (int attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                return operation.call();
            } catch (Exception failure) {
                if (attempt == maxAttempts || !isRetryable(failure)) {
                    throw failure;
                }
                Thread.sleep(delay);
                delay *= 2;
            }
        }

        throw new IllegalStateException("unreachable");
    }

    private boolean isRetryable(Exception failure) {
        return failure instanceof TransientFailure;
    }
}

class TransientFailure extends Exception {
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `maxAttempts` | Prevents infinite retry. |
| `initialDelayMillis` | First wait before retry. |
| `delay *= 2` | Exponential backoff. |
| `isRetryable` | Not every error should be retried. |
| final `throw` | Caller sees final failure. |

### Java Usage

```java
RetryPolicy retry = new RetryPolicy();

String value = retry.run(() -> "success", 3, 100);
System.out.println(value);
```

## 5. Python Coding Example

```python
import time


class TransientFailure(Exception):
    pass


def retry(operation, max_attempts=3, initial_delay=0.1):
    delay = initial_delay
    for attempt in range(1, max_attempts + 1):
        try:
            return operation()
        except TransientFailure:
            if attempt == max_attempts:
                raise
            time.sleep(delay)
            delay *= 2


print(retry(lambda: "success"))
```

### Python Usage

Use this shape when explaining:

- Retry only transient failures.
- Stop after max attempts.
- Backoff prevents hammering a weak dependency.

## 6. Where It Comes Handy in Real Life

- HTTP client calls.
- Database failover windows.
- Message publishing.
- Object storage calls.
- Payment provider transient errors.
- Service discovery lookup.
- Distributed lock acquisition.

## 7. Advantages Over Normal Code Without Pattern

Without Retry:

```text
one temporary failure becomes user-visible failure
```

With Retry:

```text
temporary failure can recover within bounded attempts
```

Benefits:

- Improves success rate for transient failures.
- Hides brief network hiccups.
- Reduces user-visible errors.
- Works well with backoff and idempotency.
- Makes failure handling consistent.

## 8. Where It Excels

- Failure is likely temporary.
- Operation is idempotent.
- Dependency has brief overload or failover.
- Total latency budget allows retry.
- Backoff and jitter are used.

## 9. Where It Fails

- Operation is not safe to repeat.
- Failure is permanent.
- No max attempts exist.
- All clients retry at the same time.
- Retry timeout exceeds user latency budget.
- Circuit breaker is missing during outages.

Retry is a sharp tool: helpful for brief failure, harmful during persistent failure.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Resilience4j Retry, Failsafe, Spring Retry |
| HTTP clients | OkHttp retry controls, Apache HttpClient, WebClient filters |
| Messaging | Broker redelivery policy, dead-letter queues |
| Cloud SDKs | AWS SDK retry policies, Google Cloud retry settings |
| Patterns | Exponential backoff, jitter, retry budget |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Handles transient failures. | Can amplify overload. |
| Improves success rate. | Adds latency. |
| Centralizes failure policy. | Unsafe for non-idempotent operations. |
| Works well with backoff. | Needs careful exception classification. |

## 12. Real-World Identification Example

Scenario: Order service calls inventory service and occasionally gets connection reset.

Retry fit:

- Connection reset is retryable.
- Operation uses idempotency key.
- Retry attempts are capped at 3.
- Backoff uses jitter.
- Total request timeout still stays under budget.

Without it:

- Temporary network blips create checkout failures.

## 13. MAANG Interview Triggers

Use Retry when you hear:

- "Transient failure."
- "Network call sometimes fails."
- "How do we handle temporary `503`?"
- "What if dependency fails briefly?"
- "How do we retry safely?"
- "Exponential backoff."

Strong answer keywords:

- max attempts
- exponential backoff
- jitter
- idempotency key
- retryable exception
- timeout budget
- retry storm
- dead-letter queue

## 14. Common Mistakes

### Mistake 1: Retrying every exception

- Why it is wrong: permanent failures waste time and load.
- Better approach: retry only known transient failures.

### Mistake 2: No backoff or jitter

- Why it is wrong: many clients retry together and overload dependency.
- Better approach: use exponential backoff with jitter.

### Mistake 3: Retrying non-idempotent operations

- Why it is wrong: duplicate side effects can happen.
- Better approach: use idempotency keys or do not retry.

### Mistake 4: Ignoring total timeout budget

- Why it is wrong: retries can exceed user latency expectations.
- Better approach: set per-attempt and overall timeouts.

## 15. Retry vs Similar Patterns

| Pattern | Difference |
|---|---|
| Retry | Repeats transient failures. |
| Circuit Breaker | Stops calls during persistent failure. |
| Timeout | Bounds each attempt duration. |
| Queue | Defers work asynchronously. |
| Idempotent Consumer | Makes repeated message processing safe. |

## 16. Retry Design Checklist

- Which errors are retryable?
- Is the operation idempotent?
- What is max attempts?
- What backoff strategy is used?
- Is jitter included?
- What is total timeout budget?
- What happens after all retries fail?
- Are retry metrics tracked?
- Is circuit breaker also needed?

## 17. Quick Revision Notes

- One-line summary: Retry handles transient failure with bounded repeated attempts.
- Three keywords: transient, backoff, idempotent.
- Interview trap: retrying non-idempotent writes without idempotency keys.
- Memory trick: try again, but not forever and not all at once.

## 18. Mini Exercise

Design retry for payment authorization.

Answer these:

1. Which failures are retryable?
2. How many attempts are allowed?
3. What idempotency key is used?
4. What is the total timeout?
5. When does the request stop retrying?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/retry/README.md](../../github-repo/retry/README.md)
- [github-repo/retry/src/main/java/com/iluwatar/retry/BusinessOperation.java](../../github-repo/retry/src/main/java/com/iluwatar/retry/BusinessOperation.java)
- [github-repo/retry/src/main/java/com/iluwatar/retry/Retry.java](../../github-repo/retry/src/main/java/com/iluwatar/retry/Retry.java)
- [github-repo/retry/src/main/java/com/iluwatar/retry/RetryExponentialBackoff.java](../../github-repo/retry/src/main/java/com/iluwatar/retry/RetryExponentialBackoff.java)
- [github-repo/retry/src/main/java/com/iluwatar/retry/FindCustomer.java](../../github-repo/retry/src/main/java/com/iluwatar/retry/FindCustomer.java)
- [github-repo/retry/src/main/java/com/iluwatar/retry/BusinessException.java](../../github-repo/retry/src/main/java/com/iluwatar/retry/BusinessException.java)
- [github-repo/retry/src/main/java/com/iluwatar/retry/App.java](../../github-repo/retry/src/main/java/com/iluwatar/retry/App.java)
