# Layered Architecture Pattern

Category: Architecture and Application Structure Patterns  
MAANG interview meter: High  
Software usage meter: Very High  
Repository module: [github-repo/layered-architecture](../../github-repo/layered-architecture)

## How to Study This Page

Use this page in three passes:

1. First pass: understand separating an application into presentation, application/service, domain, and data layers.
2. Second pass: rewrite the Java example and identify which layer each class belongs to.
3. Third pass: compare Layered Architecture with Clean Architecture, Hexagonal Architecture, and Monolithic Architecture.

By the end, you should be able to say:

> Layered Architecture organizes code by responsibility levels so higher layers call lower layers through clear boundaries.

## 1. Technical Definition

Layered Architecture is an application structure pattern that organizes software into horizontal layers, where each layer has a distinct responsibility and usually depends only on the layer directly below it.

Core idea:

- Presentation handles user/API interaction.
- Application/service layer coordinates use cases.
- Domain layer holds business concepts and rules.
- Data access layer handles persistence.
- Dependencies usually flow top-down.

### 30-Second Interview Answer

I would use Layered Architecture for most standard enterprise applications because it gives a simple, familiar separation of concerns. Controllers handle input, services coordinate business operations, domain objects model business concepts, and repositories/DAOs handle persistence. It is easy to understand and widely used. The trade-off is that layers can become pass-through boilerplate or tightly coupled if boundaries are weak.

## 2. Layman and Easy to Understand Definition

Layered Architecture is like a company with departments.

Customer support talks to users, operations handles the process, specialists apply rules, and records team stores data. Each department has a role, and requests move through the right departments.

In code:

- Controller receives the request.
- Service coordinates the work.
- Domain applies rules.
- Repository stores/retrieves data.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without layers, all concerns get mixed:

```java
public String createOrder(HttpRequest request) {
    // parse HTTP
    // validate business rules
    // run SQL
    // format response
}
```

Problems:

- UI logic mixes with business logic.
- Persistence details appear everywhere.
- Tests are hard to isolate.
- Changes ripple through unrelated code.
- Ownership is unclear.

### 3.2 The Layered Solution

Split responsibilities:

```text
Controller -> Service -> Domain -> Repository -> Database
```

Each layer has a job and hides details from the layer above.

### 3.3 Common Layers

| Layer | Responsibility |
|---|---|
| Presentation/API | HTTP, UI, CLI, request/response mapping. |
| Application/Service | Use-case orchestration and transactions. |
| Domain | Business entities, value objects, domain rules. |
| Data Access | Repositories, DAOs, SQL/ORM details. |
| Infrastructure | Database, messaging, files, external clients. |

### 3.4 Dependency Flow

Classic layered flow:

```text
Presentation
    -> Service
        -> Repository
            -> Database
```

Stricter versions invert dependencies with interfaces to protect the domain.

## 4. Java Coding Example

This example structures order creation into layers.

```java
import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;

record CreateOrderRequest(String userId, String productId, int quantity) {
}

record Order(String id, String userId, String productId, int quantity, BigDecimal total) {
}

final class OrderRepository {
    private final Map<String, Order> orders = new HashMap<>();

    void save(Order order) {
        orders.put(order.id(), order);
    }

    Order findById(String id) {
        return orders.get(id);
    }
}

final class OrderService {
    private final OrderRepository repository;

    OrderService(OrderRepository repository) {
        this.repository = repository;
    }

    Order createOrder(String userId, String productId, int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("quantity must be positive");
        }
        Order order = new Order(
            "order-" + System.nanoTime(),
            userId,
            productId,
            quantity,
            BigDecimal.valueOf(25).multiply(BigDecimal.valueOf(quantity))
        );
        repository.save(order);
        return order;
    }
}

final class OrderController {
    private final OrderService service;

    OrderController(OrderService service) {
        this.service = service;
    }

    String postOrder(CreateOrderRequest request) {
        Order order = service.createOrder(request.userId(), request.productId(), request.quantity());
        return "Created " + order.id();
    }
}
```

### Java Block by Block Explanation

`CreateOrderRequest` belongs to the presentation/API boundary.

`OrderController` receives input and delegates to the service.

`OrderService` coordinates validation and order creation.

`OrderRepository` hides storage details.

`Order` is the business object returned by the operation.

### Java Usage

Use Layered Architecture when:

- You are building a standard web/API application.
- You need a familiar team structure.
- CRUD and business workflows are both present.
- You want clear separation without heavy architecture ceremony.
- The app can be understood as presentation -> service -> data.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from decimal import Decimal
from time import time_ns


@dataclass(frozen=True)
class CreateOrderRequest:
    user_id: str
    product_id: str
    quantity: int


@dataclass(frozen=True)
class Order:
    id: str
    user_id: str
    product_id: str
    quantity: int
    total: Decimal


class OrderRepository:
    def __init__(self):
        self.orders = {}

    def save(self, order):
        self.orders[order.id] = order


class OrderService:
    def __init__(self, repository):
        self.repository = repository

    def create_order(self, user_id, product_id, quantity):
        if quantity <= 0:
            raise ValueError("quantity must be positive")
        order = Order(f"order-{time_ns()}", user_id, product_id, quantity, Decimal("25") * quantity)
        self.repository.save(order)
        return order


class OrderController:
    def __init__(self, service):
        self.service = service

    def post_order(self, request):
        order = self.service.create_order(request.user_id, request.product_id, request.quantity)
        return {"orderId": order.id}
```

### Python Usage

Python layered apps commonly use:

- FastAPI/Django views as presentation.
- Service modules for application workflows.
- Domain modules for business objects.
- Repository modules for database access.
- Infrastructure modules for third-party clients.

## 6. Where It Comes Handy in Real Life

- Spring Boot applications.
- Django/FastAPI services.
- Internal enterprise systems.
- Admin portals.
- E-commerce apps.
- Banking back-office systems.
- CRUD-heavy applications with some business logic.
- APIs maintained by large teams.

## 7. Advantages Over Normal Code Without Pattern

### Without Layered Architecture

```java
controller parses request, validates business rules, runs SQL, returns response
```

Problems:

- Low separation of concerns.
- Hard-to-test methods.
- SQL leaks into presentation.
- Business rules are duplicated.

### With Layered Architecture

```java
controller -> service -> repository
```

Benefits:

- Clear responsibility boundaries.
- Easier testing.
- Familiar team organization.
- Lower learning curve.
- Changes are more localized.

## 8. Where It Excels

- Enterprise applications.
- CRUD plus business workflows.
- Teams needing standard structure.
- Apps with moderate complexity.
- Projects where maintainability matters more than architectural purity.
- Monoliths that need internal order.

## 9. Where It Fails

- Highly complex domains where domain must be protected from infrastructure.
- Systems needing many adapters and ports.
- Apps where every layer becomes a pass-through.
- Performance-sensitive paths where layer hops add overhead.
- Teams that put all logic in services and create an anemic domain.

## 10. Prebuilt Frameworks and Packages

### Java

- Spring MVC/Spring Boot.
- Jakarta EE.
- JPA/Hibernate.
- MyBatis.
- ArchUnit for layer rules.
- Maven/Gradle modules for layer separation.

### Python

- Django apps.
- FastAPI routers/services/repositories.
- SQLAlchemy repositories.
- Flask blueprints and service modules.
- Pytest for layer-level tests.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Very familiar and easy to teach. | Can become pass-through boilerplate. |
| Clear separation of concerns. | Dependencies may still leak downward. |
| Good for enterprise apps. | Domain can become anemic. |
| Works well inside monoliths. | Layer skipping causes inconsistency. |
| Supports team ownership. | Not enough for all integration-heavy systems. |

## 12. Real-World Identification Example

Scenario:

You are building an employee management API.

Layered fit:

- `EmployeeController` handles HTTP.
- `EmployeeService` validates and coordinates.
- `Employee` models business data.
- `EmployeeRepository` handles persistence.

What would go wrong without it:

- HTTP code and SQL mix together.
- Validation duplicates across endpoints.
- Tests need full app setup.

## 13. MAANG Interview Triggers

Use Layered Architecture when you hear:

- "Standard enterprise app."
- "Separate presentation, business, and data access."
- "Maintainable monolith."
- "Controller/service/repository."
- "CRUD with business rules."
- "Clear team boundaries."

### Interview-Ready Answer Format

1. Define the layers.
2. Explain dependency direction.
3. Put request/response logic in presentation.
4. Put workflow logic in service/application layer.
5. Put business rules in domain.
6. Put persistence in repositories/DAOs.
7. Mention pass-through and anemic-domain risks.

## 14. Common Mistakes

### Mistake 1: Fat Controllers

Controllers should translate input/output, not own business workflows.

### Mistake 2: Repository Business Logic

Repositories should persist/retrieve, not decide business policy.

### Mistake 3: Pass-Through Services

If a service just calls a repository, consider whether it adds value.

### Mistake 4: Layer Skipping Everywhere

Allow exceptions only intentionally. Random skipping erodes architecture.

### Mistake 5: Anemic Domain by Default

Do not put every rule in services if the domain object should own it.

## 15. Layered Architecture vs Similar Patterns

| Pattern | Difference |
|---|---|
| Layered Architecture | Horizontal layers by responsibility. |
| Clean Architecture | Strong inward dependency rule and use-case boundaries. |
| Hexagonal Architecture | Ports/adapters around the core. |
| Service Layer | One layer inside a layered system. |
| Monolithic Architecture | Deployment style; can internally use layers. |

## 16. Layered Design Checklist

| Question | Why it matters |
|---|---|
| What layers exist? | Establishes structure. |
| What can each layer depend on? | Prevents leaks. |
| Where do transactions live? | Usually service/application layer. |
| Where do business rules live? | Avoids duplication. |
| Can layers be tested separately? | Confirms boundaries. |
| Are services adding value? | Avoids ceremony. |

## 17. Quick Revision Notes

- Presentation handles input/output.
- Service coordinates use cases.
- Domain owns business concepts.
- Repository handles persistence.
- Very common in enterprise apps.
- Watch for fat controllers and pass-through services.

## 18. Mini Exercise

Design Layered Architecture for `LeaveApproval`.

Layers:

- Controller: receives leave request.
- Service: checks balance and approval rules.
- Domain: `Employee`, `LeaveRequest`, `LeaveBalance`.
- Repository: saves request and loads balances.

## 19. Source Reference in This Repo

The repository's Layered Architecture implementation uses view, service, DAO, DTO, and entity layers in a cake-building example.

Useful files:

- [github-repo/layered-architecture/README.md](../../github-repo/layered-architecture/README.md)
- [github-repo/layered-architecture/src/main/java/view/View.java](../../github-repo/layered-architecture/src/main/java/view/View.java)
- [github-repo/layered-architecture/src/main/java/view/CakeViewImpl.java](../../github-repo/layered-architecture/src/main/java/view/CakeViewImpl.java)
- [github-repo/layered-architecture/src/main/java/service/CakeBakingService.java](../../github-repo/layered-architecture/src/main/java/service/CakeBakingService.java)
- [github-repo/layered-architecture/src/main/java/service/CakeBakingServiceImpl.java](../../github-repo/layered-architecture/src/main/java/service/CakeBakingServiceImpl.java)
- [github-repo/layered-architecture/src/main/java/dao/CakeDao.java](../../github-repo/layered-architecture/src/main/java/dao/CakeDao.java)

