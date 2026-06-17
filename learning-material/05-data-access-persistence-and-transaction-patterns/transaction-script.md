# Transaction Script Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/transaction-script](../../github-repo/transaction-script)

## How to Study This Page

Use this page in three passes:

1. First pass: understand one procedure per business transaction/request.
2. Second pass: rewrite the Java example and identify script method, validation, database calls, and transaction boundary.
3. Third pass: compare Transaction Script with Domain Model, Service Layer, and Unit of Work.

By the end, you should be able to say:

> Transaction Script organizes business logic as straightforward procedures, where each procedure handles one user/system transaction.

## 1. Technical Definition

Transaction Script is an application/persistence pattern that organizes business logic into procedural scripts, each script handling a single business transaction from input validation through persistence updates.

Core idea:

- One method/procedure per use case.
- Steps are written in sequence.
- Data access calls happen directly or through DAO/repository.
- Simple business logic stays easy to read.
- Complexity grows poorly as domain rules expand.

### 30-Second Interview Answer

I would use Transaction Script for simple business workflows where each request can be handled by a clear procedural method. The script validates input, reads data, applies rules, writes changes, and returns. It is easy to understand and fast to build. But as rules become rich and shared, Transaction Script leads to duplication, and a Domain Model or Service Layer becomes better.

## 2. Layman and Easy to Understand Definition

Transaction Script is like a checklist for one task.

For "book room", follow steps: find room, check if available, mark booked, save room. For "cancel booking", follow another checklist.

In code:

- One method handles one action.
- Steps are explicit.
- Data access is called in sequence.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Not every application needs a rich domain model.

For simple apps, modeling every rule as objects can be too much:

```java
RoomBookingPolicy policy = ...
BookingAggregate aggregate = ...
```

Problems:

- Over-modeling slows simple work.
- CRUD-like flows need direct steps.
- Developers need an easy place for simple request logic.

### 3.2 The Transaction Script Solution

Write one procedure:

```java
bookRoom(roomNumber) {
    room = dao.getById(roomNumber);
    validate room exists and is available;
    room.setBooked(true);
    dao.update(room);
}
```

Simple and direct.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Script method | Procedure for one business transaction. |
| Input/request | Data needed to execute script. |
| DAO/repository | Reads/writes data. |
| Transaction boundary | Ensures script changes commit/rollback together. |
| Result/error | Response or failure from script. |

### 3.4 Typical Flow

1. Receive request.
2. Validate input.
3. Load required data.
4. Apply simple business rules.
5. Update data.
6. Commit transaction.
7. Return result.

## 4. Java Coding Example

This example books a room using a script-style method.

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

record Room(int number, boolean booked) {
    Room book() {
        if (booked) {
            throw new IllegalStateException("Room already booked");
        }
        return new Room(number, true);
    }
}

final class RoomDao {
    private final Map<Integer, Room> rooms = new HashMap<>();

    Optional<Room> getById(int number) {
        return Optional.ofNullable(rooms.get(number));
    }

    void update(Room room) {
        rooms.put(room.number(), room);
    }

    void add(Room room) {
        rooms.put(room.number(), room);
    }
}

final class HotelScripts {
    private final RoomDao roomDao;

    HotelScripts(RoomDao roomDao) {
        this.roomDao = roomDao;
    }

    void bookRoom(int roomNumber) {
        Room room = roomDao.getById(roomNumber)
            .orElseThrow(() -> new IllegalArgumentException("room does not exist"));
        roomDao.update(room.book());
    }
}
```

### Java Block by Block Explanation

`RoomDao` handles data access.

`HotelScripts.bookRoom` is the transaction script.

The method loads the room, validates it, changes it, and saves it.

In real applications, the method would usually run inside a transaction.

### Java Usage

Use Transaction Script when:

- Business logic is simple.
- Workflows are mostly independent.
- Procedures are easy to read.
- Rapid delivery matters.
- A rich Domain Model would be overkill.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Room:
    number: int
    booked: bool

    def book(self):
        if self.booked:
            raise ValueError("Room already booked")
        return Room(self.number, True)


class HotelScripts:
    def __init__(self, rooms):
        self.rooms = rooms

    def book_room(self, room_number):
        room = self.rooms.get(room_number)
        if room is None:
            raise ValueError("room does not exist")
        self.rooms[room_number] = room.book()
```

### Python Usage

Python transaction scripts often appear as:

- Service functions.
- Command handlers.
- Use-case functions for simple flows.
- Scripts around DAO/repository calls.

## 6. Where It Comes Handy in Real Life

- Simple booking workflows.
- Admin CRUD actions.
- Batch jobs.
- Data import steps.
- Straightforward payment/refund flows.
- Legacy enterprise applications.
- Small services with simple rules.

## 7. Advantages Over Normal Code Without Pattern

### Without Transaction Script Discipline

```text
logic scattered across controllers, DAOs, helpers
```

Problems:

- No clear use-case procedure.
- Duplicate validation.
- Hard to find transaction boundary.

### With Transaction Script

```java
hotel.bookRoom(roomNumber)
```

Benefits:

- One place for one transaction.
- Easy to read.
- Fast to implement.
- Good for simple domains.

## 8. Where It Excels

- Simple business rules.
- Small apps.
- CRUD-heavy systems.
- Procedural workflows.
- Teams needing direct, understandable logic.
- Low domain complexity.

## 9. Where It Fails

- Rich domains with shared rules.
- Complex state transitions.
- Many scripts duplicating the same logic.
- Large applications needing expressive domain language.
- Rules that naturally belong to entities/value objects.

## 10. Prebuilt Frameworks and Packages

### Java

- Spring service methods with `@Transactional`.
- JDBC/DAO-based applications.
- Simple command handlers.
- Batch job step methods.

### Python

- Django service functions.
- FastAPI use-case functions.
- Celery task functions.
- SQLAlchemy transaction scripts.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Simple and direct. | Logic duplication as domain grows. |
| Easy to understand. | Can become procedural spaghetti. |
| Fast to implement. | Weak domain modeling. |
| Good for simple CRUD/workflows. | Hard to reuse shared rules. |
| Clear transaction method. | Scripts can become very large. |

## 12. Real-World Identification Example

Scenario:

You are building a small hotel booking admin tool.

Transaction Script fit:

- `bookRoom(roomNumber)`
- `cancelRoomBooking(roomNumber)`
- `changeRoomPrice(roomNumber, price)`

What would go wrong if domain grows:

- Discount, loyalty, overbooking, cancellation policy, and payment rules duplicate across scripts.

## 13. MAANG Interview Triggers

Use Transaction Script when you hear:

- "Simple business transaction."
- "One procedure per request."
- "CRUD-heavy app."
- "No complex domain model needed."
- "Straight-line workflow."
- "Service method handles full operation."

### Interview-Ready Answer Format

1. Identify the transaction/use case.
2. Write a clear script method.
3. Validate input and load data.
4. Apply simple rules.
5. Persist changes through DAO/repository.
6. Mention migration to Domain Model if rules grow.

## 14. Common Mistakes

### Mistake 1: Keeping Transaction Script After Domain Gets Rich

Move to Domain Model when rules become shared and complex.

### Mistake 2: Scripts Too Large

Split or model domain concepts when one method becomes hard to reason about.

### Mistake 3: Duplicated Validation

Shared validations should be extracted carefully.

### Mistake 4: No Transaction Boundary

Script should usually commit or roll back as one unit.

### Mistake 5: Business Logic in DAO

The script owns procedural business flow; DAO owns persistence.

## 15. Transaction Script vs Similar Patterns

| Pattern | Difference |
|---|---|
| Transaction Script | Procedure per business transaction. |
| Domain Model | Objects own business behavior and invariants. |
| Service Layer | Application API; may contain transaction scripts or coordinate domain model. |
| Unit of Work | Tracks and commits changes used by scripts/services. |
| Table Module | One class per table containing business logic. |

## 16. Transaction Script Design Checklist

| Question | Why it matters |
|---|---|
| What transaction does this method handle? | Keeps scope clear. |
| Is the logic simple? | Validates pattern fit. |
| What data is loaded/written? | Defines DAO/repository use. |
| Where is transaction boundary? | Ensures consistency. |
| Are rules duplicated? | Signals need for Domain Model. |
| Is method too long? | Signals refactoring. |

## 17. Quick Revision Notes

- One script per business transaction.
- Simple, procedural, direct.
- Great for simple CRUD/workflows.
- Uses DAO/repository for persistence.
- Can become duplicated as domain grows.
- Domain Model is better for rich rules.

## 18. Mini Exercise

Design Transaction Script for `CancelOrder`.

Steps:

- Load order.
- Check cancellable status.
- Mark cancelled.
- Restore inventory.
- Save order.
- Return refund amount.

Question:

- At what point should this move to a Domain Model?

## 19. Source Reference in This Repo

The repository's Transaction Script implementation uses `Hotel` methods such as `bookRoom` and `cancelRoomBooking` to perform full business transactions.

Useful files:

- [github-repo/transaction-script/README.md](../../github-repo/transaction-script/README.md)
- [github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/Hotel.java](../../github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/Hotel.java)
- [github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/HotelDao.java](../../github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/HotelDao.java)
- [github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/HotelDaoImpl.java](../../github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/HotelDaoImpl.java)
- [github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/Room.java](../../github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/Room.java)
- [github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/App.java](../../github-repo/transaction-script/src/main/java/com/iluwatar/transactionscript/App.java)

