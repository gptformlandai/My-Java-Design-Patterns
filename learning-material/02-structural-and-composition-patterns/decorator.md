# Decorator Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/decorator](../../github-repo/decorator)

## How to Study This Page

Use this page in three passes:

1. First pass: understand wrapping an object to add behavior without changing its class.
2. Second pass: rewrite the Java example and identify component, concrete component, decorator, and concrete decorator.
3. Third pass: compare Decorator with Proxy, Adapter, and inheritance.

By the end, you should be able to say:

> Decorator adds responsibilities to individual objects dynamically while preserving the same interface.

## 1. Technical Definition

Decorator is a structural design pattern that attaches additional behavior to an object by wrapping it in another object that implements the same interface.

Core idea:

- Define a component interface.
- Concrete components implement base behavior.
- Decorators implement the same interface.
- Each decorator holds another component and delegates to it.
- Decorators add behavior before, after, or around delegation.

### 30-Second Interview Answer

I would use Decorator when I need optional, stackable behavior for individual objects without creating many subclasses. Each decorator implements the same interface as the component and wraps another component. This is useful for streams, middleware, compression, encryption, logging, or validation. The trade-off is many small wrapper objects and harder debugging when chains get deep.

## 2. Layman and Easy to Understand Definition

Decorator is like customizing a basic coffee order.

You start with plain coffee. Then you add milk, then sugar, then whipped cream. Each add-on wraps the previous drink and changes the final result.

In code:

- Plain coffee is the base component.
- Each add-on is a decorator.
- The final object still behaves like a drink.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a data writer can optionally add compression, encryption, and logging.

Inheritance can explode:

```java
CompressedWriter
EncryptedWriter
LoggingWriter
CompressedEncryptedWriter
LoggingCompressedEncryptedWriter
```

Problems:

- Too many combinations.
- Optional behavior is hard to mix.
- Adding one feature creates many subclasses.
- Runtime composition is awkward.

### 3.2 The Decorator Solution

Wrap behavior dynamically:

```java
DataWriter writer =
    new LoggingWriter(
        new EncryptingWriter(
            new FileDataWriter()
        )
    );
```

Each wrapper implements `DataWriter`, so the client still sees one interface.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Component | Common interface for base object and decorators. |
| Concrete component | Base object with core behavior. |
| Decorator | Wrapper that implements component and holds another component. |
| Concrete decorator | Adds one responsibility. |
| Client | Uses the component interface. |

### 3.4 Mental Model

Think of Decorator as behavior layers.

1. Start with a base component.
2. Wrap it with one decorator.
3. Optionally wrap it with more decorators.
4. Client calls the outermost object.
5. Each layer adds behavior and delegates inward.

## 4. Java Coding Example

This example decorates a data writer with logging and encryption.

```java
public interface DataWriter {
    void write(String data);
}

public final class FileDataWriter implements DataWriter {
    @Override
    public void write(String data) {
        System.out.println("Writing to file: " + data);
    }
}

public abstract class DataWriterDecorator implements DataWriter {
    protected final DataWriter delegate;

    protected DataWriterDecorator(DataWriter delegate) {
        this.delegate = delegate;
    }
}

public final class LoggingWriter extends DataWriterDecorator {
    public LoggingWriter(DataWriter delegate) {
        super(delegate);
    }

    @Override
    public void write(String data) {
        System.out.println("Log: writing " + data.length() + " chars");
        delegate.write(data);
    }
}

public final class EncryptingWriter extends DataWriterDecorator {
    public EncryptingWriter(DataWriter delegate) {
        super(delegate);
    }

    @Override
    public void write(String data) {
        String encrypted = "encrypted(" + data + ")";
        delegate.write(encrypted);
    }
}
```

### Java Block by Block Explanation

#### Component

```java
public interface DataWriter {
    void write(String data);
}
```

The client depends on this interface.

#### Concrete Component

```java
public final class FileDataWriter implements DataWriter {
```

This is the base behavior.

#### Base Decorator

```java
public abstract class DataWriterDecorator implements DataWriter {
```

The base decorator stores the wrapped component.

#### Concrete Decorator

```java
public final class LoggingWriter extends DataWriterDecorator {
```

This decorator adds logging before delegating.

#### Delegation

```java
delegate.write(data);
```

The decorator forwards the call to the wrapped object.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        DataWriter writer = new LoggingWriter(
            new EncryptingWriter(
                new FileDataWriter()
            )
        );

        writer.write("hello");
    }
}
```

The outer object is still a `DataWriter`.

## 5. Python Coding Example

```python
from typing import Protocol


class DataWriter(Protocol):
    def write(self, data: str) -> None:
        ...


class FileDataWriter:
    def write(self, data: str) -> None:
        print(f"Writing to file: {data}")


class LoggingWriter:
    def __init__(self, delegate: DataWriter) -> None:
        self._delegate = delegate

    def write(self, data: str) -> None:
        print(f"Log: writing {len(data)} chars")
        self._delegate.write(data)


class EncryptingWriter:
    def __init__(self, delegate: DataWriter) -> None:
        self._delegate = delegate

    def write(self, data: str) -> None:
        self._delegate.write(f"encrypted({data})")
```

### Python Usage

```python
writer: DataWriter = LoggingWriter(EncryptingWriter(FileDataWriter()))
writer.write("hello")
```

## 6. Where It Comes Handy in Real Life

Decorator is useful for optional, composable behavior.

Examples:

- Java IO streams.
- HTTP middleware.
- Compression and encryption wrappers.
- Logging wrappers.
- Validation layers.
- Retry wrappers.
- UI borders, scrollbars, and visual effects.

## 7. Advantages Over Normal Code Without Pattern

### Without Decorator

```java
DataWriter writer = new LoggingEncryptedFileWriter();
```

Problems:

- Every combination needs a new class.
- Features are hard to reorder.
- Features are hard to add at runtime.
- Base classes become bloated.

### With Decorator

```java
DataWriter writer = new LoggingWriter(new EncryptingWriter(new FileDataWriter()));
```

Benefits:

- Features are stackable.
- Behavior can be added per object.
- Class explosion is reduced.
- Open/closed principle is supported.

## 8. Where It Excels

Decorator excels when:

- Behavior is optional.
- Features can be combined in many ways.
- You need runtime composition.
- You want to avoid subclass explosion.
- The wrapper can preserve the same interface.

## 9. Where It Fails

Decorator is a poor fit when:

- The added behavior changes the interface.
- The chain becomes too deep to reason about.
- Order of decorators is subtle and fragile.
- A single configuration object would be clearer.
- The behavior is core, not optional.

## 10. Prebuilt Libraries and Packages

### Java

Decorator-style APIs:

- `InputStream`, `BufferedInputStream`, `GZIPInputStream`.
- `OutputStream` wrappers.
- `Reader` and `Writer` wrappers.
- `Collections.unmodifiableList()`.
- `Collections.synchronizedList()`.
- Servlet filters and HTTP middleware styles.

### Python

Python examples:

- Function decorators.
- Wrapper classes.
- Context managers.
- Middleware chains.
- File-like object wrappers.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Adds behavior without modifying classes. | Many small objects. |
| Avoids subclass explosion. | Order of wrappers can matter. |
| Supports runtime composition. | Debugging chains can be harder. |
| Keeps features focused. | Object identity/type checks get tricky. |
| Preserves component interface. | Overuse can reduce readability. |

## 12. Real-World Identification Example

Scenario:

You are designing an HTTP client.

Optional behaviors:

- Logging
- Retry
- Metrics
- Authentication
- Compression

Should you use Decorator?

Yes, if each behavior can wrap a common `HttpClient` interface.

Good usage:

```java
HttpClient client =
    new MetricsClient(new RetryClient(new AuthClient(new RealHttpClient())));
```

## 13. MAANG Interview Triggers

Think Decorator when you hear:

- Add behavior dynamically.
- Stack optional features.
- Same interface as wrapped object.
- Avoid subclass explosion.
- Middleware chain.
- Java IO streams.
- Wrap one object with another.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the component interface.
2. Implement the base component.
3. Create decorators that implement the same interface.
4. Store a wrapped component inside each decorator.
5. Add behavior before/after delegation.
6. Mention trade-off: wrapper chains and ordering complexity.

## 14. Common Mistakes

### Mistake 1: Changing the Interface

If the wrapper exposes a different interface, it is Adapter, not Decorator.

### Mistake 2: Deep Chains Without Naming

Very long nested constructors are hard to read. Use factories or configuration when needed.

### Mistake 3: Hidden Ordering Bugs

Compression before encryption may differ from encryption before compression. Order must be intentional.

### Mistake 4: Decorating Everything

Do not use Decorator when one normal class is clearer.

### Mistake 5: Forgetting Delegation

A decorator usually should call the wrapped component unless it intentionally blocks or replaces behavior.

## 15. Decorator vs Similar Patterns

| Pattern | Difference |
|---|---|
| Adapter | Adapter changes interface. Decorator keeps interface and adds behavior. |
| Proxy | Proxy controls access. Decorator adds responsibilities. |
| Composite | Composite groups many children. Decorator wraps one component. |
| Strategy | Strategy changes internal algorithm. Decorator adds an outer layer. |
| Facade | Facade simplifies a subsystem. Decorator extends one object's behavior. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Component | `DataWriter` | Shared interface. |
| Concrete component | `FileDataWriter` | Base behavior. |
| Decorator | `DataWriterDecorator` | Holds wrapped component. |
| Concrete decorator | `LoggingWriter`, `EncryptingWriter` | Adds behavior. |

## 17. Quick Revision Notes

- Decorator wraps one object.
- It preserves the same interface.
- It adds optional behavior dynamically.
- Multiple decorators can be stacked.
- It avoids subclass explosion.
- Watch order and chain complexity.

## 18. Mini Exercise

Design decorators for `MessageSender`.

Base:

```java
interface MessageSender {
    void send(String message);
}
```

Decorators:

- `LoggingMessageSender`
- `RetryingMessageSender`
- `EncryptingMessageSender`

Expected usage:

```java
MessageSender sender =
    new LoggingMessageSender(new RetryingMessageSender(new RealMessageSender()));
```

## 19. Source Reference in This Repo

The repository's Decorator implementation wraps a base `Troll` component with `ClubbedTroll`.

Useful files:

- [github-repo/decorator/README.md](../../github-repo/decorator/README.md)
- [github-repo/decorator/src/main/java/com/iluwatar/decorator/Troll.java](../../github-repo/decorator/src/main/java/com/iluwatar/decorator/Troll.java)
- [github-repo/decorator/src/main/java/com/iluwatar/decorator/SimpleTroll.java](../../github-repo/decorator/src/main/java/com/iluwatar/decorator/SimpleTroll.java)
- [github-repo/decorator/src/main/java/com/iluwatar/decorator/ClubbedTroll.java](../../github-repo/decorator/src/main/java/com/iluwatar/decorator/ClubbedTroll.java)
- [github-repo/decorator/src/main/java/com/iluwatar/decorator/App.java](../../github-repo/decorator/src/main/java/com/iluwatar/decorator/App.java)
