# Guarded Suspension Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [guarded-suspension](../../github-repo/guarded-suspension)

---

## How to Study This Pattern

Study this pattern as the foundation behind "wait until condition is true" concurrency.

Remember:
- A method has a precondition.
- If the precondition is false, the thread waits.
- Another thread changes state and notifies waiting threads.

---

## 1. Technical Definition

Guarded Suspension is a concurrency pattern where a thread attempting an operation waits until a required condition becomes true, usually using a lock plus condition check.

### 30-Second Interview Answer

Guarded Suspension protects operations that cannot safely proceed until some condition is satisfied. A consumer waiting on an empty queue is the classic example. The consumer checks the guard condition in a loop, waits if it is false, and resumes when a producer updates state and notifies it.

---

## 2. Layman Explanation

Imagine arriving at a pickup counter before your order is ready. You do not keep asking every second. You wait until your name is called, then you collect the order.

The guard is:

```text
Order is ready?
```

If no, wait. If yes, proceed.

---

## 3. Bit by Bit Explanation

### Problem

Some operations are invalid until state changes:
- take from an empty queue
- read a result before it exists
- send only after a connection opens
- proceed only after configuration loads

Busy waiting wastes CPU. Proceeding early causes incorrect behavior.

### Pattern Flow

1. Thread enters a synchronized/locked method.
2. Thread checks the guard condition.
3. If the condition is false, it waits and releases the lock.
4. Another thread changes the state.
5. The other thread notifies waiting threads.
6. The waiting thread wakes up and checks the condition again.
7. If true, it performs the operation.

### Why the Loop Matters

Always use:

```java
while (!condition) {
    wait();
}
```

not:

```java
if (!condition) {
    wait();
}
```

Threads can wake up without the condition being true, and multiple waiters may compete for the same state.

---

## 4. Java Coding Example

```java
import java.util.LinkedList;
import java.util.Queue;

class GuardedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();

    public synchronized T take() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        return queue.remove();
    }

    public synchronized void put(T value) {
        queue.add(value);
        notifyAll();
    }
}

public class GuardedSuspensionDemo {
    public static void main(String[] args) {
        GuardedBuffer<String> buffer = new GuardedBuffer<>();

        new Thread(() -> {
            try {
                System.out.println("waiting for item");
                System.out.println("received " + buffer.take());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();

        new Thread(() -> {
            try {
                Thread.sleep(500);
                buffer.put("order-42");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}
```

### Java Block by Block

`take` is guarded by `queue.isEmpty()`.

If the queue is empty, the caller waits.

`put` adds data and calls `notifyAll` so waiting threads can re-check the condition.

The `while` loop protects against spurious wakeups and competing consumers.

---

## 5. Python Coding Example

```python
from threading import Condition, Thread
from collections import deque
import time


class GuardedBuffer:
    def __init__(self):
        self.items = deque()
        self.condition = Condition()

    def take(self):
        with self.condition:
            while not self.items:
                self.condition.wait()
            return self.items.popleft()

    def put(self, item):
        with self.condition:
            self.items.append(item)
            self.condition.notify_all()


buffer = GuardedBuffer()


def consumer():
    print("waiting")
    print("received", buffer.take())


def producer():
    time.sleep(0.5)
    buffer.put("order-42")


Thread(target=consumer).start()
Thread(target=producer).start()
```

---

## 6. When to Use It

Use Guarded Suspension when:
- an operation has a clear precondition
- waiting is better than failing immediately
- another thread can eventually satisfy the condition
- you need efficient waiting instead of busy polling

Interview triggers:
- "wait until item exists"
- "block until ready"
- "condition variable"
- "producer wakes consumer"
- "avoid busy waiting"

---

## 7. When Not to Use It

Avoid it when:
- the caller should fail fast instead of wait
- the wait can last forever without timeout
- you can use a higher-level blocking queue
- async callbacks would be clearer
- waiting threads would reduce scalability

Prefer `BlockingQueue`, `Semaphore`, `CountDownLatch`, or `CompletableFuture` when they directly fit.

---

## 8. Real-World Use Cases

| Use case | Guard condition |
|---|---|
| Consumer queue | Queue is not empty |
| Connection pool | Connection is available |
| Cache warmup | Value has been loaded |
| Startup dependency | Service is initialized |
| Result handoff | Result is complete |
| Rate limiter | Permit is available |

---

## 9. Advantages Over Normal Code

Without this pattern:
- code may spin in a loop
- CPU is wasted
- consumers may read invalid state
- concurrency bugs are likely

With this pattern:
- waiting is efficient
- the condition is explicit
- access is synchronized
- producer and consumer coordinate safely

---

## 10. Where It Excels

It excels in small shared-state coordination problems where one thread must wait for another thread to make progress.

The classic examples are:
- queue take
- result get
- connection acquire
- startup readiness

---

## 11. Where It Fails

It fails when:
- no thread ever sends a notification
- the guard condition is checked with `if` instead of `while`
- the lock is held while doing slow work
- `notify` wakes the wrong waiter
- there is no timeout or cancellation path

In production systems, always think about timeout and interruption.

---

## 12. Frameworks and Libraries

| Ecosystem | Tools |
|---|---|
| Java | `wait/notify`, `Condition`, `BlockingQueue`, `Semaphore` |
| Java concurrency | `CountDownLatch`, `CyclicBarrier`, `Phaser` |
| Python | `threading.Condition`, `queue.Queue` |
| Kotlin | Coroutines channels, mutex/condition equivalents |
| C++ | `std::condition_variable` |

---

## 13. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids busy waiting | Easy to misuse |
| Makes preconditions explicit | Can deadlock if locks are mishandled |
| Efficient thread coordination | Needs timeout/cancellation thinking |
| Simple for queue-like flows | Lower-level than modern abstractions |
| Protects shared state | Debugging waits can be hard |

---

## 14. Real-World Identification Scenario

Question:

> A worker thread should wait until a configuration file has finished loading. It should not poll every second. What pattern applies?

Strong answer:

Use Guarded Suspension. The worker checks a condition such as `configLoaded`. If false, it waits on a condition variable. The loader thread sets `configLoaded = true` and notifies waiters. The worker re-checks the condition in a loop before using the configuration.

---

## 15. Interview Triggers

Say Guarded Suspension when you hear:
- condition must become true
- wait without polling
- producer notifies consumer
- synchronized wait
- condition variable
- spurious wakeup
- `while` around `wait`

---

## 16. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Using `if` before `wait` | Wakeups can happen before condition is true | Use `while` |
| Forgetting notification | Waiters can block forever | Notify after state change |
| No timeout | Failures become invisible hangs | Use bounded waits where possible |
| Holding lock during slow work | Blocks other threads | Keep critical section short |
| Ignoring interruption | Shutdown becomes hard | Restore interrupt flag |

---

## 17. Pattern Comparison and Design Checklist

| Pattern | Difference |
|---|---|
| Monitor | Encapsulates mutual exclusion; guarded suspension adds condition waiting |
| Producer-Consumer | Often implemented using guarded suspension |
| Balking | Returns immediately if condition is false instead of waiting |
| Promise | Waits for an eventual value with a higher-level API |
| Semaphore | Counts permits rather than guarding arbitrary condition logic |

Design checklist:
- What exact condition guards the operation?
- Is the condition checked in a `while` loop?
- Which thread changes the condition?
- Which thread sends the notification?
- Do waiters need a timeout?
- Is interruption handled correctly?

---

## 18. Quick Revision Notes and Mini Exercise

- One-line summary: Wait until the guard condition becomes true.
- Memory hook: "Do not enter until the sign says ready."
- Best for: condition-based thread coordination.
- Avoid when: fail-fast or higher-level concurrency utilities fit better.
- Interview line: "I would guard the operation with a condition, wait in a loop, and notify after state changes."

Mini exercise:

Build a `ResultBox<T>` with `set(value)` and `get()`. `get()` should wait until `set()` is called, then return the value. Add a timeout version as a stretch goal.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/guarded-suspension/README.md)
- [GuardedQueue.java](../../github-repo/guarded-suspension/src/main/java/com/iluwatar/guarded/suspension/GuardedQueue.java)
- [App.java](../../github-repo/guarded-suspension/src/main/java/com/iluwatar/guarded/suspension/App.java)

The repo implementation uses `GuardedQueue.get()` to wait while the queue is empty and `GuardedQueue.put()` to add an item and notify the waiting thread.
