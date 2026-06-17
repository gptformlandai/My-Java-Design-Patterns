# Execute Around Pattern

Category: Language, JVM, and Idiom Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [execute-around](../../github-repo/execute-around)

---

## How to Study This Page

Study Execute Around as "put reusable setup and cleanup around caller-provided business logic."

Remember the core flow:

```text
before -> caller action -> after
```

This pattern is common when resources, locks, transactions, spans, or connections must always be opened and closed correctly.

---

## 1. Technical Definition

Execute Around is an idiom where a reusable method or object performs mandatory pre-processing and post-processing around a caller-supplied action, usually represented as a lambda, callback, or functional interface.

### 30-Second Interview Answer

Execute Around removes repeated setup and cleanup boilerplate. The framework or helper opens a resource, starts a transaction, acquires a lock, or creates a context; the caller supplies only the work to execute; then the helper guarantees cleanup afterward. I would use it for files, locks, database sessions, transactions, tracing spans, and resource-safe APIs. The trade-off is that control flow is inverted, so the helper must make exception and lifecycle behavior very clear.

---

## 2. Layman and Easy to Understand Definition

Imagine a conference room booking.

The building staff unlocks the room before your meeting and locks it afterward. You only focus on the meeting.

Execute Around does that for code: it handles the repeated before/after work around your custom action.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Resource code is easy to duplicate and forget:

```text
open file
write data
close file
```

If an exception happens, cleanup may be skipped unless every caller handles it correctly.

### Execute Around Flow

1. Define the variable work as a function or callback.
2. Helper method performs setup.
3. Helper invokes the caller-supplied action.
4. Helper performs cleanup in `finally` or try-with-resources.
5. Caller receives a small API focused on business intent.

### Core Participants

| Participant | Responsibility |
|---|---|
| Resource/helper | Owns setup and cleanup |
| Action/callback | Caller-provided work |
| Before step | Opens, starts, locks, or prepares |
| After step | Closes, commits, rolls back, unlocks, or cleans |
| Exception policy | Defines cleanup and propagation rules |
| Caller | Supplies only the meaningful operation |

---

## 4. Java Coding Example

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class LockedExecutor {
    private final Lock lock = new ReentrantLock();

    void withLock(Runnable action) {
        lock.lock();
        try {
            action.run();
        } finally {
            lock.unlock();
        }
    }
}

public class ExecuteAroundDemo {
    public static void main(String[] args) {
        LockedExecutor executor = new LockedExecutor();

        executor.withLock(() -> {
            System.out.println("critical section");
        });
    }
}
```

### Java Block by Block

`withLock` owns the lock lifecycle.

The caller only supplies the critical section.

`finally` ensures the lock is released even if the action fails.

The same wrapper can be reused everywhere lock handling is needed.

---

## 5. Python Coding Example

Python's context managers express the same idea.

```python
from contextlib import contextmanager


@contextmanager
def transaction():
    print("begin")
    try:
        yield
        print("commit")
    except Exception:
        print("rollback")
        raise
    finally:
        print("close")


with transaction():
    print("business work")
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| File I/O | Open and close reliably |
| Database transactions | Commit or rollback around work |
| Locks | Acquire and release safely |
| Tracing spans | Start and finish spans consistently |
| Security context | Run code under temporary privileges |
| Test fixtures | Setup and teardown around a test action |

---

## 7. Advantages Over Normal Code Without Pattern

Without Execute Around:
- setup and cleanup code is repeated
- callers can forget cleanup
- exception paths are inconsistent
- resource-handling rules spread across the codebase

With Execute Around:
- lifecycle rules live in one place
- caller code is focused
- cleanup is reliable
- policies such as rollback or logging are consistent

---

## 8. Where It Excels

It excels when:
- before and after steps are mandatory
- many callers need the same lifecycle rules
- cleanup must happen even on exceptions
- business logic varies but resource handling is stable
- lambdas or callbacks make the API readable
- correctness matters more than exposing manual control

---

## 9. Where It Fails

It fails when:
- the wrapper hides important control flow
- the action needs too many escape hatches
- exception handling rules are unclear
- the resource must be used outside the block
- nested wrappers become hard to read
- callers need fine-grained lifecycle control

Use explicit lifecycle management or try-with-resources when the caller must control each step directly.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java core | try-with-resources, lambdas, functional interfaces |
| Spring | TransactionTemplate, JdbcTemplate, RetryTemplate |
| Concurrency | `Lock` with helper wrappers, semaphore helpers |
| Observability | tracing span wrappers, MDC context wrappers |
| Python | context managers, `contextlib` |
| Testing | JUnit lifecycle hooks, pytest fixtures |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Removes repeated setup/cleanup | Inverts control flow |
| Improves resource safety | Can hide exception behavior |
| Centralizes lifecycle rules | Nested callbacks can hurt readability |
| Works well with lambdas | Generic wrappers may become too broad |
| Encourages consistent policies | Caller has less manual control |

---

## 12. Real-World Identification Example

Question:

> Many service methods open a database transaction, run business logic, commit, and manually rollback on exceptions. Some paths forget rollback. What pattern helps?

Strong answer:

Use Execute Around. Create a transaction helper that starts the transaction, executes a caller-provided lambda, commits on success, rolls back on failure, and closes the session in a final cleanup step. The service methods should provide only the business action.

---

## 13. MAANG Interview Triggers

Say Execute Around when you hear:
- setup and cleanup around code
- open/close pair
- lock/unlock pair
- begin/commit/rollback
- callback with resource
- resource lifecycle wrapper
- transaction template
- avoid duplicated boilerplate

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Cleanup not in `finally` | Exceptions can leak resources | Always guarantee cleanup |
| Swallowing exceptions | Failures become invisible | Propagate or map exceptions clearly |
| Wrapper does too much | API becomes magical | Keep lifecycle responsibility focused |
| Nested callback pyramids | Hard to read | Compose wrappers carefully or use context objects |
| Exposing closed resource | Caller may use invalid resource | Limit resource scope to the action |

---

## 15. Execute Around vs Similar Patterns

| Pattern | Difference |
|---|---|
| RAII | RAII ties resource cleanup to object/scope; Execute Around supplies custom work inside a lifecycle wrapper |
| Template Method | Template Method uses inheritance; Execute Around usually uses lambdas/callbacks |
| Decorator | Decorator wraps an object; Execute Around wraps an execution block |
| Command | Command represents an action; Execute Around runs an action with before/after lifecycle |
| Intercepting Filter | Filter wraps request processing; Execute Around is a general code idiom |

---

## 16. Execute Around Design Checklist

- What setup must always happen?
- What cleanup must always happen?
- What action does the caller supply?
- How are return values handled?
- How are checked and unchecked exceptions handled?
- Is cleanup guaranteed on failure?
- Is the resource visible only inside the action?
- Is the API easier than manual lifecycle code?
- Are nested wrappers still readable?

---

## 17. Quick Revision Notes

- One-line summary: Run caller logic inside reusable before/after lifecycle code.
- Memory hook: "you bring the action; helper handles the wrapper."
- Best for: resources, locks, transactions, tracing, setup/teardown.
- Avoid when: caller needs detailed lifecycle control.
- Interview line: "I would centralize the open/close or begin/commit/rollback rules and accept the variable work as a lambda."

---

## 18. Mini Exercise

Design Execute Around for a retryable operation:
- define the action interface
- log before execution
- retry on transient failure
- always record final metrics
- decide which exceptions should propagate

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/execute-around/README.md)
- [FileWriterAction.java](../../github-repo/execute-around/src/main/java/com/iluwatar/execute/around/FileWriterAction.java)
- [SimpleFileWriter.java](../../github-repo/execute-around/src/main/java/com/iluwatar/execute/around/SimpleFileWriter.java)
- [App.java](../../github-repo/execute-around/src/main/java/com/iluwatar/execute/around/App.java)

The repo implementation passes a `FileWriterAction` lambda into `SimpleFileWriter`. The writer opens the file, executes the action, and closes the file through try-with-resources.
