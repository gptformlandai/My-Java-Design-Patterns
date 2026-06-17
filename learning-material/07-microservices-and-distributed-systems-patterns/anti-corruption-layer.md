# Anti-Corruption Layer Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/anti-corruption-layer](../../github-repo/anti-corruption-layer)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the layer as a translator that protects your domain model.
2. Second pass: trace request mapping from modern model to external or legacy model and back.
3. Third pass: compare Anti-Corruption Layer with Adapter, Facade, Gateway, and Strangler.

By the end, you should be able to say:

> Anti-Corruption Layer protects one model from being polluted by another system's model.

## 1. Technical Definition

Anti-Corruption Layer is an integration pattern that places a translation boundary between systems with different domain models, APIs, data formats, or business semantics.

Core idea:

- Your system keeps its own clean model.
- External models are translated at the boundary.
- Integration details do not leak into business logic.
- Legacy or third-party semantics are isolated.
- Migration can happen without corrupting the new design.

### 30-Second Interview Answer

I would use an Anti-Corruption Layer when integrating a clean service with a legacy or external system that has different names, rules, data shapes, or semantics. Instead of letting those external concepts leak into the domain model, I create translators, adapters, and gateways at the boundary. The trade-off is extra mapping code, but it protects maintainability and lets systems evolve independently.

## 2. Layman and Easy to Understand Definition

Anti-Corruption Layer is like a translator between two teams that use different terminology.

Each team can keep speaking its own language. The translator converts meanings at the boundary so one team's confusing terms do not enter the other team's daily work.

In code:

- External request enters the boundary.
- Boundary translates it.
- Internal domain sees clean objects.
- Internal response is translated back if needed.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Legacy systems and external services often use different concepts:

```text
legacy field: cust_no
modern field: customerId

legacy state: X
modern state: CANCELLED
```

If the new system directly imports those names and rules, the new model becomes polluted.

### 3.2 The Anti-Corruption Layer Solution

Create a boundary:

```text
modern domain -> ACL -> legacy system
legacy system -> ACL -> modern domain
```

The ACL owns mapping, validation, translation, and integration quirks.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Internal model | Clean domain model owned by your service. |
| External model | Model from legacy, vendor, or another bounded context. |
| Translator | Maps between internal and external objects. |
| Gateway | Encapsulates external API calls. |
| Adapter | Converts interface or payload shape. |
| Boundary service | Coordinates translation and external communication. |

### 3.4 Request Flow

1. Internal service needs external data.
2. ACL converts internal request to external request.
3. ACL calls the external system.
4. ACL converts external response to internal model.
5. Domain logic uses only internal concepts.
6. External failures are translated to internal error types.

## 4. Java Coding Example

This example translates a legacy order shape into a modern domain shape.

```java
record LegacyOrder(String id, String customerName, String qty, String price) {
}

record Customer(String name) {
}

record ModernOrder(String id, Customer customer, int quantity, long priceInCents) {
}

class OrderTranslator {
    ModernOrder toModern(LegacyOrder legacy) {
        int quantity = Integer.parseInt(legacy.qty());
        long cents = Math.round(Double.parseDouble(legacy.price()) * 100);

        return new ModernOrder(
            legacy.id(),
            new Customer(legacy.customerName()),
            quantity,
            cents
        );
    }
}

class OrderAntiCorruptionLayer {
    private final OrderTranslator translator = new OrderTranslator();

    ModernOrder importOrder(LegacyOrder legacyOrder) {
        return translator.toModern(legacyOrder);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `LegacyOrder` | External model is not used directly by domain logic. |
| `ModernOrder` | Internal model stays clean. |
| `OrderTranslator` | Translation has one clear owner. |
| `parseInt` and price conversion | External quirks are handled at the boundary. |
| `OrderAntiCorruptionLayer` | Boundary exposes clean operations. |

### Java Usage

```java
LegacyOrder legacy = new LegacyOrder("o-1", "Asha", "2", "19.99");
ModernOrder modern = new OrderAntiCorruptionLayer().importOrder(legacy);

System.out.println(modern);
```

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class ModernOrder:
    order_id: str
    customer_name: str
    quantity: int
    price_in_cents: int


class OrderTranslator:
    def to_modern(self, legacy_order):
        return ModernOrder(
            order_id=legacy_order["id"],
            customer_name=legacy_order["customer"],
            quantity=int(legacy_order["qty"]),
            price_in_cents=round(float(legacy_order["price"]) * 100),
        )


translator = OrderTranslator()
print(translator.to_modern({"id": "o-1", "customer": "Asha", "qty": "2", "price": "19.99"}))
```

### Python Usage

Use this shape when explaining:

- External data stays at the edge.
- Mapping is explicit.
- Internal code works with stable internal language.

## 6. Where It Comes Handy in Real Life

- Legacy modernization.
- Vendor API integration.
- Domain-driven design bounded contexts.
- Mergers where systems use different models.
- Migrating monolith modules into microservices.
- Integrating batch systems with real-time services.

## 7. Advantages Over Normal Code Without Pattern

Without ACL:

```text
legacy names, types, and rules leak into new domain code
```

With ACL:

```text
translation stays at the boundary
```

Benefits:

- Protects domain model quality.
- Isolates legacy quirks.
- Reduces coupling to external APIs.
- Makes migration safer.
- Makes integration code testable.

## 8. Where It Excels

- External system has different semantics.
- Legacy model is messy or unstable.
- Clean domain model matters.
- Integration will last for a long time.
- Multiple systems must coexist during migration.

## 9. Where It Fails

- Systems already share the same model and language.
- The translation layer becomes a second domain model.
- Mapping rules are not tested.
- The ACL performs too much business workflow.
- Performance-sensitive paths do expensive transformation repeatedly.

Keep the ACL focused on translation and boundary protection.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java mapping | MapStruct, ModelMapper, Jackson, Gson |
| Integration | Apache Camel, Spring Integration, Spring Cloud OpenFeign |
| API boundary | REST clients, GraphQL clients, gRPC stubs |
| Validation | Jakarta Bean Validation, Hibernate Validator |
| Testing | WireMock, Testcontainers, contract tests |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Protects domain model from external pollution. | Adds translation code. |
| Isolates legacy quirks. | Mapping must be maintained. |
| Supports incremental modernization. | Can add latency. |
| Improves testability of integration logic. | Can duplicate some model fields. |

## 12. Real-World Identification Example

Scenario: A new order service must read from an older order platform during migration.

ACL fit:

- New service keeps `Order`, `Customer`, and `Shipment` models.
- ACL maps older field names and statuses.
- Legacy API errors are translated to modern error types.
- New domain logic never imports legacy DTOs.

Without it:

- Legacy terms appear everywhere.
- Future migration becomes harder.
- Business rules become confused across models.

## 13. MAANG Interview Triggers

Use Anti-Corruption Layer when you hear:

- "Integrate with legacy system."
- "Different bounded contexts."
- "Protect new domain model."
- "Vendor API has weird fields."
- "Modernization without rewriting everything."
- "How do we prevent model leakage?"

Strong answer keywords:

- bounded context
- translator
- adapter
- facade
- gateway
- domain language
- legacy isolation
- mapping tests

## 14. Common Mistakes

### Mistake 1: Letting external DTOs enter the domain

- Why it is wrong: external semantics spread through internal code.
- Better approach: map DTOs at the boundary.

### Mistake 2: Hiding business decisions inside mapping

- Why it is wrong: translators become hard to reason about.
- Better approach: mapping translates shape and language; domain services make business decisions.

### Mistake 3: No contract tests

- Why it is wrong: external API changes break translation silently.
- Better approach: use contract tests and sample payload tests.

### Mistake 4: Overbuilding for simple integrations

- Why it is wrong: a full ACL may be unnecessary for small stable APIs.
- Better approach: use a simple adapter when semantic mismatch is low.

## 15. Anti-Corruption Layer vs Similar Patterns

| Pattern | Difference |
|---|---|
| Anti-Corruption Layer | Protects domain semantics from another model. |
| Adapter | Converts one interface to another. |
| Facade | Simplifies access to a subsystem. |
| Gateway | Encapsulates calls to external resources. |
| Strangler | Migrates functionality gradually; often uses ACL at boundaries. |

## 16. Anti-Corruption Layer Design Checklist

- Which external concepts must not leak inward?
- What internal model should the domain use?
- Where are translators located?
- How are external errors mapped?
- Are mapping rules unit tested?
- Are external API contracts tested?
- What happens when external fields are missing?
- Is the ACL thin enough?

## 17. Quick Revision Notes

- One-line summary: ACL translates at the boundary to protect the domain model.
- Three keywords: translator, boundary, isolation.
- Interview trap: calling every adapter an ACL even when there is no semantic mismatch.
- Memory trick: keep dirty outside language outside.

## 18. Mini Exercise

Design an ACL between a new payment service and an older billing system.

Answer these:

1. What models differ?
2. Which fields need translation?
3. What errors need mapping?
4. What external terms should never enter the new domain?
5. What tests prove the ACL works?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/anti-corruption-layer/README.md](../../github-repo/anti-corruption-layer/README.md)
- [github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/AntiCorruptionLayer.java](../../github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/AntiCorruptionLayer.java)
- [github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/legacy/LegacyOrder.java](../../github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/legacy/LegacyOrder.java)
- [github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/legacy/LegacyShop.java](../../github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/legacy/LegacyShop.java)
- [github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/modern/ModernOrder.java](../../github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/modern/ModernOrder.java)
- [github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/modern/ModernShop.java](../../github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/system/modern/ModernShop.java)
- [github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/App.java](../../github-repo/anti-corruption-layer/src/main/java/com/iluwatar/corruption/App.java)
