# Factory Method Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/factory-method](../../github-repo/factory-method)

## How to Study This Page

Use this page in three passes:

1. First pass: understand how Factory Method moves object creation into an overridable method.
2. Second pass: rewrite the Java example and identify the creator, concrete creators, product, and concrete products.
3. Third pass: compare Factory Method with Simple Factory and Abstract Factory until the difference feels automatic.

By the end, you should be able to say:

> Factory Method lets a base class or interface define object creation while subclasses decide which concrete product to instantiate.

## 1. Technical Definition

Factory Method is a creational design pattern that defines a method for creating an object but lets subclasses choose the concrete class to instantiate. The client works with product and creator abstractions rather than direct concrete constructors.

Core idea:

- Put product creation behind a method.
- Let subclasses override or implement that method.
- Keep client code tied to abstractions.
- Defer exact product selection to concrete creators.

### 30-Second Interview Answer

I would use Factory Method when a framework or base workflow knows when it needs a product, but subclasses should decide which product to create. The base code calls a factory method and works with a product interface. This avoids hard-coding concrete classes in the base workflow, but it adds subclasses and can be heavier than a simple factory for small cases.

## 2. Layman and Easy to Understand Definition

Factory Method is like a restaurant chain with a common ordering process but local branches that decide the exact dish.

The main process says:

```text
Take order.
Prepare meal.
Serve meal.
```

But the branch decides:

```text
Italian branch creates pasta.
Indian branch creates biryani.
Mexican branch creates tacos.
```

The process is shared. The concrete product comes from the specific branch.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a document application has a common export workflow:

1. Create a document formatter.
2. Format the report.
3. Save the output.
4. Log the result.

If the base workflow directly creates a `PdfFormatter`, it cannot easily support CSV or HTML.

Bad design:

```java
public class ReportExporter {
    public void export(Report report) {
        Formatter formatter = new PdfFormatter();
        String output = formatter.format(report);
        save(output);
    }
}
```

Problems:

- The base workflow is tied to one product.
- New product types require editing the base class.
- Subclasses cannot easily customize object creation.
- The open/closed principle is weakened.

### 3.2 The Factory Method Solution

Move creation into a method:

```java
protected abstract Formatter createFormatter();
```

Then subclasses decide:

```java
public final class PdfReportExporter extends ReportExporter {
    @Override
    protected Formatter createFormatter() {
        return new PdfFormatter();
    }
}
```

The base workflow still controls the export process. The subclass controls the concrete product.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Product | Interface or abstract type being created. |
| Concrete product | Specific product implementation. |
| Creator | Base class or interface declaring the factory method. |
| Concrete creator | Subclass that returns a concrete product. |
| Factory method | Method responsible for creating the product. |

### 3.4 Mental Model

Think of Factory Method as a customizable creation hook.

1. Base workflow reaches a point where it needs an object.
2. Base workflow calls a factory method.
3. Concrete subclass returns the right product.
4. Base workflow uses the product through an interface.
5. New subclasses can change creation without changing the base workflow.

## 4. Java Coding Example

This example exports reports using different formatter products.

```java
public record Report(String title, String body) {}

public interface Formatter {
    String format(Report report);
}

public final class PdfFormatter implements Formatter {
    @Override
    public String format(Report report) {
        return "PDF: " + report.title() + "\n" + report.body();
    }
}

public final class HtmlFormatter implements Formatter {
    @Override
    public String format(Report report) {
        return "<h1>" + report.title() + "</h1><p>" + report.body() + "</p>";
    }
}

public abstract class ReportExporter {
    public final String export(Report report) {
        Formatter formatter = createFormatter();
        String output = formatter.format(report);
        audit(report);
        return output;
    }

    protected abstract Formatter createFormatter();

    private void audit(Report report) {
        System.out.println("Exported report: " + report.title());
    }
}

public final class PdfReportExporter extends ReportExporter {
    @Override
    protected Formatter createFormatter() {
        return new PdfFormatter();
    }
}

public final class HtmlReportExporter extends ReportExporter {
    @Override
    protected Formatter createFormatter() {
        return new HtmlFormatter();
    }
}
```

### Java Block by Block Explanation

#### Product Interface

```java
public interface Formatter {
    String format(Report report);
}
```

The creator uses this interface and does not need to know the concrete formatter.

#### Concrete Products

```java
public final class PdfFormatter implements Formatter {
```

Each formatter creates a different representation of the same report.

#### Creator

```java
public abstract class ReportExporter {
```

The creator owns the workflow. It decides the order of steps but delegates product creation.

#### Factory Method

```java
protected abstract Formatter createFormatter();
```

This is the factory method. Subclasses must implement it.

#### Concrete Creator

```java
public final class PdfReportExporter extends ReportExporter {
```

The concrete creator decides which product to instantiate.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        Report report = new Report("Revenue", "Revenue increased by 12 percent.");

        ReportExporter exporter = new PdfReportExporter();
        String output = exporter.export(report);

        System.out.println(output);
    }
}
```

The client can swap `PdfReportExporter` for `HtmlReportExporter` without changing the export workflow.

## 5. Python Coding Example

Python can model Factory Method with inheritance and abstract methods.

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass


@dataclass(frozen=True)
class Report:
    title: str
    body: str


class Formatter(ABC):
    @abstractmethod
    def format(self, report: Report) -> str:
        ...


class PdfFormatter(Formatter):
    def format(self, report: Report) -> str:
        return f"PDF: {report.title}\n{report.body}"


class HtmlFormatter(Formatter):
    def format(self, report: Report) -> str:
        return f"<h1>{report.title}</h1><p>{report.body}</p>"


class ReportExporter(ABC):
    def export(self, report: Report) -> str:
        formatter = self.create_formatter()
        output = formatter.format(report)
        self._audit(report)
        return output

    @abstractmethod
    def create_formatter(self) -> Formatter:
        ...

    def _audit(self, report: Report) -> None:
        print(f"Exported report: {report.title}")


class PdfReportExporter(ReportExporter):
    def create_formatter(self) -> Formatter:
        return PdfFormatter()


class HtmlReportExporter(ReportExporter):
    def create_formatter(self) -> Formatter:
        return HtmlFormatter()
```

### Python Usage

```python
report = Report("Revenue", "Revenue increased by 12 percent.")
exporter: ReportExporter = PdfReportExporter()
print(exporter.export(report))
```

## 6. Where It Comes Handy in Real Life

Factory Method is useful when a framework or base workflow needs subclasses to supply product details.

Examples:

- UI framework creates platform-specific buttons.
- Document exporter creates format-specific writers.
- Game engine creates level-specific enemies.
- Payment workflow creates provider-specific request builders.
- Test framework creates test runners.
- Parser framework creates tokenizers by language subclass.

## 7. Advantages Over Normal Code Without Pattern

### Without Factory Method

```java
public final class ReportExporter {
    public String export(Report report) {
        Formatter formatter = new PdfFormatter();
        return formatter.format(report);
    }
}
```

Problems:

- The workflow is hard-coded to PDF.
- Supporting HTML requires editing the class.
- Creation cannot vary cleanly by subclass.
- Product creation is mixed into workflow logic.

### With Factory Method

```java
public abstract class ReportExporter {
    public final String export(Report report) {
        return createFormatter().format(report);
    }

    protected abstract Formatter createFormatter();
}
```

Benefits:

- Base workflow stays stable.
- Subclasses choose products.
- Product creation is extensible.
- Client code can use the creator abstraction.

## 8. Where It Excels

Factory Method excels when:

- A base class owns an algorithm but subclasses choose products.
- You want extension through inheritance.
- You want to avoid hard-coded concrete products in framework code.
- Product families align with creator subclasses.
- The product type may vary by domain-specific subclass.

## 9. Where It Fails

Factory Method is a poor fit when:

- A simple factory would be enough.
- You do not want inheritance.
- Product selection is data-driven by config or enum.
- You need to create families of related products, where Abstract Factory may fit better.
- The hierarchy grows only to support object creation and adds no real behavior.

Example where Factory Method is overkill:

```java
Formatter formatter = new PdfFormatter();
```

If there is only one formatter and no extension point, direct construction is clearer.

## 10. Prebuilt Libraries and Packages

### Java

Common factory-method-like APIs:

- `Collection.iterator()`
- `Calendar.getInstance()`
- `ResourceBundle.getBundle()`
- `NumberFormat.getInstance()`
- `DocumentBuilderFactory.newDocumentBuilder()`
- Framework lifecycle hooks that let subclasses create components.

Implementation tools:

- Abstract classes with protected factory methods.
- Interfaces with creation methods.
- Template Method plus Factory Method.

### Python

Python alternatives:

- Abstract base classes.
- Subclass overrides.
- Factory functions.
- Dictionaries of constructor callables.
- Class methods for alternate constructors.

Python code often prefers composition or factory functions unless inheritance already fits the design.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples base workflow from concrete products. | Adds inheritance and subclasses. |
| Supports open/closed extension. | Can be heavier than Simple Factory. |
| Gives subclasses control over product creation. | Product selection may be less obvious to beginners. |
| Works well with Template Method. | Class hierarchy can grow quickly. |
| Keeps client tied to abstractions. | Not ideal for purely config-driven selection. |

## 12. Real-World Identification Example

Scenario:

You are building a cloud deployment framework. The base deployment flow is the same, but each cloud provider creates a different deployment client.

Creators:

- `AwsDeploymentPipeline`
- `AzureDeploymentPipeline`
- `GcpDeploymentPipeline`

Products:

- `AwsDeploymentClient`
- `AzureDeploymentClient`
- `GcpDeploymentClient`

Should you use Factory Method?

Yes, if each provider subclass owns provider-specific behavior and product creation.

Why:

- The deployment algorithm can stay in the base class.
- Each subclass can create its provider-specific client.
- The base workflow only depends on the `DeploymentClient` interface.

Good factory method usage:

```java
DeploymentPipeline pipeline = new AwsDeploymentPipeline();
pipeline.deploy(application);
```

Inside `deploy()`, the base class calls `createDeploymentClient()`.

## 13. MAANG Interview Triggers

Think Factory Method when you hear:

- Let subclasses decide which object to create.
- Defer instantiation to subclasses.
- Base workflow with customizable object creation.
- Creator and product hierarchies.
- Virtual constructor.
- Template Method plus object creation.
- Framework extension point.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the product interface.
2. Identify the base creator or workflow.
3. Put object creation behind a factory method.
4. Let concrete creators override the method.
5. Keep the base workflow using the product abstraction.
6. Mention trade-off: extra subclasses and inheritance.

## 14. Common Mistakes

### Mistake 1: Calling Every Factory a Factory Method

Bad wording:

```text
This static create() method is Factory Method.
```

A static factory can be useful, but GoF Factory Method specifically involves a method that subclasses implement or override.

### Mistake 2: Using Factory Method Without a Real Subclass Need

If there is no meaningful creator hierarchy, Simple Factory or DI may be cleaner.

### Mistake 3: Returning Concrete Products

Return the product abstraction:

```java
protected abstract Formatter createFormatter();
```

not:

```java
protected abstract PdfFormatter createFormatter();
```

### Mistake 4: Putting Too Much Logic in the Factory Method

The factory method should create or choose the product. Keep business workflow in the creator.

### Mistake 5: Confusing Product and Creator

The product is what gets created. The creator is the class that contains the factory method.

## 15. Factory Method vs Similar Patterns

| Pattern | Difference |
|---|---|
| Simple Factory | Usually one class or function chooses products. Factory Method lets subclasses override product creation. |
| Abstract Factory | Creates families of related products. Factory Method creates one product through an overridable method. |
| Builder | Builds one complex object step by step. Factory Method chooses product type through subclassing. |
| Template Method | Defines algorithm steps. Factory Method is often one step inside a template method. |
| Dependency Injection | Supplies collaborators from outside. Factory Method creates products inside a creator hierarchy. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Product | `Formatter` | Interface for created objects. |
| Concrete product | `PdfFormatter`, `HtmlFormatter` | Actual objects created. |
| Creator | `ReportExporter` | Declares factory method and owns workflow. |
| Concrete creator | `PdfReportExporter`, `HtmlReportExporter` | Implements product creation. |
| Factory method | `createFormatter()` | Creates the product. |

The important interview sentence:

> The creator controls the workflow, but concrete creators control which product gets created.

## 17. Quick Revision Notes

- Factory Method defers object creation to subclasses.
- It is a GoF creational pattern.
- It is not the same as a static factory.
- It works well with Template Method.
- Return product abstractions, not concrete products.
- Use it when a creator hierarchy already makes sense.

## 18. Mini Exercise

Design a Factory Method for `CloudBackupJob`.

Base workflow:

- Compress files.
- Create provider-specific storage client.
- Upload archive.
- Record backup result.

Concrete creators:

- `S3BackupJob`
- `AzureBlobBackupJob`
- `GcsBackupJob`

Expected factory method:

```java
protected abstract StorageClient createStorageClient();
```

## 19. Source Reference in This Repo

The repository's Factory Method implementation uses `Blacksmith` creators that manufacture different `Weapon` products.

Useful files:

- [github-repo/factory-method/README.md](../../github-repo/factory-method/README.md)
- [github-repo/factory-method/src/main/java/com/iluwatar/factory/method/Blacksmith.java](../../github-repo/factory-method/src/main/java/com/iluwatar/factory/method/Blacksmith.java)
- [github-repo/factory-method/src/main/java/com/iluwatar/factory/method/ElfBlacksmith.java](../../github-repo/factory-method/src/main/java/com/iluwatar/factory/method/ElfBlacksmith.java)
- [github-repo/factory-method/src/main/java/com/iluwatar/factory/method/OrcBlacksmith.java](../../github-repo/factory-method/src/main/java/com/iluwatar/factory/method/OrcBlacksmith.java)
- [github-repo/factory-method/src/main/java/com/iluwatar/factory/method/Weapon.java](../../github-repo/factory-method/src/main/java/com/iluwatar/factory/method/Weapon.java)
