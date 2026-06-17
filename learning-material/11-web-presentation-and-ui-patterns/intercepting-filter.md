# Intercepting Filter Pattern

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [intercepting-filter](../../github-repo/intercepting-filter)

---

## How to Study This Page

Study Intercepting Filter as the pattern behind web filters, interceptors, and middleware.

The core idea:
- request enters
- a chain of filters runs
- target handler runs
- filters may also run response/post-processing

---

## 1. Technical Definition

Intercepting Filter is a presentation/web pattern where pluggable filters process a request and/or response before or after the target handler, enabling reusable cross-cutting behavior.

### 30-Second Interview Answer

Intercepting Filter lets us apply common request-processing steps in a chain, such as authentication, logging, validation, compression, rate limiting, or tracing. Each filter handles one concern and passes control to the next filter or target. It is heavily used in servlet filters, Spring interceptors, and web middleware. The trade-offs are ordering sensitivity, performance overhead, and harder debugging through long chains.

---

## 2. Layman and Easy to Understand Definition

Imagine a request going through checkpoints before reaching the actual service:
- check authentication
- log the request
- validate input
- apply rate limit
- call handler

Each checkpoint is a filter.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Many request concerns are repeated across endpoints:
- authentication
- authorization
- logging
- tracing
- validation
- request normalization
- response compression

Putting those steps in every controller creates duplication.

### Filter Chain Flow

1. Client sends a request.
2. Filter manager or framework builds a chain.
3. Filter 1 processes the request.
4. Filter 1 delegates to Filter 2.
5. Filters continue until target handler is reached.
6. Target produces response.
7. Filters may post-process the response on the way back.

### Core Participants

| Participant | Responsibility |
|---|---|
| Filter | One pre/post-processing concern |
| Filter chain | Orders and links filters |
| Filter manager | Configures the chain |
| Target | Final request handler |
| Client/framework | Sends request into the chain |

---

## 4. Java Coding Example

```java
interface RequestFilter {
    void apply(Request request, FilterChain chain);
}

record Request(String path, boolean authenticated) {}

class FilterChain {
    private final java.util.List<RequestFilter> filters;
    private int index = 0;

    FilterChain(java.util.List<RequestFilter> filters) {
        this.filters = filters;
    }

    public void next(Request request) {
        if (index < filters.size()) {
            filters.get(index++).apply(request, this);
            return;
        }
        System.out.println("target handles " + request.path());
    }
}

public class InterceptingFilterDemo {
    public static void main(String[] args) {
        RequestFilter logging = (request, chain) -> {
            System.out.println("log " + request.path());
            chain.next(request);
        };

        RequestFilter auth = (request, chain) -> {
            if (!request.authenticated()) {
                System.out.println("401");
                return;
            }
            chain.next(request);
        };

        var chain = new FilterChain(java.util.List.of(logging, auth));
        chain.next(new Request("/orders", true));
    }
}
```

### Java Block by Block

Each filter receives the request and the chain.

The logging filter logs and delegates.

The auth filter can stop the request or delegate.

The target runs only if filters allow the chain to continue.

---

## 5. Python Coding Example

```python
class Request:
    def __init__(self, path, authenticated):
        self.path = path
        self.authenticated = authenticated


def target(request):
    print("target handles", request.path)


def logging_filter(request, next_filter):
    print("log", request.path)
    next_filter(request)


def auth_filter(request, next_filter):
    if not request.authenticated:
        print("401")
        return
    next_filter(request)


def build_chain(filters, final_handler):
    handler = final_handler
    for filter_func in reversed(filters):
        next_handler = handler
        handler = lambda request, f=filter_func, n=next_handler: f(request, n)
    return handler


chain = build_chain([logging_filter, auth_filter], target)
chain(Request("/orders", True))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Authentication | Applies before protected endpoints |
| Logging/tracing | Adds consistent request metadata |
| Input validation | Rejects malformed requests early |
| Compression | Post-processes response |
| Rate limiting | Can reject before business handler |
| Localization | Sets locale context before controller |

---

## 7. Advantages Over Normal Code Without Pattern

Without Intercepting Filter:
- common request logic is copied into handlers
- security checks can be missed
- logging/tracing is inconsistent
- ordering is implicit and hard to change

With Intercepting Filter:
- cross-cutting concerns are reusable
- filters can be ordered and configured
- handlers stay focused on business behavior
- request policies become centralized

---

## 8. Where It Excels

It excels when:
- many endpoints share cross-cutting concerns
- concerns can be handled before/after target
- filters are independently testable
- order is clear and controlled
- framework support exists

---

## 9. Where It Fails

It fails when:
- filters become full business workflows
- ordering is unclear
- filters hide side effects
- chain length creates latency
- errors are swallowed or transformed inconsistently
- state leaks between requests

Keep filters small, stateless when possible, and explicit about stop/continue behavior.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Servlet `Filter`, Spring `HandlerInterceptor`, Spring Security filters |
| Python | Django middleware, Flask before/after hooks |
| JavaScript | Express/Koa middleware |
| .NET | ASP.NET Core middleware |
| API gateways | Envoy filters, NGINX modules, gateway plugins |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reusable cross-cutting logic | Ordering matters |
| Keeps handlers clean | Long chains hurt latency |
| Easy to add/remove filters | Debugging can be harder |
| Centralizes policy | Hidden side effects are possible |
| Framework-friendly | Incorrect filter can block all requests |

---

## 12. Real-World Identification Example

Question:

> Every API endpoint needs request logging, JWT validation, request ID injection, and response compression. You do not want to duplicate that logic in every controller. What pattern fits?

Strong answer:

Use Intercepting Filter. Build a filter or middleware chain where logging, request ID, authentication, and compression are separate filters. The chain should run in a defined order, be observable, and stop early for failed authentication or invalid requests.

---

## 13. MAANG Interview Triggers

Say Intercepting Filter when you hear:
- middleware chain
- servlet filter
- request pre-processing
- response post-processing
- cross-cutting web concerns
- authentication before controller
- logging for every request

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Business logic in filters | Makes flow invisible | Keep business logic in handlers/services |
| No defined order | Behavior changes unpredictably | Configure explicit order |
| Filters mutate too much state | Hard to debug | Keep request changes minimal |
| Swallowing exceptions | Failures disappear | Use standard error mapping |
| Too many expensive filters | Adds latency | Measure and keep filters focused |

---

## 15. Intercepting Filter vs Similar Patterns

| Pattern | Difference |
|---|---|
| Chain of Responsibility | Filter chain is a web-specific form of chained processing |
| Decorator | Decorator wraps behavior; filters process request flow |
| Front Controller | Front Controller routes requests; filters run before/after handling |
| Proxy | Proxy controls access around an object; filters process web requests |
| Template Method | Template defines fixed algorithm; filters are pluggable steps |

---

## 16. Intercepting Filter Design Checklist

- What filters are needed?
- What is the order?
- Which filters can stop the chain?
- Which filters modify request/response?
- Are filters stateless?
- How are exceptions handled?
- What metrics are collected?
- Which concerns belong in controller/service instead?

---

## 17. Quick Revision Notes

- One-line summary: Pluggable request/response steps around the target handler.
- Memory hook: "checkpoints before the controller."
- Best for: auth, logging, tracing, validation, compression.
- Avoid when: the logic is endpoint-specific business behavior.
- Interview line: "I would model cross-cutting request behavior as ordered filters and keep handlers focused on domain work."

---

## 18. Mini Exercise

Design a filter chain for `/api/orders`:
- request ID filter
- logging filter
- JWT auth filter
- rate limit filter
- target handler

Define which filters can stop the chain and what response they return.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/intercepting-filter/README.md)
- [Filter.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/Filter.java)
- [AbstractFilter.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/AbstractFilter.java)
- [FilterChain.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/FilterChain.java)
- [FilterManager.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/FilterManager.java)
- [NameFilter.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/NameFilter.java)
- [Order.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/Order.java)
- [Client.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/Client.java)
- [Target.java](../../github-repo/intercepting-filter/src/main/java/com/iluwatar/intercepting/filter/Target.java)

The repo implementation uses `FilterChain` and `AbstractFilter` to link filters together, then `FilterManager.filterRequest()` runs an order through the chain.

