# Template Method Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/template-method](../../github-repo/template-method)

## How to Study This Page

Use this page in three passes:

1. First pass: understand fixed algorithm skeleton with customizable steps.
2. Second pass: rewrite the Java example and identify template method, primitive operations, and hooks.
3. Third pass: compare Template Method with Strategy and Factory Method.

By the end, you should be able to say:

> Template Method defines an algorithm skeleton in a base class and lets subclasses override selected steps without changing the algorithm order.

## 1. Technical Definition

Template Method is a behavioral design pattern where a base class defines the skeleton of an algorithm and delegates some steps to subclasses.

Core idea:

- Base class owns algorithm order.
- Some steps are fixed.
- Some steps are abstract or overridable.
- Template method is often `final`.
- Subclasses customize details.

### 30-Second Interview Answer

I would use Template Method when several classes follow the same process but differ in specific steps. The base class defines the algorithm order, and subclasses implement the variable steps. It reduces duplication and controls extension points, but it relies on inheritance, so Strategy may be better when runtime swapping or composition is preferred.

## 2. Layman and Easy to Understand Definition

Template Method is like a recipe template.

Every hot drink follows this skeleton:

```text
boil water
brew ingredient
pour into cup
add extras
```

Tea and coffee change the brew and extras steps, but the overall order stays the same.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose reports have the same generation flow.

```java
loadData();
validate();
formatAsPdf();
save();
notifyUser();
```

Another report repeats most steps but changes formatting.

Problems:

- Duplicate workflow code.
- Subclasses may change algorithm order accidentally.
- Common steps are scattered.
- Extension points are unclear.

### 3.2 The Template Method Solution

Put the skeleton in the base class:

```java
public final void generate() {
    loadData();
    validate();
    format();
    save();
}
```

Subclasses implement only `format()`.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Abstract class | Defines template method and shared steps. |
| Template method | Fixed algorithm skeleton. |
| Primitive operation | Required step implemented by subclasses. |
| Hook | Optional overridable step with default behavior. |
| Concrete class | Subclass that fills in variable steps. |

### 3.4 Mental Model

Think of Template Method as a controlled recipe.

1. Base class defines the sequence.
2. Base class runs fixed common steps.
3. Subclasses provide variable details.
4. Optional hooks customize small parts.
5. Algorithm order stays protected.

## 4. Java Coding Example

This example generates reports.

```java
public abstract class ReportGenerator {
    public final void generate() {
        loadData();
        validateData();
        formatReport();
        if (shouldNotify()) {
            notifyUser();
        }
    }

    private void loadData() {
        System.out.println("Loading data");
    }

    private void validateData() {
        System.out.println("Validating data");
    }

    protected abstract void formatReport();

    protected boolean shouldNotify() {
        return true;
    }

    private void notifyUser() {
        System.out.println("Notifying user");
    }
}

public final class PdfReportGenerator extends ReportGenerator {
    @Override
    protected void formatReport() {
        System.out.println("Formatting PDF report");
    }
}

public final class CsvReportGenerator extends ReportGenerator {
    @Override
    protected void formatReport() {
        System.out.println("Formatting CSV report");
    }

    @Override
    protected boolean shouldNotify() {
        return false;
    }
}
```

### Java Block by Block Explanation

#### Template Method

```java
public final void generate() {
```

The template method fixes the algorithm order and prevents subclasses from changing it.

#### Fixed Steps

```java
private void loadData() {
```

Fixed steps are shared by every subclass.

#### Primitive Operation

```java
protected abstract void formatReport();
```

Subclasses must implement this variable step.

#### Hook

```java
protected boolean shouldNotify() {
    return true;
}
```

A hook gives subclasses optional control.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        ReportGenerator report = new PdfReportGenerator();
        report.generate();
    }
}
```

## 5. Python Coding Example

```python
from abc import ABC, abstractmethod


class ReportGenerator(ABC):
    def generate(self) -> None:
        self._load_data()
        self._validate_data()
        self._format_report()
        if self._should_notify():
            self._notify_user()

    def _load_data(self) -> None:
        print("Loading data")

    def _validate_data(self) -> None:
        print("Validating data")

    @abstractmethod
    def _format_report(self) -> None:
        ...

    def _should_notify(self) -> bool:
        return True

    def _notify_user(self) -> None:
        print("Notifying user")


class PdfReportGenerator(ReportGenerator):
    def _format_report(self) -> None:
        print("Formatting PDF report")
```

### Python Usage

```python
generator = PdfReportGenerator()
generator.generate()
```

## 6. Where It Comes Handy in Real Life

Examples:

- Test setup and teardown flows.
- Report generation.
- Data import pipelines.
- Framework lifecycle hooks.
- Build/deploy workflows.
- Abstract collection classes.
- Request handling skeletons.

## 7. Advantages Over Normal Code Without Pattern

### Without Template Method

```java
loadData();
validate();
formatPdf();
notifyUser();
```

Problems:

- Workflow order repeats.
- Subclasses can accidentally diverge.
- Common steps are duplicated.
- Extension points are unclear.

### With Template Method

```java
report.generate();
```

Benefits:

- Algorithm order is centralized.
- Common behavior is reused.
- Subclasses customize only intended steps.
- Hooks provide controlled extension.

## 8. Where It Excels

Template Method excels when:

- Algorithm order must stay fixed.
- Many subclasses share most steps.
- Variation points are known.
- You want controlled inheritance extension.
- Framework code calls subclass hooks.

## 9. Where It Fails

It is a poor fit when:

- You need runtime algorithm swapping.
- Inheritance is already too deep.
- Subclasses need too much freedom.
- The template becomes fragile.
- Composition would be clearer.

## 10. Prebuilt Libraries and Packages

### Java

Examples:

- Abstract collection classes.
- Framework lifecycle methods.
- JUnit-style setup/teardown concepts.
- Servlet lifecycle methods.

### Python

Examples:

- Base classes with fixed public methods and overridable protected methods.
- Framework hooks.
- Abstract base classes.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reuses algorithm skeleton. | Inheritance-based. |
| Controls step order. | Can create fragile base classes. |
| Makes extension points explicit. | Runtime swapping is harder. |
| Reduces duplication. | Subclasses may depend on base internals. |
| Hooks allow optional customization. | Too many hooks become confusing. |

## 12. Real-World Identification Example

Scenario:

You are designing a data import workflow.

Fixed steps:

- Read input
- Validate schema
- Transform rows
- Save results
- Emit metrics

Variable step:

- Transform rows differs by file type.

Should you use Template Method?

Yes, if the workflow order is fixed and subclass customization is limited.

## 13. MAANG Interview Triggers

Think Template Method when you hear:

- Algorithm skeleton.
- Fixed sequence of steps.
- Subclasses fill in details.
- Hooks.
- Framework lifecycle.
- Reuse common workflow.

### Interview-Ready Answer Format

1. Identify common algorithm steps.
2. Put fixed order in base template method.
3. Mark template method `final` where appropriate.
4. Make variable steps abstract/protected.
5. Add hooks for optional behavior.
6. Mention trade-off: inheritance and fragile base class risk.

## 14. Common Mistakes

### Mistake 1: Template Method Not Final

If subclasses can override the skeleton, the pattern loses control.

### Mistake 2: Too Many Hooks

Too many extension points make behavior hard to predict.

### Mistake 3: Using Inheritance When Strategy Fits Better

If algorithm should be swapped at runtime, Strategy may be cleaner.

### Mistake 4: Base Class Knows Subclass Details

The base class should define structure, not special-case subclasses.

### Mistake 5: Hidden Side Effects

Document what each overridable step may and may not do.

## 15. Template Method vs Similar Patterns

| Pattern | Difference |
|---|---|
| Strategy | Strategy uses composition and swappable algorithms. Template Method uses inheritance and fixed skeleton. |
| Factory Method | Factory Method is often one step inside a Template Method. |
| State | State changes behavior by current state. Template Method fixes a workflow skeleton. |
| Command | Command packages an operation. Template Method structures an algorithm. |
| Hook Method | A hook is an optional extension point inside Template Method. |

## 16. Template Method Checklist

| Concern | Why it matters |
|---|---|
| Fixed steps | Defines skeleton. |
| Variable steps | Become abstract/protected methods. |
| Hooks | Optional customization. |
| Final template | Protects order. |
| Inheritance depth | Avoids fragile hierarchy. |

## 17. Quick Revision Notes

- Base class defines algorithm skeleton.
- Subclasses fill in selected steps.
- Template method is often final.
- Hooks are optional override points.
- Best when workflow order is stable.
- Strategy is better for runtime swapping.

## 18. Mini Exercise

Design Template Method for `DataImportJob`.

Template:

- `read()`
- `validate()`
- `transform()`
- `save()`
- optional `afterSave()`

Concrete classes:

- `CsvImportJob`
- `JsonImportJob`

## 19. Source Reference in This Repo

The repository's Template Method implementation uses `StealingMethod` as the abstract workflow and concrete methods for variations.

Useful files:

- [github-repo/template-method/README.md](../../github-repo/template-method/README.md)
- [github-repo/template-method/src/main/java/com/iluwatar/templatemethod/StealingMethod.java](../../github-repo/template-method/src/main/java/com/iluwatar/templatemethod/StealingMethod.java)
- [github-repo/template-method/src/main/java/com/iluwatar/templatemethod/SubtleMethod.java](../../github-repo/template-method/src/main/java/com/iluwatar/templatemethod/SubtleMethod.java)
- [github-repo/template-method/src/main/java/com/iluwatar/templatemethod/HitAndRunMethod.java](../../github-repo/template-method/src/main/java/com/iluwatar/templatemethod/HitAndRunMethod.java)
- [github-repo/template-method/src/main/java/com/iluwatar/templatemethod/HalflingThief.java](../../github-repo/template-method/src/main/java/com/iluwatar/templatemethod/HalflingThief.java)
