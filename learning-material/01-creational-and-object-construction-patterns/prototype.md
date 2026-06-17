# Prototype Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: High  
Software usage meter: Medium  
Repository module: [github-repo/prototype](../../github-repo/prototype)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why copying an existing object can be better than constructing from scratch.
2. Second pass: rewrite the Java example and explain shallow copy vs deep copy.
3. Third pass: compare Prototype with Factory, Builder, and Object Pool.

By the end, you should be able to say:

> Prototype creates new objects by copying a prepared example object, which is useful when construction is expensive or object setup is complex.

## 1. Technical Definition

Prototype is a creational design pattern that creates new objects by cloning existing prototype instances instead of directly constructing new objects from classes.

Core idea:

- Keep a prepared object as a prototype.
- Copy the prototype when a new object is needed.
- Customize the copy if needed.
- Avoid repeating expensive setup logic.

### 30-Second Interview Answer

I would use Prototype when new objects are variations of already-configured objects, or when constructing them from scratch is expensive. The system keeps prototype instances and creates new objects by copying them. The main trade-off is copy correctness: shallow copies can accidentally share mutable state, while deep copies are more expensive and harder to implement.

## 2. Layman and Easy to Understand Definition

Prototype is like using a document template.

You do not create every invoice from a blank page. You start from a prepared invoice template, copy it, and change customer-specific fields.

In code:

- The template is the prototype.
- The copy is the new object.
- Custom changes are applied after copying.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose creating a report configuration requires many expensive setup steps.

```java
ReportConfig config = new ReportConfig();
config.loadFonts();
config.loadDefaultSections();
config.loadChartThemes();
config.validateRules();
```

Problems:

- Setup logic is repeated.
- Object creation may be slow.
- Client code must know too many construction details.
- Creating many similar objects becomes noisy.

### 3.2 The Prototype Solution

Prepare one object once:

```java
ReportConfig defaultConfig = ReportConfig.defaultPrototype();
```

Then copy it:

```java
ReportConfig customerConfig = defaultConfig.copy();
customerConfig.setTitle("Customer Report");
```

The client starts from a known-good object and only changes what is different.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Prototype | Object that knows how to copy itself. |
| Concrete prototype | Specific object implementation being copied. |
| Client | Asks for copies instead of constructing from scratch. |
| Prototype registry | Optional map of named prototypes. |
| Copy method | Method such as `copy()`, `clone()`, or `deepCopy()`. |

### 3.4 Shallow Copy vs Deep Copy

| Copy type | Meaning | Risk |
|---|---|---|
| Shallow copy | Copies object fields but shares nested mutable objects. | Mutating a nested list in one copy may affect another. |
| Deep copy | Copies the object and its nested mutable objects. | More code, more cost, harder with cycles. |

Interview trap:

> Prototype is not just "use clone". The hard part is deciding what state should be shared and what state must be copied.

### 3.5 Mental Model

Think of Prototype as copy-first construction.

1. Build a valid sample object.
2. Store it as a prototype.
3. When needed, call `copy()`.
4. Make small changes to the copy.
5. Ensure mutable internal state is not accidentally shared.

## 4. Java Coding Example

This example copies a prepared notification template.

```java
import java.util.ArrayList;
import java.util.List;

public final class NotificationTemplate {
    private String subject;
    private String body;
    private final List<String> defaultRecipients;

    public NotificationTemplate(String subject, String body, List<String> defaultRecipients) {
        this.subject = subject;
        this.body = body;
        this.defaultRecipients = new ArrayList<>(defaultRecipients);
    }

    private NotificationTemplate(NotificationTemplate source) {
        this.subject = source.subject;
        this.body = source.body;
        this.defaultRecipients = new ArrayList<>(source.defaultRecipients);
    }

    public NotificationTemplate copy() {
        return new NotificationTemplate(this);
    }

    public void subject(String subject) {
        this.subject = subject;
    }

    public void addRecipient(String recipient) {
        this.defaultRecipients.add(recipient);
    }

    @Override
    public String toString() {
        return "NotificationTemplate{" +
            "subject='" + subject + '\'' +
            ", body='" + body + '\'' +
            ", defaultRecipients=" + defaultRecipients +
            '}';
    }
}
```

### Java Block by Block Explanation

#### Prototype Class

```java
public final class NotificationTemplate {
```

The class acts as both the concrete object and the prototype.

#### Defensive Constructor Copy

```java
this.defaultRecipients = new ArrayList<>(defaultRecipients);
```

The constructor avoids storing the caller's mutable list directly.

#### Copy Constructor

```java
private NotificationTemplate(NotificationTemplate source) {
```

The copy constructor defines exactly how object state should be copied.

#### Deep Enough Copy

```java
this.defaultRecipients = new ArrayList<>(source.defaultRecipients);
```

The list itself is copied so each template gets its own recipient list.

#### Copy Method

```java
public NotificationTemplate copy() {
    return new NotificationTemplate(this);
}
```

The client uses this method instead of knowing all construction details.

### Java Usage

```java
import java.util.List;

public class Demo {
    public static void main(String[] args) {
        NotificationTemplate welcomePrototype = new NotificationTemplate(
            "Welcome",
            "Thanks for joining our product.",
            List.of("audit@app.com")
        );

        NotificationTemplate customerMessage = welcomePrototype.copy();
        customerMessage.subject("Welcome, Aravind");
        customerMessage.addRecipient("aravind@example.com");

        System.out.println(welcomePrototype);
        System.out.println(customerMessage);
    }
}
```

The prototype remains reusable, while the copy can be customized.

## 5. Python Coding Example

Python has a standard `copy` module.

```python
from copy import deepcopy
from dataclasses import dataclass, field


@dataclass
class NotificationTemplate:
    subject: str
    body: str
    default_recipients: list[str] = field(default_factory=list)

    def copy(self) -> "NotificationTemplate":
        return deepcopy(self)
```

### Python Usage

```python
prototype = NotificationTemplate(
    subject="Welcome",
    body="Thanks for joining our product.",
    default_recipients=["audit@app.com"],
)

customer_message = prototype.copy()
customer_message.subject = "Welcome, Aravind"
customer_message.default_recipients.append("aravind@example.com")

print(prototype)
print(customer_message)
```

`deepcopy()` prevents the two objects from sharing the same recipient list.

## 6. Where It Comes Handy in Real Life

Prototype is useful when objects are expensive or awkward to build repeatedly.

Examples:

- Document templates.
- Report configurations.
- Game object templates.
- UI component templates.
- Workflow definitions.
- Test data templates.
- Complex request payload examples.
- Simulation entities with default state.

## 7. Advantages Over Normal Code Without Pattern

### Without Prototype

```java
NotificationTemplate template = new NotificationTemplate(
    "Welcome",
    "Thanks for joining our product.",
    List.of("audit@app.com")
);
```

Problems:

- Setup details are repeated.
- Complex defaults are easy to forget.
- Construction may be expensive.
- Client code knows too much about initialization.

### With Prototype

```java
NotificationTemplate message = welcomePrototype.copy();
message.subject("Welcome, Aravind");
```

Benefits:

- Reuses a known-good baseline.
- Reduces repeated setup.
- Allows runtime-configured object creation.
- Keeps client code smaller.

## 8. Where It Excels

Prototype excels when:

- Creating from scratch is expensive.
- Objects have many default settings.
- New objects are variations of a few known examples.
- Concrete classes are chosen at runtime.
- You want to avoid a parallel factory hierarchy.
- You need fast setup of test fixtures.

## 9. Where It Fails

Prototype is a poor fit when:

- Objects are simple to construct.
- Copy semantics are unclear.
- Objects hold external resources like open sockets.
- Deep copy is too expensive.
- Shared mutable state would cause bugs.
- Identity matters and copied objects could be misleading.

Example where Prototype is overkill:

```java
record Money(String currency, long cents) {}
```

For a small immutable value, direct construction is clearer.

## 10. Prebuilt Libraries and Packages

### Java

Java options:

- `Object.clone()` exists but is often awkward.
- Copy constructors.
- Static copy factories.
- Serialization-based copy for special cases.
- Libraries such as MapStruct or model mappers for DTO copying.

Practical Java advice:

- Prefer copy constructors or explicit `copy()` methods over blindly exposing `clone()`.

### Python

Python options:

- `copy.copy()` for shallow copy.
- `copy.deepcopy()` for deep copy.
- `dataclasses.replace()` for immutable-ish dataclass changes.
- Pydantic model copy methods.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids repeated expensive setup. | Copy logic can be tricky. |
| Creates variants from known-good defaults. | Shallow copy can share mutable state accidentally. |
| Can reduce factory class explosion. | Deep copy may be expensive. |
| Useful for runtime-configured prototypes. | External resources may not copy safely. |
| Keeps client code concise. | Object identity can become confusing. |

## 12. Real-World Identification Example

Scenario:

You are building a report system with many report templates.

Each template has:

- Sections
- Chart themes
- Default filters
- Export settings

Should you use Prototype?

Yes, if most reports start from a small set of prepared templates and then customize a few fields.

Good usage:

```java
ReportTemplate monthly = registry.get("monthly-summary").copy();
monthly.setCustomerId(customerId);
monthly.setDateRange(dateRange);
```

This avoids rebuilding the whole report configuration from scratch each time.

## 13. MAANG Interview Triggers

Think Prototype when you hear:

- Clone existing object.
- Expensive object setup.
- Template object.
- Runtime-selected object type.
- Similar objects with small variations.
- Shallow copy vs deep copy.
- Avoid many factory subclasses.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the expensive or preconfigured object.
2. Explain why construction from scratch is costly.
3. Store a prototype instance.
4. Create new instances through `copy()`.
5. Clarify shallow vs deep copy rules.
6. Mention trade-off: copy correctness and mutable state.

## 14. Common Mistakes

### Mistake 1: Shallow Copying Mutable Fields

Bad:

```java
this.recipients = source.recipients;
```

Better:

```java
this.recipients = new ArrayList<>(source.recipients);
```

### Mistake 2: Copying External Resources

Open files, sockets, threads, and database connections usually should not be blindly cloned.

### Mistake 3: Using `clone()` Without a Clear Contract

Java `clone()` has historical design issues. Prefer an explicit `copy()` method when possible.

### Mistake 4: Forgetting Identity Fields

If the prototype has an `id`, the copy may need a new `id`.

### Mistake 5: Using Prototype for Simple Objects

If construction is already simple and readable, copying adds unnecessary complexity.

## 15. Prototype vs Similar Patterns

| Pattern | Difference |
|---|---|
| Factory | Creates objects from classes or constructors. Prototype creates objects by copying existing instances. |
| Abstract Factory | Creates families of products. Prototype can create products from stored examples. |
| Builder | Assembles one complex object step by step. Prototype starts from an existing object. |
| Object Pool | Reuses the same instances. Prototype creates separate copied instances. |
| Singleton | Ensures one instance. Prototype creates many copies from one example. |

## 16. Copy Strategy Checklist

Before implementing Prototype, decide:

| Question | Why it matters |
|---|---|
| Which fields are immutable? | Immutable state can often be shared safely. |
| Which fields are mutable? | Mutable state may need a deep copy. |
| Does the object have an identity? | Copies may need new IDs. |
| Does it hold external resources? | Some resources cannot be copied safely. |
| Is copying cheaper than construction? | If not, Prototype may not help. |

## 17. Quick Revision Notes

- Prototype creates objects by copying existing examples.
- Best for expensive or complex setup.
- Shallow copy shares nested mutable objects.
- Deep copy duplicates nested mutable objects.
- Prefer explicit copy methods in Java.
- Do not copy external resources blindly.

## 18. Mini Exercise

Design a Prototype setup for `EmailCampaignTemplate`.

Fields:

- `subject`
- `htmlBody`
- `segments`
- `trackingSettings`

Rules:

- Copies must not share mutable segment lists.
- Copies should get a new campaign id.
- The base prototype should remain unchanged.

Expected usage:

```java
EmailCampaignTemplate campaign = registry.get("welcome").copy();
campaign.setCampaignId(idGenerator.nextId());
campaign.addSegment("new-users");
```

## 19. Source Reference in This Repo

The repository's Prototype implementation uses a `HeroFactoryImpl` that creates new objects by copying stored prototype instances.

Useful files:

- [github-repo/prototype/README.md](../../github-repo/prototype/README.md)
- [github-repo/prototype/src/main/java/com/iluwatar/prototype/Prototype.java](../../github-repo/prototype/src/main/java/com/iluwatar/prototype/Prototype.java)
- [github-repo/prototype/src/main/java/com/iluwatar/prototype/HeroFactory.java](../../github-repo/prototype/src/main/java/com/iluwatar/prototype/HeroFactory.java)
- [github-repo/prototype/src/main/java/com/iluwatar/prototype/HeroFactoryImpl.java](../../github-repo/prototype/src/main/java/com/iluwatar/prototype/HeroFactoryImpl.java)
- [github-repo/prototype/src/main/java/com/iluwatar/prototype/App.java](../../github-repo/prototype/src/main/java/com/iluwatar/prototype/App.java)
