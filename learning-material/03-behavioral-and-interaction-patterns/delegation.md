# Delegation Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/delegation](../../github-repo/delegation)

## How to Study This Page

Use this page in three passes:

1. First pass: understand one object handing work to another object.
2. Second pass: rewrite the Java example and identify delegator, delegate, and shared interface.
3. Third pass: compare Delegation with inheritance, Strategy, Proxy, and Decorator.

By the end, you should be able to say:

> Delegation lets an object reuse or vary behavior by forwarding work to another object instead of inheriting it.

## 1. Technical Definition

Delegation is a design pattern where an object handles a request by passing all or part of the work to another helper object.

Core idea:

- The delegator receives the call.
- The delegate performs the real operation.
- The delegator can swap delegates.
- Composition replaces or reduces inheritance.

### 30-Second Interview Answer

I would use Delegation when a class should expose an operation but another object is better suited to perform it. The delegating object keeps the public API stable and forwards work to a helper through an interface. This gives composition-based reuse and runtime flexibility. The trade-off is an extra layer of indirection, and if overused, it can make the call path harder to follow.

## 2. Layman and Easy to Understand Definition

Delegation is like a manager assigning a task to a specialist.

The manager receives the request but does not do every task personally. The manager forwards the work to the right expert.

In code:

- The manager is the delegator.
- The specialist is the delegate.
- The task is the method call.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a class uses inheritance only to reuse behavior:

```java
class ReportPrinter extends PdfPrinter {
}
```

This creates tight coupling.

Problems appear when:

- Behavior must change at runtime.
- Multiple behaviors are needed.
- Subclass count grows.
- The class inherits methods it does not need.
- Testing requires replacing the behavior.

### 3.2 The Delegation Solution

Use composition:

```java
class ReportPrinter {
    private final Printer printer;

    void print(String report) {
        printer.print(report);
    }
}
```

The object owns a helper and forwards work to it.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Delegator | Object that receives the request. |
| Delegate | Helper object that performs the operation. |
| Interface | Contract shared by delegate implementations. |
| Client | Uses delegator without needing delegate details. |

### 3.4 Mental Model

Delegation is "has-a" reuse.

1. Client calls delegator.
2. Delegator decides whether to handle or forward.
3. Delegate performs specialized work.
4. Delegator may add validation, logging, or orchestration.

## 4. Java Coding Example

This example delegates notification delivery.

```java
public interface NotificationSender {
    void send(String userId, String message);
}

public final class EmailSender implements NotificationSender {
    @Override
    public void send(String userId, String message) {
        System.out.println("Email to " + userId + ": " + message);
    }
}

public final class SmsSender implements NotificationSender {
    @Override
    public void send(String userId, String message) {
        System.out.println("SMS to " + userId + ": " + message);
    }
}

public final class NotificationService {
    private final NotificationSender sender;

    public NotificationService(NotificationSender sender) {
        this.sender = sender;
    }

    public void notifyUser(String userId, String message) {
        if (message == null || message.isBlank()) {
            throw new IllegalArgumentException("message is required");
        }
        sender.send(userId, message);
    }
}

public final class DelegationDemo {
    public static void main(String[] args) {
        NotificationService service = new NotificationService(new EmailSender());
        service.notifyUser("u-101", "Your order shipped");
    }
}
```

### Java Block by Block Explanation

`NotificationSender` is the delegate interface.

`EmailSender` and `SmsSender` are interchangeable delegates.

`NotificationService` is the delegator. It validates the request and forwards delivery to the sender.

The client depends on `NotificationService`, not on delivery internals.

### Java Usage

Use Delegation when:

- You want composition over inheritance.
- You need runtime behavior swapping.
- You want a stable facade over pluggable helpers.
- You want easy testing with fake delegates.

## 5. Python Coding Example

```python
class EmailSender:
    def send(self, user_id, message):
        print(f"Email to {user_id}: {message}")


class SmsSender:
    def send(self, user_id, message):
        print(f"SMS to {user_id}: {message}")


class NotificationService:
    def __init__(self, sender):
        self._sender = sender

    def notify_user(self, user_id, message):
        if not message:
            raise ValueError("message is required")
        self._sender.send(user_id, message)


service = NotificationService(EmailSender())
service.notify_user("u-101", "Your order shipped")
```

### Python Usage

Python uses delegation frequently through composition and duck typing.

Common examples:

- Wrappers around file-like objects.
- Service classes delegating to repositories.
- Decorators forwarding calls.
- Adapters around third-party clients.

## 6. Where It Comes Handy in Real Life

- Service classes delegating persistence to repositories.
- Controllers delegating business logic to services.
- Framework objects delegating lifecycle events.
- Wrappers delegating to underlying resources.
- UI components delegating rendering or event handling.
- Test doubles replacing real delegates.

## 7. Advantages Over Normal Code Without Pattern

### Without Delegation

```java
class NotificationService extends EmailSender {
}
```

Problems:

- Tightly couples service to one implementation.
- Hard to switch to SMS or push.
- Inheritance exposes unwanted behavior.
- Tests require real sender behavior.

### With Delegation

```java
new NotificationService(new SmsSender());
```

Benefits:

- Behavior can be swapped.
- Classes stay focused.
- Testing is easier.
- Composition keeps object relationships explicit.

## 8. Where It Excels

- Composition-based design.
- Runtime behavior variation.
- Testability.
- Reusing helper objects.
- Keeping APIs stable while internals change.
- Reducing subclass explosion.

## 9. Where It Fails

- When delegation adds no meaningful flexibility.
- When every small method simply forwards to another object.
- When call stacks become too indirect.
- When ownership between delegator and delegate is unclear.
- When inheritance is simpler and stable.

## 10. Prebuilt Libraries and Packages

### Java

- `Collections.unmodifiableList` delegates to wrapped collections.
- Dynamic proxies delegate method calls to invocation handlers.
- Spring beans often delegate work to collaborators.
- AWT/Swing event handling uses delegate/listener objects.

### Python

- `__getattr__` can delegate missing attributes.
- Wrapper classes around streams and clients.
- `functools.wraps` supports function wrapper delegation.
- Composition is idiomatic in service objects.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces inheritance coupling. | Adds another object hop. |
| Supports runtime behavior swapping. | Can hide where work actually happens. |
| Improves testability. | Too much forwarding creates noise. |
| Encourages single responsibility. | Requires clear ownership boundaries. |

## 12. Real-World Identification Example

Scenario:

You build a payment service that can charge by card, wallet, or bank transfer.

Without Delegation:

- Payment service has conditionals for every method.
- Adding a new payment method modifies the service.

With Delegation:

- Payment service delegates to `PaymentProvider`.
- Each provider handles its own protocol.
- Tests use a fake provider.

## 13. MAANG Interview Triggers

Use Delegation when you hear:

- "Composition over inheritance."
- "Forward work to a helper."
- "Swap behavior at runtime."
- "Make this easier to test."
- "Avoid subclass explosion."
- "Keep public API stable but change internals."

### Interview-Ready Answer Format

1. Identify the behavior that should vary.
2. Define a delegate interface.
3. Inject or configure delegate implementation.
4. Let delegator handle orchestration and validation.
5. Forward specialized work to delegate.
6. Mention indirection and over-forwarding as trade-offs.

## 14. Common Mistakes

### Mistake 1: Delegator Does Nothing

If a class only forwards every method, ask whether the wrapper is needed.

### Mistake 2: Delegate Type Is Concrete

Depending on a concrete delegate reduces flexibility.

### Mistake 3: Hidden Lifecycle Ownership

Make clear who creates, owns, and disposes the delegate.

### Mistake 4: Confusing Delegation with Inheritance

Delegation is object composition, not subclass reuse.

### Mistake 5: Too Many Layers

Excess delegation makes debugging harder.

## 15. Delegation vs Similar Patterns

| Pattern | Difference |
|---|---|
| Delegation | General forwarding of responsibility to helper object. |
| Strategy | Delegation specifically for interchangeable algorithms. |
| Proxy | Delegation plus access control or lazy behavior. |
| Decorator | Delegation plus additional behavior while preserving interface. |
| Adapter | Delegation plus interface conversion. |

## 16. Delegation Design Checklist

| Question | Why it matters |
|---|---|
| What work is delegated? | Keeps responsibility clear. |
| Is delegate interchangeable? | Determines whether interface is needed. |
| Who owns delegate lifecycle? | Prevents resource leaks. |
| Does delegator add value? | Avoids useless wrappers. |
| Can tests replace delegate? | Improves maintainability. |

## 17. Quick Revision Notes

- Delegation forwards work to another object.
- It favors composition over inheritance.
- Delegator owns the public interaction.
- Delegate owns specialized work.
- Great for runtime flexibility.
- Avoid empty wrapper classes.

## 18. Mini Exercise

Design Delegation for `ReportService`.

Delegates:

- `PdfRenderer`
- `CsvRenderer`
- `HtmlRenderer`

Tasks:

- `ReportService` validates report data.
- Renderer handles format-specific output.
- Tests replace renderer with a fake.

## 19. Source Reference in This Repo

The repository's Delegation implementation uses `PrinterController` as a delegator and concrete printer classes as delegates.

Useful files:

- [github-repo/delegation/README.md](../../github-repo/delegation/README.md)
- [github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/Printer.java](../../github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/Printer.java)
- [github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/PrinterController.java](../../github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/PrinterController.java)
- [github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/printers/HpPrinter.java](../../github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/printers/HpPrinter.java)
- [github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/App.java](../../github-repo/delegation/src/main/java/com/iluwatar/delegation/simple/App.java)

