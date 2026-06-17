# Value Object Pattern

Category: Domain Modeling and Business Rule Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/value-object](../../github-repo/value-object)

## How to Study This Page

Use this page in three passes:

1. First pass: understand equality by value rather than identity.
2. Second pass: rewrite the Java example and identify immutability, validation, equality, and replacement instead of mutation.
3. Third pass: compare Value Object with Entity, Money, DTO, and Parameter Object.

By the end, you should be able to say:

> A Value Object represents a domain value whose equality is based on its attributes, not its identity.

## 1. Technical Definition

Value Object is a domain modeling pattern for small immutable objects that describe a concept and are equal when their values are equal.

Core idea:

- No conceptual identity.
- Equality is based on fields.
- Object is immutable.
- Invalid values are rejected at creation.
- Changes produce a new object.

### 30-Second Interview Answer

I would use Value Object for domain concepts like email address, date range, money, address, percentage, or quantity. They do not need their own identity; two instances with the same values are equal. I make them immutable and validate them in the constructor or factory. The benefit is safer, clearer domain code. The trade-off is extra types and object creation for very simple data.

## 2. Layman and Easy to Understand Definition

Value Object is like a measurement.

Two measurements of "10 kilograms" are equal because their values are the same. You do not care which physical object instance represents the measurement.

In code:

- `new Email("a@x.com")` equals another `new Email("a@x.com")`.
- You do not update an email in place.
- You create a new value.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Primitive obsession:

```java
String email = "bad-value";
BigDecimal amount = new BigDecimal("-10");
String currency = "???";
```

Problems:

- Invalid values travel through code.
- Business meaning is hidden.
- Validation is duplicated.
- Parameter order mistakes happen.
- Equality is unclear for grouped values.

### 3.2 The Value Object Solution

Create meaningful types:

```java
Email email = new Email("a@example.com");
DateRange range = new DateRange(start, end);
```

The object protects its own validity.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Value object | Immutable domain value. |
| Fields | Values that define equality. |
| Constructor/factory | Validates invariants. |
| Equality/hash code | Based on all relevant fields. |
| Client/domain object | Uses value object instead of primitives. |

### 3.4 Entity vs Value Object

Entity:

```text
User #123 remains same user even if email changes.
```

Value Object:

```text
Email("a@example.com") is equal to any other Email("a@example.com").
```

## 4. Java Coding Example

This example models an email address as a value object.

```java
import java.util.Objects;

public final class EmailAddress {
    private final String value;

    public EmailAddress(String value) {
        if (value == null || !value.contains("@")) {
            throw new IllegalArgumentException("Invalid email address");
        }
        this.value = value.toLowerCase();
    }

    public String value() {
        return value;
    }

    @Override
    public boolean equals(Object other) {
        if (this == other) {
            return true;
        }
        if (!(other instanceof EmailAddress that)) {
            return false;
        }
        return value.equals(that.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }

    @Override
    public String toString() {
        return value;
    }
}
```

### Java Block by Block Explanation

`final class` prevents subclass surprises.

`private final String value` makes the object immutable.

The constructor validates the domain invariant.

`equals` and `hashCode` use value, not object identity.

### Java Usage

Use Value Object in Java when:

- A concept has no identity.
- Equality should be value-based.
- Validation belongs with the value.
- You want to avoid primitive obsession.
- Immutability improves safety.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class EmailAddress:
    value: str

    def __post_init__(self):
        if "@" not in self.value:
            raise ValueError("Invalid email address")
        object.__setattr__(self, "value", self.value.lower())


email_a = EmailAddress("A@example.com")
email_b = EmailAddress("a@example.com")
print(email_a == email_b)
```

### Python Usage

Python value objects are often:

- Frozen dataclasses.
- Named tuples.
- Pydantic models with frozen config.
- Small immutable classes with validation.

## 6. Where It Comes Handy in Real Life

- Email address.
- Phone number.
- Money.
- Date range.
- Address.
- Percentage.
- Quantity.
- Coordinates.
- SKU/product code.

## 7. Advantages Over Normal Code Without Pattern

### Without Value Object

```java
registerUser(String email, String phone, String countryCode)
```

Problems:

- Invalid strings can pass.
- Parameters can be swapped.
- Validation is repeated.
- Meaning is hidden.

### With Value Object

```java
registerUser(EmailAddress email, PhoneNumber phone)
```

Benefits:

- Types communicate meaning.
- Values validate themselves.
- Equality is correct.
- Immutability reduces bugs.

## 8. Where It Excels

- Domain-driven design.
- Strongly typed business concepts.
- Immutable data.
- Validation-heavy primitives.
- Safe equality and hashing.
- Shared concepts across entities.

## 9. Where It Fails

- Values with true identity/lifecycle.
- Huge mutable structures.
- Simple local variables with no domain meaning.
- Performance-sensitive hot loops where allocation matters.
- Cases where validation rules are unstable and unclear.

## 10. Prebuilt Frameworks and Packages

### Java

- Java records.
- Lombok `@Value`.
- `java.time` classes.
- JSR 354 Money API.
- Bean Validation annotations.

### Python

- `dataclasses.dataclass(frozen=True)`.
- `typing.NamedTuple`.
- Pydantic frozen models.
- `attrs` frozen classes.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Makes domain concepts explicit. | Adds more small classes. |
| Enforces validation at creation. | Extra object allocation. |
| Immutable and thread-safe. | Requires correct equality design. |
| Avoids primitive obsession. | Can be overused for trivial locals. |
| Easy to test. | Persistence mapping may need converters. |

## 12. Real-World Identification Example

Scenario:

You are modeling a subscription.

Value Object fit:

- `EmailAddress`
- `Money`
- `BillingPeriod`
- `Percentage`
- `PlanCode`

What would go wrong without it:

- Invalid strings and numbers move around.
- Validation is scattered.
- Domain language is weak.

## 13. MAANG Interview Triggers

Use Value Object when you hear:

- "Equality by value."
- "No identity."
- "Immutable domain type."
- "Avoid primitive obsession."
- "Validate value once."
- "Money/date range/email."
- "DDD value object."

### Interview-Ready Answer Format

1. Identify concept with no identity.
2. Make it immutable.
3. Validate invariants at creation.
4. Implement equality/hash by fields.
5. Replace mutation with new values.
6. Mention entities for identity/lifecycle.

## 14. Common Mistakes

### Mistake 1: Mutable Value Object

Mutation breaks safe sharing and hashing.

### Mistake 2: Equality Uses Identity

Value objects must compare by relevant fields.

### Mistake 3: No Validation

Invalid value objects defeat the purpose.

### Mistake 4: Too Many Primitive Getters Used Everywhere

If every caller immediately unwraps the value, behavior may be misplaced.

### Mistake 5: Treating Entity as Value Object

Objects with lifecycle and identity should be entities.

## 15. Value Object vs Similar Patterns

| Pattern | Difference |
|---|---|
| Value Object | Immutable domain value with value-based equality. |
| Entity | Has identity and lifecycle. |
| DTO | Data transfer shape, often not domain behavior. |
| Money | Specific value object for amount and currency. |
| Parameter Object | Groups method arguments, not necessarily a domain value. |

## 16. Value Object Design Checklist

| Question | Why it matters |
|---|---|
| Does it have identity? | If yes, it may be an entity. |
| Is it immutable? | Core value object property. |
| What fields define equality? | Prevents equality bugs. |
| What invariants exist? | Validates at creation. |
| How is it persisted? | May need converters. |
| Is the type useful or overkill? | Avoids needless wrapping. |

## 17. Quick Revision Notes

- Value Object has no identity.
- Equality is based on values.
- It should be immutable.
- Validate at creation.
- Replace mutation with new instance.
- Money is a classic example.

## 18. Mini Exercise

Design Value Object for `DateRange`.

Fields:

- `startDate`
- `endDate`

Rules:

- Start must be before or equal to end.
- Equality by both dates.
- Add method `contains(date)`.

## 19. Source Reference in This Repo

The repository's Value Object implementation uses `HeroStat` as an immutable value with value-based equality.

Useful files:

- [github-repo/value-object/README.md](../../github-repo/value-object/README.md)
- [github-repo/value-object/src/main/java/com/iluwatar/value/object/HeroStat.java](../../github-repo/value-object/src/main/java/com/iluwatar/value/object/HeroStat.java)
- [github-repo/value-object/src/main/java/com/iluwatar/value/object/App.java](../../github-repo/value-object/src/main/java/com/iluwatar/value/object/App.java)

