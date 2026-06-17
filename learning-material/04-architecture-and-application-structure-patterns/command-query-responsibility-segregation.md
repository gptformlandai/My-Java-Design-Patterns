# Command Query Responsibility Segregation Pattern

Category: Architecture and Application Structure Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/command-query-responsibility-segregation](../../github-repo/command-query-responsibility-segregation)

## How to Study This Page

Use this page in three passes:

1. First pass: understand that commands change state and queries read state.
2. Second pass: rewrite the Java example and identify command model, query model, command service, and query service.
3. Third pass: study consistency, read-model lag, event sourcing, and when CQRS is too much.

By the end, you should be able to say:

> CQRS separates write operations from read operations so each side can be modeled, optimized, scaled, and secured independently.

## 1. Technical Definition

Command Query Responsibility Segregation is an architecture pattern that separates operations that mutate state from operations that read state, often using different models, services, or data stores for each side.

Core idea:

- Commands express intent to change state.
- Queries return data without changing state.
- Write model protects invariants.
- Read model is optimized for lookup/display.
- Synchronization may be immediate or eventually consistent.

### 30-Second Interview Answer

I would use CQRS when reads and writes have different scale, shape, or consistency needs. Commands go through a write model that validates business rules and changes state. Queries use a read model optimized for fast retrieval, often denormalized. The benefit is independent optimization and clearer responsibility. The cost is synchronization complexity and possible read-after-write inconsistency.

## 2. Layman and Easy to Understand Definition

CQRS is like having two counters in an office.

One counter handles requests that change records, like submitting forms. Another counter handles lookups, like asking for status. Each counter is optimized for its own work.

In code:

- Command side writes.
- Query side reads.
- They may share a database or use separate stores.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Many applications use one model for everything:

```java
Order order = orderRepository.findById(id);
order.changeAddress(address);
orderRepository.save(order);
return orderRepository.findDashboardRow(id);
```

This becomes awkward when:

- Writes need rich domain validation.
- Reads need denormalized projections.
- Read volume is much higher than write volume.
- Different teams optimize different paths.
- Security rules differ for read and write operations.

### 3.2 The CQRS Solution

Separate the two sides:

```java
commandService.placeOrder(command);
OrderSummary summary = queryService.getOrderSummary(orderId);
```

Commands and queries become different APIs.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Command | Intent to change state. |
| Command handler/service | Validates and applies command. |
| Write model | Model optimized for invariants and updates. |
| Query | Request for data. |
| Query service | Reads data without mutation. |
| Read model | Model optimized for display/search/reporting. |

### 3.4 Consistency Model

CQRS can be implemented in levels:

1. Same database, separate services/classes.
2. Same database, separate read/write schemas or views.
3. Separate write and read stores.
4. Event-sourced write model with projections.

The more separation you add, the more you must handle synchronization.

## 4. Java Coding Example

This example separates account writes from account reads.

```java
import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;

record DepositCommand(String accountId, BigDecimal amount) {
}

record AccountSummary(String accountId, BigDecimal balance) {
}

final class Account {
    private final String id;
    private BigDecimal balance;

    Account(String id, BigDecimal balance) {
        this.id = id;
        this.balance = balance;
    }

    void deposit(BigDecimal amount) {
        if (amount.signum() <= 0) {
            throw new IllegalArgumentException("amount must be positive");
        }
        balance = balance.add(amount);
    }

    BigDecimal balance() {
        return balance;
    }
}

final class AccountWriteRepository {
    private final Map<String, Account> accounts = new HashMap<>();

    Account load(String accountId) {
        return accounts.computeIfAbsent(accountId, id -> new Account(id, BigDecimal.ZERO));
    }

    void save(String accountId, Account account) {
        accounts.put(accountId, account);
    }
}

final class AccountReadRepository {
    private final Map<String, AccountSummary> summaries = new HashMap<>();

    void update(AccountSummary summary) {
        summaries.put(summary.accountId(), summary);
    }

    AccountSummary getSummary(String accountId) {
        return summaries.get(accountId);
    }
}

final class AccountCommandService {
    private final AccountWriteRepository writeRepository;
    private final AccountReadRepository readRepository;

    AccountCommandService(AccountWriteRepository writeRepository, AccountReadRepository readRepository) {
        this.writeRepository = writeRepository;
        this.readRepository = readRepository;
    }

    void deposit(DepositCommand command) {
        Account account = writeRepository.load(command.accountId());
        account.deposit(command.amount());
        writeRepository.save(command.accountId(), account);
        readRepository.update(new AccountSummary(command.accountId(), account.balance()));
    }
}

final class AccountQueryService {
    private final AccountReadRepository readRepository;

    AccountQueryService(AccountReadRepository readRepository) {
        this.readRepository = readRepository;
    }

    AccountSummary getAccountSummary(String accountId) {
        return readRepository.getSummary(accountId);
    }
}
```

### Java Block by Block Explanation

`DepositCommand` represents write intent.

`Account` is the write model and protects deposit rules.

`AccountSummary` is the read model and is shaped for display.

`AccountCommandService` changes state and updates the read model.

`AccountQueryService` only reads.

### Java Usage

Use CQRS in Java when:

- Read and write paths are meaningfully different.
- Read traffic is much higher than write traffic.
- You need optimized projections.
- Audit/event sourcing is part of the design.
- Command validation is complex.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class DepositCommand:
    account_id: str
    amount: Decimal


@dataclass(frozen=True)
class AccountSummary:
    account_id: str
    balance: Decimal


class Account:
    def __init__(self, account_id):
        self.account_id = account_id
        self.balance = Decimal("0")

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("amount must be positive")
        self.balance += amount


class AccountCommandService:
    def __init__(self, write_store, read_store):
        self.write_store = write_store
        self.read_store = read_store

    def deposit(self, command):
        account = self.write_store.setdefault(command.account_id, Account(command.account_id))
        account.deposit(command.amount)
        self.read_store[command.account_id] = AccountSummary(command.account_id, account.balance)


class AccountQueryService:
    def __init__(self, read_store):
        self.read_store = read_store

    def get_summary(self, account_id):
        return self.read_store.get(account_id)
```

### Python Usage

In Python systems, CQRS usually appears as:

- Separate command handlers and query handlers.
- Separate write models and read DTOs.
- Projection updates from events.
- FastAPI/Django endpoints split by intent.

## 6. Where It Comes Handy in Real Life

- Banking ledgers.
- E-commerce order history.
- Social feeds.
- Analytics dashboards.
- Inventory reservation.
- Event-sourced systems.
- High-read systems with denormalized projections.
- Systems with strict write invariants.

## 7. Advantages Over Normal Code Without Pattern

### Without CQRS

```java
Order order = repository.findOrder(id);
return order.toDashboardDto();
```

Problems:

- Write model gets polluted with read-only view needs.
- Queries become slow joins.
- Reads and writes scale together.
- Security and validation boundaries blur.

### With CQRS

```java
commandService.placeOrder(command);
queryService.getOrderSummary(id);
```

Benefits:

- Write side protects rules.
- Read side is optimized for user screens.
- Teams can scale and tune sides independently.
- Command and query intent is explicit.

## 8. Where It Excels

- Complex business writes.
- High read/write imbalance.
- Denormalized read views.
- Event-driven architectures.
- Audit-heavy systems.
- Systems where commands map to business actions.

## 9. Where It Fails

- Simple CRUD apps.
- Small teams without operational maturity.
- Strong immediate read-after-write requirements with separate stores.
- Systems where read and write models are identical.
- Cases where extra services add confusion.

## 10. Prebuilt Frameworks and Packages

### Java

- Axon Framework.
- Spring Boot command/query handlers.
- Kafka for projection updates.
- JPA/Hibernate for write model persistence.
- Elasticsearch/OpenSearch for read models.
- Debezium for change data capture.

### Python

- FastAPI/Django command/query modules.
- Celery for asynchronous projection updates.
- SQLAlchemy for write stores.
- Elasticsearch/OpenSearch clients.
- Kafka libraries for event pipelines.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Optimizes reads and writes independently. | Adds architectural complexity. |
| Keeps write model focused on invariants. | Read model synchronization is hard. |
| Supports denormalized projections. | Eventual consistency can surprise users. |
| Scales high-read workloads well. | More moving parts to operate. |
| Pairs well with event sourcing. | Overkill for simple CRUD. |

## 12. Real-World Identification Example

Scenario:

You are designing order management for an e-commerce app.

CQRS fit:

- `PlaceOrderCommand` validates inventory and payment.
- Write model stores canonical order state.
- Read model stores `OrderSummaryView`.
- Customer support dashboard reads from projection.

What would go wrong without it:

- Dashboard queries hit transactional tables.
- Order aggregate gets filled with display-only fields.
- Read scale affects write performance.

## 13. MAANG Interview Triggers

Use CQRS when you hear:

- "Separate reads and writes."
- "Read model and write model."
- "Eventual consistency is acceptable."
- "Dashboard queries are too slow."
- "Command handlers."
- "Event sourcing."
- "Scale reads independently."

### Interview-Ready Answer Format

1. Define commands and queries separately.
2. Keep writes in a model that enforces invariants.
3. Build read projections optimized for screens.
4. Explain projection update flow.
5. Discuss consistency and stale reads.
6. Mention fallback for simple CRUD: normal layered service.

## 14. Common Mistakes

### Mistake 1: CQRS for Every CRUD Screen

Use it when read/write models genuinely diverge.

### Mistake 2: Ignoring Projection Lag

Users may read stale data after writes. Design UX and consistency rules.

### Mistake 3: Commands Return Complex Read Models

Commands should usually return acknowledgement or identifiers, not query-shaped views.

### Mistake 4: No Idempotency

Command handlers often need idempotency for retries.

### Mistake 5: Treating CQRS as Always Requiring Two Databases

CQRS can start as separate classes/services over the same database.

## 15. CQRS vs Similar Patterns

| Pattern | Difference |
|---|---|
| CQRS | Separates reads and writes. |
| Event Sourcing | Stores events as source of truth; often paired with CQRS. |
| Repository | Abstracts persistence, can exist on both sides. |
| Service Layer | Provides application API; may contain command/query services. |
| Clean Architecture | Boundary/dependency style that can include CQRS use cases. |

## 16. CQRS Design Checklist

| Question | Why it matters |
|---|---|
| Do reads and writes differ? | Validates CQRS fit. |
| What are commands? | Defines business write intent. |
| What read models are needed? | Optimizes screens and reports. |
| How are projections updated? | Determines consistency behavior. |
| What is acceptable staleness? | Sets user/system expectations. |
| Are commands idempotent? | Handles retries safely. |

## 17. Quick Revision Notes

- Commands mutate state.
- Queries read state.
- Write model protects invariants.
- Read model optimizes display/search.
- Eventual consistency is common.
- Do not use for simple CRUD by default.

## 18. Mini Exercise

Design CQRS for `CourseEnrollment`.

Commands:

- `EnrollStudent`
- `CancelEnrollment`
- `CompleteLesson`

Queries:

- `getStudentProgress`
- `getCourseRoster`
- `getInstructorDashboard`

Think through:

- Which read models are denormalized?
- What happens right after enrollment?

## 19. Source Reference in This Repo

The repository's CQRS implementation separates command and query services for authors and books.

Useful files:

- [github-repo/command-query-responsibility-segregation/README.md](../../github-repo/command-query-responsibility-segregation/README.md)
- [github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/commandes/CommandService.java](../../github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/commandes/CommandService.java)
- [github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/commandes/CommandServiceImpl.java](../../github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/commandes/CommandServiceImpl.java)
- [github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/queries/QueryService.java](../../github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/queries/QueryService.java)
- [github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/queries/QueryServiceImpl.java](../../github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/queries/QueryServiceImpl.java)
- [github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/app/App.java](../../github-repo/command-query-responsibility-segregation/src/main/java/com/iluwatar/cqrs/app/App.java)

