# Builder Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/builder](../../github-repo/builder)

## 1. Technical Definition

Builder is a creational design pattern that separates the construction of a complex object from its final representation. Instead of exposing one large constructor with many parameters, a separate builder object collects required and optional values step by step, validates them, and then creates the final object in one controlled `build()` operation.

Core idea:

- Keep the target object clean, usually immutable.
- Put construction complexity inside a builder.
- Make object creation readable through named methods.
- Avoid telescoping constructors and confusing parameter order.

## 2. Layman and Easy to Understand Definition

Builder is like ordering a custom pizza or building a laptop online.

You do not give one giant command like:

```text
Make laptop: 16 inch, 32 GB RAM, black, 1 TB, warranty yes, engraving no, charger type C...
```

Instead, you choose step by step:

```text
Choose screen size.
Choose RAM.
Choose storage.
Choose color.
Choose warranty.
Place order.
```

The final laptop is created only after all choices are collected. That step-by-step object creation is the Builder pattern.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose an object has required fields and many optional fields.

Example: a delivery order may need:

- Required: customer id, restaurant id
- Optional: delivery address, coupon code, contactless delivery, tip amount, item notes

Without Builder, you often get constructor overloads like this:

```java
new DeliveryOrder(customerId, restaurantId);
new DeliveryOrder(customerId, restaurantId, address);
new DeliveryOrder(customerId, restaurantId, address, coupon);
new DeliveryOrder(customerId, restaurantId, address, coupon, contactless);
```

This becomes constructor pollution. It is hard to remember parameter order, easy to pass wrong values, and painful to extend later.

### 3.2 The Builder Solution

Builder splits the process into two objects:

| Part | Responsibility |
|---|---|
| Final object | Holds the completed data and behavior. |
| Builder object | Collects construction data step by step. |

The usage usually looks like this:

```java
DeliveryOrder order = DeliveryOrder.builder("customer-1", "restaurant-9")
    .addItem("Veg Burger")
    .addItem("Fries")
    .deliveryAddress("221B Baker Street")
    .couponCode("WELCOME10")
    .contactlessDelivery(true)
    .build();
```

Each method name explains what is being set. The final object appears only when `build()` is called.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Product | The final object you want to create, such as `DeliveryOrder`. |
| Builder | A helper object that stores temporary construction state. |
| Fluent methods | Methods like `addItem()` and `couponCode()` that return the builder itself. |
| `build()` | Final step that validates input and returns the completed object. |

### 3.4 Mental Model

Think of Builder as a temporary form.

1. You fill required fields.
2. You add optional details.
3. You validate the form.
4. You submit it.
5. The system creates a final immutable object.

## 4. Java Coding Example

This example builds a `DeliveryOrder` object. The order has required fields and optional fields.

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

public final class DeliveryOrder {
    private final String customerId;
    private final String restaurantId;
    private final List<String> items;
    private final String deliveryAddress;
    private final String couponCode;
    private final boolean contactlessDelivery;
    private final BigDecimal tipAmount;

    private DeliveryOrder(Builder builder) {
        this.customerId = builder.customerId;
        this.restaurantId = builder.restaurantId;
        this.items = List.copyOf(builder.items);
        this.deliveryAddress = builder.deliveryAddress;
        this.couponCode = builder.couponCode;
        this.contactlessDelivery = builder.contactlessDelivery;
        this.tipAmount = builder.tipAmount;
    }

    public static Builder builder(String customerId, String restaurantId) {
        return new Builder(customerId, restaurantId);
    }

    public static final class Builder {
        private final String customerId;
        private final String restaurantId;
        private final List<String> items = new ArrayList<>();
        private String deliveryAddress;
        private String couponCode;
        private boolean contactlessDelivery;
        private BigDecimal tipAmount = BigDecimal.ZERO;

        private Builder(String customerId, String restaurantId) {
            this.customerId = Objects.requireNonNull(customerId, "customerId is required");
            this.restaurantId = Objects.requireNonNull(restaurantId, "restaurantId is required");
        }

        public Builder addItem(String item) {
            this.items.add(Objects.requireNonNull(item, "item is required"));
            return this;
        }

        public Builder deliveryAddress(String deliveryAddress) {
            this.deliveryAddress = deliveryAddress;
            return this;
        }

        public Builder couponCode(String couponCode) {
            this.couponCode = couponCode;
            return this;
        }

        public Builder contactlessDelivery(boolean contactlessDelivery) {
            this.contactlessDelivery = contactlessDelivery;
            return this;
        }

        public Builder tipAmount(BigDecimal tipAmount) {
            this.tipAmount = Objects.requireNonNull(tipAmount, "tipAmount is required");
            return this;
        }

        public DeliveryOrder build() {
            if (items.isEmpty()) {
                throw new IllegalStateException("At least one item is required");
            }
            if (deliveryAddress == null || deliveryAddress.isBlank()) {
                throw new IllegalStateException("Delivery address is required");
            }
            if (tipAmount.compareTo(BigDecimal.ZERO) < 0) {
                throw new IllegalStateException("Tip amount cannot be negative");
            }
            return new DeliveryOrder(this);
        }
    }

    @Override
    public String toString() {
        return "DeliveryOrder{" +
            "customerId='" + customerId + '\'' +
            ", restaurantId='" + restaurantId + '\'' +
            ", items=" + items +
            ", deliveryAddress='" + deliveryAddress + '\'' +
            ", couponCode='" + couponCode + '\'' +
            ", contactlessDelivery=" + contactlessDelivery +
            ", tipAmount=" + tipAmount +
            '}';
    }
}
```

### Java Block by Block Explanation

#### Imports

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
```

These are normal Java utility classes.

- `BigDecimal` is used for money-like values such as tip amount.
- `ArrayList` is used inside the builder while we are still collecting items.
- `List.copyOf()` is used later to make the final order's item list immutable.
- `Objects.requireNonNull()` gives clear errors for required values.

#### Final Product Class

```java
public final class DeliveryOrder {
```

`final` means no class can extend `DeliveryOrder`. This is common with builder-created objects because they are often designed to be immutable and predictable.

#### Final Fields

```java
private final String customerId;
private final String restaurantId;
private final List<String> items;
```

The final object stores its data in `final` fields. Once the object is created, these references cannot be changed.

#### Private Constructor

```java
private DeliveryOrder(Builder builder) {
    this.customerId = builder.customerId;
    this.restaurantId = builder.restaurantId;
    this.items = List.copyOf(builder.items);
}
```

The constructor is private, so outside code cannot create `DeliveryOrder` directly. It must go through the builder. This gives one controlled construction path.

`List.copyOf(builder.items)` is important. It prevents outside code from mutating the final order's items through the builder's mutable list.

#### Static Builder Entry Point

```java
public static Builder builder(String customerId, String restaurantId) {
    return new Builder(customerId, restaurantId);
}
```

This is a clean entry point. It makes usage readable:

```java
DeliveryOrder.builder("customer-1", "restaurant-9")
```

Required fields are forced at the beginning.

#### Builder Class

```java
public static final class Builder {
```

The builder is nested inside `DeliveryOrder` because it exists only to create `DeliveryOrder`. `static` means it does not require an existing `DeliveryOrder` object.

#### Required Builder Fields

```java
private final String customerId;
private final String restaurantId;
```

These are required and final inside the builder. The caller must provide them to start building.

#### Optional Builder Fields

```java
private final List<String> items = new ArrayList<>();
private String deliveryAddress;
private String couponCode;
private boolean contactlessDelivery;
private BigDecimal tipAmount = BigDecimal.ZERO;
```

These fields are optional or built step by step. The builder can keep mutable state because it is temporary. The final object should not expose that mutability.

#### Fluent Methods

```java
public Builder addItem(String item) {
    this.items.add(Objects.requireNonNull(item, "item is required"));
    return this;
}
```

This method updates the builder and returns `this`, allowing method chaining.

```java
.addItem("Veg Burger")
.addItem("Fries")
```

The same pattern appears in `deliveryAddress()`, `couponCode()`, `contactlessDelivery()`, and `tipAmount()`.

#### Build Method

```java
public DeliveryOrder build() {
    if (items.isEmpty()) {
        throw new IllegalStateException("At least one item is required");
    }
    return new DeliveryOrder(this);
}
```

`build()` is the gatekeeper.

It performs final validation and creates the real object. If something is missing or invalid, it fails before the object exists.

### Java Usage

```java
import java.math.BigDecimal;

public class Demo {
    public static void main(String[] args) {
        DeliveryOrder order = DeliveryOrder.builder("customer-1", "restaurant-9")
            .addItem("Veg Burger")
            .addItem("Fries")
            .deliveryAddress("221B Baker Street")
            .couponCode("WELCOME10")
            .contactlessDelivery(true)
            .tipAmount(new BigDecimal("2.50"))
            .build();

        System.out.println(order);
    }
}
```

#### Usage Explanation

- `builder("customer-1", "restaurant-9")` starts construction with required values.
- `addItem()` adds one item at a time.
- `deliveryAddress()` sets an optional but practically required delivery detail.
- `couponCode()` adds optional discount information.
- `contactlessDelivery()` adds a boolean flag without confusing constructor order.
- `tipAmount()` sets a money value clearly.
- `build()` validates and creates the final order.

## 5. Python Coding Example

Python has keyword arguments and dataclasses, so Builder is less frequently necessary than in Java. Still, it is useful when object creation has multiple steps, validation, or a fluent API requirement.

```python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class DeliveryOrder:
    customer_id: str
    restaurant_id: str
    items: tuple[str, ...]
    delivery_address: str
    coupon_code: str | None = None
    contactless_delivery: bool = False
    tip_amount: Decimal = Decimal("0.00")


class DeliveryOrderBuilder:
    def __init__(self, customer_id: str, restaurant_id: str) -> None:
        if not customer_id:
            raise ValueError("customer_id is required")
        if not restaurant_id:
            raise ValueError("restaurant_id is required")

        self._customer_id = customer_id
        self._restaurant_id = restaurant_id
        self._items: list[str] = []
        self._delivery_address: str | None = None
        self._coupon_code: str | None = None
        self._contactless_delivery = False
        self._tip_amount = Decimal("0.00")

    def add_item(self, item: str) -> "DeliveryOrderBuilder":
        if not item:
            raise ValueError("item is required")
        self._items.append(item)
        return self

    def delivery_address(self, address: str) -> "DeliveryOrderBuilder":
        self._delivery_address = address
        return self

    def coupon_code(self, coupon_code: str) -> "DeliveryOrderBuilder":
        self._coupon_code = coupon_code
        return self

    def contactless_delivery(self, enabled: bool) -> "DeliveryOrderBuilder":
        self._contactless_delivery = enabled
        return self

    def tip_amount(self, amount: Decimal) -> "DeliveryOrderBuilder":
        self._tip_amount = amount
        return self

    def build(self) -> DeliveryOrder:
        if not self._items:
            raise ValueError("at least one item is required")
        if not self._delivery_address:
            raise ValueError("delivery address is required")
        if self._tip_amount < Decimal("0.00"):
            raise ValueError("tip amount cannot be negative")

        return DeliveryOrder(
            customer_id=self._customer_id,
            restaurant_id=self._restaurant_id,
            items=tuple(self._items),
            delivery_address=self._delivery_address,
            coupon_code=self._coupon_code,
            contactless_delivery=self._contactless_delivery,
            tip_amount=self._tip_amount,
        )
```

### Python Block by Block Explanation

#### Dataclass Product

```python
@dataclass(frozen=True)
class DeliveryOrder:
```

The final object is a frozen dataclass. Frozen means the final order should not be mutated after creation.

#### Required and Optional Fields

```python
customer_id: str
restaurant_id: str
items: tuple[str, ...]
delivery_address: str
coupon_code: str | None = None
```

The dataclass clearly describes the shape of the final object. Some fields are required, and some have defaults.

#### Builder Constructor

```python
def __init__(self, customer_id: str, restaurant_id: str) -> None:
```

The builder starts with required values. It validates them immediately.

#### Temporary Mutable State

```python
self._items: list[str] = []
self._delivery_address: str | None = None
```

The builder can be mutable because it is not the final object. It is just the construction workspace.

#### Fluent Methods

```python
def add_item(self, item: str) -> "DeliveryOrderBuilder":
    self._items.append(item)
    return self
```

Returning `self` enables chaining.

#### Build Method

```python
def build(self) -> DeliveryOrder:
```

The build method validates all final rules and returns a frozen `DeliveryOrder`.

`tuple(self._items)` converts the mutable list into an immutable tuple for the final object.

### Python Usage

```python
from decimal import Decimal


order = (
    DeliveryOrderBuilder("customer-1", "restaurant-9")
    .add_item("Veg Burger")
    .add_item("Fries")
    .delivery_address("221B Baker Street")
    .coupon_code("WELCOME10")
    .contactless_delivery(True)
    .tip_amount(Decimal("2.50"))
    .build()
)

print(order)
```

## 6. Where It Comes Handy in Real Life

Builder is useful when real object creation has multiple optional decisions.

### Running Example: Food Delivery Checkout

Imagine a food delivery checkout flow.

1. User selects restaurant.
2. User adds items.
3. User applies coupon.
4. User chooses delivery address.
5. User enables contactless delivery.
6. User adds tip.
7. System validates and creates the final order.

This is not a single-step creation. It is naturally step by step. Builder matches this flow cleanly.

Other real-world places:

- Building HTTP requests.
- Creating immutable configuration objects.
- Building SQL queries.
- Creating test data.
- Building complex API request payloads.
- Creating UI components with many optional settings.
- Creating domain objects with required and optional fields.

## 7. Advantages Over Normal Code Without Pattern

### Without Builder

```java
new DeliveryOrder("customer-1", "restaurant-9", List.of("Burger"), "221B Baker Street", null, true, new BigDecimal("2.50"));
```

Problems:

- Hard to read.
- Easy to swap parameters.
- `null` arguments are unclear.
- Boolean arguments are unclear.
- Adding a new optional field can break constructors.
- Validation logic may get scattered.

### With Builder

```java
DeliveryOrder.builder("customer-1", "restaurant-9")
    .addItem("Burger")
    .deliveryAddress("221B Baker Street")
    .contactlessDelivery(true)
    .tipAmount(new BigDecimal("2.50"))
    .build();
```

Benefits:

- More readable.
- Safer parameter meaning.
- Optional fields are natural.
- Validation is centralized in `build()`.
- Final object can be immutable.
- New optional fields can be added without breaking older call sites.

## 8. Where It Excels

Builder excels when:

- The object has many constructor parameters.
- Some fields are required and many are optional.
- You want immutable objects.
- You need validation before final object creation.
- Object creation happens in steps.
- Method names improve readability.
- You want to avoid long constructor overload chains.
- You build configuration, request, command, or domain objects.

## 9. Where It Fails

Builder is a poor fit when:

- The object has only two or three simple fields.
- The object is always created in one obvious way.
- You add a builder to every class by habit.
- The builder duplicates too much logic from the product class.
- You forget validation and let invalid objects be built.
- The builder is reused across threads. Builders are usually mutable and not thread-safe.

Example where Builder is overkill:

```java
record Point(int x, int y) {}
```

For `Point`, a constructor is clearer than a builder.

## 10. Prebuilt Libraries and Packages

### Java

Useful libraries and tools:

- Lombok `@Builder`: generates builder code at compile time.
- Immutables: generates immutable objects and builders.
- AutoValue with builders: Google library for immutable value types.
- FreeBuilder: generates fluent builder APIs.
- Java records: not a builder library, but useful for simple immutable data carriers.

Common Java APIs with builder-like style:

- `StringBuilder`
- `StringBuffer`
- `Stream.Builder`
- `HttpRequest.newBuilder()` in Java 11+
- Many Spring, Netty, AWS SDK, and gRPC configuration APIs

### Python

Python often uses alternatives instead of formal builders:

- `dataclasses`
- `attrs`
- `pydantic`
- Keyword arguments with defaults
- Factory functions
- Fluent custom builder classes when construction is genuinely multi-step

In Python, ask first: can keyword arguments or a dataclass solve this more simply? If yes, skip Builder.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Makes complex construction readable. | Adds extra class or code. |
| Avoids telescoping constructors. | Can be overused for simple objects. |
| Helps create immutable final objects. | Builder itself is usually mutable. |
| Centralizes validation in `build()`. | More boilerplate without code generation. |
| Handles optional fields cleanly. | Callers may forget to call `build()`. |
| Makes method chaining expressive. | Bad builder design can hide required fields. |

## 12. Real-World Identification Example

Scenario:

You are designing a payment request object for a checkout system.

Fields:

- Required: customer id, amount, currency
- Optional: coupon, shipping address, billing address, fraud metadata, device id, idempotency key, retry policy, tax details

Should you use Builder?

Yes.

Why:

- The object has many optional fields.
- Some fields must be validated together.
- You want the final payment request to be immutable.
- You want call sites to be readable.
- You may add more optional fields later.

Good builder usage:

```java
PaymentRequest request = PaymentRequest.builder("customer-1", Money.usd("49.99"))
    .idempotencyKey("checkout-123")
    .coupon("WELCOME10")
    .deviceId("device-9")
    .fraudMetadata(metadata)
    .build();
```

This is better than passing ten parameters to a constructor.

## 13. MAANG Interview Triggers

Think Builder when you hear:

- Too many constructor parameters.
- Optional fields.
- Immutable object creation.
- Fluent API.
- Complex configuration object.
- Readable object construction.
- Avoid telescoping constructors.
- Need validation before constructing the final object.

## 14. Common Mistakes

### Mistake 1: Builder for Every Class

Do not use Builder for tiny objects.

Bad:

```java
UserId.builder().value("u1").build();
```

Better:

```java
new UserId("u1");
```

### Mistake 2: No Validation in `build()`

If `build()` creates invalid objects, the builder is not doing its job.

### Mistake 3: Mutable Final Object

If the final object exposes mutable lists or maps directly, callers can change it after construction.

Use defensive copies:

```java
this.items = List.copyOf(builder.items);
```

### Mistake 4: Required Fields Hidden as Optional Methods

If a field is truly required, prefer forcing it in the builder constructor or staged builder API.

## 15. Builder vs Similar Patterns

| Pattern | Difference |
|---|---|
| Factory Method | Chooses which subtype or product to create. Builder focuses on assembling one complex object. |
| Abstract Factory | Creates families of related objects. Builder configures one complex object step by step. |
| Prototype | Copies an existing object. Builder creates from explicit input. |
| Step Builder | A stricter Builder variation that forces construction order through types. |

## 16. Quick Revision Notes

- Builder solves constructor pollution.
- Best for complex objects with optional fields.
- Common in Java because Java lacks named constructor parameters.
- Helps create immutable objects.
- `build()` should validate final consistency.
- Avoid it for very simple objects.

## 17. Mini Exercise

Design a builder for `EmailMessage`.

Required fields:

- `from`
- `to`
- `subject`

Optional fields:

- `cc`
- `bcc`
- `attachments`
- `htmlBody`
- `plainTextBody`
- `priority`

Validation rules:

- At least one body must be present.
- `from` and `to` cannot be blank.
- Attachment names cannot be blank.

Expected usage:

```java
EmailMessage message = EmailMessage.builder("noreply@app.com", "user@example.com", "Welcome")
    .plainTextBody("Thanks for signing up")
    .priority(Priority.NORMAL)
    .build();
```

## 18. Source Reference in This Repo

The repository's Builder implementation uses a `Hero` object with a nested `Hero.Builder`.

Useful files:

- [github-repo/builder/README.md](../../github-repo/builder/README.md)
- [github-repo/builder/src/main/java/com/iluwatar/builder/Hero.java](../../github-repo/builder/src/main/java/com/iluwatar/builder/Hero.java)
- [github-repo/builder/src/main/java/com/iluwatar/builder/App.java](../../github-repo/builder/src/main/java/com/iluwatar/builder/App.java)
