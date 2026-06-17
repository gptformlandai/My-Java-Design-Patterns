# Observer Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/observer](../../github-repo/observer)

## How to Study This Page

Use this page in three passes:

1. First pass: understand one subject notifying many observers.
2. Second pass: rewrite the Java example and explain subscribe, unsubscribe, and notify.
3. Third pass: study ordering, memory leaks, synchronous vs asynchronous notification, and event bus differences.

By the end, you should be able to say:

> Observer lets interested objects subscribe to state changes or events without the subject knowing concrete subscriber classes.

## 1. Technical Definition

Observer is a behavioral design pattern that defines a one-to-many dependency between objects. When the subject changes state, it notifies all registered observers through a common observer interface.

Core idea:

- A subject owns state or emits events.
- Observers subscribe to the subject.
- The subject notifies observers when something changes.
- The subject depends only on the observer interface, not concrete observers.

### 30-Second Interview Answer

I would use Observer when one object changes and multiple independent components need to react, such as UI views, cache invalidators, audit loggers, or notification handlers. The subject keeps a list of observers and calls them when an event happens. It improves decoupling, but we need to manage unsubscribe behavior, notification failures, ordering, and performance when many observers exist.

## 2. Layman and Easy to Understand Definition

Observer is like subscribing to a channel.

When the channel publishes a new update, every subscriber gets notified. The channel does not need to know what each subscriber does with the update.

In code:

- The channel is the subject.
- Subscribers are observers.
- Publishing is notification.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose an order changes status. Several things may need to happen:

- Update the UI.
- Send an email.
- Write an audit log.
- Clear a cache.
- Update analytics.

Bad design:

```java
public void markPaid(Order order) {
    order.markPaid();
    emailService.sendPaidEmail(order);
    auditService.record(order);
    cache.invalidate(order.id());
    analytics.track(order);
}
```

Problems:

- The order service knows every dependent action.
- Adding a new reaction requires editing the subject.
- The method becomes harder to test.
- Unrelated side effects pile up.

### 3.2 The Observer Solution

Let observers subscribe:

```java
orderEvents.addObserver(new EmailObserver());
orderEvents.addObserver(new AuditObserver());
```

Then the subject notifies them:

```java
orderEvents.publish(new OrderEvent(orderId, "PAID"));
```

The subject does not know what each observer does.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Subject / Observable | Object that stores observers and sends notifications. |
| Observer | Interface for receiving updates. |
| Concrete observer | Specific subscriber that reacts to updates. |
| Event / state | Data sent to observers. |
| Client | Code that registers or removes observers. |

### 3.4 Push vs Pull

| Style | Meaning | Example |
|---|---|---|
| Push | Subject sends event data to observers. | `observer.update(event)` |
| Pull | Subject notifies observers, and observers query the subject. | `observer.update(); subject.currentState()` |

Push is common because it keeps the event explicit. Pull is useful when observers need different parts of subject state.

### 3.5 Mental Model

Think of Observer as a subscription list.

1. Observer registers with the subject.
2. Subject stores the observer.
3. Subject state changes or event happens.
4. Subject loops through observers.
5. Each observer receives the update.
6. Observer can unsubscribe when no longer needed.

## 4. Java Coding Example

This example publishes order status changes to observers.

```java
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public record OrderEvent(String orderId, String status) {}

public interface OrderObserver {
    void onOrderChanged(OrderEvent event);
}

public final class OrderEventPublisher {
    private final List<OrderObserver> observers = new CopyOnWriteArrayList<>();

    public void addObserver(OrderObserver observer) {
        observers.add(observer);
    }

    public void removeObserver(OrderObserver observer) {
        observers.remove(observer);
    }

    public void publish(OrderEvent event) {
        for (OrderObserver observer : observers) {
            observer.onOrderChanged(event);
        }
    }
}

public final class EmailOrderObserver implements OrderObserver {
    @Override
    public void onOrderChanged(OrderEvent event) {
        if ("PAID".equals(event.status())) {
            System.out.println("Email sent for order " + event.orderId());
        }
    }
}

public final class AuditOrderObserver implements OrderObserver {
    @Override
    public void onOrderChanged(OrderEvent event) {
        System.out.println("Audit: " + event.orderId() + " changed to " + event.status());
    }
}
```

### Java Block by Block Explanation

#### Event

```java
public record OrderEvent(String orderId, String status) {}
```

This is the update data sent to observers.

#### Observer Interface

```java
public interface OrderObserver {
    void onOrderChanged(OrderEvent event);
}
```

Observers implement this interface to receive updates.

#### Subject / Observable

```java
public final class OrderEventPublisher {
```

The publisher stores observers and notifies them when an event occurs.

#### Observer List

```java
private final List<OrderObserver> observers = new CopyOnWriteArrayList<>();
```

`CopyOnWriteArrayList` allows safe iteration if observers are added or removed during notification. It is useful when reads are common and subscription changes are less frequent.

#### Notification

```java
for (OrderObserver observer : observers) {
    observer.onOrderChanged(event);
}
```

The subject calls each observer through the shared interface.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        OrderEventPublisher publisher = new OrderEventPublisher();

        publisher.addObserver(new EmailOrderObserver());
        publisher.addObserver(new AuditOrderObserver());

        publisher.publish(new OrderEvent("order-1", "PAID"));
    }
}
```

The publisher does not know concrete observer details beyond the interface.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from typing import Protocol


@dataclass(frozen=True)
class OrderEvent:
    order_id: str
    status: str


class OrderObserver(Protocol):
    def on_order_changed(self, event: OrderEvent) -> None:
        ...


class OrderEventPublisher:
    def __init__(self) -> None:
        self._observers: list[OrderObserver] = []

    def add_observer(self, observer: OrderObserver) -> None:
        self._observers.append(observer)

    def remove_observer(self, observer: OrderObserver) -> None:
        self._observers.remove(observer)

    def publish(self, event: OrderEvent) -> None:
        for observer in list(self._observers):
            observer.on_order_changed(event)


class EmailOrderObserver:
    def on_order_changed(self, event: OrderEvent) -> None:
        if event.status == "PAID":
            print(f"Email sent for order {event.order_id}")
```

### Python Usage

```python
publisher = OrderEventPublisher()
publisher.add_observer(EmailOrderObserver())
publisher.publish(OrderEvent("order-1", "PAID"))
```

## 6. Where It Comes Handy in Real Life

Observer is common in event-driven and UI-heavy systems.

Examples:

- UI event listeners.
- Model-view updates.
- Domain event handlers.
- Cache invalidation.
- Audit logging.
- Webhook subscribers.
- Reactive streams.
- Notification systems.

## 7. Advantages Over Normal Code Without Pattern

### Without Observer

```java
order.markPaid();
emailService.send(order);
auditService.record(order);
cache.invalidate(order.id());
```

Problems:

- Subject knows all side effects.
- New behavior requires changing subject code.
- Dependencies grow.
- Testing becomes noisy.

### With Observer

```java
publisher.publish(new OrderEvent(order.id(), "PAID"));
```

Benefits:

- Subject is decoupled from concrete reactions.
- Observers can be added or removed.
- Multiple independent reactions are supported.
- Event-driven workflows become clearer.

## 8. Where It Excels

Observer excels when:

- One event should notify many subscribers.
- The subject should not know concrete observers.
- Subscribers change over time.
- UI or state updates need loose coupling.
- Side effects are independent.
- Event ordering is not the central business rule.

## 9. Where It Fails

Observer is a poor fit when:

- The reaction order is strict and business-critical.
- Notifications need transactional guarantees.
- Observer failures must rollback the subject.
- There are so many observers that performance becomes hard to control.
- Subscription lifecycle is hard to manage.
- A message queue or event bus is a better fit.

## 10. Prebuilt Libraries and Packages

### Java

Common observer-like APIs:

- `java.util.EventListener`
- Swing listeners
- JavaBeans property change listeners
- Reactive Streams
- RxJava
- Project Reactor
- Spring application events

Important note:

- `java.util.Observer` exists historically but is deprecated. Prefer explicit listener interfaces, reactive libraries, or application event mechanisms.

### Python

Python options:

- Callback lists.
- Observer interfaces with `Protocol`.
- Blinker signals.
- RxPY.
- Event hooks in frameworks.
- Async queues for decoupled event delivery.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples subject from observers. | Can cause memory leaks if observers are not removed. |
| Supports multiple subscribers. | Notification order may be unclear. |
| Allows dynamic subscription. | Observer failures can affect notification flow. |
| Fits UI and event systems well. | Debugging event chains can be harder. |
| Keeps side effects modular. | Synchronous notification can slow the subject. |

## 12. Real-World Identification Example

Scenario:

You are building an order platform.

When an order is paid:

- Email should be sent.
- Audit log should be written.
- Analytics should be updated.
- Fraud model should receive a signal.

Should you use Observer?

Yes, if these reactions are independent and the order service should not know each concrete subscriber.

Good usage:

```java
orderEvents.publish(new OrderEvent(order.id(), "PAID"));
```

For cross-service reliability, this may evolve into a message broker or event streaming system.

## 13. MAANG Interview Triggers

Think Observer when you hear:

- One-to-many notification.
- Subscribers/listeners.
- State change notification.
- UI events.
- Domain events.
- Publish/subscribe.
- Event-driven design.
- Loose coupling between source and reactions.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the subject and event.
2. Define the observer interface.
3. Let observers subscribe and unsubscribe.
4. Notify observers when state changes.
5. Decide sync vs async delivery.
6. Mention trade-offs: leaks, ordering, failures, and performance.

## 14. Common Mistakes

### Mistake 1: Forgetting to Unsubscribe

Long-lived subjects holding observers can prevent observers from being garbage collected.

### Mistake 2: Assuming Notification Order

If order matters, define it explicitly or use a workflow pattern.

### Mistake 3: Letting One Observer Break All Others

Decide whether one observer exception should stop notification or be isolated.

### Mistake 4: Doing Heavy Work Synchronously

Slow observers can slow the subject. Consider async processing for heavy work.

### Mistake 5: Confusing Observer with Message Queue

In-memory Observer is not automatically durable, distributed, or retryable.

## 15. Observer vs Similar Patterns

| Pattern | Difference |
|---|---|
| Publish/Subscribe | Often uses a broker and decouples publishers from subscribers more strongly. Observer is often direct in-memory subscription. |
| Mediator | Mediator centralizes communication between peers. Observer broadcasts changes from a subject. |
| Chain of Responsibility | Chain passes a request along handlers. Observer notifies all subscribers. |
| Command | Command encapsulates an action. Observer notifies listeners about an event. |
| Event Sourcing | Event Sourcing stores events as source of truth. Observer only reacts to notifications. |

## 16. Notification Design Checklist

| Concern | Why it matters |
|---|---|
| Sync or async | Affects latency and failure handling. |
| Ordering | Some observers may depend on sequence. |
| Error handling | One observer may fail. |
| Subscription lifecycle | Prevents leaks. |
| Event payload | Too little or too much data creates coupling. |
| Thread safety | Subjects may notify from multiple threads. |

## 17. Quick Revision Notes

- Observer is one subject to many observers.
- Observers subscribe and unsubscribe.
- Subject notifies through an interface.
- Great for UI, listeners, and domain events.
- Watch memory leaks, ordering, failures, and slow observers.
- Distributed reliability usually needs a broker, not just Observer.

## 18. Mini Exercise

Design Observer for `StockPriceTracker`.

Subject:

- `StockPricePublisher`

Observers:

- `DashboardObserver`
- `AlertObserver`
- `AuditObserver`

Rules:

- Observers can subscribe and unsubscribe.
- Each price update sends symbol, old price, and new price.
- One observer failure should not stop all notifications.

Expected usage:

```java
publisher.addObserver(new AlertObserver());
publisher.publish(new PriceChanged("AAPL", 190.00, 194.50));
```

## 19. Source Reference in This Repo

The repository's Observer implementation uses `Weather` as the subject and `WeatherObserver` implementations as observers.

Useful files:

- [github-repo/observer/README.md](../../github-repo/observer/README.md)
- [github-repo/observer/src/main/java/com/iluwatar/observer/Weather.java](../../github-repo/observer/src/main/java/com/iluwatar/observer/Weather.java)
- [github-repo/observer/src/main/java/com/iluwatar/observer/WeatherObserver.java](../../github-repo/observer/src/main/java/com/iluwatar/observer/WeatherObserver.java)
- [github-repo/observer/src/main/java/com/iluwatar/observer/Orcs.java](../../github-repo/observer/src/main/java/com/iluwatar/observer/Orcs.java)
- [github-repo/observer/src/main/java/com/iluwatar/observer/Hobbits.java](../../github-repo/observer/src/main/java/com/iluwatar/observer/Hobbits.java)
