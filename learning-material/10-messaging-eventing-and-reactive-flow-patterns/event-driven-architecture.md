# Event-Driven Architecture Pattern

Category: Messaging, Eventing, and Reactive Flow Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [event-driven-architecture](../../github-repo/event-driven-architecture)

---

## How to Study This Page

Study Event-Driven Architecture as the broad system style behind many modern distributed systems.

Focus on four questions:
- What event happened?
- Who produced it?
- Who should react to it?
- What guarantees do we need around delivery, ordering, and failure?

---

## 1. Technical Definition

Event-Driven Architecture is an architectural style where components communicate by producing, routing, and consuming events that represent important facts or state changes in the system.

### 30-Second Interview Answer

Event-Driven Architecture means the system is organized around events such as `UserCreated`, `OrderPaid`, or `PaymentFailed`. Producers emit events without tightly coupling to consumers. Consumers react independently, often asynchronously. I would use it when many parts of a system need to react to business changes, when workflows should be decoupled, and when scalability or responsiveness matters. The trade-offs are eventual consistency, ordering, duplicate handling, observability, and operational complexity.

---

## 2. Layman and Easy to Understand Definition

Imagine a company-wide notification system. One team announces, "An order was paid." Shipping, email, analytics, fraud, and rewards can each react in their own way.

The order service does not need to call every team directly.

That is the spirit of event-driven architecture:
- one thing happens
- an event is published
- interested components react

---

## 3. Bit by Bit Explanation of What It Is

### Problem

In direct service-to-service communication, one operation can become a chain of synchronous calls:

```text
Order service -> Payment service -> Inventory service -> Email service -> Analytics service
```

This creates tight coupling. If email is slow, order checkout might become slow. If analytics is down, the business flow might fail for the wrong reason.

### Event-Driven Flow

1. A producer performs a business action.
2. It creates an event that describes what already happened.
3. The event goes to a dispatcher, broker, event bus, or stream.
4. One or more consumers receive the event.
5. Each consumer performs its own reaction.
6. Failures are retried or routed to a dead-letter path.
7. Observability tracks event flow across services.

### Core Participants

| Participant | Responsibility |
|---|---|
| Event | Immutable fact such as `UserCreated` |
| Producer | Component that emits the event |
| Router/Broker | Routes events to consumers |
| Consumer/Handler | Reacts to the event |
| Event schema | Defines event structure and version |
| Dead-letter path | Holds messages that cannot be processed |

### Important Distinction

Event-driven architecture is the broad architecture style. Publish-subscribe, event queues, streams, and event sourcing are mechanisms that can appear inside it.

---

## 4. Java Coding Example

```java
import java.util.HashMap;
import java.util.Map;
import java.util.function.Consumer;

interface DomainEvent {
    String type();
}

record UserCreated(String username) implements DomainEvent {
    public String type() {
        return "UserCreated";
    }
}

class EventDispatcher {
    private final Map<String, Consumer<DomainEvent>> handlers = new HashMap<>();

    public void register(String eventType, Consumer<DomainEvent> handler) {
        handlers.put(eventType, handler);
    }

    public void dispatch(DomainEvent event) {
        Consumer<DomainEvent> handler = handlers.get(event.type());
        if (handler != null) {
            handler.accept(event);
        }
    }
}

public class EventDrivenArchitectureDemo {
    public static void main(String[] args) {
        EventDispatcher dispatcher = new EventDispatcher();

        dispatcher.register("UserCreated", event -> {
            UserCreated userCreated = (UserCreated) event;
            System.out.println("Send welcome email to " + userCreated.username());
        });

        dispatcher.dispatch(new UserCreated("aravind"));
    }
}
```

### Java Block by Block

`DomainEvent` represents a fact that happened.

`UserCreated` is a concrete event.

`EventDispatcher` stores the mapping from event type to handler.

The producer dispatches an event, and the handler reacts without the producer knowing the handler's internals.

### Java Output

```text
Send welcome email to aravind
```

---

## 5. Python Coding Example

```python
from collections import defaultdict
from dataclasses import dataclass


@dataclass(frozen=True)
class UserCreated:
    username: str


class EventBus:
    def __init__(self):
        self.handlers = defaultdict(list)

    def subscribe(self, event_type, handler):
        self.handlers[event_type].append(handler)

    def publish(self, event):
        for handler in self.handlers[type(event)]:
            handler(event)


bus = EventBus()
bus.subscribe(UserCreated, lambda event: print("send welcome email to", event.username))
bus.publish(UserCreated("aravind"))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| E-commerce checkout | Payment, inventory, email, analytics react independently |
| User onboarding | Multiple services react to user creation |
| Financial transaction monitoring | Many consumers analyze the same event stream |
| IoT telemetry | Devices emit events for downstream processing |
| Audit pipelines | Events become traceable business facts |
| Microservices integration | Services avoid direct synchronous chains |

---

## 7. Advantages Over Normal Code Without Pattern

Without Event-Driven Architecture:
- producers call consumers directly
- one slow dependency can slow the whole workflow
- adding a new reaction requires changing producer code
- failures spread across synchronous chains

With Event-Driven Architecture:
- producers and consumers are decoupled
- new consumers can be added with less producer change
- work can run asynchronously
- systems can absorb bursts through brokers and queues

---

## 8. Where It Excels

It excels when:
- many consumers need the same business fact
- the reaction can happen asynchronously
- workflows are distributed across services
- audit and observability matter
- traffic is bursty and needs buffering

---

## 9. Where It Fails

It fails when:
- every step needs a single immediate transactional response
- event ordering is not designed carefully
- consumers are not idempotent
- schemas change without compatibility
- observability is weak
- teams confuse "event emitted" with "business process complete"

Event-driven systems require discipline around contracts, retries, deduplication, and monitoring.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Spring Application Events, Spring Integration, Axon Framework |
| Messaging | Kafka, RabbitMQ, ActiveMQ, Pulsar |
| Cloud | AWS EventBridge, SNS/SQS, Google Pub/Sub, Azure Event Grid |
| Streaming | Kafka Streams, Flink, Spark Structured Streaming |
| Observability | OpenTelemetry, distributed tracing, correlation IDs |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples producers and consumers | Eventual consistency |
| Scales reactions independently | Harder debugging |
| Supports many consumers | Requires schema/version discipline |
| Buffers bursty traffic | Duplicate and out-of-order events |
| Fits microservices well | Broker becomes critical infrastructure |

---

## 12. Real-World Identification Example

Question:

> A checkout service needs to trigger shipping, email, fraud checks, rewards, and analytics after an order is paid. The checkout path should not synchronously wait for all of them. What architecture fits?

Strong answer:

Use Event-Driven Architecture. The checkout service emits an `OrderPaid` event after payment succeeds. Shipping, email, rewards, fraud, and analytics subscribe independently. Consumers must be idempotent, events should include correlation IDs and schema versions, and failed events should be retried or sent to a dead-letter queue.

---

## 13. MAANG Interview Triggers

Say Event-Driven Architecture when you hear:
- many services react to one business action
- asynchronous workflow
- decouple producers from consumers
- eventual consistency is acceptable
- bursty traffic needs buffering
- audit trail of business facts
- notification-driven system

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Treating events like RPC calls | Creates hidden coupling | Publish facts, not commands |
| No idempotency | Retries cause duplicate effects | Use event IDs and deduplication |
| No schema versioning | Consumers break on event changes | Version event contracts |
| Ignoring ordering | State can be applied incorrectly | Partition by entity key when needed |
| Weak observability | Flow is hard to debug | Add correlation IDs and tracing |

---

## 15. Event-Driven Architecture vs Similar Patterns

| Pattern | Difference |
|---|---|
| Publish-Subscribe | Pub/sub is a messaging mechanism; EDA is the architecture style |
| Event Queue | Queue buffers events for later processing, often one consumer group |
| Event Sourcing | Stores all state changes as events |
| Observer | In-process notification pattern, usually simpler |
| CQRS | Separates reads and writes; often combined with EDA |

---

## 16. Event-Driven Design Checklist

- What business fact does the event represent?
- Is the event a fact or a command?
- Who owns the event schema?
- What key controls ordering?
- Are consumers idempotent?
- What is the retry and dead-letter strategy?
- How are events traced end to end?
- What consistency model do users observe?

---

## 17. Quick Revision Notes

- One-line summary: Business facts drive the system flow.
- Memory hook: "Something happened, interested services react."
- Best for: decoupled workflows with many reactions.
- Avoid when: one synchronous transaction must complete all work.
- Interview line: "I would publish a durable event, let consumers process independently, and design for idempotency, ordering, retries, and observability."

---

## 18. Mini Exercise

Design an event-driven order system:
- emit `OrderPlaced`, `PaymentCaptured`, and `OrderShipped`
- list three consumers for each event
- define one idempotency key
- define one dead-letter condition
- decide which events need strict ordering

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/event-driven-architecture/README.md)
- [EventDispatcher.java](../../github-repo/event-driven-architecture/src/main/java/com/iluwatar/eda/framework/EventDispatcher.java)
- [Event.java](../../github-repo/event-driven-architecture/src/main/java/com/iluwatar/eda/framework/Event.java)
- [Handler.java](../../github-repo/event-driven-architecture/src/main/java/com/iluwatar/eda/framework/Handler.java)
- [UserCreatedEvent.java](../../github-repo/event-driven-architecture/src/main/java/com/iluwatar/eda/event/UserCreatedEvent.java)
- [UserCreatedEventHandler.java](../../github-repo/event-driven-architecture/src/main/java/com/iluwatar/eda/handler/UserCreatedEventHandler.java)
- [App.java](../../github-repo/event-driven-architecture/src/main/java/com/iluwatar/eda/App.java)

The repo implementation uses `EventDispatcher` to register event types with handlers, then dispatches concrete user events to their matching handlers.

