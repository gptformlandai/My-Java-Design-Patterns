# Currying Pattern

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [currying](../../github-repo/currying)

---

## How to Study This Page

Study Currying as "turn one multi-argument function into a chain of one-argument functions."

Remember the core shape:

```text
f(a, b, c) becomes f(a)(b)(c)
```

The practical value is partial application: provide some arguments now, get back a specialized function for later.

---

## 1. Technical Definition

Currying is a functional programming pattern that transforms a function with multiple parameters into a sequence of functions, each taking one parameter and returning the next function until the final result is produced.

### 30-Second Interview Answer

Currying converts a multi-argument operation into chained one-argument functions. This enables partial application, where we fix some arguments and reuse the returned specialized function later. I would use it for reusable factories, validators, configuration-heavy functions, and functional DSLs. The benefit is reuse and composability; the cost is readability in languages like Java where nested function types can become verbose.

---

## 2. Layman and Easy to Understand Definition

Imagine ordering a custom item one step at a time:

- Choose category.
- Then choose brand.
- Then choose model.
- Then choose date.

Each answer returns the next question. After the last answer, you get the final item.

That is currying.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Some functions need multiple repeated arguments:

```text
createBook(genre, author, title, date)
```

If `genre` and `author` are repeated often, passing all arguments every time is noisy.

### Currying Flow

1. Start with a function that conceptually needs many arguments.
2. Return a function after the first argument.
3. Return another function after the second argument.
4. Continue until all arguments are supplied.
5. Compute the final result.

### Core Participants

| Participant | Responsibility |
|---|---|
| Original operation | Multi-argument behavior |
| Curried function | One-argument-at-a-time version |
| Partial application | Some arguments already fixed |
| Specialized function | Returned function with context captured |
| Final argument | Completes the computation |
| Closure | Captures earlier arguments |

---

## 4. Java Coding Example

```java
import java.math.BigDecimal;
import java.util.function.Function;

public class CurryingDemo {
    public static void main(String[] args) {
        Function<BigDecimal, Function<BigDecimal, BigDecimal>> priceWithTax =
                taxRate -> amount -> amount.add(amount.multiply(taxRate));

        Function<BigDecimal, BigDecimal> usTaxPrice =
                priceWithTax.apply(new BigDecimal("0.08"));

        System.out.println(usTaxPrice.apply(new BigDecimal("100.00")));
        System.out.println(usTaxPrice.apply(new BigDecimal("250.00")));
    }
}
```

### Java Block by Block

`priceWithTax` first accepts a tax rate.

It returns a new function that accepts an amount.

`usTaxPrice` is a partially applied function with tax rate already fixed.

The same specialized function can be reused for many amounts.

---

## 5. Python Coding Example

```python
from decimal import Decimal


def price_with_tax(tax_rate):
    def apply_to(amount):
        return amount + (amount * tax_rate)
    return apply_to


us_tax_price = price_with_tax(Decimal("0.08"))

print(us_tax_price(Decimal("100.00")))
print(us_tax_price(Decimal("250.00")))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Specialized factories | Fix common constructor/config values once |
| Validation functions | Preload validation rules and apply to many inputs |
| Pricing/tax rules | Fix region or rate and reuse for many amounts |
| Functional DSLs | Express step-by-step construction cleanly |
| Test data builders | Preconfigure defaults for repeated test objects |
| Middleware configuration | Capture environment/config before request data arrives |

---

## 7. Advantages Over Normal Code Without Pattern

Without Currying:
- repeated arguments are passed again and again
- specialized functions are harder to create
- configuration may be mixed with runtime input
- factories become less composable

With Currying:
- common arguments can be fixed once
- later calls are simpler
- functions become reusable building blocks
- construction can read like a guided sequence

---

## 8. Where It Excels

It excels when:
- some arguments are known earlier than others
- configuration should be separated from runtime input
- specialized functions are reused many times
- function composition is common
- a step-by-step API improves readability
- closures are natural in the language

---

## 9. Where It Fails

It fails when:
- the language has clumsy function types
- the team is unfamiliar with functional style
- a simple method call is clearer
- argument order is not obvious
- debugging nested lambdas is painful
- business users need transparent rule definitions

Use ordinary methods, builders, or named factory classes when currying reduces readability.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Function, BiFunction, Vavr, Functional Java |
| Kotlin | Higher-order functions, lambdas, partial application helpers |
| Scala | Built-in curried function support |
| JavaScript | Ramda, Lodash/fp, custom closures |
| Python | functools.partial, closures, toolz |
| Haskell | Currying is part of the core language style |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Enables partial application | Verbose in Java |
| Separates configuration from runtime input | Can confuse developers unfamiliar with FP |
| Creates reusable specialized functions | Deep nesting hurts readability |
| Useful for DSL-like APIs | Stack traces can be less obvious |
| Works well with composition | Argument order must be carefully designed |

---

## 12. Real-World Identification Example

Question:

> A pricing engine repeatedly calculates tax-adjusted prices for the same region. Passing region and tax rate into every calculation is noisy. What pattern helps?

Strong answer:

Use Currying or partial application. Build a function that first accepts the region or tax rate and returns a specialized price calculator. Then each request only passes the amount. This separates stable configuration from per-request data and reduces repeated parameters.

---

## 13. MAANG Interview Triggers

Say Currying when you hear:
- partial application
- pre-fill arguments
- function returns function
- configure once, apply many times
- functional builder
- reusable specialized function
- curried function
- multi-argument function as chain

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Currying every method | Adds unnecessary complexity | Use only when partial application helps |
| Poor argument order | Hard to specialize useful functions | Put stable/config arguments first |
| Anonymous nested lambdas everywhere | Hard to read | Use named functions or interfaces |
| Confusing currying with composition | They solve different problems | Currying fixes arguments; composition chains outputs |
| Using when a builder is clearer | Java can become verbose | Prefer builder for object construction APIs |

---

## 15. Currying vs Similar Patterns

| Pattern | Difference |
|---|---|
| Function Composition | Composition connects function output to next input; Currying transforms argument structure |
| Builder | Builder names construction steps; Currying uses nested functions to collect arguments |
| Partial Application | Partial application is the result/use of currying, where some arguments are fixed |
| Factory | Factory creates objects; a curried function can act as a specialized factory |
| Strategy | Strategy selects behavior; Currying creates specialized behavior by capturing arguments |

---

## 16. Currying Design Checklist

- Which arguments are stable or configuration-like?
- Which arguments arrive later at runtime?
- Is partial application valuable?
- Is the argument order intuitive?
- Would a builder be clearer?
- Are function names readable?
- Can specialized functions be tested directly?
- Does the language make this readable enough?
- Are captured values immutable or safe to share?

---

## 17. Quick Revision Notes

- One-line summary: Convert `f(a, b, c)` into `f(a)(b)(c)`.
- Memory hook: "one argument at a time."
- Best for: partial application and reusable specialized functions.
- Avoid when: a normal method or builder is clearer.
- Interview line: "I would place stable configuration arguments first so the returned function can be reused with many runtime inputs."

---

## 18. Mini Exercise

Design a curried discount calculator:
- first accept customer tier
- then accept discount campaign
- then accept order amount
- return final price
- explain which partially applied function you would reuse

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/currying/README.md)
- [Book.java](../../github-repo/currying/src/main/java/com/iluwatar/currying/Book.java)
- [Genre.java](../../github-repo/currying/src/main/java/com/iluwatar/currying/Genre.java)
- [App.java](../../github-repo/currying/src/main/java/com/iluwatar/currying/App.java)

The repo implementation builds books by currying genre, author, title, and publication date into a step-by-step function chain. It also demonstrates partial application by creating genre-specific and author-specific book functions.
