# Singleton Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/singleton](../../github-repo/singleton)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the difference between "one instance" and "global mutable state".
2. Second pass: rewrite the enum singleton and initialization-on-demand holder examples.
3. Third pass: study thread safety, testing problems, and why DI-managed singletons are often better in modern apps.

By the end, you should be able to say:

> Singleton ensures one instance and a global access point, but it must be used carefully because global state can make systems harder to test and evolve.

## 1. Technical Definition

Singleton is a creational design pattern that restricts a class to a single instance and provides a controlled access point to that instance.

Core idea:

- Prevent arbitrary construction.
- Hold exactly one shared instance.
- Provide a controlled way to access it.
- Ensure safe initialization, especially in concurrent code.

### 30-Second Interview Answer

I would use Singleton only when the system truly needs exactly one shared instance, such as a process-wide runtime object or a stateless shared service. In Java, enum singleton is usually the safest simple implementation. I would avoid using Singleton as a shortcut for global mutable state because it hurts testing, lifecycle control, and dependency clarity.

## 2. Layman and Easy to Understand Definition

Singleton is like a company having one official payroll system.

Many teams may access payroll, but there should not be five independent payroll systems calculating salaries differently.

In code:

- The class controls its own instance.
- Everyone uses the same instance.
- The system prevents accidental duplicate construction.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Some objects represent process-wide coordination or expensive shared resources.

Examples:

- Runtime environment object.
- Application configuration snapshot.
- Registry of known plugins.
- Shared stateless utility service.

If many copies are created accidentally, the system may waste resources or behave inconsistently.

Bad design:

```java
Configuration config1 = new Configuration();
Configuration config2 = new Configuration();
```

If both objects load different state, the application may become unpredictable.

### 3.2 The Singleton Solution

Make construction private and expose one instance:

```java
public enum ApplicationConfig {
    INSTANCE
}
```

Now code accesses:

```java
ApplicationConfig config = ApplicationConfig.INSTANCE;
```

The enum type guarantees a single instance per JVM class loader.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Singleton class | Class that controls instance creation. |
| Private constructor | Prevents normal external construction. |
| Static instance | Stores the single instance. |
| Access method or enum value | Provides global access. |
| Client | Code that uses the shared instance. |

### 3.4 Mental Model

Think of Singleton as a controlled single entry point.

1. The class prevents normal callers from using `new`.
2. The class creates or exposes one instance.
3. All callers use that same instance.
4. Thread safety must be considered during initialization.
5. Mutable state must be treated with caution.

## 4. Java Coding Example

### Recommended Simple Java Version: Enum Singleton

```java
public enum AuditLogger {
    INSTANCE;

    public void log(String message) {
        System.out.println("[AUDIT] " + message);
    }
}
```

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        AuditLogger logger1 = AuditLogger.INSTANCE;
        AuditLogger logger2 = AuditLogger.INSTANCE;

        logger1.log("User logged in");

        System.out.println(logger1 == logger2);
    }
}
```

Output:

```text
[AUDIT] User logged in
true
```

### Java Block by Block Explanation

#### Enum Instance

```java
public enum AuditLogger {
    INSTANCE;
}
```

Java guarantees one enum constant instance per JVM class loader. This also handles serialization and reflection attacks better than many hand-written singleton implementations.

#### Instance Method

```java
public void log(String message) {
```

The singleton can expose behavior like a normal object.

#### Identity Check

```java
System.out.println(logger1 == logger2);
```

This prints `true` because both variables point to the same instance.

### Alternative Java Version: Initialization-on-Demand Holder

This version is useful when you need a normal class instead of an enum.

```java
public final class MetricsRegistry {
    private MetricsRegistry() {
    }

    private static final class Holder {
        private static final MetricsRegistry INSTANCE = new MetricsRegistry();
    }

    public static MetricsRegistry getInstance() {
        return Holder.INSTANCE;
    }

    public void record(String metricName) {
        System.out.println("Recorded metric: " + metricName);
    }
}
```

Why this works:

- The outer class loads without creating the instance.
- The holder class loads only when `getInstance()` is called.
- Java class loading makes initialization thread-safe.

## 5. Python Coding Example

Python often avoids formal singletons by using module-level objects.

```python
class AuditLogger:
    def log(self, message: str) -> None:
        print(f"[AUDIT] {message}")


audit_logger = AuditLogger()
```

Usage:

```python
from audit import audit_logger


audit_logger.log("User logged in")
```

If you need controlled creation, you can use a simple class method:

```python
class MetricsRegistry:
    _instance: "MetricsRegistry | None" = None

    def __new__(cls) -> "MetricsRegistry":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def record(self, metric_name: str) -> None:
        print(f"Recorded metric: {metric_name}")
```

Important Python note:

- Prefer module-level objects or dependency injection unless the singleton behavior is truly needed.

## 6. Where It Comes Handy in Real Life

Singleton can be useful for carefully controlled shared objects.

Examples:

- Runtime environment access.
- Application-wide immutable configuration.
- Metrics registry.
- Plugin registry.
- Stateless shared service.
- JVM-level resource manager.
- Logger facade in small applications.

Modern applications often let a DI container manage singleton-scoped services instead of manually coding the Singleton pattern.

## 7. Advantages Over Normal Code Without Pattern

### Without Singleton

```java
MetricsRegistry registry1 = new MetricsRegistry();
MetricsRegistry registry2 = new MetricsRegistry();
```

Problems:

- Multiple registries may produce inconsistent metrics.
- Expensive initialization may repeat.
- Shared state may split across objects.
- Callers may not know which instance is authoritative.

### With Singleton

```java
MetricsRegistry registry = MetricsRegistry.getInstance();
```

Benefits:

- Controlled single instance.
- Consistent access point.
- Lazy initialization is possible.
- Resource use can be limited.

## 8. Where It Excels

Singleton excels when:

- Exactly one instance is a real domain or runtime constraint.
- The object is stateless or mostly immutable.
- The object is expensive and should be shared.
- The lifecycle is process-wide.
- You need a known access point and cannot use DI.

## 9. Where It Fails

Singleton is a poor fit when:

- You use it only to avoid passing dependencies.
- The singleton stores mutable business state.
- Tests need different instances or clean state.
- Different tenants, users, requests, or environments need different state.
- A DI container can manage lifecycle more cleanly.
- You need multiple instances in the future.

Example where Singleton is harmful:

```java
public final class CurrentUser {
    public static User user;
}
```

This is global mutable request state. It can leak data across threads or requests.

## 10. Prebuilt Libraries and Packages

### Java

Common singleton-related tools and APIs:

- Java enum singleton.
- Initialization-on-demand holder idiom.
- Spring singleton bean scope.
- Guice singleton scope.
- `Runtime.getRuntime()`.
- Logger factories that cache logger instances.

Avoid older unsafe patterns:

- Unsynchronized lazy initialization.
- Double-checked locking without `volatile`.
- Mutable public static fields.

### Python

Python alternatives:

- Module-level objects.
- `functools.lru_cache` for cached factories.
- Dependency injection through function or constructor parameters.
- Application context objects.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Ensures one controlled instance. | Can become hidden global state. |
| Provides a known access point. | Harder to unit test if stateful. |
| Can reduce repeated expensive setup. | Lifecycle can be hard to control. |
| Useful for process-wide coordination. | Can hide dependencies from constructors. |
| Enum version is simple in Java. | Incorrect lazy versions can be thread-unsafe. |

## 12. Real-World Identification Example

Scenario:

You are designing an application metrics registry.

Requirements:

- All services record metrics into one registry.
- Registry is initialized once during application startup.
- Registry exposes thread-safe recording operations.
- Tests should be able to isolate or replace metrics behavior.

Should you use Singleton?

Maybe.

Good answer:

- If this is a small app without DI, a singleton registry can be acceptable.
- In a Spring or Guice application, prefer a singleton-scoped bean injected into services.
- Avoid storing request-specific or user-specific state in the singleton.

Good DI-managed usage:

```java
public class OrderService {
    private final MetricsRegistry metricsRegistry;

    public OrderService(MetricsRegistry metricsRegistry) {
        this.metricsRegistry = metricsRegistry;
    }
}
```

The registry may still be one instance, but dependencies remain visible and testable.

## 13. MAANG Interview Triggers

Think Singleton when you hear:

- Exactly one instance.
- Global access point.
- Shared process-wide resource.
- Lazy initialization.
- Thread-safe initialization.
- Enum singleton.
- Global state risk.
- DI-managed singleton scope.

### Interview-Ready Answer Format

Use this structure when answering:

1. Ask whether exactly one instance is truly required.
2. State whether the object is stateless, immutable, or mutable.
3. Choose an implementation: enum, holder idiom, or DI-managed singleton.
4. Discuss thread safety and initialization.
5. Mention testing and lifecycle trade-offs.
6. Warn against using Singleton for request-specific or mutable business state.

## 14. Common Mistakes

### Mistake 1: Treating Singleton as a Global Variable

Bad:

```java
public final class CartSingleton {
    public static final CartSingleton INSTANCE = new CartSingleton();
    private final List<Item> items = new ArrayList<>();
}
```

Shopping cart state is user-specific, not process-wide.

### Mistake 2: Non-Thread-Safe Lazy Initialization

Bad:

```java
public static Settings getInstance() {
    if (instance == null) {
        instance = new Settings();
    }
    return instance;
}
```

Two threads may create two instances. Use enum, holder idiom, synchronized access, or correct `volatile` double-check locking.

### Mistake 3: Hiding Dependencies

Bad:

```java
PaymentGateway gateway = PaymentGatewaySingleton.getInstance();
```

This hides the dependency from the constructor and makes testing harder.

### Mistake 4: Mutable Singleton Without Synchronization

If a singleton stores mutable shared state, every mutation must be thread-safe.

### Mistake 5: Singleton for Everything

Not every service needs to be globally accessible. Prefer normal objects and DI unless one instance is a real requirement.

## 15. Singleton vs Similar Patterns

| Pattern | Difference |
|---|---|
| Static utility class | Has only static methods and no object identity. Singleton is an object instance. |
| Dependency Injection | DI can provide one shared instance without global access. |
| Factory | Factory creates objects. Singleton restricts a class to one instance. |
| Monostate | Multiple instances share same static state. Singleton exposes one instance. |
| Object Pool | Manages multiple reusable instances. Singleton allows only one instance. |

## 16. Java Implementation Styles

| Style | Thread-safe | Notes |
|---|---|---|
| Enum singleton | Yes | Usually best simple Java choice. |
| Eager static instance | Yes | Simple, but creates instance whether needed or not. |
| Holder idiom | Yes | Lazy and clean for normal classes. |
| Synchronized accessor | Yes | Simple but may add synchronization overhead. |
| Double-check locking with `volatile` | Yes if correct | Easy to get wrong; use carefully. |
| Unsynchronized lazy singleton | No | Avoid in multi-threaded code. |

Prefer enum or holder idiom unless a framework manages singleton scope for you.

## 17. Quick Revision Notes

- Singleton means exactly one instance plus controlled access.
- In Java, enum singleton is usually safest.
- Holder idiom gives lazy initialization for normal classes.
- Do not use Singleton as a shortcut for global mutable state.
- DI-managed singletons are often better in modern applications.
- Always think about thread safety and testability.

## 18. Mini Exercise

Design a safe singleton or DI-managed singleton for `FeatureFlagRegistry`.

Requirements:

- Loads flags once at startup.
- Exposes `isEnabled(String flagName)`.
- Does not store request-specific state.
- Can be replaced in tests.

Question:

> Would you implement this as an enum singleton, holder singleton, or DI-managed singleton? Explain why.

## 19. Source Reference in This Repo

The repository's Singleton implementation shows several Java singleton styles, including enum singleton and double-check locking.

Useful files:

- [github-repo/singleton/README.md](../../github-repo/singleton/README.md)
- [github-repo/singleton/src/main/java/com/iluwatar/singleton/EnumIvoryTower.java](../../github-repo/singleton/src/main/java/com/iluwatar/singleton/EnumIvoryTower.java)
- [github-repo/singleton/src/main/java/com/iluwatar/singleton/InitializingOnDemandHolderIdiom.java](../../github-repo/singleton/src/main/java/com/iluwatar/singleton/InitializingOnDemandHolderIdiom.java)
- [github-repo/singleton/src/main/java/com/iluwatar/singleton/ThreadSafeDoubleCheckLocking.java](../../github-repo/singleton/src/main/java/com/iluwatar/singleton/ThreadSafeDoubleCheckLocking.java)
- [github-repo/singleton/src/main/java/com/iluwatar/singleton/App.java](../../github-repo/singleton/src/main/java/com/iluwatar/singleton/App.java)
