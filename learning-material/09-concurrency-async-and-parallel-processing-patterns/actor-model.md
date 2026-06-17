# Actor Model Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/actor-model](../../github-repo/actor-model)

## How to Study This Page

Use this page in three passes:

1. First pass: understand actor state isolation and message passing.
2. Second pass: trace send, mailbox, receive, state update, and reply.
3. Third pass: compare Actor Model with Active Object, Monitor, and Producer-Consumer.

By the end, you should be able to say:

> Actor Model builds concurrent systems from isolated actors that communicate through asynchronous messages.

## 1. Technical Definition

Actor Model is a concurrency model where independent actors own their state, process messages from a mailbox, and communicate only by sending messages.

Core idea:

- Actors do not share mutable state.
- Each actor has a mailbox.
- Messages are processed one at a time per actor.
- Actors can send messages to other actors.
- Actors can create child actors or change behavior.

### 30-Second Interview Answer

I would use Actor Model when I want high concurrency without shared mutable state. Each actor owns state and processes messages serially from its mailbox, which reduces locking. The trade-off is asynchronous debugging, message protocol design, supervision, and mailbox backpressure.

## 2. Layman and Easy to Understand Definition

Actors are independent workers with inboxes.

They do not reach into each other's notebooks. They send messages, read one message at a time, and update only their own notes.

In code:

- Actor receives message.
- Actor handles message.
- Actor updates its own state.
- Actor sends message to another actor if needed.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Shared mutable state creates races:

```text
thread A updates object
thread B updates same object
state becomes inconsistent
```

### 3.2 The Actor Solution

Avoid shared state:

```text
actor A --message--> actor B
actor B processes mailbox one message at a time
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Actor | Isolated component with state and behavior. |
| Mailbox | Queue of incoming messages. |
| Message | Immutable command or event. |
| Actor system | Starts, routes, and supervises actors. |
| Supervisor | Handles actor failure and restart strategy. |

### 3.4 Runtime Flow

1. Sender creates message.
2. Message is appended to actor mailbox.
3. Actor takes next message.
4. Actor updates private state.
5. Actor may send more messages.
6. Actor returns to mailbox loop.

## 4. Java Coding Example

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

record ActorMessage(String text) {
}

abstract class Actor implements Runnable {
    private final BlockingQueue<ActorMessage> mailbox = new LinkedBlockingQueue<>();

    void send(ActorMessage message) {
        mailbox.add(message);
    }

    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                onReceive(mailbox.take());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    protected abstract void onReceive(ActorMessage message);
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `mailbox` | Actor input queue. |
| `send` | Async message delivery. |
| `run` | Actor event loop. |
| `mailbox.take` | Actor processes one message at a time. |
| `onReceive` | Actor behavior. |

### Java Usage

```java
Actor printer = new Actor() {
    protected void onReceive(ActorMessage message) {
        System.out.println(message.text());
    }
};

new Thread(printer).start();
printer.send(new ActorMessage("hello"));
```

## 5. Python Coding Example

```python
from queue import Queue
from threading import Thread


class Actor:
    def __init__(self):
        self.mailbox = Queue()
        self.thread = Thread(target=self._run, daemon=True)
        self.thread.start()

    def send(self, message):
        self.mailbox.put(message)

    def _run(self):
        while True:
            message = self.mailbox.get()
            if message is None:
                break
            self.on_receive(message)

    def on_receive(self, message):
        print(message)
```

### Python Usage

Use this shape when explaining:

- Mailbox serializes actor messages.
- Actor owns state.
- Actors communicate by messages, not shared variables.

## 6. Where It Comes Handy in Real Life

- Chat systems.
- Game servers.
- Distributed workflows.
- IoT device state.
- Stream processing.
- Fault-isolated services.
- Concurrent simulations.

## 7. Advantages Over Normal Code Without Pattern

Without Actor Model:

```text
many threads mutate shared state
```

With Actor Model:

```text
each actor owns state and receives messages
```

Benefits:

- Reduces lock usage.
- Encourages isolation.
- Scales message-driven workloads.
- Supports supervision and recovery.
- Fits distributed systems.

## 8. Where It Excels

- State can be partitioned by actor.
- Message-driven flow is natural.
- Isolation matters.
- Failures should be contained.
- Asynchronous communication is acceptable.

## 9. Where It Fails

- Strong synchronous call semantics are required.
- Message protocols are poorly designed.
- Mailboxes grow without bounds.
- State is still shared outside actors.
- Debugging async flows is not supported by tooling.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| JVM | Akka, Apache Pekko, Quasar concepts |
| Other platforms | Erlang/OTP, Elixir, Orleans |
| Java basics | Blocking queues, executors |
| Messaging | Mailbox dispatchers, supervision strategies |
| Observability | Message tracing, mailbox depth metrics |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids shared mutable state. | Async debugging is harder. |
| Scales naturally by partitioning. | Message protocols need design. |
| Supports fault isolation. | Mailboxes can overload. |
| Serializes actor-local state. | More complex than direct calls. |

## 12. Real-World Identification Example

Scenario: Chat app tracks each room's users and messages.

Actor Model fit:

- Each room is an actor.
- Messages to a room enter its mailbox.
- Room actor owns room state.
- Other actors communicate by sending commands.

Without it:

- Many threads mutate room state directly.
- Locks become hard to reason about.

## 13. MAANG Interview Triggers

Use Actor Model when you hear:

- "No shared mutable state."
- "Message passing."
- "Actor mailbox."
- "Fault isolation."
- "Concurrent state per entity."
- "Akka or Erlang style."

Strong answer keywords:

- actor
- mailbox
- message
- isolation
- supervision
- serial processing
- backpressure
- immutable messages

## 14. Common Mistakes

### Mistake 1: Sharing actor state externally

- Why it is wrong: it breaks actor isolation.
- Better approach: access actor state only through messages.

### Mistake 2: Unbounded mailboxes

- Why it is wrong: overload becomes memory growth.
- Better approach: apply mailbox bounds and backpressure.

### Mistake 3: Blocking inside actor

- Why it is wrong: actor cannot process later messages.
- Better approach: offload blocking calls or use async replies.

### Mistake 4: Too many message types without contracts

- Why it is wrong: behavior becomes hard to test.
- Better approach: define clear message protocols.

## 15. Actor Model vs Similar Patterns

| Pattern | Difference |
|---|---|
| Actor Model | Isolated actors communicate through mailboxes. |
| Active Object | Object owns queue/thread but is usually object-centric. |
| Monitor | Protects shared state with locks. |
| Producer-Consumer | Producers and consumers exchange work through a queue. |
| Event Bus | Broadcasts events; actors own state and behavior. |

## 16. Actor Model Design Checklist

- What entity or responsibility becomes an actor?
- What messages are supported?
- Is actor state private?
- Is mailbox bounded?
- How are failures supervised?
- What work must not block actor thread?
- How are replies modeled?
- What metrics track mailbox depth?

## 17. Quick Revision Notes

- One-line summary: Actor Model avoids shared mutable state through isolated message-processing actors.
- Three keywords: actor, mailbox, message.
- Interview trap: sharing actor state directly.
- Memory trick: each actor has its own inbox and notebook.

## 18. Mini Exercise

Design actors for an online auction.

Answer these:

1. What is one actor type?
2. What messages does it accept?
3. What state does it own?
4. How are invalid bids handled?
5. What mailbox metric matters?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/actor-model/README.md](../../github-repo/actor-model/README.md)
- [github-repo/actor-model/src/main/java/com/iluwatar/actormodel/Actor.java](../../github-repo/actor-model/src/main/java/com/iluwatar/actormodel/Actor.java)
- [github-repo/actor-model/src/main/java/com/iluwatar/actormodel/ActorSystem.java](../../github-repo/actor-model/src/main/java/com/iluwatar/actormodel/ActorSystem.java)
- [github-repo/actor-model/src/main/java/com/iluwatar/actormodel/Message.java](../../github-repo/actor-model/src/main/java/com/iluwatar/actormodel/Message.java)
- [github-repo/actor-model/src/main/java/com/iluwatar/actormodel/ExampleActor.java](../../github-repo/actor-model/src/main/java/com/iluwatar/actormodel/ExampleActor.java)
- [github-repo/actor-model/src/main/java/com/iluwatar/actormodel/App.java](../../github-repo/actor-model/src/main/java/com/iluwatar/actormodel/App.java)
