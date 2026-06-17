# Fan-Out/Fan-In Pattern

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [fanout-fanin](../../github-repo/fanout-fanin)

---

## How to Study This Page

Study Fan-Out/Fan-In as "split one request into parallel subrequests, then combine the answers."

Remember the core flow:

```text
Input -> Fan out parallel work -> Wait/collect -> Fan in aggregate result
```

This pattern appears constantly in backend aggregation, async APIs, distributed queries, and performance optimization interviews.

---

## 1. Technical Definition

Fan-Out/Fan-In is a concurrency pattern where work is split into multiple independent tasks that execute in parallel, and their outputs are collected into one combined result.

### 30-Second Interview Answer

Fan-Out/Fan-In improves latency when independent subtasks can run at the same time. The system fans out work to workers or services, waits for all required responses or enough responses, and fans in the results through aggregation. I would use it for parallel API calls, search shards, image processing, or recommendation features. The trade-offs are timeouts, partial failure handling, bounded concurrency, and result ordering.

---

## 2. Layman and Easy to Understand Definition

Imagine asking five teammates to check five different documents at the same time.

- You split the work.
- Everyone works in parallel.
- You collect all findings.
- You produce one final answer.

That is Fan-Out/Fan-In.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Sequential processing is slow when independent tasks do not depend on each other.

For example:

```text
Call service A -> wait
Call service B -> wait
Call service C -> wait
Merge result
```

If each call takes 300 ms, total latency can become close to 900 ms.

### Fan-Out/Fan-In Flow

1. Receive a parent request.
2. Split it into independent subtasks.
3. Submit subtasks to workers, threads, services, or shards.
4. Run subtasks concurrently.
5. Collect results.
6. Merge, sort, sum, rank, or validate results.
7. Return one combined response.

### Core Participants

| Participant | Responsibility |
|---|---|
| Coordinator | Splits work and joins results |
| Worker task | Executes one independent unit |
| Executor/pool | Limits concurrent work |
| Future/promise | Represents pending result |
| Aggregator | Combines outputs |
| Timeout policy | Prevents waiting forever |

---

## 4. Java Coding Example

```java
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class FanOutFanInDemo {
    static int fetchScore(String source) {
        return switch (source) {
            case "profile" -> 30;
            case "orders" -> 40;
            case "support" -> 10;
            default -> 0;
        };
    }

    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(3);

        try {
            List<String> sources = List.of("profile", "orders", "support");

            List<CompletableFuture<Integer>> futures = sources.stream()
                    .map(source -> CompletableFuture.supplyAsync(() -> fetchScore(source), pool))
                    .toList();

            CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();

            int total = futures.stream()
                    .mapToInt(CompletableFuture::join)
                    .sum();

            System.out.println("customer score = " + total);
        } finally {
            pool.shutdown();
        }
    }
}
```

### Java Block by Block

Each source can be queried independently.

`CompletableFuture.supplyAsync` fans out work.

`CompletableFuture.allOf` waits for all tasks.

The final stream joins results and fans them into one score.

---

## 5. Python Coding Example

```python
from concurrent.futures import ThreadPoolExecutor, as_completed


def fetch_score(source):
    return {
        "profile": 30,
        "orders": 40,
        "support": 10,
    }.get(source, 0)


sources = ["profile", "orders", "support"]

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [executor.submit(fetch_score, source) for source in sources]
    total = sum(future.result() for future in as_completed(futures))

print("customer score =", total)
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| API aggregation | Query independent downstream services in parallel |
| Search systems | Query shards concurrently and merge ranked results |
| Recommendation systems | Fetch candidates from multiple sources |
| Image/video processing | Split media into chunks and combine processed output |
| Batch validation | Validate independent records concurrently |
| Analytics dashboards | Fetch independent metrics at the same time |

---

## 7. Advantages Over Normal Code Without Pattern

Without Fan-Out/Fan-In:
- independent calls run sequentially
- latency adds up
- coordinator logic is hidden in ad hoc code
- failures and timeouts are inconsistent

With Fan-Out/Fan-In:
- independent work runs concurrently
- total latency approaches the slowest required task
- aggregation becomes explicit
- concurrency limits can be enforced in one place

---

## 8. Where It Excels

It excels when:
- subtasks are independent
- calls are I/O bound
- the response needs a merged result
- latency matters
- partial responses are acceptable or manageable
- concurrency can be bounded safely

---

## 9. Where It Fails

It fails when:
- tasks depend on each other
- fan-out count is unbounded
- one slow dependency blocks the full response
- downstream services cannot handle bursts
- result aggregation is more expensive than the parallel work
- consistency requires a single atomic transaction

Use sequential flow, batching, caching, or queue-based processing when concurrency would overload dependencies.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | CompletableFuture, ExecutorService, ForkJoinPool, Project Reactor |
| Spring | WebClient with reactive composition, async methods |
| Python | concurrent.futures, asyncio.gather |
| JavaScript | Promise.all, Promise.allSettled |
| Distributed systems | Scatter-gather search, shard query coordinators |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces latency for independent work | More concurrency complexity |
| Makes aggregation explicit | Needs careful timeout handling |
| Good for I/O-heavy operations | Can overload downstream systems |
| Works with partial-result strategies | Error handling is more complex |
| Natural fit for shard queries | Result ordering may require extra logic |

---

## 12. Real-World Identification Example

Question:

> A product page needs pricing, inventory, recommendations, and reviews from separate services. Calling them one by one makes the page slow. What pattern helps?

Strong answer:

Use Fan-Out/Fan-In. The product service can fan out parallel calls to pricing, inventory, recommendation, and review services, then fan in the results into one response. I would use bounded concurrency, per-call timeouts, fallbacks for optional sections, and clear rules for required versus optional data.

---

## 13. MAANG Interview Triggers

Say Fan-Out/Fan-In when you hear:
- parallel independent calls
- scatter gather
- query multiple shards
- aggregate multiple service responses
- reduce end-to-end latency
- wait for all futures
- partial response
- bounded concurrency

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Unbounded fan-out | Can exhaust threads or overload services | Use bounded pools and limits |
| No timeout | One slow task blocks the whole response | Set per-task and overall deadlines |
| Treating optional data as required | Availability suffers | Return partial results when acceptable |
| Forgetting executor shutdown | Leaks resources | Manage lifecycle explicitly |
| Ignoring result ordering | Aggregated output may be inconsistent | Preserve identifiers or sort after join |

---

## 15. Fan-Out/Fan-In vs Similar Patterns

| Pattern | Difference |
|---|---|
| Map Reduce | Map Reduce has formal map, shuffle, and reduce phases for batch data |
| Pipeline | Pipeline is staged transformation; Fan-Out/Fan-In is parallel split and join |
| Promise | Promise is an async primitive; Fan-Out/Fan-In is an orchestration pattern |
| API Gateway | Gateway may use Fan-Out/Fan-In internally to aggregate responses |
| Queue-Based Load Leveling | Queue smooths load over time; Fan-Out/Fan-In tries to finish parallel work now |

---

## 16. Fan-Out/Fan-In Design Checklist

- What subtasks can run independently?
- How many tasks can be in flight?
- What executor or worker pool owns the work?
- What is the overall deadline?
- Which results are required?
- Which results are optional?
- How are failures represented?
- How are results merged and ordered?
- How do we prevent downstream overload?

---

## 17. Quick Revision Notes

- One-line summary: Split independent work, run it concurrently, aggregate the results.
- Memory hook: "scatter, wait, gather."
- Best for: parallel I/O and shard/service aggregation.
- Avoid when: tasks depend on each other or fan-out is uncontrolled.
- Interview line: "I would bound concurrency, apply deadlines, and define partial-result behavior up front."

---

## 18. Mini Exercise

Design Fan-Out/Fan-In for a search API:
- split the request across three shards
- define a timeout
- decide how to handle one failed shard
- merge results by score
- explain how to prevent too many concurrent searches

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/fanout-fanin/README.md)
- [FanOutFanIn.java](../../github-repo/fanout-fanin/src/main/java/com/iluwatar/fanout/fanin/FanOutFanIn.java)
- [SquareNumberRequest.java](../../github-repo/fanout-fanin/src/main/java/com/iluwatar/fanout/fanin/SquareNumberRequest.java)
- [Consumer.java](../../github-repo/fanout-fanin/src/main/java/com/iluwatar/fanout/fanin/Consumer.java)
- [App.java](../../github-repo/fanout-fanin/src/main/java/com/iluwatar/fanout/fanin/App.java)

The repo implementation creates delayed square-number tasks, runs them with `CompletableFuture`, waits using `CompletableFuture.allOf`, and aggregates the sum through an atomic consumer.
