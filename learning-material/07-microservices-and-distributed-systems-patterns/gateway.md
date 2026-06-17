# Gateway Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/gateway](../../github-repo/gateway)

## How to Study This Page

Use this page in three passes:

1. First pass: understand Gateway as a boundary object for external systems.
2. Second pass: trace how application code calls one clean interface instead of many remote APIs.
3. Third pass: compare Gateway with Adapter, Facade, Proxy, and Microservices API Gateway.

By the end, you should be able to say:

> Gateway wraps external services behind a clean internal interface so the rest of the application stays decoupled.

## 1. Technical Definition

Gateway is an integration pattern that encapsulates access to an external system, remote API, database, or infrastructure service behind a stable application-owned interface.

Core idea:

- External communication is isolated.
- Protocol details stay at the boundary.
- Application code calls a clean interface.
- External failures are translated into internal errors.
- External services can change without spreading changes across the codebase.

### 30-Second Interview Answer

I would use Gateway when application code needs to call external systems or remote services. Instead of scattering HTTP clients, credentials, retries, and response parsing everywhere, I hide those details behind a gateway interface. The trade-off is one more layer, but it improves testability, decoupling, and resilience.

## 2. Layman and Easy to Understand Definition

Gateway is like a company reception desk for outside vendors.

Employees do not call every vendor in a different way. They use one known desk, and that desk knows how to contact each vendor.

In code:

- Application asks gateway for something.
- Gateway talks to external system.
- Gateway converts the response.
- Application receives internal-friendly data.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

External systems usually have their own details:

- Different URLs.
- Different credentials.
- Different timeout behavior.
- Different payload shape.
- Different error codes.

If every service class handles these details directly, the code becomes hard to test and change.

### 3.2 The Gateway Solution

Create a boundary:

```text
domain/service code -> gateway interface -> external service
```

The gateway owns integration details.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Client code | Application code needing external capability. |
| Gateway interface | Stable contract used internally. |
| Gateway implementation | Calls the external system. |
| External service | Remote API, database, file system, or vendor. |
| Mapper | Converts external payloads to internal models. |

### 3.4 Request Flow

1. Application calls gateway method.
2. Gateway builds external request.
3. Gateway applies timeout, retry, auth, and headers.
4. Gateway sends the request.
5. Gateway maps external response to internal result.
6. Gateway translates failures to internal exceptions or result types.

## 4. Java Coding Example

This example wraps a remote tax service behind a gateway.

```java
interface TaxGateway {
    long calculateTaxInCents(String state, long subtotalInCents);
}

class RemoteTaxGateway implements TaxGateway {
    public long calculateTaxInCents(String state, long subtotalInCents) {
        if ("CA".equals(state)) {
            return Math.round(subtotalInCents * 0.0725);
        }
        return Math.round(subtotalInCents * 0.05);
    }
}

class CheckoutService {
    private final TaxGateway taxGateway;

    CheckoutService(TaxGateway taxGateway) {
        this.taxGateway = taxGateway;
    }

    long total(String state, long subtotalInCents) {
        return subtotalInCents + taxGateway.calculateTaxInCents(state, subtotalInCents);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `TaxGateway` | Application-owned boundary interface. |
| `RemoteTaxGateway` | External integration implementation. |
| `CheckoutService` | Business code depends on the gateway contract. |
| `total` | Business logic stays clean. |
| `calculateTaxInCents` | External complexity is hidden. |

### Java Usage

```java
CheckoutService checkout = new CheckoutService(new RemoteTaxGateway());
System.out.println(checkout.total("CA", 10_000));
```

## 5. Python Coding Example

```python
class TaxGateway:
    def calculate_tax(self, state, subtotal):
        if state == "CA":
            return round(subtotal * 0.0725)
        return round(subtotal * 0.05)


class CheckoutService:
    def __init__(self, tax_gateway):
        self.tax_gateway = tax_gateway

    def total(self, state, subtotal):
        return subtotal + self.tax_gateway.calculate_tax(state, subtotal)


checkout = CheckoutService(TaxGateway())
print(checkout.total("CA", 10000))
```

### Python Usage

Use this shape when explaining:

- Service code depends on an internal contract.
- Remote details are contained in one class.
- Tests can replace the gateway with a fake.

## 6. Where It Comes Handy in Real Life

- Payment provider integration.
- Tax calculation API.
- Shipping provider API.
- Third-party identity provider.
- Email/SMS provider.
- Database or file-system access boundary.

## 7. Advantages Over Normal Code Without Pattern

Without Gateway:

```text
external API details spread across business code
```

With Gateway:

```text
external API details stay in one boundary
```

Benefits:

- Reduces coupling.
- Improves testability.
- Centralizes retries and timeouts.
- Hides protocol details.
- Makes external API replacement easier.

## 8. Where It Excels

- External API is unstable or vendor-owned.
- Many parts of the system need the same external capability.
- You need consistent timeout and retry behavior.
- Business code should not know protocol details.
- You want fake implementations for tests.

## 9. Where It Fails

- The external call is trivial and used once.
- Gateway becomes a business workflow engine.
- It hides important failure semantics.
- It silently retries non-idempotent operations.
- It returns raw external DTOs.

Keep gateway methods aligned with application needs, not vendor endpoint names.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java HTTP | Spring WebClient, RestTemplate, Java HttpClient, OkHttp |
| Declarative clients | OpenFeign, Retrofit |
| Resilience | Resilience4j, Spring Retry, Failsafe |
| Testing | WireMock, MockWebServer, contract tests |
| Mapping | Jackson, MapStruct |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Isolates external systems. | Adds an extra layer. |
| Improves testability. | Can hide useful provider details. |
| Centralizes retries and error mapping. | Bad design can become a generic wrapper. |
| Keeps business code clean. | Needs contract and integration tests. |

## 12. Real-World Identification Example

Scenario: Checkout service calls a shipping provider for delivery estimates.

Gateway fit:

- Checkout calls `ShippingGateway`.
- Gateway handles provider URL, auth, timeout, and response mapping.
- Checkout receives a clean `DeliveryEstimate`.
- Provider changes affect gateway only.

Without it:

- Shipping API details spread across checkout code.
- Tests need real provider behavior.
- Provider migration is painful.

## 13. MAANG Interview Triggers

Use Gateway when you hear:

- "Integrate with an external service."
- "How do we hide vendor API details?"
- "How do we test code that calls remote systems?"
- "Where should retries and timeouts live?"
- "How do we replace a provider later?"

Strong answer keywords:

- boundary
- external API
- timeout
- retry
- mapping
- error translation
- fake implementation
- contract test

## 14. Common Mistakes

### Mistake 1: Returning external DTOs

- Why it is wrong: provider model leaks inward.
- Better approach: map to internal models at the gateway boundary.

### Mistake 2: Retrying unsafe operations

- Why it is wrong: duplicate side effects can happen.
- Better approach: retry only idempotent operations or use idempotency keys.

### Mistake 3: No timeout policy

- Why it is wrong: remote calls can hang and exhaust threads.
- Better approach: define connect, read, and overall timeout budgets.

### Mistake 4: Making gateway too generic

- Why it is wrong: it becomes a thin HTTP wrapper with no domain meaning.
- Better approach: expose application-level operations.

## 15. Gateway vs Similar Patterns

| Pattern | Difference |
|---|---|
| Gateway | Encapsulates access to external systems. |
| Adapter | Converts one interface to another. |
| Facade | Simplifies a subsystem. |
| Proxy | Controls access to another object or service. |
| API Gateway | Distributed edge gateway for client-to-microservice traffic. |

## 16. Gateway Design Checklist

- Which external system is hidden?
- What internal interface should callers use?
- What timeouts are required?
- Which failures are retryable?
- How are provider errors translated?
- What data mapping is needed?
- What fake implementation supports tests?
- What contract tests protect integration?

## 17. Quick Revision Notes

- One-line summary: Gateway hides external system access behind an internal interface.
- Three keywords: boundary, mapping, resilience.
- Interview trap: confusing Gateway with Microservices API Gateway.
- Memory trick: one clean door to an outside system.

## 18. Mini Exercise

Design a gateway for an SMS provider.

Answer these:

1. What internal method should callers use?
2. What external provider details are hidden?
3. What timeout and retry policy applies?
4. What response should callers receive?
5. How would you test it without the provider?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/gateway/README.md](../../github-repo/gateway/README.md)
- [github-repo/gateway/src/main/java/com/iluwatar/gateway/Gateway.java](../../github-repo/gateway/src/main/java/com/iluwatar/gateway/Gateway.java)
- [github-repo/gateway/src/main/java/com/iluwatar/gateway/GatewayFactory.java](../../github-repo/gateway/src/main/java/com/iluwatar/gateway/GatewayFactory.java)
- [github-repo/gateway/src/main/java/com/iluwatar/gateway/ExternalServiceA.java](../../github-repo/gateway/src/main/java/com/iluwatar/gateway/ExternalServiceA.java)
- [github-repo/gateway/src/main/java/com/iluwatar/gateway/ExternalServiceB.java](../../github-repo/gateway/src/main/java/com/iluwatar/gateway/ExternalServiceB.java)
- [github-repo/gateway/src/main/java/com/iluwatar/gateway/ExternalServiceC.java](../../github-repo/gateway/src/main/java/com/iluwatar/gateway/ExternalServiceC.java)
- [github-repo/gateway/src/main/java/com/iluwatar/gateway/App.java](../../github-repo/gateway/src/main/java/com/iluwatar/gateway/App.java)
