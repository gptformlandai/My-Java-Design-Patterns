# Step Builder Pattern

Category: Creational and Object Construction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/step-builder](../../github-repo/step-builder)

## How to Study This Page

Use this page in three passes:

1. First pass: understand how Step Builder forces construction order through types.
2. Second pass: rewrite the Java example and trace which interface is returned after each method call.
3. Third pass: compare Step Builder with normal Builder and decide when the extra boilerplate is worth it.

By the end, you should be able to say:

> Step Builder is a stricter Builder variant that guides callers through required construction steps and prevents calling `build()` too early.

## 1. Technical Definition

Step Builder is a creational pattern and Builder variant that splits object construction into typed stages. Each stage exposes only the next valid methods, so required fields and construction order can be enforced at compile time.

Core idea:

- Represent each construction step as an interface.
- Each method returns the next step interface.
- Hide `build()` until required steps are completed.
- Use the type system to prevent invalid construction order.

### 30-Second Interview Answer

I would use Step Builder when a normal builder is too loose because some fields are required in a specific order or some choices determine the next valid steps. It improves compile-time safety and API guidance, but it adds many interfaces and can become verbose. For most objects, a normal builder with validation in `build()` is enough.

## 2. Layman and Easy to Understand Definition

Step Builder is like an online form that shows only the next valid page.

You cannot submit before filling required pages:

```text
Enter account details.
Choose plan.
Add payment method.
Submit.
```

The form does not even show the submit button until required steps are done.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

A normal builder may allow this:

```java
SignupRequest request = SignupRequest.builder()
    .displayName("Aravind")
    .build();
```

Problems:

- Required fields may be missing.
- Errors are found at runtime.
- Callers may not know the correct order.
- API does not guide the developer enough.

### 3.2 The Step Builder Solution

Expose construction in stages:

```java
SignupRequest request = SignupRequest.builder()
    .email("user@example.com")
    .password("secret")
    .displayName("Aravind")
    .build();
```

After `builder()`, only `email()` is visible. After `email()`, only `password()` is visible. `build()` appears only after required fields are provided.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Product | Final object being created. |
| Step interface | Interface exposing methods valid at one stage. |
| Steps implementation | Class implementing all step interfaces and storing state. |
| Required step | A method that must be called before moving forward. |
| Build step | Final stage where `build()` becomes available. |

### 3.4 Mental Model

Think of Step Builder as a typed checklist.

1. Caller starts at the first step.
2. Each method records a value.
3. Each method returns the next step type.
4. Invalid next methods are not visible.
5. `build()` appears only when required steps are complete.

## 4. Java Coding Example

This example creates a signup request that requires email and password before `build()`.

```java
public record SignupRequest(String email, String password, String displayName) {

    public static EmailStep builder() {
        return new Steps();
    }

    public interface EmailStep {
        PasswordStep email(String email);
    }

    public interface PasswordStep {
        OptionalStep password(String password);
    }

    public interface OptionalStep {
        OptionalStep displayName(String displayName);
        SignupRequest build();
    }

    private static final class Steps implements EmailStep, PasswordStep, OptionalStep {
        private String email;
        private String password;
        private String displayName = "";

        @Override
        public PasswordStep email(String email) {
            if (email == null || email.isBlank()) {
                throw new IllegalArgumentException("email is required");
            }
            this.email = email;
            return this;
        }

        @Override
        public OptionalStep password(String password) {
            if (password == null || password.isBlank()) {
                throw new IllegalArgumentException("password is required");
            }
            this.password = password;
            return this;
        }

        @Override
        public OptionalStep displayName(String displayName) {
            this.displayName = displayName == null ? "" : displayName;
            return this;
        }

        @Override
        public SignupRequest build() {
            return new SignupRequest(email, password, displayName);
        }
    }
}
```

### Java Block by Block Explanation

#### Product

```java
public record SignupRequest(String email, String password, String displayName) {
```

The final object is simple and immutable.

#### First Step

```java
public static EmailStep builder() {
    return new Steps();
}
```

The builder starts by returning only the first step interface.

#### Step Interfaces

```java
public interface EmailStep {
    PasswordStep email(String email);
}
```

Each interface exposes only valid methods for that step.

#### Shared Step Implementation

```java
private static final class Steps implements EmailStep, PasswordStep, OptionalStep {
```

One internal class stores state and implements all step interfaces.

#### Returning the Next Step

```java
public PasswordStep email(String email) {
```

After email is set, the caller receives `PasswordStep`, so password is now the next visible operation.

#### Build Step

```java
public SignupRequest build() {
```

`build()` exists only on `OptionalStep`, which is reached after email and password.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        SignupRequest request = SignupRequest.builder()
            .email("user@example.com")
            .password("secret")
            .displayName("Aravind")
            .build();

        System.out.println(request);
    }
}
```

This will not compile:

```java
SignupRequest.builder().build();
```

`build()` is not available at the first step.

## 5. Python Coding Example

Python does not enforce this as naturally at runtime, but type hints can express staged builders.

```python
from dataclasses import dataclass
from typing import Protocol


@dataclass(frozen=True)
class SignupRequest:
    email: str
    password: str
    display_name: str = ""


class EmailStep(Protocol):
    def email(self, email: str) -> "PasswordStep":
        ...


class PasswordStep(Protocol):
    def password(self, password: str) -> "OptionalStep":
        ...


class OptionalStep(Protocol):
    def display_name(self, display_name: str) -> "OptionalStep":
        ...

    def build(self) -> SignupRequest:
        ...


class SignupRequestBuilder:
    def __init__(self) -> None:
        self._email = ""
        self._password = ""
        self._display_name = ""

    def email(self, email: str) -> PasswordStep:
        if not email:
            raise ValueError("email is required")
        self._email = email
        return self

    def password(self, password: str) -> OptionalStep:
        if not password:
            raise ValueError("password is required")
        self._password = password
        return self

    def display_name(self, display_name: str) -> OptionalStep:
        self._display_name = display_name
        return self

    def build(self) -> SignupRequest:
        return SignupRequest(self._email, self._password, self._display_name)


def signup_request_builder() -> EmailStep:
    return SignupRequestBuilder()
```

### Python Usage

```python
request = (
    signup_request_builder()
    .email("user@example.com")
    .password("secret")
    .display_name("Aravind")
    .build()
)
```

Python type checkers can help, but runtime enforcement is still mostly from validation.

## 6. Where It Comes Handy in Real Life

Step Builder is useful when object creation has required phases.

Examples:

- Signup request: email before password before build.
- Query builder: select before from before where.
- Workflow builder: trigger before action before activation.
- API request builder: endpoint before authentication before execution.
- Character/profile creation with alternative paths.
- Configuration builders where one choice controls next valid options.

## 7. Advantages Over Normal Code Without Pattern

### Without Step Builder

```java
SignupRequest request = SignupRequest.builder()
    .displayName("Aravind")
    .build();
```

Problems:

- Required fields can be missed.
- Runtime validation catches errors late.
- The API does not guide construction.
- Callers may guess the order.

### With Step Builder

```java
SignupRequest request = SignupRequest.builder()
    .email("user@example.com")
    .password("secret")
    .build();
```

Benefits:

- Required order is compile-time guided.
- `build()` is hidden until valid.
- API discovery is easier in IDE autocomplete.
- Invalid call sequences are harder to write.

## 8. Where It Excels

Step Builder excels when:

- Required fields must be supplied before building.
- Construction order matters.
- Some choices lead to different next steps.
- You want IDE autocomplete to guide usage.
- Runtime validation alone is not enough.
- The builder API is public and used by many teams.

## 9. Where It Fails

Step Builder is a poor fit when:

- The object has only a few obvious fields.
- A normal builder with `build()` validation is enough.
- There are many optional fields but no required order.
- The interface explosion makes code hard to maintain.
- Product requirements change frequently.

Example where Step Builder is overkill:

```java
record Point(int x, int y) {}
```

For simple values, direct construction is clearer.

## 10. Prebuilt Libraries and Packages

### Java

There is no single standard Step Builder library. It is usually hand-written when needed.

Related tools:

- Lombok `@Builder` for normal builders.
- Immutables and AutoValue for generated builders.
- Java records for simple immutable data.
- IDE autocomplete, which makes staged interfaces pleasant to use.

Important note:

- Code generation can reduce normal builder boilerplate, but Step Builder often needs custom interfaces because the order rules are domain-specific.

### Python

Python usually avoids formal Step Builder unless type-checker guidance is important.

Alternatives:

- Dataclasses.
- Pydantic models with validation.
- Keyword arguments.
- Factory functions.
- Normal builders with runtime validation.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Guides construction order. | Adds many interfaces. |
| Prevents early `build()` calls. | More boilerplate than Builder. |
| Improves IDE autocomplete. | Harder to change steps later. |
| Encodes required fields in types. | Can feel excessive for ordinary objects. |
| Helps public APIs be harder to misuse. | Still needs value validation. |

## 12. Real-World Identification Example

Scenario:

You are designing a workflow builder.

Rules:

- A workflow must start with a trigger.
- Then it must define at least one action.
- Then it can optionally add retry policy and metadata.
- It can be built only after at least one action exists.

Should you use Step Builder?

Yes, if this builder is part of a public API where misuse is common or expensive.

Good usage:

```java
Workflow workflow = Workflow.builder()
    .trigger("order.created")
    .action(sendEmail)
    .retryPolicy(retryPolicy)
    .build();
```

The API can prevent `build()` before `trigger()` and `action()`.

## 13. MAANG Interview Triggers

Think Step Builder when you hear:

- Required construction order.
- Compile-time builder safety.
- Hide `build()` until valid.
- Fluent API with stages.
- Prevent invalid object construction.
- IDE-guided object creation.
- Builder variant.

### Interview-Ready Answer Format

Use this structure when answering:

1. Explain why normal Builder is too permissive.
2. Identify required steps and optional steps.
3. Create one interface per stage.
4. Make each method return the next stage interface.
5. Expose `build()` only at the valid final stage.
6. Mention trade-off: more boilerplate and harder maintenance.

## 14. Common Mistakes

### Mistake 1: Using Step Builder for Simple Objects

If a constructor or normal builder is clear, Step Builder is unnecessary.

### Mistake 2: Too Many Tiny Steps

Do not create a separate stage for every optional field. Use stages for real required transitions.

### Mistake 3: Forgetting Runtime Validation

Step Builder can force method calls, but it cannot guarantee the value is valid.

Bad:

```java
.email("")
```

Still validate blank, malformed, or inconsistent values.

### Mistake 4: Making the Public API Too Rigid

If requirements change often, staged interfaces can become painful to maintain.

### Mistake 5: Leaking the Implementation Class

Callers should see step interfaces, not the internal `Steps` class.

## 15. Step Builder vs Similar Patterns

| Pattern | Difference |
|---|---|
| Builder | Normal Builder validates at `build()`. Step Builder prevents invalid call order through types. |
| Factory | Factory chooses product type. Step Builder guides construction of one product. |
| Abstract Factory | Creates related product families. Step Builder creates one object through stages. |
| Fluent Interface | Method chaining style. Step Builder is a specific staged fluent construction pattern. |
| Template Method | Defines algorithm steps. Step Builder defines object-construction steps. |

## 16. Step Design Checklist

Before implementing Step Builder, decide:

| Question | Why it matters |
|---|---|
| Which fields are truly required? | Only required transitions deserve stages. |
| Does order matter? | If not, normal Builder may be enough. |
| Are there branching paths? | Branching may need separate step interfaces. |
| Who uses this API? | Public APIs benefit more from guidance. |
| How often will rules change? | Frequent changes make staged builders costly. |

## 17. Quick Revision Notes

- Step Builder is a stricter Builder.
- Each stage is usually an interface.
- Methods return the next stage type.
- `build()` appears only after required steps.
- It improves compile-time guidance.
- It costs more boilerplate than normal Builder.

## 18. Mini Exercise

Design a Step Builder for `SqlQuery`.

Required order:

- `select(...)`
- `from(...)`
- optional `where(...)`
- optional `orderBy(...)`
- `build()`

Invalid:

```java
SqlQuery.builder().from("users").build();
```

Expected usage:

```java
SqlQuery query = SqlQuery.builder()
    .select("id", "email")
    .from("users")
    .where("active = true")
    .build();
```

## 19. Source Reference in This Repo

The repository's Step Builder implementation creates a `Character` through staged interfaces in `CharacterStepBuilder`.

Useful files:

- [github-repo/step-builder/README.md](../../github-repo/step-builder/README.md)
- [github-repo/step-builder/src/main/java/com/iluwatar/stepbuilder/CharacterStepBuilder.java](../../github-repo/step-builder/src/main/java/com/iluwatar/stepbuilder/CharacterStepBuilder.java)
- [github-repo/step-builder/src/main/java/com/iluwatar/stepbuilder/Character.java](../../github-repo/step-builder/src/main/java/com/iluwatar/stepbuilder/Character.java)
- [github-repo/step-builder/src/main/java/com/iluwatar/stepbuilder/App.java](../../github-repo/step-builder/src/main/java/com/iluwatar/stepbuilder/App.java)
