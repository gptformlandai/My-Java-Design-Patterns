# Unit of Work Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/unit-of-work](../../github-repo/unit-of-work)

## How to Study This Page

Use this page in three passes:

1. First pass: understand tracking changes during a business transaction.
2. Second pass: rewrite the Java example and identify new, modified, deleted, and commit steps.
3. Third pass: compare Unit of Work with Repository, Identity Map, Transaction Script, and ORM sessions.

By the end, you should be able to say:

> Unit of Work tracks changed objects and writes all changes as one coordinated transaction.

## 1. Technical Definition

Unit of Work is a persistence pattern that maintains a list of objects affected by a business transaction and coordinates writing those changes to the data store.

Core idea:

- Track new objects.
- Track modified objects.
- Track deleted objects.
- Commit changes together.
- Roll back or discard changes on failure.

### 30-Second Interview Answer

I would use Unit of Work when a use case modifies multiple objects and those changes must be persisted consistently. The unit tracks inserts, updates, and deletes during the transaction, then commits them as one batch. ORMs like Hibernate sessions often implement this internally. The trade-off is lifecycle complexity and potential memory growth if the unit of work is too large or long-lived.

## 2. Layman and Easy to Understand Definition

Unit of Work is like writing all your shopping changes on a list before updating inventory.

Instead of updating the database after every tiny change, you track the changes and apply them together when the transaction is ready.

In code:

- Register new entity.
- Register modified entity.
- Register deleted entity.
- Commit once.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without Unit of Work, every change might hit the database immediately:

```java
orderDao.update(order);
inventoryDao.update(stock);
paymentDao.insert(payment);
```

Problems:

- Partial updates can happen.
- Too many database calls.
- Transaction boundaries are scattered.
- Concurrency handling is harder.
- Rollback logic is duplicated.

### 3.2 The Unit of Work Solution

Track changes first:

```java
unitOfWork.registerModified(order);
unitOfWork.registerModified(stock);
unitOfWork.registerNew(payment);
unitOfWork.commit();
```

The commit coordinates persistence.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Unit of Work | Tracks changes and commits them. |
| Entity | Object affected by transaction. |
| Repository/DAO | Performs actual persistence. |
| Identity Map | Often ensures one in-memory object per identity. |
| Transaction boundary | Scope of work to commit/rollback. |

### 3.4 Change States

| State | Meaning |
|---|---|
| New | Entity should be inserted. |
| Dirty/modified | Entity should be updated. |
| Removed/deleted | Entity should be deleted. |
| Clean | Entity was loaded but not changed. |

## 4. Java Coding Example

This example tracks account changes before commit.

```java
import java.util.ArrayList;
import java.util.List;

record Account(String id, int balance) {
}

interface AccountDatabase {
    void insert(Account account);
    void update(Account account);
    void delete(Account account);
}

final class AccountUnitOfWork {
    private final AccountDatabase database;
    private final List<Account> newAccounts = new ArrayList<>();
    private final List<Account> modifiedAccounts = new ArrayList<>();
    private final List<Account> deletedAccounts = new ArrayList<>();

    AccountUnitOfWork(AccountDatabase database) {
        this.database = database;
    }

    void registerNew(Account account) {
        newAccounts.add(account);
    }

    void registerModified(Account account) {
        modifiedAccounts.add(account);
    }

    void registerDeleted(Account account) {
        deletedAccounts.add(account);
    }

    void commit() {
        newAccounts.forEach(database::insert);
        modifiedAccounts.forEach(database::update);
        deletedAccounts.forEach(database::delete);
        newAccounts.clear();
        modifiedAccounts.clear();
        deletedAccounts.clear();
    }
}
```

### Java Block by Block Explanation

`AccountUnitOfWork` stores lists of pending changes.

`registerNew`, `registerModified`, and `registerDeleted` record intent.

`commit` writes changes in a controlled order.

After commit, the tracked state is cleared.

### Java Usage

Use Unit of Work in Java when:

- Multiple objects change in one use case.
- You want one commit point.
- You need batching.
- You want ORM-like change tracking.
- You need rollback/error handling around coordinated persistence.

## 5. Python Coding Example

```python
class UnitOfWork:
    def __init__(self, database):
        self.database = database
        self.new = []
        self.modified = []
        self.deleted = []

    def register_new(self, entity):
        self.new.append(entity)

    def register_modified(self, entity):
        self.modified.append(entity)

    def register_deleted(self, entity):
        self.deleted.append(entity)

    def commit(self):
        for entity in self.new:
            self.database.insert(entity)
        for entity in self.modified:
            self.database.update(entity)
        for entity in self.deleted:
            self.database.delete(entity)
        self.new.clear()
        self.modified.clear()
        self.deleted.clear()
```

### Python Usage

Python Unit of Work often appears as:

- SQLAlchemy session per request/use case.
- Explicit `with unit_of_work:` context managers.
- Repository operations committed together.
- Transaction boundary in service layer.

## 6. Where It Comes Handy in Real Life

- Checkout workflows.
- Money transfers.
- Inventory reservation.
- Batch updates.
- ORM session management.
- Multi-entity domain operations.
- Event outbox commits.
- Import jobs with transactional batches.

## 7. Advantages Over Normal Code Without Pattern

### Without Unit of Work

```java
database.update(a);
database.update(b);
database.insert(c);
```

Problems:

- Changes are written piecemeal.
- Failure can leave partial state.
- Repeated calls can be inefficient.
- Rollback is harder.

### With Unit of Work

```java
uow.registerModified(a);
uow.registerModified(b);
uow.registerNew(c);
uow.commit();
```

Benefits:

- One coordinated commit.
- Better batching.
- Clear transaction boundary.
- Easier rollback strategy.

## 8. Where It Excels

- Complex transactions.
- ORM-backed applications.
- Batching database writes.
- Maintaining consistency across multiple entities.
- Service-layer transaction boundaries.
- Systems with Identity Map.

## 9. Where It Fails

- Single-row simple CRUD.
- Long-running user sessions.
- Huge units that hold too many objects.
- Workflows crossing service boundaries without distributed transaction strategy.
- Cases where database transaction alone is enough and no object tracking is needed.

## 10. Prebuilt Frameworks and Packages

### Java

- Hibernate Session.
- JPA EntityManager.
- Spring `@Transactional`.
- MyBatis SqlSession.
- Transaction managers.

### Python

- SQLAlchemy Session.
- Django transaction atomic blocks.
- Unit of Work context manager pattern.
- Repository + Unit of Work architecture.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Coordinates multiple changes. | Adds lifecycle complexity. |
| Reduces database calls through batching. | Can grow memory if long-lived. |
| Clear commit point. | Ordering changes can be tricky. |
| Supports rollback thinking. | Concurrency conflicts still need handling. |
| Common in ORMs. | Easy to misuse across request boundaries. |

## 12. Real-World Identification Example

Scenario:

You are placing an order.

Unit of Work fit:

- Insert order.
- Update inventory.
- Insert payment record.
- Insert outbox event.
- Commit all together.

What would go wrong without it:

- Payment record may save while order fails.
- Inventory may be reserved without order.
- Retrying becomes dangerous.

## 13. MAANG Interview Triggers

Use Unit of Work when you hear:

- "Commit multiple changes together."
- "Track dirty objects."
- "Batch database writes."
- "Transaction boundary."
- "ORM session."
- "Rollback coordinated changes."
- "Identity map plus change tracking."

### Interview-Ready Answer Format

1. Define transaction/use-case scope.
2. Track new, modified, and deleted objects.
3. Use repositories/DAOs for actual writes.
4. Commit changes once.
5. Roll back/discard on failure.
6. Mention lifecycle and memory risks.

## 14. Common Mistakes

### Mistake 1: Unit of Work Lives Too Long

Keep it scoped to request/use case/transaction.

### Mistake 2: No Clear Commit Boundary

If anyone can commit anywhere, consistency becomes hard.

### Mistake 3: Ignoring Write Order

Inserts, updates, and deletes may need foreign-key-aware ordering.

### Mistake 4: Confusing With Repository

Repository loads/saves objects. Unit of Work tracks and commits changes.

### Mistake 5: No Conflict Strategy

Optimistic locking/version checks may still be needed.

## 15. Unit of Work vs Similar Patterns

| Pattern | Difference |
|---|---|
| Unit of Work | Tracks changes and commits as one transaction. |
| Repository | Provides domain-friendly access to objects. |
| DAO | Encapsulates lower-level persistence operations. |
| Identity Map | Ensures one object instance per identity in a context. |
| Transaction Script | Procedure that may use Unit of Work internally. |

## 16. Unit of Work Design Checklist

| Question | Why it matters |
|---|---|
| What is the transaction scope? | Prevents long-lived contexts. |
| What states are tracked? | Defines behavior. |
| Who calls commit? | Controls consistency. |
| How are failures handled? | Enables rollback/retry. |
| Is ordering important? | Avoids constraint errors. |
| Is Identity Map needed? | Prevents duplicate objects. |

## 17. Quick Revision Notes

- Tracks new, modified, deleted objects.
- Commits changes together.
- Common inside ORMs.
- Works well with Repository and Identity Map.
- Keep scope short.
- Watch memory and ordering.

## 18. Mini Exercise

Design Unit of Work for `BankTransfer`.

Tracked changes:

- Debit source account.
- Credit destination account.
- Insert transfer record.
- Insert audit event.

Questions:

- What happens if commit fails?
- Which operations must be idempotent?

## 19. Source Reference in This Repo

The repository's Unit of Work implementation uses `ArmsDealer` to register inserts, modifications, and deletes, then commits them to `WeaponDatabase`.

Useful files:

- [github-repo/unit-of-work/README.md](../../github-repo/unit-of-work/README.md)
- [github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/UnitOfWork.java](../../github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/UnitOfWork.java)
- [github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/ArmsDealer.java](../../github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/ArmsDealer.java)
- [github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/Weapon.java](../../github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/Weapon.java)
- [github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/WeaponDatabase.java](../../github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/WeaponDatabase.java)
- [github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/App.java](../../github-repo/unit-of-work/src/main/java/com/iluwatar/unitofwork/App.java)

