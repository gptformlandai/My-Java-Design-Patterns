# Data Transfer Object Pattern

Category: Enterprise Application Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [data-transfer-object](../../github-repo/data-transfer-object)

---

## How to Study This Page

Study Data Transfer Object as "the shape of data crossing a boundary."

Remember the key idea:

```text
Domain model stays inside.
DTO crosses API, service, process, or layer boundaries.
```

For interviews, DTO is not just a Java POJO. It is about decoupling boundaries, controlling payload shape, reducing chatter, and avoiding accidental leakage of internal data.

---

## 1. Technical Definition

Data Transfer Object is an object that carries data between processes, services, layers, or API boundaries without containing business behavior.

### 30-Second Interview Answer

A DTO is a data-only object used to move data across a boundary, such as API request/response, service-to-service calls, controller-to-service boundaries, or remote calls. It decouples external contracts from internal domain models, lets us shape payloads for each client, hides sensitive fields, and reduces multiple calls by bundling related fields. The trade-off is extra mapping code and the risk of creating DTOs that mirror entities without purpose.

---

## 2. Layman and Easy to Understand Definition

Think of a shipping package.

The warehouse may store many internal details about a product, but the delivery label contains only what the courier needs: address, recipient, and tracking number.

A DTO is that delivery label for software data.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Sending domain objects directly across boundaries creates problems:

```text
internal entity -> API response
```

This can leak fields, expose database shape, couple clients to internal models, and make API changes risky.

### DTO Flow

1. Domain model/entity stores internal state and behavior.
2. Mapper or resource converts domain data into a DTO.
3. DTO crosses the boundary.
4. Client receives only the fields intended for that use case.
5. Request DTOs capture input without exposing domain constructors.
6. Response DTOs shape output for each audience.

### Core Participants

| Participant | Responsibility |
|---|---|
| Domain entity/model | Internal business data and behavior |
| Request DTO | Carries client input into the system |
| Response DTO | Carries shaped output to clients |
| Mapper/assembler | Converts between domain and DTO |
| Boundary | API, service, process, or layer edge |
| Client | Consumes the stable transfer contract |

---

## 4. Java Coding Example

```java
record UserEntity(Long id, String email, String passwordHash, String role) {}

record UserResponseDto(Long id, String email) {}

record CreateUserRequestDto(String email, String rawPassword) {}

class UserMapper {
    UserResponseDto toResponse(UserEntity user) {
        return new UserResponseDto(user.id(), user.email());
    }
}

public class DtoDemo {
    public static void main(String[] args) {
        UserEntity entity = new UserEntity(1L, "a@example.com", "hashed-secret", "ADMIN");
        UserResponseDto dto = new UserMapper().toResponse(entity);

        System.out.println(dto);
    }
}
```

### Java Block by Block

`UserEntity` has internal fields such as password hash and role.

`UserResponseDto` exposes only safe response data.

`CreateUserRequestDto` represents input data, not a database entity.

`UserMapper` makes the boundary explicit.

---

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass
class UserEntity:
    id: int
    email: str
    password_hash: str
    role: str


@dataclass
class UserResponseDto:
    id: int
    email: str


def to_response(user):
    return UserResponseDto(id=user.id, email=user.email)


entity = UserEntity(1, "a@example.com", "hashed-secret", "ADMIN")
print(to_response(entity))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| REST API responses | Shape output and hide internal fields |
| API request bodies | Validate and document input contracts |
| Microservice calls | Keep service contracts stable |
| Batch export/import | Move structured data across systems |
| GraphQL/view models | Return client-specific field sets |
| Admin vs public APIs | Expose different fields to different audiences |

---

## 7. Advantages Over Normal Code Without Pattern

Without DTO:
- API responses may leak entity fields
- clients become coupled to database/domain shape
- request validation mixes with domain behavior
- remote calls may require many small round trips

With DTO:
- boundary contracts are explicit
- fields can be shaped per use case
- sensitive internal data can be hidden
- domain model can evolve independently

---

## 8. Where It Excels

It excels when:
- data crosses service or API boundaries
- external contract stability matters
- clients need different data shapes
- sensitive fields must be excluded
- validation differs between create/update/read flows
- network round trips should be reduced

---

## 9. Where It Fails

It fails when:
- DTOs blindly mirror entities
- mapping code becomes duplicated everywhere
- business logic is placed inside DTOs
- DTO explosion creates maintenance overhead
- versioning strategy is missing
- DTOs are used inside the core domain as if they were domain models

Use entities/domain models internally and DTOs at boundaries. Add DTOs when they protect a real boundary or contract.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | records, Lombok, Jackson, Jakarta Validation |
| Mapping | MapStruct, ModelMapper, manual mappers |
| Spring | Spring MVC request/response bodies, `@Valid` |
| Serialization | Jackson, Gson, JSON-B, Protobuf |
| Python | dataclasses, Pydantic, Marshmallow |
| APIs | OpenAPI schemas, GraphQL types, gRPC messages |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples API contract from domain model | Adds mapping code |
| Prevents sensitive data leakage | Can create many classes |
| Shapes data per client/use case | Entity-mirroring DTOs add little value |
| Useful for validation and documentation | Versioning still needs discipline |
| Reduces remote-call chatter | Wrong mapping can cause bugs |

---

## 12. Real-World Identification Example

Question:

> A user API returns the database entity directly. It includes password hash, internal flags, and fields mobile clients do not need. What pattern helps?

Strong answer:

Use DTOs. Keep the user entity internal and create response DTOs for public, admin, and mobile use cases. Map only intended fields into each DTO and validate incoming request DTOs separately. This protects sensitive fields, decouples clients from persistence structure, and lets the domain model evolve without breaking API contracts.

---

## 13. MAANG Interview Triggers

Say DTO when you hear:
- API request/response shape
- hide sensitive fields
- do not expose entity
- service boundary contract
- reduce remote calls
- validation object
- public vs admin response
- map entity to response

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Returning entities directly | Leaks internal structure and sensitive fields | Map to response DTOs |
| Putting business logic in DTOs | DTO becomes confused with domain model | Keep DTO data-only |
| One DTO for every operation | Fields become optional/confusing | Use use-case-specific DTOs |
| Blindly mirroring entities | Adds classes without decoupling | Shape DTOs around boundary needs |
| No versioning plan | Client contracts break | Version APIs or evolve DTOs compatibly |

---

## 15. DTO vs Similar Patterns

| Pattern | Difference |
|---|---|
| Value Object | Value Object models domain value and behavior; DTO transfers data across boundaries |
| Domain Model | Domain Model owns business rules; DTO should not |
| Data Mapper | Data Mapper maps between objects and database; DTO mapper maps between domain and transfer shapes |
| Facade | Facade simplifies operations; DTO simplifies data transfer |
| ViewModel | ViewModel is presentation-specific; DTO is broader boundary-transfer shape |

---

## 16. DTO Design Checklist

- What boundary does this DTO cross?
- Is it request, response, event, export, or internal layer DTO?
- Which fields should be included?
- Which fields must never be included?
- Who maps between DTO and domain model?
- Where does validation happen?
- Does this DTO mirror an entity without reason?
- How will it evolve without breaking clients?
- Is serialization format stable and documented?

---

## 17. Quick Revision Notes

- One-line summary: DTO is a data-only contract for crossing boundaries.
- Memory hook: "shape the payload, protect the domain."
- Best for: APIs, service calls, request/response contracts, exports.
- Avoid when: there is no real boundary or it merely duplicates the entity.
- Interview line: "I would use DTOs to keep API contracts stable and avoid exposing internal domain or persistence details."

---

## 18. Mini Exercise

Design DTOs for an order API:
- create a `CreateOrderRequest`
- create a public `OrderResponse`
- create an admin `OrderAdminResponse`
- decide which fields should not leave the service
- write one mapper method from domain object to public response

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/data-transfer-object/README.md)
- [App.java](../../github-repo/data-transfer-object/src/main/java/com/iluwatar/datatransfer/App.java)
- [CustomerDto.java](../../github-repo/data-transfer-object/src/main/java/com/iluwatar/datatransfer/customer/CustomerDto.java)
- [CustomerResource.java](../../github-repo/data-transfer-object/src/main/java/com/iluwatar/datatransfer/customer/CustomerResource.java)
- [Product.java](../../github-repo/data-transfer-object/src/main/java/com/iluwatar/datatransfer/product/Product.java)
- [ProductDto.java](../../github-repo/data-transfer-object/src/main/java/com/iluwatar/datatransfer/product/ProductDto.java)
- [ProductResource.java](../../github-repo/data-transfer-object/src/main/java/com/iluwatar/datatransfer/product/ProductResource.java)

The repo implementation shows simple customer DTOs and richer product DTOs. `ProductResource` maps internal `Product` entities to admin/private and customer/public response DTOs, demonstrating how DTOs shape data for different audiences.
