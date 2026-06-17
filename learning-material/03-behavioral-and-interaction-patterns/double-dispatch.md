# Double Dispatch Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/double-dispatch](../../github-repo/double-dispatch)

## How to Study This Page

Use this page in three passes:

1. First pass: understand behavior selected by the runtime types of two objects.
2. Second pass: rewrite the Java example and identify first dispatch and second dispatch.
3. Third pass: compare Double Dispatch with Visitor, Strategy, and type-checking conditionals.

By the end, you should be able to say:

> Double Dispatch chooses behavior from the combination of two runtime object types, not just one receiver type.

## 1. Technical Definition

Double Dispatch is a behavioral technique where a method call is resolved using the runtime type of the receiver and the runtime type of an argument.

Core idea:

- First dispatch chooses behavior based on receiver type.
- Second dispatch chooses behavior based on argument type.
- The combination determines the final operation.
- It replaces many explicit `instanceof` or type-switch checks.

### 30-Second Interview Answer

I would use Double Dispatch when interactions depend on two object types, such as collision handling, permissions between actor/resource types, or document export operations. In Java, one virtual call dispatches on the first object, and that method calls back into the second object with a more specific method. It removes central type-checking tables, but it can become verbose when many types interact.

## 2. Layman and Easy to Understand Definition

Double Dispatch is like deciding a rule from two sides of an interaction.

For example, "what happens when object A meets object B?" depends on both A and B, not just A.

In code:

- First call asks A to interact with B.
- A calls the matching method on B.
- B now knows the exact type of A and handles the specific pair.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Single dispatch only uses the runtime type of the receiver:

```java
object.handle(other);
```

The selected method depends on `object`, but inside it you may still need to inspect `other`.

This can lead to:

```java
if (other instanceof PdfFile) {
    ...
} else if (other instanceof ImageFile) {
    ...
}
```

### 3.2 The Double Dispatch Solution

Let the second object participate in method selection:

```java
shape.collideWith(otherShape);
```

Inside:

```java
otherShape.collideWithCircle(this);
```

Now the final behavior knows both types.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| First object | Receives initial call. |
| Second object | Receives callback with concrete first type. |
| Pair-specific method | Method representing one type combination. |
| Client | Triggers interaction between two objects. |

### 3.4 Dispatch Flow

1. Client calls `a.interact(b)`.
2. Runtime selects `interact` based on `a`.
3. `a` calls `b.interactWithA(a)`.
4. Runtime selects method based on `b`.
5. Final method handles the `(A, B)` pair.

## 4. Java Coding Example

This example models document permissions based on both user type and document type.

```java
interface User {
    boolean canAccess(Document document);
    boolean canAccessPublic(PublicDocument document);
    boolean canAccessConfidential(ConfidentialDocument document);
}

final class Employee implements User {
    @Override
    public boolean canAccess(Document document) {
        return document.canBeAccessedBy(this);
    }

    @Override
    public boolean canAccessPublic(PublicDocument document) {
        return true;
    }

    @Override
    public boolean canAccessConfidential(ConfidentialDocument document) {
        return false;
    }
}

final class Admin implements User {
    @Override
    public boolean canAccess(Document document) {
        return document.canBeAccessedBy(this);
    }

    @Override
    public boolean canAccessPublic(PublicDocument document) {
        return true;
    }

    @Override
    public boolean canAccessConfidential(ConfidentialDocument document) {
        return true;
    }
}

interface Document {
    boolean canBeAccessedBy(User user);
}

final class PublicDocument implements Document {
    @Override
    public boolean canBeAccessedBy(User user) {
        return user.canAccessPublic(this);
    }
}

final class ConfidentialDocument implements Document {
    @Override
    public boolean canBeAccessedBy(User user) {
        return user.canAccessConfidential(this);
    }
}

public final class DoubleDispatchDemo {
    public static void main(String[] args) {
        User employee = new Employee();
        User admin = new Admin();
        Document confidential = new ConfidentialDocument();

        System.out.println(employee.canAccess(confidential)); // false
        System.out.println(admin.canAccess(confidential));    // true
    }
}
```

### Java Block by Block Explanation

`User.canAccess(document)` is the first dispatch on user runtime type.

`document.canBeAccessedBy(this)` is the second dispatch on document runtime type.

`canAccessConfidential` and `canAccessPublic` represent pair-specific rules.

Important detail:

- Adding new document types requires adding methods to users.
- Adding new user types requires implementing all document-specific methods.
- This is powerful but can grow quickly.

### Java Usage

Use Double Dispatch when:

- Pairwise type interaction is central to the domain.
- You want to avoid scattered `instanceof`.
- The set of types is reasonably stable.
- The interaction belongs in the domain objects.

## 5. Python Coding Example

Python supports more dynamic dispatch, but the same idea can be shown explicitly.

```python
class Employee:
    def can_access(self, document):
        return document.can_be_accessed_by_employee(self)


class Admin:
    def can_access(self, document):
        return document.can_be_accessed_by_admin(self)


class PublicDocument:
    def can_be_accessed_by_employee(self, user):
        return True

    def can_be_accessed_by_admin(self, user):
        return True


class ConfidentialDocument:
    def can_be_accessed_by_employee(self, user):
        return False

    def can_be_accessed_by_admin(self, user):
        return True


print(Employee().can_access(ConfidentialDocument()))
print(Admin().can_access(ConfidentialDocument()))
```

### Python Usage

In Python, you may also use:

- `functools.singledispatch`
- Dictionary dispatch keyed by type pairs.
- Pattern matching.
- Visitor-style methods.

Choose the approach that keeps the domain readable.

## 6. Where It Comes Handy in Real Life

- Collision handling in games/simulations.
- Access rules between actor type and resource type.
- Rendering object type against output type.
- Arithmetic operations between numeric types.
- Workflow behavior depending on request type and handler type.
- Validation rules depending on two domain dimensions.

## 7. Advantages Over Normal Code Without Pattern

### Without Double Dispatch

```java
if (user instanceof Admin && document instanceof ConfidentialDocument) {
    return true;
}
```

Problems:

- Type checks pile up.
- Rules move into a central conditional block.
- Adding a type requires editing many branches.
- Domain behavior becomes harder to locate.

### With Double Dispatch

```java
user.canAccess(document);
```

Benefits:

- Pair-specific behavior is modeled explicitly.
- Type checks are reduced.
- The interaction can be tested per pair.
- Method names document valid combinations.

## 8. Where It Excels

- Stable type hierarchies.
- Pairwise domain behavior.
- Simulation or rules engines with object interactions.
- Codebases that prefer polymorphism over type switches.
- Visitor-like designs.

## 9. Where It Fails

- Many rapidly changing types.
- Sparse interaction matrices.
- Simple rules that a map/table can express better.
- Cases where behavior does not belong inside either object.
- Languages or teams unfamiliar with the dispatch flow.

## 10. Prebuilt Libraries and Packages

### Java

- Java has single dispatch by default, so double dispatch is implemented manually.
- Visitor pattern uses double dispatch.
- Pattern matching for `switch` can sometimes be clearer for small cases.
- Rule engines can replace complex pairwise logic.

### Python

- `functools.singledispatch`
- Third-party multiple-dispatch libraries.
- Structural pattern matching.
- Dictionary dispatch by `(type(a), type(b))`.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids central type-checking logic. | Verbose in Java. |
| Models pairwise behavior clearly. | Adding new types can require many methods. |
| Uses polymorphism instead of conditionals. | Dispatch flow can be non-obvious. |
| Useful for simulations and visitors. | Interaction matrix can explode. |

## 12. Real-World Identification Example

Scenario:

You are designing a drawing app where tools interact differently with shapes.

Without Double Dispatch:

- Tool code has type checks for every shape.
- Shape code has type checks for every tool.

With Double Dispatch:

- Tool starts interaction.
- Shape receives a specific tool method.
- Pair-specific behavior is isolated.

## 13. MAANG Interview Triggers

Use Double Dispatch when you hear:

- "Behavior depends on two runtime types."
- "Avoid instanceof ladder."
- "Object-object interaction."
- "Collision logic."
- "Visitor-style dispatch."
- "Pairwise rules."

### Interview-Ready Answer Format

1. Identify the two type dimensions.
2. Define initial interaction method.
3. Add type-specific callback methods.
4. Route first dispatch into second dispatch.
5. Discuss type growth and method explosion.
6. Compare with table-driven dispatch or Visitor.

## 14. Common Mistakes

### Mistake 1: Using It for One Type Dimension

Simple polymorphism is enough when behavior depends on only one receiver.

### Mistake 2: Ignoring Type Growth

Every new type may require many new pair methods.

### Mistake 3: Hiding Business Rules Too Deeply

Pairwise behavior should still be discoverable.

### Mistake 4: Confusing It with Overloading

Java overload resolution uses compile-time parameter types, not runtime parameter types.

### Mistake 5: No Default Behavior

Define what happens for unsupported pairs.

## 15. Double Dispatch vs Similar Patterns

| Pattern | Difference |
|---|---|
| Double Dispatch | Dispatch based on two runtime types. |
| Visitor | Uses double dispatch to add operations to object structures. |
| Strategy | Selects algorithm for one context. |
| State | Changes behavior based on internal state. |
| Pattern matching | Centralized type-based branching instead of object collaboration. |

## 16. Double Dispatch Design Checklist

| Question | Why it matters |
|---|---|
| What are the two type dimensions? | Defines the interaction matrix. |
| Are types stable? | Prevents method explosion. |
| Where should pair behavior live? | Keeps design understandable. |
| What is default behavior? | Handles unsupported combinations. |
| Is table dispatch simpler? | Avoids unnecessary object complexity. |

## 17. Quick Revision Notes

- Double Dispatch selects behavior from two runtime types.
- Java needs a two-call trick.
- Visitor relies on this idea.
- Great for pairwise interactions.
- Avoid for rapidly growing type sets.
- Do not confuse overloading with runtime dispatch.

## 18. Mini Exercise

Design Double Dispatch for `DiscountPolicy`.

Types:

- Users: `GuestUser`, `MemberUser`, `PremiumUser`
- Products: `Book`, `Electronics`, `Grocery`

Task:

- Decide discount based on both user type and product type.

## 19. Source Reference in This Repo

The repository's Double Dispatch implementation models pairwise object interactions through `GameObject` and concrete object classes.

Useful files:

- [github-repo/double-dispatch/README.md](../../github-repo/double-dispatch/README.md)
- [github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/GameObject.java](../../github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/GameObject.java)
- [github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/FlamingAsteroid.java](../../github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/FlamingAsteroid.java)
- [github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/SpaceStationMir.java](../../github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/SpaceStationMir.java)
- [github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/App.java](../../github-repo/double-dispatch/src/main/java/com/iluwatar/doubledispatch/App.java)

