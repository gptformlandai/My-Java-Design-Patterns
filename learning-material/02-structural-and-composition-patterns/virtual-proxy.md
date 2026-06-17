# Virtual Proxy Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/virtual-proxy](../../github-repo/virtual-proxy)

## How to Study This Page

Use this page in three passes:

1. First pass: understand Virtual Proxy as the lazy-loading form of Proxy.
2. Second pass: rewrite the Java example and explain when the real object is created.
3. Third pass: study thread safety, hidden latency, and failure behavior.

By the end, you should be able to say:

> Virtual Proxy delays creation of an expensive real object until the client actually needs it.

## 1. Technical Definition

Virtual Proxy is a Proxy variant that controls access to an expensive object by creating it lazily, usually on the first real operation.

Core idea:

- Proxy implements the same interface as the real subject.
- Proxy initially holds no real subject.
- On first use, proxy creates the real subject.
- Later calls reuse the created subject.
- Client uses the same interface throughout.

### 30-Second Interview Answer

I would use Virtual Proxy when an object is expensive to create and may never be used. The proxy implements the same interface and delays real object creation until the first operation that needs it. This can improve startup time and memory use, but the first call may become slower and lazy initialization must be thread-safe.

## 2. Layman and Easy to Understand Definition

Virtual Proxy is like a placeholder thumbnail for a large image.

The page can load quickly with placeholders. The full image loads only when someone scrolls to it or clicks it.

In code:

- The thumbnail/placeholder is the proxy.
- The full image is the real object.
- Loading happens only when needed.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose each image object loads a large file immediately.

```java
Image image = new RealImage("hero-banner.png");
```

Problems:

- Startup becomes slow.
- Memory is used before the image is needed.
- Objects may be created but never used.
- Heavy initialization is spread across client code.

### 3.2 The Virtual Proxy Solution

Use a proxy:

```java
Image image = new LazyImageProxy("hero-banner.png");
```

The real image loads only when:

```java
image.display();
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Subject | Interface shared by proxy and real object. |
| Real subject | Expensive object created lazily. |
| Virtual proxy | Placeholder that creates real subject on demand. |
| Client | Uses the subject interface. |
| Lazy initialization | Delayed creation of the real subject. |

### 3.4 Mental Model

Think of Virtual Proxy as "pay when used".

1. Client receives a lightweight proxy.
2. Proxy stores enough data to create the real object.
3. Client calls a real operation.
4. Proxy creates the real object if missing.
5. Proxy delegates the operation.
6. Future calls reuse the real object.

## 4. Java Coding Example

This example lazily loads an image.

```java
public interface Image {
    void display();
}

public final class RealImage implements Image {
    private final String fileName;

    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + fileName);
    }

    @Override
    public void display() {
        System.out.println("Displaying image: " + fileName);
    }
}

public final class LazyImageProxy implements Image {
    private final String fileName;
    private RealImage realImage;

    public LazyImageProxy(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}
```

### Java Block by Block Explanation

#### Subject

```java
public interface Image {
    void display();
}
```

Both proxy and real object implement this interface.

#### Real Subject

```java
public final class RealImage implements Image {
```

The real object performs expensive loading in its constructor.

#### Virtual Proxy

```java
public final class LazyImageProxy implements Image {
```

The proxy is cheap to create and can stand in for the real image.

#### Lazy Initialization

```java
if (realImage == null) {
    realImage = new RealImage(fileName);
}
```

The real object is created only on first use.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        Image image = new LazyImageProxy("hero-banner.png");

        System.out.println("Proxy created");
        image.display();
        image.display();
    }
}
```

The image loads on the first `display()`, not when the proxy is created.

### Thread-Safe Variation

```java
public final class ThreadSafeLazyImageProxy implements Image {
    private final String fileName;
    private volatile RealImage realImage;

    public ThreadSafeLazyImageProxy(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void display() {
        RealImage result = realImage;
        if (result == null) {
            synchronized (this) {
                result = realImage;
                if (result == null) {
                    result = new RealImage(fileName);
                    realImage = result;
                }
            }
        }
        result.display();
    }
}
```

Use this only when the proxy may be accessed by multiple threads.

## 5. Python Coding Example

```python
class Image:
    def display(self) -> None:
        raise NotImplementedError


class RealImage(Image):
    def __init__(self, file_name: str) -> None:
        self._file_name = file_name
        self._load_from_disk()

    def _load_from_disk(self) -> None:
        print(f"Loading image from disk: {self._file_name}")

    def display(self) -> None:
        print(f"Displaying image: {self._file_name}")


class LazyImageProxy(Image):
    def __init__(self, file_name: str) -> None:
        self._file_name = file_name
        self._real_image: RealImage | None = None

    def display(self) -> None:
        if self._real_image is None:
            self._real_image = RealImage(self._file_name)
        self._real_image.display()
```

### Python Usage

```python
image: Image = LazyImageProxy("hero-banner.png")
print("Proxy created")
image.display()
image.display()
```

## 6. Where It Comes Handy in Real Life

Virtual Proxy is useful when expensive objects are not always needed.

Examples:

- Lazy-loaded images.
- ORM lazy entities.
- Large document previews.
- Video objects.
- Heavy machine learning models.
- Remote metadata loaded on first access.
- Expensive report renderers.

## 7. Advantages Over Normal Code Without Pattern

### Without Virtual Proxy

```java
Image image = new RealImage("hero-banner.png");
```

Problems:

- Heavy loading happens immediately.
- Startup time increases.
- Memory may be wasted.
- Unused objects still pay creation cost.

### With Virtual Proxy

```java
Image image = new LazyImageProxy("hero-banner.png");
```

Benefits:

- Cheap initial creation.
- Real object loads only if needed.
- Same interface for client code.
- Expensive work can be deferred.

## 8. Where It Excels

Virtual Proxy excels when:

- Real object creation is expensive.
- Many objects may never be used.
- Startup time matters.
- Memory should be used lazily.
- The proxy can preserve the same interface.

## 9. Where It Fails

Virtual Proxy is a poor fit when:

- The real object is cheap to create.
- Hidden first-call latency is unacceptable.
- Lazy initialization can fail in surprising places.
- Thread safety is hard to guarantee.
- Clients need to know loading progress explicitly.

## 10. Prebuilt Libraries and Packages

### Java

Virtual proxy examples:

- Hibernate lazy-loaded entities.
- `java.awt.Image` lazy loading.
- Spring lazy beans.
- JPA proxy references.
- Image loading frameworks.

### Python

Python examples:

- Lazy properties.
- ORM lazy relationships.
- Module-level lazy loaders.
- Proxy classes with deferred initialization.
- Cached properties.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Defers expensive object creation. | First real call may be slow. |
| Improves startup time. | Lazy failures happen later. |
| Saves memory for unused objects. | Thread safety can be tricky. |
| Preserves subject interface. | Can hide loading behavior. |
| Useful for large resources. | More indirection. |

## 12. Real-World Identification Example

Scenario:

You are designing a document viewer that shows a list of thousands of documents.

Requirements:

- Show document names quickly.
- Load full document content only when opened.
- Keep client code using `Document.display()`.

Should you use Virtual Proxy?

Yes.

Good usage:

```java
Document document = new LazyDocumentProxy(documentId);
document.display();
```

## 13. MAANG Interview Triggers

Think Virtual Proxy when you hear:

- Lazy loading.
- Expensive object creation.
- Placeholder object.
- Load on first use.
- Same interface as real object.
- Improve startup time.
- ORM proxy.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify the expensive real object.
2. Define the subject interface.
3. Create a proxy implementing the same interface.
4. Store creation parameters in the proxy.
5. Instantiate the real object on first operation.
6. Mention trade-off: first-call latency, thread safety, and delayed failures.

## 14. Common Mistakes

### Mistake 1: Not Preserving the Interface

If the proxy changes the interface, it is more like Adapter.

### Mistake 2: Ignoring Thread Safety

Two threads may create two real objects if initialization is not synchronized.

### Mistake 3: Hiding Expensive Latency

If first-call latency matters, expose loading status or prefetch intentionally.

### Mistake 4: No Failure Plan

Lazy loading may fail later than startup. Handle errors close to the operation.

### Mistake 5: Keeping Too Much State in Proxy

The proxy should hold just enough to create or access the real object.

## 15. Virtual Proxy vs Similar Patterns

| Pattern | Difference |
|---|---|
| Proxy | Virtual Proxy is a specific Proxy focused on lazy creation. |
| Lazy Loading | Lazy Loading is the technique; Virtual Proxy is one object-oriented pattern for it. |
| Decorator | Decorator adds behavior. Virtual Proxy delays real object creation. |
| Cache | Cache stores results. Virtual Proxy controls object initialization. |
| Factory | Factory creates objects on request. Virtual Proxy stands in for a specific object. |

## 16. Lazy Initialization Checklist

| Concern | Why it matters |
|---|---|
| Creation cost | The real object must be worth deferring. |
| First-call latency | Lazy loading moves cost to first use. |
| Thread safety | Multiple callers may race. |
| Failure behavior | Errors happen later. |
| Lifecycle cleanup | Lazily created resources may need closing. |

## 17. Quick Revision Notes

- Virtual Proxy is lazy-loading Proxy.
- It implements the same interface as the real object.
- It creates the real object on first use.
- It improves startup and memory use when many objects are unused.
- Watch first-call latency and thread safety.
- Use normal Proxy docs for broader proxy variants.

## 18. Mini Exercise

Design a Virtual Proxy for `ReportRenderer`.

Requirements:

- Renderer loads fonts and templates slowly.
- Report list should show quickly.
- Renderer should load only when `render()` is called.
- Multiple calls should reuse the same renderer.

Expected usage:

```java
ReportRenderer renderer = new LazyReportRendererProxy(templateId);
renderer.render(report);
```

## 19. Source Reference in This Repo

The repository's Virtual Proxy implementation uses `VideoObjectProxy` to lazily create `RealVideoObject`.

Useful files:

- [github-repo/virtual-proxy/README.md](../../github-repo/virtual-proxy/README.md)
- [github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/ExpensiveObject.java](../../github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/ExpensiveObject.java)
- [github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/VideoObjectProxy.java](../../github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/VideoObjectProxy.java)
- [github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/RealVideoObject.java](../../github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/RealVideoObject.java)
- [github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/App.java](../../github-repo/virtual-proxy/src/main/java/com/iluwatar/virtual/proxy/App.java)
