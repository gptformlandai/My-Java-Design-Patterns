# Command Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/command](../../github-repo/command)

## How to Study This Page

Use this page in three passes:

1. First pass: understand a command as a request packaged into an object.
2. Second pass: rewrite the Java example and identify command, receiver, invoker, and client.
3. Third pass: study undo, redo, queues, retries, and command history.

By the end, you should be able to say:

> Command turns an action into an object so it can be executed, queued, logged, retried, or undone later.

## 1. Technical Definition

Command is a behavioral design pattern that encapsulates a request as an object containing the information needed to perform an action.

Core idea:

- Command object represents an operation.
- Invoker triggers the command.
- Receiver performs the real work.
- Commands can be stored, queued, logged, retried, or undone.

### 30-Second Interview Answer

I would use Command when actions need to be treated as objects, such as UI buttons, job queues, undo/redo, macro recording, or retryable tasks. The invoker calls `execute()` without knowing the receiver details. For undo, the command must also store enough previous state to reverse the operation. The trade-off is extra classes or objects for each action.

## 2. Layman and Easy to Understand Definition

Command is like a restaurant order ticket.

The customer request is written down as a ticket. The kitchen can execute it later, queue it, reorder it, or track it.

In code:

- The ticket is the command.
- The waiter or system is the invoker.
- The kitchen is the receiver.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a button directly calls editor logic:

```java
button.onClick(() -> editor.boldSelection());
```

This is fine at first, but problems appear when you need:

- Undo.
- Redo.
- Keyboard shortcuts.
- Macro recording.
- Queuing actions.
- Logging actions.

### 3.2 The Command Solution

Wrap the action:

```java
Command command = new BoldCommand(editor);
command.execute();
command.undo();
```

The action becomes a first-class object.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Command | Interface with `execute()` and optional `undo()`. |
| Concrete command | Stores receiver and action data. |
| Receiver | Object that performs real work. |
| Invoker | Calls command without knowing receiver details. |
| Client | Creates commands and wires receiver/invoker. |

### 3.4 Mental Model

Think of Command as action packaging.

1. Client creates a command.
2. Command stores receiver and parameters.
3. Invoker stores or executes command.
4. Receiver does the real work.
5. Command may store history for undo.

## 4. Java Coding Example

This example uses commands for a text editor.

```java
public interface Command {
    void execute();
    void undo();
}

public final class TextEditor {
    private final StringBuilder text = new StringBuilder();

    public void append(String value) {
        text.append(value);
    }

    public void deleteLast(int count) {
        text.delete(text.length() - count, text.length());
    }

    public String content() {
        return text.toString();
    }
}

public final class AppendTextCommand implements Command {
    private final TextEditor editor;
    private final String value;

    public AppendTextCommand(TextEditor editor, String value) {
        this.editor = editor;
        this.value = value;
    }

    @Override
    public void execute() {
        editor.append(value);
    }

    @Override
    public void undo() {
        editor.deleteLast(value.length());
    }
}
```

### Java Block by Block Explanation

#### Command Interface

```java
public interface Command {
    void execute();
    void undo();
}
```

The invoker can execute and undo any command through this interface.

#### Receiver

```java
public final class TextEditor {
```

The receiver owns the real business operation.

#### Concrete Command

```java
public final class AppendTextCommand implements Command {
```

The command stores receiver and parameters.

#### Undo Logic

```java
editor.deleteLast(value.length());
```

Undo requires enough information to reverse the action.

### Java Usage

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class Demo {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        Deque<Command> history = new ArrayDeque<>();

        Command command = new AppendTextCommand(editor, "hello");
        command.execute();
        history.push(command);

        history.pop().undo();
        System.out.println(editor.content());
    }
}
```

## 5. Python Coding Example

```python
from dataclasses import dataclass
from typing import Protocol


class Command(Protocol):
    def execute(self) -> None:
        ...

    def undo(self) -> None:
        ...


class TextEditor:
    def __init__(self) -> None:
        self._text: list[str] = []

    def append(self, value: str) -> None:
        self._text.append(value)

    def delete_last(self) -> None:
        self._text.pop()

    def content(self) -> str:
        return "".join(self._text)


@dataclass
class AppendTextCommand:
    editor: TextEditor
    value: str

    def execute(self) -> None:
        self.editor.append(self.value)

    def undo(self) -> None:
        self.editor.delete_last()
```

### Python Usage

```python
editor = TextEditor()
history: list[Command] = []
command = AppendTextCommand(editor, "hello")
command.execute()
history.append(command)
history.pop().undo()
print(editor.content())
```

## 6. Where It Comes Handy in Real Life

Examples:

- UI buttons and menu actions.
- Undo/redo stacks.
- Job queues.
- Macro recording.
- Transaction logs.
- Retryable operations.
- Task schedulers.
- CLI command dispatch.

## 7. Advantages Over Normal Code Without Pattern

### Without Command

```java
button.onClick(() -> editor.append("hello"));
```

Problems:

- Harder to undo.
- Harder to queue.
- Harder to log or retry.
- Invoker may know receiver details.

### With Command

```java
Command command = new AppendTextCommand(editor, "hello");
command.execute();
```

Benefits:

- Action is an object.
- Invoker is decoupled from receiver.
- Commands can be stored and replayed.
- Undo/redo becomes possible.

## 8. Where It Excels

Command excels when:

- Actions need history.
- Actions need queueing.
- Actions need retry or scheduling.
- UI actions map to domain operations.
- Undo/redo is needed.
- Requests should be logged or replayed.

## 9. Where It Fails

Command is a poor fit when:

- The action is simple and immediate.
- No queue/history/undo/retry is needed.
- Commands become tiny one-line wrappers everywhere.
- Undo cannot be defined safely.
- A function callback is enough.

## 10. Prebuilt Libraries and Packages

### Java

Examples:

- `Runnable`
- `Callable`
- Swing `Action`
- Job queue tasks
- Command handlers in CQRS-style systems

### Python

Examples:

- Callables.
- Task queue jobs.
- Click/Typer command handlers.
- Undo command objects in editors.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples invoker from receiver. | Adds objects/classes. |
| Enables undo/redo. | Undo can be hard. |
| Supports queues and scheduling. | Too many tiny commands can clutter code. |
| Commands can be logged or replayed. | Captured state can become stale. |
| Works well for UI actions. | Error handling must be designed. |

## 12. Real-World Identification Example

Scenario:

You are building a document editor.

Actions:

- Insert text
- Delete text
- Format selection
- Move paragraph

Should you use Command?

Yes, if actions need undo, redo, history, or macro recording.

## 13. MAANG Interview Triggers

Think Command when you hear:

- Encapsulate request as object.
- Undo/redo.
- Queue actions.
- Retry operation.
- Macro recording.
- Button action.
- Job scheduling.

### Interview-Ready Answer Format

1. Identify the action.
2. Define command interface.
3. Put receiver and parameters in concrete command.
4. Let invoker execute commands.
5. Store history if undo/redo is needed.
6. Mention trade-offs: extra classes and undo complexity.

## 14. Common Mistakes

### Mistake 1: Command Without a Need

If no history, queue, delay, or decoupling is needed, a direct method call may be clearer.

### Mistake 2: Weak Undo Design

Undo requires previous state or inverse operations. Guessing is unsafe.

### Mistake 3: Invoker Knows Too Much

Invoker should call `execute()`, not inspect command internals.

### Mistake 4: Commands Holding Stale Data

Long-lived commands may point to outdated state.

### Mistake 5: No Failure Policy

Decide what happens when command execution fails after partial work.

## 15. Command vs Similar Patterns

| Pattern | Difference |
|---|---|
| Strategy | Strategy encapsulates an algorithm. Command encapsulates an action/request. |
| Chain of Responsibility | Chain routes a request. Command packages the request. |
| Memento | Memento stores state for undo. Command can use Memento to undo. |
| Observer | Observer reacts to events. Command represents executable work. |
| Template Method | Template Method fixes algorithm skeleton. Command packages operation execution. |

## 16. Command Design Checklist

| Concern | Why it matters |
|---|---|
| Receiver | Who performs the work? |
| Parameters | What data must command capture? |
| Undo | Is inverse operation possible? |
| Idempotency | Can retries run safely? |
| Persistence | Can command be logged or replayed? |

## 17. Quick Revision Notes

- Command turns action into object.
- Invoker calls command.
- Receiver does real work.
- Supports undo, queues, retries, history.
- `Runnable` is a simple command-like interface.
- Avoid overusing it for trivial calls.

## 18. Mini Exercise

Design commands for a task board.

Commands:

- `CreateTaskCommand`
- `MoveTaskCommand`
- `AssignTaskCommand`

Rules:

- Commands can execute.
- Move and assign can undo.
- Commands should be logged.

## 19. Source Reference in This Repo

The repository's Command implementation uses a `Wizard` invoker and command-like `Runnable` actions over a target object.

Useful files:

- [github-repo/command/README.md](../../github-repo/command/README.md)
- [github-repo/command/src/main/java/com/iluwatar/command/Wizard.java](../../github-repo/command/src/main/java/com/iluwatar/command/Wizard.java)
- [github-repo/command/src/main/java/com/iluwatar/command/Target.java](../../github-repo/command/src/main/java/com/iluwatar/command/Target.java)
- [github-repo/command/src/main/java/com/iluwatar/command/App.java](../../github-repo/command/src/main/java/com/iluwatar/command/App.java)
