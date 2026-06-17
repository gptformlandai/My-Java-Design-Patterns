# Repository Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/repository](../../github-repo/repository)

## How to Study This Page

Use this page in three passes:

1. First pass: understand Repository as a collection-like boundary around persisted domain objects.
2. Second pass: rewrite the Java example and identify entity, repository interface, query methods, and persistence implementation.
3. Third pass: compare Repository with DAO, Data Mapper, Unit of Work, and Service Layer.

By the end, you should be able to say:

> Repository hides persistence details behind a domain-friendly collection API so business logic can load and save entities without knowing database mechanics.

## 1. Technical Definition

Repository is a data access pattern that mediates between the domain/application layer and data mapping layer using a collection-like interface for accessing aggregate roots or entities.

Core idea:

- Domain/application code asks for domain objects.
- Repository hides SQL/ORM/API details.
- Repository methods are written in domain language.
- Implementations can use JPA, JDBC, files, APIs, or memory.
- Tests can replace real repositories with fake ones.

### 30-Second Interview Answer

I would use Repository when business logic needs to retrieve and persist domain objects without depending on storage details. The repository exposes methods like `findById`, `save`, or domain-specific queries such as `findActiveSubscriptions`. It improves testability and decouples the domain from persistence. The trade-off is extra abstraction, and generic repositories can become too weak if they only mirror CRUD.

## 2. Layman and Easy to Understand Definition

Repository is like a librarian for your application data.

You ask the librarian for a book by title or category. You do not need to know which shelf, storage room, or database table contains it.

In code:

- Caller asks repository for objects.
- Repository knows where/how to fetch them.
- Caller gets domain objects, not SQL rows.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without Repository, application logic often talks directly to storage:

```java
ResultSet rows = statement.executeQuery("select * from orders where user_id = ?");
```

Problems:

- SQL leaks into business code.
- Tests require a real database.
- Switching storage is difficult.
- Query logic is duplicated.
- Domain language is replaced by persistence language.

### 3.2 The Repository Solution

Create a domain-friendly boundary:

```java
List<Order> orders = orderRepository.findOpenOrdersForUser(userId);
```

The caller focuses on intent, not storage mechanics.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Entity/aggregate | Domain object being persisted. |
| Repository interface | Domain-facing data access contract. |
| Repository implementation | Uses database/ORM/API/in-memory storage. |
| Query/specification | Encapsulates search criteria. |
| Application/service layer | Uses repository to execute use cases. |

### 3.4 Repository Shape

Good repository methods often sound like domain questions:

```java
findById(orderId)
findOpenOrdersForUser(userId)
save(order)
delete(order)
```

Weak repository methods often expose storage too directly:

```java
executeSql(String sql)
findByColumn(String column, Object value)
```

## 4. Java Coding Example

This example uses a repository for `Order` objects.

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

record Order(String id, String userId, String status) {
}

interface OrderRepository {
    Optional<Order> findById(String id);
    List<Order> findOpenOrdersForUser(String userId);
    void save(Order order);
}

final class InMemoryOrderRepository implements OrderRepository {
    private final Map<String, Order> orders = new HashMap<>();

    @Override
    public Optional<Order> findById(String id) {
        return Optional.ofNullable(orders.get(id));
    }

    @Override
    public List<Order> findOpenOrdersForUser(String userId) {
        return orders.values().stream()
            .filter(order -> order.userId().equals(userId))
            .filter(order -> order.status().equals("OPEN"))
            .toList();
    }

    @Override
    public void save(Order order) {
        orders.put(order.id(), order);
    }
}

final class OrderService {
    private final OrderRepository orderRepository;

    OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    List<Order> openOrders(String userId) {
        return orderRepository.findOpenOrdersForUser(userId);
    }
}
```

### Java Block by Block Explanation

`Order` is the domain object.

`OrderRepository` is the boundary used by application code.

`InMemoryOrderRepository` is one implementation. A JPA or JDBC implementation could replace it.

`OrderService` depends on the interface, not the storage mechanism.

### Java Usage

Use Repository in Java when:

- Business logic should not know SQL/ORM details.
- You want testable services.
- Domain-specific queries are important.
- Multiple persistence implementations may exist.
- You are modeling aggregates in DDD-style code.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Order:
    id: str
    user_id: str
    status: str


class OrderRepository:
    def __init__(self):
        self._orders = {}

    def find_by_id(self, order_id):
        return self._orders.get(order_id)

    def find_open_orders_for_user(self, user_id):
        return [
            order for order in self._orders.values()
            if order.user_id == user_id and order.status == "OPEN"
        ]

    def save(self, order):
        self._orders[order.id] = order
```

### Python Usage

Python repositories are often plain classes around:

- SQLAlchemy sessions.
- Django ORM queries.
- External APIs.
- In-memory collections for tests.
- File/object storage.

## 6. Where It Comes Handy in Real Life

- Order repositories.
- User account repositories.
- Subscription repositories.
- Inventory repositories.
- Payment transaction repositories.
- Domain-driven aggregate loading.
- Unit tests with fake persistence.
- Clean/hexagonal architecture ports.

## 7. Advantages Over Normal Code Without Pattern

### Without Repository

```java
service -> SQL/ORM details directly
```

Problems:

- Business logic knows storage.
- Queries are duplicated.
- Tests are slow.
- Storage migration is risky.

### With Repository

```java
service -> repository -> database
```

Benefits:

- Data access is centralized.
- Services are easier to test.
- Domain language is clearer.
- Storage details are hidden.

## 8. Where It Excels

- Domain-rich applications.
- Services needing test doubles.
- Clean Architecture and Hexagonal Architecture.
- Query methods with business meaning.
- Persistence that may evolve over time.
- Aggregate-level access boundaries.

## 9. Where It Fails

- Very simple CRUD apps where ORM repositories already suffice.
- Generic repositories that hide useful query capabilities.
- Read-heavy reporting where SQL/query models are more direct.
- Cases where repository becomes a dumping ground for every query.
- Teams that confuse Repository with DAO and duplicate both layers.

## 10. Prebuilt Frameworks and Packages

### Java

- Spring Data Repository.
- JPA/Hibernate repositories.
- Micronaut Data.
- Quarkus Panache repositories.
- jOOQ can sit under repository implementations.

### Python

- SQLAlchemy repositories.
- Django managers/querysets.
- Repository classes over external APIs.
- Pytest fake repositories.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples business logic from persistence. | Adds abstraction. |
| Improves testability. | Generic CRUD repositories can be weak. |
| Centralizes query logic. | Can become too large. |
| Uses domain-friendly method names. | May duplicate ORM features. |
| Works well with Clean/Hexagonal Architecture. | Bad boundaries leak persistence details anyway. |

## 12. Real-World Identification Example

Scenario:

You are designing subscription billing.

Repository fit:

- `SubscriptionRepository.findActiveByCustomer(customerId)`
- `SubscriptionRepository.save(subscription)`
- `PaymentRepository.findPendingRetries()`

What would go wrong without it:

- SQL appears in renewal service.
- Tests need a real database.
- Query changes affect business workflow code.

## 13. MAANG Interview Triggers

Use Repository when you hear:

- "Abstract persistence."
- "Test service without database."
- "Domain-friendly data access."
- "Aggregate root loading."
- "Clean Architecture repository port."
- "Hide ORM/JDBC details."

### Interview-Ready Answer Format

1. Identify the entity or aggregate.
2. Define repository methods in domain language.
3. Keep persistence details in implementation.
4. Inject repository into services/use cases.
5. Use fake/in-memory repository in tests.
6. Mention trade-off: avoid generic CRUD-only abstraction when unnecessary.

## 14. Common Mistakes

### Mistake 1: Repository Mirrors Every Table

Repository should usually be aggregate/domain oriented, not blindly table oriented.

### Mistake 2: Exposing ORM Query Objects Everywhere

That leaks persistence concerns outside the boundary.

### Mistake 3: One Giant Repository

Split repositories by aggregate or coherent domain area.

### Mistake 4: Generic Repository for Everything

Generic CRUD may hide important domain query intent.

### Mistake 5: Repository Owns Business Rules

Repositories fetch and save. Domain/services own business decisions.

## 15. Repository vs Similar Patterns

| Pattern | Difference |
|---|---|
| Repository | Domain-facing collection-like persistence boundary. |
| DAO | Lower-level data-source abstraction, often table/record oriented. |
| Data Mapper | Maps between objects and database rows. |
| Unit of Work | Tracks changes and commits them transactionally. |
| Service Layer | Orchestrates use cases and calls repositories. |

## 16. Repository Design Checklist

| Question | Why it matters |
|---|---|
| What aggregate does it manage? | Defines repository scope. |
| Are methods domain-friendly? | Keeps persistence hidden. |
| Does it leak SQL/ORM details? | Protects callers. |
| Can it be faked in tests? | Proves decoupling. |
| Does it own business logic? | Prevents misplaced rules. |
| Is a simple ORM repository enough? | Avoids unnecessary wrappers. |

## 17. Quick Revision Notes

- Repository hides persistence behind domain methods.
- It feels like a collection of domain objects.
- Services/use cases depend on repository interfaces.
- Implementations use JPA/JDBC/API/memory.
- Different from DAO: more domain-oriented.
- Avoid giant generic repositories.

## 18. Mini Exercise

Design Repository for `CourseEnrollment`.

Methods:

- `findById(enrollmentId)`
- `findActiveByStudent(studentId)`
- `save(enrollment)`
- `findPendingCompletionEvents()`

Think through:

- Which methods are domain-specific?
- Which ones should stay in query/reporting layer instead?

## 19. Source Reference in This Repo

The repository's Repository implementation uses Spring Data style repositories for `Person` entities and specification-based queries.

Useful files:

- [github-repo/repository/README.md](../../github-repo/repository/README.md)
- [github-repo/repository/src/main/java/com/iluwatar/repository/Person.java](../../github-repo/repository/src/main/java/com/iluwatar/repository/Person.java)
- [github-repo/repository/src/main/java/com/iluwatar/repository/PersonRepository.java](../../github-repo/repository/src/main/java/com/iluwatar/repository/PersonRepository.java)
- [github-repo/repository/src/main/java/com/iluwatar/repository/PersonSpecifications.java](../../github-repo/repository/src/main/java/com/iluwatar/repository/PersonSpecifications.java)
- [github-repo/repository/src/main/java/com/iluwatar/repository/App.java](../../github-repo/repository/src/main/java/com/iluwatar/repository/App.java)

