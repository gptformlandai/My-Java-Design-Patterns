# Async Method Invocation Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/async-method-invocation](../../github-repo/async-method-invocation)

## How to Study This Page

Use this page in three passes:

1. First pass: understand that method call returns before the work completes.
2. Second pass: trace async start, future/result, callback, await, and error propagation.
3. Third pass: compare Async Method Invocation with Promise, Thread Pool, and Event-Based Async.

By the end, you should be able to say:

> Async Method Invocation starts work in the background and lets the caller collect the result later.

## 1. Technical Definition

Async Method Invocation is a concurrency pattern where a method starts an operation asynchronously and returns a handle, future, or callback instead of blocking for completion.

Core idea:

- Caller starts operation.
- Operation runs on another thread or event loop.
- Caller continues doing other work.
- Result is obtained later.
- Errors must be captured and reported asynchronously.

### 30-Second Interview Answer

I would use Async Method Invocation when an operation is slow or I/O-bound and the caller does not need the result immediately. The method returns a future-like handle or accepts a callback. The trade-off is more complex control flow, error handling, cancellation, and resource management.

## 2. Layman and Easy to Understand Definition

Async Method Invocation is like placing an order and getting a pickup number.

You do not wait at the counter. You do other things and come back when the order is ready.

In code:

- Start async work.
- Receive future/result handle.
- Continue.
- Await or callback later.
- Handle success or failure.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Synchronous calls block:

```text
caller -> slow operation -> caller waits
```

Blocking wastes caller time when useful work can continue.

### 3.2 The Async Solution

```text
caller -> start async operation -> returns future
caller -> does other work
caller -> awaits future when needed
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Async executor | Runs work in background. |
| Async result | Handle for eventual result. |
| Callback | Optional function called on completion. |
| Task | Work to execute. |
| Await/end | Joins with result when needed. |

### 3.4 Completion Flow

1. Caller submits task.
2. Executor starts task asynchronously.
3. Caller receives result handle.
4. Task completes with value or exception.
5. Callback runs or caller awaits.
6. Result or error is delivered.

## 4. Java Coding Example

```java
import java.util.concurrent.CompletableFuture;

class AsyncUserService {
    CompletableFuture<String> loadUserName(String userId) {
        return CompletableFuture.supplyAsync(() -> "user-" + userId);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `CompletableFuture` | Async result handle. |
| `supplyAsync` | Runs supplier asynchronously. |
| `loadUserName` | Returns immediately. |
| returned future | Caller can compose or await. |

### Java Usage

```java
AsyncUserService service = new AsyncUserService();

CompletableFuture<String> future = service.loadUserName("42");
future.thenAccept(System.out::println);
```

## 5. Python Coding Example

```python
import asyncio


async def load_user_name(user_id):
    await asyncio.sleep(0.1)
    return f"user-{user_id}"


async def main():
    task = asyncio.create_task(load_user_name("42"))
    print(await task)


asyncio.run(main())
```

### Python Usage

Use this shape when explaining:

- Call returns a task/future.
- Await happens only when result is needed.
- Errors surface later, not at call start.

## 6. Where It Comes Handy in Real Life

- Remote API calls.
- Disk I/O.
- UI background work.
- Parallel independent requests.
- Email sending.
- Report generation.
- Non-blocking service composition.

## 7. Advantages Over Normal Code Without Pattern

Without Async Method Invocation:

```text
caller blocks until work completes
```

With Async Method Invocation:

```text
caller starts work and continues
```

Benefits:

- Better responsiveness.
- More parallelism.
- Improved resource utilization.
- Supports callbacks and composition.
- Reduces waiting in caller thread.

## 8. Where It Excels

- Work is independent.
- Result is needed later.
- Operation is I/O-bound or long-running.
- Caller has other work to do.
- Async error handling is acceptable.

## 9. Where It Fails

- Caller immediately blocks on every future.
- Too many async tasks overload executor.
- Exceptions are ignored.
- Cancellation is not supported.
- Shared state is mutated without synchronization.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | `CompletableFuture`, `Future`, `ExecutorService`, Spring `@Async` |
| Reactive | Reactor `Mono`/`Flux`, RxJava |
| Python | `asyncio`, `concurrent.futures` |
| JavaScript | Promise, async/await |
| Messaging | Async request/reply patterns |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps caller responsive. | Control flow is harder. |
| Enables parallel work. | Error handling is async. |
| Composes independent tasks. | Can overload executors. |
| Fits I/O-heavy systems. | Debugging stack traces is harder. |

## 12. Real-World Identification Example

Scenario: Profile page needs user info, badges, and preferences from three services.

Async Method Invocation fit:

- Start three calls concurrently.
- Continue preparing page.
- Await all results.
- Return combined response.

Without it:

- Calls run sequentially.
- Latency becomes the sum of all calls.

## 13. MAANG Interview Triggers

Use Async Method Invocation when you hear:

- "Do not block caller."
- "Run operation in background."
- "Future or callback."
- "Parallelize independent I/O."
- "Improve responsiveness."

Strong answer keywords:

- future
- callback
- executor
- await
- cancellation
- exception handling
- composition
- timeout

## 14. Common Mistakes

### Mistake 1: Blocking immediately

- Why it is wrong: async call becomes synchronous.
- Better approach: start independent work first, await only when needed.

### Mistake 2: Ignoring exceptions

- Why it is wrong: failures disappear in background.
- Better approach: attach error handlers or inspect futures.

### Mistake 3: Unlimited async submission

- Why it is wrong: executor queue can explode.
- Better approach: bound executors and apply backpressure.

### Mistake 4: No cancellation path

- Why it is wrong: abandoned work wastes resources.
- Better approach: propagate cancellation and timeouts.

## 15. Async Method Invocation vs Similar Patterns

| Pattern | Difference |
|---|---|
| Async Method Invocation | Starts method asynchronously and returns result handle. |
| Promise | Structured future with chaining and error handling. |
| Callback | Function invoked on completion. |
| Active Object | Object queues its own method requests. |
| Thread Pool | Execution mechanism often used underneath. |

## 16. Async Method Invocation Design Checklist

- What executor runs the work?
- What result type is returned?
- How are exceptions surfaced?
- Is timeout supported?
- Is cancellation supported?
- Is executor bounded?
- Does caller really have parallel work?
- Are results combined safely?

## 17. Quick Revision Notes

- One-line summary: Async Method Invocation starts work now and returns a result handle for later.
- Three keywords: future, callback, executor.
- Interview trap: calling `get` immediately after every async call.
- Memory trick: start now, collect later.

## 18. Mini Exercise

Make three independent service calls async.

Answer these:

1. Which calls can start in parallel?
2. What future type is returned?
3. What timeout applies?
4. How are errors combined?
5. What executor capacity is safe?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/async-method-invocation/README.md](../../github-repo/async-method-invocation/README.md)
- [github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/AsyncExecutor.java](../../github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/AsyncExecutor.java)
- [github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/AsyncResult.java](../../github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/AsyncResult.java)
- [github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/AsyncCallback.java](../../github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/AsyncCallback.java)
- [github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/ThreadAsyncExecutor.java](../../github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/ThreadAsyncExecutor.java)
- [github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/App.java](../../github-repo/async-method-invocation/src/main/java/com/iluwatar/async/method/invocation/App.java)
