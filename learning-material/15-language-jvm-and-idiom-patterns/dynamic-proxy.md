# Dynamic Proxy Pattern

Category: Language, JVM, and Idiom Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [dynamic-proxy](../../github-repo/dynamic-proxy)

---

## How to Study This Page

Study Dynamic Proxy as "create an object at runtime that implements an interface and intercepts its method calls."

Remember the JVM flow:

```text
Interface method call -> generated proxy object -> InvocationHandler.invoke -> real behavior
```

The key is that no concrete implementation class is written by hand. The JVM generates a proxy for the interface at runtime.

---

## 1. Technical Definition

Dynamic Proxy is a JVM mechanism and design pattern where a proxy class implementing one or more interfaces is generated at runtime, and all method calls are routed through an `InvocationHandler`.

### 30-Second Interview Answer

Dynamic Proxy lets Java create an implementation of an interface at runtime. Calls on that proxy are intercepted by an `InvocationHandler`, which can add behavior such as logging, security, retries, transaction handling, remote calls, or mocks. I would use it when many interfaces need similar cross-cutting behavior and writing hand-coded proxies would be repetitive. The trade-off is reflection overhead, harder debugging, and the limitation that standard JDK dynamic proxies work with interfaces.

---

## 2. Layman and Easy to Understand Definition

Imagine a receptionist answering calls for many departments.

The caller thinks they are speaking directly to a department. In reality, every call first goes through the receptionist, who logs it, routes it, and maybe adds rules before forwarding it.

Dynamic Proxy is that runtime receptionist for interface method calls.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Many services need the same extra behavior:

```text
log before method
check permission
start transaction
call method
commit or rollback
log after method
```

Writing a separate proxy class for every interface is repetitive and brittle.

### Dynamic Proxy Flow

1. Define an interface.
2. Create an `InvocationHandler`.
3. Call `Proxy.newProxyInstance`.
4. JVM creates a runtime object that implements the interface.
5. Caller invokes interface methods on the proxy.
6. JVM routes every call to `InvocationHandler.invoke`.
7. Handler decides what real behavior to run.

### Core Participants

| Participant | Responsibility |
|---|---|
| Interface | Contract the proxy implements |
| Proxy instance | Runtime-generated object |
| InvocationHandler | Intercepts all method calls |
| Method metadata | Reflection object describing the called method |
| Arguments | Runtime method parameters |
| Target/dispatcher | Real object or logic used by the handler |

---

## 4. Java Coding Example

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

interface GreetingService {
    String greet(String name);
}

public class DynamicProxyDemo {
    public static void main(String[] args) {
        InvocationHandler handler = (proxy, method, arguments) -> {
            System.out.println("calling " + method.getName());
            if ("greet".equals(method.getName())) {
                return "Hello, " + arguments[0];
            }
            throw new UnsupportedOperationException(method.getName());
        };

        GreetingService service = (GreetingService) Proxy.newProxyInstance(
                GreetingService.class.getClassLoader(),
                new Class<?>[] {GreetingService.class},
                handler);

        System.out.println(service.greet("Aravind"));
    }
}
```

### Java Block by Block

`GreetingService` has no implementation class.

`InvocationHandler` receives every method call.

`Proxy.newProxyInstance` creates a runtime object that implements `GreetingService`.

Calling `service.greet` actually calls `handler.invoke`.

---

## 5. Python Coding Example

Python does not need JVM-style generated interface proxies, but the idea can be modeled with dynamic attribute interception.

```python
class DynamicProxy:
    def __init__(self, target):
        self.target = target

    def __getattr__(self, name):
        original = getattr(self.target, name)

        def wrapper(*args, **kwargs):
            print("calling", name)
            return original(*args, **kwargs)

        return wrapper


class GreetingService:
    def greet(self, name):
        return f"Hello, {name}"


service = DynamicProxy(GreetingService())
print(service.greet("Aravind"))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| AOP-style logging | Intercept many method calls uniformly |
| Security checks | Enforce access before method execution |
| Transactions | Begin, commit, or rollback around service methods |
| REST clients | Turn annotated interfaces into HTTP calls |
| Test mocks | Generate interface implementations dynamically |
| Lazy loading | Intercept access and load on demand |

---

## 7. Advantages Over Normal Code Without Pattern

Without Dynamic Proxy:
- each interface needs a hand-written wrapper
- cross-cutting behavior is duplicated
- adding logging/security/retries requires many edits
- framework-style abstractions are harder to build

With Dynamic Proxy:
- one handler can serve many methods
- behavior is added without changing interface clients
- runtime metadata can drive behavior
- framework code can generate implementations automatically

---

## 8. Where It Excels

It excels when:
- code depends on interfaces
- many methods need the same interception behavior
- runtime annotations or metadata drive behavior
- hand-written proxies would be repetitive
- frameworks need to generate implementations
- test doubles or clients can be interface-based

---

## 9. Where It Fails

It fails when:
- the type is a concrete class instead of an interface
- reflection overhead matters in a hot path
- method behavior needs compile-time safety
- debugging generated proxies becomes difficult
- `equals`, `hashCode`, and `toString` are not handled carefully
- runtime errors from annotations or method signatures are hard to catch

Use hand-written adapters, compile-time code generation, Byte Buddy, CGLIB, or direct implementations when the standard JDK proxy model is not enough.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java core | `java.lang.reflect.Proxy`, `InvocationHandler` |
| Spring | Spring AOP, transactional proxies, security proxies |
| Testing | Mockito, EasyMock |
| HTTP clients | Retrofit-style interfaces, Feign, MicroProfile REST Client |
| Bytecode tools | Byte Buddy, CGLIB, Javassist |
| Persistence | Hibernate lazy proxies |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids hand-written proxy classes | JDK proxy requires interfaces |
| Centralizes cross-cutting behavior | Reflection can complicate debugging |
| Enables framework-generated clients | Runtime errors may appear late |
| Good for mocks and interceptors | Performance overhead in hot paths |
| Keeps caller code interface-based | Invocation handler can become too generic |

---

## 12. Real-World Identification Example

Question:

> You want developers to define REST clients as annotated Java interfaces. At runtime, calling `userClient.findUser(7)` should build and send an HTTP request. What pattern helps?

Strong answer:

Use Dynamic Proxy. Define the API as an interface, read method annotations inside an `InvocationHandler`, and create a runtime proxy with `Proxy.newProxyInstance`. The handler receives method metadata and arguments, builds the HTTP request, sends it, and maps the response. I would validate annotations early and handle errors clearly because runtime reflection failures can be painful.

---

## 13. MAANG Interview Triggers

Say Dynamic Proxy when you hear:
- runtime interface implementation
- intercept method calls
- invocation handler
- AOP
- generated REST client
- transaction proxy
- mock interface dynamically
- Java reflection proxy

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Using for concrete classes | JDK proxy only implements interfaces | Use interface design or bytecode proxy tools |
| Ignoring `Object` methods | `toString` or `equals` may behave oddly | Handle them explicitly |
| Doing too much in one handler | Handler becomes a framework knot | Split dispatch, validation, and execution |
| No annotation validation | Errors appear during runtime calls | Validate at proxy creation |
| Using in hot loops blindly | Reflection overhead may matter | Benchmark or use generated code |

---

## 15. Dynamic Proxy vs Similar Patterns

| Pattern | Difference |
|---|---|
| Proxy | Dynamic Proxy is a runtime-generated proxy, usually for interfaces |
| Decorator | Decorator is hand-composed behavior wrapping; Dynamic Proxy intercepts calls generically |
| Adapter | Adapter translates interfaces; Dynamic Proxy implements interfaces dynamically |
| Intercepting Filter | Intercepts web request pipelines; Dynamic Proxy intercepts method calls |
| Facade | Facade simplifies a subsystem; Dynamic Proxy controls or generates behavior behind an interface |

---

## 16. Dynamic Proxy Design Checklist

- What interface will the proxy implement?
- What cross-cutting behavior is being intercepted?
- What does the handler do with method metadata?
- Are annotations or method signatures validated early?
- How are exceptions mapped?
- How are `equals`, `hashCode`, and `toString` handled?
- Is reflection overhead acceptable?
- Would compile-time code generation be safer?
- Can callers remain unaware of the proxy?

---

## 17. Quick Revision Notes

- One-line summary: JVM creates an interface implementation at runtime and routes calls to an invocation handler.
- Memory hook: "interface call becomes handler call."
- Best for: AOP, mocks, generated clients, transaction/security wrappers.
- Avoid when: concrete classes, hot paths, or compile-time safety are required.
- Interview line: "I would use a dynamic proxy when many interface methods need the same runtime interception logic."

---

## 18. Mini Exercise

Design a dynamic proxy for a metrics wrapper:
- define a service interface
- create an invocation handler
- record method name and duration
- call the real target object
- decide how to handle exceptions

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/dynamic-proxy/README.md)
- [App.java](../../github-repo/dynamic-proxy/src/main/java/com/iluwatar/dynamicproxy/App.java)
- [AlbumService.java](../../github-repo/dynamic-proxy/src/main/java/com/iluwatar/dynamicproxy/AlbumService.java)
- [AlbumInvocationHandler.java](../../github-repo/dynamic-proxy/src/main/java/com/iluwatar/dynamicproxy/AlbumInvocationHandler.java)
- [TinyRestClient.java](../../github-repo/dynamic-proxy/src/main/java/com/iluwatar/dynamicproxy/tinyrestclient/TinyRestClient.java)
- [Album.java](../../github-repo/dynamic-proxy/src/main/java/com/iluwatar/dynamicproxy/Album.java)

The repo implementation creates a dynamic proxy for `AlbumService`. Method calls are routed to `AlbumInvocationHandler`, which forwards method metadata and arguments to `TinyRestClient` so annotations can drive REST calls.
