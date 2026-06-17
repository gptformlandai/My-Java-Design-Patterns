# Promise Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [promise](../../github-repo/promise)

---

## How to Study This Pattern

Study Promise as a cleaner way to represent an asynchronous result.

The core idea:
- An operation starts now.
- The value will exist later.
- Callers attach success/failure actions without blocking immediately.

---

## 1. Technical Definition

Promise is a concurrency pattern that represents a stand-in for a future result of an asynchronous operation, allowing callbacks, transformations, and error handlers to be attached.

### 30-Second Interview Answer

A Promise represents a value that is not available yet but will be completed later. It lets the caller chain follow-up actions such as transform, consume, or handle error. In Java, `CompletableFuture` is the common production version. I would use promises to compose async I/O, parallel service calls, or background computations without deeply nested callbacks.

---

## 2. Layman Explanation

Imagine ordering something online. You do not have the item yet, but you have an order confirmation. Later the order either succeeds or fails.

The promise is that confirmation:
- pending now
- fulfilled later
- failed if something goes wrong

---

## 3. Bit by Bit Explanation

### Problem

Async operations create awkward code if every step is a nested callback:

```text
download -> parse -> validate -> save -> notify
```

Promises turn this into a chain:

```text
download().thenApply(parse).thenApply(validate).thenAccept(save)
```

### Pattern Flow

1. Start an async task.
2. Return a promise immediately.
3. Promise begins in pending/running state.
4. Task completes with value or exception.
5. Success handlers run on the value.
6. Transformations produce new promises.
7. Error handlers handle failures.

### Common States

| State | Meaning |
|---|---|
| Pending | Work is still running |
| Fulfilled | Work completed with a value |
| Rejected/Failed | Work completed with an error |
| Cancelled | Work was stopped before completion |

---

## 4. Java Coding Example

```java
import java.util.concurrent.CompletableFuture;

public class PromiseDemo {
    public static void main(String[] args) {
        CompletableFuture<Integer> lineCount =
                CompletableFuture.supplyAsync(() -> "a\nb\nc")
                        .thenApply(text -> text.split("\n").length);

        lineCount
                .thenAccept(count -> System.out.println("lines: " + count))
                .exceptionally(error -> {
                    System.out.println("failed: " + error.getMessage());
                    return null;
                })
                .join();
    }
}
```

### Java Block by Block

`supplyAsync` starts work in the background.

`thenApply` transforms the eventual string into an eventual line count.

`thenAccept` consumes the final value.

`exceptionally` handles failure.

`join` waits only at the edge of the program.

---

## 5. Python Coding Example

Python's `Future` plus callbacks gives the same idea.

```python
from concurrent.futures import ThreadPoolExecutor


def download():
    return "a\nb\nc"


def on_done(future):
    try:
        text = future.result()
        print("lines:", len(text.splitlines()))
    except Exception as error:
        print("failed:", error)


with ThreadPoolExecutor(max_workers=2) as executor:
    future = executor.submit(download)
    future.add_done_callback(on_done)
```

For Python async workflows, `asyncio.Task` is often closer to a promise-like object.

---

## 6. When to Use It

Use Promise when:
- a result will be available later
- you want to chain async steps
- you need success and error handling in one flow
- multiple async calls can run in parallel
- callbacks are becoming nested and hard to read

Interview triggers:
- "future result"
- "compose asynchronous calls"
- "avoid callback hell"
- "run in parallel and combine"
- "then apply"

---

## 7. When Not to Use It

Avoid Promise when:
- code is purely synchronous and simple
- every step must run in a strict transaction
- async execution would hide important failures
- the team needs durable queues instead of in-memory async
- backpressure and streaming are the primary concern

Use queues for durable background jobs, reactive streams for high-volume streaming, and direct synchronous calls for simple local operations.

---

## 8. Real-World Use Cases

| Use case | Why Promise fits |
|---|---|
| Parallel API calls | Combine results when all finish |
| File download then parse | Chain dependent async steps |
| UI requests | Keep UI thread responsive |
| Background computation | Return a stand-in now |
| Service aggregation | Compose multiple remote calls |
| Async validation | Attach success/failure handlers |

---

## 9. Advantages Over Normal Code

Without Promise:
- nested callbacks become hard to read
- error handling is scattered
- composition is manual
- callers may block too early

With Promise:
- async result has a first-class object
- transformations are chainable
- errors can flow through the chain
- multiple tasks can be composed

---

## 10. Where It Excels

Promise excels when async operations are value-oriented:
- fetch user
- fetch orders
- transform response
- combine results
- handle failure

It is especially useful for fan-out/fan-in workflows.

---

## 11. Where It Fails

It fails when:
- chains become too long and unreadable
- errors are swallowed
- blocking calls are placed inside async stages
- executor sizing is ignored
- the promise is never completed
- cancellation is not handled

In Java, prefer `CompletableFuture` carefully with explicit executors for production workloads.

---

## 12. Frameworks and Libraries

| Ecosystem | Tools |
|---|---|
| Java | `CompletableFuture`, `Future`, Guava `ListenableFuture` |
| JavaScript | `Promise`, `async/await` |
| Python | `asyncio.Future`, `asyncio.Task`, `concurrent.futures.Future` |
| Scala | `Future`, Cats Effect IO |
| Kotlin | `Deferred`, coroutines |

---

## 13. Pros and Cons

| Pros | Cons |
|---|---|
| Makes async values explicit | Can hide threading details |
| Supports chaining | Debugging async chains can be hard |
| Centralizes error handling | Easy to block accidentally |
| Avoids nested callbacks | Cancellation can be tricky |
| Good for composition | Not durable across process restarts |

---

## 14. Real-World Identification Scenario

Question:

> An API gateway calls profile, orders, and recommendations services in parallel, then combines the responses. What pattern helps model the async results?

Strong answer:

Use Promise/Future objects. Start all three calls concurrently and get three promises. Compose them with an all-of/join step, apply timeouts and fallbacks, then combine the successful values. In Java, I would likely use `CompletableFuture` with a bounded executor and explicit error handling.

---

## 15. Interview Triggers

Say Promise when you hear:
- future value
- async result
- chain operations
- compose callbacks
- complete later
- fan-out/fan-in
- `CompletableFuture`

---

## 16. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Calling `get` too early | Turns async into blocking sync | Block only at boundaries |
| No error handler | Failures disappear or crash late | Add exception handling |
| Using common pool blindly | Can starve unrelated work | Use explicit executor |
| Mixing blocking I/O in stages | Wastes async workers | Separate blocking pools |
| Ignoring timeout | Requests hang too long | Add timeout/fallback |

---

## 17. Pattern Comparison and Design Checklist

| Pattern | Difference |
|---|---|
| Async Method Invocation | Async method may return a promise/future |
| Callback | Promise structures callbacks into composable chains |
| Event-Based Asynchronous | Event-based async notifies by events; promise models the result |
| Reactor | Reactor handles readiness events; promises model eventual values |
| Producer-Consumer | Queue-based work handoff rather than a single future result |

Design checklist:
- What async value does the promise represent?
- Which executor runs the work?
- Where are timeouts handled?
- Where are exceptions handled?
- Is any stage accidentally blocking?
- Does cancellation need to propagate?

---

## 18. Quick Revision Notes and Mini Exercise

- One-line summary: A stand-in for a value that completes later.
- Memory hook: "order confirmation for an async result."
- Best for: composing async operations.
- Avoid when: durable background processing or streaming is needed.
- Interview line: "I would return a future/promise, chain transformations, add timeout and error handling, and block only at the outer boundary."

Mini exercise:

Use `CompletableFuture` to call two fake services in parallel, combine their responses, add a timeout, and return a fallback if either service fails.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/promise/README.md)
- [Promise.java](../../github-repo/promise/src/main/java/com/iluwatar/promise/Promise.java)
- [PromiseSupport.java](../../github-repo/promise/src/main/java/com/iluwatar/promise/PromiseSupport.java)
- [Utility.java](../../github-repo/promise/src/main/java/com/iluwatar/promise/Utility.java)
- [App.java](../../github-repo/promise/src/main/java/com/iluwatar/promise/App.java)

The repo implementation builds a custom `Promise` on top of a simplified `Future`, then demonstrates `thenApply`, `thenAccept`, async fulfillment, and error handling.
