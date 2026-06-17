# Composite Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: High  
Software usage meter: Medium  
Repository module: [github-repo/composite](../../github-repo/composite)

## How to Study This Page

Use this page in three passes:

1. First pass: understand part-whole tree structures.
2. Second pass: rewrite the Java example and identify component, leaf, composite, and client.
3. Third pass: study recursion, traversal, and where Composite becomes too general.

By the end, you should be able to say:

> Composite lets clients treat individual objects and groups of objects uniformly through the same interface.

## 1. Technical Definition

Composite is a structural design pattern that composes objects into tree structures to represent part-whole hierarchies. It lets clients treat leaves and containers through a common component interface.

Core idea:

- Define a common component interface.
- Leaf objects implement the interface directly.
- Composite objects contain child components.
- Clients call the same operation on leaves and composites.

### 30-Second Interview Answer

I would use Composite when the domain is naturally a tree, such as files and folders, UI components, menus, organization charts, or expression trees. Both single items and groups implement the same interface, so client code can recurse uniformly. The trade-off is that the interface can become too broad if leaf-only and composite-only operations are forced together.

## 2. Layman and Easy to Understand Definition

Composite is like folders on a computer.

A folder can contain:

```text
files
other folders
folders inside folders
```

You can ask both a file and a folder for size. A file returns its own size. A folder returns the combined size of everything inside it.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose you model a file system.

Without Composite, client code may branch everywhere:

```java
if (node instanceof File) {
    total += file.size();
} else if (node instanceof Folder) {
    for (Node child : folder.children()) {
        total += calculate(child);
    }
}
```

Problems:

- Client code must know concrete node types.
- Tree traversal logic repeats.
- Adding new node types is harder.
- Leaf and group behavior are not uniform.

### 3.2 The Composite Solution

Define one common component:

```java
public interface FileSystemNode {
    long size();
}
```

A file returns its own size. A folder sums child sizes. The client calls `size()` on either one.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Component | Common interface for leaves and composites. |
| Leaf | Single object with no children. |
| Composite | Object that contains child components. |
| Client | Uses component interface uniformly. |
| Tree operation | Operation that often recurses through children. |

### 3.4 Mental Model

Think of Composite as recursive uniformity.

1. Every node implements the same interface.
2. Leaves perform the operation directly.
3. Composites forward the operation to children.
4. Results may be combined.
5. Client code does not need to distinguish leaf vs group.

## 4. Java Coding Example

This example models files and folders.

```java
import java.util.ArrayList;
import java.util.List;

public interface FileSystemNode {
    String name();
    long size();
    void print(String indent);
}

public final class FileNode implements FileSystemNode {
    private final String name;
    private final long size;

    public FileNode(String name, long size) {
        this.name = name;
        this.size = size;
    }

    @Override
    public String name() {
        return name;
    }

    @Override
    public long size() {
        return size;
    }

    @Override
    public void print(String indent) {
        System.out.println(indent + "- " + name + " (" + size + ")");
    }
}

public final class FolderNode implements FileSystemNode {
    private final String name;
    private final List<FileSystemNode> children = new ArrayList<>();

    public FolderNode(String name) {
        this.name = name;
    }

    public void add(FileSystemNode child) {
        children.add(child);
    }

    @Override
    public String name() {
        return name;
    }

    @Override
    public long size() {
        return children.stream().mapToLong(FileSystemNode::size).sum();
    }

    @Override
    public void print(String indent) {
        System.out.println(indent + "+ " + name + " (" + size() + ")");
        children.forEach(child -> child.print(indent + "  "));
    }
}
```

### Java Block by Block Explanation

#### Component

```java
public interface FileSystemNode {
```

The component interface is shared by both individual files and folders.

#### Leaf

```java
public final class FileNode implements FileSystemNode {
```

A file has no child nodes. It answers operations directly.

#### Composite

```java
public final class FolderNode implements FileSystemNode {
```

A folder contains child components. Those children can be files or folders.

#### Recursive Operation

```java
return children.stream().mapToLong(FileSystemNode::size).sum();
```

The folder calculates its size by asking all children for their sizes.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        FolderNode root = new FolderNode("project");
        root.add(new FileNode("README.md", 2));

        FolderNode src = new FolderNode("src");
        src.add(new FileNode("App.java", 8));
        src.add(new FileNode("Service.java", 12));

        root.add(src);
        root.print("");
    }
}
```

The client calls `print()` once on the root.

## 5. Python Coding Example

```python
from dataclasses import dataclass, field
from typing import Protocol


class FileSystemNode(Protocol):
    def name(self) -> str:
        ...

    def size(self) -> int:
        ...

    def print(self, indent: str = "") -> None:
        ...


@dataclass
class FileNode:
    file_name: str
    file_size: int

    def name(self) -> str:
        return self.file_name

    def size(self) -> int:
        return self.file_size

    def print(self, indent: str = "") -> None:
        print(f"{indent}- {self.file_name} ({self.file_size})")


@dataclass
class FolderNode:
    folder_name: str
    children: list[FileSystemNode] = field(default_factory=list)

    def name(self) -> str:
        return self.folder_name

    def add(self, child: FileSystemNode) -> None:
        self.children.append(child)

    def size(self) -> int:
        return sum(child.size() for child in self.children)

    def print(self, indent: str = "") -> None:
        print(f"{indent}+ {self.folder_name} ({self.size()})")
        for child in self.children:
            child.print(indent + "  ")
```

### Python Usage

```python
root = FolderNode("project")
root.add(FileNode("README.md", 2))
root.print()
```

## 6. Where It Comes Handy in Real Life

Composite is common for tree-shaped domains.

Examples:

- File systems.
- UI component trees.
- Menus and menu items.
- Organization charts.
- Product bundles.
- Document object models.
- Expression trees.
- Permission groups and individual permissions.

## 7. Advantages Over Normal Code Without Pattern

### Without Composite

```java
if (item.isFolder()) {
    calculateFolder(item);
} else {
    calculateFile(item);
}
```

Problems:

- Type checks spread through client code.
- Tree traversal is repeated.
- New node types require client changes.
- Code is harder to recurse cleanly.

### With Composite

```java
long total = root.size();
```

Benefits:

- Leaf and composite are used uniformly.
- Recursive operations are localized.
- Client code is simpler.
- Tree structures become natural to model.

## 8. Where It Excels

Composite excels when:

- The domain is a tree.
- Clients should treat groups and individual items uniformly.
- Recursive operations are common.
- You need part-whole hierarchies.
- New node types should fit the same interface.

## 9. Where It Fails

Composite is a poor fit when:

- The domain is not hierarchical.
- Leaf and composite operations are very different.
- The common interface becomes fake or bloated.
- You need strict rules about which child types are allowed.
- Tree traversal performance needs specialized handling.

## 10. Prebuilt Libraries and Packages

### Java

Composite-like structures appear in:

- Swing/AWT component trees.
- XML/DOM APIs.
- JavaFX scene graph.
- File tree APIs.
- Parser ASTs.

### Python

Python examples:

- XML element trees.
- UI component trees.
- AST nodes.
- Nested dictionaries/lists for lightweight composite structures.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Models trees naturally. | Can make interfaces too general. |
| Treats leaves and groups uniformly. | Child rules can be hard to restrict. |
| Simplifies recursive client code. | Debugging deep trees can be harder. |
| Makes adding node types easier. | Operations may require Visitor for clean extension. |
| Encapsulates traversal behavior. | Cycles must be prevented if tree assumptions matter. |

## 12. Real-World Identification Example

Scenario:

You are designing a pricing engine for products and bundles.

Objects:

- Single product
- Bundle of products
- Bundle inside another bundle

Should you use Composite?

Yes.

Good usage:

```java
PricedItem cart = new Bundle("cart");
long total = cart.price();
```

The cart can contain products and bundles while the pricing code uses one interface.

## 13. MAANG Interview Triggers

Think Composite when you hear:

- Tree structure.
- Part-whole hierarchy.
- Treat individual and group uniformly.
- Recursive operation.
- File/folder.
- UI component tree.
- Organization chart.
- Product bundle.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the tree structure.
2. Define the component interface.
3. Implement leaf nodes.
4. Implement composite nodes with child components.
5. Put recursive behavior in the composite.
6. Mention trade-off: common interface may become too broad.

## 14. Common Mistakes

### Mistake 1: Forcing Child Methods onto Leaves

If `add()` is on the component interface, leaves may throw unsupported exceptions. Consider keeping child management only on composites.

### Mistake 2: Forgetting Cycle Protection

Composite assumes a tree. If a folder can contain its ancestor, recursion may never end.

### Mistake 3: Bloated Component Interface

Keep shared operations meaningful for both leaves and composites.

### Mistake 4: Type Checking Everywhere

If clients still check `instanceof FileNode`, the pattern is not helping enough.

### Mistake 5: Ignoring Traversal Cost

Large trees may need caching, iterators, or incremental updates.

## 15. Composite vs Similar Patterns

| Pattern | Difference |
|---|---|
| Decorator | Decorator wraps one component to add behavior. Composite groups many components. |
| Flyweight | Flyweight shares repeated state. Composite organizes objects into trees. |
| Visitor | Visitor adds operations across a composite tree. |
| Iterator | Iterator traverses a composite without exposing structure. |
| Facade | Facade simplifies subsystem access. Composite models part-whole structure. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Component | `FileSystemNode` | Shared interface. |
| Leaf | `FileNode` | Single object. |
| Composite | `FolderNode` | Contains child components. |
| Client | Demo code | Uses component uniformly. |

## 17. Quick Revision Notes

- Composite models trees.
- Leaves and composites share a component interface.
- Composites contain child components.
- Recursive operations are common.
- Avoid bloated component interfaces.
- Watch for cycles and traversal cost.

## 18. Mini Exercise

Design a Composite for a menu system.

Nodes:

- `MenuItem`
- `MenuGroup`

Operations:

- `render()`
- `enabledCount()`

Expected usage:

```java
MenuComponent menu = new MenuGroup("File");
menu.render();
```

## 19. Source Reference in This Repo

The repository's Composite implementation uses `LetterComposite`, `Letter`, `Word`, and `Sentence`.

Useful files:

- [github-repo/composite/README.md](../../github-repo/composite/README.md)
- [github-repo/composite/src/main/java/com/iluwatar/composite/LetterComposite.java](../../github-repo/composite/src/main/java/com/iluwatar/composite/LetterComposite.java)
- [github-repo/composite/src/main/java/com/iluwatar/composite/Letter.java](../../github-repo/composite/src/main/java/com/iluwatar/composite/Letter.java)
- [github-repo/composite/src/main/java/com/iluwatar/composite/Word.java](../../github-repo/composite/src/main/java/com/iluwatar/composite/Word.java)
- [github-repo/composite/src/main/java/com/iluwatar/composite/Sentence.java](../../github-repo/composite/src/main/java/com/iluwatar/composite/Sentence.java)
