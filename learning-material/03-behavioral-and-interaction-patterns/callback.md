# Callback Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/callback](../../github-repo/callback)

## How to Study This Page

Use this page in three passes:

1. First pass: understand a callback as code passed into another component to be called later.
2. Second pass: rewrite the Java example and identify task, callback, execution point, and completion point.
3. Third pass: compare callbacks with Observer, Promise/Future, and Command.

By the end, you should be able to say:

> Callback lets a caller give another component a function to run when work finishes or an event happens.

## 1. Technical Definition

Callback is a behavioral technique where executable behavior is passed as an argument and invoked by the receiver at a defined moment.

Core idea:

- Caller provides behavior.
- Callee performs work.
- Callee invokes the callback later.
- The callback usually represents completion, notification, or event handling.

### 30-Second Interview Answer

I would use Callback when one component starts work but wants another component to be notified later without tight coupling. The task accepts a callback and invokes it when the operation completes or when an event occurs. It is common in UI events, async APIs, schedulers, and framework hooks. The main trade-off is that control flow can become harder to follow, especially with nested callbacks and weak error handling.

## 2. Layman and Easy to Understand Definition

Callback is like leaving your phone number at a service desk.

You do not wait in line the whole time. You leave contact information, and they call you when your request is ready.

In code:

- The phone number is the callback.
- The service desk is the task executor.
- The phone call is callback execution.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a method does work and the caller wants to react after it finishes:

```java
task.execute();
sendNotification();
```

This is simple, but it couples the caller to the exact execution flow.

Problems appear when:

- Work is asynchronous.
- The task belongs to a framework.
- The task should not know caller details.
- Different callers need different follow-up behavior.
- Completion behavior should be pluggable.

### 3.2 The Callback Solution

Pass the follow-up behavior into the task:

```java
task.executeWith(() -> sendNotification());
```

The task decides when to call it.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Callback | Function or object containing behavior to run later. |
| Task | Component that does work and invokes callback. |
| Caller | Supplies the callback. |
| Event/completion point | Moment when callback is executed. |

### 3.4 Control Flow

1. Caller creates a callback.
2. Caller passes callback to task.
3. Task performs its work.
4. Task invokes callback.
5. Callback performs caller-specific follow-up logic.

## 4. Java Coding Example

This example executes a file upload and calls back after completion.

```java
public interface UploadCallback {
    void onSuccess(String fileId);
    void onFailure(Exception error);
}

public final class FileUploader {
    public void upload(String fileName, UploadCallback callback) {
        try {
            if (fileName == null || fileName.isBlank()) {
                throw new IllegalArgumentException("fileName is required");
            }

            String fileId = "file-" + fileName.hashCode();
            callback.onSuccess(fileId);
        } catch (Exception error) {
            callback.onFailure(error);
        }
    }
}

public final class CallbackDemo {
    public static void main(String[] args) {
        FileUploader uploader = new FileUploader();

        uploader.upload("report.pdf", new UploadCallback() {
            @Override
            public void onSuccess(String fileId) {
                System.out.println("Uploaded as " + fileId);
            }

            @Override
            public void onFailure(Exception error) {
                System.out.println("Upload failed: " + error.getMessage());
            }
        });
    }
}
```

### Java Block by Block Explanation

`UploadCallback` defines the hooks the task can call.

`FileUploader` owns the upload flow and invokes either `onSuccess` or `onFailure`.

`CallbackDemo` provides caller-specific behavior without changing `FileUploader`.

Important detail:

- Good callbacks include success and failure paths.
- The caller should know whether the callback is sync or async.
- The task should document whether callback can be invoked more than once.

### Java Usage

Use callbacks in Java when:

- You need event handlers.
- You are building framework extension points.
- You want completion notifications.
- You need simple asynchronous hooks.

Modern Java often represents simple callbacks with functional interfaces:

```java
uploader.upload("report.pdf", new LoggingUploadCallback());
```

## 5. Python Coding Example

```python
def upload(file_name, on_success, on_failure):
    try:
        if not file_name:
            raise ValueError("file_name is required")

        file_id = f"file-{hash(file_name)}"
        on_success(file_id)
    except Exception as error:
        on_failure(error)


def success(file_id):
    print(f"Uploaded as {file_id}")


def failure(error):
    print(f"Upload failed: {error}")


upload("report.pdf", success, failure)
```

### Python Usage

Python callbacks are usually functions, lambdas, or callable objects.

They are common in:

- Event loops.
- GUI handlers.
- Sorting/key functions.
- Async completion hooks.
- Framework extension points.

## 6. Where It Comes Handy in Real Life

- UI button click handlers.
- Background job completion hooks.
- Network request success/failure handlers.
- Scheduler completion actions.
- Framework lifecycle hooks.
- Stream processing event handlers.
- Testing utilities that notify after completion.

## 7. Advantages Over Normal Code Without Pattern

### Without Callback

```java
task.execute();
specificFollowUp();
```

Problems:

- The follow-up is fixed.
- The task cannot easily be reused by different callers.
- Async completion is hard to model.
- Error handling is often scattered.

### With Callback

```java
task.executeWith(callback);
```

Benefits:

- Caller controls follow-up behavior.
- Task stays reusable.
- Completion logic is explicit.
- Frameworks can call user code at the right time.

## 8. Where It Excels

- Event-driven applications.
- Small async workflows.
- Framework hooks.
- UI programming.
- Code that needs lightweight inversion of control.
- Situations where Observer would be too heavy.

## 9. Where It Fails

- Deeply nested async flows.
- Complex error propagation.
- Multi-step workflows with cancellation and retries.
- Cases where callback execution order is unclear.
- Scenarios where a Future, Promise, stream, or message queue would be clearer.

## 10. Prebuilt Libraries and Packages

### Java

- `Runnable`
- `Callable`
- `Consumer`
- `Function`
- `CompletableFuture` callbacks such as `thenApply`, `thenAccept`, and `exceptionally`
- GUI event listeners
- `CyclicBarrier` barrier action

### Python

- Callable functions.
- `asyncio` callbacks.
- GUI event handlers.
- `concurrent.futures.Future.add_done_callback`
- Sorting `key` callbacks.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples task from follow-up behavior. | Can make control flow hard to trace. |
| Lightweight and easy to pass around. | Nested callbacks become hard to maintain. |
| Useful for events and async completion. | Error handling can become inconsistent. |
| Avoids forcing task to know caller details. | Callback timing must be documented carefully. |

## 12. Real-World Identification Example

Scenario:

You upload a video and want to notify the user after transcoding completes.

Without Callback:

- Upload service must directly know notification logic.
- Different clients require conditionals.

With Callback:

- Upload service accepts an `onComplete` callback.
- Web client sends email.
- Admin tool writes an audit log.
- Test code records completion in memory.

## 13. MAANG Interview Triggers

Use Callback when you hear:

- "Run this after completion."
- "Notify caller when async work is done."
- "Framework should call user code."
- "Button click handler."
- "Event hook."
- "Avoid tight coupling between task and follow-up behavior."

### Interview-Ready Answer Format

1. Identify the async/event boundary.
2. Define callback interface with success and failure paths.
3. Pass callback into the task or registration API.
4. Invoke callback at the documented completion point.
5. Discuss error propagation, ordering, and cancellation.
6. Mention Future/Promise or Observer if callbacks become complex.

## 14. Common Mistakes

### Mistake 1: Only Success Callback

Always define how failures are reported.

### Mistake 2: Hidden Synchronous Execution

Callers must know whether callback runs immediately, later, or on another thread.

### Mistake 3: Callback Hell

Nested callbacks make workflows unreadable. Use Future/Promise, async/await, or orchestration.

### Mistake 4: Retaining Callbacks Forever

Callbacks can hold references and cause leaks if never removed.

### Mistake 5: Swallowing Exceptions

If callback throws, the task must define whether it logs, propagates, retries, or fails.

## 15. Callback vs Similar Patterns

| Pattern | Difference |
|---|---|
| Callback | One behavior passed for later execution. |
| Observer | Many subscribers react to subject changes. |
| Command | Action object often supports queueing, history, and undo. |
| Promise/Future | Represents eventual result and composes async stages. |
| Strategy | Chooses an algorithm, not just a completion hook. |

## 16. Callback Design Checklist

| Question | Why it matters |
|---|---|
| Is callback sync or async? | Affects threading and caller expectations. |
| Can it run more than once? | Prevents duplicate side effects. |
| Is failure reported? | Avoids silent errors. |
| Can it be cancelled? | Important for long-running work. |
| Who owns callback lifetime? | Prevents memory leaks. |

## 17. Quick Revision Notes

- Callback is executable behavior passed into another component.
- It is called later by the receiver.
- Great for events and completion.
- Always design success and failure paths.
- Avoid deep callback nesting.
- Future/Promise is often cleaner for complex async flows.

## 18. Mini Exercise

Design Callback for `PaymentProcessor`.

Callbacks:

- `onApproved(transactionId)`
- `onDeclined(reason)`
- `onError(error)`

Think through:

- Can the callback be called twice?
- What thread executes it?
- How is timeout handled?

## 19. Source Reference in This Repo

The repository's Callback implementation uses `Callback`, `Task`, and `SimpleTask` to execute a completion action after task execution.

Useful files:

- [github-repo/callback/README.md](../../github-repo/callback/README.md)
- [github-repo/callback/src/main/java/com/iluwatar/callback/Callback.java](../../github-repo/callback/src/main/java/com/iluwatar/callback/Callback.java)
- [github-repo/callback/src/main/java/com/iluwatar/callback/Task.java](../../github-repo/callback/src/main/java/com/iluwatar/callback/Task.java)
- [github-repo/callback/src/main/java/com/iluwatar/callback/SimpleTask.java](../../github-repo/callback/src/main/java/com/iluwatar/callback/SimpleTask.java)
- [github-repo/callback/src/main/java/com/iluwatar/callback/App.java](../../github-repo/callback/src/main/java/com/iluwatar/callback/App.java)

