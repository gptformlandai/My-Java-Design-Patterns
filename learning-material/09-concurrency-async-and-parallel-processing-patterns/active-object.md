# Active Object Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/active-object](../../github-repo/active-object)

## How to Study This Page

Use this page in three passes:

1. First pass: understand that method invocation and method execution happen on different threads.
2. Second pass: trace proxy method, request queue, scheduler, worker thread, and result callback.
3. Third pass: compare Active Object with Actor Model, Async Method Invocation, and Thread Pool.

By the end, you should be able to say:

> Active Object gives an object its own execution thread and request queue so callers do not block on method execution.

## 1. Technical Definition

Active Object is a concurrency pattern that decouples method calls from method execution by turning calls into queued requests executed by the object's own thread or scheduler.

Core idea:

- Client calls a public method.
- Method creates a request object.
- Request goes into queue.
- Active object's scheduler executes requests later.
- Caller may receive a future, callback, or no result.

### 30-Second Interview Answer

I would use Active Object when I want an object to expose a simple API but execute its work asynchronously and serially on its own thread. This protects internal state and keeps callers responsive. The trade-off is more lifecycle and queue management, plus async error handling.

## 2. Layman and Easy to Understand Definition

Active Object is like leaving tasks in an object's inbox.

You do not wait at the desk while the object works. You drop the request, and the object handles requests one at a time in its own execution context.

In code:

- Public method accepts request.
- Request is enqueued.
- Internal worker takes request.
- Internal worker executes request.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Direct method calls block the caller and can expose shared state:

```text
caller thread -> object method -> long work
```

### 3.2 The Active Object Solution

Put a queue and execution thread inside the object:

```text
caller -> active object API -> request queue -> active thread
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Proxy/API | Public methods called by clients. |
| Method request | Encapsulated operation. |
| Activation queue | Pending requests. |
| Scheduler | Decides next request. |
| Servant | Actual implementation. |
| Future/callback | Optional result delivery. |

### 3.4 Runtime Flow

1. Client invokes method.
2. Method creates a runnable/request.
3. Request is placed into activation queue.
4. Active object thread takes request.
5. Request is executed.
6. Result or completion is delivered asynchronously.

## 4. Java Coding Example

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

class ActiveLogger {
    private final BlockingQueue<Runnable> requests = new LinkedBlockingQueue<>();
    private final Thread worker;

    ActiveLogger() {
        worker = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    requests.take().run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        worker.start();
    }

    void log(String message) throws InterruptedException {
        requests.put(() -> System.out.println(message));
    }

    void stop() {
        worker.interrupt();
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `requests` | Activation queue. |
| `worker` | Active object's execution thread. |
| `log` | Public API creates async request. |
| `requests.take` | Scheduler loop. |
| `stop` | Lifecycle must be explicit. |

### Java Usage

```java
ActiveLogger logger = new ActiveLogger();
logger.log("hello async object");
logger.stop();
```

## 5. Python Coding Example

```python
from queue import Queue
from threading import Thread


class ActiveLogger:
    def __init__(self):
        self.requests = Queue()
        self.worker = Thread(target=self._run, daemon=True)
        self.worker.start()

    def _run(self):
        while True:
            task = self.requests.get()
            if task is None:
                break
            task()

    def log(self, message):
        self.requests.put(lambda: print(message))

    def stop(self):
        self.requests.put(None)
```

### Python Usage

Use this shape when explaining:

- Public methods enqueue work.
- One internal thread owns execution.
- Stop signal is part of lifecycle.

## 6. Where It Comes Handy in Real Life

- UI thread dispatching.
- Device control objects.
- Async file writers.
- Serialized access to mutable state.
- Background command processors.
- Object-level task queues.

## 7. Advantages Over Normal Code Without Pattern

Without Active Object:

```text
caller blocks while object performs work
```

With Active Object:

```text
caller queues request and continues
```

Benefits:

- Improves responsiveness.
- Encapsulates concurrency inside object.
- Serializes internal state changes.
- Reduces direct lock use by callers.
- Supports async result delivery.

## 8. Where It Excels

- Object has mutable state.
- Calls should not block callers.
- Operations can be serialized.
- Request ordering matters.
- A dedicated execution context is acceptable.

## 9. Where It Fails

- Work must return immediately with a result.
- Queue grows without bound.
- Object thread blocks on slow I/O.
- Lifecycle is not managed.
- Many active objects create too many threads.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Executors, single-thread executors, CompletableFuture |
| Actor frameworks | Akka, Pekko, Orleans concepts |
| UI | Swing event dispatch thread, JavaFX application thread |
| Python | `asyncio` actors, worker queues |
| Messaging | Mailbox-style dispatchers |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps caller responsive. | Adds queue and lifecycle complexity. |
| Encapsulates synchronization. | Async errors are harder. |
| Serializes state changes. | Queue can become bottleneck. |
| Preserves request order. | Too many active objects waste threads. |

## 12. Real-World Identification Example

Scenario: A metrics writer must accept measurements quickly but write them in order.

Active Object fit:

- Public `record` method enqueues write.
- Dedicated worker writes metrics.
- Caller continues immediately.
- Internal state is touched by one worker.

Without it:

- Callers block on I/O.
- Multiple threads race on writer state.

## 13. MAANG Interview Triggers

Use Active Object when you hear:

- "Object has its own thread."
- "Method invocation should not block."
- "Serialize access to internal state."
- "Async command queue."
- "Return future from object method."

Strong answer keywords:

- activation queue
- method request
- scheduler
- servant
- future
- callback
- serialized state
- lifecycle

## 14. Common Mistakes

### Mistake 1: Unbounded activation queue

- Why it is wrong: callers can overload the object.
- Better approach: bound the queue or reject requests under pressure.

### Mistake 2: Blocking inside active thread

- Why it is wrong: all later requests are delayed.
- Better approach: offload slow I/O or use separate workers.

### Mistake 3: No shutdown signal

- Why it is wrong: worker thread can leak.
- Better approach: implement stop, poison pill, or cancellation.

### Mistake 4: Exposing internal state

- Why it is wrong: callers can bypass serialized execution.
- Better approach: only mutate through queued requests.

## 15. Active Object vs Similar Patterns

| Pattern | Difference |
|---|---|
| Active Object | Object owns queue and execution context. |
| Actor Model | Actors communicate only by messages and can form systems. |
| Async Method Invocation | General non-blocking method call with future/callback. |
| Thread Pool | Shared worker threads for many tasks. |
| Monitor | Uses locks to protect shared object access. |

## 16. Active Object Design Checklist

- What methods become async requests?
- Is the request queue bounded?
- Does ordering matter?
- How are results returned?
- How are failures reported?
- How does shutdown work?
- Can the active thread block?
- How many active objects exist?

## 17. Quick Revision Notes

- One-line summary: Active Object queues method calls and executes them in its own thread.
- Three keywords: queue, scheduler, async object.
- Interview trap: forgetting lifecycle and queue bounds.
- Memory trick: object with an inbox and worker.

## 18. Mini Exercise

Design an active object for an audit writer.

Answer these:

1. What public methods enqueue requests?
2. What is the queue capacity?
3. How are write failures reported?
4. How is ordering preserved?
5. How does shutdown flush pending requests?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/active-object/README.md](../../github-repo/active-object/README.md)
- [github-repo/active-object/src/main/java/com/iluwatar/activeobject/ActiveCreature.java](../../github-repo/active-object/src/main/java/com/iluwatar/activeobject/ActiveCreature.java)
- [github-repo/active-object/src/main/java/com/iluwatar/activeobject/App.java](../../github-repo/active-object/src/main/java/com/iluwatar/activeobject/App.java)
