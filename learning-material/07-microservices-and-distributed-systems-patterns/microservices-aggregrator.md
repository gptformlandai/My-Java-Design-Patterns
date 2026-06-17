# Microservices Aggregator Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/microservices-aggregrator](../../github-repo/microservices-aggregrator)

## How to Study This Page

Use this page in three passes:

1. First pass: understand Aggregator as API composition across multiple services.
2. Second pass: trace fan-out, partial failure, fallback, and final response assembly.
3. Third pass: compare Aggregator with API Gateway, BFF, GraphQL, and CQRS read models.

By the end, you should be able to say:

> Microservices Aggregator collects data from multiple services and returns one composed response.

## 1. Technical Definition

Microservices Aggregator is an API composition pattern where one service calls multiple backend microservices, combines their responses, and returns a unified result to the caller.

Core idea:

- Caller makes one request.
- Aggregator fans out to multiple services.
- Responses are combined into one view model.
- Partial failures need fallback behavior.
- Client stays decoupled from internal service topology.

### 30-Second Interview Answer

I would use Microservices Aggregator when a client screen or API response needs data owned by multiple services. The aggregator calls those services, applies timeouts and fallbacks, then returns a composed DTO. The trade-off is that aggregation adds latency and can become a bottleneck, so it needs parallel calls, caching, circuit breakers, and careful ownership boundaries.

## 2. Layman and Easy to Understand Definition

Aggregator is like asking one person to collect updates from multiple teams and give you one summary.

You do not call every team yourself. The aggregator asks each team, combines the answers, and gives you one response.

In code:

- Client calls aggregator.
- Aggregator calls service A and service B.
- Aggregator merges results.
- Client receives one response.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

A client view often needs data from multiple services:

```text
product title -> product information service
inventory count -> inventory service
price -> pricing service
reviews -> review service
```

If the client calls every service directly, it becomes tightly coupled and slower.

### 3.2 The Aggregator Solution

Add a composition service:

```text
client -> aggregator -> information service
                   -> inventory service
                   -> price service
```

The aggregator returns a client-ready response.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Client | Needs a combined response. |
| Aggregator | Calls multiple services and composes data. |
| Downstream service | Owns one part of the data. |
| View DTO | Combined response object. |
| Fallback | Default or partial value when a dependency fails. |

### 3.4 Request Flow

1. Client requests a composed resource.
2. Aggregator validates request.
3. Aggregator calls downstream services.
4. Aggregator applies timeouts and fallbacks.
5. Aggregator combines responses.
6. Aggregator returns one DTO.

## 4. Java Coding Example

This example aggregates product title and inventory into one response.

```java
record ProductView(String title, int inventory) {
}

interface ProductInformationClient {
    String title();
}

interface ProductInventoryClient {
    Integer inventory();
}

class ProductAggregator {
    private final ProductInformationClient informationClient;
    private final ProductInventoryClient inventoryClient;

    ProductAggregator(ProductInformationClient informationClient,
                      ProductInventoryClient inventoryClient) {
        this.informationClient = informationClient;
        this.inventoryClient = inventoryClient;
    }

    ProductView product() {
        String title = informationClient.title();
        Integer inventory = inventoryClient.inventory();

        return new ProductView(
            title == null ? "Unknown product" : title,
            inventory == null ? -1 : inventory
        );
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `ProductView` | Combined client-facing DTO. |
| `ProductInformationClient` | One downstream data owner. |
| `ProductInventoryClient` | Another downstream data owner. |
| `ProductAggregator` | Composes responses. |
| fallback values | Handles partial downstream failure. |

### Java Usage

```java
ProductAggregator aggregator = new ProductAggregator(
    () -> "Keyboard",
    () -> 12
);

System.out.println(aggregator.product());
```

## 5. Python Coding Example

```python
class ProductAggregator:
    def __init__(self, information_client, inventory_client):
        self.information_client = information_client
        self.inventory_client = inventory_client

    def product(self):
        title = self.information_client()
        inventory = self.inventory_client()
        return {
            "title": title or "Unknown product",
            "inventory": inventory if inventory is not None else -1,
        }


aggregator = ProductAggregator(
    information_client=lambda: "Keyboard",
    inventory_client=lambda: 12,
)

print(aggregator.product())
```

### Python Usage

Use this shape when explaining:

- Aggregator owns response composition.
- Downstream services still own their data.
- Partial results need explicit fallback behavior.

## 6. Where It Comes Handy in Real Life

- Product detail pages.
- Dashboard APIs.
- Travel search results.
- User profile summary.
- Order detail screen.
- Admin console views.

## 7. Advantages Over Normal Code Without Pattern

Without Aggregator:

```text
client calls many services and stitches data itself
```

With Aggregator:

```text
client calls one service and receives a composed response
```

Benefits:

- Simplifies clients.
- Reduces client round trips.
- Hides backend topology.
- Centralizes composition logic.
- Supports fallback and partial responses.

## 8. Where It Excels

- One screen needs multiple service-owned fields.
- Client networks are slow or unreliable.
- Backend services change independently.
- You need a stable response shape.
- Aggregation logic is mostly read-only.

## 9. Where It Fails

- Aggregator starts owning domain logic.
- Fan-out calls are made serially and increase latency.
- Too many dependencies make response availability poor.
- Aggregator becomes a single bottleneck.
- Data needs complex cross-service transactions.

Prefer parallel calls and strict timeout budgets.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Spring Boot, WebClient, CompletableFuture, Reactor |
| API composition | GraphQL, Apollo Federation, Netflix DGS |
| Resilience | Resilience4j, circuit breakers, bulkheads |
| Caching | Redis, Caffeine, CDN edge caching |
| Observability | OpenTelemetry traces across fan-out calls |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Simplifies client integration. | Adds another service hop. |
| Reduces client-side fan-out. | Can increase tail latency. |
| Encapsulates composition logic. | Can become a bottleneck. |
| Supports fallback responses. | Ownership boundaries can blur. |

## 12. Real-World Identification Example

Scenario: Product page needs title, inventory, price, and seller rating.

Aggregator fit:

- Client calls `/product-summary/{id}`.
- Aggregator calls information, inventory, pricing, and reputation services.
- Aggregator returns a page-ready response.
- If rating service fails, response still includes core product data.

Without it:

- Client calls many services.
- Mobile latency increases.
- Service URLs and contracts leak to clients.

## 13. MAANG Interview Triggers

Use Aggregator when you hear:

- "One page needs data from many services."
- "How do we reduce client round trips?"
- "How do we compose microservice responses?"
- "How do we handle partial failure in a read API?"
- "Should the client call all services directly?"

Strong answer keywords:

- API composition
- fan-out
- fallback
- timeout budget
- parallel calls
- DTO
- circuit breaker
- cache

## 14. Common Mistakes

### Mistake 1: Sequential fan-out calls

- Why it is wrong: latency becomes the sum of all dependencies.
- Better approach: call independent services in parallel.

### Mistake 2: No partial failure behavior

- Why it is wrong: one optional service can fail the whole response.
- Better approach: define required versus optional dependencies.

### Mistake 3: Owning downstream business rules

- Why it is wrong: data ownership becomes unclear.
- Better approach: aggregator composes, owning services decide domain truth.

### Mistake 4: No caching strategy

- Why it is wrong: hot views overload downstream services.
- Better approach: cache stable response pieces or final composed DTOs.

## 15. Aggregator vs Similar Patterns

| Pattern | Difference |
|---|---|
| Microservices Aggregator | Composes responses from multiple services. |
| API Gateway | Edge entry point for routing and policy; may include aggregation. |
| BFF | Client-specific backend, often includes aggregation. |
| GraphQL | Query language that can compose fields from many resolvers. |
| CQRS Read Model | Precomputed view optimized for reads instead of live fan-out. |

## 16. Aggregator Design Checklist

- What response does the client need?
- Which services own each field?
- Which dependencies are required?
- Which dependencies can fallback?
- Are calls parallel?
- What is the total timeout budget?
- What can be cached?
- How are traces propagated?
- Who owns the composed DTO contract?

## 17. Quick Revision Notes

- One-line summary: Aggregator combines multiple service responses into one view.
- Three keywords: fan-out, compose, fallback.
- Interview trap: letting aggregator become the owner of all business logic.
- Memory trick: one collector, many sources, one response.

## 18. Mini Exercise

Design an aggregator for a user dashboard.

Answer these:

1. Which services are called?
2. Which data is required versus optional?
3. What happens if one service times out?
4. Which calls can be parallel?
5. What response can be cached?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/microservices-aggregrator/README.md](../../github-repo/microservices-aggregrator/README.md)
- [github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/Aggregator.java](../../github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/Aggregator.java)
- [github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/Product.java](../../github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/Product.java)
- [github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/ProductInformationClient.java](../../github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/ProductInformationClient.java)
- [github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/ProductInventoryClient.java](../../github-repo/microservices-aggregrator/aggregator-service/src/main/java/com/iluwatar/aggregator/microservices/ProductInventoryClient.java)
- [github-repo/microservices-aggregrator/information-microservice/src/main/java/com/iluwatar/information/microservice/InformationController.java](../../github-repo/microservices-aggregrator/information-microservice/src/main/java/com/iluwatar/information/microservice/InformationController.java)
- [github-repo/microservices-aggregrator/inventory-microservice/src/main/java/com/iluwatar/inventory/microservice/InventoryController.java](../../github-repo/microservices-aggregrator/inventory-microservice/src/main/java/com/iluwatar/inventory/microservice/InventoryController.java)
