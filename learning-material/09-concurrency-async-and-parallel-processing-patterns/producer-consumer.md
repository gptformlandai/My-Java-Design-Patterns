# Producer-Consumer Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/producer-consumer](../../github-repo/producer-consumer)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the queue as the buffer between producers and consumers.
2. Second pass: trace what happens when the queue is full, empty, and healthy.
3. Third pass: compare Producer-Consumer with Queue-Based Load Leveling, Backpressure, and Thread Pool.

By the end, you should be able to say:

> Producer-Consumer decouples work creation from work processing using a shared buffer.

## 1. Technical Definition

Producer-Consumer is a concurrency pattern where producer threads create items and consumer threads process them through a synchronized queue or buffer.

Core idea:

- Producers do not call consumers directly.
- A queue buffers items.
- Consumers take items when ready.
- Full queues slow producers.
- Empty queues make consumers wait.

### 30-Second Interview Answer

I would use Producer-Consumer when one part of the system generates work and another part processes it at a different rate. A bounded blocking queue decouples producers from consumers and gives natural flow control. The key trade-offs are queue capacity, ordering, backpressure, shutdown, and making consumers idempotent if items can be retried.

## 2. Layman and Easy to Understand Definition

Producer-Consumer is like a work table between two teams.

One team puts tasks on the table. Another team takes tasks from the table. The teams do not need to work at the same speed.

In code:

- Producer puts item into queue.
- Queue stores item safely.
- Consumer takes item.
- Consumer processes item.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

If producers call consumers directly:

```text
producer -> consumer
```

the producer is blocked by consumer speed. If many producers call at once, the consumer can be overwhelmed.

### 3.2 The Pattern Solution

Add a queue:

```text
producer -> queue -> consumer
```

The queue absorbs short speed differences.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Producer | Creates work items. |
| Consumer | Processes work items. |
| Buffer | Shared queue between them. |
| Blocking put | Producer waits when buffer is full. |
| Blocking take | Consumer waits when buffer is empty. |
| Shutdown signal | Tells consumers when to stop. |

### 3.4 Runtime Flow

1. Producer creates item.
2. Producer puts item into queue.
3. If queue is full, producer waits or fails.
4. Consumer takes item from queue.
5. If queue is empty, consumer waits.
6. Consumer processes item.

## 4. Java Coding Example

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

record Job(String id, String payload) {
}

class Producer implements Runnable {
    private final BlockingQueue<Job> queue;

    Producer(BlockingQueue<Job> queue) {
        this.queue = queue;
    }

    public void run() {
        try {
            queue.put(new Job("job-1", "resize-image"));
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

class Consumer implements Runnable {
    private final BlockingQueue<Job> queue;

    Consumer(BlockingQueue<Job> queue) {
        this.queue = queue;
    }

    public void run() {
        try {
            Job job = queue.take();
            System.out.println("processing " + job.id());
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `BlockingQueue` | Thread-safe buffer. |
| `ArrayBlockingQueue` | Bounded capacity. |
| `put` | Waits when full. |
| `take` | Waits when empty. |
| interrupt handling | Required for clean shutdown. |

### Java Usage

```java
BlockingQueue<Job> queue = new ArrayBlockingQueue<>(10);
new Thread(new Producer(queue)).start();
new Thread(new Consumer(queue)).start();
```

## 5. Python Coding Example

```python
from queue import Queue
from threading import Thread


def producer(queue):
    queue.put({"id": "job-1", "payload": "resize-image"})


def consumer(queue):
    job = queue.get()
    try:
        print(f"processing {job['id']}")
    finally:
        queue.task_done()


queue = Queue(maxsize=10)
Thread(target=producer, args=(queue,)).start()
Thread(target=consumer, args=(queue,)).start()
```

### Python Usage

Use this shape when explaining:

- Queue size is the buffer.
- Producer and consumer run independently.
- Bounded queues create backpressure.

## 6. Where It Comes Handy in Real Life

- Background job processing.
- Image or video processing.
- Logging pipelines.
- Request ingestion.
- Message consumers.
- Stream processing.
- File import pipelines.

## 7. Advantages Over Normal Code Without Pattern

Without Producer-Consumer:

```text
producer directly waits for consumer
```

With Producer-Consumer:

```text
queue decouples production and processing
```

Benefits:

- Better concurrency.
- Natural buffering.
- Easier scaling of consumers.
- Less direct coupling.
- Smoother handling of bursts.

## 8. Where It Excels

- Work units are independent.
- Producers and consumers have different speeds.
- Short bursts need buffering.
- Processing can be asynchronous.
- Consumers can be scaled horizontally.

## 9. Where It Fails

- Work must be completed synchronously.
- Queue is unbounded.
- Consumers are not idempotent.
- Ordering is strict but multiple consumers reorder work.
- Shutdown protocol is missing.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | `BlockingQueue`, `ExecutorService`, Disruptor |
| Messaging | Kafka, RabbitMQ, ActiveMQ, SQS |
| Python | `queue.Queue`, `asyncio.Queue`, Celery |
| Reactive | Project Reactor, RxJava |
| Cloud | Pub/Sub, Service Bus, managed queues |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples producers and consumers. | Adds queue management. |
| Handles speed mismatch. | Adds processing latency. |
| Supports parallel consumers. | Can duplicate or reorder work. |
| Enables bounded buffering. | Needs shutdown and error handling. |

## 12. Real-World Identification Example

Scenario: Users upload photos and thumbnails must be generated.

Producer-Consumer fit:

- Upload service produces thumbnail jobs.
- Queue buffers jobs.
- Workers consume and generate thumbnails.
- More workers can be added during traffic spikes.

Without it:

- Upload requests block on thumbnail generation.
- Traffic spikes overload the image processor.

## 13. MAANG Interview Triggers

Use Producer-Consumer when you hear:

- "Producer is faster than consumer."
- "Need async processing."
- "Use a queue."
- "Handle work in background."
- "How do we decouple ingestion and processing?"
- "Bounded buffer."

Strong answer keywords:

- blocking queue
- bounded buffer
- backpressure
- consumer workers
- poison pill
- idempotency
- queue depth
- shutdown

## 14. Common Mistakes

### Mistake 1: Unbounded queue

- Why it is wrong: memory becomes the hidden limit.
- Better approach: use bounded queues and define overflow behavior.

### Mistake 2: Ignoring shutdown

- Why it is wrong: consumers can block forever.
- Better approach: use interrupts, cancellation, or poison pill messages.

### Mistake 3: Assuming exactly-once processing

- Why it is wrong: failures and retries can duplicate work.
- Better approach: make consumers idempotent.

### Mistake 4: No monitoring

- Why it is wrong: queue buildup can hide overload.
- Better approach: monitor queue depth, oldest item age, and consumer errors.

## 15. Producer-Consumer vs Similar Patterns

| Pattern | Difference |
|---|---|
| Producer-Consumer | In-process or logical queue between producers and consumers. |
| Queue-Based Load Leveling | Distributed reliability pattern using queues to smooth load. |
| Thread Pool | Reuses worker threads to execute submitted tasks. |
| Backpressure | Consumer signals producer to slow down. |
| Poison Pill | Shutdown message often used with producer-consumer queues. |

## 16. Producer-Consumer Design Checklist

- What is the item schema?
- Is the queue bounded?
- How many producers and consumers exist?
- What happens when queue is full?
- What happens when processing fails?
- Is item processing idempotent?
- Is ordering required?
- How does shutdown work?
- What queue metrics are monitored?

## 17. Quick Revision Notes

- One-line summary: Producer-Consumer uses a queue to decouple work creation and processing.
- Three keywords: queue, producer, consumer.
- Interview trap: forgetting bounded capacity and shutdown.
- Memory trick: work table between maker and processor.

## 18. Mini Exercise

Design a producer-consumer pipeline for sending emails.

Answer these:

1. What does the producer enqueue?
2. How many consumers run?
3. What is the queue capacity?
4. What happens on email provider failure?
5. How do consumers shut down?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/producer-consumer/README.md](../../github-repo/producer-consumer/README.md)
- [github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/ItemQueue.java](../../github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/ItemQueue.java)
- [github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/Producer.java](../../github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/Producer.java)
- [github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/Consumer.java](../../github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/Consumer.java)
- [github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/Item.java](../../github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/Item.java)
- [github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/App.java](../../github-repo/producer-consumer/src/main/java/com/iluwatar/producer/consumer/App.java)
