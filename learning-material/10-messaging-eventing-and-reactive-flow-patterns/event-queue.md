# Event Queue Pattern

Category: Messaging, Eventing, and Reactive Flow Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [event-queue](../../github-repo/event-queue)

---

## How to Study This Page

Study Event Queue as the pattern that separates "request was made" from "request is processed."

Remember the simple shape:

```text
producer -> queue -> consumer
```

The queue is the buffer that lets both sides move at different speeds.

---

## 1. Technical Definition

Event Queue is a messaging pattern where events or requests are placed into a queue and processed asynchronously by one or more consumers later.

### 30-Second Interview Answer

An Event Queue buffers work between producers and consumers. Producers enqueue events quickly and continue, while consumers process events asynchronously. I would use it when work is slow, bursty, or should not block the caller, such as email sending, media processing, or order fulfillment. The key trade-offs are queue growth, latency, retry behavior, ordering, backpressure, and duplicate processing.

---

## 2. Layman and Easy to Understand Definition

Imagine a ticket counter. Customers submit tickets. Workers pick tickets from the line when they are ready.

The customer does not need to wait for the worker to finish before submitting the ticket.

That line is the event queue.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Some work is too slow or too bursty to handle directly in the caller's path:
- sending emails
- resizing images
- processing uploaded files
- charging delayed jobs
- playing sounds
- updating search indexes

Direct execution blocks the caller and couples producer speed to consumer speed.

### Event Queue Flow

1. Producer creates an event or work item.
2. Producer enqueues it.
3. Producer returns quickly.
4. Consumer polls or blocks on the queue.
5. Consumer processes one event at a time or in batches.
6. Failed events are retried, moved to a dead-letter queue, or dropped based on policy.
7. Queue metrics show backlog, lag, and failures.

### Core Participants

| Participant | Responsibility |
|---|---|
| Producer | Creates events/work items |
| Queue | Stores events until processing |
| Consumer | Processes queued events |
| Retry policy | Handles transient failures |
| Dead-letter queue | Stores poison or permanently failed events |
| Backpressure policy | Controls overload |

---

## 4. Java Coding Example

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

record Event(String type, String payload) {}

class EventWorker implements Runnable {
    private final BlockingQueue<Event> queue;

    EventWorker(BlockingQueue<Event> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            while (!Thread.currentThread().isInterrupted()) {
                Event event = queue.take();
                System.out.println("processing " + event.type() + " " + event.payload());
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

public class EventQueueDemo {
    public static void main(String[] args) throws Exception {
        BlockingQueue<Event> queue = new LinkedBlockingQueue<>();
        Thread worker = new Thread(new EventWorker(queue));
        worker.start();

        queue.put(new Event("EmailRequested", "user-123"));
        queue.put(new Event("ReportRequested", "report-456"));

        Thread.sleep(300);
        worker.interrupt();
    }
}
```

### Java Block by Block

`BlockingQueue` is the buffer.

The producer calls `put` and does not process the event directly.

The worker calls `take`, which waits efficiently until work arrives.

The producer and consumer are now decoupled in time.

---

## 5. Python Coding Example

```python
from queue import Queue
from threading import Thread
import time


def worker(queue):
    while True:
        event = queue.get()
        try:
            if event is None:
                break
            print("processing", event)
        finally:
            queue.task_done()


queue = Queue()
thread = Thread(target=worker, args=(queue,))
thread.start()

queue.put(("EmailRequested", "user-123"))
queue.put(("ReportRequested", "report-456"))
queue.put(None)

queue.join()
thread.join()
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Email sending | Slow external service should not block request |
| Media processing | CPU-heavy work can run later |
| Order fulfillment | Downstream systems process asynchronously |
| Log ingestion | Producers send quickly, consumers batch |
| Notification delivery | Retries can be handled outside request path |
| Audio/playback requests | Events are buffered and processed by a service thread |

---

## 7. Advantages Over Normal Code Without Pattern

Without Event Queue:
- caller waits for slow work
- bursts overload consumers immediately
- retry logic is tangled with producer logic
- one dependency failure can fail the whole request

With Event Queue:
- producers return quickly
- consumers process at controlled speed
- bursts are buffered
- retry and dead-letter behavior can be centralized

---

## 8. Where It Excels

It excels when:
- work can happen later
- tasks are independent
- producer traffic is bursty
- consumer capacity is limited
- queue backlog is acceptable and observable

---

## 9. Where It Fails

It fails when:
- the caller needs an immediate result
- queue latency is not acceptable
- the queue can grow without bounds
- consumers are not idempotent
- retry storms overload downstream systems
- ordering is required but not designed

Always define queue size limits, retry policy, and overload behavior.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | `BlockingQueue`, `ExecutorService`, Spring Integration |
| Brokers | RabbitMQ, ActiveMQ, Kafka, Pulsar |
| Cloud | AWS SQS, Azure Queue Storage, Google Cloud Tasks |
| Python | `queue.Queue`, Celery, RQ |
| Streaming | Kafka consumer groups, Flink sources |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples producer and consumer speed | Adds queue latency |
| Buffers bursts | Backlog can grow |
| Enables asynchronous processing | Requires retry design |
| Smooths load | Duplicates can happen |
| Improves resilience | Ordering can be difficult |

---

## 12. Real-World Identification Example

Question:

> Users upload images. The API should respond quickly, but images must be scanned, resized, and indexed. What pattern helps?

Strong answer:

Use an Event Queue. The upload API stores the file and enqueues an `ImageUploaded` event. Workers consume the queue and perform scanning, resizing, and indexing. I would set queue limits, retry transient failures, move bad messages to a dead-letter queue, and make workers idempotent using the image ID.

---

## 13. MAANG Interview Triggers

Say Event Queue when you hear:
- asynchronous worker
- buffer bursty traffic
- background jobs
- producer and consumer run at different speeds
- queue backlog
- retry failed tasks
- work can happen later

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Unbounded queue | Memory or storage can explode | Set limits and backpressure |
| No idempotency | Retries duplicate side effects | Use dedupe keys |
| No dead-letter path | Bad events block progress | Send repeated failures to DLQ |
| Ignoring lag | Users see silent delays | Monitor queue depth and age |
| Assuming FIFO always holds | Distributed queues may reorder | Design ordering by key if needed |

---

## 15. Event Queue vs Similar Patterns

| Pattern | Difference |
|---|---|
| Publish-Subscribe | Pub/sub fans out; queue often distributes work to consumers |
| Producer-Consumer | Event Queue is a common producer-consumer implementation |
| Event-Driven Architecture | Event Queue is one mechanism inside EDA |
| Reactor | Reactor handles readiness events; queues store work |
| Thread Pool Executor | Thread pool executes tasks; queue often feeds the pool |

---

## 16. Event Queue Design Checklist

- What is the queue capacity?
- What is the max acceptable event age?
- How many consumers are needed?
- What is the retry policy?
- What goes to dead-letter?
- Are consumers idempotent?
- Is ordering required?
- What metrics define queue health?

---

## 17. Quick Revision Notes

- One-line summary: Put work in a queue and process it later.
- Memory hook: "ticket line between producer and worker."
- Best for: background work and burst buffering.
- Avoid when: immediate response from processing is required.
- Interview line: "I would enqueue the work, process with idempotent workers, monitor lag, and define retry plus dead-letter behavior."

---

## 18. Mini Exercise

Design an event queue for sending password reset emails:
- define the event payload
- choose retry count and delay
- decide when to dead-letter
- define one idempotency key
- add two metrics for operations

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/event-queue/README.md)
- [Audio.java](../../github-repo/event-queue/src/main/java/com/iluwatar/event/queue/Audio.java)
- [PlayMessage.java](../../github-repo/event-queue/src/main/java/com/iluwatar/event/queue/PlayMessage.java)
- [App.java](../../github-repo/event-queue/src/main/java/com/iluwatar/event/queue/App.java)

The repo implementation uses `Audio` as a queue-backed service. `playSound()` enqueues `PlayMessage` objects, while an update thread drains the queue and processes audio requests later.

