# Parameter Object Pattern

Category: Domain Modeling and Business Rule Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/parameter-object](../../github-repo/parameter-object)

## How to Study This Page

Use this page in three passes:

1. First pass: understand grouping related method parameters into one named object.
2. Second pass: rewrite the Java example and identify long parameter list, parameter object, builder/defaults, and consuming service.
3. Third pass: compare Parameter Object with Value Object, DTO, Builder, and Command.

By the end, you should be able to say:

> Parameter Object replaces a long list of related method arguments with one structured object.

## 1. Technical Definition

Parameter Object is a refactoring and design pattern that groups related parameters into a single object to simplify method signatures and improve readability, evolution, and validation.

Core idea:

- Long parameter list becomes one object.
- Related values are named together.
- Defaults and validation can live in one place.
- Method signatures become stable as parameters evolve.
- Builder/factory can make construction readable.

### 30-Second Interview Answer

I would use Parameter Object when a method has many related parameters, especially optional filters or search criteria. Instead of `search(type, sortBy, sortOrder, page, size)`, I pass `SearchCriteria`. This makes the signature easier to read and evolve. The trade-off is that using it for two unrelated parameters adds unnecessary ceremony, and a parameter object should not become a vague bag of everything.

## 2. Layman and Easy to Understand Definition

Parameter Object is like filling out one form instead of answering ten separate questions every time.

The form groups related information, gives each field a name, and can provide defaults.

In code:

- Many arguments become one object.
- The object gives the arguments a shared name.
- The method becomes easier to call and evolve.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Long parameter list:

```java
search("shoes", "brand", SortOrder.ASC, 1, 20, true);
```

Problems:

- Hard to remember argument order.
- Optional values create overloads.
- Adding parameters breaks callers.
- Related values are not named as a concept.
- Validation is scattered.

### 3.2 The Parameter Object Solution

Group parameters:

```java
SearchCriteria criteria = SearchCriteria.builder()
    .type("shoes")
    .sortBy("brand")
    .sortOrder(SortOrder.ASC)
    .page(1)
    .pageSize(20)
    .build();

search(criteria);
```

The method receives one meaningful object.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Parameter object | Groups related arguments. |
| Fields | Former method parameters. |
| Builder/factory | Optional construction helper. |
| Consumer method | Method that accepts parameter object. |
| Validation/defaults | Rules applied to grouped inputs. |

### 3.4 Good Parameter Object Names

Good:

```java
SearchCriteria
Pagination
DateRange
PaymentRequest
ReportOptions
```

Weak:

```java
Params
Data
Input
EverythingObject
```

## 4. Java Coding Example

This example replaces a long search signature.

```java
enum SortOrder {
    ASCENDING, DESCENDING
}

public final class SearchCriteria {
    private final String type;
    private final String sortBy;
    private final SortOrder sortOrder;
    private final int page;
    private final int pageSize;

    private SearchCriteria(Builder builder) {
        this.type = builder.type;
        this.sortBy = builder.sortBy;
        this.sortOrder = builder.sortOrder;
        this.page = builder.page;
        this.pageSize = builder.pageSize;
    }

    public static Builder builder() {
        return new Builder();
    }

    public String type() {
        return type;
    }

    public String sortBy() {
        return sortBy;
    }

    public SortOrder sortOrder() {
        return sortOrder;
    }

    public int page() {
        return page;
    }

    public int pageSize() {
        return pageSize;
    }

    public static final class Builder {
        private String type = "all";
        private String sortBy = "createdAt";
        private SortOrder sortOrder = SortOrder.DESCENDING;
        private int page = 1;
        private int pageSize = 20;

        public Builder type(String type) {
            this.type = type;
            return this;
        }

        public Builder sortBy(String sortBy) {
            this.sortBy = sortBy;
            return this;
        }

        public Builder sortOrder(SortOrder sortOrder) {
            this.sortOrder = sortOrder;
            return this;
        }

        public Builder page(int page) {
            this.page = page;
            return this;
        }

        public Builder pageSize(int pageSize) {
            this.pageSize = pageSize;
            return this;
        }

        public SearchCriteria build() {
            if (page < 1 || pageSize < 1) {
                throw new IllegalArgumentException("page and pageSize must be positive");
            }
            return new SearchCriteria(this);
        }
    }
}

final class SearchService {
    String search(SearchCriteria criteria) {
        return "type=" + criteria.type()
            + ", sortBy=" + criteria.sortBy()
            + ", order=" + criteria.sortOrder()
            + ", page=" + criteria.page()
            + ", pageSize=" + criteria.pageSize();
    }
}
```

### Java Block by Block Explanation

`SearchCriteria` groups related search parameters.

The builder provides defaults and readable construction.

`build` validates the grouped values.

`SearchService.search` has one stable parameter instead of a long list.

### Java Usage

Use Parameter Object in Java when:

- A method has many related parameters.
- Optional parameters are growing.
- Call sites are hard to read.
- Parameters travel together through multiple methods.
- Defaults and validation belong together.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class SearchCriteria:
    type: str = "all"
    sort_by: str = "created_at"
    sort_order: str = "descending"
    page: int = 1
    page_size: int = 20

    def __post_init__(self):
        if self.page < 1 or self.page_size < 1:
            raise ValueError("page and page_size must be positive")


class SearchService:
    def search(self, criteria):
        return {
            "type": criteria.type,
            "sortBy": criteria.sort_by,
            "sortOrder": criteria.sort_order,
            "page": criteria.page,
            "pageSize": criteria.page_size,
        }


criteria = SearchCriteria(type="shoes", sort_by="brand")
print(SearchService().search(criteria))
```

### Python Usage

Python parameter objects are often:

- Dataclasses.
- Pydantic request models.
- TypedDicts for simple data.
- Query/filter objects.
- Command/request objects.

## 6. Where It Comes Handy in Real Life

- Search filters.
- Pagination options.
- Report generation options.
- Payment request details.
- API request models.
- Batch job configuration.
- Export options.
- Notification preferences.

## 7. Advantages Over Normal Code Without Pattern

### Without Parameter Object

```java
search(type, sortBy, sortOrder, page, pageSize, includeArchived);
```

Problems:

- Hard to read call sites.
- Easy to pass values in wrong order.
- Overloads multiply.
- Adding parameters breaks callers.

### With Parameter Object

```java
search(criteria);
```

Benefits:

- Call sites are clearer.
- Defaults are centralized.
- Validation is grouped.
- Method signature is stable.
- Related values have a name.

## 8. Where It Excels

- Long parameter lists.
- Optional settings.
- Search/filter/report APIs.
- Values passed through multiple layers.
- Stable public method signatures.
- Builder-friendly construction.

## 9. Where It Fails

- Only one or two simple parameters.
- Parameters are unrelated.
- Object becomes a dumping ground.
- Required fields are unclear.
- Parameter object starts owning business behavior that belongs elsewhere.

## 10. Prebuilt Frameworks and Packages

### Java

- Java records for simple parameter objects.
- Builder pattern.
- Lombok `@Builder`.
- Bean Validation.
- Spring MVC request DTOs.

### Python

- Dataclasses.
- Pydantic models.
- TypedDict.
- attrs.
- FastAPI request/query models.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Simplifies method signatures. | Adds another type. |
| Improves call-site readability. | Can become a vague data bag. |
| Centralizes defaults and validation. | Overkill for tiny methods. |
| Evolves without breaking signatures. | Required vs optional fields need care. |
| Groups related data clearly. | Can hide unrelated parameters together. |

## 12. Real-World Identification Example

Scenario:

You are designing product search.

Parameter Object fit:

- `query`
- `category`
- `sortBy`
- `sortOrder`
- `page`
- `pageSize`
- `priceRange`

What would go wrong without it:

- Method signatures become long.
- Callers confuse parameter order.
- New filters require many overload changes.

## 13. MAANG Interview Triggers

Use Parameter Object when you hear:

- "Long parameter list."
- "Too many optional arguments."
- "Search criteria."
- "Pass same parameters through layers."
- "Stable method signature."
- "Group related arguments."
- "Builder with defaults."

### Interview-Ready Answer Format

1. Identify related parameters.
2. Name the concept they represent.
3. Create immutable parameter object.
4. Add validation/defaults.
5. Replace method signature with one object.
6. Mention avoiding dumping unrelated values together.

## 14. Common Mistakes

### Mistake 1: Parameter Object With Unrelated Fields

The object should represent one coherent concept.

### Mistake 2: Vague Name

Names like `Params` or `Data` hide intent.

### Mistake 3: Mutable Shared Parameter Object

Mutable objects can cause surprising behavior when reused.

### Mistake 4: No Validation

Invalid grouped input should be caught early.

### Mistake 5: Confusing Parameter Object With Domain Value Object

Some parameter objects are just request shapes, not domain concepts.

## 15. Parameter Object vs Similar Patterns

| Pattern | Difference |
|---|---|
| Parameter Object | Groups method arguments. |
| Value Object | Domain value with value-based equality and invariants. |
| DTO | Transfers data across process/layer boundaries. |
| Builder | Helps construct complex objects. |
| Command | Represents an action/request, often with behavior or handling semantics. |

## 16. Parameter Object Design Checklist

| Question | Why it matters |
|---|---|
| Are parameters related? | Justifies grouping. |
| What concept names the group? | Improves readability. |
| Which fields are required? | Prevents invalid construction. |
| What defaults exist? | Simplifies callers. |
| Should it be immutable? | Avoids reuse surprises. |
| Is it really a command/DTO/value object? | Clarifies responsibility. |

## 17. Quick Revision Notes

- Parameter Object replaces long argument lists.
- It groups related values.
- It stabilizes method signatures.
- Builders help with optional fields.
- Keep it cohesive and named.
- Do not create vague bags of unrelated data.

## 18. Mini Exercise

Design Parameter Object for `GenerateReport`.

Fields:

- `startDate`
- `endDate`
- `format`
- `includeSummary`
- `sortBy`
- `timezone`

Rules:

- Start date must be before end date.
- Format defaults to PDF.
- Timezone defaults to UTC.

## 19. Source Reference in This Repo

The repository's Parameter Object implementation uses `ParameterObject`, `SearchService`, and `SortOrder` to simplify a search method signature.

Useful files:

- [github-repo/parameter-object/README.md](../../github-repo/parameter-object/README.md)
- [github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/ParameterObject.java](../../github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/ParameterObject.java)
- [github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/SearchService.java](../../github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/SearchService.java)
- [github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/SortOrder.java](../../github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/SortOrder.java)
- [github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/App.java](../../github-repo/parameter-object/src/main/java/com/iluwatar/parameter/object/App.java)

