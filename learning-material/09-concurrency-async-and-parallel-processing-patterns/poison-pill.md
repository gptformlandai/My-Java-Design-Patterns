# Poison Pill Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [poison-pill](../../github-repo/poison-pill)

---

## How to Study This Pattern

Study Poison Pill as graceful shutdown for queue consumers.

Remember:
- Normal messages mean "do work."
- One special message means "stop."
- Consumers exit after seeing the stop signal.

---

## 1. Technical Definition

Poison Pill is a messaging pattern where a special sentinel message is inserted into a queue to tell consumers that no more normal work should be processed and they should terminate gracefully.

### 30-Second Interview Answer

Poison Pill is used to shut down producer-consumer systems cleanly. Instead of killing consumers externally, the producer sends a special message through the same queue. When a consumer reads it, it finishes and exits. It is simple and effective, but you must send enough pills for the number of consumers and ensure the pill is ordered after real work.

---

## 2. Layman Explanation

Imagine a service counter receiving tickets. Normal tickets are work. At the end of the day, a final ticket says:

```text
CLOSE AFTER THIS
```

The worker sees it and stops accepting more tickets.

---

## 3. Bit by Bit Explanation

### Problem

Consumers often block on queues waiting for work.

If the system needs to shut down, you need a clean way to tell them:
- stop waiting
- do not accept new work
- exit the loop
- release resources

Interrupting threads directly can be fragile. A poison pill uses the existing message channel.

### Pattern Flow

1. Producer puts normal messages into a queue.
2. Consumer takes and processes messages.
3. Shutdown begins.
4. Producer stops accepting new work.
5. Producer inserts a special poison message.
6. Consumer reads the poison message.
7. Consumer breaks its processing loop and exits.

### Important Rule

If there are `N` consumers, you usually need `N` poison messages, unless the poison message is reinserted or broadcast by the queue system.

---

## 4. Java Coding Example

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

class Worker implements Runnable {
    static final String POISON = "__STOP__";

    private final BlockingQueue<String> queue;

    Worker(BlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            while (true) {
                String message = queue.take();
                if (POISON.equals(message)) {
                    System.out.println("worker stopping");
                    break;
                }
                System.out.println("processing " + message);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

public class PoisonPillDemo {
    public static void main(String[] args) throws Exception {
        BlockingQueue<String> queue = new LinkedBlockingQueue<>();
        Thread worker = new Thread(new Worker(queue));
        worker.start();

        queue.put("task-1");
        queue.put("task-2");
        queue.put(Worker.POISON);

        worker.join();
    }
}
```

### Java Block by Block

The consumer loop reads from a blocking queue.

Normal messages are processed.

The special `POISON` message causes the loop to break.

Shutdown is communicated through the same channel as work.

---

## 5. Python Coding Example

```python
from queue import Queue
from threading import Thread

POISON = object()


def worker(queue):
    while True:
        message = queue.get()
        try:
            if message is POISON:
                print("worker stopping")
                break
            print("processing", message)
        finally:
            queue.task_done()


queue = Queue()
thread = Thread(target=worker, args=(queue,))
thread.start()

queue.put("task-1")
queue.put("task-2")
queue.put(POISON)

queue.join()
thread.join()
```

---

## 6. When to Use It

Use Poison Pill when:
- consumers block on a queue
- shutdown should be graceful
- message ordering is clear
- producers can control the end of the stream
- consumers should finish current work before exiting

Interview triggers:
- "stop queue consumers"
- "graceful shutdown"
- "sentinel message"
- "worker should exit"
- "producer-consumer termination"

---

## 7. When Not to Use It

Avoid it when:
- the queue is priority-based and the pill might jump ahead of work
- consumers are distributed and message delivery is uncertain
- shutdown must be immediate
- work can be retried and the poison message might be redelivered incorrectly
- you cannot guarantee one stop signal per consumer

Use cancellation tokens, coordinated shutdown APIs, consumer group control, or orchestration-level shutdown for more complex systems.

---

## 8. Real-World Use Cases

| Use case | Why poison pill fits |
|---|---|
| Local worker queue | Simple consumer termination |
| Batch pipeline | End-of-input marker |
| Test harness | Stop background workers predictably |
| In-memory producer-consumer | Uses same queue path |
| Stream parser | Sentinel means no more records |
| Thread-pool demo | Special task can terminate loop workers |

---

## 9. Advantages Over Normal Code

Without Poison Pill:
- consumers may block forever
- shutdown requires external interruption
- producer and consumer termination logic is scattered

With Poison Pill:
- shutdown is explicit
- consumers exit through normal flow
- existing queue semantics are reused
- pending messages can finish before termination

---

## 10. Where It Excels

It excels in simple bounded worker systems where:
- the queue is FIFO
- consumer count is known
- shutdown is graceful
- processing should finish before exit

---

## 11. Where It Fails

It fails when:
- the poison pill is not the final message
- there are multiple consumers but only one pill
- a consumer accidentally processes the pill like normal data
- a producer sends more work after stopping
- priority queues reorder the shutdown signal

In distributed queues, be careful with redelivery and dead-letter behavior.

---

## 12. Frameworks and Libraries

| Ecosystem | Tools |
|---|---|
| Java | `BlockingQueue`, executor worker loops |
| Python | `queue.Queue`, multiprocessing queues |
| Messaging | Kafka, RabbitMQ, SQS with explicit end markers in limited workflows |
| Actor systems | Actor stop messages |
| Batch systems | End-of-stream markers |

---

## 13. Pros and Cons

| Pros | Cons |
|---|---|
| Simple graceful shutdown | Needs one signal per consumer |
| Uses existing queue | Can be mishandled as normal data |
| Lets work drain first | Ordering matters |
| Easy to test | Not ideal for complex distributed shutdown |
| Decouples producer and consumer | Immediate shutdown needs another mechanism |

---

## 14. Real-World Identification Scenario

Question:

> A producer feeds tasks to three worker threads through a blocking queue. At shutdown, all workers should finish queued tasks and then stop. What should you do?

Strong answer:

Use Poison Pill. Stop accepting new tasks, enqueue all remaining work, then enqueue three poison messages, one per worker. Each worker processes normal tasks until it reads a poison message, then exits cleanly. I would also ensure the queue is FIFO and the poison messages are inserted after real work.

---

## 15. Interview Triggers

Say Poison Pill when you hear:
- sentinel message
- stop worker
- end of stream
- graceful queue shutdown
- terminate consumer loop
- final message
- producer-consumer shutdown

---

## 16. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Sending one pill for many consumers | Only one consumer exits | Send one per consumer |
| Sending pill before work drains | Work may be skipped | Insert after final normal message |
| Using a normal valid value as sentinel | Ambiguous | Use a unique marker object |
| Allowing sends after stop | Work may never be processed | Reject new messages after shutdown |
| Ignoring priority queues | Pill may arrive too early | Use explicit shutdown coordination |

---

## 17. Pattern Comparison and Design Checklist

| Pattern | Difference |
|---|---|
| Producer-Consumer | Poison Pill is a termination technique for producer-consumer queues |
| Cancellation Token | Out-of-band shutdown signal instead of a queue message |
| Interrupt | Thread-level signal; poison pill is application-level |
| Command | Poison pill can be modeled as a special command |
| Actor Model | Actors often stop through a special stop message |

Design checklist:
- How many consumers are running?
- How many poison messages are needed?
- Can the queue reorder messages?
- Are producers prevented from sending after stop?
- Should consumers drain all work before stopping?
- How will shutdown be tested?

---

## 18. Quick Revision Notes and Mini Exercise

- One-line summary: Send a special queue message to stop consumers.
- Memory hook: "final ticket says stop."
- Best for: graceful shutdown of blocking consumer loops.
- Avoid when: message ordering or consumer count is uncertain.
- Interview line: "For N consumers, I would enqueue N poison pills after all real work."

Mini exercise:

Implement a queue with three worker threads. Enqueue ten normal tasks, then enqueue three stop signals. Verify each worker exits and all ten tasks are processed.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/poison-pill/README.md)
- [Message.java](../../github-repo/poison-pill/src/main/java/com/iluwatar/poison/pill/Message.java)
- [Producer.java](../../github-repo/poison-pill/src/main/java/com/iluwatar/poison/pill/Producer.java)
- [Consumer.java](../../github-repo/poison-pill/src/main/java/com/iluwatar/poison/pill/Consumer.java)
- [SimpleMessageQueue.java](../../github-repo/poison-pill/src/main/java/com/iluwatar/poison/pill/SimpleMessageQueue.java)
- [App.java](../../github-repo/poison-pill/src/main/java/com/iluwatar/poison/pill/App.java)

The repo implementation defines `Message.POISON_PILL`. `Producer.stop()` puts it into the queue, and `Consumer.consume()` exits when it reads that marker.
