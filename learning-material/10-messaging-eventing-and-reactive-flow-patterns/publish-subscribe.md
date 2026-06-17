# Publish-Subscribe Pattern

Category: Messaging, Eventing, and Reactive Flow Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [publish-subscribe](../../github-repo/publish-subscribe)

---

## How to Study This Page

Study Publish-Subscribe as the most common event distribution pattern.

Keep the core sentence in mind:

> Publishers do not send directly to receivers; they publish to a topic, and subscribers receive what they subscribed to.

---

## 1. Technical Definition

Publish-Subscribe is a messaging pattern where publishers send messages to topics or channels, and subscribers independently receive messages from the topics they are interested in.

### 30-Second Interview Answer

Publish-Subscribe decouples senders from receivers. A publisher emits a message to a topic, and all subscribers to that topic receive it. I would use it when multiple independent components need to react to the same event, such as sending email, updating analytics, and triggering notifications. The major design concerns are delivery guarantees, subscriber isolation, ordering, duplicate handling, and backpressure.

---

## 2. Layman and Easy to Understand Definition

Think of subscribing to a topic feed. The publisher posts an update to the topic. Everyone subscribed to that topic receives it.

The publisher does not know who the subscribers are.

Subscribers can join or leave without changing publisher code.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Direct notification creates coupling:

```text
Producer -> Subscriber A
Producer -> Subscriber B
Producer -> Subscriber C
```

Every new subscriber requires producer awareness.

### Pub/Sub Flow

1. A topic or channel is created.
2. Subscribers register interest in that topic.
3. A publisher publishes a message to the topic.
4. The broker/topic dispatches the message to subscribers.
5. Each subscriber processes independently.
6. Failed subscribers can retry without blocking unrelated subscribers.

### Core Participants

| Participant | Responsibility |
|---|---|
| Publisher | Sends messages |
| Topic/Channel | Groups messages by interest |
| Subscriber | Receives matching messages |
| Broker | Stores/routes messages in distributed systems |
| Message | Payload plus metadata |

---

## 4. Java Coding Example

```java
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.function.Consumer;

class PubSub {
    private final Map<String, List<Consumer<String>>> subscribers = new ConcurrentHashMap<>();

    public void subscribe(String topic, Consumer<String> subscriber) {
        subscribers.computeIfAbsent(topic, key -> new CopyOnWriteArrayList<>()).add(subscriber);
    }

    public void publish(String topic, String message) {
        for (Consumer<String> subscriber : subscribers.getOrDefault(topic, List.of())) {
            subscriber.accept(message);
        }
    }
}

public class PublishSubscribeDemo {
    public static void main(String[] args) {
        PubSub pubSub = new PubSub();

        pubSub.subscribe("orders", message -> System.out.println("email: " + message));
        pubSub.subscribe("orders", message -> System.out.println("analytics: " + message));

        pubSub.publish("orders", "OrderPaid#123");
    }
}
```

### Java Block by Block

`subscribe` adds a consumer to a topic.

`publish` sends the message to every subscriber currently attached to that topic.

The publisher does not call email or analytics directly.

### Java Output

```text
email: OrderPaid#123
analytics: OrderPaid#123
```

---

## 5. Python Coding Example

```python
from collections import defaultdict


class PubSub:
    def __init__(self):
        self.subscribers = defaultdict(list)

    def subscribe(self, topic, subscriber):
        self.subscribers[topic].append(subscriber)

    def publish(self, topic, message):
        for subscriber in self.subscribers[topic]:
            subscriber(message)


pubsub = PubSub()
pubsub.subscribe("orders", lambda message: print("email:", message))
pubsub.subscribe("orders", lambda message: print("analytics:", message))
pubsub.publish("orders", "OrderPaid#123")
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Notification systems | Many subscribers react to one event |
| Analytics pipelines | Metrics consumers receive business events |
| Cache invalidation | Many nodes hear invalidation messages |
| Microservices events | Services subscribe to relevant topics |
| Chat channels | Users receive messages for subscribed channels |
| Market data | Consumers subscribe to symbol or feed topics |

---

## 7. Advantages Over Normal Code Without Pattern

Without Pub/Sub:
- publisher knows every receiver
- adding a receiver changes publisher code
- one receiver failure can affect the sender
- fan-out logic is duplicated

With Pub/Sub:
- publishers and subscribers are loosely coupled
- new subscribers can be added independently
- fan-out is centralized in the topic/broker
- subscribers process at their own pace when brokered

---

## 8. Where It Excels

It excels when:
- one message should reach many independent consumers
- consumers can change over time
- senders should not know receivers
- asynchronous processing is acceptable
- topic-based routing is natural

---

## 9. Where It Fails

It fails when:
- the sender needs an immediate response from one known receiver
- exactly-once side effects are assumed without design
- subscribers cannot handle duplicate messages
- topic names become uncontrolled
- slow consumers create backlog without backpressure

Pub/Sub is not a magic delivery guarantee. The broker and consumer design decide the guarantees.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Spring Events, Reactor sinks, Guava EventBus |
| Brokers | Kafka, RabbitMQ, ActiveMQ, Pulsar |
| Cloud | Google Pub/Sub, AWS SNS, AWS EventBridge, Azure Service Bus |
| JavaScript | EventEmitter, Redis Pub/Sub |
| Python | Blinker, Redis Pub/Sub, Celery broadcasts |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Loose coupling | More operational moving parts |
| Easy fan-out | Delivery semantics vary |
| Dynamic subscribers | Duplicate messages are common |
| Asynchronous processing | Harder end-to-end tracing |
| Scales independent consumers | Ordering can be tricky |

---

## 12. Real-World Identification Example

Question:

> A payment service emits `PaymentCaptured`. Email, analytics, fraud, and rewards all need to react, and new consumers may be added later. What pattern fits?

Strong answer:

Use Publish-Subscribe. The payment service publishes `PaymentCaptured` to a topic. Each subscriber consumes independently. I would include event IDs for deduplication, schema versions for compatibility, consumer-level retries, and observability through correlation IDs.

---

## 13. MAANG Interview Triggers

Say Publish-Subscribe when you hear:
- one-to-many event distribution
- topic/channel
- subscribers register interest
- decouple sender and receivers
- broadcast to multiple consumers
- async notifications
- dynamic subscribers

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Assuming all subscribers process instantly | Subscribers can lag or fail | Monitor lag and retries |
| No deduplication | Retries can double-apply side effects | Use event IDs |
| Too many broad topics | Consumers filter too much | Design meaningful topic boundaries |
| Synchronous subscriber callbacks in critical path | Slow subscriber hurts publisher | Use async dispatch or broker |
| No schema evolution plan | Consumers break on payload changes | Version schemas |

---

## 15. Publish-Subscribe vs Similar Patterns

| Pattern | Difference |
|---|---|
| Observer | Usually in-process; pub/sub often uses broker/topics |
| Event Queue | Queue often load-balances work; pub/sub fans out to subscribers |
| Event-Driven Architecture | Pub/sub is one mechanism inside EDA |
| Event Aggregator | Aggregator centralizes events from multiple sources |
| Message Bus | Broader infrastructure concept that may include pub/sub |

---

## 16. Publish-Subscribe Design Checklist

- What topics exist?
- Who owns each topic schema?
- Is delivery at-most-once, at-least-once, or effectively-once?
- Can subscribers process messages idempotently?
- What happens when a subscriber is slow?
- Is ordering required within a topic?
- How are subscribers monitored?
- How do consumers unsubscribe or replay?

---

## 17. Quick Revision Notes

- One-line summary: Publish to a topic, subscribers receive independently.
- Memory hook: "topic feed with listeners."
- Best for: fan-out event distribution.
- Avoid when: one direct request/response is needed.
- Interview line: "I would publish to a topic, let independent subscribers consume, and design for dedupe, retries, ordering, and subscriber lag."

---

## 18. Mini Exercise

Design a pub/sub notification flow:
- create topics for `user.signup`, `order.paid`, and `invoice.failed`
- add at least two subscribers per topic
- decide which subscribers require retries
- define one metric to monitor subscriber lag

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/publish-subscribe/README.md)
- [Topic.java](../../github-repo/publish-subscribe/src/main/java/com/iluwatar/publish/subscribe/model/Topic.java)
- [Message.java](../../github-repo/publish-subscribe/src/main/java/com/iluwatar/publish/subscribe/model/Message.java)
- [Publisher.java](../../github-repo/publish-subscribe/src/main/java/com/iluwatar/publish/subscribe/publisher/Publisher.java)
- [PublisherImpl.java](../../github-repo/publish-subscribe/src/main/java/com/iluwatar/publish/subscribe/publisher/PublisherImpl.java)
- [Subscriber.java](../../github-repo/publish-subscribe/src/main/java/com/iluwatar/publish/subscribe/subscriber/Subscriber.java)
- [App.java](../../github-repo/publish-subscribe/src/main/java/com/iluwatar/publish/subscribe/App.java)

The repo implementation models topics with subscriber sets. `PublisherImpl` publishes to registered topics, and `Topic.publish()` invokes each subscriber asynchronously.

