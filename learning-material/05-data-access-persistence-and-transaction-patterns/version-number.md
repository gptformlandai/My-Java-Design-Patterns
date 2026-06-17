# Version Number Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/version-number](../../github-repo/version-number)

## How to Study This Page

Use this page in three passes:

1. First pass: understand version number as a change counter on persisted data.
2. Second pass: rewrite the Java example and identify read version, update version, version increment, and mismatch.
3. Third pass: compare Version Number with Optimistic Offline Lock, timestamps, ETags, and audit history.

By the end, you should be able to say:

> Version Number adds a changing version field to records so systems can detect stale updates and track state changes.

## 1. Technical Definition

Version Number is a persistence pattern where each record/entity carries a version value that changes whenever the record is updated.

Core idea:

- Entity has version field.
- Reads include version.
- Updates compare expected version.
- Successful update increments version.
- Mismatch means caller used stale data.

### 30-Second Interview Answer

I would use Version Number to detect stale writes and support optimistic locking. Each row has a version column. When a client reads version 3 and saves, the update succeeds only if the current version is still 3, then increments to 4. If the current version changed, we reject or merge. Version Number is the mechanism; Optimistic Offline Lock is the broader concurrency pattern that uses it.

## 2. Layman and Easy to Understand Definition

Version Number is like a document revision number.

If you edited revision 5 but the document is now revision 6, your copy is stale. The system stops you from overwriting newer work.

In code:

- Read object with version.
- Save object with same expected version.
- Increment version after successful save.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without versioning:

```text
Two users read same book record.
Both edit.
Last save wins.
First user's update disappears.
```

Problems:

- No stale-write detection.
- Debugging lost updates is hard.
- Audit of state freshness is weak.
- APIs cannot tell if client data is old.

### 3.2 The Version Number Solution

Add a version:

```java
Book(id=1, title="Draft", version=0)
```

Update only when expected version matches:

```java
if (incoming.version != current.version) {
    throw new VersionMismatchException();
}
current.version++;
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Version field | Counter/timestamp identifying current record version. |
| Entity | Data carrying the version. |
| Repository | Checks and increments version. |
| Client copy | Detached data with old version. |
| Conflict exception | Signals stale update. |

### 3.4 Version Types

| Type | Notes |
|---|---|
| Integer counter | Simple and common. |
| Timestamp | Useful but can suffer precision/clock issues. |
| Database rowversion | Database-generated version bytes/counter. |
| ETag | HTTP representation version. |

## 4. Java Coding Example

This example protects book updates.

```java
import java.util.HashMap;
import java.util.Map;

record Book(long id, String title, long version) {
    Book withTitle(String newTitle) {
        return new Book(id, newTitle, version);
    }

    Book nextVersion() {
        return new Book(id, title, version + 1);
    }
}

final class VersionMismatchException extends RuntimeException {
    VersionMismatchException(String message) {
        super(message);
    }
}

final class BookRepository {
    private final Map<Long, Book> books = new HashMap<>();

    void add(Book book) {
        books.put(book.id(), book);
    }

    Book get(long id) {
        Book book = books.get(id);
        return new Book(book.id(), book.title(), book.version());
    }

    void update(Book incoming) {
        Book current = books.get(incoming.id());
        if (incoming.version() != current.version()) {
            throw new VersionMismatchException("stale version");
        }
        books.put(incoming.id(), incoming.nextVersion());
    }
}
```

### Java Block by Block Explanation

`Book` contains the version field.

`get` returns a copy to simulate a detached read.

`update` checks incoming version against current version.

Successful update increments the version.

### Java Usage

Use Version Number when:

- Concurrent writes are possible.
- You need stale update detection.
- APIs need ETag-like behavior.
- Optimistic locking is used.
- You want a simple change counter.

## 5. Python Coding Example

```python
from dataclasses import dataclass, replace


@dataclass(frozen=True)
class Book:
    id: int
    title: str
    version: int


class BookRepository:
    def __init__(self):
        self.books = {}

    def add(self, book):
        self.books[book.id] = book

    def get(self, book_id):
        return replace(self.books[book_id])

    def update(self, incoming):
        current = self.books[incoming.id]
        if incoming.version != current.version:
            raise RuntimeError("stale version")
        self.books[incoming.id] = Book(incoming.id, incoming.title, incoming.version + 1)
```

### Python Usage

Python apps use Version Number with:

- SQLAlchemy version columns.
- Django custom integer version fields.
- REST ETags.
- Conditional update SQL.
- Document-store revision fields.

## 6. Where It Comes Handy in Real Life

- Content editing.
- Profile updates.
- Inventory records.
- Product catalog edits.
- Distributed APIs.
- HTTP conditional requests.
- Optimistic concurrency control.
- Offline/mobile sync.

## 7. Advantages Over Normal Code Without Pattern

### Without Version Number

```java
repository.update(book);
```

Problems:

- Stale writes overwrite current data.
- No simple conflict signal.
- Clients cannot tell freshness.

### With Version Number

```java
if (incoming.version == current.version) update;
```

Benefits:

- Detects stale updates.
- Enables optimistic locking.
- Simple to understand.
- Works with APIs and databases.

## 8. Where It Excels

- Low-conflict concurrent updates.
- Web request workflows.
- REST APIs.
- Entity versioning.
- Optimistic lock implementations.
- Systems needing simple freshness checks.

## 9. Where It Fails

- High-conflict hot rows.
- Cases requiring full audit history.
- Merging complex concurrent edits.
- Version not consistently included in writes.
- Clock-based timestamp versions with clock skew/precision problems.

## 10. Prebuilt Frameworks and Packages

### Java

- JPA `@Version`.
- Hibernate version columns.
- Spring Data JPA.
- HTTP ETags.
- Database rowversion/timestamp features.

### Python

- SQLAlchemy `version_id_col`.
- Django custom fields/signals.
- HTTP ETag libraries.
- Document database revision fields.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Simple conflict detection. | Requires version field everywhere. |
| Enables optimistic locking. | Conflicts still need handling. |
| Lightweight to implement. | Not full audit history. |
| Works well with APIs. | Hot rows can fail often. |
| Prevents silent stale writes. | Manual implementations are easy to get wrong. |

## 12. Real-World Identification Example

Scenario:

A CMS article has `version = 12`.

Version Number fit:

- Editor A saves version 12, article becomes 13.
- Editor B tries saving version 12.
- System rejects B's stale update.

What would go wrong without it:

- Editor B silently overwrites Editor A.

## 13. MAANG Interview Triggers

Use Version Number when you hear:

- "Detect stale writes."
- "Optimistic locking."
- "Version column."
- "ETag."
- "Lost update prevention."
- "Concurrent edits."
- "Increment on update."

### Interview-Ready Answer Format

1. Add version field to entity.
2. Return version on reads.
3. Require expected version on writes.
4. Compare expected vs current version.
5. Increment version after success.
6. On mismatch, reject/retry/merge.

## 14. Common Mistakes

### Mistake 1: Version Increment Without Compare

Incrementing alone does not prevent stale writes.

### Mistake 2: Client Does Not Send Version

Server cannot detect whether client edited stale data.

### Mistake 3: Using Timestamp Poorly

Low precision or clock skew can break conflict detection.

### Mistake 4: Confusing Version with Audit Log

Version detects freshness; audit log records history.

### Mistake 5: Ignoring Bulk Updates

Bulk updates must also respect or intentionally bypass version rules.

## 15. Version Number vs Similar Patterns

| Pattern | Difference |
|---|---|
| Version Number | Mechanism for detecting entity version changes. |
| Optimistic Offline Lock | Concurrency strategy often implemented with Version Number. |
| Audit Log | Records history of changes. |
| Timestamp | Possible version representation. |
| ETag | HTTP representation/version token. |

## 16. Version Number Design Checklist

| Question | Why it matters |
|---|---|
| What type is version? | Counter/timestamp/rowversion. |
| Who increments version? | Database or application. |
| Is version returned on reads? | Needed by clients. |
| Is version checked on writes? | Prevents stale updates. |
| What happens on mismatch? | Defines conflict behavior. |
| Do bulk updates respect version? | Avoids hidden overwrite bugs. |

## 17. Quick Revision Notes

- Version Number tracks record freshness.
- It increments on successful update.
- It detects stale writes.
- It is often used by Optimistic Offline Lock.
- It is not an audit log.
- Always compare before update.

## 18. Mini Exercise

Design Version Number for `Product`.

Fields:

- `id`
- `name`
- `price`
- `version`

Update rule:

- Save only if expected version matches.
- Increment version after update.
- Return conflict if stale.

## 19. Source Reference in This Repo

The repository's Version Number implementation uses `Book` and `BookRepository` to detect stale updates.

Useful files:

- [github-repo/version-number/README.md](../../github-repo/version-number/README.md)
- [github-repo/version-number/src/main/java/com/iluwatar/versionnumber/Book.java](../../github-repo/version-number/src/main/java/com/iluwatar/versionnumber/Book.java)
- [github-repo/version-number/src/main/java/com/iluwatar/versionnumber/BookRepository.java](../../github-repo/version-number/src/main/java/com/iluwatar/versionnumber/BookRepository.java)
- [github-repo/version-number/src/main/java/com/iluwatar/versionnumber/VersionMismatchException.java](../../github-repo/version-number/src/main/java/com/iluwatar/versionnumber/VersionMismatchException.java)
- [github-repo/version-number/src/main/java/com/iluwatar/versionnumber/App.java](../../github-repo/version-number/src/main/java/com/iluwatar/versionnumber/App.java)

