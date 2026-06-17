# Pipeline Pattern

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [pipeline](../../github-repo/pipeline)

---

## How to Study This Page

Study Pipeline as "pass data through ordered stages where each stage does one job."

Remember the core flow:

```text
Input -> Stage 1 -> Stage 2 -> Stage 3 -> Output
```

The main design instinct is to keep each stage small, explicit, and easy to test.

---

## 1. Technical Definition

Pipeline is a processing pattern where data flows through a sequence of independent stages, and the output of one stage becomes the input of the next stage.

### 30-Second Interview Answer

Pipeline organizes a workflow as ordered transformations. Each stage performs one focused operation and passes its output to the next stage. I would use it for ETL, request filters, compiler steps, image processing, validation chains, and text processing. The benefit is clarity and composability; the trade-off is that stage boundaries, error handling, and performance overhead need careful design.

---

## 2. Layman and Easy to Understand Definition

Think of an assembly line.

- One station cleans the item.
- The next station validates it.
- The next station packages it.
- The final station sends it out.

Each station has one job, and the item moves forward step by step.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Complex processing often becomes one long method:

```text
parse -> validate -> enrich -> transform -> save
```

When all logic sits together, it is hard to test, reorder, reuse, or replace individual steps.

### Pipeline Flow

1. Receive input.
2. Pass it into the first stage.
3. First stage returns transformed output.
4. Pass output into the next stage.
5. Continue until the final stage.
6. Return or persist the final output.

### Core Participants

| Participant | Responsibility |
|---|---|
| Input | Data entering the pipeline |
| Stage/handler | One transformation or operation |
| Pipeline | Connects stages in order |
| Intermediate output | Output passed between stages |
| Error policy | Decides stop, skip, retry, or compensate |
| Final output | Result after all stages |

---

## 4. Java Coding Example

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

class Pipeline<I, O> {
    private final Function<I, O> chain;

    private Pipeline(Function<I, O> chain) {
        this.chain = chain;
    }

    static <T> Pipeline<T, T> start() {
        return new Pipeline<>(Function.identity());
    }

    <N> Pipeline<I, N> add(Function<O, N> next) {
        return new Pipeline<>(chain.andThen(next));
    }

    O execute(I input) {
        return chain.apply(input);
    }
}

public class PipelineDemo {
    public static void main(String[] args) {
        Pipeline<String, List<String>> pipeline = Pipeline.<String>start()
                .add(String::trim)
                .add(String::toLowerCase)
                .add(text -> text.replace(",", ""))
                .add(text -> new ArrayList<>(List.of(text.split("\\s+"))));

        System.out.println(pipeline.execute("  Pay, Ship, Notify  "));
    }
}
```

### Java Block by Block

`Pipeline.start` begins with an identity stage.

`add` composes the existing chain with the next function.

`execute` runs the input through every stage in order.

Each transformation can be tested independently.

---

## 5. Python Coding Example

```python
def trim(text):
    return text.strip()


def normalize(text):
    return text.lower().replace(",", "")


def tokenize(text):
    return text.split()


def run_pipeline(value, stages):
    for stage in stages:
        value = stage(value)
    return value


result = run_pipeline("  Pay, Ship, Notify  ", [trim, normalize, tokenize])
print(result)
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| ETL workflows | Extract, transform, validate, and load are natural stages |
| Request middleware | Each middleware step modifies or checks a request |
| Compiler design | Lexing, parsing, analysis, and code generation are ordered stages |
| Text processing | Clean, normalize, tokenize, and classify text |
| Image processing | Resize, filter, encode, and store images |
| Data validation | Apply ordered checks and transformations |

---

## 7. Advantages Over Normal Code Without Pattern

Without Pipeline:
- one method contains too many processing details
- reordering steps is risky
- individual steps are hard to test
- shared mutable state can spread across the workflow

With Pipeline:
- each stage has one responsibility
- stages are reusable
- flow is readable
- tests can target one stage or the full chain

---

## 8. Where It Excels

It excels when:
- processing has a clear sequence
- stages are reusable
- each stage can be tested independently
- stages can be added, removed, or reordered
- data shape changes are intentional
- observability per stage is useful

---

## 9. Where It Fails

It fails when:
- stages are tightly coupled through hidden state
- every stage needs to call back into every other stage
- control flow branches heavily
- stage ordering is unclear
- error handling differs wildly per item
- pipeline overhead is larger than the work

Use a state machine, workflow engine, or explicit orchestration when transitions are complex and conditional.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Java Stream API, Spring Integration, Apache Camel |
| Data | Apache Beam, Spark pipelines, Flink jobs |
| Web | Servlet filters, Spring WebFlux filters, Express middleware |
| Python | itertools, generator pipelines, Luigi, Airflow |
| Build systems | Gradle tasks, CI/CD pipeline stages |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Clear ordered flow | Stage boundaries can become too fine-grained |
| Easy to test stages | Debugging across many stages needs tracing |
| Encourages single responsibility | Type changes between stages can be awkward |
| Supports reuse and composition | Branching workflows can become messy |
| Works well with streaming data | Shared mutable state breaks the model |

---

## 12. Real-World Identification Example

Question:

> A payment import job parses CSV rows, validates fields, enriches customer data, maps to domain objects, and writes records. The current code is one 600-line method. What pattern helps?

Strong answer:

Use Pipeline. I would split the job into parse, validate, enrich, map, and persist stages. Each stage receives one data shape and returns the next. This makes the workflow easier to test, reorder, observe, and replace. I would define error handling per stage, especially for validation failures and downstream write failures.

---

## 13. MAANG Interview Triggers

Say Pipeline when you hear:
- ordered processing stages
- ETL flow
- middleware chain
- transform data step by step
- compiler phases
- reusable processing steps
- validate then enrich then persist
- stage-level observability

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Hidden shared mutable state | Stages become coupled | Pass explicit data between stages |
| Too many tiny stages | Flow becomes noisy | Group steps by meaningful responsibility |
| No error policy | Failures are inconsistent | Define stop, skip, retry, or dead-letter behavior |
| Ambiguous stage names | Readers cannot understand intent | Name stages by business action |
| Ignoring data shape changes | Later stages receive unexpected input | Use clear input/output types |

---

## 15. Pipeline vs Similar Patterns

| Pattern | Difference |
|---|---|
| Chain of Responsibility | Chain decides who handles; Pipeline usually runs all stages in order |
| Function Composition | Function composition is the functional mechanism; Pipeline is the architectural workflow |
| Collection Pipeline | Collection pipeline applies stream operations to collections in memory |
| Map Reduce | Map Reduce adds distributed grouping and reduction |
| Template Method | Template Method fixes algorithm skeleton in inheritance; Pipeline composes stages |

---

## 16. Pipeline Design Checklist

- What is the input type?
- What are the ordered stages?
- What does each stage return?
- Can each stage be tested independently?
- Does any stage need shared state?
- How are errors handled?
- Can stages be retried safely?
- Where will metrics and logs be captured?
- Is the pipeline sync, async, batch, or streaming?

---

## 17. Quick Revision Notes

- One-line summary: Process data through ordered, focused stages.
- Memory hook: "assembly line for data."
- Best for: ETL, middleware, validation, and transformation workflows.
- Avoid when: control flow is highly conditional or stateful.
- Interview line: "I would make stage inputs and outputs explicit and define failure handling at stage boundaries."

---

## 18. Mini Exercise

Design a pipeline for user signup:
- define stages for normalize, validate, persist, notify
- specify each stage input and output
- decide what happens when validation fails
- decide whether notification failure should fail signup
- add one metric per stage

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/pipeline/README.md)
- [Handler.java](../../github-repo/pipeline/src/main/java/com/iluwatar/pipeline/Handler.java)
- [Pipeline.java](../../github-repo/pipeline/src/main/java/com/iluwatar/pipeline/Pipeline.java)
- [RemoveAlphabetsHandler.java](../../github-repo/pipeline/src/main/java/com/iluwatar/pipeline/RemoveAlphabetsHandler.java)
- [RemoveDigitsHandler.java](../../github-repo/pipeline/src/main/java/com/iluwatar/pipeline/RemoveDigitsHandler.java)
- [ConvertToCharArrayHandler.java](../../github-repo/pipeline/src/main/java/com/iluwatar/pipeline/ConvertToCharArrayHandler.java)
- [App.java](../../github-repo/pipeline/src/main/java/com/iluwatar/pipeline/App.java)

The repo implementation composes handlers so a string passes through alphabet removal, digit removal, and conversion to a character array.
