# Factory Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/factory](../../github-repo/factory)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why object creation logic should not be scattered across client code.
2. Second pass: rewrite the Java factory example and explain how the client avoids concrete classes.
3. Third pass: study the difference between Simple Factory, Factory Method, and Abstract Factory.

By the end, you should be able to say:

> Factory centralizes object creation so client code asks for a product by intent and receives an object through a common interface.

## 1. Technical Definition

Factory is a creational pattern where object creation is moved into a dedicated method or class. The client asks the factory for an object, and the factory decides which concrete class to instantiate.

Core idea:

- Hide `new ConcreteClass()` from client code.
- Return a common interface or parent type.
- Keep object-selection logic in one place.
- Make client code depend on product behavior, not product construction.

Important naming note:

- In many codebases, "Factory" means Simple Factory: one class or method creates different concrete products.
- "Factory Method" is the GoF pattern where subclasses override a creation method.
- "Abstract Factory" creates families of related products.

### 30-Second Interview Answer

I would use a Factory when client code needs one of several related implementations but should not know how to construct them. The factory accepts a type, configuration, or context and returns a product interface. This centralizes creation logic and reduces coupling, but it can become a large conditional block if too many product types are added without structure.

## 2. Layman and Easy to Understand Definition

Factory is like ordering coffee at a counter.

You say:

```text
I want a latte.
```

You do not build the drink yourself. The counter knows which cup, milk, beans, and process to use.

In code:

- You ask the factory for a product.
- The factory chooses the concrete class.
- You receive something you can use through a shared interface.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose an application sends notifications.

Without a factory, client code may do this:

```java
if (channel.equals("EMAIL")) {
    sender = new EmailNotificationSender();
} else if (channel.equals("SMS")) {
    sender = new SmsNotificationSender();
} else if (channel.equals("PUSH")) {
    sender = new PushNotificationSender();
}
```

Problems:

- Creation logic is repeated in many places.
- Client code knows every concrete class.
- Adding a new type requires touching many call sites.
- Tests and configuration become harder to manage.

### 3.2 The Factory Solution

Move creation into one factory:

```java
NotificationSender sender = NotificationSenderFactory.create(Channel.EMAIL);
sender.send("Welcome");
```

The client only knows:

- Which type it wants.
- The common interface it will use.

The factory knows:

- Which concrete class matches the requested type.
- How to create that object.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Product interface | Shared contract returned by the factory. |
| Concrete product | Specific implementation, such as email or SMS sender. |
| Factory | Class or method that creates products. |
| Client | Code that asks the factory for a product and uses it through the interface. |
| Type key | Enum, string, config value, or context that guides creation. |

### 3.4 Mental Model

Think of Factory as a controlled object counter.

1. Client says what kind of object it needs.
2. Factory checks the request.
3. Factory creates the right concrete object.
4. Factory returns it as an interface.
5. Client uses the object without knowing its construction details.

## 4. Java Coding Example

This example creates notification senders through a factory.

```java
import java.util.EnumMap;
import java.util.Map;
import java.util.function.Supplier;

public interface NotificationSender {
    void send(String message);
}

public final class EmailNotificationSender implements NotificationSender {
    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

public final class SmsNotificationSender implements NotificationSender {
    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

public enum Channel {
    EMAIL,
    SMS
}

public final class NotificationSenderFactory {
    private static final Map<Channel, Supplier<NotificationSender>> SENDERS =
        new EnumMap<>(Channel.class);

    static {
        SENDERS.put(Channel.EMAIL, EmailNotificationSender::new);
        SENDERS.put(Channel.SMS, SmsNotificationSender::new);
    }

    private NotificationSenderFactory() {
    }

    public static NotificationSender create(Channel channel) {
        Supplier<NotificationSender> supplier = SENDERS.get(channel);
        if (supplier == null) {
            throw new IllegalArgumentException("Unsupported channel: " + channel);
        }
        return supplier.get();
    }
}
```

### Java Block by Block Explanation

#### Product Interface

```java
public interface NotificationSender {
    void send(String message);
}
```

The client uses this interface and does not need to know the concrete sender class.

#### Concrete Products

```java
public final class EmailNotificationSender implements NotificationSender {
```

Each concrete product implements the same behavior in a different way.

#### Type Key

```java
public enum Channel {
    EMAIL,
    SMS
}
```

An enum is safer than raw strings because invalid values are harder to pass accidentally.

#### Factory Class

```java
public final class NotificationSenderFactory {
```

The factory centralizes object creation and hides concrete constructors from the client.

#### Creation Method

```java
public static NotificationSender create(Channel channel) {
```

The factory returns the interface type. That is the key design move.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        NotificationSender sender = NotificationSenderFactory.create(Channel.EMAIL);
        sender.send("Welcome to the app");
    }
}
```

The client does not call `new EmailNotificationSender()`.

## 5. Python Coding Example

Python factories are often functions or small classes.

```python
from enum import Enum
from typing import Protocol


class NotificationSender(Protocol):
    def send(self, message: str) -> None:
        ...


class EmailNotificationSender:
    def send(self, message: str) -> None:
        print(f"Email: {message}")


class SmsNotificationSender:
    def send(self, message: str) -> None:
        print(f"SMS: {message}")


class Channel(Enum):
    EMAIL = "email"
    SMS = "sms"


def create_notification_sender(channel: Channel) -> NotificationSender:
    factories = {
        Channel.EMAIL: EmailNotificationSender,
        Channel.SMS: SmsNotificationSender,
    }

    try:
        return factories[channel]()
    except KeyError as exc:
        raise ValueError(f"Unsupported channel: {channel}") from exc
```

### Python Usage

```python
sender = create_notification_sender(Channel.EMAIL)
sender.send("Welcome to the app")
```

## 6. Where It Comes Handy in Real Life

Factory is useful when selection and creation should live in one controlled place.

Examples:

- Creating notification senders.
- Creating payment provider clients.
- Creating file parsers by file type.
- Creating serializers by format.
- Creating database connectors by environment.
- Creating UI widgets by platform.
- Creating game objects by type.

## 7. Advantages Over Normal Code Without Pattern

### Without Factory

```java
NotificationSender sender = new EmailNotificationSender();
```

This looks simple, but repeated concrete construction spreads knowledge across the codebase.

Problems:

- Client code depends on concrete classes.
- Object creation rules are duplicated.
- Changes require many edits.
- Configuration and feature flags get scattered.

### With Factory

```java
NotificationSender sender = NotificationSenderFactory.create(Channel.EMAIL);
```

Benefits:

- Centralized creation.
- Less coupling to concrete classes.
- Easier to add validation around creation.
- Cleaner client code.
- Easier integration with configuration.

## 8. Where It Excels

Factory excels when:

- There are multiple concrete implementations.
- The caller should not know construction details.
- Object creation depends on config, enum, request type, or environment.
- You want to return an interface.
- You want one place to handle unsupported types.
- Construction has repeated setup logic.

## 9. Where It Fails

Factory is a poor fit when:

- There is only one obvious concrete class.
- The constructor is simple and stable.
- A factory just wraps `new` without adding clarity.
- The factory becomes a giant `switch` with too many responsibilities.
- The real need is dependency wiring, which may be better handled by DI.

Example where Factory is overkill:

```java
record UserId(String value) {}
```

For a tiny value object, direct construction is clearer.

## 10. Prebuilt Libraries and Packages

### Java

Common factory-like APIs:

- `Calendar.getInstance()`
- `ResourceBundle.getBundle()`
- `NumberFormat.getInstance()`
- `Charset.forName()`
- `EnumSet.of()`
- `DocumentBuilderFactory.newInstance()`
- `LoggerFactory.getLogger()`

Common implementation helpers:

- `Supplier<T>`
- `Map<Key, Supplier<Product>>`
- Dependency Injection containers
- Service Provider Interface mechanisms

### Python

Python factory tools are often simple language features:

- Factory functions.
- Class objects as callables.
- Dictionaries mapping keys to constructors.
- `@classmethod` alternate constructors.
- Entry points for plugin loading.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Centralizes object creation. | Adds another class or method. |
| Reduces concrete-class coupling. | Can become a large conditional block. |
| Makes product selection explicit. | Can hide construction complexity if poorly named. |
| Works well with interfaces. | May be overkill for simple objects. |
| Improves consistency across call sites. | Does not automatically solve dependency lifecycle. |

## 12. Real-World Identification Example

Scenario:

You are designing an import service that supports CSV, JSON, and XML files.

Products:

- `CsvParser`
- `JsonParser`
- `XmlParser`

Should you use Factory?

Yes.

Why:

- The caller should ask for a parser based on file type.
- All parsers can implement a `FileParser` interface.
- Unsupported file types can fail in one controlled place.
- Adding a parser should not require rewriting import workflows.

Good factory usage:

```java
FileParser parser = FileParserFactory.create(file.extension());
ImportResult result = parser.parse(file);
```

## 13. MAANG Interview Triggers

Think Factory when you hear:

- Choose implementation by type.
- Hide concrete classes.
- Centralize creation logic.
- Return interface.
- Avoid repeated `new`.
- Config-driven object creation.
- Parser by format.
- Client should not know implementation details.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the family of products.
2. Define the common interface.
3. Explain what input selects the product.
4. Put selection and construction inside a factory.
5. Return the interface to the caller.
6. Mention a trade-off: factory can grow too large if not kept focused.

## 14. Common Mistakes

### Mistake 1: Factory That Does Nothing

Bad:

```java
User user = UserFactory.create(name);
```

If `UserFactory.create(name)` only calls `new User(name)`, the factory may not be useful.

### Mistake 2: Returning Concrete Types

Bad:

```java
EmailNotificationSender sender = NotificationSenderFactory.createEmail();
```

Better:

```java
NotificationSender sender = NotificationSenderFactory.create(Channel.EMAIL);
```

### Mistake 3: Giant Switch Factory

A factory with hundreds of cases may need smaller factories, registration, or plugin loading.

### Mistake 4: Confusing Factory with Factory Method

Simple Factory usually has one factory class. Factory Method usually uses inheritance and lets subclasses decide creation.

### Mistake 5: Hiding Runtime Failures

If an unsupported type is passed, fail clearly with a useful error.

## 15. Factory vs Similar Patterns

| Pattern | Difference |
|---|---|
| Constructor | Directly creates one known class. Factory chooses among possible products. |
| Builder | Builds one complex object step by step. Factory chooses which object type to create. |
| Factory Method | Uses subclass overriding to decide product creation. Simple Factory centralizes selection in one class or method. |
| Abstract Factory | Creates families of related products. Factory often creates one product family member. |
| Dependency Injection | Provides dependencies to classes. Factory creates products on request. |

## 16. Factory Implementation Styles

| Style | Example | Best use |
|---|---|---|
| Static factory method | `CoinFactory.getCoin(type)` | Simple creation without state. |
| Factory object | `parserFactory.create(format)` | Creation needs configuration or dependencies. |
| Map-based factory | `Map<Type, Supplier<Product>>` | Avoiding long switch statements. |
| Registry factory | Register product creators at startup | Plugin-like systems. |
| DI-backed factory | Factory receives dependencies from container | Products need injected collaborators. |

Start simple. Move to registry or DI-backed factories only when the product list or construction rules justify it.

## 17. Quick Revision Notes

- Factory centralizes object creation.
- Client asks for intent, not a concrete constructor.
- Return the product interface when possible.
- Use enums or typed keys instead of raw strings.
- Avoid factories that only wrap simple constructors.
- Factory chooses type; Builder configures one complex object.

## 18. Mini Exercise

Design a factory for `ReportExporter`.

Supported formats:

- `PDF`
- `CSV`
- `JSON`

Rules:

- All exporters implement `ReportExporter`.
- Unsupported formats should throw a clear exception.
- Client code should not call `new PdfReportExporter()`.

Expected usage:

```java
ReportExporter exporter = ReportExporterFactory.create(ReportFormat.PDF);
exporter.export(report);
```

## 19. Source Reference in This Repo

The repository's Factory implementation creates `Coin` objects using `CoinFactory`.

Useful files:

- [github-repo/factory/README.md](../../github-repo/factory/README.md)
- [github-repo/factory/src/main/java/com/iluwatar/factory/CoinFactory.java](../../github-repo/factory/src/main/java/com/iluwatar/factory/CoinFactory.java)
- [github-repo/factory/src/main/java/com/iluwatar/factory/CoinType.java](../../github-repo/factory/src/main/java/com/iluwatar/factory/CoinType.java)
- [github-repo/factory/src/main/java/com/iluwatar/factory/Coin.java](../../github-repo/factory/src/main/java/com/iluwatar/factory/Coin.java)
