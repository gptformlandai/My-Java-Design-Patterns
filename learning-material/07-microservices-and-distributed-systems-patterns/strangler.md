# Strangler Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/strangler](../../github-repo/strangler)

## How to Study This Page

Use this page in three passes:

1. First pass: understand the pattern as incremental replacement of a legacy system.
2. Second pass: trace routing from old implementation to partial migration to new implementation.
3. Third pass: compare Strangler with big-bang rewrite, branch by abstraction, and anti-corruption layer.

By the end, you should be able to say:

> Strangler replaces a legacy system gradually by routing slices of functionality to a new implementation.

## 1. Technical Definition

Strangler is a modernization pattern where new functionality is built around an existing system and traffic is gradually redirected until the old system can be retired.

Core idea:

- Keep the old system running.
- Build new capability beside it.
- Route selected features or users to the new path.
- Validate behavior during migration.
- Remove old functionality after traffic is fully moved.

### 30-Second Interview Answer

I would use Strangler when a legacy system is too risky to rewrite all at once. I would place a routing layer in front, migrate one capability at a time, compare results, and gradually move traffic to new services. The trade-off is temporary duplication and routing complexity, but it reduces migration risk and keeps the business running.

## 2. Layman and Easy to Understand Definition

Strangler is like replacing an old store section by section while the store stays open.

You do not close everything for a massive rebuild. You modernize one area, move customers to it, then repeat until the old area is gone.

In code:

- Old system keeps serving.
- New system handles one migrated feature.
- Router decides old versus new.
- Migration expands gradually.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Legacy systems often cannot be replaced in one release:

- Too much hidden behavior.
- Too many dependencies.
- Business cannot tolerate downtime.
- Full rewrite may take years.
- Requirements are not fully known.

### 3.2 The Strangler Solution

Put a controlled boundary in front:

```text
client -> router -> old system
client -> router -> new service
```

Move one feature at a time.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Legacy system | Existing application being replaced. |
| New system | Replacement capability. |
| Routing layer | Chooses old or new path. |
| Feature slice | Migrated business capability. |
| Compatibility layer | Keeps old and new models working together. |
| Cutover plan | Steps to move traffic and retire old code. |

### 3.4 Migration Flow

1. Identify one bounded feature.
2. Put routing in front of old system.
3. Build new implementation for that feature.
4. Route a small amount of traffic to new implementation.
5. Compare behavior and fix gaps.
6. Increase traffic.
7. Retire the old feature path.

## 4. Java Coding Example

This example routes migrated operations to a new implementation while keeping old operations on the legacy implementation.

```java
interface PricingService {
    int priceFor(String sku);
}

class LegacyPricingService implements PricingService {
    public int priceFor(String sku) {
        return 100;
    }
}

class NewPricingService implements PricingService {
    public int priceFor(String sku) {
        return 95;
    }
}

class StranglerRouter implements PricingService {
    private final PricingService legacy;
    private final PricingService modern;

    StranglerRouter(PricingService legacy, PricingService modern) {
        this.legacy = legacy;
        this.modern = modern;
    }

    public int priceFor(String sku) {
        if (sku.startsWith("NEW-")) {
            return modern.priceFor(sku);
        }
        return legacy.priceFor(sku);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `PricingService` | Stable interface in front of both systems. |
| `LegacyPricingService` | Existing behavior remains available. |
| `NewPricingService` | Replacement implementation. |
| `StranglerRouter` | Cutover logic lives at the boundary. |
| `sku.startsWith` | Represents feature/user/tenant routing. |

### Java Usage

```java
PricingService service = new StranglerRouter(
    new LegacyPricingService(),
    new NewPricingService()
);

System.out.println(service.priceFor("OLD-1"));
System.out.println(service.priceFor("NEW-1"));
```

## 5. Python Coding Example

```python
class LegacyPricing:
    def price_for(self, sku):
        return 100


class NewPricing:
    def price_for(self, sku):
        return 95


class StranglerRouter:
    def __init__(self, legacy, modern):
        self.legacy = legacy
        self.modern = modern

    def price_for(self, sku):
        if sku.startswith("NEW-"):
            return self.modern.price_for(sku)
        return self.legacy.price_for(sku)


router = StranglerRouter(LegacyPricing(), NewPricing())
print(router.price_for("OLD-1"))
print(router.price_for("NEW-1"))
```

### Python Usage

Use this shape when explaining:

- Both old and new implementations coexist.
- Routing controls migration.
- Feature slices can move gradually.

## 6. Where It Comes Handy in Real Life

- Monolith to microservices migration.
- Legacy UI replacement.
- Database modernization.
- Replacing old billing or order systems.
- Moving from on-prem systems to cloud.
- Incremental platform rewrite.

## 7. Advantages Over Normal Code Without Pattern

Without Strangler:

```text
rewrite everything, switch everything, hope it works
```

With Strangler:

```text
migrate one slice, validate, expand
```

Benefits:

- Reduces rewrite risk.
- Keeps the system running.
- Enables incremental delivery.
- Allows real traffic validation.
- Makes rollback easier per slice.

## 8. Where It Excels

- Legacy system is large and risky.
- Business needs continuous operation.
- Features can be separated gradually.
- Traffic can be routed by path, user, tenant, or feature.
- Teams can deliver migration incrementally.

## 9. Where It Fails

- Old and new systems must share highly coupled state.
- No clear feature boundaries exist.
- Routing rules become unmanageable.
- Data synchronization is ignored.
- The migration never retires old paths.

Define a retirement plan for every migrated slice.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Routing | API Gateway, NGINX, Envoy, Spring Cloud Gateway |
| Release control | Feature flags, LaunchDarkly, Unleash, OpenFeature |
| Migration testing | Shadow traffic, contract tests, golden-master tests |
| Integration | Anti-Corruption Layer, adapters, CDC pipelines |
| Observability | Distributed tracing, metrics, comparison dashboards |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Lowers migration risk. | Requires temporary old/new coexistence. |
| Supports incremental delivery. | Adds routing and synchronization complexity. |
| Enables rollback per feature. | Can prolong migration if not governed. |
| Keeps business running. | Duplicate implementations may exist temporarily. |

## 12. Real-World Identification Example

Scenario: A retailer wants to migrate order history out of a legacy monolith.

Strangler fit:

- Gateway routes `/orders/history` to new service for beta users.
- Other order routes stay on monolith.
- New service reads replicated order data.
- Traffic gradually expands after validation.
- Old route is removed when migration completes.

Without it:

- Full rewrite delays value.
- Cutover risk is concentrated in one release.
- Rollback is painful.

## 13. MAANG Interview Triggers

Use Strangler when you hear:

- "Migrate a monolith to microservices."
- "Cannot rewrite everything at once."
- "Need zero downtime modernization."
- "How do we replace a legacy system safely?"
- "How do we gradually move traffic?"
- "How do we reduce migration risk?"

Strong answer keywords:

- incremental migration
- routing layer
- feature slice
- shadow traffic
- dual write
- cutover
- rollback
- retirement plan
- anti-corruption layer

## 14. Common Mistakes

### Mistake 1: No clear slice boundaries

- Why it is wrong: migration gets tangled with many dependencies.
- Better approach: migrate bounded capabilities with clear APIs.

### Mistake 2: Forgetting data migration

- Why it is wrong: new code cannot work without reliable data access.
- Better approach: design replication, backfill, dual write, or ownership transfer.

### Mistake 3: Running old and new forever

- Why it is wrong: cost and complexity never go away.
- Better approach: define decommission milestones.

### Mistake 4: No behavior comparison

- Why it is wrong: new path may drift from expected behavior.
- Better approach: use shadow traffic, contract tests, and reconciliation reports.

## 15. Strangler vs Similar Patterns

| Pattern | Difference |
|---|---|
| Strangler | Gradually replaces old system with new system. |
| Big-Bang Rewrite | Replaces everything at once. |
| Branch by Abstraction | Adds an abstraction to swap implementation safely inside code. |
| Anti-Corruption Layer | Protects model boundaries during integration. |
| Feature Flag | Controls exposure; often used inside strangler migration. |

## 16. Strangler Design Checklist

- What is the first feature slice?
- What routing layer controls traffic?
- How is data synchronized?
- How do you compare old and new behavior?
- What is the rollback path?
- What metrics prove the new path works?
- What old code will be deleted?
- What is the final decommission date?

## 17. Quick Revision Notes

- One-line summary: Strangler modernizes legacy systems one slice at a time.
- Three keywords: route, migrate, retire.
- Interview trap: migrating without a plan to delete old paths.
- Memory trick: do not replace the whole building; move one room at a time.

## 18. Mini Exercise

Plan a Strangler migration for a legacy checkout system.

Answer these:

1. What feature moves first?
2. How will traffic route old versus new?
3. What data must be copied or shared?
4. How will you validate behavior?
5. When can the old feature be removed?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/strangler/README.md](../../github-repo/strangler/README.md)
- [github-repo/strangler/src/main/java/com/iluwatar/strangler/OldArithmetic.java](../../github-repo/strangler/src/main/java/com/iluwatar/strangler/OldArithmetic.java)
- [github-repo/strangler/src/main/java/com/iluwatar/strangler/HalfArithmetic.java](../../github-repo/strangler/src/main/java/com/iluwatar/strangler/HalfArithmetic.java)
- [github-repo/strangler/src/main/java/com/iluwatar/strangler/NewArithmetic.java](../../github-repo/strangler/src/main/java/com/iluwatar/strangler/NewArithmetic.java)
- [github-repo/strangler/src/main/java/com/iluwatar/strangler/OldSource.java](../../github-repo/strangler/src/main/java/com/iluwatar/strangler/OldSource.java)
- [github-repo/strangler/src/main/java/com/iluwatar/strangler/HalfSource.java](../../github-repo/strangler/src/main/java/com/iluwatar/strangler/HalfSource.java)
- [github-repo/strangler/src/main/java/com/iluwatar/strangler/NewSource.java](../../github-repo/strangler/src/main/java/com/iluwatar/strangler/NewSource.java)
- [github-repo/strangler/src/main/java/com/iluwatar/strangler/App.java](../../github-repo/strangler/src/main/java/com/iluwatar/strangler/App.java)
