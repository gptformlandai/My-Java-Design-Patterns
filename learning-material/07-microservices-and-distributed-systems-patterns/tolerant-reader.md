# Tolerant Reader Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/tolerant-reader](../../github-repo/tolerant-reader)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why consumers should ignore unknown fields from evolving producers.
2. Second pass: trace old consumer reading newer payloads without breaking.
3. Third pass: compare Tolerant Reader with schema versioning, anti-corruption layer, and consumer-driven contracts.

By the end, you should be able to say:

> Tolerant Reader makes consumers resilient by reading only the fields they need and ignoring unknown data.

## 1. Technical Definition

Tolerant Reader is an integration pattern where a consumer parses only the data it requires and safely ignores additional or unknown fields in a message or response.

Core idea:

- Producers evolve payloads over time.
- Consumers should not break on extra fields.
- Required fields are validated.
- Unknown fields are ignored.
- Services can evolve more independently.

### 30-Second Interview Answer

I would use Tolerant Reader when services exchange JSON, XML, events, or serialized data that may evolve independently. Consumers should require only the fields they actually need and ignore unknown additions. The trade-off is that ignoring unknown fields can hide producer mistakes, so required fields, schema tests, and contract tests are still important.

## 2. Layman and Easy to Understand Definition

Tolerant Reader is like reading a form and caring only about the fields needed for your job.

If the form adds a new optional field tomorrow, your process does not fail just because you do not use it.

In code:

- Read required fields.
- Ignore unknown fields.
- Apply defaults for missing optional fields.
- Fail only when required data is invalid.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Distributed systems evolve:

```json
{
  "id": "u1",
  "name": "Asha",
  "tier": "gold"
}
```

Later, producer adds:

```json
{
  "id": "u1",
  "name": "Asha",
  "tier": "gold",
  "marketingSegment": "A"
}
```

A strict consumer may fail even though it does not need `marketingSegment`.

### 3.2 The Tolerant Reader Solution

Consumer reads only what it needs:

```text
required: id, name
optional: tier
unknown: ignore
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Producer | Sends payload or event. |
| Consumer | Reads payload. |
| Required fields | Fields consumer must have. |
| Optional fields | Fields with defaults or nullable behavior. |
| Unknown fields | Extra fields ignored by consumer. |
| Contract tests | Tests that prevent incompatible changes. |

### 3.4 Compatibility Flow

1. Producer sends payload.
2. Consumer parses known fields.
3. Consumer validates required fields.
4. Consumer ignores unknown fields.
5. Consumer applies defaults for missing optional fields.
6. Consumer continues working across additive producer changes.

## 4. Java Coding Example

This example uses a map to read only required fields.

```java
import java.util.Map;

record CustomerSummary(String id, String name, String tier) {
}

class CustomerReader {
    CustomerSummary read(Map<String, String> payload) {
        String id = require(payload, "id");
        String name = require(payload, "name");
        String tier = payload.getOrDefault("tier", "standard");

        return new CustomerSummary(id, name, tier);
    }

    private String require(Map<String, String> payload, String field) {
        String value = payload.get(field);
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("Missing required field: " + field);
        }
        return value;
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `payload` | Incoming data may contain extra fields. |
| `require` | Required data is still enforced. |
| `getOrDefault` | Optional fields can have safe defaults. |
| `CustomerSummary` | Consumer creates its own needed model. |
| ignored fields | Unknown additions do not break the reader. |

### Java Usage

```java
Map<String, String> payload = Map.of(
    "id", "u1",
    "name", "Asha",
    "tier", "gold",
    "newField", "ignored"
);

System.out.println(new CustomerReader().read(payload));
```

## 5. Python Coding Example

```python
def read_customer(payload):
    if not payload.get("id"):
        raise ValueError("missing id")
    if not payload.get("name"):
        raise ValueError("missing name")

    return {
        "id": payload["id"],
        "name": payload["name"],
        "tier": payload.get("tier", "standard"),
    }


incoming = {
    "id": "u1",
    "name": "Asha",
    "tier": "gold",
    "newField": "ignored",
}

print(read_customer(incoming))
```

### Python Usage

Use this shape when explaining:

- Unknown fields are ignored.
- Required fields are still validated.
- Consumer maps into its own model.

## 6. Where It Comes Handy in Real Life

- REST API clients.
- Event consumers.
- Message queue subscribers.
- JSON/XML integrations.
- Public API evolution.
- Versioned data migrations.

## 7. Advantages Over Normal Code Without Pattern

Without Tolerant Reader:

```text
consumer breaks when producer adds harmless fields
```

With Tolerant Reader:

```text
consumer ignores fields it does not need
```

Benefits:

- Improves backward compatibility.
- Reduces coordinated deployment needs.
- Allows additive schema evolution.
- Keeps consumers focused on their needs.
- Makes integrations more resilient.

## 8. Where It Excels

- Producers and consumers deploy independently.
- Payloads evolve additively.
- Consumers need only a subset of fields.
- Public APIs add optional fields.
- Event schemas evolve over time.

## 9. Where It Fails

- Producer removes required fields.
- Field meaning changes incompatibly.
- Unknown fields are actually important warnings.
- Consumer ignores validation too broadly.
- Contract tests are absent.

Tolerant reading helps additive changes; it does not make breaking changes safe.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java JSON | Jackson `ignoreUnknown`, Gson lenient parsing |
| Schema | Avro compatibility rules, Protobuf unknown field behavior, JSON Schema |
| Testing | Pact consumer-driven contracts, Spring Cloud Contract |
| Messaging | Kafka schema registry compatibility modes |
| APIs | OpenAPI with backward-compatible field additions |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Supports additive schema evolution. | Can hide producer mistakes. |
| Reduces tight coupling. | Required field rules still need care. |
| Helps independent deployments. | Debugging ignored data can be harder. |
| Keeps consumer model small. | Does not protect against semantic changes. |

## 12. Real-World Identification Example

Scenario: Order service publishes `OrderCreated` events. Billing service needs only order id, user id, and total.

Tolerant Reader fit:

- Billing reads required billing fields.
- New shipping fields are ignored.
- Billing does not need deployment when producer adds optional fields.
- Contract tests still protect required fields.

Without it:

- Billing may break on harmless event additions.
- Teams must coordinate every additive change.

## 13. MAANG Interview Triggers

Use Tolerant Reader when you hear:

- "How do APIs evolve without breaking clients?"
- "Producer adds fields."
- "Backward compatibility."
- "Consumers deploy independently."
- "Event schema evolution."
- "Ignore unknown JSON fields?"

Strong answer keywords:

- unknown fields
- required fields
- additive changes
- backward compatibility
- schema registry
- contract tests
- consumer model
- robustness principle

## 14. Common Mistakes

### Mistake 1: Ignoring missing required fields

- Why it is wrong: consumer may process invalid data.
- Better approach: tolerate unknown fields but validate required fields strictly.

### Mistake 2: Treating all schema changes as safe

- Why it is wrong: removals and semantic changes can still break consumers.
- Better approach: define compatibility rules and versioning.

### Mistake 3: Sharing producer DTOs directly

- Why it is wrong: consumer becomes coupled to full producer model.
- Better approach: map incoming data into consumer-owned models.

### Mistake 4: No contract tests

- Why it is wrong: producer may unknowingly break required fields.
- Better approach: use consumer-driven contracts for critical integrations.

## 15. Tolerant Reader vs Similar Patterns

| Pattern | Difference |
|---|---|
| Tolerant Reader | Consumer ignores unknown data and reads what it needs. |
| Anti-Corruption Layer | Translates between different domain models. |
| Adapter | Converts one interface to another. |
| Schema Versioning | Labels and manages payload versions. |
| Consumer-Driven Contract | Tests producer against consumer expectations. |

## 16. Tolerant Reader Design Checklist

- What fields are required?
- What fields are optional?
- What defaults are safe?
- Are unknown fields ignored by the parser?
- Are required fields contract-tested?
- What changes are considered breaking?
- Is the consumer model independent?
- Is invalid data observable through logs or metrics?

## 17. Quick Revision Notes

- One-line summary: Tolerant Reader ignores unknown fields while validating required ones.
- Three keywords: ignore, require, evolve.
- Interview trap: thinking tolerant means accepting invalid required data.
- Memory trick: read only your fields, leave the rest alone.

## 18. Mini Exercise

Design a tolerant reader for a payment event.

Answer these:

1. Which fields are required?
2. Which fields are optional?
3. What unknown fields are ignored?
4. What defaults are safe?
5. What contract test protects consumers?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/tolerant-reader/README.md](../../github-repo/tolerant-reader/README.md)
- [github-repo/tolerant-reader/src/main/java/com/iluwatar/tolerantreader/RainbowFishSerializer.java](../../github-repo/tolerant-reader/src/main/java/com/iluwatar/tolerantreader/RainbowFishSerializer.java)
- [github-repo/tolerant-reader/src/main/java/com/iluwatar/tolerantreader/App.java](../../github-repo/tolerant-reader/src/main/java/com/iluwatar/tolerantreader/App.java)
