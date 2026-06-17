# Facade Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/facade](../../github-repo/facade)

## How to Study This Page

Use this page in three passes:

1. First pass: understand how a facade simplifies a complex subsystem.
2. Second pass: rewrite the Java example and identify facade, subsystem classes, and client.
3. Third pass: compare Facade with Adapter, Proxy, and Service Layer.

By the end, you should be able to say:

> Facade provides a simple, unified interface over a complex subsystem without necessarily hiding all subsystem access.

## 1. Technical Definition

Facade is a structural design pattern that provides a simplified interface to a set of classes, APIs, or subsystem operations.

Core idea:

- Hide common subsystem orchestration behind one simple API.
- Reduce client dependencies on many subsystem classes.
- Make common workflows easier to use.
- Keep subsystem classes available for advanced use when appropriate.

### 30-Second Interview Answer

I would use Facade when clients repeatedly coordinate several subsystem objects to perform a common workflow. A facade exposes a simple method such as `checkout()` or `startMovie()` and internally calls the subsystem in the right order. It improves readability and reduces coupling, but it can become a god object if too much unrelated behavior is placed behind it.

## 2. Layman and Easy to Understand Definition

Facade is like a hotel front desk.

You do not personally coordinate housekeeping, billing, room service, and maintenance. You call the front desk, and the front desk coordinates the right teams.

In code:

- The front desk is the facade.
- The teams are subsystem classes.
- You are the client.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose checkout requires multiple subsystem calls:

```java
inventory.reserve(cart);
payment.charge(customer, cart.total());
shipping.createLabel(order);
email.sendConfirmation(order);
```

Problems:

- Client code must know subsystem order.
- Error handling gets repeated.
- Subsystem details leak into many places.
- Common workflows become noisy.

### 3.2 The Facade Solution

Create one simple API:

```java
CheckoutFacade checkout = new CheckoutFacade(inventory, payment, shipping, email);
Receipt receipt = checkout.placeOrder(customer, cart);
```

The facade coordinates the subsystem internally.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Facade | Simple entry point for clients. |
| Subsystem classes | Lower-level services or APIs. |
| Client | Uses facade instead of coordinating subsystem directly. |
| Workflow method | Coarse-grained method that calls subsystem operations. |

### 3.4 Mental Model

Think of Facade as a simplified front door.

1. Client calls one high-level method.
2. Facade validates or prepares input.
3. Facade calls subsystem classes in the right order.
4. Facade combines or returns the result.
5. Client avoids subsystem complexity.

## 4. Java Coding Example

This example creates a checkout facade.

```java
public record Cart(long totalCents) {}

public record Customer(String id, String email) {}

public record Receipt(String orderId, boolean paid) {}

public final class InventoryService {
    public void reserve(Cart cart) {
        System.out.println("Reserved inventory");
    }
}

public final class PaymentService {
    public boolean charge(Customer customer, long amountCents) {
        System.out.println("Charged " + customer.id());
        return true;
    }
}

public final class ShippingService {
    public String createShipment(Customer customer, Cart cart) {
        System.out.println("Created shipment");
        return "order-123";
    }
}

public final class EmailService {
    public void sendConfirmation(Customer customer, String orderId) {
        System.out.println("Sent confirmation to " + customer.email());
    }
}

public final class CheckoutFacade {
    private final InventoryService inventory;
    private final PaymentService payment;
    private final ShippingService shipping;
    private final EmailService email;

    public CheckoutFacade(
        InventoryService inventory,
        PaymentService payment,
        ShippingService shipping,
        EmailService email
    ) {
        this.inventory = inventory;
        this.payment = payment;
        this.shipping = shipping;
        this.email = email;
    }

    public Receipt placeOrder(Customer customer, Cart cart) {
        inventory.reserve(cart);
        boolean paid = payment.charge(customer, cart.totalCents());
        String orderId = shipping.createShipment(customer, cart);
        email.sendConfirmation(customer, orderId);
        return new Receipt(orderId, paid);
    }
}
```

### Java Block by Block Explanation

#### Subsystem Classes

```java
public final class InventoryService {
```

Each subsystem has its own focused responsibility.

#### Facade

```java
public final class CheckoutFacade {
```

The facade provides a simple entry point for the common checkout workflow.

#### Workflow Method

```java
public Receipt placeOrder(Customer customer, Cart cart) {
```

This method coordinates inventory, payment, shipping, and email.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        CheckoutFacade checkout = new CheckoutFacade(
            new InventoryService(),
            new PaymentService(),
            new ShippingService(),
            new EmailService()
        );

        Receipt receipt = checkout.placeOrder(
            new Customer("customer-1", "user@example.com"),
            new Cart(4999)
        );

        System.out.println(receipt);
    }
}
```

The client calls one method instead of coordinating four services.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Cart:
    total_cents: int


@dataclass(frozen=True)
class Customer:
    customer_id: str
    email: str


class InventoryService:
    def reserve(self, cart: Cart) -> None:
        print("Reserved inventory")


class PaymentService:
    def charge(self, customer: Customer, amount_cents: int) -> bool:
        print(f"Charged {customer.customer_id}")
        return True


class ShippingService:
    def create_shipment(self, customer: Customer, cart: Cart) -> str:
        print("Created shipment")
        return "order-123"


class CheckoutFacade:
    def __init__(
        self,
        inventory: InventoryService,
        payment: PaymentService,
        shipping: ShippingService,
    ) -> None:
        self._inventory = inventory
        self._payment = payment
        self._shipping = shipping

    def place_order(self, customer: Customer, cart: Cart) -> str:
        self._inventory.reserve(cart)
        self._payment.charge(customer, cart.total_cents)
        return self._shipping.create_shipment(customer, cart)
```

### Python Usage

```python
checkout = CheckoutFacade(InventoryService(), PaymentService(), ShippingService())
order_id = checkout.place_order(Customer("customer-1", "user@example.com"), Cart(4999))
print(order_id)
```

## 6. Where It Comes Handy in Real Life

Facade is useful for simplifying subsystem workflows.

Examples:

- Checkout orchestration.
- Home theater control.
- File import pipeline.
- Cloud deployment workflow.
- Payment provider integration.
- Analytics event ingestion.
- Complex SDK wrappers.

## 7. Advantages Over Normal Code Without Pattern

### Without Facade

```java
inventory.reserve(cart);
payment.charge(customer, amount);
shipping.createShipment(customer, cart);
email.sendConfirmation(customer, orderId);
```

Problems:

- Clients know too much.
- Call order is repeated.
- Error handling spreads.
- Subsystem coupling grows.

### With Facade

```java
Receipt receipt = checkout.placeOrder(customer, cart);
```

Benefits:

- Client code is simpler.
- Common workflow is centralized.
- Subsystem details are hidden for ordinary use.
- Dependencies become easier to manage.

## 8. Where It Excels

Facade excels when:

- A subsystem has many classes.
- Clients need only common workflows.
- You want a stable entry point.
- You want to layer a system.
- Repeated orchestration should be centralized.

## 9. Where It Fails

Facade is a poor fit when:

- The subsystem is already simple.
- The facade becomes a god object.
- It hides too much and blocks advanced use cases.
- It mixes unrelated workflows.
- It becomes the only place where logic lives.

## 10. Prebuilt Libraries and Packages

### Java

Facade-like APIs:

- Service layer methods.
- SDK client classes.
- Spring service classes over repositories/clients.
- `JdbcTemplate` simplifying JDBC.
- Higher-level cloud SDK clients.

### Python

Python examples:

- Service objects.
- SDK wrapper classes.
- CLI command handlers over many services.
- Simplified modules that wrap lower-level APIs.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Simplifies common workflows. | Can become too large. |
| Reduces client coupling. | May hide useful subsystem features. |
| Centralizes orchestration. | Can turn into a procedural god class. |
| Provides stable entry point. | Adds another abstraction layer. |
| Improves readability. | Poorly designed facades can become bottlenecks. |

## 12. Real-World Identification Example

Scenario:

You are designing an import system.

Subsystems:

- File storage
- Schema validator
- Parser
- Transformer
- Database writer
- Notification service

Should you use Facade?

Yes, for the common import workflow.

Good usage:

```java
ImportResult result = importFacade.importFile(fileId);
```

## 13. MAANG Interview Triggers

Think Facade when you hear:

- Simplify complex subsystem.
- Unified interface.
- Common workflow.
- Reduce client coupling.
- Entry point to a layer.
- Hide orchestration.
- SDK wrapper.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the complex subsystem.
2. Identify the common workflow clients need.
3. Create a facade with coarse-grained methods.
4. Let the facade coordinate subsystem calls.
5. Keep subsystem classes focused and testable.
6. Mention trade-off: facade can become too broad.

## 14. Common Mistakes

### Mistake 1: God Facade

Do not put every operation in one giant facade.

### Mistake 2: Hiding All Subsystem Access

Advanced clients may still need direct lower-level APIs.

### Mistake 3: Business Logic Dump

Facade should coordinate. Core rules should remain in appropriate services or domain objects.

### Mistake 4: Too Many Tiny Facades

If each facade method simply calls one method, it may not add value.

### Mistake 5: Confusing Facade with Adapter

Facade simplifies; Adapter changes an incompatible interface.

## 15. Facade vs Similar Patterns

| Pattern | Difference |
|---|---|
| Adapter | Adapter converts one interface to another. Facade simplifies a subsystem. |
| Proxy | Proxy controls access to one subject. Facade coordinates many subsystem classes. |
| Decorator | Decorator adds behavior to one object. Facade provides a simpler API. |
| Service Layer | Service Layer is an architectural boundary; it often acts like a facade. |
| Mediator | Mediator coordinates peer objects; Facade provides a simplified external entry point. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Facade | `CheckoutFacade` | Simple client-facing API. |
| Subsystem classes | Inventory/payment/shipping/email services | Detailed operations. |
| Client | Demo code | Calls facade. |

## 17. Quick Revision Notes

- Facade simplifies a subsystem.
- It provides coarse-grained methods.
- It reduces client coupling.
- It centralizes common orchestration.
- It does not have to hide every subsystem class.
- Avoid god facades.

## 18. Mini Exercise

Design a Facade for `VideoPublishingFacade`.

Subsystems:

- Upload storage
- Transcoder
- Thumbnail generator
- Metadata service
- Notification service

Expected usage:

```java
PublishResult result = videoPublishingFacade.publish(videoFile);
```

## 19. Source Reference in This Repo

The repository's Facade implementation uses `DwarvenGoldmineFacade` to simplify operations over several mine worker subsystem classes.

Useful files:

- [github-repo/facade/README.md](../../github-repo/facade/README.md)
- [github-repo/facade/src/main/java/com/iluwatar/facade/DwarvenGoldmineFacade.java](../../github-repo/facade/src/main/java/com/iluwatar/facade/DwarvenGoldmineFacade.java)
- [github-repo/facade/src/main/java/com/iluwatar/facade/DwarvenMineWorker.java](../../github-repo/facade/src/main/java/com/iluwatar/facade/DwarvenMineWorker.java)
- [github-repo/facade/src/main/java/com/iluwatar/facade/App.java](../../github-repo/facade/src/main/java/com/iluwatar/facade/App.java)
