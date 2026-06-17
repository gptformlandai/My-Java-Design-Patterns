# Chain of Responsibility Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/chain-of-responsibility](../../github-repo/chain-of-responsibility)

## How to Study This Page

Use this page in three passes:

1. First pass: understand a request moving through handlers until one handles it.
2. Second pass: rewrite the Java example and identify request, handler, concrete handlers, and chain order.
3. Third pass: study when a chain should stop, continue, or fail if nobody handles the request.

By the end, you should be able to say:

> Chain of Responsibility lets a request pass through a sequence of handlers without the sender knowing which handler will process it.

## 1. Technical Definition

Chain of Responsibility is a behavioral design pattern where a request is passed along a chain of potential handlers. Each handler decides whether to handle the request, pass it to the next handler, or sometimes do both.

Core idea:

- Sender does not know the concrete receiver.
- Each handler has a chance to process the request.
- Handler order matters.
- The chain can be configured dynamically.

### 30-Second Interview Answer

I would use Chain of Responsibility when multiple handlers may process a request and the sender should not choose the handler directly. Examples include middleware, filters, validation pipelines, logging, and support escalation. Each handler checks whether it can handle the request and either handles it or forwards it. The trade-offs are harder debugging, ordering sensitivity, and the risk that no handler handles the request.

## 2. Layman and Easy to Understand Definition

Chain of Responsibility is like customer support escalation.

A basic issue starts with level 1 support. If they cannot solve it, they pass it to level 2. If level 2 cannot solve it, it goes to a specialist.

The customer does not need to know who will solve the issue. The chain decides.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a request can be handled by different teams.

Bad design:

```java
if (request.type().equals("BILLING")) {
    billingHandler.handle(request);
} else if (request.type().equals("TECHNICAL")) {
    technicalHandler.handle(request);
} else if (request.type().equals("SECURITY")) {
    securityHandler.handle(request);
}
```

Problems:

- Sender knows every handler.
- Adding a handler changes sender code.
- Handler order is embedded in conditionals.
- Reusable request pipelines are hard to build.

### 3.2 The Chain Solution

Create handlers and link them:

```java
SupportHandler chain =
    new BillingHandler(new TechnicalHandler(new SecurityHandler(null)));

chain.handle(request);
```

The sender sends the request to the first handler only.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Request | Object being processed. |
| Handler | Interface or base class for processing requests. |
| Concrete handler | Specific handler that may process the request. |
| Next handler | The next link in the chain. |
| Client | Sends request to the first handler. |

### 3.4 Stop vs Continue

| Style | Meaning | Example |
|---|---|---|
| Handle and stop | First matching handler ends the chain. | Support escalation. |
| Handle and continue | Every handler can add behavior. | Middleware filters. |
| Validate and stop on failure | First failure ends processing. | Validation pipeline. |

Decide this rule early. It changes the whole design.

## 4. Java Coding Example

This example routes support tickets.

```java
public record SupportTicket(String id, String type, String message) {}

public abstract class SupportHandler {
    private final SupportHandler next;

    protected SupportHandler(SupportHandler next) {
        this.next = next;
    }

    public final boolean handle(SupportTicket ticket) {
        if (canHandle(ticket)) {
            process(ticket);
            return true;
        }
        if (next != null) {
            return next.handle(ticket);
        }
        return false;
    }

    protected abstract boolean canHandle(SupportTicket ticket);

    protected abstract void process(SupportTicket ticket);
}

public final class BillingSupportHandler extends SupportHandler {
    public BillingSupportHandler(SupportHandler next) {
        super(next);
    }

    @Override
    protected boolean canHandle(SupportTicket ticket) {
        return "BILLING".equals(ticket.type());
    }

    @Override
    protected void process(SupportTicket ticket) {
        System.out.println("Billing handled " + ticket.id());
    }
}

public final class TechnicalSupportHandler extends SupportHandler {
    public TechnicalSupportHandler(SupportHandler next) {
        super(next);
    }

    @Override
    protected boolean canHandle(SupportTicket ticket) {
        return "TECHNICAL".equals(ticket.type());
    }

    @Override
    protected void process(SupportTicket ticket) {
        System.out.println("Technical support handled " + ticket.id());
    }
}
```

### Java Block by Block Explanation

#### Request

```java
public record SupportTicket(String id, String type, String message) {}
```

The request carries enough data for handlers to decide whether they can process it.

#### Base Handler

```java
public abstract class SupportHandler {
```

The base handler stores the next handler and controls forwarding.

#### Final Handle Method

```java
public final boolean handle(SupportTicket ticket) {
```

The template for chain behavior is fixed: try current handler, otherwise forward.

#### Concrete Handler

```java
protected boolean canHandle(SupportTicket ticket) {
    return "BILLING".equals(ticket.type());
}
```

Each handler owns one decision.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        SupportHandler chain = new BillingSupportHandler(
            new TechnicalSupportHandler(null)
        );

        boolean handled = chain.handle(
            new SupportTicket("ticket-1", "TECHNICAL", "Cannot log in")
        );

        System.out.println(handled);
    }
}
```

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class SupportTicket:
    ticket_id: str
    ticket_type: str
    message: str


class SupportHandler:
    def __init__(self, next_handler: "SupportHandler | None" = None) -> None:
        self._next = next_handler

    def handle(self, ticket: SupportTicket) -> bool:
        if self.can_handle(ticket):
            self.process(ticket)
            return True
        if self._next is not None:
            return self._next.handle(ticket)
        return False

    def can_handle(self, ticket: SupportTicket) -> bool:
        raise NotImplementedError

    def process(self, ticket: SupportTicket) -> None:
        raise NotImplementedError


class BillingSupportHandler(SupportHandler):
    def can_handle(self, ticket: SupportTicket) -> bool:
        return ticket.ticket_type == "BILLING"

    def process(self, ticket: SupportTicket) -> None:
        print(f"Billing handled {ticket.ticket_id}")
```

### Python Usage

```python
chain = BillingSupportHandler()
print(chain.handle(SupportTicket("ticket-1", "BILLING", "Refund needed")))
```

## 6. Where It Comes Handy in Real Life

Examples:

- Servlet filters.
- HTTP middleware.
- Validation pipelines.
- Logging chains.
- Authorization checks.
- Support escalation.
- Event bubbling in UI frameworks.
- Exception handling pipelines.

## 7. Advantages Over Normal Code Without Pattern

### Without Chain

```java
if (billing.canHandle(ticket)) {
    billing.handle(ticket);
} else if (technical.canHandle(ticket)) {
    technical.handle(ticket);
}
```

Problems:

- Sender knows handlers.
- Order is hard-coded.
- Adding handlers changes sender code.
- Reuse is limited.

### With Chain

```java
chain.handle(ticket);
```

Benefits:

- Sender is decoupled.
- Handler order can be configured.
- Handlers stay focused.
- New handlers can be inserted.

## 8. Where It Excels

Chain of Responsibility excels when:

- More than one object may handle a request.
- Handler is not known in advance.
- Handler order matters.
- You need configurable pipelines.
- Request processing should be extensible.

## 9. Where It Fails

It is a poor fit when:

- Exactly one handler is always known.
- Request must always be handled and missing handlers are unacceptable.
- The chain becomes very long and opaque.
- Handler order is too subtle.
- A simple map from type to handler would be clearer.

## 10. Prebuilt Libraries and Packages

### Java

Examples:

- Servlet `FilterChain`.
- Spring Security filter chain.
- Logging frameworks.
- Netty channel pipeline.
- Exception resolver chains.

### Python

Examples:

- Web middleware.
- Request hooks.
- Validation pipelines.
- ASGI/WSGI middleware.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples sender from receiver. | Flow can be hard to trace. |
| Supports configurable handler order. | Request may go unhandled. |
| Keeps handlers focused. | Order bugs can be subtle. |
| Easy to insert new handlers. | Long chains can add latency. |
| Works well for middleware. | Error handling must be designed. |

## 12. Real-World Identification Example

Scenario:

You are designing an HTTP request pipeline.

Steps:

- Authentication
- Authorization
- Rate limiting
- Request validation
- Handler dispatch

Should you use Chain of Responsibility?

Yes, especially if each step can accept, reject, modify, or forward the request.

## 13. MAANG Interview Triggers

Think Chain of Responsibility when you hear:

- Pipeline.
- Middleware.
- Request handlers.
- Escalation.
- Pass to next handler.
- Filter chain.
- Validation chain.
- Event bubbling.

### Interview-Ready Answer Format

1. Identify the request.
2. Define the handler interface.
3. Give each handler a next handler.
4. Decide stop vs continue behavior.
5. Configure chain order.
6. Mention trade-offs: debugging, ordering, and unhandled requests.

## 14. Common Mistakes

### Mistake 1: No Default Handler

If unhandled requests are invalid, add a final fallback handler.

### Mistake 2: Hidden Handler Order

Make chain order explicit in configuration or construction.

### Mistake 3: Handlers Doing Too Much

Each handler should own one responsibility.

### Mistake 4: Swallowing Errors

Define whether errors stop the chain or continue.

### Mistake 5: Using Chain Instead of Map Dispatch

If request type maps directly to one handler, a map may be simpler.

## 15. Chain of Responsibility vs Similar Patterns

| Pattern | Difference |
|---|---|
| Command | Command encapsulates a request. Chain routes a request through handlers. |
| Decorator | Decorator wraps one object to add behavior. Chain passes a request along multiple handlers. |
| Observer | Observer notifies all subscribers. Chain usually stops or proceeds in order. |
| Strategy | Strategy chooses one algorithm. Chain offers many handlers a chance. |
| Pipeline | Pipeline usually applies all stages. Chain may stop when handled. |

## 16. Chain Design Checklist

| Concern | Why it matters |
|---|---|
| Stop or continue | Defines core behavior. |
| Handler order | Changes results. |
| Fallback handler | Prevents silent misses. |
| Error policy | Avoids inconsistent failures. |
| Observability | Needed for debugging chains. |

## 17. Quick Revision Notes

- Request enters first handler.
- Handler may process or pass forward.
- Sender does not know final receiver.
- Great for middleware and filters.
- Order matters.
- Add fallback handling if needed.

## 18. Mini Exercise

Design a chain for loan approval.

Handlers:

- Basic eligibility
- Credit score check
- Income verification
- Risk review

Expected usage:

```java
ApprovalHandler chain = new EligibilityHandler(new CreditScoreHandler(new RiskReviewHandler(null)));
boolean approved = chain.handle(application);
```

## 19. Source Reference in This Repo

The repository's Chain of Responsibility implementation uses `RequestHandler` implementations and an `OrcKing` chain builder.

Useful files:

- [github-repo/chain-of-responsibility/README.md](../../github-repo/chain-of-responsibility/README.md)
- [github-repo/chain-of-responsibility/src/main/java/com/iluwatar/chain/RequestHandler.java](../../github-repo/chain-of-responsibility/src/main/java/com/iluwatar/chain/RequestHandler.java)
- [github-repo/chain-of-responsibility/src/main/java/com/iluwatar/chain/Request.java](../../github-repo/chain-of-responsibility/src/main/java/com/iluwatar/chain/Request.java)
- [github-repo/chain-of-responsibility/src/main/java/com/iluwatar/chain/OrcKing.java](../../github-repo/chain-of-responsibility/src/main/java/com/iluwatar/chain/OrcKing.java)
