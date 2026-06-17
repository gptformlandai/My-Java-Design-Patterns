# Single Table Inheritance Pattern

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/single-table-inheritance](../../github-repo/single-table-inheritance)

## How to Study This Page

Use this page in three passes:

1. First pass: understand storing an entire class hierarchy in one database table.
2. Second pass: rewrite the Java example and identify base class, subclasses, discriminator column, and nullable columns.
3. Third pass: compare Single Table Inheritance with Class Table Inheritance, Concrete Table Inheritance, and composition.

By the end, you should be able to say:

> Single Table Inheritance maps a class hierarchy into one table using a discriminator column to identify each subclass.

## 1. Technical Definition

Single Table Inheritance is a persistence mapping pattern where all classes in an inheritance hierarchy are stored in one table, with a discriminator column indicating the concrete type of each row.

Core idea:

- One table stores all subclasses.
- Common columns represent base fields.
- Subclass-specific columns are nullable for other types.
- Discriminator tells ORM which subclass to instantiate.
- Reads avoid joins but table can become sparse.

### 30-Second Interview Answer

I would use Single Table Inheritance when subclasses are similar and I want simple, fast reads without joins. The table contains common fields plus subtype-specific fields, and a discriminator column identifies the subtype. It works well for small stable hierarchies. The downside is nullable columns, weak database constraints for subtype-only fields, and a table that can become messy as subclasses diverge.

## 2. Layman and Easy to Understand Definition

Single Table Inheritance is like storing cars, trucks, and trains in one vehicle spreadsheet.

Every row has common columns like manufacturer and model. Some rows use car-only columns, some use truck-only columns, and a `vehicle_type` column tells you what each row represents.

In code:

- Base class is `Vehicle`.
- Subclasses are `Car`, `Truck`, etc.
- One table stores all of them.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Object-oriented code uses inheritance:

```java
Vehicle
  Car
  Truck
  Train
```

Relational databases do not have inheritance in the same way.

Problems:

- How do you store subclasses?
- Should each subclass have its own table?
- Should common fields be duplicated?
- How do you query all vehicles together?

### 3.2 The Single Table Solution

Use one table:

```text
vehicle
id | vehicle_type | manufacturer | model | trunk_capacity | towing_capacity
```

Rows use only columns relevant to their type.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Base class | Parent type shared by all subclasses. |
| Subclass | Concrete specialized type. |
| Single table | Stores all hierarchy rows. |
| Discriminator column | Identifies row subtype. |
| ORM mapper | Creates correct subclass from row. |

### 3.4 Table Shape

Example:

| id | type | manufacturer | trunk_capacity | towing_capacity |
|---|---|---|---|---|
| 1 | CAR | Tesla | 825 | null |
| 2 | TRUCK | Ford | null | 14000 |

This is simple but can become sparse.

## 4. Java Coding Example

This example shows a simplified JPA-style hierarchy.

```java
abstract class Vehicle {
    private final long id;
    private final String manufacturer;

    protected Vehicle(long id, String manufacturer) {
        this.id = id;
        this.manufacturer = manufacturer;
    }

    long id() {
        return id;
    }

    String manufacturer() {
        return manufacturer;
    }
}

final class Car extends Vehicle {
    private final int trunkCapacity;

    Car(long id, String manufacturer, int trunkCapacity) {
        super(id, manufacturer);
        this.trunkCapacity = trunkCapacity;
    }
}

final class Truck extends Vehicle {
    private final int towingCapacity;

    Truck(long id, String manufacturer, int towingCapacity) {
        super(id, manufacturer);
        this.towingCapacity = towingCapacity;
    }
}
```

JPA mapping idea:

```java
// @Entity
// @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
// @DiscriminatorColumn(name = "vehicle_type")
abstract class VehicleEntity {
}

// @DiscriminatorValue("CAR")
class CarEntity extends VehicleEntity {
}

// @DiscriminatorValue("TRUCK")
class TruckEntity extends VehicleEntity {
}
```

### Java Block by Block Explanation

`Vehicle` represents common state.

`Car` and `Truck` represent specialized subclasses.

The single database table stores fields for all subclasses.

The discriminator column tells the mapper which subclass to create.

### Java Usage

Use Single Table Inheritance in Java when:

- Subclasses share many fields.
- Hierarchy is small and stable.
- Querying all types together is common.
- You want to avoid joins.
- Nullable subtype columns are acceptable.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Vehicle:
    id: int
    manufacturer: str


@dataclass(frozen=True)
class Car(Vehicle):
    trunk_capacity: int


@dataclass(frozen=True)
class Truck(Vehicle):
    towing_capacity: int


def map_row(row):
    if row["vehicle_type"] == "CAR":
        return Car(row["id"], row["manufacturer"], row["trunk_capacity"])
    if row["vehicle_type"] == "TRUCK":
        return Truck(row["id"], row["manufacturer"], row["towing_capacity"])
    raise ValueError("unknown vehicle type")
```

### Python Usage

Python ORMs can model single-table inheritance through polymorphic mapping, commonly with:

- A discriminator/type column.
- Base class mapping.
- Subclass-specific nullable columns.

## 6. Where It Comes Handy in Real Life

- Payment methods with similar fields.
- Notification templates.
- Vehicle/item catalogs.
- User roles with small subtype differences.
- Product types with shared lifecycle.
- Polymorphic ORM models.

## 7. Advantages Over Normal Code Without Pattern

### Without a Mapping Strategy

```text
Inheritance in code, unclear persistence in database
```

Problems:

- Ad hoc tables.
- Duplicated columns.
- Hard polymorphic queries.
- Confusing object reconstruction.

### With Single Table Inheritance

```text
one table + discriminator
```

Benefits:

- Simple schema.
- Fast polymorphic reads.
- No joins for hierarchy.
- ORM support is straightforward.

## 8. Where It Excels

- Small inheritance hierarchies.
- Subclasses with similar data.
- Frequent queries over base type.
- ORM-managed applications.
- Read performance where joins would hurt.

## 9. Where It Fails

- Subclasses have very different fields.
- Many subtype-only non-null constraints are required.
- Table becomes sparse with many nullable columns.
- Hierarchy changes frequently.
- Composition would model variation better.

## 10. Prebuilt Frameworks and Packages

### Java

- JPA `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`.
- Hibernate discriminator columns.
- Spring Data JPA repositories over base type.

### Python

- SQLAlchemy single table inheritance.
- Django model inheritance options, depending on mapping style.
- Custom mappers with discriminator fields.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Simple one-table schema. | Many nullable columns. |
| Fast reads without joins. | Weak subtype-specific constraints. |
| Easy polymorphic queries. | Table can become wide. |
| Good ORM support. | Poor fit for divergent subclasses. |
| Simple inserts/updates. | Type changes can be awkward. |

## 12. Real-World Identification Example

Scenario:

You store multiple notification types: email, SMS, and push.

Single Table Inheritance fit:

- All have ID, recipient, status, created time.
- Email has subject/body.
- SMS has text.
- Push has device token.
- `notification_type` identifies subtype.

What would go wrong if subclasses diverge:

- Many null columns.
- Constraints become hard.
- Separate tables may be cleaner.

## 13. MAANG Interview Triggers

Use Single Table Inheritance when you hear:

- "Map class hierarchy to relational database."
- "Discriminator column."
- "Avoid joins."
- "Polymorphic query over base class."
- "Nullable subtype columns."
- "JPA inheritance strategy."

### Interview-Ready Answer Format

1. Identify base class and subclasses.
2. Define single table with common and subtype columns.
3. Add discriminator column.
4. Explain ORM reconstruction.
5. Discuss null columns and constraints.
6. Compare with class-table/concrete-table inheritance.

## 14. Common Mistakes

### Mistake 1: Using It for Divergent Subclasses

If subclasses barely share fields, one table becomes messy.

### Mistake 2: Ignoring Constraints

Subtype-specific required fields are hard to enforce in one table.

### Mistake 3: Too Many Subclasses

The table grows wide and sparse.

### Mistake 4: Confusing Discriminator With Business Status

Type discriminator identifies class, not lifecycle state.

### Mistake 5: Choosing Inheritance When Composition Fits Better

Not every variation needs subclassing.

## 15. Single Table Inheritance vs Similar Patterns

| Pattern | Difference |
|---|---|
| Single Table Inheritance | One table for whole hierarchy. |
| Class Table Inheritance | Base and subclass tables joined together. |
| Concrete Table Inheritance | One table per concrete class. |
| Data Mapper | General mapping object/row transfer. |
| Composition | Models variation through fields/components instead of inheritance. |

## 16. Single Table Inheritance Design Checklist

| Question | Why it matters |
|---|---|
| Are subclasses similar? | Validates one-table fit. |
| What is discriminator value? | Enables subtype reconstruction. |
| How many nullable columns? | Detects table sparsity. |
| Are subtype constraints needed? | May require checks/triggers. |
| Are polymorphic queries common? | Justifies strategy. |
| Could composition be better? | Avoids inheritance misuse. |

## 17. Quick Revision Notes

- One table stores entire hierarchy.
- Discriminator column identifies subtype.
- Fast polymorphic reads.
- Many nullable columns are the main cost.
- Best for similar, stable subclasses.
- Compare with class-table inheritance.

## 18. Mini Exercise

Design Single Table Inheritance for `PaymentMethod`.

Subtypes:

- `CreditCard`
- `BankAccount`
- `Wallet`

Table:

- Common columns.
- Subtype-specific columns.
- Discriminator values.

## 19. Source Reference in This Repo

The repository's Single Table Inheritance implementation maps a `Vehicle` hierarchy into one table with subclasses such as `Car` and `Truck`.

Useful files:

- [github-repo/single-table-inheritance/README.md](../../github-repo/single-table-inheritance/README.md)
- [github-repo/single-table-inheritance/src/main/java/com/iluwatar/entity/Vehicle.java](../../github-repo/single-table-inheritance/src/main/java/com/iluwatar/entity/Vehicle.java)
- [github-repo/single-table-inheritance/src/main/java/com/iluwatar/entity/Car.java](../../github-repo/single-table-inheritance/src/main/java/com/iluwatar/entity/Car.java)
- [github-repo/single-table-inheritance/src/main/java/com/iluwatar/entity/Truck.java](../../github-repo/single-table-inheritance/src/main/java/com/iluwatar/entity/Truck.java)
- [github-repo/single-table-inheritance/src/main/java/com/iluwatar/repository/VehicleRepository.java](../../github-repo/single-table-inheritance/src/main/java/com/iluwatar/repository/VehicleRepository.java)
- [github-repo/single-table-inheritance/src/main/java/com/iluwatar/SingleTableInheritance.java](../../github-repo/single-table-inheritance/src/main/java/com/iluwatar/SingleTableInheritance.java)

