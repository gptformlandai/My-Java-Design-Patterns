# Iterator Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: High  
Software usage meter: Very High  
Repository module: [github-repo/iterator](../../github-repo/iterator)

## How to Study This Page

Use this page in three passes:

1. First pass: understand traversal without exposing internal structure.
2. Second pass: rewrite the Java example and identify aggregate, iterator, current position, and traversal rule.
3. Third pass: study fail-fast behavior, snapshot iteration, lazy iteration, and concurrent modification.

By the end, you should be able to say:

> Iterator provides a standard way to traverse a collection without exposing how that collection is stored.

## 1. Technical Definition

Iterator is a behavioral design pattern that provides sequential access to elements of an aggregate object without exposing the aggregate's internal representation.

Core idea:

- Collection creates an iterator.
- Iterator tracks traversal position.
- Client uses `hasNext()` and `next()`.
- Collection internals remain hidden.

### 30-Second Interview Answer

I would use Iterator when clients need to traverse a collection or custom data structure without knowing its storage details. The iterator owns traversal state and exposes a simple interface such as `hasNext()` and `next()`. It is useful for lists, trees, filtered views, lazy streams, and custom traversal orders. Trade-offs include concurrent modification behavior and whether the iterator is snapshot, fail-fast, or live.

## 2. Layman and Easy to Understand Definition

Iterator is like a bookmark moving through a book.

You do not need to know how the book was printed or bound. You just ask:

```text
Is there a next page?
Give me the next page.
```

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose a playlist stores songs internally.

Bad design:

```java
for (int i = 0; i < playlist.internalArray.length; i++) {
    play(playlist.internalArray[i]);
}
```

Problems:

- Client knows internal storage.
- Storage cannot change without breaking clients.
- Custom traversal order is hard.
- Filtering logic gets repeated.

### 3.2 The Iterator Solution

Expose an iterator:

```java
Iterator<Song> iterator = playlist.iterator();
while (iterator.hasNext()) {
    play(iterator.next());
}
```

The playlist can store songs however it wants.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Aggregate | Collection being traversed. |
| Iterator | Object that tracks traversal state. |
| Concrete iterator | Specific traversal implementation. |
| Element | Object returned during traversal. |
| Client | Uses iterator without knowing internals. |

### 3.4 Traversal Styles

| Style | Example |
|---|---|
| Forward | List from first to last. |
| Reverse | List from last to first. |
| Filtered | Only active items. |
| Tree traversal | In-order, pre-order, breadth-first. |
| Lazy | Produces items as needed. |

## 4. Java Coding Example

This example iterates over a playlist.

```java
import java.util.List;
import java.util.NoSuchElementException;

public record Song(String title, boolean favorite) {}

public interface SongIterator {
    boolean hasNext();
    Song next();
}

public final class Playlist {
    private final List<Song> songs;

    public Playlist(List<Song> songs) {
        this.songs = List.copyOf(songs);
    }

    public SongIterator iterator() {
        return new AllSongsIterator(songs);
    }

    public SongIterator favoriteIterator() {
        return new FavoriteSongIterator(songs);
    }
}

public final class AllSongsIterator implements SongIterator {
    private final List<Song> songs;
    private int index;

    public AllSongsIterator(List<Song> songs) {
        this.songs = songs;
    }

    @Override
    public boolean hasNext() {
        return index < songs.size();
    }

    @Override
    public Song next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        return songs.get(index++);
    }
}

public final class FavoriteSongIterator implements SongIterator {
    private final List<Song> songs;
    private int index;

    public FavoriteSongIterator(List<Song> songs) {
        this.songs = songs;
    }

    @Override
    public boolean hasNext() {
        while (index < songs.size() && !songs.get(index).favorite()) {
            index++;
        }
        return index < songs.size();
    }

    @Override
    public Song next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        return songs.get(index++);
    }
}
```

### Java Block by Block Explanation

#### Aggregate

```java
public final class Playlist {
```

The aggregate owns the collection and creates iterators.

#### Iterator Interface

```java
public interface SongIterator {
    boolean hasNext();
    Song next();
}
```

This is the traversal contract.

#### Traversal State

```java
private int index;
```

The iterator owns current position.

#### Filtered Iterator

```java
while (index < songs.size() && !songs.get(index).favorite()) {
    index++;
}
```

The iterator hides filtering logic from the client.

### Java Usage

```java
import java.util.List;

public class Demo {
    public static void main(String[] args) {
        Playlist playlist = new Playlist(List.of(
            new Song("Intro", false),
            new Song("Favorite Track", true)
        ));

        SongIterator iterator = playlist.favoriteIterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next().title());
        }
    }
}
```

## 5. Python Coding Example

Python has built-in iterator protocol.

```python
from dataclasses import dataclass
from collections.abc import Iterator


@dataclass(frozen=True)
class Song:
    title: str
    favorite: bool


class Playlist:
    def __init__(self, songs: list[Song]) -> None:
        self._songs = list(songs)

    def __iter__(self) -> Iterator[Song]:
        return iter(self._songs)

    def favorites(self) -> Iterator[Song]:
        return (song for song in self._songs if song.favorite)
```

### Python Usage

```python
playlist = Playlist([Song("Intro", False), Song("Favorite Track", True)])
for song in playlist.favorites():
    print(song.title)
```

## 6. Where It Comes Handy in Real Life

Examples:

- Java collections.
- Tree traversal.
- Database cursors.
- Pagination.
- Stream processing.
- File line iteration.
- Filtered views.
- UI component traversal.

## 7. Advantages Over Normal Code Without Pattern

### Without Iterator

```java
playlist.songs.get(i);
```

Problems:

- Internal storage leaks.
- Traversal logic repeats.
- Changing storage breaks clients.
- Custom traversal is awkward.

### With Iterator

```java
while (iterator.hasNext()) {
    play(iterator.next());
}
```

Benefits:

- Internals are hidden.
- Multiple traversal styles are possible.
- Client code is uniform.
- Traversal state is isolated.

## 8. Where It Excels

Iterator excels when:

- Data structure internals should be hidden.
- Multiple traversal orders are needed.
- Traversal state should be separate from collection.
- Collections may be lazy or remote.
- Client code should be storage-agnostic.

## 9. Where It Fails

Iterator is a poor fit when:

- Direct indexed access is required for performance.
- Traversal is trivial and language constructs already solve it.
- Concurrent modification rules are unclear.
- Iterator lifecycle is hard to manage.
- The collection is tiny and custom iterator adds ceremony.

## 10. Prebuilt Libraries and Packages

### Java

Examples:

- `java.util.Iterator`
- `Iterable`
- `ListIterator`
- Streams
- Spliterator
- JDBC `ResultSet` behaves cursor-like

### Python

Examples:

- `__iter__`
- `__next__`
- Generators
- `itertools`
- Database cursors

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Hides collection internals. | Concurrent modification can be tricky. |
| Supports multiple traversal styles. | Extra object/state. |
| Standardizes client code. | Iterator may become invalid. |
| Works for lazy traversal. | Error semantics must be clear. |
| Separates traversal from collection. | Can hide expensive traversal. |

## 12. Real-World Identification Example

Scenario:

You are designing a tree of comments and replies.

Traversal needs:

- Depth-first.
- Breadth-first.
- Only visible comments.

Should you use Iterator?

Yes, if clients should not know tree internals.

## 13. MAANG Interview Triggers

Think Iterator when you hear:

- Traverse collection.
- Hide internal representation.
- Cursor.
- `hasNext()` and `next()`.
- Tree traversal.
- Lazy sequence.
- Filtered traversal.

### Interview-Ready Answer Format

1. Identify the aggregate.
2. Define iterator interface.
3. Store traversal state in iterator.
4. Let aggregate create iterator.
5. Support needed traversal styles.
6. Mention trade-offs: concurrent modification and lifecycle.

## 14. Common Mistakes

### Mistake 1: Exposing Internal Collection

Returning mutable internal lists defeats encapsulation.

### Mistake 2: Undefined End Behavior

Define what happens when `next()` is called after the end.

### Mistake 3: Ignoring Modification During Iteration

Decide fail-fast, snapshot, or live behavior.

### Mistake 4: Iterator Does Too Much

Complex business filtering may belong elsewhere.

### Mistake 5: Hiding Expensive Remote Calls

If `next()` fetches from network, document latency and failure behavior.

## 15. Iterator vs Similar Patterns

| Pattern | Difference |
|---|---|
| Composite | Composite models trees. Iterator traverses them. |
| Visitor | Visitor performs operations across structures. Iterator returns elements one by one. |
| Cursor | Cursor is a common concrete form of Iterator. |
| Stream | Stream can be a higher-level lazy traversal pipeline. |
| Repository | Repository retrieves collections; Iterator traverses them. |

## 16. Iterator Design Checklist

| Concern | Why it matters |
|---|---|
| Traversal order | Defines user-visible behavior. |
| End behavior | Avoids ambiguous errors. |
| Mutation policy | Prevents subtle bugs. |
| Lazy or eager | Affects memory and latency. |
| Resource cleanup | Needed for cursors/files/network streams. |

## 17. Quick Revision Notes

- Iterator traverses without exposing internals.
- Iterator owns traversal state.
- Aggregate creates iterator.
- Supports multiple traversal styles.
- Java uses `Iterator` and `Iterable`.
- Watch concurrent modification and resource cleanup.

## 18. Mini Exercise

Design iterators for `TaskBoard`.

Traversal styles:

- All tasks.
- Only overdue tasks.
- Tasks by assignee.

Expected usage:

```java
TaskIterator iterator = board.overdueTasks();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

## 19. Source Reference in This Repo

The repository's Iterator implementation includes custom iterators for a treasure chest and a binary search tree.

Useful files:

- [github-repo/iterator/README.md](../../github-repo/iterator/README.md)
- [github-repo/iterator/src/main/java/com/iluwatar/iterator/Iterator.java](../../github-repo/iterator/src/main/java/com/iluwatar/iterator/Iterator.java)
- [github-repo/iterator/src/main/java/com/iluwatar/iterator/list/TreasureChest.java](../../github-repo/iterator/src/main/java/com/iluwatar/iterator/list/TreasureChest.java)
- [github-repo/iterator/src/main/java/com/iluwatar/iterator/list/TreasureChestItemIterator.java](../../github-repo/iterator/src/main/java/com/iluwatar/iterator/list/TreasureChestItemIterator.java)
- [github-repo/iterator/src/main/java/com/iluwatar/iterator/bst/BstIterator.java](../../github-repo/iterator/src/main/java/com/iluwatar/iterator/bst/BstIterator.java)
