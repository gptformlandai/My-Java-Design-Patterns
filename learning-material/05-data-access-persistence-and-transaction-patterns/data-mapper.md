# Data Mapper Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/data-mapper](../../github-repo/data-mapper)

## How to Study This Page

Use this page in three passes:

1. First pass: understand separating domain objects from database rows.
2. Second pass: rewrite the Java example and identify domain object, mapper interface, mapper implementation, and storage.
3. Third pass: compare Data Mapper with DAO, Repository, Active Record, and ORM.

By the end, you should be able to say:

> Data Mapper moves data between domain objects and persistence storage while keeping both sides independent.

## 1. Technical Definition

Data Mapper is a persistence pattern that transfers data between in-memory objects and a database while keeping domain objects and database schema independent.

Core idea:

- Domain objects do not know SQL.
- Database rows do not shape every domain decision.
- Mapper handles conversion both ways.
- Persistence logic is outside the entity.
- Complex mappings can evolve separately from business behavior.

### 30-Second Interview Answer

I would use Data Mapper when I want domain objects free of persistence logic. The mapper reads rows and creates objects, and takes objects and writes rows. This is common in ORMs such as Hibernate or SQLAlchemy. The benefit is clean separation between domain and storage; the trade-off is mapping complexity and possible performance overhead.

## 2. Layman and Easy to Understand Definition

Data Mapper is like a translator between two languages.

The domain speaks in objects. The database speaks in tables and rows. The mapper translates both directions so neither side has to speak the other's language directly.

In code:

- Domain object has business meaning.
- Database row has storage shape.
- Mapper converts between them.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

If domain objects save themselves directly:

```java
student.save();
```

the object now knows persistence details.

Problems:

- Domain objects depend on database APIs.
- Testing domain logic requires persistence setup.
- Schema changes leak into business classes.
- Persistence and business responsibilities mix.

### 3.2 The Data Mapper Solution

Move persistence mapping outside:

```java
studentMapper.insert(student);
Optional<Student> found = studentMapper.find(studentId);
```

The mapper owns database transfer.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Domain object | In-memory business object. |
| Data mapper interface | Mapping/data transfer contract. |
| Data mapper implementation | Reads/writes storage and creates objects. |
| Data source | Database or other persistence store. |
| Service/repository | Uses mapper to persist domain objects. |

### 3.4 Mapping Direction

Object to row:

```text
Student -> insert/update SQL values
```

Row to object:

```text
ResultSet/record -> Student
```

## 4. Java Coding Example

This example maps `Student` objects to an in-memory table.

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

record Student(int id, String name, char grade) {
}

interface StudentDataMapper {
    void insert(Student student);
    void update(Student student);
    void delete(Student student);
    Optional<Student> find(int id);
}

final class InMemoryStudentDataMapper implements StudentDataMapper {
    private final Map<Integer, Map<String, Object>> table = new HashMap<>();

    @Override
    public void insert(Student student) {
        table.put(student.id(), toRow(student));
    }

    @Override
    public void update(Student student) {
        table.put(student.id(), toRow(student));
    }

    @Override
    public void delete(Student student) {
        table.remove(student.id());
    }

    @Override
    public Optional<Student> find(int id) {
        return Optional.ofNullable(table.get(id)).map(this::toStudent);
    }

    private Map<String, Object> toRow(Student student) {
        return Map.of("id", student.id(), "name", student.name(), "grade", student.grade());
    }

    private Student toStudent(Map<String, Object> row) {
        return new Student((int) row.get("id"), (String) row.get("name"), (char) row.get("grade"));
    }
}
```

### Java Block by Block Explanation

`Student` is persistence-ignorant.

`StudentDataMapper` defines data transfer operations.

`InMemoryStudentDataMapper` maps `Student` to row-like maps and back.

In real code, the row mapping would use JDBC, JPA internals, or another storage API.

### Java Usage

Use Data Mapper when:

- Domain objects should not know persistence.
- Mapping between objects and tables is non-trivial.
- You want ORM-style separation.
- Schema and domain model evolve independently.
- Rich domain model matters.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Student:
    id: int
    name: str
    grade: str


class StudentDataMapper:
    def __init__(self):
        self.table = {}

    def insert(self, student):
        self.table[student.id] = self._to_row(student)

    def update(self, student):
        self.table[student.id] = self._to_row(student)

    def delete(self, student):
        self.table.pop(student.id, None)

    def find(self, student_id):
        row = self.table.get(student_id)
        return self._to_student(row) if row else None

    def _to_row(self, student):
        return {"id": student.id, "name": student.name, "grade": student.grade}

    def _to_student(self, row):
        return Student(row["id"], row["name"], row["grade"])
```

### Python Usage

Python Data Mapper appears in:

- SQLAlchemy classical/declarative mapping.
- Custom mappers over database rows.
- DTO-to-domain conversion layers.
- Persistence adapters in Clean Architecture.

## 6. Where It Comes Handy in Real Life

- ORM internals.
- Rich domain models.
- Legacy database schemas.
- Complex joins converted to objects.
- Clean/hexagonal persistence adapters.
- Applications avoiding Active Record.
- Systems with separate read/write models.

## 7. Advantages Over Normal Code Without Pattern

### Without Data Mapper

```java
student.setConnection(connection);
student.save();
```

Problems:

- Domain object knows database.
- Business logic and persistence mix.
- Tests need database setup.
- Schema changes affect domain code.

### With Data Mapper

```java
studentMapper.insert(student);
```

Benefits:

- Domain stays clean.
- Mapping is centralized.
- Database can change independently.
- Rich domain behavior is easier to preserve.

## 8. Where It Excels

- Complex domain models.
- Complex database schemas.
- ORM-backed applications.
- Enterprise systems.
- Persistence-ignorant domain objects.
- Applications requiring clear separation of concerns.

## 9. Where It Fails

- Simple CRUD apps where Active Record is enough.
- Very performance-sensitive paths with heavy mapping overhead.
- Small scripts where mapping abstraction is unnecessary.
- Teams unfamiliar with ORM/session lifecycle.
- Cases where mapper becomes a giant god class.

## 10. Prebuilt Frameworks and Packages

### Java

- Hibernate/JPA.
- MyBatis mappers.
- MapStruct for DTO mappings.
- jOOQ record mapping.
- Spring JDBC RowMapper.

### Python

- SQLAlchemy ORM.
- Django ORM, though closer to Active Record.
- Pydantic/dataclass mappers.
- Marshmallow schemas.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps domain persistence-ignorant. | Mapping code can be complex. |
| Separates schema from object model. | Performance overhead is possible. |
| Supports rich domain models. | Requires lifecycle discipline. |
| Centralizes object/row conversion. | Debugging ORM behavior can be hard. |
| Helps with legacy schemas. | Overkill for simple CRUD. |

## 12. Real-World Identification Example

Scenario:

You have a legacy customer table but want a clean `Customer` domain object.

Data Mapper fit:

- Mapper reads old column names.
- Mapper builds clean domain object.
- Domain object does not know legacy schema.

What would go wrong without it:

- Legacy column names leak everywhere.
- Domain rules become tied to database shape.

## 13. MAANG Interview Triggers

Use Data Mapper when you hear:

- "Keep domain objects persistence ignorant."
- "Map rows to objects."
- "ORM behavior."
- "Legacy schema."
- "Separate object model from database model."
- "Avoid Active Record."

### Interview-Ready Answer Format

1. Identify domain object and storage shape.
2. Define mapper operations.
3. Convert row-to-object and object-to-row.
4. Keep persistence APIs out of domain object.
5. Mention ORM and Unit of Work interactions.
6. Discuss mapping complexity/performance trade-off.

## 14. Common Mistakes

### Mistake 1: Domain Object Still Knows Persistence

That defeats the purpose of Data Mapper.

### Mistake 2: Mapper Owns Business Rules

Mapper should translate data, not decide business policy.

### Mistake 3: Mapping Every Layer to Every Other Layer

Only map at meaningful boundaries.

### Mistake 4: Ignoring Identity

Use Identity Map/Unit of Work when object identity matters.

### Mistake 5: Overusing for Simple CRUD

Sometimes Active Record or framework repositories are simpler.

## 15. Data Mapper vs Similar Patterns

| Pattern | Difference |
|---|---|
| Data Mapper | Transfers data between objects and persistence. |
| DAO | Encapsulates data access operations. |
| Repository | Domain-facing collection abstraction. |
| Active Record | Entity contains persistence methods. |
| Unit of Work | Tracks object changes and commits them. |

## 16. Data Mapper Design Checklist

| Question | Why it matters |
|---|---|
| What object is mapped? | Defines target domain model. |
| What is storage shape? | Defines row/document mapping. |
| Is identity preserved? | Prevents duplicates. |
| Who owns transactions? | Usually Unit of Work/session. |
| Does mapper contain rules? | Avoids responsibility leak. |
| Is mapping worth the complexity? | Avoids over-engineering. |

## 17. Quick Revision Notes

- Data Mapper maps objects to rows and rows to objects.
- Domain objects stay persistence ignorant.
- Common in ORMs.
- Works with Repository and Unit of Work.
- Avoid business logic in mappers.
- Mapping complexity is the main cost.

## 18. Mini Exercise

Design Data Mapper for `Employee`.

Domain object:

- `employeeId`
- `fullName`
- `department`

Database columns:

- `emp_id`
- `first_name`
- `last_name`
- `dept_code`

Task:

- Map row to object.
- Map object back to update statement/row.

## 19. Source Reference in This Repo

The repository's Data Mapper implementation uses `Student`, `StudentDataMapper`, and `StudentDataMapperImpl`.

Useful files:

- [github-repo/data-mapper/README.md](../../github-repo/data-mapper/README.md)
- [github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/Student.java](../../github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/Student.java)
- [github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/StudentDataMapper.java](../../github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/StudentDataMapper.java)
- [github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/StudentDataMapperImpl.java](../../github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/StudentDataMapperImpl.java)
- [github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/App.java](../../github-repo/data-mapper/src/main/java/com/iluwatar/datamapper/App.java)

