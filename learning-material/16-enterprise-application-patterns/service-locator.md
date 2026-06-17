# Service Locator Pattern

Category: Enterprise Application Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [service-locator](../../github-repo/service-locator)

---

## How to Study This Page

Study Service Locator as "ask a central object to find a service for you."

Remember the core flow:

```text
client -> service locator -> cache or lookup context -> service
```

Also remember the interview caveat: Service Locator can hide dependencies, so dependency injection is usually preferred for normal application code.

---

## 1. Technical Definition

Service Locator is an enterprise application pattern that provides a centralized lookup mechanism for services, often using a cache and external lookup context to return service instances by name or type.

### 30-Second Interview Answer

Service Locator centralizes how clients obtain services. A client asks the locator for a service by name or type; the locator checks a cache or registry, creates or looks up the service if needed, and returns it. It can decouple clients from service creation and expensive lookup details. The trade-off is hidden dependencies and harder testing, so modern applications often prefer dependency injection unless dynamic lookup is genuinely needed.

---

## 2. Layman and Easy to Understand Definition

Think of a hotel concierge.

Guests do not directly find every driver, tour guide, or restaurant contact. They ask the concierge, and the concierge locates the right service.

Service Locator is that concierge for application services.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Clients may need services whose creation is expensive or environment-specific:

```text
client -> create service directly
client -> know JNDI name
client -> manage cache
client -> handle lookup failure
```

That spreads lookup logic across the codebase.

### Service Locator Flow

1. Client requests a service by name or type.
2. Locator checks whether the service is already cached.
3. If cached, locator returns it.
4. If not cached, locator asks a context/factory/registry to look it up.
5. Locator caches the service if appropriate.
6. Client executes the service without knowing lookup details.

### Core Participants

| Participant | Responsibility |
|---|---|
| Client | Requests a service |
| Service Locator | Central lookup API |
| Service interface | Contract returned to clients |
| Lookup context | Creates or finds services |
| Cache | Reuses located services |
| Concrete service | Performs the actual work |

---

## 4. Java Coding Example

```java
import java.util.HashMap;
import java.util.Map;

interface PaymentService {
    void pay();
}

class CardPaymentService implements PaymentService {
    public void pay() {
        System.out.println("paid by card");
    }
}

class ServiceLocator {
    private final Map<String, Object> cache = new HashMap<>();

    PaymentService getPaymentService(String name) {
        return (PaymentService) cache.computeIfAbsent(name, key -> new CardPaymentService());
    }
}

public class ServiceLocatorDemo {
    public static void main(String[] args) {
        ServiceLocator locator = new ServiceLocator();
        PaymentService service = locator.getPaymentService("card");
        service.pay();
    }
}
```

### Java Block by Block

`ServiceLocator` hides creation and caching.

Client asks for `"card"` and receives a service.

The service is returned through an interface.

In real applications, use typed keys or DI when possible.

---

## 5. Python Coding Example

```python
class CardPaymentService:
    def pay(self):
        print("paid by card")


class ServiceLocator:
    def __init__(self):
        self.cache = {}

    def get_payment_service(self, name):
        if name not in self.cache:
            self.cache[name] = CardPaymentService()
        return self.cache[name]


locator = ServiceLocator()
locator.get_payment_service("card").pay()
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Legacy enterprise apps | Centralized JNDI/service lookup |
| Plugin systems | Services discovered dynamically |
| Expensive lookup | Cache service instances after first lookup |
| Framework internals | Resolve services based on runtime metadata |
| Modular systems | Service name/type may be known only at runtime |
| Migration code | Bridge old lookup style to newer DI style |

---

## 7. Advantages Over Normal Code Without Pattern

Without Service Locator:
- lookup code is duplicated
- clients know construction details
- expensive services may be repeatedly created
- service caching is inconsistent

With Service Locator:
- lookup is centralized
- clients use service interfaces
- caching can be handled in one place
- lookup mechanisms can change behind the locator

---

## 8. Where It Excels

It excels when:
- services must be discovered dynamically
- lookup is expensive or environment-specific
- service names are known at runtime
- clients should not know lookup mechanics
- framework code needs late binding
- a cache improves repeated lookup performance

---

## 9. Where It Fails

It fails when:
- it hides normal constructor dependencies
- tests need to mock global locator state
- missing services fail only at runtime
- string keys are fragile
- clients become coupled to the locator API
- dependency injection would make dependencies explicit

Prefer dependency injection for application services with known dependencies. Use Service Locator mostly for dynamic lookup or framework infrastructure.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java/Jakarta | JNDI, Java `ServiceLoader` |
| Spring | `ApplicationContext#getBean`, though constructor injection is preferred |
| OSGi | Service registry lookup |
| MicroProfile | CDI/service discovery concepts |
| Python | plugin managers, entry points |
| Legacy enterprise | EJB/JNDI service lookup layers |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Centralizes lookup logic | Hides dependencies |
| Can cache expensive services | Runtime key errors |
| Decouples clients from concrete construction | Harder to test than DI |
| Useful for dynamic discovery | Can become global state |
| Works in legacy enterprise systems | Can obscure object graph |

---

## 12. Real-World Identification Example

Question:

> A legacy enterprise app looks up remote services through JNDI in many places. Each class repeats lookup and caching logic. What pattern helps?

Strong answer:

Use Service Locator as a central lookup layer. Clients ask the locator for a service by name or type; the locator checks cache first, performs JNDI lookup when needed, and returns the service interface. I would keep this scoped and consider migrating known dependencies to dependency injection to make dependencies explicit.

---

## 13. MAANG Interview Triggers

Say Service Locator when you hear:
- central service lookup
- JNDI lookup
- service cache
- dynamic service discovery
- client asks for service
- hide service creation
- legacy enterprise service access
- locator versus dependency injection

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Using locator everywhere | Dependencies become hidden | Prefer constructor injection |
| String service names | Typos fail at runtime | Use typed keys/constants |
| Global mutable cache | Tests become order-dependent | Use scoped locator or reset cache |
| No failure handling | Missing services cause null errors | Return optional or throw clear exception |
| Caching non-thread-safe services | Shared instance may corrupt state | Understand service lifecycle and thread safety |

---

## 15. Service Locator vs Similar Patterns

| Pattern | Difference |
|---|---|
| Registry | Registry stores keyed objects; Service Locator actively finds services and may cache them |
| Dependency Injection | DI pushes dependencies into clients; Service Locator clients pull dependencies |
| Factory | Factory creates objects; Service Locator locates services, often from cache/context |
| Singleton | Singleton controls one instance; Service Locator is a lookup mechanism |
| Repository | Repository accesses domain data; Service Locator accesses service instances |

---

## 16. Service Locator Design Checklist

- What service is being located?
- Is lookup dynamic or known at compile time?
- Would dependency injection be clearer?
- What key or type identifies the service?
- Is the key type-safe?
- Should located services be cached?
- Are services thread-safe if cached?
- How are missing services handled?
- How will tests replace or reset services?

---

## 17. Quick Revision Notes

- One-line summary: Central object that finds and returns services for clients.
- Memory hook: "concierge for services."
- Best for: dynamic lookup, legacy JNDI, framework internals, plugin discovery.
- Avoid when: constructor injection can make dependencies explicit.
- Interview line: "I would use Service Locator only when runtime lookup is required; otherwise DI is cleaner and more testable."

---

## 18. Mini Exercise

Design a service locator for report exporters:
- define exporter service interface
- support lookup by format
- cache created exporters
- handle missing format cleanly
- explain why DI might be better for fixed exporters

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/service-locator/README.md)
- [Service.java](../../github-repo/service-locator/src/main/java/com/iluwatar/servicelocator/Service.java)
- [ServiceImpl.java](../../github-repo/service-locator/src/main/java/com/iluwatar/servicelocator/ServiceImpl.java)
- [ServiceLocator.java](../../github-repo/service-locator/src/main/java/com/iluwatar/servicelocator/ServiceLocator.java)
- [ServiceCache.java](../../github-repo/service-locator/src/main/java/com/iluwatar/servicelocator/ServiceCache.java)
- [InitContext.java](../../github-repo/service-locator/src/main/java/com/iluwatar/servicelocator/InitContext.java)
- [App.java](../../github-repo/service-locator/src/main/java/com/iluwatar/servicelocator/App.java)

The repo implementation asks `ServiceLocator` for JNDI-style service names. The locator checks `ServiceCache`, uses `InitContext` to create missing services, caches them, and returns the `Service` interface to the caller.
