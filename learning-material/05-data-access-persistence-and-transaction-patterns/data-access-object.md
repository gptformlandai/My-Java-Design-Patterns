# Data Access Object Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/data-access-object](../../github-repo/data-access-object)

## How to Study This Page

Use this page in three passes:

1. First pass: understand DAO as an interface over a data source.
2. Second pass: rewrite the Java example and identify entity, DAO interface, in-memory DAO, and database DAO.
3. Third pass: compare DAO with Repository, Data Mapper, Active Record, and Service Layer.

By the end, you should be able to say:

> DAO separates database access code from business logic by exposing CRUD-style operations through an interface.

## 1. Technical Definition

Data Access Object is a persistence pattern that provides an abstract interface to a database or other storage mechanism, hiding low-level data access details from the rest of the application.

Core idea:

- DAO interface defines data operations.
- DAO implementation knows SQL/data-source details.
- Business logic calls DAO instead of database directly.
- Multiple DAO implementations can exist.
- DAO is often closer to tables/records than Repository.

### 30-Second Interview Answer

I would use DAO when I want to isolate database operations behind an interface. The DAO owns CRUD operations and SQL/JDBC/ORM interaction for a data entity. This keeps services/controllers away from persistence details and makes data access replaceable in tests. Compared with Repository, DAO is usually more data-source or table oriented, while Repository is more domain/aggregate oriented.

## 2. Layman and Easy to Understand Definition

DAO is like a data clerk.

You ask the clerk to add, update, delete, or fetch a record. You do not need to know which filing cabinet, database table, or query is used.

In code:

- Service asks DAO for data.
- DAO talks to database.
- Service never writes SQL directly.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without DAO, database code spreads everywhere:

```java
Connection connection = dataSource.getConnection();
PreparedStatement statement = connection.prepareStatement("select * from customers where id = ?");
```

Problems:

- SQL is duplicated.
- Connections and exceptions leak.
- Business code becomes hard to read.
- Tests need a database.
- Changing storage touches many classes.

### 3.2 The DAO Solution

Create a data access interface:

```java
Optional<Customer> customer = customerDao.getById(id);
```

The DAO implementation handles the database work.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Entity/DTO | Data object being stored or retrieved. |
| DAO interface | Storage operation contract. |
| DAO implementation | JDBC/ORM/API/in-memory implementation. |
| Data source | Database, file, API, or memory. |
| Service/client | Calls DAO methods. |

### 3.4 DAO Shape

Common DAO methods:

```java
getAll()
getById(id)
add(entity)
update(entity)
delete(entity)
```

DAO names are often data-centric:

```java
CustomerDao
RoomDao
ProductDao
```

## 4. Java Coding Example

This example uses a DAO for customer records.

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

record Customer(int id, String firstName, String lastName) {
}

interface CustomerDao {
    List<Customer> getAll();
    Optional<Customer> getById(int id);
    boolean add(Customer customer);
    boolean update(Customer customer);
    boolean delete(int id);
}

final class InMemoryCustomerDao implements CustomerDao {
    private final Map<Integer, Customer> customers = new HashMap<>();

    @Override
    public List<Customer> getAll() {
        return List.copyOf(customers.values());
    }

    @Override
    public Optional<Customer> getById(int id) {
        return Optional.ofNullable(customers.get(id));
    }

    @Override
    public boolean add(Customer customer) {
        return customers.putIfAbsent(customer.id(), customer) == null;
    }

    @Override
    public boolean update(Customer customer) {
        if (!customers.containsKey(customer.id())) {
            return false;
        }
        customers.put(customer.id(), customer);
        return true;
    }

    @Override
    public boolean delete(int id) {
        return customers.remove(id) != null;
    }
}
```

### Java Block by Block Explanation

`Customer` is the data object.

`CustomerDao` defines data operations.

`InMemoryCustomerDao` is one implementation. A JDBC implementation could use SQL with the same interface.

Services can depend on `CustomerDao` without caring how data is stored.

### Java Usage

Use DAO in Java when:

- You want to isolate JDBC/SQL code.
- You need multiple persistence implementations.
- You are building a layered architecture.
- Data access is mostly CRUD.
- You want easy in-memory test implementations.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Customer:
    id: int
    first_name: str
    last_name: str


class CustomerDao:
    def __init__(self):
        self._customers = {}

    def get_all(self):
        return list(self._customers.values())

    def get_by_id(self, customer_id):
        return self._customers.get(customer_id)

    def add(self, customer):
        if customer.id in self._customers:
            return False
        self._customers[customer.id] = customer
        return True

    def update(self, customer):
        if customer.id not in self._customers:
            return False
        self._customers[customer.id] = customer
        return True

    def delete(self, customer_id):
        return self._customers.pop(customer_id, None) is not None
```

### Python Usage

Python DAOs often appear as:

- Thin classes around SQL queries.
- Adapters over external APIs.
- Persistence helpers in layered apps.
- Test-friendly in-memory implementations.

## 6. Where It Comes Handy in Real Life

- Customer CRUD.
- Room booking records.
- User profile tables.
- Admin tools.
- Legacy JDBC codebases.
- Applications with multiple data sources.
- Data access layers in enterprise apps.

## 7. Advantages Over Normal Code Without Pattern

### Without DAO

```java
service -> JDBC/SQL directly
```

Problems:

- Persistence code is scattered.
- Services are harder to test.
- Connection handling leaks.
- Database changes touch business logic.

### With DAO

```java
service -> customerDao -> database
```

Benefits:

- Persistence is isolated.
- CRUD code is centralized.
- Implementations can be swapped.
- Tests can use fake DAOs.

## 8. Where It Excels

- Simple CRUD data access.
- Layered architecture.
- JDBC-heavy codebases.
- Multiple storage implementations.
- Separating SQL from business services.
- Legacy modernization.

## 9. Where It Fails

- Rich domain models where Repository is a better abstraction.
- Complex object graphs needing Data Mapper/ORM.
- Reporting queries better handled by query services.
- DAO methods that become business operations.
- Duplicate DAO + Repository layers with no clear distinction.

## 10. Prebuilt Frameworks and Packages

### Java

- JDBC templates.
- Spring JdbcTemplate.
- MyBatis mappers.
- JPA EntityManager wrapped by DAO.
- Apache DbUtils.

### Python

- sqlite3 wrappers.
- SQLAlchemy data access classes.
- Django model managers.
- Repository/DAO classes around external APIs.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Hides database details. | Can become low-level CRUD boilerplate. |
| Keeps SQL out of services. | Less domain-oriented than Repository. |
| Easy to swap implementations. | May duplicate ORM abstractions. |
| Good for layered apps. | Can grow into large data utility classes. |
| Test doubles are straightforward. | Business logic can accidentally leak in. |

## 12. Real-World Identification Example

Scenario:

You are building a hotel admin app with rooms stored in H2/Postgres.

DAO fit:

- `RoomDao.getById(roomNumber)`
- `RoomDao.update(room)`
- `RoomDao.getAll()`

What would go wrong without it:

- Booking code contains SQL.
- Schema changes affect business workflow.
- Testing requires real database setup.

## 13. MAANG Interview Triggers

Use DAO when you hear:

- "Separate database code from business logic."
- "CRUD abstraction."
- "JDBC/SQL wrapper."
- "Multiple persistence implementations."
- "Layered architecture data access layer."
- "Keep services free of SQL."

### Interview-Ready Answer Format

1. Define the data object.
2. Define DAO interface with CRUD/data-source operations.
3. Implement DAO using database technology.
4. Inject DAO into services.
5. Use fake/in-memory DAO for tests.
6. Compare with Repository for domain-oriented access.

## 14. Common Mistakes

### Mistake 1: DAO Contains Business Rules

DAO should handle persistence, not decide domain policy.

### Mistake 2: DAO Leaks Connections or ResultSets

Hide low-level database resources.

### Mistake 3: Catching and Swallowing Database Errors

Translate or propagate errors intentionally.

### Mistake 4: DAO and Repository Duplicated Blindly

Use both only when they have distinct responsibilities.

### Mistake 5: Table-Centric Design Everywhere

DAO is fine for data access, but rich domains may need aggregate-oriented repositories.

## 15. DAO vs Similar Patterns

| Pattern | Difference |
|---|---|
| DAO | Data-source/table/record oriented persistence abstraction. |
| Repository | Domain/aggregate oriented collection abstraction. |
| Data Mapper | Maps objects to database rows and back. |
| Active Record | Entity contains persistence methods. |
| Service Layer | Orchestrates business use cases and calls DAOs/repositories. |

## 16. DAO Design Checklist

| Question | Why it matters |
|---|---|
| What data object does DAO manage? | Defines scope. |
| What CRUD operations are required? | Defines interface. |
| Does DAO expose SQL resources? | Avoids leaks. |
| Are errors translated clearly? | Helps callers handle failures. |
| Can it be tested with fake storage? | Improves testability. |
| Is Repository a better fit? | Avoids table-centric domain code. |

## 17. Quick Revision Notes

- DAO abstracts data source access.
- It is usually CRUD/table oriented.
- It hides SQL/JDBC details.
- Services call DAOs.
- Repository is more domain-oriented.
- Keep business rules out of DAO.

## 18. Mini Exercise

Design DAO for `Invoice`.

Methods:

- `getAll()`
- `getById(invoiceId)`
- `add(invoice)`
- `update(invoice)`
- `delete(invoiceId)`

Implement:

- `InMemoryInvoiceDao`
- `JdbcInvoiceDao`

## 19. Source Reference in This Repo

The repository's DAO implementation uses `CustomerDao` with in-memory and database-backed implementations.

Useful files:

- [github-repo/data-access-object/README.md](../../github-repo/data-access-object/README.md)
- [github-repo/data-access-object/src/main/java/com/iluwatar/dao/Customer.java](../../github-repo/data-access-object/src/main/java/com/iluwatar/dao/Customer.java)
- [github-repo/data-access-object/src/main/java/com/iluwatar/dao/CustomerDao.java](../../github-repo/data-access-object/src/main/java/com/iluwatar/dao/CustomerDao.java)
- [github-repo/data-access-object/src/main/java/com/iluwatar/dao/InMemoryCustomerDao.java](../../github-repo/data-access-object/src/main/java/com/iluwatar/dao/InMemoryCustomerDao.java)
- [github-repo/data-access-object/src/main/java/com/iluwatar/dao/DbCustomerDao.java](../../github-repo/data-access-object/src/main/java/com/iluwatar/dao/DbCustomerDao.java)
- [github-repo/data-access-object/src/main/java/com/iluwatar/dao/App.java](../../github-repo/data-access-object/src/main/java/com/iluwatar/dao/App.java)

