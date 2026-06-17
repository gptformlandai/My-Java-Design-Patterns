# Domain Model Pattern

Category: Architecture and Application Structure Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/domain-model](../../github-repo/domain-model)

## How to Study This Page

Use this page in three passes:

1. First pass: understand domain objects containing both state and behavior.
2. Second pass: rewrite the Java example and identify entity, value object, behavior, invariant, and repository/DAO.
3. Third pass: compare Domain Model with Transaction Script, Anemic Domain Model, Active Record, and Service Layer.

By the end, you should be able to say:

> Domain Model represents business concepts as objects that own both data and behavior, so business rules live close to the concepts they protect.

## 1. Technical Definition

Domain Model is an application structure pattern where the core business domain is represented through interconnected objects that encapsulate state, behavior, and rules.

Core idea:

- Domain objects represent real business concepts.
- Behavior lives with the data it protects.
- Invariants are enforced inside the model.
- Services orchestrate but do not own every rule.
- Persistence is kept separate when possible.

### 30-Second Interview Answer

I would use Domain Model when business logic is rich enough that simple CRUD scripts become hard to maintain. Objects like `Order`, `Account`, or `Subscription` own behavior such as `cancel`, `renew`, or `reserveInventory`, and enforce invariants internally. This improves maintainability and expressiveness. The trade-off is that mapping and persistence can become more complex than in a simple transaction script.

## 2. Layman and Easy to Understand Definition

Domain Model is like giving business objects responsibility for their own rules.

An order knows how to cancel itself safely. A subscription knows when it can renew. An account knows whether a withdrawal is allowed.

In code:

- Data and behavior stay together.
- Objects protect their own valid state.
- Services coordinate instead of micromanaging every rule.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without a Domain Model, business logic often becomes procedural:

```java
if (order.status.equals("SHIPPED")) {
    throw new IllegalStateException("cannot cancel");
}
order.status = "CANCELLED";
```

Problems:

- Rules are duplicated across services.
- Objects become data bags.
- State changes bypass invariants.
- Business language disappears from code.
- New rules are hard to place.

### 3.2 The Domain Model Solution

Move rules into domain objects:

```java
order.cancel();
```

The object decides whether the operation is valid.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Entity | Object with identity and lifecycle. |
| Value object | Immutable object identified by value. |
| Aggregate | Consistency boundary around related objects. |
| Repository/DAO | Loads and saves domain objects. |
| Domain service | Domain operation that does not fit one entity. |
| Application service | Coordinates use cases around the model. |

### 3.4 Rich vs Anemic Model

Anemic:

```java
order.setStatus(CANCELLED);
```

Rich:

```java
order.cancel();
```

The rich model names the business operation and protects its rules.

## 4. Java Coding Example

This example models an order with business behavior.

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

record Money(BigDecimal amount) {
    Money add(Money other) {
        return new Money(amount.add(other.amount()));
    }

    Money multiply(int quantity) {
        return new Money(amount.multiply(BigDecimal.valueOf(quantity)));
    }
}

record OrderLine(String productId, Money price, int quantity) {
    Money total() {
        return price.multiply(quantity);
    }
}

final class Order {
    enum Status {
        DRAFT, PLACED, CANCELLED
    }

    private final String id;
    private final List<OrderLine> lines = new ArrayList<>();
    private Status status = Status.DRAFT;

    Order(String id) {
        this.id = id;
    }

    void addLine(String productId, Money price, int quantity) {
        if (status != Status.DRAFT) {
            throw new IllegalStateException("Only draft orders can be changed");
        }
        if (quantity <= 0) {
            throw new IllegalArgumentException("quantity must be positive");
        }
        lines.add(new OrderLine(productId, price, quantity));
    }

    void place() {
        if (lines.isEmpty()) {
            throw new IllegalStateException("Cannot place empty order");
        }
        status = Status.PLACED;
    }

    void cancel() {
        if (status != Status.PLACED) {
            throw new IllegalStateException("Only placed orders can be cancelled");
        }
        status = Status.CANCELLED;
    }

    Money total() {
        return lines.stream()
            .map(OrderLine::total)
            .reduce(new Money(BigDecimal.ZERO), Money::add);
    }
}
```

### Java Block by Block Explanation

`Money` is a value object.

`OrderLine` is part of the order model and knows its own total.

`Order` is an entity with identity and lifecycle.

`addLine`, `place`, and `cancel` are business operations, not simple setters.

Rules such as "cannot place empty order" live inside the model.

### Java Usage

Use Domain Model when:

- Business rules are non-trivial.
- State transitions matter.
- You want expressive business operations.
- Duplicate procedural logic is appearing in services.
- Domain language matters to the team.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class Money:
    amount: Decimal

    def multiply(self, quantity):
        return Money(self.amount * quantity)


@dataclass(frozen=True)
class OrderLine:
    product_id: str
    price: Money
    quantity: int

    def total(self):
        return self.price.multiply(self.quantity)


class Order:
    def __init__(self, order_id):
        self.id = order_id
        self.lines = []
        self.status = "DRAFT"

    def add_line(self, product_id, price, quantity):
        if self.status != "DRAFT":
            raise ValueError("Only draft orders can be changed")
        if quantity <= 0:
            raise ValueError("quantity must be positive")
        self.lines.append(OrderLine(product_id, price, quantity))

    def place(self):
        if not self.lines:
            raise ValueError("Cannot place empty order")
        self.status = "PLACED"

    def cancel(self):
        if self.status != "PLACED":
            raise ValueError("Only placed orders can be cancelled")
        self.status = "CANCELLED"
```

### Python Usage

In Python:

- Use domain classes for behavior-heavy concepts.
- Use dataclasses for value objects.
- Keep ORM concerns from dominating domain behavior when possible.
- Let services coordinate, not own every rule.

## 6. Where It Comes Handy in Real Life

- Orders and returns.
- Bank accounts.
- Subscriptions.
- Insurance claims.
- Booking/reservation systems.
- Inventory allocation.
- Loan approval.
- Pricing and discount rules.

## 7. Advantages Over Normal Code Without Pattern

### Without Domain Model

```java
service checks status, updates fields, calculates totals, repeats rules
```

Problems:

- Business logic is scattered.
- Entities are passive data holders.
- Rules are duplicated.
- Invalid state is easier to create.

### With Domain Model

```java
order.place();
order.cancel();
```

Benefits:

- Business language appears in code.
- Rules live near data.
- Invariants are protected.
- Services become simpler coordinators.

## 8. Where It Excels

- Rich business domains.
- Complex state transitions.
- Long-lived enterprise products.
- DDD-style designs.
- Systems where correctness matters.
- Codebases where business experts and engineers share vocabulary.

## 9. Where It Fails

- Simple CRUD tables.
- Data import/export jobs.
- Reporting-only services.
- Domains with almost no behavior.
- Teams that over-model every noun.
- ORM constraints that make domain behavior awkward.

## 10. Prebuilt Frameworks and Packages

### Java

- JPA/Hibernate, with care around entity behavior.
- Spring Data repositories.
- jMolecules for DDD annotations/conventions.
- Vavr for value-style modeling.
- ArchUnit for dependency rules.

### Python

- Dataclasses and value objects.
- SQLAlchemy classical mapping.
- Django models, with care around fat models.
- Pydantic for boundary DTOs, not necessarily domain objects.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps rules close to data. | More modeling effort than CRUD scripts. |
| Expressive business language. | Persistence mapping can be harder. |
| Protects invariants. | Can be overused in simple domains. |
| Reduces duplicated service logic. | Needs discipline around object boundaries. |
| Supports DDD thinking. | Poor models can become tangled object graphs. |

## 12. Real-World Identification Example

Scenario:

You are building subscription billing.

Domain Model fit:

- `Subscription.renew()`
- `Subscription.cancel()`
- `Subscription.pause()`
- `BillingPeriod`
- `Money`

What would go wrong without it:

- Services duplicate status checks.
- Invalid transitions are possible.
- Billing rules scatter across controllers/jobs.

## 13. MAANG Interview Triggers

Use Domain Model when you hear:

- "Rich business logic."
- "Business rules should be centralized."
- "Avoid anemic domain."
- "State transitions."
- "Domain-driven design."
- "Entities and value objects."
- "Invariants."

### Interview-Ready Answer Format

1. Identify core business concepts.
2. Model entities and value objects.
3. Put behavior and invariants in domain objects.
4. Use repositories to persist/load objects.
5. Use services for orchestration.
6. Mention avoiding over-modeling for simple CRUD.

## 14. Common Mistakes

### Mistake 1: Data Bags with Getters and Setters Only

That is an anemic model, not a rich domain model.

### Mistake 2: Putting Every Rule in Services

If the rule protects an object's state, the object should likely own it.

### Mistake 3: ORM-Driven Design

Tables should not be the only force shaping the domain model.

### Mistake 4: Huge Aggregate Boundaries

Large aggregates create locking, loading, and complexity problems.

### Mistake 5: Over-Modeling Simple CRUD

Not every app needs rich domain objects.

## 15. Domain Model vs Similar Patterns

| Pattern | Difference |
|---|---|
| Domain Model | Rich objects own data and behavior. |
| Transaction Script | Procedures own business logic step by step. |
| Active Record | Object combines domain data and persistence methods. |
| Service Layer | Coordinates use cases around domain objects. |
| Anemic Domain Model | Data-only objects with logic elsewhere. |

## 16. Domain Model Design Checklist

| Question | Why it matters |
|---|---|
| What are the core concepts? | Defines model vocabulary. |
| What invariants must always hold? | Drives behavior placement. |
| Which objects have identity? | Identifies entities. |
| Which objects are values? | Encourages immutability. |
| What is the aggregate boundary? | Controls consistency scope. |
| Which logic belongs in service? | Avoids bloated entities. |

## 17. Quick Revision Notes

- Domain Model is data plus behavior.
- Entities have identity.
- Value objects are defined by value.
- Invariants live inside the model.
- Services coordinate use cases.
- Avoid for simple CRUD.

## 18. Mini Exercise

Design Domain Model for `HotelBooking`.

Objects:

- `Booking`
- `Room`
- `Guest`
- `DateRange`
- `Money`

Behavior:

- `confirm()`
- `cancel()`
- `changeDates()`
- `calculatePrice()`

## 19. Source Reference in This Repo

The repository's Domain Model implementation uses customer and product domain objects with behavior plus DAO implementations for persistence.

Useful files:

- [github-repo/domain-model/README.md](../../github-repo/domain-model/README.md)
- [github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/Customer.java](../../github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/Customer.java)
- [github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/Product.java](../../github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/Product.java)
- [github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/CustomerDao.java](../../github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/CustomerDao.java)
- [github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/ProductDao.java](../../github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/ProductDao.java)
- [github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/App.java](../../github-repo/domain-model/src/main/java/com/iluwatar/domainmodel/App.java)

