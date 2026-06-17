# Abstract Factory Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/abstract-factory](../../github-repo/abstract-factory)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the phrase "families of related objects".
2. Second pass: rewrite the Java example and identify the abstract factory, concrete factories, abstract products, and concrete products.
3. Third pass: compare Abstract Factory with Factory, Factory Method, and Dependency Injection.

By the end, you should be able to say:

> Abstract Factory creates related products that must be used together, while hiding the concrete family from client code.

## 1. Technical Definition

Abstract Factory is a creational design pattern that provides an interface for creating families of related or dependent objects without specifying their concrete classes.

Core idea:

- Group related object creation behind one factory interface.
- Keep product families consistent.
- Let client code depend on abstract products.
- Swap the whole family by changing the concrete factory.

### 30-Second Interview Answer

I would use Abstract Factory when the system needs a set of related objects that must match each other, such as UI widgets for the same platform or cloud components for the same provider. The client receives an abstract factory and asks it for abstract products. This keeps the family consistent and hides concrete classes, but it adds multiple interfaces and factories, so it is overkill for one product type.

## 2. Layman and Easy to Understand Definition

Abstract Factory is like choosing a furniture style for an entire room.

If you choose modern style, you get:

```text
modern chair
modern table
modern sofa
```

If you choose classic style, you get:

```text
classic chair
classic table
classic sofa
```

You do not mix random pieces accidentally. One factory creates one consistent family.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a UI app supports web and mobile rendering.

Bad design:

```java
Button button = new WebButton();
Dialog dialog = new MobileDialog();
Menu menu = new WebMenu();
```

Problems:

- Products from different families can be mixed.
- Client code knows concrete classes.
- Switching the whole platform requires editing many places.
- Consistency rules are scattered.

### 3.2 The Abstract Factory Solution

Create one factory interface for the related products:

```java
public interface UiFactory {
    Button createButton();
    Dialog createDialog();
    Menu createMenu();
}
```

Then create concrete factories:

```java
public final class WebUiFactory implements UiFactory {
    public Button createButton() {
        return new WebButton();
    }
}
```

The client uses only `UiFactory`, `Button`, `Dialog`, and `Menu`.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Abstract factory | Interface that creates related products. |
| Concrete factory | Creates one concrete product family. |
| Abstract product | Product interface used by the client. |
| Concrete product | Family-specific implementation. |
| Client | Receives a factory and uses products through interfaces. |

### 3.4 Mental Model

Think of Abstract Factory as a family selector.

1. Choose a family, such as web or mobile.
2. Pass the matching concrete factory to the client.
3. Client asks the factory for related products.
4. Factory returns products from the same family.
5. Client uses abstract product interfaces.

## 4. Java Coding Example

This example creates UI components for web or mobile.

```java
public interface Button {
    void render();
}

public interface Dialog {
    void open();
}

public final class WebButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering web button");
    }
}

public final class WebDialog implements Dialog {
    @Override
    public void open() {
        System.out.println("Opening web dialog");
    }
}

public final class MobileButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering mobile button");
    }
}

public final class MobileDialog implements Dialog {
    @Override
    public void open() {
        System.out.println("Opening mobile dialog");
    }
}

public interface UiFactory {
    Button createButton();
    Dialog createDialog();
}

public final class WebUiFactory implements UiFactory {
    @Override
    public Button createButton() {
        return new WebButton();
    }

    @Override
    public Dialog createDialog() {
        return new WebDialog();
    }
}

public final class MobileUiFactory implements UiFactory {
    @Override
    public Button createButton() {
        return new MobileButton();
    }

    @Override
    public Dialog createDialog() {
        return new MobileDialog();
    }
}
```

### Java Block by Block Explanation

#### Abstract Products

```java
public interface Button {
    void render();
}
```

`Button` and `Dialog` are product interfaces. Client code depends on them, not on web or mobile classes.

#### Concrete Products

```java
public final class WebButton implements Button {
```

Concrete products belong to one family. `WebButton` and `WebDialog` belong together.

#### Abstract Factory

```java
public interface UiFactory {
    Button createButton();
    Dialog createDialog();
}
```

The factory interface lists all products in the family.

#### Concrete Factory

```java
public final class WebUiFactory implements UiFactory {
```

This factory creates only web products, keeping the family consistent.

### Java Usage

```java
public final class Screen {
    private final Button button;
    private final Dialog dialog;

    public Screen(UiFactory factory) {
        this.button = factory.createButton();
        this.dialog = factory.createDialog();
    }

    public void render() {
        button.render();
        dialog.open();
    }
}

public class Demo {
    public static void main(String[] args) {
        UiFactory factory = new WebUiFactory();
        Screen screen = new Screen(factory);
        screen.render();
    }
}
```

To switch the whole family, pass `new MobileUiFactory()`.

## 5. Python Coding Example

Python can model Abstract Factory with protocols and concrete factory classes.

```python
from typing import Protocol


class Button(Protocol):
    def render(self) -> None:
        ...


class Dialog(Protocol):
    def open(self) -> None:
        ...


class WebButton:
    def render(self) -> None:
        print("Rendering web button")


class WebDialog:
    def open(self) -> None:
        print("Opening web dialog")


class MobileButton:
    def render(self) -> None:
        print("Rendering mobile button")


class MobileDialog:
    def open(self) -> None:
        print("Opening mobile dialog")


class UiFactory(Protocol):
    def create_button(self) -> Button:
        ...

    def create_dialog(self) -> Dialog:
        ...


class WebUiFactory:
    def create_button(self) -> Button:
        return WebButton()

    def create_dialog(self) -> Dialog:
        return WebDialog()


class MobileUiFactory:
    def create_button(self) -> Button:
        return MobileButton()

    def create_dialog(self) -> Dialog:
        return MobileDialog()
```

### Python Usage

```python
class Screen:
    def __init__(self, factory: UiFactory) -> None:
        self._button = factory.create_button()
        self._dialog = factory.create_dialog()

    def render(self) -> None:
        self._button.render()
        self._dialog.open()


screen = Screen(WebUiFactory())
screen.render()
```

## 6. Where It Comes Handy in Real Life

Abstract Factory is useful when product compatibility matters.

Examples:

- UI components for one platform or theme.
- Cloud provider clients for AWS, Azure, or GCP.
- Database-specific query builders, connections, and migrations.
- Game assets for one level theme.
- Payment provider request, response, and validator objects.
- Serialization families for JSON, XML, or Avro.

## 7. Advantages Over Normal Code Without Pattern

### Without Abstract Factory

```java
Button button = new WebButton();
Dialog dialog = new MobileDialog();
```

Problems:

- Related products may be mixed incorrectly.
- Client code knows concrete classes.
- Swapping product families is expensive.
- Product creation is scattered.

### With Abstract Factory

```java
UiFactory factory = new WebUiFactory();
Button button = factory.createButton();
Dialog dialog = factory.createDialog();
```

Benefits:

- Related products stay consistent.
- Client code depends on abstractions.
- Whole families can be swapped.
- Product creation is centralized by family.

## 8. Where It Excels

Abstract Factory excels when:

- Multiple products must be created together.
- Products belong to compatible families.
- You want to swap families at runtime or startup.
- Client code should not know concrete product classes.
- Family consistency is more important than individual object creation.

## 9. Where It Fails

Abstract Factory is a poor fit when:

- There is only one product type.
- Product families do not need consistency.
- Adding new product kinds is very frequent.
- A simple factory is enough.
- A DI container can wire the family more simply.

Example where Abstract Factory is overkill:

```java
NotificationSender sender = NotificationSenderFactory.create(Channel.EMAIL);
```

This creates one product, not a family.

## 10. Prebuilt Libraries and Packages

### Java

Common abstract-factory-like APIs:

- `DocumentBuilderFactory`
- `TransformerFactory`
- `XPathFactory`
- Swing look-and-feel factories
- JDBC driver/provider families

Implementation helpers:

- Interfaces for product families.
- Dependency Injection profiles.
- Configuration-driven factory selection.

### Python

Python alternatives:

- Factory objects.
- Modules representing product families.
- Dictionaries of factory functions.
- Dependency injection at application startup.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps related products consistent. | Adds several interfaces and classes. |
| Hides concrete product families. | Can feel heavy for small systems. |
| Makes family swapping easier. | Adding a new product kind affects every factory. |
| Encourages interface-based design. | Concrete family selection still needs wiring. |
| Localizes family-specific construction. | More indirection for beginners. |

## 12. Real-World Identification Example

Scenario:

You are building a cloud deployment tool that supports AWS and Azure.

Each provider needs related products:

- Storage client
- Queue client
- Metrics client

Should you use Abstract Factory?

Yes, if provider-specific clients must be used as a consistent family.

Good usage:

```java
CloudFactory factory = new AwsCloudFactory();
StorageClient storage = factory.createStorageClient();
QueueClient queue = factory.createQueueClient();
MetricsClient metrics = factory.createMetricsClient();
```

This avoids accidentally using AWS storage with Azure queue behavior.

## 13. MAANG Interview Triggers

Think Abstract Factory when you hear:

- Families of related objects.
- Products must be compatible.
- Platform-specific UI components.
- Cloud-provider-specific clients.
- Swap whole product family.
- Hide concrete classes.
- Factory of factories.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the product family.
2. Define abstract product interfaces.
3. Define the abstract factory interface.
4. Implement one concrete factory per family.
5. Inject or select the concrete factory at startup.
6. Mention trade-off: adding new product kinds affects all factories.

## 14. Common Mistakes

### Mistake 1: Using Abstract Factory for One Product

If you only create one product type, use Factory or Factory Method instead.

### Mistake 2: Returning Concrete Product Types

Bad:

```java
WebButton createButton();
```

Better:

```java
Button createButton();
```

### Mistake 3: Mixing Families

Do not create a factory that returns `WebButton` and `MobileDialog`. The whole point is consistency.

### Mistake 4: Ignoring the Cost of New Product Kinds

Adding `Menu createMenu()` means every concrete factory must change.

### Mistake 5: Hiding Bad Boundaries Behind Factories

If product families are not real domain concepts, the pattern may add ceremony without value.

## 15. Abstract Factory vs Similar Patterns

| Pattern | Difference |
|---|---|
| Factory | Usually creates one product type. Abstract Factory creates related product families. |
| Factory Method | Lets subclasses decide one product creation method. Abstract Factory groups multiple creation methods. |
| Builder | Configures one complex object step by step. Abstract Factory creates several related objects. |
| Dependency Injection | Wires dependencies from outside. DI can provide the selected abstract factory. |
| Prototype | Clones existing objects. Abstract Factory usually creates new related products. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Abstract factory | `UiFactory` | Creates abstract products. |
| Concrete factory | `WebUiFactory`, `MobileUiFactory` | Creates one product family. |
| Abstract product | `Button`, `Dialog` | Interfaces client code uses. |
| Concrete product | `WebButton`, `MobileDialog` | Actual family-specific objects. |
| Client | `Screen` | Uses factory and product abstractions. |

The key interview sentence:

> Abstract Factory protects product-family consistency.

## 17. Quick Revision Notes

- Abstract Factory creates families of related objects.
- Client code depends on abstract products.
- Concrete factories represent product families.
- It is stronger than Factory when compatibility matters.
- Adding a new family is easy.
- Adding a new product kind can be expensive.

## 18. Mini Exercise

Design an Abstract Factory for `DatabaseToolkit`.

Product family:

- `Connection`
- `QueryBuilder`
- `MigrationRunner`

Families:

- PostgreSQL
- MySQL

Expected usage:

```java
DatabaseToolkit toolkit = new PostgresToolkit();
Connection connection = toolkit.createConnection();
QueryBuilder queryBuilder = toolkit.createQueryBuilder();
MigrationRunner migrationRunner = toolkit.createMigrationRunner();
```

## 19. Source Reference in This Repo

The repository's Abstract Factory implementation creates related kingdom products through `KingdomFactory`.

Useful files:

- [github-repo/abstract-factory/README.md](../../github-repo/abstract-factory/README.md)
- [github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/KingdomFactory.java](../../github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/KingdomFactory.java)
- [github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/ElfKingdomFactory.java](../../github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/ElfKingdomFactory.java)
- [github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/OrcKingdomFactory.java](../../github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/OrcKingdomFactory.java)
- [github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/Kingdom.java](../../github-repo/abstract-factory/src/main/java/com/iluwatar/abstractfactory/Kingdom.java)
