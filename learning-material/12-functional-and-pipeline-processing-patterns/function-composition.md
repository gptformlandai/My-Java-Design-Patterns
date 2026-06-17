# Function Composition Pattern

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [function-composition](../../github-repo/function-composition)

---

## How to Study This Page

Study Function Composition as "connect small functions so the output of one becomes the input of the next."

Remember the core flow:

```text
f(x) -> g(result) -> h(result)
```

In Java, this is commonly expressed with `andThen` and `compose`.

---

## 1. Technical Definition

Function Composition is a functional pattern where two or more functions are combined into a new function by feeding the output of one function into the input of the next.

### 30-Second Interview Answer

Function Composition lets us build a larger operation by chaining small functions. Each function focuses on one transformation, and the composed function executes them in sequence. I would use it for normalization, validation steps, data conversion, and reusable transformations. The benefit is clean composition and testable small functions; the trade-off is readability when chains become too long or types become hard to follow.

---

## 2. Layman and Easy to Understand Definition

Think of editing text:

- trim spaces
- convert to lowercase
- remove punctuation

Each step is a tiny function. If you connect them, you get one reusable text-cleaning function.

That is Function Composition.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Transformation code can become repetitive:

```text
value1 = trim(input)
value2 = lower(value1)
value3 = removeSymbols(value2)
return value3
```

When the same sequence is needed in many places, a composed function is cleaner.

### Function Composition Flow

1. Define small functions.
2. Ensure output type of one function matches input type of the next.
3. Compose them in the required order.
4. Store the composed function.
5. Apply it like a normal function.

### Core Participants

| Participant | Responsibility |
|---|---|
| First function | Transforms original input |
| Next function | Consumes previous output |
| Composition operator | Connects functions |
| Composed function | New reusable function |
| Input/output types | Must align between stages |
| Execution order | Determines final behavior |

---

## 4. Java Coding Example

```java
import java.util.function.Function;

public class FunctionCompositionDemo {
    public static void main(String[] args) {
        Function<String, String> trim = String::trim;
        Function<String, String> lower = String::toLowerCase;
        Function<String, String> removeSpaces = text -> text.replace(" ", "-");

        Function<String, String> slugify =
                trim.andThen(lower).andThen(removeSpaces);

        System.out.println(slugify.apply("  New Product Launch  "));
    }
}
```

### Java Block by Block

`trim`, `lower`, and `removeSpaces` are small transformations.

`andThen` composes them in left-to-right execution order.

`slugify` is now one reusable function.

The composed function can be passed around like any other `Function`.

---

## 5. Python Coding Example

```python
def compose(*functions):
    def composed(value):
        for function in functions:
            value = function(value)
        return value
    return composed


slugify = compose(
    str.strip,
    str.lower,
    lambda text: text.replace(" ", "-"),
)

print(slugify("  New Product Launch  "))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Input normalization | Trim, lower, sanitize, validate |
| Data mapping | Convert one representation into another |
| API response shaping | Apply reusable transformations before returning |
| Domain calculations | Chain small business calculations |
| Middleware helpers | Compose request/response transformations |
| Test utilities | Build reusable fixture transformation functions |

---

## 7. Advantages Over Normal Code Without Pattern

Without Function Composition:
- repeated transformation sequences appear everywhere
- intermediate variables add noise
- small functions are less reusable
- order of operations may become inconsistent

With Function Composition:
- small functions remain testable
- composed function documents the flow
- behavior can be reused as one unit
- transformation order is explicit

---

## 8. Where It Excels

It excels when:
- transformations are pure or mostly pure
- each function has one clear responsibility
- output and input types line up cleanly
- the sequence is reused
- readability improves with names
- functions can be tested independently

---

## 9. Where It Fails

It fails when:
- functions have hidden side effects
- type conversions become confusing
- chains become too long
- branching logic dominates
- debugging intermediate values is difficult
- one composed function tries to do an entire workflow

Use named intermediate steps, a pipeline, or explicit orchestration when the sequence needs branching, state, or rich error handling.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Function.andThen, Function.compose, UnaryOperator |
| Java functional libs | Vavr, Functional Java |
| Python | functools, toolz, custom compose helpers |
| JavaScript | Ramda compose/pipe, Lodash/fp flow |
| Kotlin | Function references, extension helpers |
| Scala | `andThen`, `compose`, Cats |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Builds reusable transformations | Long chains can be hard to read |
| Encourages small focused functions | Type mismatch errors can be noisy |
| Reduces repeated glue code | Side effects make order risky |
| Easy to test pieces | Debugging intermediate values needs tooling |
| Works well with pipelines | Branching logic does not fit cleanly |

---

## 12. Real-World Identification Example

Question:

> A service normalizes product titles in many places by trimming, lowercasing, removing repeated spaces, and replacing spaces with dashes. The logic is repeated across controllers. What pattern helps?

Strong answer:

Use Function Composition. Define each text transformation as a small function and compose them into one `slugify` function. Controllers can call that single function, while each step remains independently testable. If the flow grows into many stages with error handling, I would move toward a pipeline.

---

## 13. MAANG Interview Triggers

Say Function Composition when you hear:
- chain small functions
- output becomes next input
- reusable transformation
- compose and andThen
- avoid repeated intermediate variables
- pure functions
- functional data conversion
- build one operation from small operations

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Mixing side effects into functions | Order becomes dangerous | Keep transformations pure |
| Overlong chains | Hard to understand and debug | Name meaningful sub-compositions |
| Ignoring type alignment | Composition will not compile or will be confusing | Match output and input types carefully |
| Confusing `compose` and `andThen` | Order may reverse | Use names or tests to confirm execution order |
| Hiding business workflows in one function | Too much behavior disappears | Use pipeline/orchestration for larger flows |

---

## 15. Function Composition vs Similar Patterns

| Pattern | Difference |
|---|---|
| Pipeline | Pipeline is a broader workflow structure; Function Composition is the function-level chaining mechanism |
| Currying | Currying changes how arguments are supplied; Composition chains function outputs to inputs |
| Collection Pipeline | Collection Pipeline applies operations to many elements; Function Composition builds one function |
| Decorator | Decorator wraps object behavior; Function Composition combines functions |
| Chain of Responsibility | Chain may stop when one handler accepts; Composition usually runs every function in order |

---

## 16. Function Composition Design Checklist

- What are the smallest useful functions?
- Do output/input types align?
- Is the order obvious?
- Should `andThen` or `compose` be used?
- Are functions pure enough?
- Is the composed function named by business intent?
- Is the chain short enough to read?
- Should intermediate functions be unit tested?
- Is this becoming a pipeline instead?

---

## 17. Quick Revision Notes

- One-line summary: Build a new function by feeding one function's output into the next.
- Memory hook: "function Lego."
- Best for: reusable transformations and normalization.
- Avoid when: branching, side effects, or complex error handling dominate.
- Interview line: "I would keep transformations small, pure, and named, then compose them in the order the business expects."

---

## 18. Mini Exercise

Create a composed function for usernames:
- trim input
- lowercase it
- replace spaces with underscores
- remove invalid characters
- explain how you would test the order

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/function-composition/README.md)
- [FunctionComposer.java](../../github-repo/function-composition/src/main/java/com/iluwatar/function/composition/FunctionComposer.java)
- [App.java](../../github-repo/function-composition/src/main/java/com/iluwatar/function/composition/App.java)

The repo implementation composes two integer functions with `Function.andThen`; one doubles the value and the next squares the doubled result.
