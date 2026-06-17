# Event Sourcing Pattern

Category: Messaging, Eventing, and Reactive Flow Patterns  
MAANG interview meter: High  
Software usage meter: Medium  
Repository module: [event-sourcing](../../github-repo/event-sourcing)

---

## How to Study This Page

Study Event Sourcing as a persistence pattern, not just a messaging pattern.

The core idea:
- do not store only the latest state
- store every state-changing event
- rebuild current state by replaying events

---

## 1. Technical Definition

Event Sourcing is a persistence pattern where every state change is stored as an immutable event in an append-only event log, and current state is derived by replaying those events.

### 30-Second Interview Answer

Event Sourcing stores the history of changes instead of only the current row. For an account, we store events such as `AccountCreated`, `MoneyDeposited`, and `MoneyTransferred`; the balance is rebuilt by replaying them. I would use it when auditability, replay, temporal queries, and traceability are core requirements. The trade-offs are event versioning, storage growth, replay cost, eventual consistency, and operational complexity.

---

## 2. Layman and Easy to Understand Definition

Imagine a bank statement. The bank does not only remember your current balance. It stores every deposit, withdrawal, and transfer.

Your current balance is the result of replaying that history.

That is Event Sourcing.

---

## 3. Bit by Bit Explanation of What It Is

### Normal Persistence

Typical CRUD stores current state:

```text
Account(id=1, balance=900)
```

When a deposit happens, the row is overwritten.

### Event-Sourced Persistence

Event sourcing stores facts:

```text
AccountCreated(id=1)
MoneyDeposited(id=1, amount=1000)
MoneyWithdrawn(id=1, amount=100)
```

The current balance is computed from the events.

### Event Sourcing Flow

1. Command requests a business action.
2. Domain validates whether the action is allowed.
3. Domain emits one or more events.
4. Events are appended to the event store.
5. Events are applied to update current in-memory/projected state.
6. Read models can be built from the event stream.
7. On recovery, events are replayed to rebuild state.

### Core Participants

| Participant | Responsibility |
|---|---|
| Command | Request to do something |
| Domain event | Fact that something happened |
| Event store/journal | Append-only system of record |
| Aggregate | Rebuilds state from events |
| Projection/read model | Query-friendly view built from events |
| Snapshot | Optional checkpoint to reduce replay cost |

---

## 4. Java Coding Example

```java
import java.util.ArrayList;
import java.util.List;

sealed interface AccountEvent permits AccountCreated, MoneyDeposited {}
record AccountCreated(String accountId) implements AccountEvent {}
record MoneyDeposited(String accountId, int amount) implements AccountEvent {}

class Account {
    private int balance;

    public void apply(AccountEvent event) {
        if (event instanceof AccountCreated) {
            balance = 0;
        } else if (event instanceof MoneyDeposited deposit) {
            balance += deposit.amount();
        }
    }

    public int balance() {
        return balance;
    }
}

public class EventSourcingDemo {
    public static void main(String[] args) {
        List<AccountEvent> eventStore = new ArrayList<>();
        eventStore.add(new AccountCreated("A-1"));
        eventStore.add(new MoneyDeposited("A-1", 100));
        eventStore.add(new MoneyDeposited("A-1", 50));

        Account account = new Account();
        for (AccountEvent event : eventStore) {
            account.apply(event);
        }

        System.out.println(account.balance());
    }
}
```

### Java Block by Block

`AccountEvent` is the base type for stored facts.

The event store is append-only in concept.

`Account.apply` rebuilds current state from each event.

The account balance is not stored as the only truth; it is derived from history.

---

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class AccountCreated:
    account_id: str


@dataclass(frozen=True)
class MoneyDeposited:
    account_id: str
    amount: int


class Account:
    def __init__(self):
        self.balance = 0

    def apply(self, event):
        if isinstance(event, AccountCreated):
            self.balance = 0
        elif isinstance(event, MoneyDeposited):
            self.balance += event.amount


events = [
    AccountCreated("A-1"),
    MoneyDeposited("A-1", 100),
    MoneyDeposited("A-1", 50),
]

account = Account()
for event in events:
    account.apply(event)

print(account.balance)
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Banking ledger | Complete audit trail |
| Order lifecycle | Every transition is preserved |
| Inventory movement | Rebuild stock from movements |
| Compliance systems | Historical state is explainable |
| Collaboration tools | Replay document changes |
| Debugging production issues | Reproduce exact sequence of events |

---

## 7. Advantages Over Normal Code Without Pattern

Without Event Sourcing:
- old state is overwritten
- audit history must be bolted on
- reconstructing past state is hard
- debugging "how did we get here?" is painful

With Event Sourcing:
- every change is recorded
- state can be rebuilt
- history is queryable
- new read models can be created by replaying events

---

## 8. Where It Excels

It excels when:
- audit trail is mandatory
- business history matters
- events are natural domain language
- read models need to be rebuilt
- compensating actions are needed
- temporal queries are valuable

---

## 9. Where It Fails

It fails when:
- the domain is simple CRUD
- the team does not need full history
- event versioning is ignored
- replay time becomes too expensive
- external side effects happen during replay
- event store consistency is weak

For production, side effects must be separated from deterministic state replay.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Axon Framework, EventStoreDB Java client |
| Event stores | EventStoreDB, Kafka, PostgreSQL append tables |
| CQRS | Axon, Lagom, custom projections |
| Serialization | Avro, Protobuf, JSON Schema |
| Cloud | DynamoDB streams, Kafka-compatible platforms |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Full audit history | More complex than CRUD |
| State can be replayed | Event versioning is hard |
| Great for debugging | Replay can be expensive |
| Enables new projections | Storage grows continuously |
| Natural for ledger domains | Eventual consistency in read models |

---

## 12. Real-World Identification Example

Question:

> You are designing a wallet system. Compliance requires a complete immutable history of every balance change and the ability to reconstruct historical balances. What pattern fits?

Strong answer:

Use Event Sourcing. Store immutable events such as `WalletCreated`, `MoneyAdded`, `MoneyReserved`, and `MoneyCaptured` in an append-only event store. Rebuild wallet state by replaying events. Use snapshots for long streams, version event schemas, keep external side effects outside replay, and build read models for fast queries.

---

## 13. MAANG Interview Triggers

Say Event Sourcing when you hear:
- audit log is system of record
- reconstruct state at any time
- immutable history
- replay events
- ledger
- temporal queries
- rebuild projections

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Calling external APIs during replay | Replay repeats side effects | Separate side effects from state application |
| No event versioning | Old events become unreadable | Version and migrate events |
| Storing only deltas without meaning | Hard to understand history | Store business facts |
| No snapshots for long streams | Recovery becomes slow | Add snapshots carefully |
| Updating state before durable append | Crash can lose facts | Append atomically before or with state change |

---

## 15. Event Sourcing vs Similar Patterns

| Pattern | Difference |
|---|---|
| Event-Driven Architecture | EDA is communication style; event sourcing is persistence style |
| CQRS | CQRS separates reads/writes; often paired with event sourcing |
| Audit Log | Audit log records history but may not rebuild state |
| Transaction Log | Low-level storage log; event sourcing stores business events |
| Snapshot | Snapshot optimizes event sourcing but is not the source of truth |

---

## 16. Event Sourcing Design Checklist

- What events represent business facts?
- What is the aggregate boundary?
- How are event streams keyed?
- What consistency guarantee does append need?
- How are event schemas versioned?
- How are projections rebuilt?
- When are snapshots created?
- Which side effects must not run during replay?

---

## 17. Quick Revision Notes

- One-line summary: Store every state change as an immutable event.
- Memory hook: "current state equals replayed history."
- Best for: audit-heavy domains and ledgers.
- Avoid when: simple CRUD is enough.
- Interview line: "I would append immutable domain events, rebuild aggregates by replay, create projections for reads, and handle versioning, snapshots, and side effects carefully."

---

## 18. Mini Exercise

Design event sourcing for an order aggregate:
- define events for create, pay, ship, cancel
- decide invalid transitions
- define one read model
- define one snapshot rule
- explain how replay avoids sending duplicate emails

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/event-sourcing/README.md)
- [DomainEvent.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/event/DomainEvent.java)
- [AccountCreateEvent.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/event/AccountCreateEvent.java)
- [MoneyDepositEvent.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/event/MoneyDepositEvent.java)
- [MoneyTransferEvent.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/event/MoneyTransferEvent.java)
- [DomainEventProcessor.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/processor/DomainEventProcessor.java)
- [JsonFileJournal.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/processor/JsonFileJournal.java)
- [AccountAggregate.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/state/AccountAggregate.java)
- [App.java](../../github-repo/event-sourcing/src/main/java/com/iluwatar/event/sourcing/app/App.java)

The repo implementation records account events in a JSON journal and recovers in-memory account state by replaying stored events through `DomainEventProcessor.recover()`.

