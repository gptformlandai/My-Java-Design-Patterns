# Strategy Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/strategy](../../github-repo/strategy)

## How to Study This Page

Use this page in three passes:

1. First pass: understand Strategy as interchangeable algorithms behind one interface.
2. Second pass: rewrite the Java example and explain context, strategy interface, and concrete strategies.
3. Third pass: compare Strategy with State, Template Method, and simple lambdas.

By the end, you should be able to say:

> Strategy lets a context choose from a family of interchangeable algorithms without hard-coding conditional logic.

## 1. Technical Definition

Strategy is a behavioral design pattern that defines a family of algorithms, encapsulates each one behind a common interface, and makes them interchangeable at runtime or configuration time.

Core idea:

- Put each algorithm in its own strategy.
- Context depends on the strategy interface.
- Strategy can be injected or changed.
- Replace large conditionals with polymorphism or functions.

### 30-Second Interview Answer

I would use Strategy when a class supports multiple ways to perform an operation, such as pricing, routing, sorting, payment, compression, or validation. Instead of putting many `if` or `switch` branches in the context, I define a strategy interface and implement each algorithm separately. This improves extensibility and testability, but too many tiny strategies can add unnecessary indirection.

## 2. Layman and Easy to Understand Definition

Strategy is like choosing a route option in a navigation app.

You can choose:

```text
fastest route
shortest route
avoid tolls
scenic route
```

The navigation app stays the same. Only the route algorithm changes.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose checkout supports different discount rules.

Bad design:

```java
if (discountType.equals("PERCENTAGE")) {
    total = total - total * 10 / 100;
} else if (discountType.equals("FIXED")) {
    total = total - 500;
} else if (discountType.equals("LOYALTY")) {
    total = total - loyaltyPoints;
}
```

Problems:

- The checkout class knows every algorithm.
- Adding a new rule changes existing code.
- Testing each algorithm requires the whole checkout class.
- Conditional logic grows over time.

### 3.2 The Strategy Solution

Move each algorithm behind a common interface:

```java
public interface DiscountStrategy {
    long apply(long subtotalCents);
}
```

Then inject the chosen strategy:

```java
CheckoutService checkout = new CheckoutService(new PercentageDiscountStrategy(10));
long total = checkout.totalAfterDiscount(10000);
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Strategy | Common interface for algorithms. |
| Concrete strategy | Specific algorithm implementation. |
| Context | Object that uses a strategy. |
| Client | Chooses or configures the strategy. |

### 3.4 Strategy Selection Timing

| Timing | Example |
|---|---|
| Compile time | Context always receives one strategy in constructor. |
| Startup time | Configuration chooses strategy. |
| Runtime | User request or feature flag chooses strategy. |
| Per call | Method parameter provides strategy for one operation. |

### 3.5 Mental Model

Think of Strategy as a swappable algorithm slot.

1. Context needs an operation.
2. Operation has multiple possible algorithms.
3. Define one strategy interface.
4. Implement each algorithm separately.
5. Context delegates to the selected strategy.

## 4. Java Coding Example

This example applies different discount strategies.

```java
public interface DiscountStrategy {
    long apply(long subtotalCents);
}

public final class NoDiscountStrategy implements DiscountStrategy {
    @Override
    public long apply(long subtotalCents) {
        return subtotalCents;
    }
}

public final class PercentageDiscountStrategy implements DiscountStrategy {
    private final int percentage;

    public PercentageDiscountStrategy(int percentage) {
        if (percentage < 0 || percentage > 100) {
            throw new IllegalArgumentException("percentage must be between 0 and 100");
        }
        this.percentage = percentage;
    }

    @Override
    public long apply(long subtotalCents) {
        return subtotalCents - (subtotalCents * percentage / 100);
    }
}

public final class FixedDiscountStrategy implements DiscountStrategy {
    private final long discountCents;

    public FixedDiscountStrategy(long discountCents) {
        this.discountCents = discountCents;
    }

    @Override
    public long apply(long subtotalCents) {
        return Math.max(0, subtotalCents - discountCents);
    }
}

public final class CheckoutService {
    private DiscountStrategy discountStrategy;

    public CheckoutService(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public void changeDiscountStrategy(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public long totalAfterDiscount(long subtotalCents) {
        return discountStrategy.apply(subtotalCents);
    }
}
```

### Java Block by Block Explanation

#### Strategy Interface

```java
public interface DiscountStrategy {
    long apply(long subtotalCents);
}
```

Every discount algorithm exposes the same operation.

#### Concrete Strategy

```java
public final class PercentageDiscountStrategy implements DiscountStrategy {
```

This class contains one algorithm and its own validation.

#### Context

```java
public final class CheckoutService {
```

The context uses a strategy but does not know the details of each algorithm.

#### Strategy Delegation

```java
return discountStrategy.apply(subtotalCents);
```

The context delegates the variable part of behavior to the strategy.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        CheckoutService checkout = new CheckoutService(
            new PercentageDiscountStrategy(10)
        );

        System.out.println(checkout.totalAfterDiscount(10000));

        checkout.changeDiscountStrategy(new FixedDiscountStrategy(2500));
        System.out.println(checkout.totalAfterDiscount(10000));
    }
}
```

### Java Lambda Strategy

If the strategy has one method, Java lambdas can be clean:

```java
DiscountStrategy holidayDiscount = subtotal -> subtotal - (subtotal * 15 / 100);
CheckoutService checkout = new CheckoutService(holidayDiscount);
```

Use classes when the algorithm has meaningful state, naming, validation, or tests.

## 5. Python Coding Example

Python can use protocols, functions, or callable objects.

```python
from typing import Protocol


class DiscountStrategy(Protocol):
    def apply(self, subtotal_cents: int) -> int:
        ...


class PercentageDiscountStrategy:
    def __init__(self, percentage: int) -> None:
        if percentage < 0 or percentage > 100:
            raise ValueError("percentage must be between 0 and 100")
        self._percentage = percentage

    def apply(self, subtotal_cents: int) -> int:
        return subtotal_cents - (subtotal_cents * self._percentage // 100)


class FixedDiscountStrategy:
    def __init__(self, discount_cents: int) -> None:
        self._discount_cents = discount_cents

    def apply(self, subtotal_cents: int) -> int:
        return max(0, subtotal_cents - self._discount_cents)


class CheckoutService:
    def __init__(self, discount_strategy: DiscountStrategy) -> None:
        self._discount_strategy = discount_strategy

    def total_after_discount(self, subtotal_cents: int) -> int:
        return self._discount_strategy.apply(subtotal_cents)
```

### Python Usage

```python
checkout = CheckoutService(PercentageDiscountStrategy(10))
print(checkout.total_after_discount(10_000))
```

### Python Callable Strategy

If you control the API, Python can use plain callables:

```python
from collections.abc import Callable


class CallableCheckoutService:
    def __init__(self, discount_strategy: Callable[[int], int]) -> None:
        self._discount_strategy = discount_strategy

    def total_after_discount(self, subtotal_cents: int) -> int:
        return self._discount_strategy(subtotal_cents)


def no_discount(subtotal_cents: int) -> int:
    return subtotal_cents


checkout = CallableCheckoutService(no_discount)
```

## 6. Where It Comes Handy in Real Life

Strategy is common wherever algorithms vary.

Examples:

- Sorting with comparators.
- Pricing and discounts.
- Payment provider selection.
- Routing algorithms.
- Compression algorithms.
- Validation policies.
- Retry policies.
- Authentication methods.
- Recommendation ranking formulas.

## 7. Advantages Over Normal Code Without Pattern

### Without Strategy

```java
switch (discountType) {
    case "PERCENTAGE" -> applyPercentage();
    case "FIXED" -> applyFixed();
    case "LOYALTY" -> applyLoyalty();
}
```

Problems:

- Context grows with every algorithm.
- Open/closed principle is weakened.
- Each algorithm is harder to test independently.
- Conditional logic becomes repetitive.

### With Strategy

```java
long total = discountStrategy.apply(subtotal);
```

Benefits:

- Algorithms are isolated.
- Context stays small.
- New algorithms can be added without editing context.
- Strategies can be tested independently.
- Strategy can be chosen by configuration or request.

## 8. Where It Excels

Strategy excels when:

- Several algorithms solve the same problem.
- The algorithm should vary independently from the context.
- You want to remove conditionals.
- The behavior may change at runtime.
- Each algorithm deserves separate tests.
- The client can choose the policy.

## 9. Where It Fails

Strategy is a poor fit when:

- There is only one algorithm.
- The algorithms are tiny and unlikely to change.
- Strategy objects create more complexity than the conditional.
- The context needs to know too much about each strategy.
- The behavior is state-dependent rather than policy-dependent.

## 10. Prebuilt Libraries and Packages

### Java

Common strategy-like APIs:

- `Comparator<T>`
- `Predicate<T>`
- `Function<T, R>`
- `Collector`
- Retry policies in resilience libraries
- Authentication providers
- Layout managers in GUI frameworks

Modern Java note:

- Many simple strategies can be represented as lambdas or method references.

### Python

Python strategy options:

- Functions.
- Callable objects.
- Protocols.
- Dictionaries mapping names to functions.
- Dependency injection.
- Plug-in registries.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Removes large conditionals. | Adds more classes or functions. |
| Encapsulates algorithms. | Client may need to choose strategy. |
| Improves testability. | Too many tiny strategies can feel noisy. |
| Supports runtime behavior changes. | Strategy interface must be designed carefully. |
| Follows open/closed principle. | Shared data between strategies can become awkward. |

## 12. Real-World Identification Example

Scenario:

You are designing a shipping quote service.

Algorithms:

- Cheapest shipping
- Fastest shipping
- Eco-friendly shipping
- Carrier-preferred shipping

Should you use Strategy?

Yes.

Good usage:

```java
ShippingQuoteService service = new ShippingQuoteService(new CheapestRouteStrategy());
Quote quote = service.quote(request);
```

The quote service stays stable while route selection algorithms evolve separately.

## 13. MAANG Interview Triggers

Think Strategy when you hear:

- Interchangeable algorithms.
- Choose behavior at runtime.
- Replace conditional logic.
- Policy object.
- Comparator.
- Pricing rule.
- Retry policy.
- Validation policy.
- Family of algorithms.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the variable algorithm.
2. Define the strategy interface.
3. Implement concrete strategies.
4. Inject or select a strategy in the context.
5. Delegate algorithm work to the strategy.
6. Mention trade-off: more objects and strategy selection complexity.

## 14. Common Mistakes

### Mistake 1: Strategy for One Algorithm

Do not create a strategy hierarchy if there is only one stable behavior.

### Mistake 2: Context Still Has a Huge Switch

If the context still selects every algorithm manually, consider moving selection to a factory, registry, or configuration layer.

### Mistake 3: Leaky Strategy Interface

Bad:

```java
apply(Order order, Coupon coupon, Customer customer, Database db)
```

If the strategy needs everything, the boundary may be wrong.

### Mistake 4: Confusing Strategy with State

Strategy is usually chosen by client/configuration. State changes behavior because internal state changes.

### Mistake 5: Too Many Tiny Classes

For very small algorithms, lambdas or enum strategies may be clearer.

## 15. Strategy vs Similar Patterns

| Pattern | Difference |
|---|---|
| State | Same structure, different intent. State changes behavior based on internal state transitions. Strategy is chosen as a policy/algorithm. |
| Template Method | Template Method uses inheritance for algorithm skeleton. Strategy uses composition for interchangeable algorithms. |
| Command | Command encapsulates a request/action. Strategy encapsulates an algorithm. |
| Decorator | Decorator adds behavior around an object. Strategy replaces the algorithm used by a context. |
| Factory | Factory can choose and create the appropriate strategy. |

## 16. Strategy Selection Styles

| Style | Example | Best use |
|---|---|---|
| Constructor injection | `new CheckoutService(strategy)` | Required policy. |
| Setter change | `changeStrategy(strategy)` | Runtime switching. |
| Method parameter | `sort(items, comparator)` | One-operation strategy. |
| Registry | `strategies.get(type)` | Config/request-based selection. |
| Lambda | `subtotal -> subtotal * 90 / 100` | Small single-method strategies. |

## 17. Quick Revision Notes

- Strategy encapsulates interchangeable algorithms.
- Context depends on strategy interface.
- Use it to remove growing conditionals.
- Strategies are easy to test independently.
- Lambdas can implement simple strategies in Java/Python.
- Do not confuse Strategy with State.

## 18. Mini Exercise

Design Strategy for `FraudScoringService`.

Strategies:

- `RulesBasedFraudStrategy`
- `MachineLearningFraudStrategy`
- `HybridFraudStrategy`

Rules:

- All strategies return a fraud score from 0 to 100.
- The service should not know algorithm internals.
- The selected strategy may come from configuration.

Expected usage:

```java
FraudScoringService service = new FraudScoringService(new RulesBasedFraudStrategy());
int score = service.score(transaction);
```

## 19. Source Reference in This Repo

The repository's Strategy implementation uses `DragonSlayingStrategy` and `DragonSlayer` to switch algorithms.

Useful files:

- [github-repo/strategy/README.md](../../github-repo/strategy/README.md)
- [github-repo/strategy/src/main/java/com/iluwatar/strategy/DragonSlayingStrategy.java](../../github-repo/strategy/src/main/java/com/iluwatar/strategy/DragonSlayingStrategy.java)
- [github-repo/strategy/src/main/java/com/iluwatar/strategy/DragonSlayer.java](../../github-repo/strategy/src/main/java/com/iluwatar/strategy/DragonSlayer.java)
- [github-repo/strategy/src/main/java/com/iluwatar/strategy/MeleeStrategy.java](../../github-repo/strategy/src/main/java/com/iluwatar/strategy/MeleeStrategy.java)
- [github-repo/strategy/src/main/java/com/iluwatar/strategy/LambdaStrategy.java](../../github-repo/strategy/src/main/java/com/iluwatar/strategy/LambdaStrategy.java)
