# Combinator Pattern

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [combinator](../../github-repo/combinator)

---

## How to Study This Page

Study Combinator as "build complex behavior by combining small behavior blocks."

Remember the core idea:

```text
small function + small function + combining rule = richer function
```

This pattern is common in validation rules, parsers, filters, search predicates, authorization checks, and functional programming.

---

## 1. Technical Definition

Combinator is a functional pattern where simple functions or rules are combined using higher-order operations to create more complex functions without modifying the original pieces.

### 30-Second Interview Answer

Combinator builds complex behavior by composing small reusable functions with operators such as `and`, `or`, `not`, `then`, or domain-specific combiners. I would use it when rules need to be reusable and dynamically assembled, such as validation, filtering, search, parsing, or authorization. The benefit is composability; the cost is that deeply nested combinations can be harder to read and debug.

---

## 2. Layman and Easy to Understand Definition

Think of building search filters:

- contains "error"
- does not contain "debug"
- contains "payment" or "checkout"

Each rule is simple. Combining rules creates a powerful search.

That is the Combinator pattern.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Business rules often multiply:

```text
rule A
rule B
rule C
rule A and B
rule A or C
rule A and B but not C
```

Hardcoding every combination leads to duplicate classes or condition-heavy methods.

### Combinator Flow

1. Define a small function/rule interface.
2. Implement simple rules.
3. Add combiners such as `and`, `or`, and `not`.
4. Build complex rules by combining simple rules.
5. Execute the final combined rule like a normal function.

### Core Participants

| Participant | Responsibility |
|---|---|
| Primitive rule | Small independent behavior |
| Combiner | Joins rules into a new rule |
| Higher-order function | Accepts or returns functions |
| Composite rule | Resulting combined behavior |
| Input | Data being evaluated |
| Result | Boolean, list, parse result, or transformed value |

---

## 4. Java Coding Example

```java
import java.util.function.Predicate;

record User(String email, boolean active, int age) {}

interface Rule<T> extends Predicate<T> {
    default Rule<T> andRule(Rule<T> other) {
        return value -> this.test(value) && other.test(value);
    }

    default Rule<T> orRule(Rule<T> other) {
        return value -> this.test(value) || other.test(value);
    }

    default Rule<T> notRule() {
        return value -> !this.test(value);
    }
}

public class CombinatorDemo {
    public static void main(String[] args) {
        Rule<User> hasEmail = user -> user.email() != null && user.email().contains("@");
        Rule<User> isActive = User::active;
        Rule<User> isAdult = user -> user.age() >= 18;

        Rule<User> canLogin = hasEmail.andRule(isActive).andRule(isAdult);

        System.out.println(canLogin.test(new User("a@example.com", true, 22)));
    }
}
```

### Java Block by Block

Each `Rule` is a simple predicate.

`andRule`, `orRule`, and `notRule` return new rules.

`canLogin` is built from smaller reusable rules.

No original rule has to know who combines it later.

---

## 5. Python Coding Example

```python
def and_rule(left, right):
    return lambda value: left(value) and right(value)


def or_rule(left, right):
    return lambda value: left(value) or right(value)


def not_rule(rule):
    return lambda value: not rule(value)


has_email = lambda user: "@" in user.get("email", "")
is_active = lambda user: user.get("active") is True
is_adult = lambda user: user.get("age", 0) >= 18

can_login = and_rule(and_rule(has_email, is_active), is_adult)

print(can_login({"email": "a@example.com", "active": True, "age": 22}))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Validation rules | Combine reusable checks into larger validation policies |
| Search filters | Combine contains, exclude, and alternative rules |
| Authorization | Build permission checks from smaller predicates |
| Parsers | Parser combinators build grammar from small parsers |
| Feature targeting | Combine audience rules for experiments |
| Pricing rules | Compose eligibility and discount conditions |

---

## 7. Advantages Over Normal Code Without Pattern

Without Combinator:
- rule combinations become duplicated
- large condition blocks grow quickly
- simple rules are not reusable
- testing every combination is harder

With Combinator:
- simple rules stay small
- complex rules are assembled declaratively
- combinations can be tested independently
- new policies can reuse old pieces

---

## 8. Where It Excels

It excels when:
- rules are small and reusable
- combinations change often
- behavior can be represented as functions
- teams want declarative rule assembly
- testing primitive rules separately is valuable
- domain language benefits from `and`, `or`, `not`, or custom combiners

---

## 9. Where It Fails

It fails when:
- combinations become deeply nested
- rule execution needs complex shared state
- debugging anonymous functions is painful
- rule ordering has hidden side effects
- non-functional developers find the style unclear
- performance requires a simpler direct implementation

Use named rules, explicit classes, or a rules engine when rule graphs become large or business-owned.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Predicate, Function, Optional, Vavr |
| Validation | Hibernate Validator, FluentValidator-style APIs |
| Parsing | jparsec, Parboiled, parser combinator libraries |
| Python | functools, toolz, returns |
| JavaScript | Ramda, functional predicate helpers |
| Scala/Kotlin | Cats, Arrow, parser combinator libraries |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Highly reusable rules | Nested combinations can be hard to read |
| Declarative behavior assembly | Debugging lambdas can be harder |
| Avoids duplicate condition code | Requires functional-programming comfort |
| Easy to unit test small rules | Poor naming creates confusion |
| Supports dynamic rule building | Side effects make behavior unpredictable |

---

## 12. Real-World Identification Example

Question:

> A signup flow has many reusable checks: valid email, adult age, active region, not blocked, and optional promo eligibility. Product keeps asking for new combinations. What pattern helps?

Strong answer:

Use Combinator. Model each check as a small rule and provide combiners such as `and`, `or`, and `not`. Then build policies like `validEmail.and(adult).and(activeRegion).and(notBlocked)`. This avoids rewriting large condition blocks and keeps each rule testable.

---

## 13. MAANG Interview Triggers

Say Combinator when you hear:
- combine small rules
- reusable predicates
- validation policy
- parser combinators
- search query filters
- compose conditions dynamically
- and/or/not rules
- declarative rule assembly

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Anonymous everything | Logic becomes unreadable | Name important rules |
| Side effects inside rules | Combination order changes behavior | Keep rules pure when possible |
| Deep nesting | Hard to debug and explain | Break into named intermediate rules |
| No short-circuit awareness | Expensive rules may run unnecessarily | Order cheap checks first |
| Using for business-owned rule catalogs | Code releases slow rule changes | Consider a rules engine or config model |

---

## 15. Combinator vs Similar Patterns

| Pattern | Difference |
|---|---|
| Function Composition | Composition chains outputs to inputs; Combinator combines same-shaped behaviors/rules |
| Specification | Specification is a domain-rule pattern; Combinator is a functional composition technique often used inside it |
| Composite | Composite models tree structures; Combinator builds behavior from functions |
| Strategy | Strategy selects one algorithm; Combinator builds a new algorithm from smaller ones |
| Collection Pipeline | Collection Pipeline transforms data collections; Combinator combines operations or predicates |

---

## 16. Combinator Design Checklist

- What is the smallest useful rule?
- What input type does every rule accept?
- What output type does every rule return?
- Which combiners are needed?
- Are rules pure and side-effect-free?
- Are combined rules named clearly?
- Is short-circuiting important?
- How will rule failures be explained to users?
- When does this need a rules engine instead?

---

## 17. Quick Revision Notes

- One-line summary: Combine small functions or rules into richer behavior.
- Memory hook: "rules as building blocks."
- Best for: validation, search, parsing, authorization, and policy composition.
- Avoid when: rule graphs become unreadable or heavily stateful.
- Interview line: "I would define small named predicates and combine them through domain-friendly operators."

---

## 18. Mini Exercise

Build combinators for loan approval:
- define rules for credit score, income, existing debt, and region
- combine them into one approval rule
- add one optional fast-track rule
- decide which rules should short-circuit
- explain how you would return failure reasons

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/combinator/README.md)
- [Finder.java](../../github-repo/combinator/src/main/java/com/iluwatar/combinator/Finder.java)
- [Finders.java](../../github-repo/combinator/src/main/java/com/iluwatar/combinator/Finders.java)
- [CombinatorApp.java](../../github-repo/combinator/src/main/java/com/iluwatar/combinator/CombinatorApp.java)

The repo implementation defines a `Finder` interface and combines simple text finders with `and`, `or`, and `not` operations to create advanced search behavior.
