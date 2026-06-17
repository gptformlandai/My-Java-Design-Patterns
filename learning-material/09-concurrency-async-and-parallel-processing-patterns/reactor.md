# Reactor Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/reactor](../../github-repo/reactor)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the event loop as a demultiplexer for many I/O events.
2. Second pass: trace channel registration, readiness event, dispatch, and handler execution.
3. Third pass: compare Reactor with Proactor, Thread-Pool Executor, and event-based async.

By the end, you should be able to say:

> Reactor uses an event loop to handle many I/O connections with a small number of threads.

## 1. Technical Definition

Reactor is a concurrent event-handling pattern where an event loop waits for readiness events on multiple handles and dispatches each event to the right handler.

Core idea:

- Many channels are registered with a reactor.
- Event loop waits for readiness.
- Dispatcher sends ready events to handlers.
- Handlers perform non-blocking work.
- A thread pool may handle heavier processing.

### 30-Second Interview Answer

I would use Reactor for high-concurrency network servers where one thread per connection would be too expensive. The reactor waits for socket readiness, dispatches events to handlers, and keeps I/O non-blocking. The trade-off is complexity: handlers must avoid blocking the event loop, and state machines become harder to debug.

## 2. Layman and Easy to Understand Definition

Reactor is like a front desk watching many counters at once.

When any counter signals that it needs attention, the front desk sends the right worker to handle it.

In code:

- Register channels.
- Wait for events.
- Find matching handler.
- Dispatch event.
- Handler processes event.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Thread-per-connection servers do not scale well:

```text
10000 sockets -> 10000 threads -> memory and context switch cost
```

### 3.2 The Reactor Solution

Use non-blocking I/O and event demultiplexing:

```text
channels -> selector/event loop -> handlers
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Handle/channel | I/O resource such as socket. |
| Reactor | Event loop that waits for readiness. |
| Dispatcher | Sends events to handlers. |
| Event handler | Application logic for event. |
| Selector | OS/JVM primitive for readiness events. |

### 3.4 Runtime Flow

1. Server registers channel and handler.
2. Reactor waits for I/O readiness.
3. A channel becomes readable or writable.
4. Reactor identifies the handler.
5. Dispatcher invokes handler.
6. Handler processes without blocking the event loop.

## 4. Java Coding Example

```java
import java.util.HashMap;
import java.util.Map;

interface EventHandler {
    void handle(String event);
}

class SimpleReactor {
    private final Map<String, EventHandler> handlers = new HashMap<>();

    void register(String channel, EventHandler handler) {
        handlers.put(channel, handler);
    }

    void onReady(String channel, String event) {
        EventHandler handler = handlers.get(channel);
        if (handler != null) {
            handler.handle(event);
        }
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `register` | Associates handle with handler. |
| `onReady` | Represents readiness event. |
| `handlers` | Lookup from channel to behavior. |
| `EventHandler` | Application-specific processing. |
| no blocking call | Handler should stay quick. |

### Java Usage

```java
SimpleReactor reactor = new SimpleReactor();
reactor.register("socket-1", event -> System.out.println("handled " + event));
reactor.onReady("socket-1", "readable");
```

## 5. Python Coding Example

```python
class Reactor:
    def __init__(self):
        self.handlers = {}

    def register(self, channel, handler):
        self.handlers[channel] = handler

    def on_ready(self, channel, event):
        handler = self.handlers.get(channel)
        if handler:
            handler(event)


reactor = Reactor()
reactor.register("socket-1", lambda event: print(f"handled {event}"))
reactor.on_ready("socket-1", "readable")
```

### Python Usage

Use this shape when explaining:

- Reactor maps events to handlers.
- Real implementations use selectors or event loops.
- Blocking work must move off the event loop.

## 6. Where It Comes Handy in Real Life

- Non-blocking web servers.
- TCP/UDP servers.
- Chat servers.
- API gateways.
- Netty-based services.
- Event-loop frameworks.
- High-concurrency I/O systems.

## 7. Advantages Over Normal Code Without Pattern

Without Reactor:

```text
one thread blocks per connection
```

With Reactor:

```text
few event-loop threads handle many connections
```

Benefits:

- Better connection scalability.
- Lower thread overhead.
- Efficient non-blocking I/O.
- Central event dispatching.
- Good fit for network servers.

## 8. Where It Excels

- Many mostly idle connections.
- I/O waits dominate CPU work.
- Handlers can be non-blocking.
- Low thread overhead matters.
- Event-driven architecture is acceptable.

## 9. Where It Fails

- Handlers block the event loop.
- CPU-heavy work is not offloaded.
- State machines become too complex.
- Backpressure is ignored.
- Long operations run inside event handler.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Java NIO, Netty, Vert.x, Undertow |
| Reactive | Project Reactor, RxJava event loops |
| Python | `asyncio`, Twisted |
| JavaScript | Node.js event loop |
| Network | libevent, libuv |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Handles many connections efficiently. | More complex than blocking code. |
| Uses fewer threads. | Blocking handlers damage performance. |
| Centralizes event dispatch. | Debugging async flow is harder. |
| Fits non-blocking I/O. | Requires careful backpressure. |

## 12. Real-World Identification Example

Scenario: A chat server needs to handle 100,000 connected clients.

Reactor fit:

- Clients are mostly idle.
- Event loop waits for socket readiness.
- Handlers process messages quickly.
- Heavy processing is dispatched to worker pool.

Without it:

- One thread per client is too expensive.

## 13. MAANG Interview Triggers

Use Reactor when you hear:

- "Many concurrent connections."
- "Non-blocking I/O."
- "Event loop."
- "Selector."
- "Netty."
- "Avoid one thread per socket."

Strong answer keywords:

- event loop
- selector
- readiness
- dispatcher
- handler
- non-blocking
- backpressure
- worker pool

## 14. Common Mistakes

### Mistake 1: Blocking inside handler

- Why it is wrong: one slow handler blocks many connections.
- Better approach: offload blocking work to a worker pool.

### Mistake 2: No backpressure

- Why it is wrong: writes and reads can grow buffers without bound.
- Better approach: pause reads, bound buffers, and track pending writes.

### Mistake 3: Treating event loop like thread pool

- Why it is wrong: event loop is for readiness and quick dispatch.
- Better approach: keep event-loop tasks short.

### Mistake 4: Sharing mutable state casually

- Why it is wrong: handler state can still race when dispatched across threads.
- Better approach: isolate state or synchronize carefully.

## 15. Reactor vs Similar Patterns

| Pattern | Difference |
|---|---|
| Reactor | Handles readiness events and dispatches handlers. |
| Proactor | Handles completion events after async operation finishes. |
| Thread Pool | Executes tasks with reusable workers. |
| Event-Based Asynchronous | General async event notification style. |
| Observer | Subscribers react to events, but not necessarily I/O readiness. |

## 16. Reactor Design Checklist

- What channels are registered?
- What events are handled?
- Are handlers non-blocking?
- What work goes to worker pool?
- How is backpressure applied?
- How are errors handled?
- How many event loops are needed?
- What metrics show event-loop lag?

## 17. Quick Revision Notes

- One-line summary: Reactor dispatches many non-blocking I/O events through an event loop.
- Three keywords: selector, handler, event loop.
- Interview trap: blocking the event loop.
- Memory trick: one loop watches many doors.

## 18. Mini Exercise

Design a Reactor-based notification server.

Answer these:

1. What channels are registered?
2. What events are handled?
3. Which work must not run on event loop?
4. How is slow client backpressure handled?
5. What metric shows event-loop health?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/reactor/README.md](../../github-repo/reactor/README.md)
- [github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/NioReactor.java](../../github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/NioReactor.java)
- [github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/Dispatcher.java](../../github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/Dispatcher.java)
- [github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/ThreadPoolDispatcher.java](../../github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/ThreadPoolDispatcher.java)
- [github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/ChannelHandler.java](../../github-repo/reactor/src/main/java/com/iluwatar/reactor/framework/ChannelHandler.java)
- [github-repo/reactor/src/main/java/com/iluwatar/reactor/app/LoggingHandler.java](../../github-repo/reactor/src/main/java/com/iluwatar/reactor/app/LoggingHandler.java)
- [github-repo/reactor/src/main/java/com/iluwatar/reactor/app/App.java](../../github-repo/reactor/src/main/java/com/iluwatar/reactor/app/App.java)
