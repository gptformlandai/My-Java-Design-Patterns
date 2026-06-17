# Hexagonal Architecture Pattern

Category: Architecture and Application Structure Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/hexagonal-architecture](../../github-repo/hexagonal-architecture)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the core application surrounded by ports and adapters.
2. Second pass: rewrite the Java example and identify primary ports, secondary ports, primary adapters, and secondary adapters.
3. Third pass: compare Hexagonal Architecture with Clean Architecture and Layered Architecture.

By the end, you should be able to say:

> Hexagonal Architecture keeps the domain/application core independent by making all outside communication happen through ports and adapters.

## 1. Technical Definition

Hexagonal Architecture, also called Ports and Adapters, is an application architecture style where the core business logic exposes and consumes ports, while external systems connect through adapters.

Core idea:

- Application core contains use cases and domain logic.
- Primary ports are APIs the outside world calls.
- Secondary ports are interfaces the core needs from outside systems.
- Primary adapters drive the app, such as REST or CLI.
- Secondary adapters implement external dependencies, such as database or payment.

### 30-Second Interview Answer

I would use Hexagonal Architecture when I want the application core to be testable and independent from UI, database, messaging, and vendor integrations. The core defines ports. Adapters translate between outside technology and those ports. This lets me test the core with fake adapters and replace infrastructure without changing business logic. The cost is more interfaces and mapping.

## 2. Layman and Easy to Understand Definition

Hexagonal Architecture is like a universal power adapter for your business logic.

The core logic has stable plugs. Different outside systems connect through adapters. If the database or API changes, you swap the adapter instead of rewiring the core.

In code:

- Core is the application.
- Ports are interfaces.
- Adapters are technology-specific implementations.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without clear ports, business logic talks directly to everything:

```java
controller -> service -> database client
                  -> payment SDK
                  -> email SDK
```

Problems:

- Unit tests need real infrastructure.
- Vendors leak into the core.
- Replacing external systems is expensive.
- Use cases become hard to run from CLI, REST, or messaging.

### 3.2 The Hexagonal Solution

Define ports around the core:

```java
interface PaymentPort {
    void charge(String accountId, Money amount);
}
```

Adapters implement those ports:

```java
final class StripePaymentAdapter implements PaymentPort {
    public void charge(String accountId, Money amount) {
        // call Stripe SDK
    }
}
```

The core depends only on `PaymentPort`.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Core/application | Business rules and use cases. |
| Primary port | Interface/API for driving the application. |
| Primary adapter | REST controller, CLI, scheduler, message consumer. |
| Secondary port | Interface required by the core. |
| Secondary adapter | Database, payment provider, email provider, external API. |

### 3.4 Flow

1. Primary adapter receives input.
2. It converts input into a core call.
3. Core executes use case.
4. Core calls secondary ports when external help is needed.
5. Secondary adapters perform technology-specific work.
6. Result is translated back outward.

## 4. Java Coding Example

This example models a ticket purchase.

```java
import java.math.BigDecimal;

record TicketRequest(String userId, String eventId) {
}

record Ticket(String id, String userId, String eventId) {
}

interface TicketPurchasePort {
    Ticket buyTicket(TicketRequest request);
}

interface TicketRepositoryPort {
    void save(Ticket ticket);
}

interface PaymentPort {
    void charge(String userId, BigDecimal amount);
}

final class TicketPurchaseService implements TicketPurchasePort {
    private final TicketRepositoryPort ticketRepository;
    private final PaymentPort paymentPort;

    TicketPurchaseService(TicketRepositoryPort ticketRepository, PaymentPort paymentPort) {
        this.ticketRepository = ticketRepository;
        this.paymentPort = paymentPort;
    }

    @Override
    public Ticket buyTicket(TicketRequest request) {
        paymentPort.charge(request.userId(), BigDecimal.valueOf(50));
        Ticket ticket = new Ticket("ticket-" + request.eventId(), request.userId(), request.eventId());
        ticketRepository.save(ticket);
        return ticket;
    }
}

final class RestTicketController {
    private final TicketPurchasePort ticketPurchase;

    RestTicketController(TicketPurchasePort ticketPurchase) {
        this.ticketPurchase = ticketPurchase;
    }

    String postBuyTicket(String userId, String eventId) {
        Ticket ticket = ticketPurchase.buyTicket(new TicketRequest(userId, eventId));
        return "Created " + ticket.id();
    }
}

final class InMemoryTicketRepository implements TicketRepositoryPort {
    @Override
    public void save(Ticket ticket) {
        System.out.println("Saved " + ticket);
    }
}
```

### Java Block by Block Explanation

`TicketPurchasePort` is a primary port. Outside systems can drive the use case through it.

`TicketRepositoryPort` and `PaymentPort` are secondary ports required by the core.

`TicketPurchaseService` is the core application service.

`RestTicketController` is a primary adapter.

`InMemoryTicketRepository` is a secondary adapter.

### Java Usage

Use Hexagonal Architecture in Java when:

- The app integrates with several external systems.
- You need testable use cases.
- You want adapters for REST, CLI, events, or batch jobs.
- Vendor SDKs should not leak into business logic.
- Infrastructure changes are expected.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class TicketRequest:
    user_id: str
    event_id: str


@dataclass(frozen=True)
class Ticket:
    id: str
    user_id: str
    event_id: str


class TicketPurchaseService:
    def __init__(self, ticket_repository, payment_port):
        self.ticket_repository = ticket_repository
        self.payment_port = payment_port

    def buy_ticket(self, request):
        self.payment_port.charge(request.user_id, Decimal("50"))
        ticket = Ticket(f"ticket-{request.event_id}", request.user_id, request.event_id)
        self.ticket_repository.save(ticket)
        return ticket


class RestTicketController:
    def __init__(self, ticket_purchase):
        self.ticket_purchase = ticket_purchase

    def post_buy_ticket(self, user_id, event_id):
        ticket = self.ticket_purchase.buy_ticket(TicketRequest(user_id, event_id))
        return {"ticketId": ticket.id}
```

### Python Usage

In Python:

- Keep FastAPI/Django code as adapters.
- Use protocols/abstract classes for ports when useful.
- Use in-memory adapters for tests.
- Keep SQLAlchemy/external SDKs outside core use cases.

## 6. Where It Comes Handy in Real Life

- Payment processing.
- Notification systems.
- Order processing.
- Banking integrations.
- Microservice internals.
- Batch + API + event consumer entry points.
- Systems where external providers change.
- Domains needing fast core tests.

## 7. Advantages Over Normal Code Without Pattern

### Without Hexagonal Architecture

```java
service -> Stripe SDK
service -> JDBC
service -> REST response
```

Problems:

- Core depends on technology.
- Tests are slow and brittle.
- Replacing vendors changes use cases.
- Input/output channels are hard to add.

### With Hexagonal Architecture

```java
adapter -> primary port -> core -> secondary port -> adapter
```

Benefits:

- Core is independent.
- Adapters are replaceable.
- Use cases are easy to test.
- External system failures are isolated.

## 8. Where It Excels

- Integration-heavy applications.
- Microservices with clear boundaries.
- Domains that require high testability.
- Systems with multiple input channels.
- Infrastructure/vendor replacement.
- Long-lived services.

## 9. Where It Fails

- Tiny applications.
- Simple CRUD with stable framework/database.
- Teams that create ports for every trivial helper.
- Apps where architecture ceremony slows delivery.
- Cases where Clean Architecture boundaries are already enough.

## 10. Prebuilt Frameworks and Packages

### Java

- Spring Boot for adapters and dependency injection.
- Guice for dependency injection.
- ArchUnit for boundary tests.
- JUnit/Mockito for port-based tests.
- MapStruct for adapter mapping.

### Python

- FastAPI/Django/Flask as primary adapters.
- SQLAlchemy as secondary adapter.
- Dependency Injector.
- Pytest fixtures as fake adapters.
- `typing.Protocol` for ports.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps core independent from infrastructure. | Adds interfaces and mapping. |
| Very testable. | Can be overkill for simple apps. |
| Adapters are replaceable. | Requires clear naming discipline. |
| Supports multiple input/output channels. | Too many ports can fragment code. |
| Makes external dependencies explicit. | New developers may need orientation. |

## 12. Real-World Identification Example

Scenario:

You are building a subscription service.

Hexagonal fit:

- REST controller and message consumer are primary adapters.
- `SubscribeUserPort` is primary port.
- `PaymentGatewayPort`, `UserRepositoryPort`, and `EmailPort` are secondary ports.
- Stripe, PostgreSQL, and SendGrid are adapters.

What would go wrong without it:

- Vendor SDKs leak into the use case.
- Tests require real network calls.
- Replacing payment provider touches business logic.

## 13. MAANG Interview Triggers

Use Hexagonal Architecture when you hear:

- "Ports and adapters."
- "Replace database/provider without changing core."
- "Test business logic without infrastructure."
- "Multiple delivery mechanisms."
- "Core should be independent."
- "External systems should be isolated."

### Interview-Ready Answer Format

1. Identify the application core.
2. Define primary ports for use cases.
3. Define secondary ports for external needs.
4. Implement adapters at the edge.
5. Wire dependencies from outside in.
6. Mention trade-off: abstraction and mapping overhead.

## 14. Common Mistakes

### Mistake 1: Ports Mirror Vendor APIs

Ports should express what the core needs, not the external provider's API shape.

### Mistake 2: Core Imports Adapter Classes

This breaks the architecture.

### Mistake 3: Too Many Tiny Ports

Ports should represent meaningful boundaries.

### Mistake 4: Adapters Contain Business Rules

Adapters translate and delegate; core owns rules.

### Mistake 5: Confusing Hexagonal with Folder Naming

The architecture is about dependency direction and ports, not just package names.

## 15. Hexagonal Architecture vs Similar Patterns

| Pattern | Difference |
|---|---|
| Hexagonal Architecture | Core surrounded by ports/adapters. |
| Clean Architecture | Similar dependency rule with more explicit rings. |
| Layered Architecture | Organizes by layers; may still let inner code depend on infrastructure. |
| Service Layer | Often lives inside the hexagon as application use cases. |
| Adapter Pattern | Object-level interface conversion; hexagonal uses adapters architecturally. |

## 16. Hexagonal Design Checklist

| Question | Why it matters |
|---|---|
| What is the core use case? | Defines the center. |
| What drives the app? | Finds primary adapters. |
| What does the core need externally? | Defines secondary ports. |
| Are ports technology-neutral? | Keeps core independent. |
| Can the core run with fake adapters? | Proves testability. |
| Are adapters thin? | Prevents business leakage. |

## 17. Quick Revision Notes

- Also called Ports and Adapters.
- Core defines or uses ports.
- Adapters connect external systems.
- Primary adapters drive the app.
- Secondary adapters are called by the app.
- Great for testability and replaceable infrastructure.

## 18. Mini Exercise

Design Hexagonal Architecture for `InvoicePayment`.

Primary adapters:

- REST endpoint.
- Scheduled retry job.

Secondary ports:

- `InvoiceRepository`
- `PaymentGateway`
- `ReceiptSender`

Goal:

- Core payment logic should run in tests with in-memory adapters.

## 19. Source Reference in This Repo

The repository's Hexagonal Architecture implementation models a lottery application with domain services, repositories, banking adapters, and event log adapters.

Useful files:

- [github-repo/hexagonal-architecture/README.md](../../github-repo/hexagonal-architecture/README.md)
- [github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/domain/LotteryService.java](../../github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/domain/LotteryService.java)
- [github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/domain/LotteryAdministration.java](../../github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/domain/LotteryAdministration.java)
- [github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/database/LotteryTicketRepository.java](../../github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/database/LotteryTicketRepository.java)
- [github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/banking/WireTransfers.java](../../github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/banking/WireTransfers.java)
- [github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/eventlog/LotteryEventLog.java](../../github-repo/hexagonal-architecture/src/main/java/com/iluwatar/hexagonal/eventlog/LotteryEventLog.java)

