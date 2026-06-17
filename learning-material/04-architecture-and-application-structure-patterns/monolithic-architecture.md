# Monolithic Architecture Pattern

Category: Architecture and Application Structure Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/monolithic-architecture](../../github-repo/monolithic-architecture)

## How to Study This Page

Use this page in three passes:

1. First pass: understand one deployable application containing all major features.
2. Second pass: rewrite the Java example and identify modules, controllers, services, repositories, and shared database.
3. Third pass: compare Monolithic Architecture with Microservices, Modular Monolith, and Layered Architecture.

By the end, you should be able to say:

> Monolithic Architecture packages an application's features into one deployable unit, which is simple to build and operate early but harder to scale and change independently as the system grows.

## 1. Technical Definition

Monolithic Architecture is an application structure where the user interface, business logic, and data access for multiple features are built, deployed, and usually scaled as a single unit.

Core idea:

- One codebase or tightly coupled codebase.
- One deployable artifact.
- Features run in one process/runtime.
- Shared database is common.
- Internal modularity may still exist.

### 30-Second Interview Answer

I would choose a monolith for early-stage or moderately complex products when simplicity, fast development, and easy deployment matter more than independent scaling. A monolith can still be clean and modular internally using layers or modules. The risk is that as the codebase and team grow, changes, deployments, and scaling become coupled. At that point, a modular monolith or selective service extraction may be better.

## 2. Layman and Easy to Understand Definition

Monolithic Architecture is like one large store that contains every department under one roof.

It is easy for customers to visit and easy for the owner to manage at first. But if one department gets huge, you cannot scale or renovate only that department without affecting the whole building.

In code:

- All features are packaged together.
- One deployment updates everything.
- Modules may exist, but runtime is shared.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Every application needs a deployment shape.

For a new product, splitting into many services too early creates problems:

- More deployments.
- More networking.
- More observability.
- More distributed failures.
- More team coordination.

### 3.2 The Monolithic Solution

Start with one deployable application:

```text
web/API + business logic + persistence access -> one artifact
```

The app can still be organized internally:

```text
orders module
users module
payments module
```

But it deploys together.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Single deployable | One app artifact/process. |
| Modules/packages | Internal feature organization. |
| Shared database | Common storage used by the app. |
| Controllers/API | Input layer inside the monolith. |
| Services/domain | Business logic inside the monolith. |
| Repositories | Persistence access inside the monolith. |

### 3.4 Monolith Types

| Type | Meaning |
|---|---|
| Big ball of mud | Unstructured monolith with tangled code. |
| Layered monolith | One deployment but organized by layers. |
| Modular monolith | One deployment with strong internal module boundaries. |
| Distributed monolith | Multiple services that are so coupled they behave like one bad monolith. |

## 4. Java Coding Example

This example shows one application with multiple feature services.

```java
import java.util.ArrayList;
import java.util.List;

record User(String id, String email) {
}

record Product(String id, String name, int stock) {
}

record Order(String id, String userId, String productId) {
}

final class UserService {
    User register(String email) {
        return new User("user-" + System.nanoTime(), email);
    }
}

final class ProductService {
    private final List<Product> products = new ArrayList<>();

    Product addProduct(String name, int stock) {
        Product product = new Product("product-" + System.nanoTime(), name, stock);
        products.add(product);
        return product;
    }
}

final class OrderService {
    Order placeOrder(String userId, String productId) {
        return new Order("order-" + System.nanoTime(), userId, productId);
    }
}

public final class EcommerceMonolith {
    public static void main(String[] args) {
        UserService users = new UserService();
        ProductService products = new ProductService();
        OrderService orders = new OrderService();

        User user = users.register("a@example.com");
        Product product = products.addProduct("Laptop", 10);
        Order order = orders.placeOrder(user.id(), product.id());

        System.out.println(order);
    }
}
```

### Java Block by Block Explanation

`UserService`, `ProductService`, and `OrderService` represent separate features.

`EcommerceMonolith` runs all features in one process.

The example can be organized by layers/modules while still being one deployable unit.

Important detail:

- Monolith does not mean messy.
- A good monolith has strong internal boundaries.
- A bad monolith lets everything depend on everything.

### Java Usage

Use Monolithic Architecture in Java when:

- Product is early or medium complexity.
- Team is small or medium.
- Independent scaling is not yet needed.
- Simpler deployment is valuable.
- Most features share one transactional boundary.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from time import time_ns


@dataclass(frozen=True)
class User:
    id: str
    email: str


@dataclass(frozen=True)
class Product:
    id: str
    name: str
    stock: int


@dataclass(frozen=True)
class Order:
    id: str
    user_id: str
    product_id: str


class UserService:
    def register(self, email):
        return User(f"user-{time_ns()}", email)


class ProductService:
    def add_product(self, name, stock):
        return Product(f"product-{time_ns()}", name, stock)


class OrderService:
    def place_order(self, user_id, product_id):
        return Order(f"order-{time_ns()}", user_id, product_id)


users = UserService()
products = ProductService()
orders = OrderService()

user = users.register("a@example.com")
product = products.add_product("Laptop", 10)
print(orders.place_order(user.id, product.id))
```

### Python Usage

Python monoliths often use:

- Django project with multiple apps.
- FastAPI app with multiple routers/modules.
- Celery workers in the same codebase.
- Shared relational database.
- Internal modules to keep boundaries clear.

## 6. Where It Comes Handy in Real Life

- MVPs and startups.
- Internal business apps.
- Admin portals.
- Traditional enterprise systems.
- E-commerce applications at early/medium scale.
- Products needing fast iteration.
- Systems with strong transactional consistency across features.

## 7. Advantages Over Normal Code Without Pattern

### Without an Intentional Monolith

```text
one app, but no module boundaries
everything imports everything
```

Problems:

- Codebase becomes tangled.
- Teams collide.
- Changes become risky.
- Extracting services later is hard.

### With an Intentional Monolith

```text
one deployable, clear modules, clear layers
```

Benefits:

- Simple deployment.
- Easy local development.
- Strong consistency.
- Lower operational burden.
- Good stepping stone to modular monolith or services.

## 8. Where It Excels

- Simplicity.
- Fast feature delivery.
- Easy debugging.
- Single-process transactions.
- Small-to-medium teams.
- Lower infrastructure cost.
- Local development and testing.

## 9. Where It Fails

- Very large teams.
- Features requiring independent scale.
- Frequent deployments by independent teams.
- Fault isolation requirements.
- Technology diversity requirements.
- Poor internal boundaries.
- Long build/deploy cycles.

## 10. Prebuilt Frameworks and Packages

### Java

- Spring Boot single deployable JAR.
- Jakarta EE WAR/EAR.
- Maven/Gradle multi-module monoliths.
- Modulith-style architecture with Spring Modulith.
- ArchUnit for module boundary checks.

### Python

- Django monolith.
- FastAPI modular app.
- Flask app factory with blueprints.
- Celery in same repository.
- SQLAlchemy/Django ORM with shared database.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Simple to develop and deploy. | Harder to scale features independently. |
| Easy local debugging. | Deployments affect the whole app. |
| Strong consistency is easier. | Codebase can become tangled. |
| Lower operational overhead. | Build/test times can grow. |
| Good starting architecture. | Fault isolation is limited. |

## 12. Real-World Identification Example

Scenario:

You are launching a new marketplace.

Monolith fit:

- Users, listings, orders, and payments live in one app.
- One database transaction can cover key workflows.
- Team can ship quickly.
- Internal modules separate feature ownership.

What would go wrong with premature microservices:

- Distributed transactions become hard.
- Local development slows down.
- Observability and deployment overhead increase.
- Service boundaries are guessed too early.

## 13. MAANG Interview Triggers

Use Monolithic Architecture when you hear:

- "Early-stage product."
- "Simple deployment."
- "Small team."
- "Strong consistency."
- "Avoid premature microservices."
- "Modular monolith."
- "When would you split services?"

### Interview-Ready Answer Format

1. State whether monolith is a good starting point.
2. Explain internal modular boundaries.
3. Use layers/services/repositories inside the monolith.
4. Mention single deployment and shared database.
5. Discuss scaling and team-size limits.
6. Explain extraction path to services when boundaries stabilize.

## 14. Common Mistakes

### Mistake 1: Equating Monolith with Bad Design

A monolith can be clean, layered, and modular.

### Mistake 2: No Internal Boundaries

Without module rules, the monolith becomes tangled.

### Mistake 3: Premature Microservices

Splitting too early creates distributed complexity without clear benefit.

### Mistake 4: Shared Database Abuse

Even in one database, modules should avoid casually owning each other's tables.

### Mistake 5: No Extraction Strategy

Track module boundaries and hotspots so future service extraction is possible.

## 15. Monolithic Architecture vs Similar Patterns

| Pattern | Difference |
|---|---|
| Monolithic Architecture | One deployable application. |
| Modular Monolith | One deployable with strict internal module boundaries. |
| Microservices | Independently deployable services. |
| Layered Architecture | Internal organization style, often inside a monolith. |
| Service-Oriented Architecture | Distributed services around business capabilities. |

## 16. Monolith Design Checklist

| Question | Why it matters |
|---|---|
| What are the internal modules? | Prevents big-ball-of-mud design. |
| Are dependencies controlled? | Keeps future extraction possible. |
| Is one deployment acceptable? | Validates monolith fit. |
| Do features need independent scale? | May trigger service extraction. |
| Is shared database safe? | Avoids ownership confusion. |
| Are build/test times manageable? | Detects growth pain. |

## 17. Quick Revision Notes

- Monolith is one deployable unit.
- It can still be layered and modular.
- Great for simplicity and early speed.
- Watch team scale and independent scaling needs.
- Modular monolith is often a strong intermediate design.
- Avoid distributed complexity too early.

## 18. Mini Exercise

Design a modular monolith for `FoodDelivery`.

Modules:

- Users.
- Restaurants.
- Orders.
- Payments.
- Delivery.

Questions:

- Which module owns each table?
- Which workflows require transactions?
- Which module might be extracted first and why?

## 19. Source Reference in This Repo

The repository's Monolithic Architecture implementation models an e-commerce app with users, products, orders, controllers, repositories, and one Spring Boot application.

Useful files:

- [github-repo/monolithic-architecture/README.md](../../github-repo/monolithic-architecture/README.md)
- [github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/EcommerceApp.java](../../github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/EcommerceApp.java)
- [github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/controller/UserController.java](../../github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/controller/UserController.java)
- [github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/controller/ProductController.java](../../github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/controller/ProductController.java)
- [github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/controller/OrderController.java](../../github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/controller/OrderController.java)
- [github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/repository/OrderRepository.java](../../github-repo/monolithic-architecture/src/main/java/com/iluwatar/monolithic/repository/OrderRepository.java)

