# Bridge Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: High  
Software usage meter: Medium  
Repository module: [github-repo/bridge](../../github-repo/bridge)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the problem of two dimensions growing independently.
2. Second pass: rewrite the Java example and identify the abstraction side and implementation side.
3. Third pass: compare Bridge with Adapter, Strategy, and Decorator.

By the end, you should be able to say:

> Bridge separates an abstraction from its implementation so both sides can evolve independently without creating a class explosion.

## 1. Technical Definition

Bridge is a structural design pattern that decouples an abstraction from its implementation by moving implementation behavior into a separate hierarchy and connecting the two through composition.

Core idea:

- Split one large inheritance hierarchy into two smaller hierarchies.
- The abstraction owns a reference to an implementation.
- Both sides can vary independently.
- Prefer composition over multiplying subclasses.

### 30-Second Interview Answer

I would use Bridge when a design has two dimensions of variation, such as devices and remotes, shapes and renderers, or notifications and delivery channels. Instead of creating every combination as a subclass, I keep one hierarchy for the abstraction and another for implementation, then connect them by composition. This reduces class explosion but adds indirection.

## 2. Layman and Easy to Understand Definition

Bridge is like a remote control and a TV.

You can have different remotes:

```text
basic remote
advanced remote
voice remote
```

And different TVs:

```text
Sony TV
Samsung TV
LG TV
```

You do not create `SonyBasicRemote`, `SonyVoiceRemote`, `SamsungBasicRemote`, and so on. A remote talks to a TV interface. That connection is the bridge.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose you model reports and output formats using inheritance.

```java
class PdfSalesReport {}
class CsvSalesReport {}
class HtmlSalesReport {}
class PdfInventoryReport {}
class CsvInventoryReport {}
class HtmlInventoryReport {}
```

Problems:

- Each new report type multiplies classes.
- Each new format multiplies classes.
- Business behavior and rendering behavior are mixed.
- The hierarchy becomes hard to extend.

### 3.2 The Bridge Solution

Split the dimensions:

```java
abstract class Report {
    protected final Renderer renderer;
}

interface Renderer {
    void render(String title, String body);
}
```

Now `SalesReport` can use `PdfRenderer`, `HtmlRenderer`, or any future renderer without creating report-format subclasses.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Abstraction | High-level concept used by clients. |
| Refined abstraction | Specialized abstraction variant. |
| Implementor | Interface for implementation behavior. |
| Concrete implementor | Specific implementation detail. |
| Bridge | Composition link from abstraction to implementor. |

### 3.4 Mental Model

Think of Bridge as two ladders connected by a handle.

1. One ladder represents business abstraction.
2. Another ladder represents implementation details.
3. The abstraction holds an implementation reference.
4. New abstractions do not require new implementations.
5. New implementations do not require new abstractions.

## 4. Java Coding Example

This example separates reports from renderers.

```java
public interface Renderer {
    void render(String title, String body);
}

public final class PdfRenderer implements Renderer {
    @Override
    public void render(String title, String body) {
        System.out.println("PDF: " + title + "\n" + body);
    }
}

public final class HtmlRenderer implements Renderer {
    @Override
    public void render(String title, String body) {
        System.out.println("<h1>" + title + "</h1><p>" + body + "</p>");
    }
}

public abstract class Report {
    protected final Renderer renderer;

    protected Report(Renderer renderer) {
        this.renderer = renderer;
    }

    public abstract void export();
}

public final class SalesReport extends Report {
    public SalesReport(Renderer renderer) {
        super(renderer);
    }

    @Override
    public void export() {
        renderer.render("Sales Report", "Revenue increased by 12 percent.");
    }
}

public final class InventoryReport extends Report {
    public InventoryReport(Renderer renderer) {
        super(renderer);
    }

    @Override
    public void export() {
        renderer.render("Inventory Report", "Current stock is healthy.");
    }
}
```

### Java Block by Block Explanation

#### Implementor

```java
public interface Renderer {
    void render(String title, String body);
}
```

`Renderer` is the implementation side. It handles output details.

#### Concrete Implementors

```java
public final class PdfRenderer implements Renderer {
```

Each renderer implements output in a different way.

#### Abstraction

```java
public abstract class Report {
    protected final Renderer renderer;
}
```

`Report` is the abstraction side. It owns a `Renderer` reference instead of hard-coding output.

#### Refined Abstraction

```java
public final class SalesReport extends Report {
```

`SalesReport` defines report-specific content and delegates rendering.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        Report report = new SalesReport(new PdfRenderer());
        report.export();

        Report sameReportDifferentFormat = new SalesReport(new HtmlRenderer());
        sameReportDifferentFormat.export();
    }
}
```

The same abstraction can use different implementations.

## 5. Python Coding Example

```python
from abc import ABC, abstractmethod
from typing import Protocol


class Renderer(Protocol):
    def render(self, title: str, body: str) -> None:
        ...


class PdfRenderer:
    def render(self, title: str, body: str) -> None:
        print(f"PDF: {title}\n{body}")


class HtmlRenderer:
    def render(self, title: str, body: str) -> None:
        print(f"<h1>{title}</h1><p>{body}</p>")


class Report(ABC):
    def __init__(self, renderer: Renderer) -> None:
        self._renderer = renderer

    @abstractmethod
    def export(self) -> None:
        ...


class SalesReport(Report):
    def export(self) -> None:
        self._renderer.render("Sales Report", "Revenue increased by 12 percent.")
```

### Python Usage

```python
report = SalesReport(PdfRenderer())
report.export()
```

## 6. Where It Comes Handy in Real Life

Bridge is useful when two axes vary independently.

Examples:

- Shapes and renderers.
- Reports and output formats.
- Notifications and delivery providers.
- Remote controls and devices.
- UI abstractions and platform implementations.
- Persistence abstractions and database drivers.

## 7. Advantages Over Normal Code Without Pattern

### Without Bridge

```java
class PdfSalesReport {}
class HtmlSalesReport {}
class PdfInventoryReport {}
class HtmlInventoryReport {}
```

Problems:

- Class count grows by multiplication.
- Changes in one dimension affect another.
- Code duplication increases.
- Extension becomes awkward.

### With Bridge

```java
Report report = new SalesReport(new PdfRenderer());
```

Benefits:

- Abstractions and implementations vary independently.
- Class count grows by addition, not multiplication.
- Composition makes runtime switching possible.
- Implementation details are hidden behind interfaces.

## 8. Where It Excels

Bridge excels when:

- Two independent dimensions are growing.
- Inheritance is causing class explosion.
- Runtime implementation switching is useful.
- Abstraction code should not depend on concrete implementation details.
- You need long-term extensibility.

## 9. Where It Fails

Bridge is a poor fit when:

- There is only one dimension of variation.
- A simple Strategy is enough.
- The design is small and unlikely to grow.
- The extra abstraction makes code harder to understand.
- The two sides are not truly independent.

## 10. Prebuilt Libraries and Packages

### Java

Bridge-like designs appear in:

- JDBC APIs and database driver implementations.
- Logging APIs and logging backends.
- UI abstractions over platform-specific rendering.
- Drawing APIs with rendering backends.

### Python

Python often uses:

- Protocols or abstract base classes.
- Composition with injected implementation objects.
- Plugin registries.
- Strategy-like objects when only one dimension varies.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids class explosion. | Adds indirection. |
| Separates abstraction from implementation. | More interfaces/classes. |
| Allows independent extension. | Can be overkill for small designs. |
| Supports runtime implementation switching. | Requires careful naming. |
| Encourages composition. | Two hierarchies can be harder to trace. |

## 12. Real-World Identification Example

Scenario:

You are designing notifications. Message types and delivery channels both vary.

Message types:

- Welcome notification
- Password reset notification
- Invoice notification

Delivery channels:

- Email
- SMS
- Push

Should you use Bridge?

Yes, if message behavior and delivery behavior should evolve independently.

Good usage:

```java
Notification notification = new InvoiceNotification(new EmailSender());
notification.send();
```

## 13. MAANG Interview Triggers

Think Bridge when you hear:

- Two dimensions vary independently.
- Avoid class explosion.
- Decouple abstraction from implementation.
- Prefer composition over inheritance.
- Runtime implementation choice.
- Separate API from platform details.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify both dimensions of variation.
2. Make one side the abstraction hierarchy.
3. Make the other side the implementation hierarchy.
4. Connect them through composition.
5. Show how a new abstraction or implementation is added independently.
6. Mention trade-off: more indirection and more interfaces.

## 14. Common Mistakes

### Mistake 1: Using Bridge for One Dimension

If only one thing varies, Strategy or simple composition may be enough.

### Mistake 2: Confusing Bridge with Adapter

Adapter fixes incompatible existing interfaces. Bridge is designed up front to let two dimensions vary.

### Mistake 3: Letting Abstraction Know Concrete Implementors

Bad:

```java
this.renderer = new PdfRenderer();
```

Better:

```java
this.renderer = renderer;
```

### Mistake 4: Creating Two Hierarchies Too Early

Do not use Bridge before there is real variation.

### Mistake 5: Poor Names

Bridge is easier to understand when the abstraction and implementor names reveal their separate responsibilities.

## 15. Bridge vs Similar Patterns

| Pattern | Difference |
|---|---|
| Adapter | Adapter makes incompatible interfaces work together. Bridge separates dimensions before they explode. |
| Strategy | Strategy usually swaps one behavior. Bridge separates two object hierarchies. |
| Decorator | Decorator stacks extra behavior. Bridge separates abstraction from implementation. |
| Abstract Factory | Abstract Factory creates product families. Bridge composes abstraction with implementation. |
| Dependency Injection | DI can inject the implementor used by a Bridge. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Abstraction | `Report` | High-level API. |
| Refined abstraction | `SalesReport` | Specialized high-level behavior. |
| Implementor | `Renderer` | Implementation interface. |
| Concrete implementor | `PdfRenderer`, `HtmlRenderer` | Implementation details. |

## 17. Quick Revision Notes

- Bridge separates abstraction from implementation.
- It helps when two dimensions vary independently.
- It avoids subclass multiplication.
- The abstraction holds an implementor reference.
- Adapter is often after-the-fact; Bridge is usually designed up front.
- Do not use it before the variation is real.

## 18. Mini Exercise

Design a Bridge for `Chart`.

Abstractions:

- `BarChart`
- `LineChart`

Implementors:

- `SvgRenderer`
- `CanvasRenderer`
- `PdfRenderer`

Expected usage:

```java
Chart chart = new BarChart(new SvgRenderer());
chart.draw(data);
```

## 19. Source Reference in This Repo

The repository's Bridge implementation separates `Weapon` abstractions from `Enchantment` implementations.

Useful files:

- [github-repo/bridge/README.md](../../github-repo/bridge/README.md)
- [github-repo/bridge/src/main/java/com/iluwatar/bridge/Weapon.java](../../github-repo/bridge/src/main/java/com/iluwatar/bridge/Weapon.java)
- [github-repo/bridge/src/main/java/com/iluwatar/bridge/Sword.java](../../github-repo/bridge/src/main/java/com/iluwatar/bridge/Sword.java)
- [github-repo/bridge/src/main/java/com/iluwatar/bridge/Hammer.java](../../github-repo/bridge/src/main/java/com/iluwatar/bridge/Hammer.java)
- [github-repo/bridge/src/main/java/com/iluwatar/bridge/Enchantment.java](../../github-repo/bridge/src/main/java/com/iluwatar/bridge/Enchantment.java)
