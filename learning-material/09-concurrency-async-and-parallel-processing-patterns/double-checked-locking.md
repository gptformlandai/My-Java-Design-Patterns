# Double-Checked Locking Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: High  
Software usage meter: Medium  
Repository module: [github-repo/double-checked-locking](../../github-repo/double-checked-locking)

## How to Study This Page

Use this page in three passes:

1. First pass: understand lazy initialization with minimal locking after initialization.
2. Second pass: trace first null check, synchronized block, second null check, and volatile visibility.
3. Third pass: compare with eager initialization, initialization-on-demand holder, and singleton.

By the end, you should be able to say:

> Double-Checked Locking lazily initializes a shared object once while avoiding lock cost after initialization.

## 1. Technical Definition

Double-Checked Locking is a concurrency optimization where code checks whether a shared object is initialized before and after acquiring a lock.

Core idea:

- First check avoids locking once initialized.
- Lock protects initialization.
- Second check prevents duplicate creation.
- `volatile` or safe publication is required in Java.
- The pattern is most useful for expensive lazy initialization.

### 30-Second Interview Answer

I would use Double-Checked Locking only when lazy initialization is required and lock overhead matters. In Java, the shared reference must be `volatile` to prevent visibility and reordering bugs. Often I prefer simpler alternatives like eager initialization, enum singleton, or initialization-on-demand holder unless the use case truly needs this optimization.

## 2. Layman and Easy to Understand Definition

Double-Checked Locking is checking the door twice.

First, you check quickly without using the lock. If setup might still be needed, you lock and check again before doing the expensive setup.

In code:

- If object exists, return it.
- If missing, acquire lock.
- Check again.
- Create once.
- Return object.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Simple synchronized access is safe but can be expensive:

```java
synchronized Resource get() {
    if (resource == null) {
        resource = new Resource();
    }
    return resource;
}
```

After initialization, every call still pays lock overhead.

### 3.2 The Double Check Solution

```text
check without lock
lock
check again
initialize
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Lazy field | Object created on first use. |
| First check | Fast path after initialization. |
| Lock | Protects initialization. |
| Second check | Avoids duplicate initialization. |
| Volatile | Ensures safe publication in Java. |

### 3.4 Why `volatile` Matters

Without safe publication, another thread can observe a reference before the object is fully initialized. In modern Java, `volatile` prevents the unsafe reordering that made older implementations broken.

## 4. Java Coding Example

```java
class ExpensiveService {
    String execute() {
        return "done";
    }
}

class ServiceRegistry {
    private volatile ExpensiveService service;

    ExpensiveService getService() {
        ExpensiveService local = service;
        if (local == null) {
            synchronized (this) {
                local = service;
                if (local == null) {
                    local = new ExpensiveService();
                    service = local;
                }
            }
        }
        return local;
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `volatile` | Safe publication. |
| local variable | Avoids repeated volatile reads. |
| first `if` | Fast path. |
| `synchronized` | Protects initialization. |
| second `if` | Prevents duplicate construction. |

### Java Usage

```java
ServiceRegistry registry = new ServiceRegistry();
System.out.println(registry.getService().execute());
```

## 5. Python Coding Example

Python has different memory and threading semantics, but the idea can be shown with a lock.

```python
from threading import Lock


class Registry:
    def __init__(self):
        self._service = None
        self._lock = Lock()

    def get_service(self):
        if self._service is None:
            with self._lock:
                if self._service is None:
                    self._service = object()
        return self._service
```

### Python Usage

Use this shape when explaining:

- Fast path avoids lock after creation.
- Lock protects first creation.
- Simpler lazy initialization is usually preferred if performance does not demand this.

## 6. Where It Comes Handy in Real Life

- Expensive singleton-like resources.
- Lazy caches.
- Lazy client initialization.
- Optional heavy dependencies.
- Rarely used service adapters.

## 7. Advantages Over Normal Code Without Pattern

Without Double-Checked Locking:

```text
every access may lock
```

With Double-Checked Locking:

```text
only first initialization locks
```

Benefits:

- Lazy initialization.
- Lower lock overhead after initialization.
- Thread-safe single creation.
- Avoids eager cost for unused resources.

## 8. Where It Excels

- Object creation is expensive.
- Object may never be used.
- Access after initialization is very frequent.
- Correct safe publication is understood.

## 9. Where It Fails

- `volatile` is missing in Java.
- Initialization has side effects that must be retried carefully.
- Simpler alternatives would be clearer.
- The object must be reset or reloaded.
- Lock contention during startup is acceptable anyway.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java language | `volatile`, `synchronized`, static holder idiom |
| Java utilities | `AtomicReference`, `Supplier`, `Lazy` helpers in libraries |
| Dependency injection | Spring singleton beans, Guice singletons |
| Alternatives | Enum singleton, eager initialization, memoized supplier |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids lock after initialization. | Easy to implement incorrectly. |
| Lazily creates expensive object. | More complex than eager init. |
| Thread-safe when done correctly. | Usually unnecessary with modern alternatives. |
| Useful in hot paths. | Harder to reason about memory visibility. |

## 12. Real-World Identification Example

Scenario: A service lazily creates a heavy parser used by only some requests.

Double-Checked Locking fit:

- Parser is expensive.
- Many requests use it after creation.
- Only one parser should exist.
- `volatile` plus synchronized block safely publishes it.

Without it:

- Eager initialization slows startup.
- Fully synchronized getter locks every access.

## 13. MAANG Interview Triggers

Use Double-Checked Locking when you hear:

- "Thread-safe lazy singleton."
- "Avoid synchronized overhead."
- "Why volatile is needed?"
- "Safe publication."
- "Lazy initialization under concurrency."

Strong answer keywords:

- first check
- synchronized
- second check
- volatile
- safe publication
- memory reordering
- initialization-on-demand holder

## 14. Common Mistakes

### Mistake 1: Missing `volatile`

- Why it is wrong: another thread may see partially initialized state.
- Better approach: mark the shared reference `volatile` or use safer alternatives.

### Mistake 2: Synchronizing the whole method unnecessarily

- Why it is wrong: every access pays lock cost.
- Better approach: lock only around initialization if optimization is needed.

### Mistake 3: Using it when simple code is enough

- Why it is wrong: complexity is not justified.
- Better approach: use eager initialization or holder idiom.

### Mistake 4: Resetting the field casually

- Why it is wrong: lifecycle becomes unsafe and confusing.
- Better approach: treat it as one-time initialization.

## 15. Double-Checked Locking vs Similar Patterns

| Pattern | Difference |
|---|---|
| Double-Checked Locking | Lazy init with minimized lock overhead. |
| Singleton | Ensures one instance; may use this pattern. |
| Lazy Loading | Broader concept of delayed creation. |
| Initialization-on-demand holder | Simpler Java lazy singleton using class loading. |
| Eager Initialization | Creates object at startup without lazy checks. |

## 16. Double-Checked Locking Design Checklist

- Is lazy initialization necessary?
- Is the object expensive enough to justify complexity?
- Is the field `volatile`?
- Is there a second check inside the lock?
- Is the object safely constructed?
- Is reset/reload avoided?
- Would holder idiom be simpler?

## 17. Quick Revision Notes

- One-line summary: Double-Checked Locking lazily creates one object with minimal post-init locking.
- Three keywords: lazy, volatile, synchronized.
- Interview trap: forgetting `volatile`.
- Memory trick: check, lock, check again.

## 18. Mini Exercise

Implement a lazy configuration parser.

Answer these:

1. Why is lazy creation needed?
2. What field must be `volatile`?
3. Where is the second check?
4. What simpler alternative could replace it?
5. What test would expose duplicate initialization?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/double-checked-locking/README.md](../../github-repo/double-checked-locking/README.md)
- [github-repo/double-checked-locking/src/main/java/com/iluwatar/doublechecked/locking/Inventory.java](../../github-repo/double-checked-locking/src/main/java/com/iluwatar/doublechecked/locking/Inventory.java)
- [github-repo/double-checked-locking/src/main/java/com/iluwatar/doublechecked/locking/Item.java](../../github-repo/double-checked-locking/src/main/java/com/iluwatar/doublechecked/locking/Item.java)
- [github-repo/double-checked-locking/src/main/java/com/iluwatar/doublechecked/locking/App.java](../../github-repo/double-checked-locking/src/main/java/com/iluwatar/doublechecked/locking/App.java)
