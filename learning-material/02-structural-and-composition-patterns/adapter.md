# Adapter Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/adapter](../../github-repo/adapter)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the mismatch between the interface a client expects and the interface an existing class provides.
2. Second pass: rewrite the Java example and identify the target, adaptee, adapter, and client.
3. Third pass: compare Adapter with Proxy, Facade, and Decorator so you do not mix them up in interviews.

By the end, you should be able to say:

> Adapter lets existing incompatible code work with a client by translating one interface into another.

## 1. Technical Definition

Adapter is a structural design pattern that converts the interface of an existing class into another interface expected by client code. It allows incompatible classes to collaborate without changing either side's core code.

Core idea:

- The client expects a target interface.
- An existing object has a different interface.
- The adapter implements the target interface.
- The adapter delegates to the existing object and translates calls.

### 30-Second Interview Answer

I would use Adapter when I need to integrate an existing class, third-party library, or legacy system whose interface does not match what my application expects. I create an adapter that implements my application's target interface and delegates to the incompatible object. This keeps client code stable and avoids changing vendor or legacy code.

## 2. Layman and Easy to Understand Definition

Adapter is like a power plug adapter.

Your laptop charger may have a plug shape that does not fit the wall outlet. You do not change the laptop or rebuild the wall. You use an adapter in between.

In code:

- The laptop is the client.
- The wall outlet is the existing incompatible system.
- The plug adapter is the adapter class.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose your application expects this interface:

```java
public interface PaymentProcessor {
    PaymentResult pay(PaymentRequest request);
}
```

But a third-party payment SDK exposes this:

```java
public class LegacyPaymentClient {
    public LegacyPaymentResponse makeCharge(String accountId, long cents) {
        // third-party SDK call
    }
}
```

Problems:

- Your application expects `pay()`.
- The SDK exposes `makeCharge()`.
- Your domain uses `PaymentRequest`.
- The SDK wants primitive vendor-specific parameters.
- You do not own the SDK code.

### 3.2 The Adapter Solution

Create an adapter:

```java
public final class LegacyPaymentAdapter implements PaymentProcessor {
    private final LegacyPaymentClient client;

    public LegacyPaymentAdapter(LegacyPaymentClient client) {
        this.client = client;
    }

    @Override
    public PaymentResult pay(PaymentRequest request) {
        LegacyPaymentResponse response =
            client.makeCharge(request.accountId(), request.amountCents());
        return new PaymentResult(response.id(), response.successful());
    }
}
```

Now your application can use `PaymentProcessor`, while the adapter handles the SDK mismatch.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Target | Interface the client expects. |
| Client | Code that depends on the target interface. |
| Adaptee | Existing incompatible class or API. |
| Adapter | Class that implements the target and delegates to the adaptee. |
| Translation logic | Mapping between client concepts and adaptee concepts. |

### 3.4 Object Adapter vs Class Adapter

| Type | How it works | Java practicality |
|---|---|---|
| Object adapter | Adapter wraps an adaptee object and delegates to it. | Most common in Java. |
| Class adapter | Adapter subclasses the adaptee and implements the target. | Less common because Java has single inheritance. |

Most Java code uses object adapters because composition is flexible and works with final or third-party classes.

### 3.5 Mental Model

Think of Adapter as interface translation.

1. Client calls the interface it understands.
2. Adapter receives the call.
3. Adapter converts inputs into adaptee format.
4. Adapter calls the adaptee.
5. Adapter converts the result back into client format.

## 4. Java Coding Example

This example adapts a legacy payment client to a clean application interface.

```java
import java.util.Objects;

public record PaymentRequest(String accountId, long amountCents) {}

public record PaymentResult(String transactionId, boolean successful) {}

public interface PaymentProcessor {
    PaymentResult pay(PaymentRequest request);
}

public final class LegacyPaymentResponse {
    private final String chargeId;
    private final String status;

    public LegacyPaymentResponse(String chargeId, String status) {
        this.chargeId = chargeId;
        this.status = status;
    }

    public String chargeId() {
        return chargeId;
    }

    public String status() {
        return status;
    }
}

public final class LegacyPaymentClient {
    public LegacyPaymentResponse makeCharge(String accountId, long cents) {
        System.out.println("Charging " + accountId + " for " + cents + " cents");
        return new LegacyPaymentResponse("charge-123", "APPROVED");
    }
}

public final class LegacyPaymentAdapter implements PaymentProcessor {
    private final LegacyPaymentClient client;

    public LegacyPaymentAdapter(LegacyPaymentClient client) {
        this.client = Objects.requireNonNull(client);
    }

    @Override
    public PaymentResult pay(PaymentRequest request) {
        LegacyPaymentResponse response =
            client.makeCharge(request.accountId(), request.amountCents());

        boolean successful = "APPROVED".equals(response.status());
        return new PaymentResult(response.chargeId(), successful);
    }
}
```

### Java Block by Block Explanation

#### Target Interface

```java
public interface PaymentProcessor {
    PaymentResult pay(PaymentRequest request);
}
```

This is what the rest of your application wants to use.

#### Adaptee

```java
public final class LegacyPaymentClient {
```

This is the incompatible existing class. It has useful behavior, but its interface does not match your application.

#### Adapter

```java
public final class LegacyPaymentAdapter implements PaymentProcessor {
```

The adapter implements the target interface, so clients can use it as a normal `PaymentProcessor`.

#### Delegation

```java
client.makeCharge(request.accountId(), request.amountCents());
```

The adapter delegates real work to the adaptee.

#### Result Translation

```java
boolean successful = "APPROVED".equals(response.status());
return new PaymentResult(response.chargeId(), successful);
```

The adapter converts vendor-specific output into application output.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        LegacyPaymentClient legacyClient = new LegacyPaymentClient();
        PaymentProcessor processor = new LegacyPaymentAdapter(legacyClient);

        PaymentResult result = processor.pay(new PaymentRequest("customer-1", 4999));
        System.out.println(result);
    }
}
```

The client uses `PaymentProcessor` and does not know about `LegacyPaymentClient`.

## 5. Python Coding Example

Python adapters are often small wrapper classes.

```python
from dataclasses import dataclass
from typing import Protocol


@dataclass(frozen=True)
class PaymentRequest:
    account_id: str
    amount_cents: int


@dataclass(frozen=True)
class PaymentResult:
    transaction_id: str
    successful: bool


class PaymentProcessor(Protocol):
    def pay(self, request: PaymentRequest) -> PaymentResult:
        ...


class LegacyPaymentClient:
    def make_charge(self, account_id: str, cents: int) -> dict[str, str]:
        print(f"Charging {account_id} for {cents} cents")
        return {"charge_id": "charge-123", "status": "APPROVED"}


class LegacyPaymentAdapter:
    def __init__(self, client: LegacyPaymentClient) -> None:
        self._client = client

    def pay(self, request: PaymentRequest) -> PaymentResult:
        response = self._client.make_charge(request.account_id, request.amount_cents)
        return PaymentResult(
            transaction_id=response["charge_id"],
            successful=response["status"] == "APPROVED",
        )
```

### Python Usage

```python
processor: PaymentProcessor = LegacyPaymentAdapter(LegacyPaymentClient())
result = processor.pay(PaymentRequest("customer-1", 4999))
print(result)
```

## 6. Where It Comes Handy in Real Life

Adapter is extremely common in integration-heavy systems.

Examples:

- Wrapping third-party payment SDKs.
- Converting legacy APIs to new domain interfaces.
- Adapting vendor-specific cloud clients.
- Translating external DTOs into internal request objects.
- Wrapping old persistence APIs behind repository interfaces.
- Converting callback-based APIs into promise/future APIs.
- Adapting file formats or protocol-specific clients.

## 7. Advantages Over Normal Code Without Pattern

### Without Adapter

```java
LegacyPaymentResponse response = legacyClient.makeCharge(accountId, cents);
boolean successful = "APPROVED".equals(response.status());
```

Problems:

- Vendor-specific code spreads everywhere.
- Client code knows legacy naming and status values.
- Replacing the SDK becomes expensive.
- Domain logic gets mixed with integration translation.

### With Adapter

```java
PaymentProcessor processor = new LegacyPaymentAdapter(legacyClient);
PaymentResult result = processor.pay(request);
```

Benefits:

- Client code uses a clean interface.
- Vendor translation is isolated.
- Replacing the adaptee requires one new adapter.
- Tests can use fake target implementations.

## 8. Where It Excels

Adapter excels when:

- You must use existing code with an incompatible interface.
- You do not control the adaptee source code.
- You want to isolate third-party or legacy concepts.
- Multiple vendors should fit one internal interface.
- You want to protect domain code from integration details.

## 9. Where It Fails

Adapter is a poor fit when:

- Interfaces already match.
- You control both sides and can change the source interface cleanly.
- The adapter hides important behavior differences.
- Translation logic becomes too complex and should be its own service.
- You use it to force incompatible domain concepts together.

Example where Adapter is overkill:

```java
PaymentProcessor processor = new StripePaymentProcessor();
```

If `StripePaymentProcessor` already implements your target interface, no adapter is needed.

## 10. Prebuilt Libraries and Packages

### Java

Common adapter-like APIs:

- `InputStreamReader` adapts byte streams to character readers.
- `OutputStreamWriter` adapts character writers to byte streams.
- `Arrays.asList()` adapts an array to a list view.
- `Collections.enumeration()` adapts a collection to an enumeration.
- `XmlAdapter` adapts XML-bound types.

Implementation helpers:

- Composition.
- Interfaces.
- DTO mappers.
- Anti-corruption layers in larger systems.

### Python

Common Python adapter techniques:

- Wrapper classes.
- Functions that translate inputs and outputs.
- Protocols for target interfaces.
- Decorators around third-party callables.
- DTO/model conversion functions.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Integrates incompatible interfaces. | Adds another layer. |
| Isolates third-party or legacy APIs. | Translation logic can become complex. |
| Keeps client code stable. | Can hide semantic mismatches. |
| Supports vendor replacement. | Too many adapters can make flow harder to trace. |
| Encourages interface-based design. | Poor naming can obscure what is being adapted. |

## 12. Real-World Identification Example

Scenario:

You are replacing a payment provider but your checkout service already depends on `PaymentProcessor`.

New vendor SDK exposes:

- `createCharge(customerRef, amountInMinorUnits)`
- vendor status strings
- vendor exception types

Should you use Adapter?

Yes.

Good usage:

```java
PaymentProcessor processor = new NewVendorPaymentAdapter(vendorClient);
checkoutService = new CheckoutService(processor);
```

The checkout service remains stable while the adapter handles vendor translation.

## 13. MAANG Interview Triggers

Think Adapter when you hear:

- Incompatible interfaces.
- Legacy system integration.
- Third-party SDK wrapper.
- Convert one API shape to another.
- Client expects one interface but object provides another.
- Anti-corruption layer.
- Wrapper for compatibility.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the target interface the application wants.
2. Identify the incompatible adaptee.
3. Create an adapter that implements the target.
4. Delegate to the adaptee.
5. Translate inputs, outputs, and errors.
6. Mention trade-off: extra layer and possible semantic mismatch.

## 14. Common Mistakes

### Mistake 1: Letting Vendor Types Leak

Bad:

```java
PaymentResult pay(LegacyPaymentRequest request);
```

Better:

```java
PaymentResult pay(PaymentRequest request);
```

The target interface should use your application's concepts.

### Mistake 2: No Error Translation

If the adaptee throws vendor exceptions, translate them into application exceptions where appropriate.

### Mistake 3: Adapter Doing Business Logic

Adapters should translate interfaces. Keep core domain decisions in services.

### Mistake 4: Hiding Incompatible Semantics

If two systems mean different things by "approved", do not blindly map the values.

### Mistake 5: Confusing Adapter with Proxy

Adapter changes the interface. Proxy usually keeps the same interface and controls access.

## 15. Adapter vs Similar Patterns

| Pattern | Difference |
|---|---|
| Proxy | Same interface, controls access. Adapter changes an interface to match what the client expects. |
| Decorator | Same interface, adds behavior. Adapter changes interface compatibility. |
| Facade | Simplifies a subsystem. Adapter converts one interface to another. |
| Bridge | Separates abstraction from implementation for long-term variation. Adapter is often added after interfaces already mismatch. |
| Anti-Corruption Layer | A broader boundary around an external system. Adapter is often one class inside that boundary. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Target | `PaymentProcessor` | Interface expected by the client. |
| Client | Checkout service or demo code | Uses the target interface. |
| Adaptee | `LegacyPaymentClient` | Existing incompatible object. |
| Adapter | `LegacyPaymentAdapter` | Converts target calls to adaptee calls. |

The key interview sentence:

> Adapter is about interface compatibility, not access control.

## 17. Quick Revision Notes

- Adapter converts one interface into another.
- Client depends on the target interface.
- Adapter wraps and delegates to the adaptee.
- Use it for legacy or third-party integration.
- Translate inputs, outputs, and errors.
- Do not put core business logic inside adapters.

## 18. Mini Exercise

Design an Adapter for `ShippingProvider`.

Your application expects:

```java
interface ShippingProvider {
    ShippingLabel createLabel(Shipment shipment);
}
```

A vendor SDK provides:

```java
VendorLabel buyPostage(String destinationZip, int weightOunces);
```

Tasks:

- Create `VendorShippingAdapter`.
- Translate `Shipment` into vendor parameters.
- Translate `VendorLabel` into `ShippingLabel`.
- Translate vendor errors into application errors.

## 19. Source Reference in This Repo

The repository's Adapter implementation uses `FishingBoatAdapter` to make `FishingBoat` usable as a `RowingBoat` for `Captain`.

Useful files:

- [github-repo/adapter/README.md](../../github-repo/adapter/README.md)
- [github-repo/adapter/src/main/java/com/iluwatar/adapter/FishingBoatAdapter.java](../../github-repo/adapter/src/main/java/com/iluwatar/adapter/FishingBoatAdapter.java)
- [github-repo/adapter/src/main/java/com/iluwatar/adapter/RowingBoat.java](../../github-repo/adapter/src/main/java/com/iluwatar/adapter/RowingBoat.java)
- [github-repo/adapter/src/main/java/com/iluwatar/adapter/FishingBoat.java](../../github-repo/adapter/src/main/java/com/iluwatar/adapter/FishingBoat.java)
- [github-repo/adapter/src/main/java/com/iluwatar/adapter/Captain.java](../../github-repo/adapter/src/main/java/com/iluwatar/adapter/Captain.java)
