# Event Aggregator Pattern

Category: Messaging, Eventing, and Reactive Flow Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [event-aggregator](../../github-repo/event-aggregator)

---

## How to Study This Page

Study Event Aggregator as a central event hub inside an application.

The key idea:
- many sources emit events
- one aggregator listens to them
- clients subscribe to the aggregator instead of every source

---

## 1. Technical Definition

Event Aggregator is a pattern where a central component collects events from multiple sources and routes them to interested observers, reducing direct subscriptions between many components.

### 30-Second Interview Answer

Event Aggregator centralizes event routing. Instead of a client subscribing to many event sources one by one, it subscribes to an aggregator. The aggregator listens to multiple sources and forwards relevant events. I would use it in large in-process applications, complex UIs, or modular systems where many components emit events and direct observer wiring becomes messy.

---

## 2. Layman and Easy to Understand Definition

Imagine a building reception desk. Many departments send updates to reception, and people who need updates ask reception instead of visiting every department.

The reception desk is the event aggregator.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

With many event sources, subscriptions become tangled:

```text
Client -> Source A
Client -> Source B
Client -> Source C
Client -> Source D
```

Each client must know every source.

### Event Aggregator Flow

1. Event sources emit events.
2. Aggregator subscribes to those sources.
3. Clients subscribe to the aggregator.
4. Source emits an event.
5. Aggregator receives it.
6. Aggregator forwards it to interested clients.

### Core Participants

| Participant | Responsibility |
|---|---|
| Event source | Emits events |
| Event aggregator | Collects and routes events |
| Observer/listener | Receives relevant events |
| Event type | Identifies what happened |
| Subscription map | Stores observer interest |

---

## 4. Java Coding Example

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.function.Consumer;

enum EventType {
    CPU_HIGH,
    DISK_FULL
}

class EventAggregator {
    private final Map<EventType, List<Consumer<EventType>>> observers = new HashMap<>();

    public void subscribe(EventType type, Consumer<EventType> observer) {
        observers.computeIfAbsent(type, ignored -> new ArrayList<>()).add(observer);
    }

    public void onEvent(EventType type) {
        for (Consumer<EventType> observer : observers.getOrDefault(type, List.of())) {
            observer.accept(type);
        }
    }
}

class ServerMetricSource {
    private final EventAggregator aggregator;

    ServerMetricSource(EventAggregator aggregator) {
        this.aggregator = aggregator;
    }

    public void detectDiskFull() {
        aggregator.onEvent(EventType.DISK_FULL);
    }
}

public class EventAggregatorDemo {
    public static void main(String[] args) {
        EventAggregator aggregator = new EventAggregator();
        aggregator.subscribe(EventType.DISK_FULL, event -> System.out.println("alert: " + event));

        ServerMetricSource source = new ServerMetricSource(aggregator);
        source.detectDiskFull();
    }
}
```

### Java Block by Block

`EventAggregator` stores observers by event type.

`ServerMetricSource` reports events to the aggregator.

Observers subscribe to the aggregator, not to every source.

The central routing point reduces wiring complexity.

---

## 5. Python Coding Example

```python
from collections import defaultdict


class EventAggregator:
    def __init__(self):
        self.observers = defaultdict(list)

    def subscribe(self, event_type, observer):
        self.observers[event_type].append(observer)

    def on_event(self, event_type):
        for observer in self.observers[event_type]:
            observer(event_type)


class MetricSource:
    def __init__(self, aggregator):
        self.aggregator = aggregator

    def detect_disk_full(self):
        self.aggregator.on_event("DISK_FULL")


aggregator = EventAggregator()
aggregator.subscribe("DISK_FULL", lambda event: print("alert:", event))

source = MetricSource(aggregator)
source.detect_disk_full()
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Complex desktop UI | Many widgets emit events |
| Modular monolith | Modules need loose in-process communication |
| Application event bus | Components subscribe through one hub |
| Monitoring dashboard | Many sources feed alert observers |
| Plugin systems | Plugins listen to app events |
| Domain notifications | Internal events are routed centrally |

---

## 7. Advantages Over Normal Code Without Pattern

Without Event Aggregator:
- clients know too many event sources
- subscription wiring is duplicated
- adding sources requires touching many clients
- event routing logic is scattered

With Event Aggregator:
- clients subscribe to one component
- routing logic is centralized
- sources and observers are more decoupled
- adding a new source is easier

---

## 8. Where It Excels

It excels when:
- there are many event sources
- events are in-process or inside one bounded application
- clients need a simplified subscription point
- source-to-listener wiring is becoming hard to maintain

---

## 9. Where It Fails

It fails when:
- the aggregator becomes a huge god object
- all events are routed through one bottleneck
- event ownership becomes unclear
- distributed delivery guarantees are required
- observers perform slow work synchronously

For distributed systems, use a broker or event bus with clear contracts.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Guava EventBus, Spring ApplicationEventPublisher |
| UI | JavaFX events, Swing event listeners |
| Messaging | In-process event buses |
| .NET | MediatR, Prism EventAggregator |
| JavaScript | EventEmitter, mitt |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces subscription complexity | Can become a central bottleneck |
| Decouples sources and observers | Harder to trace event ownership |
| Simplifies clients | Can hide dependencies |
| Good for in-process events | Not enough for durable messaging |
| Easier to add sources | Synchronous handlers can block |

---

## 12. Real-World Identification Example

Question:

> A desktop app has many panels, background services, and plugins. Components need to react to internal events, but direct listener wiring is becoming messy. What pattern helps?

Strong answer:

Use Event Aggregator. Components publish internal events to a central aggregator, and panels/plugins subscribe to the event types they care about. I would keep event types well-named, avoid slow synchronous handlers, and prevent the aggregator from becoming a place for business logic.

---

## 13. MAANG Interview Triggers

Say Event Aggregator when you hear:
- central event hub
- many event sources
- clients should not subscribe to every source
- in-process events
- simplify event wiring
- decouple UI components
- plugin notifications

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Putting business logic in aggregator | It becomes hard to maintain | Keep aggregator as routing layer |
| No unsubscribe support | Memory leaks in long-running apps | Add lifecycle-aware subscription |
| One generic event type | Handlers become fragile | Use explicit event types |
| Synchronous slow observers | Blocks all event delivery | Dispatch asynchronously if needed |
| Using it for distributed guarantees | It is usually in-process | Use brokered messaging |

---

## 15. Event Aggregator vs Similar Patterns

| Pattern | Difference |
|---|---|
| Observer | Observer wires sources to observers; aggregator centralizes many sources |
| Mediator | Mediator coordinates interactions; aggregator focuses on event collection and routing |
| Publish-Subscribe | Pub/sub often uses topics/brokers; aggregator is often in-process |
| Event Bus | Event bus is broader infrastructure; aggregator is a specific centralizing pattern |
| Facade | Facade simplifies method calls; aggregator simplifies event subscriptions |

---

## 16. Event Aggregator Design Checklist

- What sources does the aggregator observe?
- What event types are supported?
- Who can subscribe?
- How do subscribers unsubscribe?
- Are handlers synchronous or asynchronous?
- Is the aggregator only routing, or doing too much?
- How are errors in observers handled?
- How is event flow traced?

---

## 17. Quick Revision Notes

- One-line summary: One central hub gathers events from many sources.
- Memory hook: "reception desk for internal events."
- Best for: in-process systems with many event sources.
- Avoid when: durable distributed messaging is required.
- Interview line: "I would use an aggregator to centralize event subscriptions while keeping business logic in handlers, not in the aggregator."

---

## 18. Mini Exercise

Design an event aggregator for a dashboard:
- define three event sources
- define four event types
- define two observers
- add unsubscribe behavior
- decide whether observer calls should be synchronous or async

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/event-aggregator/README.md)
- [EventEmitter.java](../../github-repo/event-aggregator/src/main/java/com/iluwatar/event/aggregator/EventEmitter.java)
- [EventObserver.java](../../github-repo/event-aggregator/src/main/java/com/iluwatar/event/aggregator/EventObserver.java)
- [KingsHand.java](../../github-repo/event-aggregator/src/main/java/com/iluwatar/event/aggregator/KingsHand.java)
- [Event.java](../../github-repo/event-aggregator/src/main/java/com/iluwatar/event/aggregator/Event.java)
- [Scout.java](../../github-repo/event-aggregator/src/main/java/com/iluwatar/event/aggregator/Scout.java)
- [App.java](../../github-repo/event-aggregator/src/main/java/com/iluwatar/event/aggregator/App.java)

The repo implementation has event emitters, observers, and an aggregator class that receives events from multiple emitters and forwards them to registered observers.

