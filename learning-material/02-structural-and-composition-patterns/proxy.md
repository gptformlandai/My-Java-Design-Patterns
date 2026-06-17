# Proxy Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/proxy](../../github-repo/proxy)

## How to Study This Page

Use this page in three passes:

1. First pass: understand that Proxy usually exposes the same interface as the real object.
2. Second pass: rewrite the Java example and identify the subject, real subject, proxy, and client.
3. Third pass: study proxy types: protection, virtual, remote, caching, and logging proxies.

By the end, you should be able to say:

> Proxy controls access to a real object while presenting the same interface to the client.

## 1. Technical Definition

Proxy is a structural design pattern that provides a surrogate or placeholder object for another object. The proxy implements the same interface as the real subject and controls access, creation, communication, caching, authorization, or monitoring around the real subject.

Core idea:

- Client depends on a subject interface.
- Real subject performs the real work.
- Proxy implements the same interface.
- Proxy adds control before or after delegating to the real subject.

### 30-Second Interview Answer

I would use Proxy when I want clients to use the same interface but need control around the real object, such as authorization, lazy loading, remote communication, caching, logging, or rate limiting. The proxy can deny, delay, forward, cache, or monitor calls. The main trade-off is extra indirection and the risk of hiding expensive or remote behavior behind a normal-looking method call.

## 2. Layman and Easy to Understand Definition

Proxy is like a security desk in front of an office.

You do not walk directly into the CEO's office. You talk to the security desk first. The desk checks whether you are allowed, then either lets you through or denies access.

In code:

- The CEO's office is the real object.
- The security desk is the proxy.
- The visitor is the client.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a document store exposes sensitive documents.

Direct access:

```java
DocumentService service = new RealDocumentService();
Document document = service.open("salary-report");
```

Problems:

- Anyone with the service can open documents.
- Authorization may be repeated in many places.
- Logging and audit may be scattered.
- Expensive documents may load even when access should be denied.

### 3.2 The Proxy Solution

Put a proxy in front:

```java
DocumentService service =
    new SecureDocumentServiceProxy(new RealDocumentService(), currentUser);
```

The client still calls:

```java
service.open("salary-report");
```

But the proxy decides whether to forward the call to the real service.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Subject | Interface shared by proxy and real subject. |
| Real subject | Object that performs the actual work. |
| Proxy | Object that controls access to the real subject. |
| Client | Code that calls the subject interface. |
| Control logic | Authorization, lazy loading, caching, logging, etc. |

### 3.4 Common Proxy Types

| Type | Purpose |
|---|---|
| Protection proxy | Checks permissions before access. |
| Virtual proxy | Lazily creates expensive objects. |
| Remote proxy | Represents an object in another process or machine. |
| Caching proxy | Stores results to avoid repeated expensive calls. |
| Logging / monitoring proxy | Records calls, timing, or metrics. |
| Smart reference proxy | Adds extra behavior around object access. |

### 3.5 Mental Model

Think of Proxy as same door, controlled entry.

1. Client calls the subject interface.
2. Proxy receives the call.
3. Proxy checks rules or adds behavior.
4. Proxy may return early, throw, cache, or log.
5. Proxy delegates to the real subject when appropriate.
6. Client receives the result through the same interface.

## 4. Java Coding Example

This example creates a protection proxy for a document service.

```java
import java.util.Map;
import java.util.Objects;
import java.util.Set;

public record Document(String id, String content) {}

public record User(String id, Set<String> roles) {
    public boolean hasRole(String role) {
        return roles.contains(role);
    }
}

public interface DocumentService {
    Document open(String documentId);
}

public final class RealDocumentService implements DocumentService {
    private final Map<String, Document> documents = Map.of(
        "public-guide", new Document("public-guide", "Welcome guide"),
        "salary-report", new Document("salary-report", "Confidential salaries")
    );

    @Override
    public Document open(String documentId) {
        System.out.println("Loading document " + documentId);
        Document document = documents.get(documentId);
        if (document == null) {
            throw new IllegalArgumentException("Unknown document: " + documentId);
        }
        return document;
    }
}

public final class SecureDocumentServiceProxy implements DocumentService {
    private final DocumentService realService;
    private final User currentUser;

    public SecureDocumentServiceProxy(DocumentService realService, User currentUser) {
        this.realService = Objects.requireNonNull(realService);
        this.currentUser = Objects.requireNonNull(currentUser);
    }

    @Override
    public Document open(String documentId) {
        if (documentId.startsWith("salary") && !currentUser.hasRole("HR")) {
            throw new SecurityException("User cannot access " + documentId);
        }
        System.out.println("Audit: " + currentUser.id() + " opened " + documentId);
        return realService.open(documentId);
    }
}
```

### Java Block by Block Explanation

#### Subject Interface

```java
public interface DocumentService {
    Document open(String documentId);
}
```

Both the real service and proxy implement this interface.

#### Real Subject

```java
public final class RealDocumentService implements DocumentService {
```

The real subject performs the actual work: loading documents.

#### Proxy

```java
public final class SecureDocumentServiceProxy implements DocumentService {
```

The proxy has the same interface as the real subject, so clients can use it transparently.

#### Access Control

```java
if (documentId.startsWith("salary") && !currentUser.hasRole("HR")) {
    throw new SecurityException("User cannot access " + documentId);
}
```

The proxy controls whether the real subject is called.

#### Delegation

```java
return realService.open(documentId);
```

If the access check passes, the proxy forwards the call.

### Java Usage

```java
import java.util.Set;

public class Demo {
    public static void main(String[] args) {
        User user = new User("user-1", Set.of("EMPLOYEE"));

        DocumentService service = new SecureDocumentServiceProxy(
            new RealDocumentService(),
            user
        );

        Document guide = service.open("public-guide");
        System.out.println(guide);
    }
}
```

The client depends only on `DocumentService`. It does not need to know whether the implementation is real or proxied.

### Virtual Proxy Variation

```java
public final class LazyDocumentServiceProxy implements DocumentService {
    private DocumentService realService;

    @Override
    public Document open(String documentId) {
        if (realService == null) {
            realService = new RealDocumentService();
        }
        return realService.open(documentId);
    }
}
```

This delays creation of the expensive real service until the first call.

## 5. Python Coding Example

Python proxies are usually wrapper classes with the same methods.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Document:
    document_id: str
    content: str


@dataclass(frozen=True)
class User:
    user_id: str
    roles: set[str]

    def has_role(self, role: str) -> bool:
        return role in self.roles


class RealDocumentService:
    def __init__(self) -> None:
        self._documents = {
            "public-guide": Document("public-guide", "Welcome guide"),
            "salary-report": Document("salary-report", "Confidential salaries"),
        }

    def open(self, document_id: str) -> Document:
        print(f"Loading document {document_id}")
        return self._documents[document_id]


class SecureDocumentServiceProxy:
    def __init__(self, real_service: RealDocumentService, current_user: User) -> None:
        self._real_service = real_service
        self._current_user = current_user

    def open(self, document_id: str) -> Document:
        if document_id.startswith("salary") and not self._current_user.has_role("HR"):
            raise PermissionError(f"User cannot access {document_id}")
        print(f"Audit: {self._current_user.user_id} opened {document_id}")
        return self._real_service.open(document_id)
```

### Python Usage

```python
user = User("user-1", {"EMPLOYEE"})
service = SecureDocumentServiceProxy(RealDocumentService(), user)
print(service.open("public-guide"))
```

## 6. Where It Comes Handy in Real Life

Proxy is common anywhere access or lifecycle needs control.

Examples:

- Authorization proxies around services.
- Lazy-loading proxies in ORMs.
- Remote service proxies in RPC systems.
- HTTP client proxies.
- Caching proxies around expensive calls.
- Logging and metrics wrappers.
- Rate-limiting proxies.
- Mocking frameworks and dynamic proxies.

## 7. Advantages Over Normal Code Without Pattern

### Without Proxy

```java
Document document = realService.open("salary-report");
```

Problems:

- Access control may be scattered.
- Audit logging may be forgotten.
- Expensive objects may be created too early.
- Remote/cached behavior may leak into client code.

### With Proxy

```java
DocumentService service = new SecureDocumentServiceProxy(realService, user);
Document document = service.open("salary-report");
```

Benefits:

- Client uses the same interface.
- Access control is centralized.
- Lazy loading or caching can be added transparently.
- Real subject remains focused on core behavior.

## 8. Where It Excels

Proxy excels when:

- You need access control.
- You want lazy initialization.
- You need caching around expensive operations.
- The real object is remote.
- You want logging, metrics, or rate limiting around calls.
- You want clients to keep using the same interface.

## 9. Where It Fails

Proxy is a poor fit when:

- There is no access or lifecycle behavior to add.
- The proxy hides expensive remote calls too well.
- The extra layer makes debugging harder.
- The proxy violates the subject contract.
- A decorator better describes optional behavior layering.

Example where Proxy is overkill:

```java
DocumentService service = new RealDocumentService();
```

If no access control, caching, laziness, or remote handling is needed, the proxy adds little value.

## 10. Prebuilt Libraries and Packages

### Java

Common proxy mechanisms:

- `java.lang.reflect.Proxy`
- Spring AOP proxies
- Hibernate lazy-loading proxies
- Mockito mocks
- JDK dynamic proxies
- Byte Buddy and CGLIB
- RMI stubs and remote proxies

Common proxy use cases:

- Transactions.
- Security.
- Caching.
- Metrics.
- Lazy loading.
- Remote calls.

### Python

Python proxy techniques:

- Wrapper classes.
- `__getattr__` forwarding.
- Decorators.
- Context managers.
- Mock objects.
- HTTP or RPC client stubs.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Controls access to real objects. | Adds indirection. |
| Can add lazy loading. | Can hide expensive behavior. |
| Centralizes security, caching, or logging. | Debugging call flow can be harder. |
| Keeps real subject focused. | Must preserve the subject contract. |
| Clients use the same interface. | Thread safety can become tricky. |

## 12. Real-World Identification Example

Scenario:

You are designing a file storage service.

Requirements:

- Only authorized users can read private files.
- Every read must be audited.
- Large file metadata should be loaded lazily.
- Client code should still call `FileStore.read(fileId)`.

Should you use Proxy?

Yes.

Good usage:

```java
FileStore fileStore = new AuditedSecureFileStoreProxy(realFileStore, currentUser);
FileContent content = fileStore.read(fileId);
```

The proxy controls access and audit while preserving the `FileStore` interface.

## 13. MAANG Interview Triggers

Think Proxy when you hear:

- Controlled access.
- Same interface as real object.
- Lazy loading.
- Remote object.
- Caching wrapper.
- Security check before operation.
- Audit/log/metrics around calls.
- Placeholder for expensive object.
- Dynamic proxy or AOP.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the subject interface.
2. Identify the real subject.
3. Add a proxy implementing the same interface.
4. Put access/lazy/cache/logging behavior in the proxy.
5. Delegate to the real subject when allowed.
6. Mention trade-off: extra indirection and hidden cost.

## 14. Common Mistakes

### Mistake 1: Changing the Interface

If the wrapper exposes a different interface, it is probably Adapter, not Proxy.

### Mistake 2: Putting Core Business Logic in Proxy

Proxy should control access or add cross-cutting behavior. Core domain rules usually belong in services or domain objects.

### Mistake 3: Hiding Remote Calls Too Much

A remote proxy may look like a local object, but latency and failure behavior are different. Design with timeouts and retries.

### Mistake 4: Breaking the Subject Contract

The proxy should behave like the real subject from the client's perspective, except for documented control behavior.

### Mistake 5: Ignoring Thread Safety

Caching, lazy initialization, and counters inside a proxy may need synchronization or concurrent data structures.

## 15. Proxy vs Similar Patterns

| Pattern | Difference |
|---|---|
| Adapter | Adapter changes the interface. Proxy keeps the same interface and controls access. |
| Decorator | Decorator adds responsibilities dynamically. Proxy controls access or lifecycle. |
| Facade | Facade simplifies a subsystem. Proxy represents one object and controls access to it. |
| Bridge | Bridge separates abstraction and implementation. Proxy stands in front of a real subject. |
| Gateway | Gateway hides remote system details. It may use proxies internally. |

## 16. Classic Structure

| Role | Example in this page | Responsibility |
|---|---|---|
| Subject | `DocumentService` | Shared interface. |
| Real subject | `RealDocumentService` | Performs real work. |
| Proxy | `SecureDocumentServiceProxy` | Controls access and delegates. |
| Client | Demo or service code | Uses subject interface. |

The key interview sentence:

> Proxy is about controlled access through the same interface.

## 17. Quick Revision Notes

- Proxy stands in front of a real object.
- Proxy usually implements the same interface.
- It can add security, caching, logging, remote communication, or lazy loading.
- It delegates to the real subject when appropriate.
- Adapter changes interface; Proxy controls access.
- Be careful with hidden latency, failure, and thread safety.

## 18. Mini Exercise

Design a Proxy for `ImageLoader`.

Subject:

```java
interface ImageLoader {
    Image load(String imageId);
}
```

Requirements:

- Load real images lazily.
- Cache loaded images.
- Log load time.
- Return cached images without calling the real loader again.

Expected usage:

```java
ImageLoader loader = new CachingImageLoaderProxy(new RealImageLoader());
Image image = loader.load("hero-banner");
```

## 19. Source Reference in This Repo

The repository's Proxy implementation uses `WizardTowerProxy` to control access to an `IvoryTower` through the same `WizardTower` interface.

Useful files:

- [github-repo/proxy/README.md](../../github-repo/proxy/README.md)
- [github-repo/proxy/src/main/java/com/iluwatar/proxy/WizardTower.java](../../github-repo/proxy/src/main/java/com/iluwatar/proxy/WizardTower.java)
- [github-repo/proxy/src/main/java/com/iluwatar/proxy/WizardTowerProxy.java](../../github-repo/proxy/src/main/java/com/iluwatar/proxy/WizardTowerProxy.java)
- [github-repo/proxy/src/main/java/com/iluwatar/proxy/IvoryTower.java](../../github-repo/proxy/src/main/java/com/iluwatar/proxy/IvoryTower.java)
- [github-repo/proxy/src/main/java/com/iluwatar/proxy/App.java](../../github-repo/proxy/src/main/java/com/iluwatar/proxy/App.java)
