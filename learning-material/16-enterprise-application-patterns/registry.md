# Registry Pattern

Category: Enterprise Application Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [registry](../../github-repo/registry)

---

## How to Study This Page

Study Registry as "a well-known place to register and retrieve objects by key."

Remember the core idea:

```text
key -> registry -> object
```

The pattern is useful, but it can also become hidden global state. That tension is the mature interview answer.

---

## 1. Technical Definition

Registry is an enterprise application pattern that maintains a central mapping from keys to objects so other parts of the application can register, retrieve, and reuse shared instances.

### 30-Second Interview Answer

Registry provides a central lookup point for objects by key. It can help manage shared resources, plugins, handlers, or named objects without every caller constructing them directly. I would use it for framework-level registration, plugin maps, or controlled shared object lookup. The trade-off is that it can become global mutable state, hide dependencies, and make testing harder if overused.

---

## 2. Layman and Easy to Understand Definition

Think of a company directory.

You do not need to know where every employee sits. You search the directory by name or employee ID and get the right contact.

A Registry is that directory for application objects.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Some objects need to be shared or found by name:

```text
"pdfExporter" -> PdfExporter
"emailTemplate" -> EmailTemplate
"customer-123" -> Customer
```

Passing every object everywhere can be noisy, but constructing duplicates can be wrong or expensive.

### Registry Flow

1. Create a registry object.
2. Register objects with keys.
3. Caller asks the registry for an object by key.
4. Registry returns the existing object or `null`/optional if missing.
5. Registry may enforce lifecycle, uniqueness, or thread-safety rules.

### Core Participants

| Participant | Responsibility |
|---|---|
| Registry | Stores key-to-object mappings |
| Key | Identifier used for lookup |
| Registered object | Shared object being stored |
| Registrar | Code that adds objects |
| Client | Code that retrieves objects |
| Lifecycle policy | Rules for creation, replacement, and removal |

---

## 4. Java Coding Example

```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

interface Handler {
    void handle(String input);
}

class HandlerRegistry {
    private final Map<String, Handler> handlers = new ConcurrentHashMap<>();

    void register(String key, Handler handler) {
        handlers.put(key, handler);
    }

    Handler get(String key) {
        return handlers.get(key);
    }
}

public class RegistryDemo {
    public static void main(String[] args) {
        HandlerRegistry registry = new HandlerRegistry();
        registry.register("email", input -> System.out.println("email: " + input));

        registry.get("email").handle("welcome");
    }
}
```

### Java Block by Block

`HandlerRegistry` owns the key-to-handler map.

`register` adds a handler.

`get` retrieves the handler by key.

The caller does not construct the handler directly.

---

## 5. Python Coding Example

```python
class Registry:
    def __init__(self):
        self.items = {}

    def register(self, key, value):
        self.items[key] = value

    def get(self, key):
        return self.items.get(key)


registry = Registry()
registry.register("email", lambda value: print("email:", value))
registry.get("email")("welcome")
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Plugin systems | Register handlers by plugin name |
| Serializer registry | Map content type to serializer |
| Command handlers | Map command name to handler |
| Shared resources | Reuse named resources under controlled rules |
| Framework extension points | Discover and store extensions |
| Test fixtures | Register fake implementations by key |

---

## 7. Advantages Over Normal Code Without Pattern

Without Registry:
- object lookup may be scattered
- duplicate instances may be created accidentally
- plugins or handlers need manual wiring everywhere
- key-based lookup logic is repeated

With Registry:
- lookup logic is centralized
- objects can be reused by key
- registration can be controlled
- frameworks can support dynamic extension points

---

## 8. Where It Excels

It excels when:
- objects are naturally identified by key
- object instances should be shared or reused
- registration happens during startup or configuration
- callers need lookup without knowing construction details
- framework extension points are needed
- lifecycle rules are well-defined

---

## 9. Where It Fails

It fails when:
- it becomes a dumping ground for global state
- dependencies become hidden
- test setup requires global registry mutation
- keys are stringly typed and error-prone
- lifecycle and replacement rules are unclear
- dependency injection would be clearer

Use constructor injection for normal application dependencies. Reserve Registry for genuine keyed lookup or framework registration.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Maps, `ServiceLoader`, enum registries |
| Spring | Bean registry/ApplicationContext, though DI is preferred for dependencies |
| Jakarta | JNDI registry-style lookup |
| Plugin systems | Java SPI, PF4J |
| Python | dictionaries, plugin registries, entry points |
| Serialization | Jackson module/type registries |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Centralized keyed lookup | Can hide dependencies |
| Supports shared objects | Can become global mutable state |
| Useful for plugins and handlers | String keys can be fragile |
| Reduces duplicate creation | Lifecycle rules can get complex |
| Easy to extend dynamically | Testing needs isolation/reset strategy |

---

## 12. Real-World Identification Example

Question:

> A reporting framework supports many exporters: PDF, CSV, JSON, and Excel. New exporters should be registered at startup and looked up by format name. What pattern helps?

Strong answer:

Use Registry. Create an exporter registry keyed by format, register each exporter during startup, and let report code retrieve the exporter for the requested format. I would avoid using it as a general dependency container and keep registration lifecycle explicit.

---

## 13. MAANG Interview Triggers

Say Registry when you hear:
- central object lookup
- key to object map
- plugin registration
- handler registry
- shared named objects
- service registry
- global lookup point
- framework extension point

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Using registry for all dependencies | Hides dependencies | Prefer dependency injection |
| Mutable global registry in tests | Tests affect each other | Reset or use scoped registries |
| String keys everywhere | Typos fail at runtime | Use constants, enums, or typed keys |
| No lifecycle policy | Replacement/removal is unclear | Define ownership and lifecycle rules |
| No thread safety | Concurrent access corrupts state | Use thread-safe data structures when needed |

---

## 15. Registry vs Similar Patterns

| Pattern | Difference |
|---|---|
| Service Locator | Service Locator finds/creates services; Registry mainly stores and returns registered objects |
| Singleton | Singleton ensures one instance; Registry maps many keys to objects |
| Factory | Factory creates objects; Registry retrieves registered objects |
| Dependency Injection | DI provides dependencies explicitly; Registry is pulled from by clients |
| Multiton | Multiton controls one instance per key; Registry may store arbitrary objects per key |

---

## 16. Registry Design Checklist

- What objects are registered?
- What is the key type?
- Who registers objects?
- Who reads from the registry?
- Is lookup allowed to return missing?
- Are objects mutable or shared safely?
- Is the registry global or scoped?
- How is it reset in tests?
- Would dependency injection be clearer?

---

## 17. Quick Revision Notes

- One-line summary: Central key-to-object lookup store.
- Memory hook: "application directory."
- Best for: plugins, handlers, serializers, named shared objects.
- Avoid when: it merely hides ordinary dependencies.
- Interview line: "I would use Registry for keyed extension lookup, not as a general replacement for dependency injection."

---

## 18. Mini Exercise

Design a registry for notification handlers:
- choose key type for email, SMS, and push
- define handler interface
- register three handlers
- retrieve one handler by key
- explain how tests avoid global state leakage

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/registry/README.md)
- [Customer.java](../../github-repo/registry/src/main/java/com/iluwatar/registry/Customer.java)
- [CustomerRegistry.java](../../github-repo/registry/src/main/java/com/iluwatar/registry/CustomerRegistry.java)
- [App.java](../../github-repo/registry/src/main/java/com/iluwatar/registry/App.java)

The repo implementation uses a singleton `CustomerRegistry` backed by a `ConcurrentHashMap`. Customers are registered by ID and retrieved later through the registry.
