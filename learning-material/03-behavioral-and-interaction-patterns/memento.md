# Memento Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/memento](../../github-repo/memento)

## How to Study This Page

Use this page in three passes:

1. First pass: understand snapshot and restore without exposing internal fields.
2. Second pass: rewrite the Java example and identify originator, memento, and caretaker.
3. Third pass: compare Memento with Command, Prototype, event sourcing, and database snapshots.

By the end, you should be able to say:

> Memento captures an object's state so it can be restored later without exposing the object's internals.

## 1. Technical Definition

Memento is a behavioral design pattern that captures and externalizes an object's internal state so the object can be restored to that state later while preserving encapsulation.

Core idea:

- Originator creates a snapshot of its state.
- Memento stores that snapshot.
- Caretaker stores mementos but does not inspect them.
- Originator restores itself from a memento.

### 30-Second Interview Answer

I would use Memento when I need undo, rollback, or point-in-time restore without exposing an object's internal representation. The originator creates memento snapshots, a caretaker stores them, and only the originator knows how to restore from them. The trade-off is memory cost, especially when snapshots are large or frequent.

## 2. Layman and Easy to Understand Definition

Memento is like saving a game checkpoint.

You capture the current state. Later, if something goes wrong, you restore from that checkpoint.

In code:

- Game/player is the originator.
- Checkpoint is the memento.
- Save-slot manager is the caretaker.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a text editor needs undo:

```java
String oldText = editor.text;
String oldCursor = editor.cursor;
```

This exposes internal state.

Problems:

- Encapsulation breaks.
- Caretaker knows too much about internals.
- Restore logic is duplicated.
- Future internal changes break undo code.

### 3.2 The Memento Solution

Let the object create and restore its own snapshots:

```java
EditorMemento snapshot = editor.save();
editor.type("hello");
editor.restore(snapshot);
```

The caretaker stores snapshots but treats them as opaque.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Originator | Object whose state is saved/restored. |
| Memento | Snapshot object. |
| Caretaker | Stores snapshots but does not inspect internals. |
| Restore operation | Originator uses memento to return to old state. |

### 3.4 Snapshot Flow

1. Caretaker asks originator for snapshot.
2. Originator creates memento.
3. Caretaker stores memento.
4. State changes.
5. Caretaker passes memento back to originator.
6. Originator restores internal state.

## 4. Java Coding Example

This example implements undo for a text editor.

```java
import java.util.ArrayDeque;
import java.util.Deque;

final class TextEditor {
    private String text = "";
    private int cursor = 0;

    void type(String value) {
        text = text.substring(0, cursor) + value + text.substring(cursor);
        cursor += value.length();
    }

    Snapshot save() {
        return new Snapshot(text, cursor);
    }

    void restore(Snapshot snapshot) {
        this.text = snapshot.text;
        this.cursor = snapshot.cursor;
    }

    String content() {
        return text;
    }

    static final class Snapshot {
        private final String text;
        private final int cursor;

        private Snapshot(String text, int cursor) {
            this.text = text;
            this.cursor = cursor;
        }
    }
}

final class EditorHistory {
    private final Deque<TextEditor.Snapshot> undoStack = new ArrayDeque<>();

    void push(TextEditor editor) {
        undoStack.push(editor.save());
    }

    void undo(TextEditor editor) {
        if (!undoStack.isEmpty()) {
            editor.restore(undoStack.pop());
        }
    }
}

public final class MementoDemo {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        EditorHistory history = new EditorHistory();

        history.push(editor);
        editor.type("hello");

        history.push(editor);
        editor.type(" world");

        history.undo(editor);
        System.out.println(editor.content()); // hello
    }
}
```

### Java Block by Block Explanation

`TextEditor` is the originator.

`Snapshot` is the memento and is nested so internals stay controlled.

`EditorHistory` is the caretaker. It stores snapshots but cannot modify editor internals.

`restore` keeps the reconstruction logic inside the originator.

### Java Usage

Use Memento when:

- Undo/redo is required.
- You need rollback after failed operations.
- Object state is private and should remain private.
- Snapshots are manageable in size.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Snapshot:
    text: str
    cursor: int


class TextEditor:
    def __init__(self):
        self._text = ""
        self._cursor = 0

    def type(self, value):
        self._text = self._text[:self._cursor] + value + self._text[self._cursor:]
        self._cursor += len(value)

    def save(self):
        return Snapshot(self._text, self._cursor)

    def restore(self, snapshot):
        self._text = snapshot.text
        self._cursor = snapshot.cursor

    @property
    def content(self):
        return self._text


editor = TextEditor()
history = []
history.append(editor.save())
editor.type("hello")
history.append(editor.save())
editor.type(" world")
editor.restore(history.pop())
print(editor.content)
```

### Python Usage

Python can use:

- Frozen dataclasses as snapshots.
- Copies for small objects.
- Serialization for larger snapshots.
- Command + Memento for undoable operations.

## 6. Where It Comes Handy in Real Life

- Text editor undo/redo.
- Drawing app checkpoints.
- Workflow rollback.
- Form draft restore.
- Game save points.
- Transaction rollback in memory.
- Configuration version snapshots.

## 7. Advantages Over Normal Code Without Pattern

### Without Memento

```java
history.add(editor.getText());
history.add(editor.getCursor());
```

Problems:

- Exposes internal state.
- History manager knows editor internals.
- Restore logic spreads outside originator.
- Internal changes break external code.

### With Memento

```java
history.push(editor.save());
editor.restore(snapshot);
```

Benefits:

- Encapsulation is preserved.
- Snapshots are opaque to caretaker.
- Restore logic stays in originator.
- Undo history is simpler.

## 8. Where It Excels

- Undo/redo.
- Checkpoint/rollback.
- Encapsulated state restoration.
- Small to medium object snapshots.
- Workflows where failed steps need rollback.
- Time travel debugging for bounded state.

## 9. Where It Fails

- Very large objects.
- Very frequent snapshots.
- Distributed state across many services.
- Long-term audit history.
- Cases where event sourcing is required.
- Snapshots with external resource handles.

## 10. Prebuilt Libraries and Packages

### Java

- Serialization APIs can create snapshots, with caution.
- Command pattern commonly pairs with Memento for undo.
- Persistence/versioning tools can store state snapshots.
- Database transactions provide rollback at storage layer.

### Python

- `copy.deepcopy`
- `pickle` for controlled serialization scenarios.
- Dataclasses for immutable snapshots.
- ORM transaction rollback.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Preserves encapsulation. | Snapshots can consume memory. |
| Supports undo and rollback. | Snapshot lifecycle must be managed. |
| Keeps restore logic inside originator. | Deep copies may be expensive. |
| Caretaker stays simple. | External resources are hard to snapshot. |

## 12. Real-World Identification Example

Scenario:

You are building a multi-step form where users can undo edits.

Without Memento:

- The form history stores individual private fields.
- Restore code breaks whenever form internals change.

With Memento:

- Form creates snapshots.
- History stack stores snapshots.
- Form restores itself from the selected snapshot.

## 13. MAANG Interview Triggers

Use Memento when you hear:

- "Undo/redo."
- "Rollback object state."
- "Save checkpoint."
- "Restore without exposing internals."
- "State snapshot."
- "Caretaker should not inspect state."

### Interview-Ready Answer Format

1. Identify the originator state to preserve.
2. Define opaque memento snapshot.
3. Let originator create and restore snapshots.
4. Let caretaker store history.
5. Bound memory and snapshot count.
6. Mention Command for undoable actions or event sourcing for audit history.

## 14. Common Mistakes

### Mistake 1: Exposing Snapshot Internals Publicly

The caretaker should not depend on internal fields.

### Mistake 2: Unlimited History

Bound the number or size of snapshots.

### Mistake 3: Snapshotting External Resources

Files, sockets, and database connections need separate restore logic.

### Mistake 4: Using Memento for Audit Logs

Memento restores state; event sourcing explains how state changed.

### Mistake 5: Mutable Mementos

Mutable snapshots can corrupt undo history.

## 15. Memento vs Similar Patterns

| Pattern | Difference |
|---|---|
| Memento | Captures state for later restore. |
| Command | Encapsulates action; may use memento for undo. |
| Prototype | Clones objects to create new instances. |
| Event Sourcing | Stores events as source of truth. |
| Snapshot | General state capture idea; Memento preserves encapsulation. |

## 16. Memento Design Checklist

| Question | Why it matters |
|---|---|
| What state must be restored? | Defines snapshot content. |
| Is memento opaque? | Preserves encapsulation. |
| How many snapshots are kept? | Controls memory. |
| Are snapshots immutable? | Prevents history corruption. |
| Are external resources involved? | Requires special handling. |

## 17. Quick Revision Notes

- Memento is snapshot plus restore.
- Originator creates and restores mementos.
- Caretaker stores but does not inspect.
- Great for undo/rollback.
- Watch memory usage.
- Pair with Command for undoable actions.

## 18. Mini Exercise

Design Memento for `DrawingCanvas`.

State:

- Shapes.
- Selected shape.
- Zoom level.

Operations:

- `save()`
- `restore(snapshot)`
- `undo()`

## 19. Source Reference in This Repo

The repository's Memento implementation stores and restores object snapshots using `Star` and `StarMemento`.

Useful files:

- [github-repo/memento/README.md](../../github-repo/memento/README.md)
- [github-repo/memento/src/main/java/com/iluwatar/memento/Star.java](../../github-repo/memento/src/main/java/com/iluwatar/memento/Star.java)
- [github-repo/memento/src/main/java/com/iluwatar/memento/StarMemento.java](../../github-repo/memento/src/main/java/com/iluwatar/memento/StarMemento.java)
- [github-repo/memento/src/main/java/com/iluwatar/memento/StarType.java](../../github-repo/memento/src/main/java/com/iluwatar/memento/StarType.java)
- [github-repo/memento/src/main/java/com/iluwatar/memento/App.java](../../github-repo/memento/src/main/java/com/iluwatar/memento/App.java)

