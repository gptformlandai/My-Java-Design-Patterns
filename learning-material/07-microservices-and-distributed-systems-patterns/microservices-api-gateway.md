# Microservices API Gateway Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/microservices-api-gateway](../../github-repo/microservices-api-gateway)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the gateway as the single client-facing entry point.
2. Second pass: trace routing, authentication, aggregation, rate limiting, and fallback behavior.
3. Third pass: compare API Gateway with load balancer, reverse proxy, BFF, and service mesh.

By the end, you should be able to say:

> An API Gateway hides internal microservice complexity behind one stable client-facing API.

## 1. Technical Definition

Microservices API Gateway is an architectural pattern where a gateway service receives client requests, applies cross-cutting policies, routes calls to backend services, and may aggregate or transform responses.

Core idea:

- Clients call one endpoint.
- The gateway routes to many internal services.
- Cross-cutting concerns are centralized.
- Client-specific APIs can be shaped at the edge.
- Internal services can evolve without exposing every detail to clients.

### 30-Second Interview Answer

I would use an API Gateway when clients need a stable, secure, simplified entry point into many microservices. The gateway handles routing, authentication, rate limiting, request shaping, response aggregation, and sometimes protocol translation. The trade-off is that the gateway becomes critical infrastructure, so it must be highly available, horizontally scalable, observable, and kept thin enough to avoid becoming a business-logic dumping ground.

## 2. Layman and Easy to Understand Definition

An API Gateway is like the front desk of a large office.

Visitors do not need to know every department. They go to the front desk, and the front desk sends them to the right place or collects answers from multiple departments.

In software:

- Client talks to gateway.
- Gateway talks to services.
- Services stay internal.
- Gateway returns the response.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without a gateway, clients may need to call many services directly:

```text
mobile app -> product service
mobile app -> price service
mobile app -> inventory service
mobile app -> review service
```

This creates problems:

- Client knows too much about backend topology.
- Authentication logic gets duplicated.
- Mobile clients make many network calls.
- Backend service changes break clients.
- Rate limits and observability are scattered.

### 3.2 The API Gateway Solution

Put a controlled edge service in front:

```text
client -> api gateway -> internal services
```

The gateway becomes the public API surface.

### 3.3 Main Responsibilities

| Responsibility | Example |
|---|---|
| Routing | `/orders` goes to order service. |
| Authentication | Validate JWT or session. |
| Authorization | Check scopes and roles. |
| Aggregation | Combine product, price, and image data. |
| Rate limiting | Protect backend services. |
| Transformation | Convert response shape for mobile or desktop. |
| Observability | Emit logs, metrics, and traces at the edge. |

### 3.4 Request Flow

1. Client sends request to gateway.
2. Gateway authenticates the caller.
3. Gateway checks rate limits and policy.
4. Gateway routes to one or more backend services.
5. Gateway aggregates or transforms responses.
6. Gateway returns a client-friendly response.
7. Gateway records metrics and traces.

## 4. Java Coding Example

This example shows a gateway aggregating product details from price and image services.

```java
record ProductView(String id, String price, String imageUrl) {
}

interface PriceClient {
    String priceFor(String productId);
}

interface ImageClient {
    String imageFor(String productId);
}

class ProductGateway {
    private final PriceClient priceClient;
    private final ImageClient imageClient;

    ProductGateway(PriceClient priceClient, ImageClient imageClient) {
        this.priceClient = priceClient;
        this.imageClient = imageClient;
    }

    ProductView desktopProduct(String productId) {
        String price = priceClient.priceFor(productId);
        String image = imageClient.imageFor(productId);
        return new ProductView(productId, price, image);
    }

    ProductView mobileProduct(String productId) {
        String price = priceClient.priceFor(productId);
        return new ProductView(productId, price, null);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `ProductGateway` | Gateway owns the client-facing operation. |
| `PriceClient` | Internal service call hidden from client. |
| `ImageClient` | Another internal dependency. |
| `desktopProduct` | Aggregates data for a rich client. |
| `mobileProduct` | Returns a smaller client-specific response. |

### Java Usage

```java
ProductGateway gateway = new ProductGateway(
    productId -> "$49",
    productId -> "https://cdn.example.com/" + productId + ".png"
);

System.out.println(gateway.desktopProduct("p-100"));
```

## 5. Python Coding Example

```python
class ProductGateway:
    def __init__(self, price_client, image_client):
        self.price_client = price_client
        self.image_client = image_client

    def desktop_product(self, product_id):
        return {
            "id": product_id,
            "price": self.price_client(product_id),
            "imageUrl": self.image_client(product_id),
        }

    def mobile_product(self, product_id):
        return {
            "id": product_id,
            "price": self.price_client(product_id),
        }


gateway = ProductGateway(
    price_client=lambda product_id: "$49",
    image_client=lambda product_id: f"https://cdn.example.com/{product_id}.png",
)

print(gateway.desktop_product("p-100"))
```

### Python Usage

Use this shape when explaining:

- One public operation can call multiple internal services.
- Different clients can receive different response shapes.
- Backend service URLs remain hidden from clients.

## 6. Where It Comes Handy in Real Life

- E-commerce product detail page.
- Mobile app backend.
- Public API for internal microservices.
- Partner API with strict policy controls.
- Legacy clients that need stable endpoints.
- Edge service for authentication and rate limiting.

## 7. Advantages Over Normal Code Without Pattern

Without API Gateway:

```text
client calls every backend service directly
```

With API Gateway:

```text
client calls gateway; gateway coordinates backend calls
```

Benefits:

- Simplifies clients.
- Centralizes edge policies.
- Reduces client round trips.
- Hides internal service topology.
- Enables client-specific response shaping.

## 8. Where It Excels

- Many microservices serve one user journey.
- Mobile clients need fewer calls.
- Security and rate limiting must be enforced consistently.
- Backend services change faster than clients.
- Public API must remain stable.

## 9. Where It Fails

- Gateway contains too much business logic.
- Gateway is deployed as a single instance.
- Gateway synchronously calls too many services.
- One slow dependency slows the entire response.
- Gateway team becomes a release bottleneck.

Keep the gateway thin: edge policy, routing, composition, and client shaping.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Spring Cloud Gateway, Netflix Zuul, Micronaut Gateway patterns |
| Cloud | AWS API Gateway, Azure API Management, Google API Gateway |
| Edge proxy | Kong, NGINX, Envoy, Traefik |
| Service mesh edge | Istio ingress gateway, Linkerd with ingress |
| Observability | OpenTelemetry, Prometheus, Grafana |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Single client-facing entry point. | Can become a bottleneck. |
| Central security and rate limiting. | Adds another critical service. |
| Hides internal topology. | Can collect too much business logic. |
| Reduces client round trips. | Aggregation can increase tail latency. |

## 12. Real-World Identification Example

Scenario: Product page needs product name, price, image, inventory, and recommendation data.

API Gateway fit:

- Browser calls `/product-page/{id}`.
- Gateway fans out to backend services.
- Gateway applies auth, rate limits, and tracing.
- Gateway returns a page-ready DTO.

Without it:

- Browser makes many calls.
- Backend service topology leaks to the client.
- Each service must enforce edge policies independently.

## 13. MAANG Interview Triggers

Use API Gateway when you hear:

- "Many microservices behind one app."
- "Mobile app needs fewer round trips."
- "How do clients discover services?"
- "Where do authentication and rate limiting live?"
- "How do we expose public APIs safely?"
- "How do we aggregate data from many services?"

Strong answer keywords:

- routing
- aggregation
- BFF
- authn/authz
- rate limiting
- caching
- circuit breaker
- timeouts
- tracing

## 14. Common Mistakes

### Mistake 1: Turning gateway into a monolith

- Why it is wrong: business logic accumulates at the edge.
- Better approach: keep domain rules inside owning services.

### Mistake 2: No timeout budget

- Why it is wrong: one slow dependency can exhaust request threads.
- Better approach: enforce per-call timeouts, retries with limits, and fallbacks.

### Mistake 3: One generic gateway for every client need

- Why it is wrong: desktop, mobile, and partner clients may need different shapes.
- Better approach: use BFF variants when client needs diverge.

### Mistake 4: Ignoring gateway availability

- Why it is wrong: if the gateway fails, every client path fails.
- Better approach: run multiple instances across zones with health checks.

## 15. API Gateway vs Similar Patterns

| Pattern | Difference |
|---|---|
| API Gateway | Client-facing service that routes, secures, and composes APIs. |
| Load Balancer | Distributes traffic across instances of the same service. |
| Reverse Proxy | Forwards requests; may not know application-specific composition. |
| BFF | Client-specific API gateway variant. |
| Service Mesh | Handles service-to-service networking inside the cluster. |
| Facade | General simplification interface; gateway is a distributed facade. |

## 16. API Gateway Design Checklist

- Which clients use the gateway?
- Which routes are public?
- Where is authentication validated?
- What are rate limits per user, app, and endpoint?
- What backend calls are aggregated?
- What timeout budget applies to the full request?
- What fallback response is acceptable?
- How are traces propagated?
- How is the gateway scaled and deployed?

## 17. Quick Revision Notes

- One-line summary: API Gateway is the secure, client-facing entry point into microservices.
- Three keywords: routing, aggregation, policy.
- Interview trap: putting core domain logic inside the gateway.
- Memory trick: one front door, many backend rooms.

## 18. Mini Exercise

Design an API Gateway endpoint for a food delivery order screen.

Answer these:

1. Which backend services does it call?
2. Which fields are needed by mobile?
3. What timeout does each backend get?
4. What data can be cached?
5. What happens if recommendation service fails?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/microservices-api-gateway/README.md](../../github-repo/microservices-api-gateway/README.md)
- [github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/ApiGateway.java](../../github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/ApiGateway.java)
- [github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/ImageClient.java](../../github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/ImageClient.java)
- [github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/PriceClient.java](../../github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/PriceClient.java)
- [github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/DesktopProduct.java](../../github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/DesktopProduct.java)
- [github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/MobileProduct.java](../../github-repo/microservices-api-gateway/api-gateway-service/src/main/java/com/iluwatar/api/gateway/MobileProduct.java)
- [github-repo/microservices-api-gateway/image-microservice/src/main/java/com/iluwatar/image/microservice/ImageController.java](../../github-repo/microservices-api-gateway/image-microservice/src/main/java/com/iluwatar/image/microservice/ImageController.java)
- [github-repo/microservices-api-gateway/price-microservice/src/main/java/com/iluwatar/price/microservice/PriceController.java](../../github-repo/microservices-api-gateway/price-microservice/src/main/java/com/iluwatar/price/microservice/PriceController.java)
