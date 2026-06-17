# Master-Worker Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/master-worker](../../github-repo/master-worker)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the master as coordinator and workers as parallel executors.
2. Second pass: trace split, assign, execute, collect, and merge.
3. Third pass: compare Master-Worker with Thread Pool, Fork-Join, MapReduce, and Producer-Consumer.

By the end, you should be able to say:

> Master-Worker splits a larger job into independent pieces, runs them in parallel, and combines the results.

## 1. Technical Definition

Master-Worker is a parallel processing pattern where a master component divides work into subtasks, distributes them to workers, and aggregates their results.

Core idea:

- Master owns coordination.
- Work is decomposed into independent tasks.
- Workers execute tasks concurrently.
- Results are collected.
- Master merges final result.

### 30-Second Interview Answer

I would use Master-Worker when a large task can be split into independent subtasks, such as image processing, matrix operations, crawling, or batch computation. The master partitions work, assigns tasks to workers, tracks progress and failures, and merges results. The trade-off is coordination overhead and load balancing complexity.

## 2. Layman and Easy to Understand Definition

Master-Worker is like a project lead splitting a big job among several teammates.

Each teammate works on a piece. The lead gathers the results and creates the final answer.

In code:

- Master divides input.
- Workers process partitions.
- Master waits for results.
- Master merges output.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

One thread may take too long for large independent work:

```text
process 1 million records sequentially
```

If records can be split, parallel workers can finish faster.

### 3.2 The Master-Worker Solution

```text
input -> master -> worker 1
                -> worker 2
                -> worker 3
results -> master -> final result
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Master | Splits work and merges results. |
| Worker | Executes assigned subtask. |
| Input partition | Portion of the original work. |
| Result | Worker output. |
| Scheduler | Assigns tasks to workers. |
| Aggregator | Combines results. |

### 3.4 Failure Flow

1. Worker fails or times out.
2. Master detects missing result.
3. Master retries task or assigns another worker.
4. Master marks job failed if retry budget is exhausted.

## 4. Java Coding Example

```java
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class SumMaster {
    private final ExecutorService pool = Executors.newFixedThreadPool(4);

    int sum(List<Integer> numbers) throws Exception {
        var first = pool.submit(() -> numbers.subList(0, numbers.size() / 2).stream().mapToInt(i -> i).sum());
        var second = pool.submit(() -> numbers.subList(numbers.size() / 2, numbers.size()).stream().mapToInt(i -> i).sum());
        return first.get() + second.get();
    }

    void shutdown() {
        pool.shutdown();
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `SumMaster` | Coordinates work. |
| `subList` | Splits input into partitions. |
| `submit` | Assigns work to workers. |
| `Future.get` | Collects worker result. |
| addition | Aggregates final result. |

### Java Usage

```java
SumMaster master = new SumMaster();
System.out.println(master.sum(List.of(1, 2, 3, 4)));
master.shutdown();
```

## 5. Python Coding Example

```python
from concurrent.futures import ThreadPoolExecutor


def parallel_sum(numbers):
    middle = len(numbers) // 2
    with ThreadPoolExecutor(max_workers=2) as pool:
        left = pool.submit(sum, numbers[:middle])
        right = pool.submit(sum, numbers[middle:])
        return left.result() + right.result()


print(parallel_sum([1, 2, 3, 4]))
```

### Python Usage

Use this shape when explaining:

- Master partitions input.
- Workers process in parallel.
- Master combines results.

## 6. Where It Comes Handy in Real Life

- Batch data processing.
- Matrix operations.
- Web crawling.
- File conversion.
- Parallel search.
- Distributed computation.
- MapReduce-style workloads.

## 7. Advantages Over Normal Code Without Pattern

Without Master-Worker:

```text
single worker processes everything
```

With Master-Worker:

```text
many workers process partitions concurrently
```

Benefits:

- Faster processing for splittable work.
- Clear coordination role.
- Scales workers independently.
- Natural fit for distributed execution.
- Better utilization of cores or machines.

## 8. Where It Excels

- Work can be partitioned.
- Subtasks are mostly independent.
- Results can be merged.
- Worker count can be tuned.
- Failures can be retried per task.

## 9. Where It Fails

- Work cannot be split.
- Merge step dominates runtime.
- Partitions are uneven.
- Workers share too much mutable state.
- Master becomes a bottleneck.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | `ExecutorService`, ForkJoinPool, parallel streams |
| Distributed | Hadoop MapReduce, Spark, Flink |
| Workflow | Temporal, Airflow, Argo Workflows |
| Python | `concurrent.futures`, multiprocessing |
| Cloud | Batch jobs, serverless fan-out/fan-in |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Parallelizes large work. | Adds coordination overhead. |
| Scales worker count. | Requires good partitioning. |
| Isolates task execution. | Master can bottleneck. |
| Supports retry per task. | Result aggregation can be complex. |

## 12. Real-World Identification Example

Scenario: A service must transpose huge matrices.

Master-Worker fit:

- Master divides matrix rows or blocks.
- Workers transpose partitions.
- Master merges blocks into final matrix.
- Failed blocks can be retried.

Without it:

- One thread performs all computation slowly.

## 13. MAANG Interview Triggers

Use Master-Worker when you hear:

- "Split large task into smaller tasks."
- "Parallel processing."
- "Fan-out and fan-in."
- "Aggregate worker results."
- "How do we distribute computation?"
- "MapReduce-like."

Strong answer keywords:

- partition
- worker
- coordinator
- fan-out
- fan-in
- aggregation
- load balancing
- retry

## 14. Common Mistakes

### Mistake 1: Uneven partitioning

- Why it is wrong: one slow worker delays the whole job.
- Better approach: create balanced partitions or dynamic task stealing.

### Mistake 2: Shared mutable state

- Why it is wrong: workers race and corrupt results.
- Better approach: isolate worker input and merge immutable results.

### Mistake 3: Ignoring worker failure

- Why it is wrong: one failed task loses the job.
- Better approach: track task status and retry failed partitions.

### Mistake 4: Too many tiny tasks

- Why it is wrong: coordination overhead dominates.
- Better approach: choose partition size based on work cost.

## 15. Master-Worker vs Similar Patterns

| Pattern | Difference |
|---|---|
| Master-Worker | Master splits work and aggregates worker results. |
| Thread Pool | Generic reusable workers for tasks. |
| Fork-Join | Recursive task splitting and joining. |
| Producer-Consumer | Producer creates items consumed from a queue. |
| MapReduce | Distributed master-worker style for map and reduce phases. |

## 16. Master-Worker Design Checklist

- Can the input be partitioned?
- How many workers are needed?
- How are partitions assigned?
- How are results merged?
- What happens when a worker fails?
- Is work balanced?
- Is shared state avoided?
- What metrics track progress?

## 17. Quick Revision Notes

- One-line summary: Master-Worker splits work, runs pieces in parallel, and merges results.
- Three keywords: split, workers, merge.
- Interview trap: ignoring skew and straggler workers.
- Memory trick: one coordinator, many executors.

## 18. Mini Exercise

Design master-worker processing for generating thumbnails.

Answer these:

1. How is input split?
2. How many workers run?
3. What does each worker return?
4. How are failures retried?
5. What makes one worker a straggler?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/master-worker/README.md](../../github-repo/master-worker/README.md)
- [github-repo/master-worker/src/main/java/com/iluwatar/masterworker/system/MasterWorker.java](../../github-repo/master-worker/src/main/java/com/iluwatar/masterworker/system/MasterWorker.java)
- [github-repo/master-worker/src/main/java/com/iluwatar/masterworker/system/systemmaster/Master.java](../../github-repo/master-worker/src/main/java/com/iluwatar/masterworker/system/systemmaster/Master.java)
- [github-repo/master-worker/src/main/java/com/iluwatar/masterworker/system/systemworkers/Worker.java](../../github-repo/master-worker/src/main/java/com/iluwatar/masterworker/system/systemworkers/Worker.java)
- [github-repo/master-worker/src/main/java/com/iluwatar/masterworker/Input.java](../../github-repo/master-worker/src/main/java/com/iluwatar/masterworker/Input.java)
- [github-repo/master-worker/src/main/java/com/iluwatar/masterworker/Result.java](../../github-repo/master-worker/src/main/java/com/iluwatar/masterworker/Result.java)
- [github-repo/master-worker/src/main/java/com/iluwatar/masterworker/App.java](../../github-repo/master-worker/src/main/java/com/iluwatar/masterworker/App.java)
