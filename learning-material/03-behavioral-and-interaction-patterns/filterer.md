# Filterer Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/filterer](../../github-repo/filterer)

## How to Study This Page

Use this page in three passes:

1. First pass: understand a container returning filtered versions of itself.
2. Second pass: rewrite the Java example and identify container, element, predicate, and filtered result.
3. Third pass: compare Filterer with Specification, Iterator, Stream, and Chain of Responsibility.

By the end, you should be able to say:

> Filterer lets a container-like object expose fluent filtering while preserving its own domain type.

## 1. Technical Definition

Filterer is a behavioral pattern where an object provides a filtering API that returns a filtered object of the same domain family based on predicates.

Core idea:

- A domain object owns a collection of elements.
- A filterer accepts predicates.
- Filtering returns a new narrowed domain object.
- The caller stays in domain language instead of raw collection language.

### 30-Second Interview Answer

I would use Filterer when a domain aggregate contains items and clients need flexible filtering without exposing mutable internals. The aggregate exposes a `filtered()` API that accepts predicates and returns a new filtered aggregate. This keeps domain type information and supports fluent filtering. The trade-off is that it can duplicate what streams already do if the domain wrapper adds no value.

## 2. Layman and Easy to Understand Definition

Filterer is like filtering a shopping catalog but still getting a catalog back.

You ask for "only available items" or "only electronics", and the result is not just a loose list. It is still a catalog with the same behavior, but with fewer items.

In code:

- Catalog is the container.
- Products are elements.
- Predicate is the condition.
- Filtered catalog is the result.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

A domain object may expose raw collections:

```java
List<Product> products = catalog.products();
List<Product> available = products.stream()
    .filter(Product::isAvailable)
    .toList();
```

Problems:

- Caller leaves the domain abstraction.
- Filtering logic is repeated everywhere.
- Type information may be weakened.
- Mutable collections can leak internal state.

### 3.2 The Filterer Solution

Let the domain object filter itself:

```java
Catalog available = catalog.filtered().by(Product::isAvailable);
```

The result remains a `Catalog`.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Container | Domain object that owns elements. |
| Element | Item being filtered. |
| Filterer | Object/function accepting predicates. |
| Predicate | Condition used to keep elements. |
| Filtered result | New domain object with matching elements. |

### 3.4 Mental Model

Filterer is domain-aware filtering.

1. Client asks container for its filter API.
2. Client supplies a predicate.
3. Container applies predicate internally.
4. Container returns a filtered copy or view.
5. Client continues using the domain object.

## 4. Java Coding Example

This example filters a product catalog.

```java
import java.util.List;
import java.util.function.Predicate;

@FunctionalInterface
interface Filterer<G, E> {
    G by(Predicate<? super E> predicate);
}

record Product(String name, String category, boolean available) {
}

final class ProductCatalog {
    private final List<Product> products;

    ProductCatalog(List<Product> products) {
        this.products = List.copyOf(products);
    }

    List<Product> products() {
        return products;
    }

    Filterer<ProductCatalog, Product> filtered() {
        return predicate -> new ProductCatalog(
            products.stream()
                .filter(predicate)
                .toList()
        );
    }
}

public final class FiltererDemo {
    public static void main(String[] args) {
        ProductCatalog catalog = new ProductCatalog(List.of(
            new Product("Book", "education", true),
            new Product("Camera", "electronics", false),
            new Product("Laptop", "electronics", true)
        ));

        ProductCatalog availableElectronics = catalog
            .filtered().by(Product::available)
            .filtered().by(product -> product.category().equals("electronics"));

        System.out.println(availableElectronics.products());
    }
}
```

### Java Block by Block Explanation

`Filterer<G, E>` expresses "filter elements of type `E` and return group type `G`."

`ProductCatalog` stores products immutably using `List.copyOf`.

`filtered()` returns a filter function that creates a new `ProductCatalog`.

Chaining works because the result is still a `ProductCatalog`.

### Java Usage

Use Filterer when:

- You need filtered domain objects, not just raw lists.
- Filtering should preserve type and behavior.
- You want fluent domain-specific filtering.
- You want to protect internal collections.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Product:
    name: str
    category: str
    available: bool


class ProductCatalog:
    def __init__(self, products):
        self._products = tuple(products)

    @property
    def products(self):
        return self._products

    def by(self, predicate):
        return ProductCatalog(product for product in self._products if predicate(product))


catalog = ProductCatalog([
    Product("Book", "education", True),
    Product("Camera", "electronics", False),
    Product("Laptop", "electronics", True),
])

available_electronics = catalog.by(lambda p: p.available).by(
    lambda p: p.category == "electronics"
)

print(available_electronics.products)
```

### Python Usage

In Python, Filterer often appears as:

- Domain collection methods.
- Query objects.
- Lazy generator wrappers.
- Data pipeline filters.

Use plain list comprehensions when no domain wrapper is needed.

## 6. Where It Comes Handy in Real Life

- Product catalogs.
- Search results.
- Threat or alert dashboards.
- Policy filtering.
- Inventory systems.
- Domain collections with reusable predicates.
- APIs that should return typed result objects.

## 7. Advantages Over Normal Code Without Pattern

### Without Filterer

```java
List<Product> result = catalog.products().stream()
    .filter(Product::available)
    .toList();
```

Problems:

- Result is just a list.
- Caller manipulates collection details.
- Domain behavior is lost.
- Filtering logic can be duplicated.

### With Filterer

```java
ProductCatalog result = catalog.filtered().by(Product::available);
```

Benefits:

- Result remains domain-specific.
- Internal collection is protected.
- Filtering is fluent.
- Predicates are reusable.

## 8. Where It Excels

- Domain collections.
- Immutable result objects.
- Fluent filtering APIs.
- Search/result narrowing.
- Type-preserving operations.
- Systems where filters are composed dynamically.

## 9. Where It Fails

- Simple one-off filtering.
- Very large datasets that should be filtered in a database.
- Cases where Java Streams already express everything clearly.
- Filters with side effects.
- Complex business rules better modeled as Specification.

## 10. Prebuilt Libraries and Packages

### Java

- `java.util.stream.Stream`
- `Predicate`
- Spring Data query APIs
- QueryDSL
- JPA Criteria API
- Guava immutable collections

### Python

- `filter`
- List comprehensions.
- Generator expressions.
- Pandas filtering.
- Query libraries and ORMs.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Preserves domain type after filtering. | Can duplicate stream/query APIs. |
| Protects internal collections. | May create many intermediate objects. |
| Supports fluent filter composition. | Not ideal for database-sized data. |
| Keeps filtering close to domain model. | Predicate side effects can surprise callers. |

## 12. Real-World Identification Example

Scenario:

You are building a security dashboard that groups alerts by system.

Without Filterer:

- Every screen pulls alert lists and filters manually.
- Type information is lost after filtering.

With Filterer:

- `ThreatAwareSystem.filtered().by(...)` returns a filtered system.
- Screens still work with system-level behavior.
- Predicates are reusable for severity, source, and probability.

## 13. MAANG Interview Triggers

Use Filterer when you hear:

- "Filter a domain container."
- "Return a filtered version of the same object."
- "Avoid exposing internal collection."
- "Compose filters dynamically."
- "Keep typed result after filtering."
- "Fluent query-like API."

### Interview-Ready Answer Format

1. Identify the container and element types.
2. Keep the container immutable or protected.
3. Define a filter API that accepts predicates.
4. Return a filtered container, not raw internals.
5. Discuss performance and database pushdown.
6. Compare with Stream and Specification.

## 14. Common Mistakes

### Mistake 1: Filtering In Memory Too Late

For huge datasets, push filtering to the database or search engine.

### Mistake 2: Returning Mutable Internal Lists

That breaks encapsulation and can corrupt the container.

### Mistake 3: Filterer Adds No Domain Value

Use streams directly if a typed wrapper does not help.

### Mistake 4: Side-Effect Predicates

Predicates should be pure whenever possible.

### Mistake 5: Losing Generic Type Safety

Design generic bounds carefully when subtypes are involved.

## 15. Filterer vs Similar Patterns

| Pattern | Difference |
|---|---|
| Filterer | Returns filtered domain container. |
| Stream | General data pipeline over elements. |
| Iterator | Traverses elements but does not define filter semantics. |
| Specification | Encapsulates business rules as reusable predicates. |
| Chain of Responsibility | Passes request through handlers, not element filtering. |

## 16. Filterer Design Checklist

| Question | Why it matters |
|---|---|
| What is the container type? | Determines returned domain object. |
| What is the element type? | Defines predicate input. |
| Is result immutable? | Prevents accidental mutation. |
| Should filtering be lazy? | Matters for large collections. |
| Should filters run in database? | Avoids memory and latency problems. |

## 17. Quick Revision Notes

- Filterer filters container-like domain objects.
- It returns a filtered object, not just a list.
- Predicates define filter criteria.
- Great for domain collections.
- Avoid for huge datasets unless filtering is pushed down.
- Compare with Stream and Specification.

## 18. Mini Exercise

Design Filterer for `OrderBook`.

Filters:

- Orders above a price.
- Orders for a symbol.
- Orders with status `OPEN`.

Expected usage:

```java
OrderBook openTechOrders = orderBook
    .filtered().by(Order::isOpen)
    .filtered().by(order -> order.symbol().equals("TECH"));
```

## 19. Source Reference in This Repo

The repository's Filterer implementation models threat-aware systems that return filtered versions of themselves.

Useful files:

- [github-repo/filterer/README.md](../../github-repo/filterer/README.md)
- [github-repo/filterer/src/main/java/com/iluwatar/filterer/domain/Filterer.java](../../github-repo/filterer/src/main/java/com/iluwatar/filterer/domain/Filterer.java)
- [github-repo/filterer/src/main/java/com/iluwatar/filterer/threat/ThreatAwareSystem.java](../../github-repo/filterer/src/main/java/com/iluwatar/filterer/threat/ThreatAwareSystem.java)
- [github-repo/filterer/src/main/java/com/iluwatar/filterer/threat/SimpleThreatAwareSystem.java](../../github-repo/filterer/src/main/java/com/iluwatar/filterer/threat/SimpleThreatAwareSystem.java)
- [github-repo/filterer/src/main/java/com/iluwatar/filterer/App.java](../../github-repo/filterer/src/main/java/com/iluwatar/filterer/App.java)

