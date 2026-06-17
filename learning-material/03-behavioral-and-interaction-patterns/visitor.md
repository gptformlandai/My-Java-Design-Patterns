# Visitor Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: High  
Software usage meter: Medium  
Repository module: [github-repo/visitor](../../github-repo/visitor)

## How to Study This Page

Use this page in three passes:

1. First pass: understand adding new operations to a stable object structure.
2. Second pass: rewrite the Java example and identify element, concrete elements, visitor, and accept method.
3. Third pass: study the trade-off: easy to add operations, harder to add new element types.

By the end, you should be able to say:

> Visitor lets you add operations across a stable set of element types without putting those operations inside the element classes.

## 1. Technical Definition

Visitor is a behavioral design pattern that separates an operation from the object structure it operates on. Elements accept a visitor, and the visitor has overloaded visit methods for each concrete element type.

Core idea:

- Element classes expose `accept(visitor)`.
- Visitor interface has one method per element type.
- Concrete visitors implement new operations.
- Object structure stays mostly unchanged.

### 30-Second Interview Answer

I would use Visitor when I have a stable object structure, such as an AST or document tree, and I need to add many operations like validation, export, metrics, or pretty-printing without modifying every element repeatedly. It makes adding operations easy, but adding new element types is expensive because every visitor interface and implementation may need a new method.

## 2. Layman and Easy to Understand Definition

Visitor is like sending different inspectors through the same building.

One inspector checks fire safety. Another checks electrical wiring. Another checks accessibility. The rooms do not change, but each visitor performs a different operation on each room type.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose an expression tree has several node types.

Operations keep growing:

- Evaluate.
- Pretty print.
- Validate.
- Count variables.
- Convert to SQL.

If every operation is placed inside each node class, the node classes become crowded.

### 3.2 The Visitor Solution

Nodes accept a visitor:

```java
expression.accept(new PrettyPrintVisitor());
expression.accept(new EvaluationVisitor());
```

Each visitor implements one operation across all node types.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Element | Interface with `accept(visitor)`. |
| Concrete element | Specific node or object type. |
| Visitor | Interface with visit methods for element types. |
| Concrete visitor | Implements one operation. |
| Object structure | Collection/tree of elements. |

### 3.4 Double Dispatch

Visitor relies on double dispatch:

1. Runtime chooses the element's `accept()` method.
2. Inside `accept()`, the element calls the matching visitor method.

```java
visitor.visitNumber(this);
```

This lets the visitor run type-specific logic without client `instanceof` checks.

## 4. Java Coding Example

This example visits expression nodes.

```java
public interface Expression {
    void accept(ExpressionVisitor visitor);
}

public final class NumberExpression implements Expression {
    private final int value;

    public NumberExpression(int value) {
        this.value = value;
    }

    public int value() {
        return value;
    }

    @Override
    public void accept(ExpressionVisitor visitor) {
        visitor.visitNumber(this);
    }
}

public final class AddExpression implements Expression {
    private final Expression left;
    private final Expression right;

    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    public Expression left() {
        return left;
    }

    public Expression right() {
        return right;
    }

    @Override
    public void accept(ExpressionVisitor visitor) {
        visitor.visitAdd(this);
    }
}

public interface ExpressionVisitor {
    void visitNumber(NumberExpression expression);
    void visitAdd(AddExpression expression);
}

public final class PrintVisitor implements ExpressionVisitor {
    @Override
    public void visitNumber(NumberExpression expression) {
        System.out.print(expression.value());
    }

    @Override
    public void visitAdd(AddExpression expression) {
        System.out.print("(");
        expression.left().accept(this);
        System.out.print(" + ");
        expression.right().accept(this);
        System.out.print(")");
    }
}
```

### Java Block by Block Explanation

#### Element

```java
public interface Expression {
    void accept(ExpressionVisitor visitor);
}
```

Every element accepts a visitor.

#### Concrete Element

```java
public final class NumberExpression implements Expression {
```

The element knows its own type and calls the matching visitor method.

#### Visitor Interface

```java
public interface ExpressionVisitor {
```

The visitor interface lists operations for each element type.

#### Concrete Visitor

```java
public final class PrintVisitor implements ExpressionVisitor {
```

This visitor implements one operation: printing the expression.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        Expression expression = new AddExpression(
            new NumberExpression(1),
            new NumberExpression(2)
        );

        expression.accept(new PrintVisitor());
    }
}
```

## 5. Python Coding Example

Python can implement visitor with explicit methods.

```python
from dataclasses import dataclass
from typing import Protocol


class ExpressionVisitor(Protocol):
    def visit_number(self, expression: "NumberExpression") -> None:
        ...

    def visit_add(self, expression: "AddExpression") -> None:
        ...


class Expression(Protocol):
    def accept(self, visitor: ExpressionVisitor) -> None:
        ...


@dataclass(frozen=True)
class NumberExpression:
    value: int

    def accept(self, visitor: ExpressionVisitor) -> None:
        visitor.visit_number(self)


@dataclass(frozen=True)
class AddExpression:
    left: Expression
    right: Expression

    def accept(self, visitor: ExpressionVisitor) -> None:
        visitor.visit_add(self)


class PrintVisitor:
    def visit_number(self, expression: NumberExpression) -> None:
        print(expression.value, end="")

    def visit_add(self, expression: AddExpression) -> None:
        print("(", end="")
        expression.left.accept(self)
        print(" + ", end="")
        expression.right.accept(self)
        print(")", end="")
```

### Python Usage

```python
expression = AddExpression(NumberExpression(1), NumberExpression(2))
expression.accept(PrintVisitor())
```

## 6. Where It Comes Handy in Real Life

Examples:

- Abstract syntax trees.
- Compiler passes.
- Document object models.
- Validation over object trees.
- Exporters over stable structures.
- Static analysis tools.
- Reporting over hierarchical models.

## 7. Advantages Over Normal Code Without Pattern

### Without Visitor

```java
if (node instanceof NumberExpression number) {
    print(number.value());
}
```

Problems:

- Type checks spread everywhere.
- New operations are scattered.
- Element classes may become bloated.
- Traversal logic repeats.

### With Visitor

```java
expression.accept(new PrintVisitor());
```

Benefits:

- Operations are grouped by visitor.
- New operations can be added as new visitor classes.
- Element classes stay focused.
- Works well with composite structures.

## 8. Where It Excels

Visitor excels when:

- Element types are stable.
- New operations are frequent.
- Object structure is tree-like.
- Operations need type-specific behavior.
- You want to avoid polluting element classes.

## 9. Where It Fails

Visitor is a poor fit when:

- New element types are frequent.
- Visitor interface changes often.
- Operation logic belongs naturally inside elements.
- The structure is simple.
- Double dispatch makes code harder for the team.

## 10. Prebuilt Libraries and Packages

### Java

Examples:

- Compiler and parser AST visitors.
- Java annotation processing visitors.
- File tree visitors.
- DOM traversal visitors.

### Python

Examples:

- `ast.NodeVisitor`
- Static analysis tools.
- Tree walkers.
- Custom document processors.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Easy to add new operations. | Hard to add new element types. |
| Keeps element classes focused. | Visitor interface can become large. |
| Groups operation logic together. | Double dispatch can be confusing. |
| Works well with trees. | Can expose element internals. |
| Avoids repeated type checks. | Lots of boilerplate in Java. |

## 12. Real-World Identification Example

Scenario:

You are building an expression language.

Elements:

- Number
- Variable
- Add
- Multiply

Operations:

- Evaluate
- Pretty print
- Validate
- Generate SQL

Should you use Visitor?

Yes, if node types are stable and operations keep growing.

## 13. MAANG Interview Triggers

Think Visitor when you hear:

- Add operations without changing element classes.
- Stable object structure.
- AST.
- Double dispatch.
- Tree operations.
- Type-specific operation logic.
- Export/validate/print over same structure.

### Interview-Ready Answer Format

1. Identify stable element types.
2. Add `accept(visitor)` to elements.
3. Define visitor interface with one visit method per element.
4. Implement one concrete visitor per operation.
5. Traverse object structure through accept calls.
6. Mention trade-off: adding new element types is expensive.

## 14. Common Mistakes

### Mistake 1: Using Visitor When Element Types Change Often

Every new element type forces updates to all visitors.

### Mistake 2: Visitor Doing Too Many Operations

One visitor should usually represent one operation.

### Mistake 3: Exposing Too Much Internal State

Visitors may need element data, but avoid exposing everything casually.

### Mistake 4: Replacing Simple Polymorphism Too Early

If there is one operation, normal methods may be clearer.

### Mistake 5: Ignoring Traversal Ownership

Decide whether elements traverse children or visitors control traversal.

## 15. Visitor vs Similar Patterns

| Pattern | Difference |
|---|---|
| Composite | Composite models tree structure. Visitor adds operations over that structure. |
| Iterator | Iterator traverses elements one by one. Visitor performs type-specific operations. |
| Strategy | Strategy swaps one algorithm. Visitor adds operations across many element types. |
| Command | Command packages an action. Visitor performs operation across object structure. |
| Interpreter | Interpreter evaluates language grammar; Visitor can implement operations over AST nodes. |

## 16. Visitor Design Checklist

| Concern | Why it matters |
|---|---|
| Stable element types | Visitor depends on fixed type set. |
| Growing operations | Visitor shines when operations grow. |
| Traversal owner | Elements or visitors must own traversal. |
| Return values | Java visitors often need generic return types for results. |
| Internal exposure | Visitors may pressure element encapsulation. |

## 17. Quick Revision Notes

- Visitor adds operations to stable structures.
- Elements implement `accept(visitor)`.
- Visitor has one method per element type.
- Great for ASTs and trees.
- Easy to add operations.
- Hard to add element types.

## 18. Mini Exercise

Design Visitor for a document tree.

Elements:

- `Heading`
- `Paragraph`
- `ImageBlock`

Visitors:

- `PlainTextExportVisitor`
- `WordCountVisitor`
- `ValidationVisitor`

Expected usage:

```java
document.accept(new WordCountVisitor());
```

## 19. Source Reference in This Repo

The repository's Visitor implementation uses `UnitVisitor` with `Commander`, `Sergeant`, and `Soldier` element types.

Useful files:

- [github-repo/visitor/README.md](../../github-repo/visitor/README.md)
- [github-repo/visitor/src/main/java/com/iluwatar/visitor/UnitVisitor.java](../../github-repo/visitor/src/main/java/com/iluwatar/visitor/UnitVisitor.java)
- [github-repo/visitor/src/main/java/com/iluwatar/visitor/Unit.java](../../github-repo/visitor/src/main/java/com/iluwatar/visitor/Unit.java)
- [github-repo/visitor/src/main/java/com/iluwatar/visitor/Commander.java](../../github-repo/visitor/src/main/java/com/iluwatar/visitor/Commander.java)
- [github-repo/visitor/src/main/java/com/iluwatar/visitor/SoldierVisitor.java](../../github-repo/visitor/src/main/java/com/iluwatar/visitor/SoldierVisitor.java)
