# Optimistic Offline Lock Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/optimistic-offline-lock](../../github-repo/optimistic-offline-lock)

## How to Study This Page

Use this page in three passes:

1. First pass: understand detecting write conflicts at commit/update time instead of locking early.
2. Second pass: rewrite the Java example and identify original version, current version, update, and conflict.
3. Third pass: compare optimistic locking with pessimistic locking, Version Number, and database transactions.

By the end, you should be able to say:

> Optimistic Offline Lock lets users work concurrently and detects conflicting updates using a version check before saving.

## 1. Technical Definition

Optimistic Offline Lock is a concurrency pattern that prevents lost updates by validating that a record has not changed since it was read before committing an update.

Core idea:

- Read data with version/timestamp.
- Work without holding long locks.
- Before update, compare original version with current version.
- If versions match, update succeeds and version advances.
- If versions differ, reject/retry/merge.

### 30-Second Interview Answer

I would use Optimistic Offline Lock when conflicts are possible but uncommon, and holding locks for long user workflows would hurt scalability. Each record carries a version. The update succeeds only if the stored version still matches the version the client read. If not, we detect a conflict and ask the caller to retry or merge. It is great for web apps but poor for high-conflict hot rows.

## 2. Layman and Easy to Understand Definition

Optimistic Offline Lock is like editing a shared document with a version stamp.

You make changes without blocking others. When you save, the system checks whether someone else saved a newer version first. If yes, your save is rejected so you do not overwrite their changes.

In code:

- Read version 3.
- Try to save version 3.
- Database is still version 3: save succeeds.
- Database is now version 4: conflict.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Lost update:

```text
Alice reads balance version 1
Bob reads balance version 1
Alice saves balance version 2
Bob saves stale balance and overwrites Alice
```

Problems:

- Last write silently wins.
- User changes disappear.
- Long locks are not practical in web apps.
- Data integrity suffers.

### 3.2 The Optimistic Offline Lock Solution

Check version before saving:

```sql
update card
set balance = ?, version = version + 1
where id = ? and version = ?
```

If zero rows update, someone else changed the row first.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Version/timestamp | Conflict detection value. |
| Original record | Data as read by client. |
| Current record | Data currently in storage. |
| Update command | Save attempt using original version. |
| Conflict handler | Retry, reject, or merge strategy. |

### 3.4 Conflict Flow

1. Client reads entity with version.
2. Client makes changes.
3. Update checks stored version.
4. If equal, update and increment version.
5. If different, raise conflict.
6. Caller reloads, merges, or retries.

## 4. Java Coding Example

This example protects account updates.

```java
import java.util.HashMap;
import java.util.Map;

record Account(long id, int balance, int version) {
    Account deposit(int amount) {
        return new Account(id, balance + amount, version);
    }
}

final class OptimisticLockException extends RuntimeException {
    OptimisticLockException(String message) {
        super(message);
    }
}

final class AccountRepository {
    private final Map<Long, Account> store = new HashMap<>();

    AccountRepository() {
        store.put(1L, new Account(1L, 100, 0));
    }

    Account find(long id) {
        Account account = store.get(id);
        return new Account(account.id(), account.balance(), account.version());
    }

    void update(Account changed) {
        Account current = store.get(changed.id());
        if (current.version() != changed.version()) {
            throw new OptimisticLockException("stale version");
        }
        store.put(changed.id(), new Account(
            changed.id(),
            changed.balance(),
            changed.version() + 1
        ));
    }
}
```

### Java Block by Block Explanation

`Account` carries a `version`.

`find` returns a copy to simulate detached/offline work.

`update` compares current stored version with the version read earlier.

If versions mismatch, the update fails instead of overwriting someone else's change.

### Java Usage

Use Optimistic Offline Lock when:

- Conflicts are rare but damaging.
- Long database locks are unacceptable.
- Users edit data across requests.
- You need lost-update protection.
- Retry/merge is acceptable.

## 5. Python Coding Example

```python
from dataclasses import dataclass, replace


@dataclass(frozen=True)
class Account:
    id: int
    balance: int
    version: int


class AccountRepository:
    def __init__(self):
        self.store = {1: Account(1, 100, 0)}

    def find(self, account_id):
        account = self.store[account_id]
        return replace(account)

    def update(self, changed):
        current = self.store[changed.id]
        if current.version != changed.version:
            raise RuntimeError("stale version")
        self.store[changed.id] = Account(changed.id, changed.balance, changed.version + 1)
```

### Python Usage

Python apps often implement optimistic locking with:

- Integer version columns.
- Timestamp columns.
- SQL `where id = ? and version = ?`.
- ORM versioning support.
- API ETags and `If-Match` headers.

## 6. Where It Comes Handy in Real Life

- Profile editing.
- Inventory updates with low conflict.
- Content management systems.
- Bank card/account updates.
- Admin screens.
- Offline/mobile sync.
- Distributed web applications.

## 7. Advantages Over Normal Code Without Pattern

### Without Optimistic Offline Lock

```java
repository.save(staleEntity);
```

Problems:

- Last write wins silently.
- Lost updates are possible.
- Users overwrite each other's work.

### With Optimistic Offline Lock

```java
update where id = ? and version = ?
```

Benefits:

- Conflicts are detected.
- No long-duration lock is held.
- System scales better for low-conflict data.
- Users can retry or merge.

## 8. Where It Excels

- Low-conflict data.
- Web/mobile workflows.
- Detached entities.
- User edits spanning multiple requests.
- High-read systems.
- Systems where conflict detection is enough.

## 9. Where It Fails

- Hot rows with frequent conflicts.
- Workflows requiring guaranteed exclusive access.
- Users cannot tolerate retries.
- Complex merge logic with many fields.
- Systems that forget to include version in update condition.

## 10. Prebuilt Frameworks and Packages

### Java

- JPA `@Version`.
- Hibernate optimistic locking.
- Spring Data JPA versioned entities.
- HTTP ETag/If-Match.
- Database row version/timestamp columns.

### Python

- SQLAlchemy `version_id_col`.
- Django custom version fields.
- HTTP ETags.
- Manual SQL conditional updates.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids long locks. | Conflicts require retry/merge. |
| Prevents silent lost updates. | Poor for high-conflict rows. |
| Scales well when conflicts are rare. | Requires version columns/checks. |
| Works across web requests. | Users may see save failures. |
| Pairs with Version Number. | Merge logic can be complex. |

## 12. Real-World Identification Example

Scenario:

Two admins edit the same product price.

Optimistic lock fit:

- Both read product version 7.
- Admin A saves, version becomes 8.
- Admin B saves version 7 and gets conflict.
- Admin B reloads and decides what to do.

What would go wrong without it:

- Admin B silently overwrites Admin A.

## 13. MAANG Interview Triggers

Use Optimistic Offline Lock when you hear:

- "Lost update problem."
- "Concurrent edits."
- "Do not hold locks across requests."
- "Version check before update."
- "Low conflict probability."
- "ETag/If-Match."

### Interview-Ready Answer Format

1. Read record with version.
2. Let user/process modify detached copy.
3. Update with `where id and version match`.
4. Increment version on success.
5. On mismatch, reject/retry/merge.
6. Mention high-conflict cases may need pessimistic lock/serialization.

## 14. Common Mistakes

### Mistake 1: Version Not Included in Update Condition

Then the conflict is not actually detected.

### Mistake 2: Auto-Retry Without Thinking

Blind retry can overwrite business intent.

### Mistake 3: Using It on Hot Counters

High-conflict data may need atomic database operations or locking.

### Mistake 4: Bad UX on Conflict

Users need clear reload/merge guidance.

### Mistake 5: Version Not Returned to Client

Client must submit the version it originally read.

## 15. Optimistic Offline Lock vs Similar Patterns

| Pattern | Difference |
|---|---|
| Optimistic Offline Lock | Detects conflicts at save time without long lock. |
| Pessimistic Lock | Locks data before editing. |
| Version Number | Mechanism often used to implement optimistic locking. |
| Unit of Work | Coordinates changes; may use optimistic checks at commit. |
| Transaction Isolation | Database-level visibility/concurrency guarantees. |

## 16. Optimistic Lock Design Checklist

| Question | Why it matters |
|---|---|
| What version field is used? | Enables conflict check. |
| Is version included in update? | Prevents lost updates. |
| What is conflict response? | Defines retry/merge UX. |
| Are conflicts rare? | Validates optimistic approach. |
| Is update idempotent? | Helps retry safely. |
| Does API expose version/ETag? | Lets clients participate. |

## 17. Quick Revision Notes

- Assumes conflicts are rare.
- Detects conflict at save time.
- Usually uses version number/timestamp.
- Prevents lost updates.
- No long locks.
- Bad fit for hot rows.

## 18. Mini Exercise

Design Optimistic Offline Lock for `Article`.

Fields:

- `id`
- `title`
- `body`
- `version`

Flow:

- Editor loads article.
- Another editor saves first.
- First editor attempts save.
- System detects conflict.

## 19. Source Reference in This Repo

The repository's Optimistic Offline Lock implementation updates a `Card` only if its version has not changed during the transaction.

Useful files:

- [github-repo/optimistic-offline-lock/README.md](../../github-repo/optimistic-offline-lock/README.md)
- [github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/model/Card.java](../../github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/model/Card.java)
- [github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/service/CardUpdateService.java](../../github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/service/CardUpdateService.java)
- [github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/repository/JpaRepository.java](../../github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/repository/JpaRepository.java)
- [github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/api/UpdateService.java](../../github-repo/optimistic-offline-lock/src/main/java/com/iluwatar/api/UpdateService.java)

