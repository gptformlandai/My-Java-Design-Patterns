# Clean Architecture Pattern

Category: Architecture and Application Structure Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/clean-architecture](../../github-repo/clean-architecture)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the dependency rule: source code dependencies point inward toward business rules.
2. Second pass: rewrite the Java example and identify entity, use case, port, adapter, and controller.
3. Third pass: compare Clean Architecture with Layered Architecture, Hexagonal Architecture, and Service Layer.

By the end, you should be able to say:

> Clean Architecture keeps business rules independent of frameworks, databases, and UI by forcing outer layers to depend inward on use cases and entities.

## 1. Technical Definition

Clean Architecture is an application architecture style that separates software into concentric responsibility boundaries where dependencies point inward toward enterprise rules and application use cases.

Core idea:

- Entities hold core business rules.
- Use cases orchestrate application-specific workflows.
- Interface adapters translate external input/output.
- Frameworks and databases stay at the outer edge.
- Inner layers do not know outer layers.

### 30-Second Interview Answer

I would use Clean Architecture when the domain and use cases must survive UI, database, framework, or delivery mechanism changes. The dependency rule is the key: controllers, repositories, and frameworks depend inward on use-case interfaces and entities, never the other way around. This improves testability and long-term maintainability, but it adds abstraction and can be overkill for small CRUD apps.

## 2. Layman and Easy to Understand Definition

Clean Architecture is like protecting the engine of a car from the dashboard and tires.

You can change the display, tires, or paint without redesigning the engine. In software, the "engine" is your business logic. Databases, APIs, and web frameworks are replaceable outer parts.

In code:

- Business rules are inner layers.
- Controllers and databases are outer layers.
- Interfaces define how the two talk.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Many applications start like this:

```java
public Order checkout(String userId) {
    ResultSet rows = jdbc.query("select * from cart where user_id = ?", userId);
    // business rules mixed with SQL and HTTP-specific details
}
```

This works at first, but the design becomes fragile.

Problems:

- Business logic depends on database details.
- Tests need frameworks and infrastructure.
- UI changes affect domain code.
- Database changes ripple through use cases.
- Business rules are hard to find.

### 3.2 The Clean Architecture Solution

Move business behavior inward:

```java
Order order = checkoutUseCase.checkout(userId);
```

The use case depends on ports:

```java
interface CartRepository {
    Cart loadCart(String userId);
}
```

The database adapter implements that port outside the core.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Entity | Core business object and invariant holder. |
| Use case | Application workflow, such as checkout or register user. |
| Port | Interface required by use case. |
| Adapter | External implementation of a port, such as database or HTTP. |
| Controller | Converts user/API request into use-case call. |
| Presenter/DTO | Converts use-case result into output shape. |

### 3.4 Dependency Rule

The dependency rule is the whole game:

```text
Frameworks -> Adapters -> Use Cases -> Entities
```

Allowed:

- Controller calls use case.
- Use case calls repository interface.
- Repository implementation calls database.

Not allowed:

- Entity imports Spring annotations.
- Use case imports SQL library.
- Domain object knows HTTP request details.

## 4. Java Coding Example

This example models checkout with a use case and repository ports.

```java
import java.math.BigDecimal;
import java.util.List;

record Product(String id, String name, BigDecimal price) {
}

record CartItem(Product product, int quantity) {
    BigDecimal total() {
        return product.price().multiply(BigDecimal.valueOf(quantity));
    }
}

final class Order {
    private final String userId;
    private final List<CartItem> items;

    Order(String userId, List<CartItem> items) {
        if (items.isEmpty()) {
            throw new IllegalArgumentException("Cannot checkout empty cart");
        }
        this.userId = userId;
        this.items = List.copyOf(items);
    }

    BigDecimal total() {
        return items.stream()
            .map(CartItem::total)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    String userId() {
        return userId;
    }
}

interface CartRepository {
    List<CartItem> findItems(String userId);
    void clear(String userId);
}

interface OrderRepository {
    void save(Order order);
}

final class CheckoutUseCase {
    private final CartRepository cartRepository;
    private final OrderRepository orderRepository;

    CheckoutUseCase(CartRepository cartRepository, OrderRepository orderRepository) {
        this.cartRepository = cartRepository;
        this.orderRepository = orderRepository;
    }

    Order checkout(String userId) {
        List<CartItem> items = cartRepository.findItems(userId);
        Order order = new Order(userId, items);
        orderRepository.save(order);
        cartRepository.clear(userId);
        return order;
    }
}

final class CheckoutController {
    private final CheckoutUseCase checkoutUseCase;

    CheckoutController(CheckoutUseCase checkoutUseCase) {
        this.checkoutUseCase = checkoutUseCase;
    }

    String postCheckout(String userId) {
        Order order = checkoutUseCase.checkout(userId);
        return "Created order for " + order.userId() + " total=" + order.total();
    }
}
```

### Java Block by Block Explanation

`Product`, `CartItem`, and `Order` are inner business objects.

`CartRepository` and `OrderRepository` are ports. The use case depends on these interfaces, not a database.

`CheckoutUseCase` owns the application workflow: load cart, create order, save order, clear cart.

`CheckoutController` is an outer adapter. It translates an incoming request into a use-case call.

Important detail:

- The use case knows nothing about HTTP.
- The entity knows nothing about persistence.
- Infrastructure implements interfaces defined by the inner layers.

### Java Usage

Use Clean Architecture in Java when:

- You want framework-independent business rules.
- You need fast unit tests.
- The domain is expected to grow.
- Multiple delivery mechanisms may exist, such as REST, batch jobs, and messaging.
- Infrastructure choices may change.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class Product:
    id: str
    name: str
    price: Decimal


@dataclass(frozen=True)
class CartItem:
    product: Product
    quantity: int

    def total(self):
        return self.product.price * self.quantity


class Order:
    def __init__(self, user_id, items):
        if not items:
            raise ValueError("Cannot checkout empty cart")
        self.user_id = user_id
        self.items = tuple(items)

    def total(self):
        return sum(item.total() for item in self.items)


class CheckoutUseCase:
    def __init__(self, cart_repository, order_repository):
        self.cart_repository = cart_repository
        self.order_repository = order_repository

    def checkout(self, user_id):
        items = self.cart_repository.find_items(user_id)
        order = Order(user_id, items)
        self.order_repository.save(order)
        self.cart_repository.clear(user_id)
        return order
```

### Python Usage

In Python, Clean Architecture usually means:

- Keep framework code in outer modules.
- Keep use cases as plain Python classes/functions.
- Define repository protocols or abstract base classes.
- Test use cases with in-memory adapters.
- Avoid importing Django/FastAPI/SQLAlchemy inside domain objects.

## 6. Where It Comes Handy in Real Life

- Checkout and order management.
- Banking workflows.
- Insurance claims.
- Healthcare workflows.
- Subscription billing.
- Long-lived enterprise applications.
- Applications with REST, batch, and event-driven entry points.
- Domains where tests must run without infrastructure.

## 7. Advantages Over Normal Code Without Pattern

### Without Clean Architecture

```java
controller -> service -> jdbc/sql/framework/domain mixed together
```

Problems:

- Business rules are tangled with infrastructure.
- Testing requires slow setup.
- Framework migration is painful.
- Use cases are not explicit.

### With Clean Architecture

```java
controller -> use case -> repository port -> adapter
```

Benefits:

- Business rules are isolated.
- Unit tests are fast.
- Frameworks are replaceable.
- Use cases are easy to discuss in interviews.
- Boundaries make code ownership clearer.

## 8. Where It Excels

- Complex business domains.
- Long-lived products.
- High testability requirements.
- Multiple input/output channels.
- Teams that need clear ownership boundaries.
- Systems where infrastructure changes should not affect business rules.

## 9. Where It Fails

- Tiny CRUD apps.
- Prototypes.
- Short-lived internal scripts.
- Teams unfamiliar with dependency inversion.
- Codebases where every abstraction becomes ceremony.
- Simple apps where Layered Architecture is enough.

## 10. Prebuilt Frameworks and Packages

### Java

- Spring Boot can be used in outer adapters.
- Jakarta EE can be used in outer adapters.
- MapStruct for DTO mapping.
- ArchUnit for enforcing dependency rules.
- JUnit/Mockito for use-case tests.

### Python

- FastAPI/Django/Flask as outer adapters.
- SQLAlchemy as repository implementation.
- `typing.Protocol` for ports.
- Pytest for use-case tests.
- Dependency Injector or simple manual wiring.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Protects business logic from infrastructure. | Adds interfaces and layers. |
| Improves testability. | Can feel heavy for CRUD apps. |
| Makes use cases explicit. | Requires discipline around dependencies. |
| Supports framework/database replacement. | Mapping DTOs can add boilerplate. |
| Helps teams reason about boundaries. | Poorly named ports become confusing. |

## 12. Real-World Identification Example

Scenario:

You are designing an e-commerce checkout system.

Clean Architecture fit:

- `CheckoutUseCase` owns checkout flow.
- `Order` owns business invariants.
- `PaymentGateway` is a port.
- Stripe/Adyen/in-house payment adapter is outside.
- REST controller is just an input adapter.

What would go wrong without it:

- Payment SDK calls leak into checkout logic.
- Database queries appear inside business rules.
- Unit tests require external systems.
- Changing payment provider becomes risky.

## 13. MAANG Interview Triggers

Use Clean Architecture when you hear:

- "Keep business logic independent of frameworks."
- "Test use cases without database."
- "Multiple adapters: API, CLI, batch, events."
- "Dependency inversion."
- "Long-lived domain."
- "Replace infrastructure without rewriting core."

### Interview-Ready Answer Format

1. Identify core entities and use cases.
2. Define ports needed by use cases.
3. Put controllers, persistence, and external clients outside.
4. Make dependencies point inward.
5. Test use cases with in-memory adapters.
6. Mention trade-off: extra abstractions for maintainability.

## 14. Common Mistakes

### Mistake 1: Framework Annotations in Entities

This couples the core to infrastructure.

### Mistake 2: Repository Interfaces Designed Around SQL

Ports should reflect use-case needs, not table operations only.

### Mistake 3: Use Cases Become Thin Pass-Through Methods

If there is no workflow, the abstraction may be unnecessary.

### Mistake 4: DTO Mapping Everywhere Without Purpose

Map across boundaries, not between every small method.

### Mistake 5: Dependencies Point Outward

If the use case imports a controller, ORM entity, or HTTP class, the boundary is broken.

## 15. Clean Architecture vs Similar Patterns

| Pattern | Difference |
|---|---|
| Clean Architecture | Concentric boundaries with inward dependencies. |
| Hexagonal Architecture | Focuses on ports and adapters around the core. |
| Layered Architecture | Organizes by vertical layers; dependencies often go top-down. |
| Service Layer | Provides application API/use-case boundary inside a broader architecture. |
| Domain Model | Rich business objects used inside the core. |

## 16. Clean Architecture Design Checklist

| Question | Why it matters |
|---|---|
| What are the entities? | Defines stable business concepts. |
| What are the use cases? | Defines application workflows. |
| What ports does each use case need? | Keeps infrastructure replaceable. |
| Do dependencies point inward? | Enforces the architecture. |
| Can use cases run in unit tests? | Proves decoupling. |
| Are adapters thin? | Prevents framework logic from becoming business logic. |

## 17. Quick Revision Notes

- Dependency rule: inward only.
- Entities and use cases are the core.
- Controllers and databases are adapters.
- Ports are interfaces needed by use cases.
- Great for testability and long-lived domains.
- Avoid ceremony for tiny CRUD apps.

## 18. Mini Exercise

Design Clean Architecture for `SubscriptionRenewal`.

Core:

- `Subscription`
- `RenewalPolicy`
- `RenewSubscriptionUseCase`

Ports:

- `SubscriptionRepository`
- `PaymentGateway`
- `EmailSender`

Adapters:

- REST controller.
- SQL repository.
- Payment provider client.

## 19. Source Reference in This Repo

The repository's Clean Architecture implementation models shopping cart and order workflows with controllers, repositories, domain objects, and a shopping cart service/use case.

Useful files:

- [github-repo/clean-architecture/README.md](../../github-repo/clean-architecture/README.md)
- [github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/Product.java](../../github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/Product.java)
- [github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/Order.java](../../github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/Order.java)
- [github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/ShoppingCartService.java](../../github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/ShoppingCartService.java)
- [github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/CartController.java](../../github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/CartController.java)
- [github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/OrderController.java](../../github-repo/clean-architecture/src/main/java/com/iluwatar/cleanarchitecture/OrderController.java)

