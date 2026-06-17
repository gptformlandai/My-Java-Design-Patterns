# Object Pool Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/object-pool](../../github-repo/object-pool)

## How to Study This Page

Use this page in three passes:

1. First pass: understand checkout, use, reset, and check-in.
2. Second pass: rewrite the Java example and explain why `try-with-resources` helps prevent leaks.
3. Third pass: study failure modes: pool exhaustion, stale state, thread safety, and resource leaks.

By the end, you should be able to say:

> Object Pool reuses expensive objects by lending them to clients and requiring clients to return them after use.

## 1. Technical Definition

Object Pool is a creational pattern that manages a set of reusable objects. Instead of creating and destroying expensive objects repeatedly, clients borrow objects from the pool and return them when finished.

Core idea:

- Keep reusable objects ready.
- Checkout an object when needed.
- Return it after use.
- Reset or validate objects before reuse.
- Limit expensive resource creation.

### 30-Second Interview Answer

I would use Object Pool when objects are expensive to create and safe to reuse, such as database connections, threads, or network clients. The pool controls object count, checkout, return, and cleanup. The main risks are leaks, stale state, pool exhaustion, and thread-safety bugs, so production pools need timeouts, validation, and safe return handling.

## 2. Layman and Easy to Understand Definition

Object Pool is like a bike-sharing station.

You do not build a new bike every time you need to ride. You check out a bike, use it, and return it so someone else can use it later.

In code:

- The pool is the bike station.
- The object is the bike.
- Checkout means borrow.
- Check-in means return.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Some objects are expensive to create.

Examples:

- Database connections.
- Threads.
- Socket connections.
- Large buffers.
- Parser instances with heavy setup.

Bad design:

```java
DatabaseConnection connection = new DatabaseConnection();
connection.query(sql);
connection.close();
```

Doing this for every request may be slow and may overload the database.

### 3.2 The Object Pool Solution

Borrow and return objects:

```java
DatabaseConnection connection = pool.checkOut();
try {
    connection.query(sql);
} finally {
    pool.checkIn(connection);
}
```

The expensive object is reused instead of destroyed.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Pool | Manages available and in-use objects. |
| Pooled object | Expensive reusable object. |
| Client | Borrows and returns the object. |
| Checkout | Operation that lends an object. |
| Check-in | Operation that returns an object. |
| Reset / validation | Cleanup before the object is reused. |

### 3.4 Mental Model

Think of Object Pool as controlled borrowing.

1. Pool creates or receives reusable objects.
2. Client checks out one object.
3. Pool marks it as in use.
4. Client performs work.
5. Client returns the object.
6. Pool resets and makes it available again.

## 4. Java Coding Example

This example pools reusable `ReportBuffer` objects.

```java
import java.util.ArrayDeque;
import java.util.Deque;

public final class ReportBuffer {
    private final StringBuilder builder = new StringBuilder();

    public void append(String text) {
        builder.append(text);
    }

    public String content() {
        return builder.toString();
    }

    public void reset() {
        builder.setLength(0);
    }
}

public final class ReportBufferPool {
    private final Deque<ReportBuffer> available = new ArrayDeque<>();
    private final int maxSize;
    private int created;

    public ReportBufferPool(int maxSize) {
        this.maxSize = maxSize;
    }

    public synchronized ReportBuffer checkOut() {
        if (!available.isEmpty()) {
            return available.removeFirst();
        }
        if (created < maxSize) {
            created++;
            return new ReportBuffer();
        }
        throw new IllegalStateException("No report buffers available");
    }

    public synchronized void checkIn(ReportBuffer buffer) {
        buffer.reset();
        available.addLast(buffer);
    }
}
```

### Java Block by Block Explanation

#### Pooled Object

```java
public final class ReportBuffer {
```

This object holds reusable internal memory.

#### Reset Method

```java
public void reset() {
    builder.setLength(0);
}
```

Reset is critical. Without it, the next borrower may see old data.

#### Pool State

```java
private final Deque<ReportBuffer> available = new ArrayDeque<>();
private final int maxSize;
private int created;
```

The pool tracks available objects and prevents unlimited creation.

#### Checkout

```java
public synchronized ReportBuffer checkOut() {
```

The method is synchronized because multiple threads may borrow from the pool.

#### Check-In

```java
public synchronized void checkIn(ReportBuffer buffer) {
    buffer.reset();
    available.addLast(buffer);
}
```

The pool resets the object before making it available again.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        ReportBufferPool pool = new ReportBufferPool(2);

        ReportBuffer buffer = pool.checkOut();
        try {
            buffer.append("Revenue report");
            System.out.println(buffer.content());
        } finally {
            pool.checkIn(buffer);
        }
    }
}
```

The `finally` block matters. It returns the object even when work fails.

### Safer Java Usage with Lease Object

```java
public final class BufferLease implements AutoCloseable {
    private final ReportBufferPool pool;
    private final ReportBuffer buffer;
    private boolean closed;

    public BufferLease(ReportBufferPool pool, ReportBuffer buffer) {
        this.pool = pool;
        this.buffer = buffer;
    }

    public ReportBuffer get() {
        return buffer;
    }

    @Override
    public void close() {
        if (!closed) {
            closed = true;
            pool.checkIn(buffer);
        }
    }
}
```

This allows:

```java
try (BufferLease lease = new BufferLease(pool, pool.checkOut())) {
    lease.get().append("Revenue report");
    System.out.println(lease.get().content());
}
```

## 5. Python Coding Example

Python can model a pool with a context manager.

```python
from collections import deque
from contextlib import contextmanager
from typing import Iterator


class ReportBuffer:
    def __init__(self) -> None:
        self._parts: list[str] = []

    def append(self, text: str) -> None:
        self._parts.append(text)

    def content(self) -> str:
        return "".join(self._parts)

    def reset(self) -> None:
        self._parts.clear()


class ReportBufferPool:
    def __init__(self, max_size: int) -> None:
        self._max_size = max_size
        self._created = 0
        self._available: deque[ReportBuffer] = deque()

    def check_out(self) -> ReportBuffer:
        if self._available:
            return self._available.popleft()
        if self._created < self._max_size:
            self._created += 1
            return ReportBuffer()
        raise RuntimeError("No report buffers available")

    def check_in(self, buffer: ReportBuffer) -> None:
        buffer.reset()
        self._available.append(buffer)

    @contextmanager
    def lease(self) -> Iterator[ReportBuffer]:
        buffer = self.check_out()
        try:
            yield buffer
        finally:
            self.check_in(buffer)
```

### Python Usage

```python
pool = ReportBufferPool(max_size=2)

with pool.lease() as buffer:
    buffer.append("Revenue report")
    print(buffer.content())
```

The context manager guarantees return on success or failure.

## 6. Where It Comes Handy in Real Life

Object Pool is useful when resources are costly and reusable.

Examples:

- Database connection pools.
- Thread pools.
- HTTP connection pools.
- Byte buffer pools.
- Parser pools.
- GPU or rendering resources.
- Game entity pools for frequently spawned objects.

## 7. Advantages Over Normal Code Without Pattern

### Without Object Pool

```java
Connection connection = DriverManager.getConnection(url);
```

Doing this per request can be expensive.

Problems:

- Slow creation path.
- Resource exhaustion.
- Repeated initialization.
- More pressure on garbage collection.

### With Object Pool

```java
Connection connection = pool.checkOut();
try {
    runQuery(connection);
} finally {
    pool.checkIn(connection);
}
```

Benefits:

- Reuses expensive resources.
- Controls maximum resource count.
- Can improve latency.
- Can protect downstream systems from overload.

## 8. Where It Excels

Object Pool excels when:

- Object creation is expensive.
- Objects are reusable after reset.
- Maximum concurrency must be controlled.
- Resource count must be bounded.
- Allocation and garbage collection are performance bottlenecks.

## 9. Where It Fails

Object Pool is a poor fit when:

- Objects are cheap to create.
- Objects are not safely reusable.
- Reset is hard or unreliable.
- The pool causes contention.
- Leaked objects are likely.
- Modern runtime allocation is already fast enough.

Example where Object Pool is overkill:

```java
new StringBuilder();
```

For ordinary short-lived objects, pooling may make performance worse.

## 10. Prebuilt Libraries and Packages

### Java

Common production pools:

- HikariCP for JDBC connections.
- Apache Commons Pool.
- Java executor thread pools.
- HTTP client connection pools.
- Netty buffer pools.

Important production features:

- Max size.
- Timeout.
- Validation.
- Reset.
- Health checks.
- Leak detection.
- Metrics.

### Python

Common Python pool examples:

- SQLAlchemy connection pools.
- `concurrent.futures.ThreadPoolExecutor`.
- HTTP connection pools in `urllib3` and `requests`.
- Async connection pools for database clients.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces expensive object creation. | Adds lifecycle complexity. |
| Controls resource count. | Leaks can exhaust the pool. |
| Can improve latency. | Stale state can leak between users. |
| Helps protect downstream systems. | Requires thread-safe management. |
| Can reduce garbage collection pressure. | Pool contention can hurt throughput. |

## 12. Real-World Identification Example

Scenario:

You are designing a service that talks to a database.

Requirements:

- Thousands of requests per minute.
- Database connections are expensive.
- Database can handle only a bounded number of connections.
- Connections must be reused safely.

Should you use Object Pool?

Yes, but use a proven connection pool rather than writing your own.

Good usage:

```java
try (Connection connection = dataSource.getConnection()) {
    runQuery(connection);
}
```

The pool is hidden behind `DataSource`, and `close()` returns the connection to the pool.

## 13. MAANG Interview Triggers

Think Object Pool when you hear:

- Expensive object creation.
- Connection pool.
- Thread pool.
- Resource reuse.
- Checkout and return.
- Pool exhaustion.
- Bounded resources.
- Reset before reuse.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the expensive reusable resource.
2. Explain why repeated creation is costly.
3. Define checkout and check-in behavior.
4. Add max size and timeout.
5. Reset or validate objects before reuse.
6. Mention trade-offs: leaks, contention, stale state, and complexity.

## 14. Common Mistakes

### Mistake 1: Forgetting to Return Objects

Bad:

```java
ReportBuffer buffer = pool.checkOut();
buffer.append("data");
```

Better:

```java
ReportBuffer buffer = pool.checkOut();
try {
    buffer.append("data");
} finally {
    pool.checkIn(buffer);
}
```

### Mistake 2: Not Resetting State

If pooled objects keep old state, one user may see another user's data.

### Mistake 3: No Max Size

An unbounded pool can become a memory leak or overload a downstream system.

### Mistake 4: No Timeout

If every object is checked out, callers should not wait forever.

### Mistake 5: Pooling Cheap Objects

Pooling cheap short-lived objects can be slower than normal allocation.

## 15. Object Pool vs Similar Patterns

| Pattern | Difference |
|---|---|
| Prototype | Creates new copies. Object Pool reuses existing instances. |
| Singleton | Allows one instance. Object Pool manages multiple reusable instances. |
| Factory | Creates objects. Object Pool lends and reuses objects. |
| Flyweight | Shares immutable or intrinsic state. Object Pool leases mutable reusable objects. |
| Cache | Stores data/results for lookup. Object Pool manages resource lifecycle. |

## 16. Production Pool Checklist

| Concern | Why it matters |
|---|---|
| Max size | Prevents unlimited resource growth. |
| Checkout timeout | Avoids infinite waiting. |
| Reset | Prevents stale state leaks. |
| Validation | Removes broken objects. |
| Metrics | Shows saturation and leaks. |
| Thread safety | Required for shared pools. |
| Leak detection | Finds borrowers that never return objects. |

## 17. Quick Revision Notes

- Object Pool reuses expensive objects.
- Clients check out and check in objects.
- Always return objects in `finally` or via scoped lease.
- Reset state before reuse.
- Production pools need max size, timeout, validation, and metrics.
- Do not pool cheap objects by default.

## 18. Mini Exercise

Design an Object Pool for `PdfRenderer`.

Requirements:

- Renderer initialization is expensive.
- Maximum pool size is 5.
- Borrowers must return renderers.
- Renderer state must be reset before reuse.
- Checkout should fail after a timeout.

Expected usage:

```java
try (RendererLease lease = rendererPool.borrow()) {
    lease.renderer().render(document);
}
```

## 19. Source Reference in This Repo

The repository's Object Pool implementation uses `ObjectPool<T>` and an `OliphauntPool` that tracks available and in-use objects.

Useful files:

- [github-repo/object-pool/README.md](../../github-repo/object-pool/README.md)
- [github-repo/object-pool/src/main/java/com/iluwatar/object/pool/ObjectPool.java](../../github-repo/object-pool/src/main/java/com/iluwatar/object/pool/ObjectPool.java)
- [github-repo/object-pool/src/main/java/com/iluwatar/object/pool/OliphauntPool.java](../../github-repo/object-pool/src/main/java/com/iluwatar/object/pool/OliphauntPool.java)
- [github-repo/object-pool/src/main/java/com/iluwatar/object/pool/Oliphaunt.java](../../github-repo/object-pool/src/main/java/com/iluwatar/object/pool/Oliphaunt.java)
- [github-repo/object-pool/src/main/java/com/iluwatar/object/pool/App.java](../../github-repo/object-pool/src/main/java/com/iluwatar/object/pool/App.java)
