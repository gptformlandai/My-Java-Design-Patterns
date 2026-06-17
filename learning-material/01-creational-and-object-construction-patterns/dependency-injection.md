# Dependency Injection Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/dependency-injection](../../github-repo/dependency-injection)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why classes should receive dependencies instead of creating them directly.
2. Second pass: rewrite the Java constructor-injection example and explain how it improves testing.
3. Third pass: study the trade-offs, injection styles, and DI-vs-Service-Locator comparison for interviews.

By the end, you should be able to say:

> Dependency Injection makes a class depend on abstractions and receive its collaborators from the outside, which reduces coupling and makes testing much easier.

## 1. Technical Definition

Dependency Injection is a design pattern where an object receives the dependencies it needs from outside instead of constructing them internally. The class focuses on behavior, while object wiring is handled by the caller, factory, framework, or container.

Core idea:

- A class declares what it needs.
- Another object provides those dependencies.
- The class depends on abstractions where possible.
- Construction decisions move out of business logic.

### 30-Second Interview Answer

I would use Dependency Injection when a class needs collaborators such as repositories, gateways, clients, clocks, or configuration. Instead of creating those objects inside the class, I pass them in through the constructor or a DI framework. This reduces coupling, improves testability, and lets production code and test code provide different implementations.

## 2. Layman and Easy to Understand Definition

Dependency Injection is like hiring a chef who receives ingredients from the restaurant supply team.

The chef should not leave the kitchen, find farms, negotiate prices, and buy vegetables every time they cook. The chef should focus on cooking. The supply team gives the chef the ingredients.

In code:

- The chef is the class.
- The ingredients are dependencies.
- The supply team is the injector, factory, framework, or application setup code.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose an order service charges a customer by calling a payment gateway.

Bad design:

```java
public class OrderService {
    private final PaymentGateway gateway = new StripePaymentGateway();

    public void checkout(Order order) {
        gateway.charge(order.totalAmount());
    }
}
```

Problems:

- `OrderService` is tightly coupled to `StripePaymentGateway`.
- Tests may accidentally call real external systems.
- Switching to another gateway requires editing `OrderService`.
- The class mixes business behavior with dependency creation.

### 3.2 The Dependency Injection Solution

Move dependency creation outside the class:

```java
public class OrderService {
    private final PaymentGateway gateway;

    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    public void checkout(Order order) {
        gateway.charge(order.totalAmount());
    }
}
```

Now the service receives a gateway. Production code can inject Stripe. Test code can inject a fake.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Client class | The class that needs a dependency, such as `OrderService`. |
| Dependency | The collaborator being used, such as `PaymentGateway`. |
| Abstraction | Interface or parent type that hides implementation details. |
| Concrete implementation | Real implementation, such as `StripePaymentGateway`. |
| Injector | Code or framework that creates and wires objects together. |

### 3.4 Dependency Injection and Inversion of Control

Dependency Injection is a common form of Inversion of Control.

Without DI:

```text
OrderService controls creation of StripePaymentGateway.
```

With DI:

```text
Application setup controls creation and gives PaymentGateway to OrderService.
```

The control of dependency creation is inverted. The class no longer decides which concrete dependency it gets.

### 3.5 Mental Model

Think of DI as external wiring.

1. Define the dependency interface.
2. Implement one or more concrete dependencies.
3. Make the consumer accept the dependency from outside.
4. Wire real implementations in production.
5. Wire fake or mock implementations in tests.

## 4. Java Coding Example

This example creates a checkout service that depends on a `PaymentGateway`.

```java
import java.math.BigDecimal;
import java.util.Objects;

public interface PaymentGateway {
    Receipt charge(String customerId, BigDecimal amount);
}

public record Receipt(String id, boolean successful) {}

public final class StripePaymentGateway implements PaymentGateway {
    @Override
    public Receipt charge(String customerId, BigDecimal amount) {
        System.out.println("Charging " + customerId + " amount " + amount);
        return new Receipt("stripe-receipt-1", true);
    }
}

public final class CheckoutService {
    private final PaymentGateway paymentGateway;

    public CheckoutService(PaymentGateway paymentGateway) {
        this.paymentGateway = Objects.requireNonNull(paymentGateway);
    }

    public Receipt checkout(String customerId, BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("amount must be positive");
        }
        return paymentGateway.charge(customerId, amount);
    }
}
```

### Java Block by Block Explanation

#### Dependency Interface

```java
public interface PaymentGateway {
    Receipt charge(String customerId, BigDecimal amount);
}
```

`CheckoutService` depends on this interface, not on Stripe, PayPal, or any specific vendor.

#### Concrete Implementation

```java
public final class StripePaymentGateway implements PaymentGateway {
```

This is one possible production implementation. Another implementation can be added without changing `CheckoutService`.

#### Constructor Injection

```java
public CheckoutService(PaymentGateway paymentGateway) {
    this.paymentGateway = Objects.requireNonNull(paymentGateway);
}
```

The dependency is required and is provided through the constructor. This is usually the cleanest DI style in Java because the object cannot exist without its required collaborators.

#### Business Method

```java
public Receipt checkout(String customerId, BigDecimal amount) {
```

The method focuses on checkout behavior. It does not create the payment gateway.

### Java Usage

```java
import java.math.BigDecimal;

public class Demo {
    public static void main(String[] args) {
        PaymentGateway gateway = new StripePaymentGateway();
        CheckoutService checkoutService = new CheckoutService(gateway);

        Receipt receipt = checkoutService.checkout("customer-1", new BigDecimal("49.99"));
        System.out.println(receipt);
    }
}
```

### Java Test Usage

```java
import java.math.BigDecimal;

public final class FakePaymentGateway implements PaymentGateway {
    BigDecimal chargedAmount;

    @Override
    public Receipt charge(String customerId, BigDecimal amount) {
        this.chargedAmount = amount;
        return new Receipt("fake-receipt", true);
    }
}

public class CheckoutServiceTest {
    public static void main(String[] args) {
        FakePaymentGateway fakeGateway = new FakePaymentGateway();
        CheckoutService service = new CheckoutService(fakeGateway);

        service.checkout("customer-1", new BigDecimal("10.00"));

        System.out.println(fakeGateway.chargedAmount);
    }
}
```

The test does not need Stripe credentials, network access, or real payment behavior.

## 5. Python Coding Example

Python often uses constructor injection with protocols or duck typing.

```python
from dataclasses import dataclass
from decimal import Decimal
from typing import Protocol


@dataclass(frozen=True)
class Receipt:
    receipt_id: str
    successful: bool


class PaymentGateway(Protocol):
    def charge(self, customer_id: str, amount: Decimal) -> Receipt:
        ...


class StripePaymentGateway:
    def charge(self, customer_id: str, amount: Decimal) -> Receipt:
        print(f"Charging {customer_id} amount {amount}")
        return Receipt("stripe-receipt-1", True)


class CheckoutService:
    def __init__(self, payment_gateway: PaymentGateway) -> None:
        self._payment_gateway = payment_gateway

    def checkout(self, customer_id: str, amount: Decimal) -> Receipt:
        if amount <= Decimal("0.00"):
            raise ValueError("amount must be positive")
        return self._payment_gateway.charge(customer_id, amount)
```

### Python Usage

```python
from decimal import Decimal


service = CheckoutService(StripePaymentGateway())
receipt = service.checkout("customer-1", Decimal("49.99"))
print(receipt)
```

### Python Test Fake

```python
from decimal import Decimal


class FakePaymentGateway:
    def __init__(self) -> None:
        self.charged_amount: Decimal | None = None

    def charge(self, customer_id: str, amount: Decimal) -> Receipt:
        self.charged_amount = amount
        return Receipt("fake-receipt", True)


fake = FakePaymentGateway()
service = CheckoutService(fake)
service.checkout("customer-1", Decimal("10.00"))
print(fake.charged_amount)
```

## 6. Where It Comes Handy in Real Life

Dependency Injection is common in backend systems, services, frameworks, and tests.

Examples:

- Inject repositories into services.
- Inject HTTP clients into API clients.
- Inject payment gateways into checkout logic.
- Inject clocks into time-sensitive logic.
- Inject feature flags into application services.
- Inject caches, queues, metrics clients, and loggers.
- Inject fake dependencies in unit tests.

## 7. Advantages Over Normal Code Without Pattern

### Without Dependency Injection

```java
public class InvoiceService {
    private final EmailClient emailClient = new SendGridEmailClient();
}
```

Problems:

- Hard to test without SendGrid.
- Hard to switch providers.
- Hidden object creation inside business class.
- More coupling to concrete infrastructure.

### With Dependency Injection

```java
public class InvoiceService {
    private final EmailClient emailClient;

    public InvoiceService(EmailClient emailClient) {
        this.emailClient = emailClient;
    }
}
```

Benefits:

- Easier unit testing.
- Cleaner separation of concerns.
- Easier provider replacement.
- Dependencies are visible from the constructor.
- Business code depends on abstractions.

## 8. Where It Excels

Dependency Injection excels when:

- A class talks to external systems.
- You need fast unit tests.
- Dependencies vary by environment.
- You want clean application layering.
- You want to follow the Dependency Inversion Principle.
- Construction is handled by Spring, Guice, Micronaut, Dagger, or another container.

## 9. Where It Fails

Dependency Injection is a poor fit when:

- The object has no meaningful dependency.
- You inject simple value objects that should just be constructor parameters.
- You over-abstract every small class.
- The object graph becomes hard to understand.
- Framework magic hides where objects come from.
- You use field injection and make required dependencies invisible.

Example where DI is overkill:

```java
record Point(int x, int y) {}
```

`Point` does not need injected dependencies.

## 10. Prebuilt Libraries and Packages

### Java

Common DI frameworks:

- Spring Framework / Spring Boot
- Google Guice
- Dagger
- Jakarta CDI
- Micronaut
- Quarkus Arc

Common annotations:

- `@Inject`
- `@Autowired`
- `@Component`
- `@Service`
- `@Configuration`
- `@Bean`

Important note:

- A DI framework wires objects, but it does not automatically make the design good. You still need clear boundaries, meaningful abstractions, and sane lifecycle management.

### Python

Python often uses simpler approaches:

- Constructor parameters
- Function parameters
- `typing.Protocol`
- Manual wiring in application startup
- `dependency-injector` library
- FastAPI dependency system

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces coupling to concrete classes. | Can add setup complexity. |
| Makes unit testing easier. | Framework wiring can feel magical. |
| Makes dependencies visible. | Too many abstractions can make code harder to read. |
| Supports environment-specific implementations. | Incorrect scopes can cause lifecycle bugs. |
| Encourages dependency inversion. | Field injection can hide required dependencies. |

## 12. Real-World Identification Example

Scenario:

You are designing an order service that must support multiple payment providers.

Fields and collaborators:

- Required: order repository, payment gateway, inventory service
- Optional: fraud checker, metrics client, feature flag client

Should you use Dependency Injection?

Yes.

Why:

- Payment providers may change.
- External dependencies should be mockable in tests.
- The service should focus on business rules, not object construction.
- Different environments may use different implementations.

Good DI usage:

```java
OrderService service = new OrderService(
    orderRepository,
    paymentGateway,
    inventoryService
);
```

This is better than creating those dependencies inside `OrderService`.

## 13. MAANG Interview Triggers

Think Dependency Injection when you hear:

- Loose coupling.
- Testability.
- Mocking external dependencies.
- Constructor injection.
- Inversion of Control.
- Dependency Inversion Principle.
- Replaceable implementations.
- Spring, Guice, or service wiring.
- Avoid hard-coded dependencies.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the class and its dependencies.
2. Explain why direct construction is harmful.
3. Introduce an interface or abstraction if there are multiple implementations.
4. Inject dependencies through the constructor.
5. Mention testability and environment-specific wiring.
6. Mention a trade-off: configuration complexity and object graph visibility.

## 14. Common Mistakes

### Mistake 1: Field Injection Everywhere

Field injection hides required dependencies.

Bad:

```java
class OrderService {
    @Autowired
    private PaymentGateway paymentGateway;
}
```

Better:

```java
class OrderService {
    private final PaymentGateway paymentGateway;

    OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

### Mistake 2: Injecting Concrete Classes Without Reason

If an implementation may change, depend on an interface.

### Mistake 3: Too Many Tiny Interfaces

Do not create an interface for every class by habit. Create abstractions where they help substitution, testing, or architectural boundaries.

### Mistake 4: Service Locator Disguised as DI

This hides dependencies:

```java
PaymentGateway gateway = ServiceLocator.get(PaymentGateway.class);
```

The class is still looking up its own dependencies. Prefer constructor injection when possible.

### Mistake 5: Wrong Object Scope

A singleton service that holds request-specific state can leak data across requests. Scope matters.

## 15. Dependency Injection vs Similar Patterns

| Pattern | Difference |
|---|---|
| Factory | Creates objects on demand. DI provides already-created dependencies to consumers. |
| Factory Method | Lets subclasses decide which product to create. DI wires collaborators from outside. |
| Service Locator | Client asks a registry for dependencies. DI gives dependencies directly to the client. |
| Singleton | Ensures one instance. DI may manage a singleton scope, but DI is not the same as Singleton. |
| Strategy | Encapsulates interchangeable behavior. DI is often used to inject a chosen strategy. |

## 16. Injection Styles

| Style | Example | Best use |
|---|---|---|
| Constructor injection | `new Service(repository)` | Required dependencies. |
| Setter injection | `service.setRepository(repository)` | Optional or late-bound dependencies. |
| Method injection | `service.run(repository)` | Dependency needed only for one operation. |
| Field injection | `@Autowired private Repository repository` | Avoid for core application code when possible. |
| Framework injection | Spring or Guice creates the graph | Large applications with managed lifecycles. |

Constructor injection is usually the default recommendation because it makes required dependencies explicit and supports immutable fields.

## 17. Quick Revision Notes

- DI means receive dependencies from outside.
- Prefer constructor injection for required dependencies.
- Depend on abstractions when substitution matters.
- DI improves testability and loose coupling.
- DI frameworks help wiring, but design boundaries still matter.
- Avoid hiding dependencies through field injection or service locators.

## 18. Mini Exercise

Design dependency injection for `NotificationService`.

Required dependencies:

- `UserRepository`
- `EmailClient`
- `SmsClient`

Rules:

- Send email for users with verified email.
- Send SMS for users with verified phone.
- Tests should not call real email or SMS providers.

Expected usage:

```java
NotificationService service = new NotificationService(
    userRepository,
    emailClient,
    smsClient
);
```

## 19. Source Reference in This Repo

The repository's Dependency Injection implementation uses a `Wizard` that receives interchangeable `Tobacco` dependencies.

Useful files:

- [github-repo/dependency-injection/README.md](../../github-repo/dependency-injection/README.md)
- [github-repo/dependency-injection/src/main/java/com/iluwatar/dependency/injection/App.java](../../github-repo/dependency-injection/src/main/java/com/iluwatar/dependency/injection/App.java)
- [github-repo/dependency-injection/src/main/java/com/iluwatar/dependency/injection/AdvancedWizard.java](../../github-repo/dependency-injection/src/main/java/com/iluwatar/dependency/injection/AdvancedWizard.java)
- [github-repo/dependency-injection/src/main/java/com/iluwatar/dependency/injection/TobaccoModule.java](../../github-repo/dependency-injection/src/main/java/com/iluwatar/dependency/injection/TobaccoModule.java)
