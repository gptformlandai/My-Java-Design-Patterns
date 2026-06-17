# Specification Pattern

Category: Domain Modeling and Business Rule Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/specification](../../github-repo/specification)

## How to Study This Page

Use this page in three passes:

1. First pass: understand a specification as a reusable business rule object.
2. Second pass: rewrite the Java example and identify specification, candidate object, `isSatisfiedBy`, and rule composition.
3. Third pass: compare Specification with Strategy, Predicate, Repository query methods, and Chain of Responsibility.

By the end, you should be able to say:

> Specification turns business rules into reusable, composable objects that can check whether a candidate satisfies a condition.

## 1. Technical Definition

Specification is a domain modeling pattern that encapsulates a business rule or selection criterion as an object and allows rules to be combined using boolean logic.

Core idea:

- A specification checks a candidate object.
- Rules are named and reusable.
- Rules can be combined with `and`, `or`, and `not`.
- Application code stops repeating conditionals.
- Specifications can be used for validation, filtering, and querying.

### 30-Second Interview Answer

I would use Specification when business rules need to be reused, combined, tested, or passed around. Instead of scattering `if` conditions, I define rule objects such as `EligibleForDiscount` or `ActiveCustomerSpec`. Then I can compose them with `and`, `or`, and `not`. The trade-off is extra abstraction, and for simple one-off conditions a plain predicate or inline check is enough.

## 2. Layman and Easy to Understand Definition

Specification is like a checklist rule.

For example, "customer is active", "order total is above 100", and "country is US" are separate rules. You can combine them to create a bigger rule: active customer AND order total above 100 AND country is US.

In code:

- One rule is one specification.
- Candidate object is checked against the rule.
- Rules can be reused and combined.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Business rules often start as conditionals:

```java
if (customer.isActive() && order.total().compareTo(BigDecimal.valueOf(100)) > 0) {
    applyDiscount(order);
}
```

This is fine once, but problems appear when:

- The same rule is needed in many places.
- Rules are combined differently for different flows.
- Rules need names for communication with business teams.
- Rules need unit tests.
- Rules need to be translated into database queries.

### 3.2 The Specification Solution

Move the rule into an object:

```java
Specification<Order> eligible = new ActiveCustomerSpec()
    .and(new OrderTotalAtLeastSpec(new BigDecimal("100")));

if (eligible.isSatisfiedBy(order)) {
    applyDiscount(order);
}
```

The rule is now explicit and reusable.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Specification | Rule object with `isSatisfiedBy`. |
| Candidate | Object being checked. |
| Concrete specification | One business condition. |
| Composite specification | Combines rules with boolean logic. |
| Client | Uses specification for validation/filter/query. |

### 3.4 Rule Composition

Specifications usually support:

```java
specA.and(specB)
specA.or(specB)
specA.not()
```

This is useful when business rules are built from smaller pieces.

## 4. Java Coding Example

This example checks whether orders are eligible for free shipping.

```java
import java.math.BigDecimal;

record Customer(String id, boolean active, String country) {
}

record Order(String id, Customer customer, BigDecimal total) {
}

interface Specification<T> {
    boolean isSatisfiedBy(T candidate);

    default Specification<T> and(Specification<T> other) {
        return candidate -> this.isSatisfiedBy(candidate) && other.isSatisfiedBy(candidate);
    }

    default Specification<T> or(Specification<T> other) {
        return candidate -> this.isSatisfiedBy(candidate) || other.isSatisfiedBy(candidate);
    }

    default Specification<T> not() {
        return candidate -> !this.isSatisfiedBy(candidate);
    }
}

final class ActiveCustomerSpec implements Specification<Order> {
    @Override
    public boolean isSatisfiedBy(Order order) {
        return order.customer().active();
    }
}

final class DomesticOrderSpec implements Specification<Order> {
    @Override
    public boolean isSatisfiedBy(Order order) {
        return order.customer().country().equals("US");
    }
}

final class MinimumTotalSpec implements Specification<Order> {
    private final BigDecimal minimum;

    MinimumTotalSpec(BigDecimal minimum) {
        this.minimum = minimum;
    }

    @Override
    public boolean isSatisfiedBy(Order order) {
        return order.total().compareTo(minimum) >= 0;
    }
}

public final class SpecificationDemo {
    public static void main(String[] args) {
        Specification<Order> freeShippingRule = new ActiveCustomerSpec()
            .and(new DomesticOrderSpec())
            .and(new MinimumTotalSpec(new BigDecimal("50.00")));

        Order order = new Order("o-1", new Customer("c-1", true, "US"), new BigDecimal("75.00"));
        System.out.println(freeShippingRule.isSatisfiedBy(order));
    }
}
```

### Java Block by Block Explanation

`Specification<T>` defines the rule contract.

`and`, `or`, and `not` allow composition.

`ActiveCustomerSpec`, `DomesticOrderSpec`, and `MinimumTotalSpec` are concrete business rules.

`freeShippingRule` combines smaller rules into one larger policy.

### Java Usage

Use Specification in Java when:

- Rules need names.
- Rules are reused.
- Rules are combined dynamically.
- Rules need isolated tests.
- Filtering and validation use the same concept.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class Customer:
    id: str
    active: bool
    country: str


@dataclass(frozen=True)
class Order:
    id: str
    customer: Customer
    total: Decimal


class Specification:
    def is_satisfied_by(self, candidate):
        raise NotImplementedError

    def __and__(self, other):
        return LambdaSpec(lambda candidate: self.is_satisfied_by(candidate) and other.is_satisfied_by(candidate))

    def __or__(self, other):
        return LambdaSpec(lambda candidate: self.is_satisfied_by(candidate) or other.is_satisfied_by(candidate))

    def not_(self):
        return LambdaSpec(lambda candidate: not self.is_satisfied_by(candidate))


class LambdaSpec(Specification):
    def __init__(self, rule):
        self.rule = rule

    def is_satisfied_by(self, candidate):
        return self.rule(candidate)


class ActiveCustomerSpec(Specification):
    def is_satisfied_by(self, order):
        return order.customer.active


class DomesticOrderSpec(Specification):
    def is_satisfied_by(self, order):
        return order.customer.country == "US"


rule = ActiveCustomerSpec() & DomesticOrderSpec()
print(rule.is_satisfied_by(Order("o-1", Customer("c-1", True, "US"), Decimal("75"))))
```

### Python Usage

Python can use:

- Specification classes.
- Plain predicates.
- Callable objects.
- Query builder filters.
- Dataclasses for parameterized rules.

Use classes when rules need names, composition, and unit tests.

## 6. Where It Comes Handy in Real Life

- Discount eligibility.
- Loan approval rules.
- Search/filter criteria.
- Validation rules.
- Shipping eligibility.
- Fraud checks.
- Access control rules.
- Repository query criteria.

## 7. Advantages Over Normal Code Without Pattern

### Without Specification

```java
if (customer.isActive() && order.total().compareTo(minimum) >= 0 && country.equals("US")) {
    ...
}
```

Problems:

- Rules are anonymous.
- Logic is duplicated.
- Combination is hard to test.
- Business language is hidden in conditionals.

### With Specification

```java
freeShippingRule.isSatisfiedBy(order);
```

Benefits:

- Rules have names.
- Rules are reusable.
- Rules are composable.
- Rules are testable.
- Business meaning is visible.

## 8. Where It Excels

- Rule-heavy domains.
- Dynamic filtering.
- Domain-driven design.
- Validation logic reused in multiple places.
- Query criteria with business names.
- Systems where business rules change often.

## 9. Where It Fails

- One-off simple conditions.
- Rules that are easier as database constraints.
- Over-composed rules that are hard to debug.
- Rules with heavy side effects.
- Cases where a workflow, not a boolean rule, is needed.

## 10. Prebuilt Frameworks and Packages

### Java

- `Predicate<T>`
- Spring Data JPA `Specification`
- Criteria API
- QueryDSL
- Drools/rule engines for larger rule systems

### Python

- Predicate functions.
- SQLAlchemy query filters.
- Django Q objects.
- Rule engine libraries.
- Pydantic validators for validation use cases.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reusable business rules. | Adds classes/abstraction. |
| Composable with boolean logic. | Can be overused for simple checks. |
| Easy to unit test. | Debugging composed rules can be harder. |
| Makes domain language explicit. | Query translation may be complex. |
| Reduces duplicate conditionals. | Side effects do not fit well. |

## 12. Real-World Identification Example

Scenario:

You are designing coupons for an e-commerce checkout.

Specification fit:

- `CustomerIsActive`
- `CartTotalAtLeast`
- `CouponNotExpired`
- `ShippingCountryAllowed`

What would go wrong without it:

- Coupon rules duplicate across checkout, preview, and admin validation.
- Product managers cannot map rule names to code easily.

## 13. MAANG Interview Triggers

Use Specification when you hear:

- "Reusable business rule."
- "Combine criteria dynamically."
- "Filter by business rules."
- "Avoid duplicated if conditions."
- "Rule object."
- "Predicate composition."
- "Domain validation."

### Interview-Ready Answer Format

1. Identify candidate object.
2. Define a specification interface.
3. Implement small named rules.
4. Compose rules with `and`, `or`, and `not`.
5. Use rules for validation/filter/query.
6. Mention overkill for simple one-off conditions.

## 14. Common Mistakes

### Mistake 1: Using Specification for Every If Statement

Use it when rules need reuse, names, testing, or composition.

### Mistake 2: Side Effects in Specifications

Specifications should usually answer yes/no, not mutate state.

### Mistake 3: Huge Specification Classes

Prefer small rules that compose.

### Mistake 4: Mixing Query Translation and Domain Logic Carelessly

In-memory rules and database predicates may need separate representations.

### Mistake 5: Hiding Rule Names Behind Lambdas

For important business rules, named classes or named functions communicate better.

## 15. Specification vs Similar Patterns

| Pattern | Difference |
|---|---|
| Specification | Encapsulates a reusable boolean business rule. |
| Strategy | Encapsulates an algorithm or behavior. |
| Predicate | Language/library-level function; Specification adds domain meaning and composition. |
| Chain of Responsibility | Passes request through handlers rather than combining boolean rules. |
| Repository | May accept/use specifications to query domain objects. |

## 16. Specification Design Checklist

| Question | Why it matters |
|---|---|
| What candidate is checked? | Defines generic type. |
| What rule is named? | Captures business language. |
| Is rule reusable? | Justifies pattern. |
| Can rules compose? | Enables flexible criteria. |
| Does rule have side effects? | Usually a smell. |
| Must rule translate to SQL? | Affects implementation design. |

## 17. Quick Revision Notes

- Specification is a reusable rule object.
- It answers whether a candidate satisfies a condition.
- Rules compose with `and`, `or`, `not`.
- Great for validation and filtering.
- Avoid side effects.
- Plain predicates are enough for tiny cases.

## 18. Mini Exercise

Design Specification for `LoanApplication`.

Rules:

- Applicant credit score is above threshold.
- Income is above minimum.
- Debt-to-income ratio is acceptable.
- Applicant is not blocked.

Compose:

```java
eligible = creditScoreOk.and(incomeOk).and(debtRatioOk).and(notBlocked);
```

## 19. Source Reference in This Repo

The repository's Specification implementation uses selector classes that can be combined with conjunction, disjunction, and negation.

Useful files:

- [github-repo/specification/README.md](../../github-repo/specification/README.md)
- [github-repo/specification/src/main/java/com/iluwatar/specification/selector/AbstractSelector.java](../../github-repo/specification/src/main/java/com/iluwatar/specification/selector/AbstractSelector.java)
- [github-repo/specification/src/main/java/com/iluwatar/specification/selector/ConjunctionSelector.java](../../github-repo/specification/src/main/java/com/iluwatar/specification/selector/ConjunctionSelector.java)
- [github-repo/specification/src/main/java/com/iluwatar/specification/selector/DisjunctionSelector.java](../../github-repo/specification/src/main/java/com/iluwatar/specification/selector/DisjunctionSelector.java)
- [github-repo/specification/src/main/java/com/iluwatar/specification/selector/NegationSelector.java](../../github-repo/specification/src/main/java/com/iluwatar/specification/selector/NegationSelector.java)
- [github-repo/specification/src/main/java/com/iluwatar/specification/app/App.java](../../github-repo/specification/src/main/java/com/iluwatar/specification/app/App.java)

