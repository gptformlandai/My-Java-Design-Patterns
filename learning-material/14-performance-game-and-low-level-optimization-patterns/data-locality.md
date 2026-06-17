# Data Locality Pattern

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [data-locality](../../github-repo/data-locality)

---

## How to Study This Page

Study Data Locality as "make the CPU touch nearby data in the order it needs it."

Remember the core idea:

```text
Performance is not only algorithm complexity.
Memory access pattern also matters.
```

This pattern becomes important when you are processing large arrays, game entities, simulation objects, analytics buffers, or tight loops where cache misses dominate runtime.

---

## 1. Technical Definition

Data Locality is an optimization pattern that arranges and processes data so values accessed together are stored near each other in memory, improving CPU cache utilization and reducing memory access latency.

### 30-Second Interview Answer

Data Locality improves performance by organizing data around how it is accessed. Instead of jumping across many object graphs, we keep related data in contiguous arrays and process same-type data together. This increases cache hits and reduces memory stalls. I would consider it for game loops, physics engines, analytics, vectorized processing, and high-throughput systems. The trade-off is that code may become less object-oriented, less flexible, and harder to maintain unless profiling proves the need.

---

## 2. Layman and Easy to Understand Definition

Imagine cooking with all ingredients for one step placed beside you.

If you need to walk across the kitchen for every spice, cooking slows down. If the next few ingredients are already on the counter, you move faster.

Data Locality does the same thing for the CPU: keep the next needed data close.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Object-oriented code often scatters data:

```text
entity -> ai component somewhere
entity -> physics component somewhere else
entity -> render component somewhere else
```

When a loop updates thousands of objects, the CPU may spend time waiting for memory instead of doing useful work.

### Data Locality Flow

1. Identify a hot loop or data-heavy path.
2. Measure that memory access is a bottleneck.
3. Group data by access pattern.
4. Store frequently processed data in contiguous arrays or compact structures.
5. Iterate linearly through related data.
6. Keep cold or rarely used data out of the hot path.

### Core Participants

| Participant | Responsibility |
|---|---|
| Hot loop | Performance-sensitive repeated processing |
| Hot data | Fields needed by the hot loop |
| Cold data | Fields rarely needed in the hot loop |
| Contiguous storage | Arrays or compact buffers |
| Access pattern | Order in which data is read/written |
| Cache behavior | Whether nearby data is reused efficiently |

---

## 4. Java Coding Example

```java
class PhysicsSystem {
    private final float[] x;
    private final float[] y;
    private final float[] velocityX;
    private final float[] velocityY;

    PhysicsSystem(int count) {
        this.x = new float[count];
        this.y = new float[count];
        this.velocityX = new float[count];
        this.velocityY = new float[count];
    }

    void update(float deltaSeconds) {
        for (int i = 0; i < x.length; i++) {
            x[i] += velocityX[i] * deltaSeconds;
            y[i] += velocityY[i] * deltaSeconds;
        }
    }
}

public class DataLocalityDemo {
    public static void main(String[] args) {
        PhysicsSystem physics = new PhysicsSystem(10_000);
        physics.update(0.016f);
    }
}
```

### Java Block by Block

The physics data is stored in arrays instead of many scattered objects.

The update loop walks forward through each array.

The CPU can load nearby values efficiently.

This style is common in entity-component-system designs and high-performance simulations.

---

## 5. Python Coding Example

```python
class PhysicsSystem:
    def __init__(self, count):
        self.x = [0.0] * count
        self.y = [0.0] * count
        self.vx = [1.0] * count
        self.vy = [1.0] * count

    def update(self, delta_seconds):
        for i in range(len(self.x)):
            self.x[i] += self.vx[i] * delta_seconds
            self.y[i] += self.vy[i] * delta_seconds


physics = PhysicsSystem(10_000)
physics.update(0.016)
```

In real Python performance work, libraries such as NumPy are usually better because they store numeric data compactly and run vectorized loops in native code.

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Game loops | Thousands of entities need the same update each frame |
| Physics simulation | Numeric arrays are processed repeatedly |
| Rendering engines | Vertex, transform, and material data are accessed in batches |
| Analytics pipelines | Large arrays or columns are scanned repeatedly |
| Databases | Columnar storage benefits scans over selected fields |
| Machine learning | Tensor operations rely on contiguous memory and vectorization |

---

## 7. Advantages Over Normal Code Without Pattern

Without Data Locality:
- hot loops may chase scattered object references
- CPU cache misses can dominate runtime
- memory bandwidth is wasted
- object allocation and garbage collection pressure may increase

With Data Locality:
- data is processed in predictable order
- cache hit rate improves
- memory bandwidth is used more efficiently
- hot code becomes easier to vectorize or parallelize

---

## 8. Where It Excels

It excels when:
- profiling shows memory access is a bottleneck
- data is processed in large batches
- the same fields are accessed repeatedly
- object graphs are too pointer-heavy
- latency or frame time is strict
- the access pattern is stable enough to design around

---

## 9. Where It Fails

It fails when:
- the code path is not performance-critical
- object modeling clarity matters more than raw speed
- data access patterns change often
- premature optimization makes code harder to maintain
- concurrency introduces false sharing
- the runtime/language prevents meaningful memory layout control

Use clear domain objects first, then optimize data layout after measurement shows the need.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Primitive arrays, ByteBuffer, Agrona, fastutil, Eclipse Collections primitive collections |
| JVM game/dev | LibGDX, Artemis-ODB, Ashley ECS |
| Python | NumPy, pandas columnar operations, PyArrow |
| C/C++ | Struct-of-arrays layouts, SIMD libraries, data-oriented design |
| Databases | Columnar stores such as Parquet, ORC, ClickHouse |
| Hardware-aware work | JMH, perf, async-profiler, Java Flight Recorder |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Better cache utilization | More complex data layout |
| Lower memory access latency | Less natural object modeling |
| Can reduce allocation pressure | Harder to evolve if access patterns change |
| Helps batch processing | Requires profiling knowledge |
| Can enable vectorization | May introduce false sharing or alignment issues |

---

## 12. Real-World Identification Example

Question:

> A game server updates 100,000 entities every tick. Profiling shows most time is spent waiting on memory while traversing entity objects and nested components. What pattern helps?

Strong answer:

Use Data Locality. I would identify the hot fields needed by each system, store them in compact arrays or component-specific buffers, and update one component type at a time. For example, run all AI updates, then all physics updates, then all rendering-related updates. I would verify improvement with profiling because this pattern trades design flexibility for performance.

---

## 13. MAANG Interview Triggers

Say Data Locality when you hear:
- CPU cache misses
- memory access bottleneck
- game loop performance
- entity component system
- hot loop over many objects
- object graph too scattered
- columnar storage
- data-oriented design

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Optimizing before profiling | Adds complexity without proof | Measure first |
| Keeping hot and cold data together | Wastes cache space | Split hot data from cold data |
| Using boxed types in hot arrays | Adds indirection and allocation | Prefer primitives or compact buffers |
| Ignoring access order | Layout alone may not help | Iterate in the same order data is stored |
| Overfitting to one workload | Future changes become painful | Document the access pattern and benchmark |

---

## 15. Data Locality vs Similar Patterns

| Pattern | Difference |
|---|---|
| Flyweight | Flyweight reduces duplicated object state; Data Locality optimizes memory layout and access order |
| Object Pool | Object Pool reuses objects; Data Locality improves cache-friendly processing |
| Iterator | Iterator controls traversal; Data Locality designs storage to make traversal efficient |
| Collection Pipeline | Collection Pipeline improves expression of transformations; Data Locality improves low-level access performance |
| Spatial Partition | Spatial Partition reduces spatial search work; Data Locality reduces memory access cost |

---

## 16. Data Locality Design Checklist

- What loop is performance-critical?
- What fields are read or written in that loop?
- Are those fields stored together or scattered?
- Is object allocation affecting the hot path?
- Can hot and cold data be separated?
- Can primitive arrays or compact buffers help?
- Does iteration order match memory layout?
- Have benchmarks proven the improvement?
- How will maintainers understand the optimized layout?

---

## 17. Quick Revision Notes

- One-line summary: Store and process related hot data together to improve cache efficiency.
- Memory hook: "near data is fast data."
- Best for: game loops, simulations, numeric processing, column scans.
- Avoid when: no measured performance problem exists.
- Interview line: "I would profile first, then group hot fields by access pattern and process them linearly."

---

## 18. Mini Exercise

Refactor a particle update loop:
- identify hot fields such as position, velocity, and lifetime
- separate hot fields from rarely used metadata
- store hot fields in arrays
- update particles in a linear loop
- define one benchmark to prove improvement

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/data-locality/README.md)
- [Application.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/Application.java)
- [GameEntity.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/game/GameEntity.java)
- [AiComponentManager.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/game/component/manager/AiComponentManager.java)
- [PhysicsComponentManager.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/game/component/manager/PhysicsComponentManager.java)
- [RenderComponentManager.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/game/component/manager/RenderComponentManager.java)
- [AiComponent.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/game/component/AiComponent.java)
- [PhysicsComponent.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/game/component/PhysicsComponent.java)
- [RenderComponent.java](../../github-repo/data-locality/src/main/java/com/iluwatar/data/locality/game/component/RenderComponent.java)

The repo implementation groups AI, physics, and render components into separate managers. The game update processes one component type across all entities before moving to the next component type, which demonstrates cache-friendly batch processing.
