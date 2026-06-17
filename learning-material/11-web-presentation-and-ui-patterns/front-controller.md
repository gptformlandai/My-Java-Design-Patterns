# Front Controller Pattern

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [front-controller](../../github-repo/front-controller)

---

## How to Study This Page

Study Front Controller as the single front door for web requests.

The pattern answers:
- Where do all requests enter?
- Where do common concerns run?
- How does a request get routed to the right handler?

---

## 1. Technical Definition

Front Controller is a presentation-layer pattern where all incoming requests pass through a single controller that performs common processing and delegates each request to the appropriate handler, command, or view.

### 30-Second Interview Answer

Front Controller centralizes request handling. Every HTTP request enters through one front controller, which can apply common behavior such as authentication, logging, locale setup, error handling, and routing. It then dispatches to the right controller/action/view. Spring MVC's `DispatcherServlet` is the classic Java example. The trade-off is that the central dispatcher must stay thin and configurable, or it becomes a bottleneck.

---

## 2. Layman and Easy to Understand Definition

Imagine a reception desk for an office. Everyone enters through reception. Reception checks who they are, understands what they need, and sends them to the correct department.

The reception desk is the front controller.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

If every page or handler processes requests independently, common behavior gets duplicated:
- authentication
- logging
- routing
- validation setup
- exception handling
- view selection

### Front Controller Flow

1. Client sends a request.
2. Front controller receives it.
3. Front controller applies common behavior.
4. Dispatcher maps request to command/handler.
5. Handler performs request-specific work.
6. Handler returns a view or response.
7. Front controller finalizes response or error handling.

### Core Participants

| Participant | Responsibility |
|---|---|
| Front controller | Central request entry point |
| Dispatcher/router | Chooses the handler |
| Command/handler | Request-specific work |
| View/response | User-facing output |
| Common services | Auth, logging, exception handling |

---

## 4. Java Coding Example

```java
import java.util.HashMap;
import java.util.Map;

interface Handler {
    String handle();
}

class FrontController {
    private final Map<String, Handler> routes = new HashMap<>();

    public void register(String path, Handler handler) {
        routes.put(path, handler);
    }

    public String handleRequest(String path) {
        System.out.println("log request: " + path);
        Handler handler = routes.getOrDefault(path, () -> "404");
        return handler.handle();
    }
}

public class FrontControllerDemo {
    public static void main(String[] args) {
        FrontController controller = new FrontController();
        controller.register("/profile", () -> "profile page");
        controller.register("/orders", () -> "orders page");

        System.out.println(controller.handleRequest("/profile"));
        System.out.println(controller.handleRequest("/missing"));
    }
}
```

### Java Block by Block

All requests enter `handleRequest`.

Common behavior such as logging runs once in the front controller.

The route table chooses the correct handler.

Unknown routes use a consistent fallback.

---

## 5. Python Coding Example

```python
class FrontController:
    def __init__(self):
        self.routes = {}

    def register(self, path, handler):
        self.routes[path] = handler

    def handle_request(self, path):
        print("log request:", path)
        handler = self.routes.get(path, lambda: "404")
        return handler()


controller = FrontController()
controller.register("/profile", lambda: "profile page")
controller.register("/orders", lambda: "orders page")

print(controller.handle_request("/profile"))
print(controller.handle_request("/missing"))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Web frameworks | One entry point can route all requests |
| Authentication | Common security checks run centrally |
| Logging and tracing | Every request gets consistent metadata |
| Error handling | Exceptions map to consistent responses |
| Template rendering | Controller can select views consistently |
| API gateways inside apps | Central route dispatch |

---

## 7. Advantages Over Normal Code Without Pattern

Without Front Controller:
- common behavior is duplicated
- route handling is scattered
- error behavior is inconsistent
- security checks can be missed

With Front Controller:
- requests enter through one known point
- cross-cutting behavior is centralized
- routing is explicit
- default/error handling is consistent

---

## 8. Where It Excels

It excels when:
- many routes need common behavior
- framework conventions support central dispatch
- requests need consistent logging/auth/error handling
- handler lookup should be configurable

---

## 9. Where It Fails

It fails when:
- the front controller contains all business logic
- routing rules become too magical
- one central class becomes too large
- common behavior should actually be a filter/middleware
- the dispatch path adds unnecessary latency

Keep the front controller as routing and orchestration, not business logic.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Spring MVC `DispatcherServlet`, Struts, JSF |
| Python | Django URL dispatcher, Flask app router |
| JavaScript | Express router, Next.js routing layer |
| Ruby | Rails router/controller stack |
| PHP | Symfony front controller, Laravel public entry |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Centralized request handling | Can become a god object |
| Consistent auth/logging/errors | Adds routing layer complexity |
| Easy global policy changes | Misconfiguration affects many routes |
| Framework-friendly | Can become bottleneck if poorly designed |
| Clean fallback handling | Too much reflection can be fragile |

---

## 12. Real-World Identification Example

Question:

> You are designing a web framework. Every request must pass through authentication, logging, route lookup, and consistent error handling before calling a page-specific handler. What pattern fits?

Strong answer:

Use Front Controller. Put a single entry point in front of all requests. It runs common request processing, delegates routing to a dispatcher, invokes the selected handler, and maps errors to standard responses. I would keep business logic out of the front controller and place cross-cutting request steps in filters or middleware where appropriate.

---

## 13. MAANG Interview Triggers

Say Front Controller when you hear:
- single entry point for requests
- central request handling
- dispatcher servlet
- route all web requests
- common auth/logging/error handling
- command selected by request
- consistent fallback page

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Putting business logic in front controller | Central class becomes unmaintainable | Delegate to handlers/services |
| Hardcoding too many routes | Routing becomes brittle | Use configuration or route registry |
| Skipping error fallback | Unknown routes fail inconsistently | Add standard 404/error handling |
| Mixing filters and dispatch logic | Concerns blur | Use filters/middleware for cross-cutting steps |
| Reflection without safeguards | Runtime errors are harder to trace | Prefer explicit mappings when possible |

---

## 15. Front Controller vs Similar Patterns

| Pattern | Difference |
|---|---|
| MVC | Front Controller is often the central controller entry used by MVC web frameworks |
| Intercepting Filter | Filters run before/after request handling; front controller routes requests |
| Command | Commands can represent request-specific handlers selected by the front controller |
| Facade | Facade simplifies API access; front controller handles web request flow |
| API Gateway | Gateway is usually cross-service boundary; front controller is inside an app |

---

## 16. Front Controller Design Checklist

- What is the single entry point?
- What common behavior runs before routing?
- How are routes mapped to handlers?
- What happens on unknown routes?
- How are exceptions converted to responses?
- Which concerns belong in filters instead?
- Is business logic delegated away?
- How is request tracing added?

---

## 17. Quick Revision Notes

- One-line summary: One central web entry point routes every request.
- Memory hook: "reception desk for requests."
- Best for: web apps and frameworks with many routes.
- Avoid when: it becomes a business-logic dumping ground.
- Interview line: "I would centralize request entry, run common handling, dispatch to specific handlers, and keep the front controller thin."

---

## 18. Mini Exercise

Design a front controller for three routes:
- `/login`
- `/profile`
- `/orders`

Add one common logging step, one authentication decision, one unknown-route fallback, and one handler per route.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/front-controller/README.md)
- [FrontController.java](../../github-repo/front-controller/src/main/java/com/iluwatar/front/controller/FrontController.java)
- [Dispatcher.java](../../github-repo/front-controller/src/main/java/com/iluwatar/front/controller/Dispatcher.java)
- [Command.java](../../github-repo/front-controller/src/main/java/com/iluwatar/front/controller/Command.java)
- [UnknownCommand.java](../../github-repo/front-controller/src/main/java/com/iluwatar/front/controller/UnknownCommand.java)
- [View.java](../../github-repo/front-controller/src/main/java/com/iluwatar/front/controller/View.java)
- [App.java](../../github-repo/front-controller/src/main/java/com/iluwatar/front/controller/App.java)

The repo implementation uses `FrontController.handleRequest()` as the single entry point, then `Dispatcher` maps the request to a command and falls back to `UnknownCommand` when needed.

