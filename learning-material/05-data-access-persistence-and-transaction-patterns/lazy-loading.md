# Lazy Loading Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/lazy-loading](../../github-repo/lazy-loading)

## How to Study This Page

Use this page in three passes:

1. First pass: understand delaying expensive loading until the value is actually needed.
2. Second pass: rewrite the Java example and identify holder, heavy object, first access, and cached access.
3. Third pass: study thread safety, latency spikes, ORM lazy loading, and N+1 query problems.

By the end, you should be able to say:

> Lazy Loading defers creating or fetching expensive data until first use, often caching it afterward.

## 1. Technical Definition

Lazy Loading is a performance and persistence pattern where an object, association, or resource is not initialized until it is accessed.

Core idea:

- Expensive object is represented by a holder/proxy.
- First access triggers loading.
- Loaded value is usually cached.
- Startup and memory usage improve.
- First-use latency and concurrency must be managed.

### 30-Second Interview Answer

I would use Lazy Loading when an object or related data is expensive to create and may not be needed for every request. A holder, proxy, or ORM association loads the value on first access and caches it. It improves startup time and memory usage, but it can create latency spikes, thread-safety issues, and N+1 query problems if used carelessly.

## 2. Layman and Easy to Understand Definition

Lazy Loading is like opening a file only when someone actually asks to read it.

You do not open every file at application startup. You wait until the file is needed, then load it and keep it ready for next time.

In code:

- Holder starts empty.
- First call creates/loads value.
- Later calls reuse value.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Eager loading creates everything upfront:

```java
Profile profile = loadProfile(userId);
List<Order> orders = loadOrders(userId);
List<Invoice> invoices = loadInvoices(userId);
```

Problems:

- Slow startup/request time.
- Memory wasted on unused data.
- Heavy objects created unnecessarily.
- Related data fetched even when not displayed.

### 3.2 The Lazy Loading Solution

Delay loading:

```java
User user = userRepository.findById(userId);
List<Order> orders = user.getOrders(); // loads only now
```

The value is fetched when accessed.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Holder/proxy | Object that delays loading. |
| Heavy/resource object | Expensive data or object. |
| Loader | Function/code that creates or fetches value. |
| Cache field | Stores loaded value after first access. |
| Client | Accesses value without knowing loading mechanics. |

### 3.4 Loading Styles

| Style | Meaning |
|---|---|
| Lazy initialization | Create object on first method call. |
| Virtual proxy | Proxy loads real object on demand. |
| ORM lazy association | Related entity/collection loaded when accessed. |
| Lazy supplier | Function computes and caches value. |

## 4. Java Coding Example

This example lazily loads a customer profile.

```java
import java.util.function.Supplier;

final class CustomerProfile {
    private final String customerId;

    CustomerProfile(String customerId) {
        this.customerId = customerId;
        System.out.println("Loaded profile for " + customerId);
    }

    String customerId() {
        return customerId;
    }
}

final class LazyValue<T> {
    private Supplier<T> supplier;
    private T value;

    LazyValue(Supplier<T> supplier) {
        this.supplier = supplier;
    }

    synchronized T get() {
        if (value == null) {
            value = supplier.get();
            supplier = null;
        }
        return value;
    }
}

final class Customer {
    private final String id;
    private final LazyValue<CustomerProfile> profile;

    Customer(String id) {
        this.id = id;
        this.profile = new LazyValue<>(() -> new CustomerProfile(id));
    }

    CustomerProfile profile() {
        return profile.get();
    }
}
```

### Java Block by Block Explanation

`CustomerProfile` represents the expensive object.

`LazyValue` stores a supplier and creates the value only on first `get`.

`Customer` exposes `profile()` normally while hiding lazy-loading mechanics.

The `synchronized` block prevents two threads from loading the same value at the same time.

### Java Usage

Use Lazy Loading in Java when:

- Object creation is expensive.
- Data may not be needed.
- Startup time matters.
- ORM relationships are large.
- Loading can safely happen later.

## 5. Python Coding Example

```python
class CustomerProfile:
    def __init__(self, customer_id):
        self.customer_id = customer_id
        print(f"Loaded profile for {customer_id}")


class Customer:
    def __init__(self, customer_id):
        self.id = customer_id
        self._profile = None

    @property
    def profile(self):
        if self._profile is None:
            self._profile = CustomerProfile(self.id)
        return self._profile


customer = Customer("c-101")
print("Profile not loaded yet")
print(customer.profile.customer_id)
```

### Python Usage

Python lazy loading often uses:

- `@property`
- Cached properties.
- Lazy import patterns.
- ORM lazy relationships.
- Generator-based data loading.

## 6. Where It Comes Handy in Real Life

- ORM relationships.
- User profile details.
- Large images/documents.
- Configuration sections.
- Expensive service clients.
- Caches.
- UI tabs loaded on demand.
- Startup optimization.

## 7. Advantages Over Normal Code Without Pattern

### Without Lazy Loading

```java
Customer customer = new Customer(id, loadProfile(id), loadOrders(id), loadInvoices(id));
```

Problems:

- Loads everything upfront.
- Wastes memory and time.
- Startup/request can be slow.

### With Lazy Loading

```java
Customer customer = new Customer(id);
customer.profile(); // loads only if needed
```

Benefits:

- Faster initial load.
- Lower memory usage.
- Expensive work happens only when needed.
- Can improve perceived performance.

## 8. Where It Excels

- Expensive optional data.
- Large object graphs.
- ORM associations.
- Feature panels rarely opened.
- Applications sensitive to startup time.
- Resources that can be safely loaded later.

## 9. Where It Fails

- Data is always needed anyway.
- First-use latency is unacceptable.
- Lazy access happens inside tight loops.
- ORM lazy loading causes N+1 queries.
- Thread safety is ignored.
- Loading can fail at surprising times.

## 10. Prebuilt Frameworks and Packages

### Java

- JPA/Hibernate `FetchType.LAZY`.
- Spring lazy beans.
- Guava `Suppliers.memoize`.
- Java `Supplier`.
- Virtual Proxy pattern.

### Python

- `functools.cached_property`.
- SQLAlchemy lazy relationships.
- Django queryset lazy evaluation.
- Lazy imports.
- Generators.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces startup work. | First access can be slow. |
| Saves memory. | Can hide expensive operations. |
| Avoids unused data loading. | Thread safety can be tricky. |
| Works well with proxies/ORMs. | N+1 query risk. |
| Improves perceived performance. | Errors occur later than expected. |

## 12. Real-World Identification Example

Scenario:

You load a product page, but reviews are below the fold.

Lazy Loading fit:

- Product details load immediately.
- Reviews load when section is opened/scrolled.
- Images load when visible.

What would go wrong without it:

- Initial page loads slowly.
- Bandwidth is wasted on unseen data.

## 13. MAANG Interview Triggers

Use Lazy Loading when you hear:

- "Load only when needed."
- "Expensive object creation."
- "Improve startup time."
- "ORM relationship loading."
- "Avoid loading entire object graph."
- "Virtual proxy."
- "N+1 queries."

### Interview-Ready Answer Format

1. Identify expensive or optional data.
2. Wrap it in holder/proxy/supplier.
3. Load on first access.
4. Cache after loading if appropriate.
5. Handle thread safety and failure.
6. Mention N+1 and first-access latency trade-offs.

## 14. Common Mistakes

### Mistake 1: Lazy Loading Everything

If data is always needed, eager loading may be simpler and faster.

### Mistake 2: Ignoring N+1 Queries

Lazy ORM collections in loops can generate many database calls.

### Mistake 3: No Thread Safety

Concurrent first access can create duplicates or corrupt state.

### Mistake 4: Surprise Failure Point

Loading can fail later, far away from original object creation.

### Mistake 5: Hidden Network Calls

Make expensive lazy boundaries visible in code reviews and performance tests.

## 15. Lazy Loading vs Similar Patterns

| Pattern | Difference |
|---|---|
| Lazy Loading | Defers creation/fetch until first use. |
| Eager Loading | Loads everything upfront. |
| Virtual Proxy | Object proxy that lazy-loads real object. |
| Cache-Aside | Application explicitly loads into cache on miss. |
| Singleton | Can be lazily initialized, but solves single-instance ownership. |

## 16. Lazy Loading Design Checklist

| Question | Why it matters |
|---|---|
| Is the object expensive? | Justifies laziness. |
| Is it often unused? | Determines benefit. |
| Should value be cached? | Avoids repeated load. |
| Is access concurrent? | Requires synchronization. |
| Can loading fail? | Needs error handling. |
| Could this cause N+1? | Protects performance. |

## 17. Quick Revision Notes

- Lazy Loading defers work until first access.
- It often caches after loading.
- Great for expensive optional data.
- Watch first-access latency.
- ORM lazy loading can cause N+1 queries.
- Thread safety matters.

## 18. Mini Exercise

Design Lazy Loading for `UserAccount`.

Lazy fields:

- `profile`
- `orderHistory`
- `savedPaymentMethods`

Questions:

- Which should be cached?
- Which might need eager loading?
- What happens if loading fails?

## 19. Source Reference in This Repo

The repository's Lazy Loading implementation compares naive, synchronized, and Java 8 supplier-based lazy holders.

Useful files:

- [github-repo/lazy-loading/README.md](../../github-repo/lazy-loading/README.md)
- [github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/Heavy.java](../../github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/Heavy.java)
- [github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/HolderNaive.java](../../github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/HolderNaive.java)
- [github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/HolderThreadSafe.java](../../github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/HolderThreadSafe.java)
- [github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/Java8Holder.java](../../github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/Java8Holder.java)
- [github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/App.java](../../github-repo/lazy-loading/src/main/java/com/iluwatar/lazy/loading/App.java)

