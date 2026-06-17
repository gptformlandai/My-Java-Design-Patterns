# Event-Based Asynchronous Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [event-based-asynchronous](../../github-repo/event-based-asynchronous)

---

## How to Study This Pattern

Read this when you see long-running work that should not block the caller.

Focus on three ideas:
- The caller starts an operation and continues doing other work.
- The operation runs in the background.
- Completion is reported later through an event, callback, listener, or status object.

---

## 1. Technical Definition

Event-Based Asynchronous is a concurrency pattern where an operation is started without blocking the caller, and the result, completion, cancellation, or failure is communicated later through an event-style notification.

### 30-Second Interview Answer

Event-Based Asynchronous lets a system start work and return control immediately. The work runs independently, usually on another thread or event loop, and notifies the caller when it finishes. I would use it for UI responsiveness, background I/O, uploads, report generation, and systems where waiting synchronously would waste threads or hurt latency.

---

## 2. Layman Explanation

Imagine submitting a service request and getting a tracking number. You do not stand at the counter until the work is complete. You continue your day, and the system notifies you when the request changes state.

That is event-based async:
- Start the work.
- Keep moving.
- React when an event says the work is complete.

---

## 3. Bit by Bit Explanation

### Problem

Some tasks take time:
- network calls
- file downloads
- database exports
- email sending
- image processing
- external API calls

If the caller waits directly, the thread is blocked.

### Pattern Flow

1. A client asks an event manager or service to create work.
2. The service creates an event with an ID and duration/status.
3. The event starts in the background.
4. The client can query status, cancel, or continue other work.
5. When the event completes, a listener/callback updates state.
6. Failed or cancelled work is reported through status/error handling.

### Core Participants

| Participant | Responsibility |
|---|---|
| Client | Starts work and reacts later |
| Event object | Represents the long-running operation |
| Event manager | Tracks running events and lifecycle operations |
| Background worker | Executes the work |
| Completion listener | Receives done/failure notification |

---

## 4. Java Coding Example

```java
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.function.Consumer;

class AsyncEvents {
    private final ExecutorService executor = Executors.newFixedThreadPool(2);
    private final Map<String, String> statuses = new ConcurrentHashMap<>();

    public String submit(String jobName, Consumer<String> onComplete) {
        String id = UUID.randomUUID().toString();
        statuses.put(id, "RUNNING");

        executor.submit(() -> {
            try {
                Thread.sleep(500);
                statuses.put(id, "COMPLETED");
                onComplete.accept(id + " completed: " + jobName);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                statuses.put(id, "FAILED");
            }
        });

        return id;
    }

    public String status(String id) {
        return statuses.getOrDefault(id, "UNKNOWN");
    }

    public void shutdown() {
        executor.shutdownNow();
    }
}

public class EventBasedAsyncDemo {
    public static void main(String[] args) throws Exception {
        AsyncEvents events = new AsyncEvents();

        String jobId = events.submit("monthly-report", System.out::println);
        System.out.println("Submitted job: " + jobId);
        System.out.println("Immediate status: " + events.status(jobId));

        Thread.sleep(800);
        System.out.println("Later status: " + events.status(jobId));

        events.shutdown();
    }
}
```

### Java Block by Block

`submit` creates a job ID and marks the job as `RUNNING`.

The executor runs the task in the background, so the caller is not blocked.

The callback `onComplete` behaves like a completion event.

`status` lets the caller check the operation later.

### Java Output

```text
Submitted job: <id>
Immediate status: RUNNING
<id> completed: monthly-report
Later status: COMPLETED
```

---

## 5. Python Coding Example

```python
from concurrent.futures import ThreadPoolExecutor
from uuid import uuid4
import time


class AsyncEvents:
    def __init__(self):
        self.executor = ThreadPoolExecutor(max_workers=2)
        self.statuses = {}

    def submit(self, job_name, on_complete):
        job_id = str(uuid4())
        self.statuses[job_id] = "RUNNING"

        def run():
            time.sleep(0.5)
            self.statuses[job_id] = "COMPLETED"
            on_complete(f"{job_id} completed: {job_name}")

        self.executor.submit(run)
        return job_id

    def status(self, job_id):
        return self.statuses.get(job_id, "UNKNOWN")


events = AsyncEvents()
job_id = events.submit("monthly-report", print)
print("submitted", job_id)
print("immediate", events.status(job_id))
time.sleep(0.8)
print("later", events.status(job_id))
events.executor.shutdown()
```

---

## 6. When to Use It

Use Event-Based Asynchronous when:
- callers must stay responsive
- work is long-running or I/O-heavy
- completion can be handled later
- multiple independent operations can run concurrently
- polling alone would be wasteful
- you need lifecycle operations like start, status, cancel, complete, fail

Interview triggers:
- "do not block the request thread"
- "notify user when complete"
- "run in background"
- "upload processing"
- "download manager"
- "UI must remain responsive"

---

## 7. When Not to Use It

Do not use it when:
- the operation is tiny and synchronous code is clearer
- the caller truly needs the result immediately
- ordering must be strict and simple
- async lifecycle state would be more complexity than value
- failure handling cannot be made reliable

Use direct calls for simple local operations, `CompletableFuture` for composition-heavy Java async, and queues for durable cross-service async work.

---

## 8. Real-World Use Cases

| Use case | Why it fits |
|---|---|
| File upload processing | User gets response while scanning/transcoding continues |
| Email sending | Request path does not wait for SMTP |
| Report generation | Long work can notify when ready |
| GUI action handlers | UI thread remains free |
| External API calls | Callers avoid blocking during slow I/O |
| Background indexing | Search index updates happen asynchronously |

---

## 9. Advantages Over Normal Code

Without this pattern:
- callers block during slow work
- UI/request threads can freeze
- throughput drops under I/O waits
- cancellation/status is often bolted on later

With this pattern:
- callers stay responsive
- work lifecycle is explicit
- completion/failure notification is structured
- the system can run many independent jobs

---

## 10. Where It Excels

It excels when latency to acknowledge work matters more than latency to finish the work.

Examples:
- "Your video is being processed"
- "Your export will be emailed"
- "Your payment verification is pending"
- "Your import has started"

---

## 11. Where It Fails

It can fail when:
- callbacks are not idempotent
- failures are swallowed
- cancellation is partial
- job state is only stored in memory
- too many background tasks are created without limits
- the caller assumes completion happened immediately

For critical jobs, persist job state and use a durable queue.

---

## 12. Frameworks and Libraries

| Ecosystem | Tools |
|---|---|
| Java | `CompletableFuture`, `ExecutorService`, Spring `@Async`, Project Reactor |
| Java UI | SwingWorker, JavaFX Task |
| Messaging | Kafka, RabbitMQ, JMS |
| Python | `asyncio`, `concurrent.futures`, Celery |
| JavaScript | Promises, async/await, event emitters |

---

## 13. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps callers responsive | Harder control flow |
| Improves thread utilization | More complex error handling |
| Supports status/cancel lifecycle | Race conditions around job state |
| Good for I/O-heavy work | Debugging can be harder |
| Enables completion notifications | In-memory jobs can be lost on crash |

---

## 14. Real-World Identification Scenario

Question:

> A user uploads a large CSV. The API should respond quickly, validate the file in the background, and notify the user when import is complete. What pattern can help?

Strong answer:

Use Event-Based Asynchronous. The upload endpoint stores the file, creates an import job, returns a job ID, and starts validation/import in the background. The UI can poll status or receive a notification. Failures are stored with the job, and retry/cancel can be implemented through explicit job lifecycle states.

---

## 15. Interview Triggers

Say Event-Based Asynchronous when you hear:
- background task
- notification on completion
- event listener
- non-blocking workflow
- long-running operation
- status endpoint
- async processing
- fire and callback

---

## 16. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Starting unlimited threads | Can exhaust memory/CPU | Use bounded executors |
| Ignoring failures | Users never know the job failed | Store failure states |
| No cancellation path | Wasteful long-running work | Add cancel and timeout |
| Assuming async means durable | In-memory work dies on restart | Use a queue for critical jobs |
| Callback does too much | Completion path becomes fragile | Keep callback small and delegate |

---

## 17. Pattern Comparison and Design Checklist

| Pattern | Difference |
|---|---|
| Callback | A callback is a notification mechanism; event-based async is the broader lifecycle style |
| Promise | Promise represents a future value and composes results |
| Observer | Observer notifies subscribers about events; async events may use observers internally |
| Producer-Consumer | Queue-based handoff; often more durable and backpressure-friendly |
| Reactor | Event loop dispatches readiness events rather than one operation per worker |

Design checklist:
- What state can the job be in?
- Where is job state stored?
- How does the caller learn completion?
- How are failures and cancellations represented?
- What limits prevent too many background jobs?
- Does the task need durability across restarts?

---

## 18. Quick Revision Notes and Mini Exercise

- One-line summary: Start work now, notify later.
- Memory hook: "tracking number for background work."
- Best for: responsiveness and long-running I/O.
- Avoid when: the result is needed immediately and the work is simple.
- Interview line: "I would return a job ID, track state, run the task asynchronously, and notify or expose status on completion."

Mini exercise:

Design an async export API that returns a job ID immediately. Add states for `PENDING`, `RUNNING`, `COMPLETED`, `FAILED`, and `CANCELLED`, then decide how the UI checks status and downloads the result.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/event-based-asynchronous/README.md)
- [EventManager.java](../../github-repo/event-based-asynchronous/src/main/java/com/iluwatar/event/asynchronous/EventManager.java)
- [AsyncEvent.java](../../github-repo/event-based-asynchronous/src/main/java/com/iluwatar/event/asynchronous/AsyncEvent.java)
- [Event.java](../../github-repo/event-based-asynchronous/src/main/java/com/iluwatar/event/asynchronous/Event.java)
- [ThreadCompleteListener.java](../../github-repo/event-based-asynchronous/src/main/java/com/iluwatar/event/asynchronous/ThreadCompleteListener.java)
- [App.java](../../github-repo/event-based-asynchronous/src/main/java/com/iluwatar/event/asynchronous/App.java)

The repo implementation centers on an `EventManager` that creates events, tracks them by ID, starts/cancels them, and receives completion callbacks from `AsyncEvent`.
