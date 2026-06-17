# Backpressure Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/backpressure](../../github-repo/backpressure)

## How to Study This Page

Use this page in three passes:

1. First pass: understand backpressure as downstream demand controlling upstream production.
2. Second pass: trace producer, subscriber, request count, processing speed, and next demand signal.
3. Third pass: compare Backpressure with Rate Limiting, Throttling, Queues, and Load Shedding.

By the end, you should be able to say:

> Backpressure prevents overload by letting consumers control how much data producers send.

## 1. Technical Definition

Backpressure is a flow-control pattern where a consumer signals its capacity or demand to an upstream producer so the producer does not overwhelm it.

Core idea:

- Producer can emit faster than consumer can process.
- Consumer communicates demand or capacity.
- Producer slows down, buffers, drops, or stops.
- System avoids unbounded queues and memory pressure.
- Throughput aligns with downstream capacity.

### 30-Second Interview Answer

I would use backpressure when producers can generate events faster than consumers can process them, especially in streaming, reactive, and message-processing systems. Consumers explicitly request demand, or the system uses bounded queues and policies to slow, drop, or buffer. The trade-off is higher latency or reduced throughput, but it prevents memory exhaustion and cascading overload.

## 2. Layman and Easy to Understand Definition

Backpressure is like a worker saying, "Do not send me more boxes until I finish these."

Instead of letting boxes pile up forever, the receiver controls the pace.

In code:

- Producer has data.
- Consumer processes slowly.
- Consumer requests only what it can handle.
- Producer respects demand.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Fast producers can overwhelm slow consumers:

```text
producer emits 10,000 events/sec
consumer processes 1,000 events/sec
queue grows forever
memory grows forever
service fails
```

### 3.2 The Backpressure Solution

Consumer controls demand:

```text
consumer: send 10 items
producer: sends 10
consumer: processes 5
consumer: send 5 more
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Producer | Emits data or tasks. |
| Consumer | Processes data or tasks. |
| Demand signal | Number of items consumer is ready for. |
| Buffer | Temporary storage between producer and consumer. |
| Policy | Slow, drop, buffer, or fail when overloaded. |

### 3.4 Common Strategies

| Strategy | Meaning |
|---|---|
| Pull-based demand | Consumer requests N items. |
| Bounded queue | Producer blocks or fails when queue is full. |
| Drop policy | Discard low-value items. |
| Sampling | Keep representative data. |
| Windowing | Process data in batches. |

## 4. Java Coding Example

This example shows a consumer requesting bounded batches.

```java
import java.util.ArrayDeque;
import java.util.Queue;

class DemandPublisher {
    private final Queue<Integer> items = new ArrayDeque<>();

    DemandPublisher(int count) {
        for (int i = 1; i <= count; i++) {
            items.add(i);
        }
    }

    void publish(int demand, SlowSubscriber subscriber) {
        for (int i = 0; i < demand && !items.isEmpty(); i++) {
            subscriber.onNext(items.poll());
        }
    }
}

class SlowSubscriber {
    void onNext(Integer item) {
        System.out.println("processed " + item);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `demand` | Consumer-controlled request count. |
| `publish` | Producer sends only requested amount. |
| `items` | Pending data remains upstream. |
| `SlowSubscriber` | Consumer processes at its pace. |
| loop condition | Prevents unlimited emission. |

### Java Usage

```java
DemandPublisher publisher = new DemandPublisher(100);
SlowSubscriber subscriber = new SlowSubscriber();

publisher.publish(10, subscriber);
publisher.publish(5, subscriber);
```

## 5. Python Coding Example

```python
class Publisher:
    def __init__(self, items):
        self.items = list(items)

    def publish(self, demand, subscriber):
        for _ in range(min(demand, len(self.items))):
            subscriber(self.items.pop(0))


def slow_subscriber(item):
    print(f"processed {item}")


publisher = Publisher(range(1, 101))
publisher.publish(10, slow_subscriber)
publisher.publish(5, slow_subscriber)
```

### Python Usage

Use this shape when explaining:

- Consumer asks for a limited amount.
- Producer does not flood consumer.
- Pending work stays bounded or upstream.

## 6. Where It Comes Handy in Real Life

- Reactive streams.
- Kafka consumers.
- WebSocket streams.
- Log ingestion pipelines.
- Data processing jobs.
- API response streaming.
- Message consumers with slow downstream dependencies.

## 7. Advantages Over Normal Code Without Pattern

Without Backpressure:

```text
producer floods consumer until queues and memory explode
```

With Backpressure:

```text
producer sends only what consumer can handle
```

Benefits:

- Prevents memory exhaustion.
- Stabilizes throughput.
- Protects slow consumers.
- Makes overload behavior explicit.
- Reduces cascading failure.

## 8. Where It Excels

- Producer and consumer speeds differ.
- Streams are long-running.
- Queues must stay bounded.
- Consumer capacity changes over time.
- Data can be paused, buffered, sampled, or dropped.

## 9. Where It Fails

- Producer cannot slow down.
- Data cannot be dropped or delayed.
- Buffers are unbounded.
- Demand signals are ignored.
- Multiple downstream consumers have different speeds.

When source cannot slow down, use bounded queues, shedding, or durable buffering.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Project Reactor, RxJava, Akka Streams |
| Standard | Reactive Streams specification, Java Flow API |
| Messaging | Kafka pause/resume, consumer lag controls |
| Async | CompletableFuture with bounded executors |
| Infrastructure | Bounded queues, semaphores, bulkheads |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Prevents consumer overload. | Can increase latency. |
| Keeps memory bounded. | Requires producer cooperation. |
| Stabilizes pipelines. | Drop policies can lose data. |
| Makes capacity explicit. | Harder with multiple downstreams. |

## 12. Real-World Identification Example

Scenario: Log ingestion service receives more logs than indexing service can write.

Backpressure fit:

- Indexing service consumes in bounded batches.
- Ingestion slows or buffers when consumer lag grows.
- Low-priority logs may be sampled.
- Memory stays bounded.

Without it:

- Queue grows until service crashes.
- Indexing falls further behind.

## 13. MAANG Interview Triggers

Use Backpressure when you hear:

- "Producer is faster than consumer."
- "Queue grows without bound."
- "Streaming system overload."
- "How do we prevent memory pressure?"
- "Consumer lag keeps increasing."
- "Reactive streams."

Strong answer keywords:

- demand
- bounded queue
- consumer capacity
- flow control
- drop policy
- pause/resume
- lag
- buffer

## 14. Common Mistakes

### Mistake 1: Unbounded queues

- Why it is wrong: memory becomes the hidden failure point.
- Better approach: use bounded buffers with explicit overflow policy.

### Mistake 2: Ignoring demand signals

- Why it is wrong: consumer still gets overloaded.
- Better approach: make producer respect request counts or pause signals.

### Mistake 3: Treating backpressure as retry

- Why it is wrong: retry repeats failed work; backpressure slows production.
- Better approach: use backpressure before overload becomes failure.

### Mistake 4: No overload policy

- Why it is wrong: system behavior is accidental under stress.
- Better approach: choose block, drop, sample, queue, or fail-fast deliberately.

## 15. Backpressure vs Similar Patterns

| Pattern | Difference |
|---|---|
| Backpressure | Consumer controls upstream rate. |
| Rate Limiting | Gate controls request rate by key. |
| Throttling | Controls resource consumption. |
| Queue Load Leveling | Buffers bursts between producers and consumers. |
| Circuit Breaker | Stops calls to failing dependencies. |

## 16. Backpressure Design Checklist

- Who is the producer?
- Who is the consumer?
- How is demand signaled?
- Is the buffer bounded?
- What happens when buffer fills?
- Can data be dropped or sampled?
- How is consumer lag measured?
- How are slow downstreams isolated?

## 17. Quick Revision Notes

- One-line summary: Backpressure aligns producer speed with consumer capacity.
- Three keywords: demand, bounded, flow.
- Interview trap: using unbounded queues and calling it resilience.
- Memory trick: receiver controls the faucet.

## 18. Mini Exercise

Design backpressure for a metrics ingestion pipeline.

Answer these:

1. What is the producer?
2. What is the consumer?
3. What signal slows the producer?
4. What happens when the buffer is full?
5. Which data can be sampled?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/backpressure/README.md](../../github-repo/backpressure/README.md)
- [github-repo/backpressure/src/main/java/com/iluwatar/backpressure/Publisher.java](../../github-repo/backpressure/src/main/java/com/iluwatar/backpressure/Publisher.java)
- [github-repo/backpressure/src/main/java/com/iluwatar/backpressure/Subscriber.java](../../github-repo/backpressure/src/main/java/com/iluwatar/backpressure/Subscriber.java)
- [github-repo/backpressure/src/main/java/com/iluwatar/backpressure/App.java](../../github-repo/backpressure/src/main/java/com/iluwatar/backpressure/App.java)
