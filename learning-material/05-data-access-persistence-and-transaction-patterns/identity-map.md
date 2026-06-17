# Identity Map Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/identity-map](../../github-repo/identity-map)

## How to Study This Page

Use this page in three passes:

1. First pass: understand keeping one in-memory object per database identity inside a context.
2. Second pass: rewrite the Java example and identify key, map, finder, database lookup, and cache hit.
3. Third pass: compare Identity Map with Cache, Repository, Unit of Work, and ORM session.

By the end, you should be able to say:

> Identity Map prevents duplicate in-memory objects for the same database row by reusing objects from a context-local map.

## 1. Technical Definition

Identity Map is a persistence pattern that keeps a map of objects already loaded in a transaction/session so each database identity is represented by exactly one in-memory object.

Core idea:

- Key is persistent identity.
- Map stores loaded object.
- Lookup checks map before database.
- Same identity returns same object instance.
- Scope is usually request, transaction, or ORM session.

### 30-Second Interview Answer

I would use Identity Map when repeated loads of the same record within a transaction should return the same object instance. It avoids duplicate database reads and prevents inconsistent updates to multiple copies of the same entity. ORMs often include this in their session/persistence context. The trade-off is stale data and memory growth if the map is too long-lived.

## 2. Layman and Easy to Understand Definition

Identity Map is like remembering who already checked in.

If the same person comes to the desk again, you reuse their existing record instead of creating a duplicate.

In code:

- Ask map for object by ID.
- If present, return it.
- If absent, load from database and store it.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without Identity Map:

```java
Person a = database.find(10);
Person b = database.find(10);
```

Now `a` and `b` may represent the same database row but be different objects.

Problems:

- Duplicate database reads.
- Conflicting in-memory changes.
- Object equality/identity confusion.
- Harder Unit of Work tracking.

### 3.2 The Identity Map Solution

Check memory first:

```java
Person person = identityMap.get(id);
if (person == null) {
    person = database.find(id);
    identityMap.put(id, person);
}
```

Now each ID has one object instance in the context.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Identity key | Database ID or natural key. |
| Identity map | Context-local map from key to object. |
| Finder/repository | Checks map before database. |
| Database/data source | Loads object on cache miss. |
| Unit of Work/session | Often owns the identity map scope. |

### 3.4 Scope

Identity Map should usually be scoped to:

- One request.
- One transaction.
- One Unit of Work.
- One ORM session.

It should usually not be a global forever cache.

## 4. Java Coding Example

This example keeps one `Person` instance per ID.

```java
import java.util.HashMap;
import java.util.Map;

record Person(int id, String name) {
}

final class PersonDatabase {
    private final Map<Integer, Person> rows = Map.of(
        1, new Person(1, "Ada"),
        2, new Person(2, "Grace")
    );

    Person find(int id) {
        System.out.println("Database lookup for " + id);
        return rows.get(id);
    }
}

final class IdentityMap {
    private final Map<Integer, Person> loaded = new HashMap<>();

    Person get(int id) {
        return loaded.get(id);
    }

    void put(Person person) {
        loaded.put(person.id(), person);
    }
}

final class PersonFinder {
    private final PersonDatabase database;
    private final IdentityMap identityMap;

    PersonFinder(PersonDatabase database, IdentityMap identityMap) {
        this.database = database;
        this.identityMap = identityMap;
    }

    Person find(int id) {
        Person cached = identityMap.get(id);
        if (cached != null) {
            return cached;
        }
        Person loaded = database.find(id);
        if (loaded != null) {
            identityMap.put(loaded);
        }
        return loaded;
    }
}
```

### Java Block by Block Explanation

`PersonDatabase` simulates persistent storage.

`IdentityMap` stores loaded objects by ID.

`PersonFinder` checks the map before hitting the database.

Repeated `find(1)` calls return the object from the map after first load.

### Java Usage

Use Identity Map when:

- Same entity may be loaded repeatedly in one transaction.
- Object identity matters.
- You use Unit of Work.
- Duplicate in-memory copies could cause inconsistent updates.
- You want to avoid repeated database reads inside a context.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass
class Person:
    id: int
    name: str


class PersonFinder:
    def __init__(self, database):
        self.database = database
        self.identity_map = {}

    def find(self, person_id):
        if person_id in self.identity_map:
            return self.identity_map[person_id]

        person = self.database.get(person_id)
        if person is not None:
            self.identity_map[person_id] = person
        return person
```

### Python Usage

Python Identity Map appears in:

- SQLAlchemy session identity map.
- Unit of Work implementations.
- Request-scoped persistence contexts.
- Custom repositories that cache loaded entities during a use case.

## 6. Where It Comes Handy in Real Life

- ORM sessions.
- Rich domain transactions.
- Repeated entity lookups.
- Unit of Work implementations.
- Graph loading with shared references.
- Avoiding duplicate updates to same row.
- Consistency inside request scope.

## 7. Advantages Over Normal Code Without Pattern

### Without Identity Map

```java
personA = find(1);
personB = find(1);
```

Problems:

- Two objects may represent one row.
- Updating one copy does not update the other.
- Database is queried repeatedly.

### With Identity Map

```java
personA == personB
```

Benefits:

- One object per identity.
- Fewer database reads.
- Unit of Work tracking is simpler.
- Object graph consistency improves.

## 8. Where It Excels

- Transaction-scoped persistence.
- ORM behavior.
- Repeated lookups.
- Object graphs with shared references.
- Systems using Unit of Work.
- Rich domain models.

## 9. Where It Fails

- Long-lived global caches.
- Highly volatile data where stale reads are unacceptable.
- Simple one-off reads.
- Huge scans that would fill memory.
- Distributed systems where identity is shared across many processes.

## 10. Prebuilt Frameworks and Packages

### Java

- Hibernate first-level cache/session.
- JPA persistence context.
- EclipseLink identity maps.
- Unit of Work implementations.

### Python

- SQLAlchemy Session identity map.
- Django ORM instance caching in limited query contexts.
- Custom Unit of Work maps.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Prevents duplicate objects. | Can return stale data. |
| Reduces repeated database reads. | Consumes memory. |
| Helps Unit of Work track changes. | Scope must be controlled. |
| Preserves object identity. | Not a distributed cache. |
| Improves consistency inside a transaction. | Can hide reload needs. |

## 12. Real-World Identification Example

Scenario:

An order service loads the same customer through order, payment, and shipment paths.

Identity Map fit:

- First lookup loads customer.
- Later lookups reuse same object.
- All changes apply to one in-memory instance.

What would go wrong without it:

- Multiple customer objects conflict.
- Unit of Work may miss one changed copy.

## 13. MAANG Interview Triggers

Use Identity Map when you hear:

- "Same row loaded multiple times."
- "One object per identity."
- "ORM session cache."
- "First-level cache."
- "Unit of Work."
- "Avoid duplicate in-memory entities."

### Interview-Ready Answer Format

1. Define identity key.
2. Scope the map to transaction/session.
3. Check map before database.
4. Store loaded object on miss.
5. Integrate with Unit of Work if changes are tracked.
6. Mention stale data and memory risks.

## 14. Common Mistakes

### Mistake 1: Making It Global Forever

Identity Map is usually context-scoped, not a global cache.

### Mistake 2: No Eviction/Scope Control

Long-lived maps can grow unbounded.

### Mistake 3: Ignoring Staleness

Another transaction may update the database after the object is loaded.

### Mistake 4: Confusing With Cache

Identity Map preserves identity in a context; cache optimizes reuse across contexts.

### Mistake 5: Bad Equality Assumptions

Object identity and value equality should be designed intentionally.

## 15. Identity Map vs Similar Patterns

| Pattern | Difference |
|---|---|
| Identity Map | One object per identity within a context. |
| Cache | Reuses data for performance, often across contexts. |
| Unit of Work | Tracks changes and commits them. |
| Repository | Loads/saves objects through domain API. |
| Data Mapper | Converts rows to objects and back. |

## 16. Identity Map Design Checklist

| Question | Why it matters |
|---|---|
| What is the identity key? | Defines map key. |
| What is the scope? | Prevents stale/memory issues. |
| Who owns the map? | Usually session/Unit of Work. |
| What happens on cache miss? | Loads from database. |
| How are updates tracked? | Integrates with Unit of Work. |
| Is staleness acceptable? | Determines reload policy. |

## 17. Quick Revision Notes

- Identity Map keeps one object per ID.
- Scope is usually transaction/session.
- It avoids duplicate reads and objects.
- Common in ORM first-level cache.
- Works with Unit of Work.
- Not the same as global cache.

## 18. Mini Exercise

Design Identity Map for `Product`.

Requirements:

- Key by `productId`.
- Repository checks map first.
- Database loads on miss.
- Same product ID returns same object during request.

Question:

- When should the map be cleared?

## 19. Source Reference in This Repo

The repository's Identity Map implementation uses `PersonFinder` and `IdentityMap` to avoid repeated database lookups for the same person ID.

Useful files:

- [github-repo/identity-map/README.md](../../github-repo/identity-map/README.md)
- [github-repo/identity-map/src/main/java/com/iluwatar/identitymap/Person.java](../../github-repo/identity-map/src/main/java/com/iluwatar/identitymap/Person.java)
- [github-repo/identity-map/src/main/java/com/iluwatar/identitymap/IdentityMap.java](../../github-repo/identity-map/src/main/java/com/iluwatar/identitymap/IdentityMap.java)
- [github-repo/identity-map/src/main/java/com/iluwatar/identitymap/PersonFinder.java](../../github-repo/identity-map/src/main/java/com/iluwatar/identitymap/PersonFinder.java)
- [github-repo/identity-map/src/main/java/com/iluwatar/identitymap/PersonDbSimulator.java](../../github-repo/identity-map/src/main/java/com/iluwatar/identitymap/PersonDbSimulator.java)
- [github-repo/identity-map/src/main/java/com/iluwatar/identitymap/App.java](../../github-repo/identity-map/src/main/java/com/iluwatar/identitymap/App.java)

