# Flyweight Pattern

Category: Structural and Composition Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/flyweight](../../github-repo/flyweight)

## How to Study This Page

Use this page in three passes:

1. First pass: understand intrinsic state vs extrinsic state.
2. Second pass: rewrite the Java example and explain why shared flyweights must be immutable or safely reusable.
3. Third pass: study when memory savings justify the added complexity.

By the end, you should be able to say:

> Flyweight reduces memory usage by sharing common intrinsic state across many logical objects while keeping context-specific extrinsic state outside.

## 1. Technical Definition

Flyweight is a structural design pattern that minimizes memory usage by sharing common state among many similar objects instead of storing repeated state in each object.

Core idea:

- Extract shared intrinsic state.
- Store context-specific extrinsic state outside the flyweight.
- Reuse flyweight instances through a factory or cache.
- Avoid depending on object identity.

### 30-Second Interview Answer

I would use Flyweight when an application creates a huge number of similar objects and repeated state dominates memory usage. The shared immutable state becomes the flyweight, while unique context like position, color, or owner is passed in from outside. It can save memory, but it adds complexity and only helps when there are many repeated objects.

## 2. Layman and Easy to Understand Definition

Flyweight is like printing a book with reusable letter shapes.

The shape of the letter `A` is shared. Each occurrence has its own page position, font size, or color.

In code:

- The letter shape is intrinsic shared state.
- The position is extrinsic external state.
- Many logical letters reuse one shared object.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a text editor creates one object per character:

```java
new CharacterGlyph("A", fontFamily, vectorShape, x, y, color);
```

If a document has millions of characters, repeated font and shape data wastes memory.

Problems:

- Many objects duplicate the same internal data.
- Memory usage grows quickly.
- Garbage collection pressure increases.
- Object identity may become misleading.

### 3.2 The Flyweight Solution

Split state:

```java
Glyph glyph = glyphFactory.get("A", "Inter");
glyph.draw(x, y, color);
```

The glyph stores shared state like character shape and font. The caller supplies position and color.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Flyweight | Shared object containing intrinsic state. |
| Intrinsic state | Shared state stored inside the flyweight. |
| Extrinsic state | Context-specific state passed from outside. |
| Flyweight factory | Returns shared instances from a cache. |
| Client | Stores or supplies extrinsic state. |

### 3.4 Intrinsic vs Extrinsic State

| State type | Example | Stored where |
|---|---|---|
| Intrinsic | Character shape, font family | Inside flyweight |
| Extrinsic | x/y position, color, selection state | Outside flyweight |

The interview trap:

> If every object still stores all state internally, it is not really Flyweight.

### 3.5 Mental Model

Think of Flyweight as shared identity-free data.

1. Find repeated state.
2. Move repeated state into a shared object.
3. Make shared state immutable or safe.
4. Keep unique state outside.
5. Use a factory/cache to reuse flyweights.

## 4. Java Coding Example

This example shares glyph objects in a text editor.

```java
import java.util.HashMap;
import java.util.Map;

public record GlyphKey(char character, String fontFamily) {}

public final class Glyph {
    private final char character;
    private final String fontFamily;

    public Glyph(char character, String fontFamily) {
        this.character = character;
        this.fontFamily = fontFamily;
    }

    public void draw(int x, int y, String color) {
        System.out.println(
            "Draw " + character + " in " + fontFamily +
            " at (" + x + ", " + y + ") color=" + color
        );
    }
}

public final class GlyphFactory {
    private final Map<GlyphKey, Glyph> cache = new HashMap<>();

    public Glyph getGlyph(char character, String fontFamily) {
        GlyphKey key = new GlyphKey(character, fontFamily);
        return cache.computeIfAbsent(key, ignored -> new Glyph(character, fontFamily));
    }

    public int cachedGlyphCount() {
        return cache.size();
    }
}
```

### Java Block by Block Explanation

#### Flyweight Key

```java
public record GlyphKey(char character, String fontFamily) {}
```

The key identifies shared intrinsic state.

#### Flyweight

```java
public final class Glyph {
```

The glyph stores only shared state.

#### Extrinsic State

```java
public void draw(int x, int y, String color) {
```

Position and color are passed in because they vary per occurrence.

#### Flyweight Factory

```java
return cache.computeIfAbsent(key, ignored -> new Glyph(character, fontFamily));
```

The factory reuses an existing flyweight when possible.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        GlyphFactory factory = new GlyphFactory();

        Glyph a1 = factory.getGlyph('A', "Inter");
        Glyph a2 = factory.getGlyph('A', "Inter");

        a1.draw(10, 20, "black");
        a2.draw(30, 20, "blue");

        System.out.println(a1 == a2);
        System.out.println(factory.cachedGlyphCount());
    }
}
```

`a1` and `a2` are the same shared flyweight, but they are drawn with different extrinsic state.

## 5. Python Coding Example

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Glyph:
    character: str
    font_family: str

    def draw(self, x: int, y: int, color: str) -> None:
        print(f"Draw {self.character} in {self.font_family} at ({x}, {y}) color={color}")


class GlyphFactory:
    def __init__(self) -> None:
        self._cache: dict[tuple[str, str], Glyph] = {}

    def get_glyph(self, character: str, font_family: str) -> Glyph:
        key = (character, font_family)
        if key not in self._cache:
            self._cache[key] = Glyph(character, font_family)
        return self._cache[key]
```

### Python Usage

```python
factory = GlyphFactory()
a1 = factory.get_glyph("A", "Inter")
a2 = factory.get_glyph("A", "Inter")

a1.draw(10, 20, "black")
a2.draw(30, 20, "blue")
print(a1 is a2)
```

## 6. Where It Comes Handy in Real Life

Flyweight is useful when many objects share repeated state.

Examples:

- Text editor glyphs.
- Game tiles.
- Map markers with shared icons.
- CAD objects.
- Font rendering.
- Interned strings.
- Cached wrapper values.
- Repeated UI style objects.

## 7. Advantages Over Normal Code Without Pattern

### Without Flyweight

```java
new Glyph('A', "Inter", x, y, color);
```

Problems:

- Shared data repeats.
- Memory grows with every logical object.
- Many duplicate objects are created.
- Performance may suffer.

### With Flyweight

```java
Glyph glyph = factory.getGlyph('A', "Inter");
glyph.draw(x, y, color);
```

Benefits:

- Shared state is stored once.
- Memory footprint can drop significantly.
- Object creation is reduced.
- Common state is centralized.

## 8. Where It Excels

Flyweight excels when:

- There are many similar objects.
- Repeated intrinsic state is large.
- Extrinsic state can be stored separately.
- Shared objects can be immutable.
- Memory pressure is real and measurable.

## 9. Where It Fails

Flyweight is a poor fit when:

- Object count is small.
- Most state is unique.
- Shared objects need mutable per-client state.
- Identity matters.
- Memory savings are not worth added complexity.

## 10. Prebuilt Libraries and Packages

### Java

Flyweight-like examples:

- String literal interning.
- `Integer.valueOf()` and small wrapper caches.
- Font and glyph caches.
- Game asset caches.
- `Enum` constants for repeated values.

### Python

Python examples:

- String interning.
- Small integer caching.
- `functools.lru_cache` for shared computed objects.
- Asset registries and caches.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces memory usage. | Adds cache/factory complexity. |
| Avoids duplicate intrinsic state. | Requires careful state separation. |
| Can reduce object creation. | Identity checks become misleading. |
| Works well with immutable data. | Mutable shared state can cause bugs. |
| Useful at large scale. | May not help without many repeated objects. |

## 12. Real-World Identification Example

Scenario:

You are designing a map app that displays millions of points.

Repeated state:

- Marker icon
- Marker category
- Rendering style

Unique state:

- Latitude
- Longitude
- Label

Should you use Flyweight?

Yes, if shared marker style data is large enough to matter.

Good usage:

```java
MarkerStyle style = styleFactory.get("restaurant");
style.draw(latitude, longitude, label);
```

## 13. MAANG Interview Triggers

Think Flyweight when you hear:

- Huge number of similar objects.
- Memory optimization.
- Shared intrinsic state.
- Extrinsic context.
- Object identity does not matter.
- Text glyphs.
- Game tiles.
- Cache repeated immutable objects.

### Interview-Ready Answer Format

Use this structure when answering:

1. Identify repeated state.
2. Separate intrinsic and extrinsic state.
3. Store intrinsic state in immutable flyweights.
4. Use a factory/cache to reuse flyweights.
5. Pass extrinsic state during operations.
6. Mention trade-off: complexity and identity confusion.

## 14. Common Mistakes

### Mistake 1: Keeping All State Inside the Flyweight

If position and color are stored inside a shared glyph, different occurrences overwrite each other.

### Mistake 2: Mutable Shared State

Flyweights should usually be immutable. Mutable shared state causes cross-client bugs.

### Mistake 3: No Measurement

Use Flyweight for real memory pressure, not as premature optimization.

### Mistake 4: Depending on Object Identity

Two logical objects may use the same flyweight instance.

### Mistake 5: Unbounded Cache

The factory cache may become a memory leak if keys are unbounded.

## 15. Flyweight vs Similar Patterns

| Pattern | Difference |
|---|---|
| Object Pool | Object Pool lends mutable objects. Flyweight shares immutable/reusable intrinsic state. |
| Cache | Cache stores lookup results. Flyweight is about sharing object state to reduce memory. |
| Singleton | Singleton has one global instance. Flyweight has many shared instances by key. |
| Prototype | Prototype copies objects. Flyweight shares objects. |
| Composite | Composite creates tree structures. Composite may use Flyweight for repeated leaves. |

## 16. Flyweight Checklist

| Question | Why it matters |
|---|---|
| Are there many objects? | Flyweight needs scale to pay off. |
| Is repeated state large? | Small state may not be worth sharing. |
| Can shared state be immutable? | Immutability prevents cross-client bugs. |
| Can unique state live outside? | This is the core split. |
| Is cache size bounded? | Avoid memory leaks. |

## 17. Quick Revision Notes

- Flyweight shares intrinsic state.
- Extrinsic state is supplied from outside.
- Use a factory/cache to reuse instances.
- Best for many similar objects.
- Shared state should be immutable.
- Avoid identity-dependent logic.

## 18. Mini Exercise

Design Flyweight for a tile-based map.

Intrinsic state:

- Tile image
- Terrain type
- Movement cost

Extrinsic state:

- x/y position
- visibility
- selected state

Expected usage:

```java
TileType grass = tileFactory.get("grass");
grass.draw(x, y, visible);
```

## 19. Source Reference in This Repo

The repository's Flyweight implementation uses `PotionFactory` to reuse potion instances by `PotionType`.

Useful files:

- [github-repo/flyweight/README.md](../../github-repo/flyweight/README.md)
- [github-repo/flyweight/src/main/java/com/iluwatar/flyweight/PotionFactory.java](../../github-repo/flyweight/src/main/java/com/iluwatar/flyweight/PotionFactory.java)
- [github-repo/flyweight/src/main/java/com/iluwatar/flyweight/Potion.java](../../github-repo/flyweight/src/main/java/com/iluwatar/flyweight/Potion.java)
- [github-repo/flyweight/src/main/java/com/iluwatar/flyweight/PotionType.java](../../github-repo/flyweight/src/main/java/com/iluwatar/flyweight/PotionType.java)
- [github-repo/flyweight/src/main/java/com/iluwatar/flyweight/AlchemistShop.java](../../github-repo/flyweight/src/main/java/com/iluwatar/flyweight/AlchemistShop.java)
