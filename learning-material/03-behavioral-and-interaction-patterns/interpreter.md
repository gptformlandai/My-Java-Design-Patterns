# Interpreter Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Low  
Repository module: [github-repo/interpreter](../../github-repo/interpreter)

## How to Study This Page

Use this page in three passes:

1. First pass: understand grammar rules represented as objects.
2. Second pass: rewrite the Java example and identify terminal expressions, non-terminal expressions, and evaluation.
3. Third pass: compare Interpreter with parsers, expression trees, Strategy, and Visitor.

By the end, you should be able to say:

> Interpreter represents a small language grammar as objects and evaluates sentences by walking that object structure.

## 1. Technical Definition

Interpreter is a behavioral design pattern that defines a representation for a language grammar and provides an interpreter to evaluate sentences in that language.

Core idea:

- Each grammar rule becomes a class.
- Terminal expressions represent atomic values.
- Non-terminal expressions compose other expressions.
- The client builds or parses an expression tree.
- Interpretation evaluates that tree.

### 30-Second Interview Answer

I would use Interpreter for a small, stable domain-specific language like simple rules, arithmetic expressions, or query filters. Each grammar element becomes an expression object with an `interpret()` method. Complex expressions compose simpler expressions. The main trade-off is scalability: for complex grammars, a real parser generator or existing language engine is usually better.

## 2. Layman and Easy to Understand Definition

Interpreter is like a tiny calculator that understands a small language.

Instead of hardcoding every expression, the program turns the expression into objects. Each object knows how to evaluate its own part.

In code:

- Number is a terminal expression.
- Plus or multiply is a non-terminal expression.
- The full expression is a tree.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose users enter rules:

```text
amount > 100 AND country == "US"
```

Hardcoding every possible rule creates messy conditionals.

Problems:

- Rules change often.
- Expressions can be nested.
- Users or config files define logic.
- You need reusable grammar pieces.

### 3.2 The Interpreter Solution

Represent rules as expression objects:

```java
Expression rule = new AndExpression(
    new AmountGreaterThan(100),
    new CountryEquals("US")
);

boolean allowed = rule.interpret(order);
```

Each expression evaluates itself.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Abstract expression | Common interface with `interpret`. |
| Terminal expression | Atomic grammar element. |
| Non-terminal expression | Composes other expressions. |
| Context | External data needed for interpretation. |
| Client/parser | Builds the expression tree. |

### 3.4 Mental Model

Interpreter is object-oriented grammar.

1. Define grammar pieces.
2. Represent each piece as a class.
3. Compose pieces into a tree.
4. Evaluate tree using `interpret`.
5. Return result.

## 4. Java Coding Example

This example interprets simple order rules.

```java
record Order(int amount, String country) {
}

interface RuleExpression {
    boolean interpret(Order order);
}

final class AmountGreaterThan implements RuleExpression {
    private final int threshold;

    AmountGreaterThan(int threshold) {
        this.threshold = threshold;
    }

    @Override
    public boolean interpret(Order order) {
        return order.amount() > threshold;
    }
}

final class CountryEquals implements RuleExpression {
    private final String expectedCountry;

    CountryEquals(String expectedCountry) {
        this.expectedCountry = expectedCountry;
    }

    @Override
    public boolean interpret(Order order) {
        return order.country().equals(expectedCountry);
    }
}

final class AndExpression implements RuleExpression {
    private final RuleExpression left;
    private final RuleExpression right;

    AndExpression(RuleExpression left, RuleExpression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public boolean interpret(Order order) {
        return left.interpret(order) && right.interpret(order);
    }
}

public final class InterpreterDemo {
    public static void main(String[] args) {
        RuleExpression rule = new AndExpression(
            new AmountGreaterThan(100),
            new CountryEquals("US")
        );

        System.out.println(rule.interpret(new Order(150, "US")));
        System.out.println(rule.interpret(new Order(80, "US")));
    }
}
```

### Java Block by Block Explanation

`RuleExpression` is the expression interface.

`AmountGreaterThan` and `CountryEquals` are terminal expressions.

`AndExpression` is a non-terminal expression because it composes other expressions.

`Order` is the context passed during interpretation.

### Java Usage

Use Interpreter when:

- Grammar is small.
- Rules are composed dynamically.
- You need an expression tree.
- Domain experts configure simple rules.
- You want each rule object to be testable.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Order:
    amount: int
    country: str


class AmountGreaterThan:
    def __init__(self, threshold):
        self.threshold = threshold

    def interpret(self, order):
        return order.amount > self.threshold


class CountryEquals:
    def __init__(self, expected_country):
        self.expected_country = expected_country

    def interpret(self, order):
        return order.country == self.expected_country


class AndExpression:
    def __init__(self, left, right):
        self.left = left
        self.right = right

    def interpret(self, order):
        return self.left.interpret(order) and self.right.interpret(order)


rule = AndExpression(AmountGreaterThan(100), CountryEquals("US"))
print(rule.interpret(Order(150, "US")))
print(rule.interpret(Order(80, "US")))
```

### Python Usage

Python often uses simpler tools first:

- Functions.
- Dictionaries.
- AST modules.
- Parser libraries.
- Rule engines.

Use class-based Interpreter when the expression tree itself is important.

## 6. Where It Comes Handy in Real Life

- Arithmetic expression evaluation.
- Rule engines for small domains.
- Query filters.
- Validation languages.
- Configuration expressions.
- Template condition evaluation.
- Feature flag rules.
- Search query parsing.

## 7. Advantages Over Normal Code Without Pattern

### Without Interpreter

```java
if (amount > 100 && country.equals("US")) {
    ...
}
```

Problems:

- Rules are hardcoded.
- Composition is difficult.
- Runtime rule changes are hard.
- Complex conditionals become unreadable.

### With Interpreter

```java
RuleExpression rule = new AndExpression(a, b);
rule.interpret(order);
```

Benefits:

- Rules become objects.
- Expressions compose naturally.
- Each expression is testable.
- Grammar is explicit.

## 8. Where It Excels

- Small grammars.
- Stable DSLs.
- Rule composition.
- Expression trees.
- Education/interview examples.
- Configurable business rules with limited scope.

## 9. Where It Fails

- Complex programming languages.
- Large grammars.
- Performance-sensitive parsing.
- Ambiguous syntax.
- Cases needing strong tooling, error recovery, and optimization.
- Rules better handled by SQL/search engines.

## 10. Prebuilt Libraries and Packages

### Java

- `java.util.regex.Pattern`
- ANTLR
- JavaCC
- Spring Expression Language
- MVEL
- Drools for rule engines

### Python

- `ast`
- `re`
- `lark`
- `pyparsing`
- `textX`
- `rule-engine`

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Makes grammar explicit. | Class count grows with grammar size. |
| Expressions compose cleanly. | Not ideal for complex languages. |
| Each rule is testable. | Can be slower than compiled/evaluated forms. |
| Useful for small DSLs. | Parsing and error messages need extra work. |

## 12. Real-World Identification Example

Scenario:

You are building feature flag targeting rules:

```text
country == "US" AND plan == "PREMIUM"
```

Without Interpreter:

- Rules are hardcoded in services.
- Product changes require deployments.

With Interpreter:

- Rules are parsed into expressions.
- Expressions evaluate against user context.
- New combinations use existing grammar objects.

## 13. MAANG Interview Triggers

Use Interpreter when you hear:

- "Small domain-specific language."
- "Parse and evaluate simple rules."
- "Expression tree."
- "Grammar represented by classes."
- "Configurable rules."
- "Arithmetic expression evaluator."

### Interview-Ready Answer Format

1. Define the grammar scope.
2. Create expression interface.
3. Implement terminal expressions.
4. Implement composite expressions.
5. Build expression tree from config/parser.
6. Mention parser generators for complex grammars.

## 14. Common Mistakes

### Mistake 1: Using It for Large Languages

Use parser generators and language tooling for complex syntax.

### Mistake 2: Mixing Parsing and Interpretation Too Much

Parsing builds the tree; interpretation evaluates it.

### Mistake 3: No Context Object

Rules often need external data. Pass it explicitly.

### Mistake 4: Ignoring Error Reporting

Invalid input needs clear messages and safe handling.

### Mistake 5: Rebuilding Trees Repeatedly

Cache parsed expressions when rules are reused.

## 15. Interpreter vs Similar Patterns

| Pattern | Difference |
|---|---|
| Interpreter | Represents grammar and evaluates expressions. |
| Composite | Expression trees often use Composite structure. |
| Visitor | Can add operations over expression trees. |
| Strategy | Chooses one algorithm, not a grammar tree. |
| Specification | Encapsulates business predicates; may use Interpreter internally. |

## 16. Interpreter Design Checklist

| Question | Why it matters |
|---|---|
| Is grammar small? | Prevents class explosion. |
| What is the context object? | Supplies runtime data. |
| Are expressions immutable? | Enables caching and reuse. |
| Who parses input? | Separates syntax from evaluation. |
| What happens on invalid input? | Avoids unsafe evaluation. |

## 17. Quick Revision Notes

- Interpreter represents grammar as objects.
- Terminal expressions are leaves.
- Non-terminal expressions compose children.
- `interpret(context)` evaluates the tree.
- Great for small DSLs.
- Avoid for complex languages.

## 18. Mini Exercise

Design Interpreter for a coupon rule language.

Rules:

- `cartTotal > 50`
- `customerTier == GOLD`
- `AND`
- `OR`

Expected usage:

```java
RuleExpression rule = new AndExpression(totalRule, tierRule);
boolean eligible = rule.interpret(cartContext);
```

## 19. Source Reference in This Repo

The repository's Interpreter implementation evaluates basic math expressions with expression classes.

Useful files:

- [github-repo/interpreter/README.md](../../github-repo/interpreter/README.md)
- [github-repo/interpreter/src/main/java/com/iluwatar/interpreter/Expression.java](../../github-repo/interpreter/src/main/java/com/iluwatar/interpreter/Expression.java)
- [github-repo/interpreter/src/main/java/com/iluwatar/interpreter/NumberExpression.java](../../github-repo/interpreter/src/main/java/com/iluwatar/interpreter/NumberExpression.java)
- [github-repo/interpreter/src/main/java/com/iluwatar/interpreter/PlusExpression.java](../../github-repo/interpreter/src/main/java/com/iluwatar/interpreter/PlusExpression.java)
- [github-repo/interpreter/src/main/java/com/iluwatar/interpreter/App.java](../../github-repo/interpreter/src/main/java/com/iluwatar/interpreter/App.java)

