# Feature Toggle Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/feature-toggle](../../github-repo/feature-toggle)

## How to Study This Page

Use this page in three passes:

1. First pass: understand feature toggle as runtime control over code paths.
2. Second pass: trace config flag, user targeting, enabled branch, disabled branch, and rollback.
3. Third pass: compare Feature Toggle with canary release, A/B testing, branch by abstraction, and configuration.

By the end, you should be able to say:

> Feature Toggle lets teams enable or disable behavior at runtime without redeploying code.

## 1. Technical Definition

Feature Toggle is a release and operations pattern where conditional runtime configuration controls whether a feature or code path is active.

Core idea:

- Code is deployed with both old and new behavior.
- A flag decides which path runs.
- Flags can target users, tenants, regions, or percentages.
- Bad features can be disabled quickly.
- Old flags must be removed after rollout.

### 30-Second Interview Answer

I would use Feature Toggles to decouple deployment from release. New code can be deployed disabled, enabled for internal users or a small percentage, monitored, and rolled back instantly by turning the flag off. The trade-off is complexity: flags multiply test combinations and become technical debt if not retired.

## 2. Layman and Easy to Understand Definition

Feature Toggle is like a switch on a control panel.

The wiring is already installed, but the switch decides whether the feature is active.

In code:

- Check flag.
- If enabled, run new behavior.
- If disabled, run old behavior.
- Change flag without redeploy.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Deploying and releasing at the same moment is risky:

- A new feature may fail in production.
- Only some users should see it.
- Rollback may require redeploy.
- Teams need gradual rollout and experiments.

### 3.2 The Feature Toggle Solution

Guard behavior behind a flag:

```text
if feature enabled:
    run new path
else:
    run old path
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Toggle key | Unique feature flag name. |
| Toggle provider | Config file, database, or feature flag service. |
| Targeting rule | Which users or tenants receive the feature. |
| Toggle point | Conditional branch in code. |
| Owner | Team responsible for flag lifecycle. |
| Expiry | Planned removal date. |

### 3.4 Toggle Types

| Type | Purpose |
|---|---|
| Release toggle | Gradual rollout. |
| Ops toggle | Disable risky behavior during incidents. |
| Experiment toggle | A/B testing. |
| Permission toggle | Enable feature for paid or specific users. |
| Kill switch | Emergency disable path. |

## 4. Java Coding Example

This example toggles a new welcome message.

```java
import java.util.Map;

record User(String id, boolean premium) {
}

class FeatureFlags {
    private final Map<String, Boolean> flags;

    FeatureFlags(Map<String, Boolean> flags) {
        this.flags = flags;
    }

    boolean enabled(String key) {
        return flags.getOrDefault(key, false);
    }
}

class WelcomeService {
    private final FeatureFlags flags;

    WelcomeService(FeatureFlags flags) {
        this.flags = flags;
    }

    String message(User user) {
        if (flags.enabled("enhanced-welcome") && user.premium()) {
            return "Welcome back, premium user " + user.id();
        }
        return "Welcome";
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `FeatureFlags` | Runtime source of flag decisions. |
| `enabled` | Safe default is disabled. |
| `premium` | Targeting condition. |
| `WelcomeService` | Toggle point in application behavior. |
| two branches | Old and new behavior coexist temporarily. |

### Java Usage

```java
FeatureFlags flags = new FeatureFlags(Map.of("enhanced-welcome", true));
WelcomeService service = new WelcomeService(flags);

System.out.println(service.message(new User("u-1", true)));
```

## 5. Python Coding Example

```python
class FeatureFlags:
    def __init__(self, flags):
        self.flags = flags

    def enabled(self, key):
        return self.flags.get(key, False)


def welcome(user, flags):
    if flags.enabled("enhanced-welcome") and user["premium"]:
        return f"Welcome back, premium user {user['id']}"
    return "Welcome"


flags = FeatureFlags({"enhanced-welcome": True})
print(welcome({"id": "u-1", "premium": True}, flags))
```

### Python Usage

Use this shape when explaining:

- Flags should default safely.
- Targeting is part of the decision.
- Old flags need cleanup after rollout.

## 6. Where It Comes Handy in Real Life

- Gradual feature rollout.
- Canary releases.
- A/B tests.
- Kill switches.
- Premium feature gating.
- Regional rollout.
- Operational risk reduction.

## 7. Advantages Over Normal Code Without Pattern

Without Feature Toggle:

```text
deploy equals release; rollback requires redeploy
```

With Feature Toggle:

```text
deploy code now, release behavior later
```

Benefits:

- Faster rollback.
- Safer deployments.
- Supports experiments.
- Enables targeted rollout.
- Reduces long-lived branches.

## 8. Where It Excels

- New feature has production risk.
- Rollout should be gradual.
- Different user groups need different behavior.
- Operations need emergency disable.
- Teams practice continuous delivery.

## 9. Where It Fails

- Toggles never get removed.
- Too many nested flags make logic unreadable.
- Tests ignore both enabled and disabled paths.
- Flag defaults are unsafe.
- Flag config changes lack audit trail.

Treat toggles as short-lived production controls with owners and expiry dates.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Services | LaunchDarkly, Unleash, Split, ConfigCat |
| Standards | OpenFeature |
| Java | FF4J, Togglz, Spring configuration |
| Delivery | Canary, blue-green, progressive rollout |
| Observability | Flag change audit logs, metrics by variation |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Decouples deploy from release. | Adds conditional complexity. |
| Enables fast rollback. | Creates test matrix growth. |
| Supports targeted rollout. | Old flags become technical debt. |
| Helps experiments. | Misconfiguration can impact users. |

## 12. Real-World Identification Example

Scenario: Search team deploys a new ranking algorithm.

Feature Toggle fit:

- New ranking code is deployed disabled.
- Internal users get the flag first.
- Then 5 percent of users see it.
- Metrics compare click-through and latency.
- Flag is disabled instantly if errors rise.

Without it:

- Rollout requires full deployment.
- Rollback is slower.
- Experiment control is harder.

## 13. MAANG Interview Triggers

Use Feature Toggle when you hear:

- "Deploy without releasing."
- "Gradual rollout."
- "Kill switch."
- "A/B test."
- "Canary users."
- "How do we rollback without redeploy?"

Strong answer keywords:

- flag
- targeting
- rollout
- kill switch
- audit
- cleanup
- default off
- experiment

## 14. Common Mistakes

### Mistake 1: Permanent temporary flags

- Why it is wrong: code becomes cluttered and hard to test.
- Better approach: assign owner and expiry date to every flag.

### Mistake 2: Unsafe default

- Why it is wrong: config outage may accidentally enable risky behavior.
- Better approach: default risky features to disabled.

### Mistake 3: Not testing both paths

- Why it is wrong: disabled branch can break silently.
- Better approach: test enabled and disabled behavior while flag exists.

### Mistake 4: No audit trail

- Why it is wrong: production behavior changes without traceability.
- Better approach: log who changed flags, when, and why.

## 15. Feature Toggle vs Similar Patterns

| Pattern | Difference |
|---|---|
| Feature Toggle | Runtime switch for behavior. |
| Canary Release | Sends traffic to new deployment version. |
| A/B Testing | Measures behavior variants with users. |
| Branch by Abstraction | Code technique for swapping implementation safely. |
| Configuration | General runtime settings; feature flags are controlled behavior gates. |

## 16. Feature Toggle Design Checklist

- What is the flag name?
- What is the safe default?
- Who owns the flag?
- Who can change it?
- What users or tenants are targeted?
- What metrics decide rollout?
- What is the rollback plan?
- When will the flag be deleted?
- Are both branches tested?

## 17. Quick Revision Notes

- One-line summary: Feature Toggle controls production behavior without redeploy.
- Three keywords: flag, rollout, cleanup.
- Interview trap: forgetting toggle debt and test matrix growth.
- Memory trick: ship the wiring, flip the switch later.

## 18. Mini Exercise

Design a feature toggle for a new checkout flow.

Answer these:

1. What is the flag key?
2. What is the default value?
3. Which users see it first?
4. What metrics decide expansion?
5. When is the flag removed?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/feature-toggle/README.md](../../github-repo/feature-toggle/README.md)
- [github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/pattern/Service.java](../../github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/pattern/Service.java)
- [github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/pattern/propertiesversion/PropertiesFeatureToggleVersion.java](../../github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/pattern/propertiesversion/PropertiesFeatureToggleVersion.java)
- [github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/pattern/tieredversion/TieredFeatureToggleVersion.java](../../github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/pattern/tieredversion/TieredFeatureToggleVersion.java)
- [github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/user/User.java](../../github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/user/User.java)
- [github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/user/UserGroup.java](../../github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/user/UserGroup.java)
- [github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/App.java](../../github-repo/feature-toggle/src/main/java/com/iluwatar/featuretoggle/App.java)
