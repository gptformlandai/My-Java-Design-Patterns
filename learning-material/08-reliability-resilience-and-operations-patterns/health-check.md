# Health Check Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [github-repo/health-check](../../github-repo/health-check)

## How to Study This Page

Use this page in three passes:

1. First pass: understand health checks as machine-readable service status.
2. Second pass: trace liveness, readiness, dependency checks, and load balancer decisions.
3. Third pass: compare Health Check with monitoring, metrics, heartbeat, and circuit breaker.

By the end, you should be able to say:

> Health Check exposes whether a service is alive and ready so platforms can route traffic and recover failures.

## 1. Technical Definition

Health Check is an operations pattern where a service exposes one or more endpoints or signals that report its operational status and dependency readiness.

Core idea:

- Liveness answers "should this process be restarted?"
- Readiness answers "should this instance receive traffic?"
- Dependency checks verify critical resources.
- Platforms use checks for routing and recovery.
- Humans and alerts use checks for diagnosis.

### 30-Second Interview Answer

I would use health checks for every production service. Liveness should be shallow and show whether the process is alive. Readiness should verify whether the instance can safely serve traffic, including critical dependencies. The trade-off is false positives and expensive checks, so checks must be fast, bounded, and separated by purpose.

## 2. Layman and Easy to Understand Definition

Health Check is like a service saying, "I am alive" and "I am ready to work."

Being alive is not the same as being ready. A process can run but still be unable to serve traffic if its database connection is broken.

In code:

- Platform calls health endpoint.
- Service returns UP or DOWN.
- Load balancer routes only to ready instances.
- Orchestrator restarts unhealthy instances.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Without health checks:

- Load balancer may send traffic to broken instances.
- Orchestrator may not restart stuck processes.
- Deployments may route traffic before startup completes.
- Operators lack a quick status signal.

### 3.2 The Health Check Solution

Expose status:

```text
GET /health/live  -> process alive
GET /health/ready -> dependencies ready
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Health endpoint | API returning status. |
| Liveness check | Detects stuck or dead process. |
| Readiness check | Detects whether traffic should be routed. |
| Dependency check | Tests database, queue, cache, or external dependency. |
| Load balancer | Routes away from unhealthy instances. |
| Orchestrator | Restarts or replaces unhealthy instances. |

### 3.4 Check Types

| Type | Question | Common action |
|---|---|---|
| Liveness | Is process alive? | Restart if down. |
| Readiness | Can it serve traffic? | Remove from load balancer. |
| Startup | Has initialization finished? | Delay liveness failures. |
| Deep dependency | Are critical dependencies working? | Alert or mark not ready. |

## 4. Java Coding Example

This example separates liveness from readiness.

```java
class HealthStatus {
    private boolean databaseReady;

    HealthStatus(boolean databaseReady) {
        this.databaseReady = databaseReady;
    }

    String liveness() {
        return "UP";
    }

    String readiness() {
        return databaseReady ? "UP" : "DOWN";
    }

    void setDatabaseReady(boolean databaseReady) {
        this.databaseReady = databaseReady;
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `liveness` | Process is running. |
| `readiness` | Instance can receive traffic. |
| `databaseReady` | Critical dependency check. |
| `setDatabaseReady` | Readiness changes with dependency state. |
| `UP/DOWN` | Machine-readable status. |

### Java Usage

```java
HealthStatus health = new HealthStatus(false);

System.out.println(health.liveness());
System.out.println(health.readiness());

health.setDatabaseReady(true);
System.out.println(health.readiness());
```

## 5. Python Coding Example

```python
class HealthStatus:
    def __init__(self):
        self.database_ready = False

    def live(self):
        return {"status": "UP"}

    def ready(self):
        return {"status": "UP" if self.database_ready else "DOWN"}


health = HealthStatus()
print(health.live())
print(health.ready())
health.database_ready = True
print(health.ready())
```

### Python Usage

Use this shape when explaining:

- Liveness should be shallow.
- Readiness can include critical dependencies.
- Checks must be fast and bounded.

## 6. Where It Comes Handy in Real Life

- Kubernetes probes.
- Load balancer target health.
- Blue-green deployment cutover.
- Auto-healing systems.
- Service dashboards.
- Dependency readiness during startup.
- Incident triage.

## 7. Advantages Over Normal Code Without Pattern

Without Health Check:

```text
platform cannot reliably know if instance should receive traffic
```

With Health Check:

```text
platform routes, restarts, and alerts based on explicit status
```

Benefits:

- Better availability.
- Safer deployments.
- Automatic recovery.
- Faster diagnosis.
- Clear machine-readable status.

## 8. Where It Excels

- Services run behind load balancers.
- Orchestrators manage instances.
- Dependencies can fail independently.
- Startup takes time.
- Automated deployment and recovery are required.

## 9. Where It Fails

- Health check is too expensive.
- Liveness depends on external database and causes restart loops.
- Readiness ignores critical dependencies.
- Checks have no timeout.
- Endpoint leaks sensitive details.

Separate liveness and readiness, and keep both bounded.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Spring Boot Actuator, MicroProfile Health |
| Kubernetes | Liveness, readiness, startup probes |
| Load balancers | AWS ELB health checks, NGINX, Envoy |
| Monitoring | Prometheus, Grafana, Datadog, New Relic |
| Resilience | Dependency checks with timeouts and circuit breakers |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Enables automatic recovery. | Bad checks can cause false restarts. |
| Improves traffic routing. | Deep checks can be expensive. |
| Supports safer deployments. | Requires careful dependency choice. |
| Helps incident diagnosis. | Can expose operational details if unsecured. |

## 12. Real-World Identification Example

Scenario: Order service starts before database migration finishes.

Health Check fit:

- Liveness returns UP because process runs.
- Readiness returns DOWN until database is reachable and schema is ready.
- Load balancer does not route traffic yet.
- Once ready, instance enters rotation.

Without it:

- Traffic reaches instance too early.
- Users see errors during deployment.

## 13. MAANG Interview Triggers

Use Health Check when you hear:

- "How does load balancer know an instance is healthy?"
- "Kubernetes probes."
- "Avoid traffic to broken instances."
- "Auto-restart failed services."
- "Readiness vs liveness."
- "Safe deployment rollout."

Strong answer keywords:

- liveness
- readiness
- startup probe
- dependency check
- timeout
- load balancer
- auto-healing
- false positive

## 14. Common Mistakes

### Mistake 1: Deep liveness checks

- Why it is wrong: dependency outage can cause every instance to restart.
- Better approach: keep liveness shallow and put dependencies in readiness.

### Mistake 2: No timeout

- Why it is wrong: health endpoint can hang and create unclear platform behavior.
- Better approach: set strict timeouts for dependency checks.

### Mistake 3: Always returning UP

- Why it is wrong: platform routes traffic to broken instances.
- Better approach: readiness must reflect critical serving capability.

### Mistake 4: Exposing sensitive details

- Why it is wrong: attackers learn internal dependency names or failures.
- Better approach: expose minimal public status and detailed internal diagnostics securely.

## 15. Health Check vs Similar Patterns

| Pattern | Difference |
|---|---|
| Health Check | Reports service status for routing and recovery. |
| Monitoring | Collects metrics, logs, and alerts over time. |
| Heartbeat | Periodic signal that a process is alive. |
| Circuit Breaker | Protects callers from failing dependencies. |
| Synthetic Check | External probe that tests user-facing behavior. |

## 16. Health Check Design Checklist

- What is the liveness endpoint?
- What is the readiness endpoint?
- Which dependencies are critical for readiness?
- What timeout applies to each check?
- What details are public versus private?
- What does the orchestrator do on failure?
- How does startup delay work?
- What metrics track health transitions?

## 17. Quick Revision Notes

- One-line summary: Health Check tells platforms if an instance is alive and ready.
- Three keywords: liveness, readiness, probes.
- Interview trap: using database checks for liveness and causing restart loops.
- Memory trick: alive is not the same as ready.

## 18. Mini Exercise

Design health checks for a payment service.

Answer these:

1. What does liveness check?
2. What does readiness check?
3. Which dependency failures remove it from traffic?
4. What timeout applies?
5. What status is safe to expose publicly?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/health-check/README.md](../../github-repo/health-check/README.md)
- [github-repo/health-check/src/main/java/com/iluwatar/health/check/App.java](../../github-repo/health-check/src/main/java/com/iluwatar/health/check/App.java)
- [github-repo/health-check/src/main/java/com/iluwatar/health/check/CustomHealthIndicator.java](../../github-repo/health-check/src/main/java/com/iluwatar/health/check/CustomHealthIndicator.java)
- [github-repo/health-check/src/main/java/com/iluwatar/health/check/MemoryHealthIndicator.java](../../github-repo/health-check/src/main/java/com/iluwatar/health/check/MemoryHealthIndicator.java)
- [github-repo/health-check/src/main/java/com/iluwatar/health/check/CpuHealthIndicator.java](../../github-repo/health-check/src/main/java/com/iluwatar/health/check/CpuHealthIndicator.java)
- [github-repo/health-check/src/main/java/com/iluwatar/health/check/DatabaseTransactionHealthIndicator.java](../../github-repo/health-check/src/main/java/com/iluwatar/health/check/DatabaseTransactionHealthIndicator.java)
- [github-repo/health-check/src/main/java/com/iluwatar/health/check/AsynchronousHealthChecker.java](../../github-repo/health-check/src/main/java/com/iluwatar/health/check/AsynchronousHealthChecker.java)
