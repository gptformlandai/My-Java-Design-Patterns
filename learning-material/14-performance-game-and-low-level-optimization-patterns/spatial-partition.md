# Spatial Partition Pattern

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Medium  
Software usage meter: Low  
Repository module: [spatial-partition](../../github-repo/spatial-partition)

---

## How to Study This Page

Study Spatial Partition as "divide space so you only search nearby things."

Remember the core idea:

```text
Instead of checking every object against every object,
put objects into spatial regions and query only relevant regions.
```

This pattern is especially useful in game engines, simulations, collision detection, maps, and spatial databases.

---

## 1. Technical Definition

Spatial Partition is an optimization pattern that divides a physical or logical space into regions and stores objects by location so spatial queries can inspect only relevant nearby objects.

### 30-Second Interview Answer

Spatial Partition reduces expensive spatial searches. Instead of comparing every object with every other object, we store objects in a grid, quadtree, octree, k-d tree, or similar spatial index. Then queries like "near me", collision detection, or visible objects only inspect nearby regions. It is useful in games, maps, simulations, and rendering. The trade-off is extra data structure complexity, memory overhead, and update cost when objects move.

---

## 2. Layman and Easy to Understand Definition

Imagine finding a person in a city.

Searching every person in the entire city is slow. If you know the neighborhood first, you search a much smaller area.

Spatial Partition gives software that same shortcut by dividing space into neighborhoods.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Naive spatial checks often use all-pairs comparison:

```text
for each object A
  for each object B
    check distance or collision
```

For many objects, this becomes expensive very quickly.

### Spatial Partition Flow

1. Define the world bounds.
2. Choose a partition structure such as grid, quadtree, octree, or k-d tree.
3. Insert objects based on their coordinates.
4. For a query, find only regions that overlap the query area.
5. Check exact collisions or distances only among returned candidates.
6. Update or rebuild the partition when objects move.

### Core Participants

| Participant | Responsibility |
|---|---|
| World space | Area being partitioned |
| Spatial index | Grid/tree that stores objects by location |
| Region/cell/node | Smaller area inside the world |
| Object position | Coordinates used for insertion |
| Query range | Area being searched |
| Candidate set | Nearby objects that need exact checks |

---

## 4. Java Coding Example

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

record Unit(int id, int x, int y) {}

class SpatialGrid {
    private final int cellSize;
    private final Map<String, List<Unit>> cells = new HashMap<>();

    SpatialGrid(int cellSize) {
        this.cellSize = cellSize;
    }

    void insert(Unit unit) {
        cells.computeIfAbsent(key(unit.x(), unit.y()), ignored -> new ArrayList<>())
                .add(unit);
    }

    List<Unit> nearby(Unit unit) {
        List<Unit> result = new ArrayList<>();
        int cx = unit.x() / cellSize;
        int cy = unit.y() / cellSize;

        for (int dx = -1; dx <= 1; dx++) {
            for (int dy = -1; dy <= 1; dy++) {
                result.addAll(cells.getOrDefault((cx + dx) + ":" + (cy + dy), List.of()));
            }
        }
        return result;
    }

    private String key(int x, int y) {
        return (x / cellSize) + ":" + (y / cellSize);
    }
}

public class SpatialPartitionDemo {
    public static void main(String[] args) {
        SpatialGrid grid = new SpatialGrid(10);
        Unit a = new Unit(1, 12, 18);
        Unit b = new Unit(2, 14, 17);
        Unit c = new Unit(3, 90, 90);

        grid.insert(a);
        grid.insert(b);
        grid.insert(c);

        System.out.println(grid.nearby(a));
    }
}
```

### Java Block by Block

The grid divides the world into fixed-size cells.

Each unit is inserted into the cell matching its coordinates.

`nearby` checks the unit's cell and adjacent cells.

The final exact collision or distance check would run only on these candidates.

---

## 5. Python Coding Example

```python
from collections import defaultdict


class SpatialGrid:
    def __init__(self, cell_size):
        self.cell_size = cell_size
        self.cells = defaultdict(list)

    def _key(self, x, y):
        return x // self.cell_size, y // self.cell_size

    def insert(self, unit):
        self.cells[self._key(unit["x"], unit["y"])].append(unit)

    def nearby(self, unit):
        cx, cy = self._key(unit["x"], unit["y"])
        result = []
        for dx in (-1, 0, 1):
            for dy in (-1, 0, 1):
                result.extend(self.cells.get((cx + dx, cy + dy), []))
        return result


grid = SpatialGrid(cell_size=10)
for unit in [{"id": 1, "x": 12, "y": 18}, {"id": 2, "x": 14, "y": 17}, {"id": 3, "x": 90, "y": 90}]:
    grid.insert(unit)

print(grid.nearby({"id": 1, "x": 12, "y": 18}))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Collision detection | Only nearby objects can collide |
| Game AI perception | Units only need to inspect nearby entities |
| Rendering visibility | Only objects in visible regions need rendering |
| Map search | Nearby restaurants, drivers, or locations can be indexed spatially |
| Simulation | Particles or agents interact with local neighbors |
| Ray tracing | Spatial structures reduce intersection checks |

---

## 7. Advantages Over Normal Code Without Pattern

Without Spatial Partition:
- every object may be checked against every other object
- collision detection becomes slow as object count grows
- many distance checks are wasted on far-away objects
- large worlds are hard to query efficiently

With Spatial Partition:
- queries inspect relevant regions first
- exact checks run on fewer candidates
- large worlds scale better
- nearby-object features become practical

---

## 8. Where It Excels

It excels when:
- many objects exist in space
- most interactions are local
- queries repeat often
- object count is large enough to justify index overhead
- objects can be assigned to regions efficiently
- approximate candidate filtering is acceptable before exact checks

---

## 9. Where It Fails

It fails when:
- object count is small
- almost every object interacts with every other object
- objects move so often that index maintenance dominates
- region sizes are chosen poorly
- data distribution is extremely skewed
- correctness depends on checking exact geometry but candidate filtering is wrong

Use a simple loop for tiny datasets, or choose a different spatial index when a fixed grid, quadtree, or octree does not match the distribution.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | JTS Topology Suite, Lucene spatial, GeoTools |
| Databases | PostGIS, MongoDB geospatial indexes, Elasticsearch geo queries |
| Games | Box2D broad-phase collision, Bullet broadphase, Unity physics spatial structures |
| Python | Shapely, GeoPandas spatial indexes, scipy.spatial KDTree |
| Graphics | BVH, octree, k-d tree implementations |
| Cloud/maps | H3, S2 Geometry, geohash-based indexes |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces spatial query cost | Adds data structure complexity |
| Avoids many unnecessary checks | Uses extra memory |
| Scales better for large worlds | Moving objects require updates |
| Good for collision and range queries | Poor partition choice hurts performance |
| Works with many index types | Edge cases near region boundaries need care |

---

## 12. Real-World Identification Example

Question:

> A 2D game has 20,000 moving objects. Collision detection checks every object against every other object each frame, causing frame drops. What pattern helps?

Strong answer:

Use Spatial Partition. I would divide the world into a grid or quadtree, insert objects by position each frame or update their cells as they move, and query only nearby regions for collision candidates. Then I would run exact collision checks only on those candidates. The key trade-offs are index rebuild cost, memory overhead, and choosing the right cell size or tree capacity.

---

## 13. MAANG Interview Triggers

Say Spatial Partition when you hear:
- collision detection
- nearest neighbors
- range query
- many objects in 2D or 3D space
- avoid all-pairs comparison
- quadtree or octree
- geospatial index
- broad-phase collision detection

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Using it for tiny datasets | Index overhead may exceed savings | Use simple loops first |
| Bad cell size | Too many or too few objects per region | Tune with real data |
| Ignoring boundary cases | Nearby objects in adjacent cells are missed | Query neighboring regions too |
| Not updating moving objects | Index becomes stale | Reinsert, update, or rebuild as needed |
| Assuming candidates are exact matches | False positives still exist | Run exact collision/distance checks after query |

---

## 15. Spatial Partition vs Similar Patterns

| Pattern | Difference |
|---|---|
| Data Locality | Data Locality improves memory access; Spatial Partition reduces spatial search work |
| Flyweight | Flyweight reduces duplicated object state; Spatial Partition indexes object locations |
| Composite | Trees such as quadtrees are hierarchical, but the goal is spatial querying |
| Caching | Caching reuses previous results; Spatial Partition narrows current query candidates |
| Repository | Repository abstracts data access; Spatial Partition optimizes location-based access |

---

## 16. Spatial Partition Design Checklist

- What spatial queries must be fast?
- Are objects in 2D, 3D, or higher-dimensional space?
- How many objects exist?
- How often do objects move?
- Should the structure be a grid, quadtree, octree, k-d tree, BVH, geohash, H3, or S2?
- What region size or capacity should be used?
- How are boundary cases handled?
- Are exact checks still performed after candidate lookup?
- How will performance be measured?

---

## 17. Quick Revision Notes

- One-line summary: Divide space into regions so spatial queries check fewer objects.
- Memory hook: "search the neighborhood, not the whole world."
- Best for: collision detection, nearby search, map queries, simulations.
- Avoid when: object count is small or movement/update cost dominates.
- Interview line: "I would use the spatial index as a broad phase, then run exact checks on returned candidates."

---

## 18. Mini Exercise

Design spatial partitioning for delivery-driver search:
- choose a spatial index
- define how drivers are inserted or updated
- query nearby drivers for a pickup point
- handle drivers near cell boundaries
- explain how you would benchmark query latency

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/spatial-partition/README.md)
- [App.java](../../github-repo/spatial-partition/src/main/java/com/iluwatar/spatialpartition/App.java)
- [QuadTree.java](../../github-repo/spatial-partition/src/main/java/com/iluwatar/spatialpartition/QuadTree.java)
- [Rect.java](../../github-repo/spatial-partition/src/main/java/com/iluwatar/spatialpartition/Rect.java)
- [Point.java](../../github-repo/spatial-partition/src/main/java/com/iluwatar/spatialpartition/Point.java)
- [Bubble.java](../../github-repo/spatial-partition/src/main/java/com/iluwatar/spatialpartition/Bubble.java)
- [SpatialPartitionGeneric.java](../../github-repo/spatial-partition/src/main/java/com/iluwatar/spatialpartition/SpatialPartitionGeneric.java)
- [SpatialPartitionBubbles.java](../../github-repo/spatial-partition/src/main/java/com/iluwatar/spatialpartition/SpatialPartitionBubbles.java)

The repo implementation compares all-bubble collision checks with a quadtree-based approach. `QuadTree` inserts points by region and queries rectangular ranges, while `SpatialPartitionBubbles` uses those query results as collision candidates.
