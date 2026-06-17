# Saga Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [github-repo/saga](../../github-repo/saga)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why distributed transactions are hard when each service owns its database.
2. Second pass: follow the happy path and then the compensation path.
3. Third pass: compare orchestration, choreography, two-phase commit, and eventual consistency.

By the end, you should be able to say:

> Saga coordinates a long business transaction as smaller local transactions plus compensating actions.

## 1. Technical Definition

Saga is a distributed transaction pattern that breaks one cross-service business transaction into a sequence of local transactions, each with a compensating action used when later steps fail.

Core idea:

- Each service updates its own database locally.
- Services communicate through commands or events.
- Later failure triggers compensating actions.
- Consistency is eventual, not immediate.
- The saga can be orchestrated centrally or choreographed through events.

### 30-Second Interview Answer

I would use Saga when a business workflow spans multiple microservices that own separate databases, such as order, payment, inventory, and shipping. Instead of using distributed locks or two-phase commit, each service performs a local transaction and publishes the next step. If a later step fails, the saga executes compensating actions. The trade-off is eventual consistency and more complex failure handling.

## 2. Layman and Easy to Understand Definition

Saga is like booking a trip in steps.

You book a flight, then a hotel, then a car. If the hotel fails, you cancel the flight. You do not lock every company system until all bookings finish.

In code:

- Step 1 succeeds.
- Step 2 succeeds.
- Step 3 fails.
- Compensation reverses earlier successful steps.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

In microservices, each service usually owns its own database:

```text
order service -> order database
payment service -> payment database
inventory service -> inventory database
shipping service -> shipping database
```

One user action can require all of them.

Traditional database transactions do not easily span all services without heavy coordination.

### 3.2 The Saga Solution

Model the workflow as a sequence:

```text
create order
reserve inventory
charge payment
create shipment
```

Each step commits locally.

If `charge payment` fails, the saga runs:

```text
release inventory
cancel order
```

### 3.3 Saga Styles

| Style | Meaning |
|---|---|
| Orchestration | A central orchestrator tells each service what to do next. |
| Choreography | Services react to events and publish the next event. |

### 3.4 Main Participants

| Participant | Meaning |
|---|---|
| Saga | Whole business workflow. |
| Local transaction | One service updates its own state. |
| Compensation | Business undo action. |
| Orchestrator | Central controller in orchestration style. |
| Event bus | Communication backbone in choreography style. |
| Saga state | Current step, result, retries, and failure reason. |

### 3.5 Failure Path

1. Saga starts.
2. One or more local transactions succeed.
3. A later transaction fails.
4. Saga marks the failed step.
5. Compensation runs in reverse order.
6. Saga ends as compensated or failed-for-manual-review.

## 4. Java Coding Example

This example shows a simple orchestrated saga.

```java
import java.util.ArrayList;
import java.util.List;

interface SagaStep {
    void execute();
    void compensate();
}

class OrderSaga {
    private final List<SagaStep> steps;

    OrderSaga(List<SagaStep> steps) {
        this.steps = steps;
    }

    void run() {
        List<SagaStep> completed = new ArrayList<>();

        try {
            for (SagaStep step : steps) {
                step.execute();
                completed.add(step);
            }
        } catch (RuntimeException failure) {
            for (int i = completed.size() - 1; i >= 0; i--) {
                completed.get(i).compensate();
            }
            throw failure;
        }
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `SagaStep` | Every step has forward and compensating behavior. |
| `completed` | Only completed steps need compensation. |
| `try` block | Happy path executes local transactions. |
| reverse loop | Rollback is business compensation in reverse order. |
| `throw failure` | Caller still learns the saga failed. |

### Java Usage

```java
SagaStep inventory = new SagaStep() {
    public void execute() {
        System.out.println("reserve inventory");
    }

    public void compensate() {
        System.out.println("release inventory");
    }
};

SagaStep payment = new SagaStep() {
    public void execute() {
        throw new RuntimeException("payment declined");
    }

    public void compensate() {
        System.out.println("refund payment");
    }
};

new OrderSaga(List.of(inventory, payment)).run();
```

## 5. Python Coding Example

```python
class Saga:
    def __init__(self):
        self.steps = []

    def step(self, execute, compensate):
        self.steps.append((execute, compensate))
        return self

    def run(self):
        completed = []
        try:
            for execute, compensate in self.steps:
                execute()
                completed.append(compensate)
        except Exception:
            for compensate in reversed(completed):
                compensate()
            raise


saga = Saga()
saga.step(
    execute=lambda: print("reserve inventory"),
    compensate=lambda: print("release inventory"),
)
saga.step(
    execute=lambda: (_ for _ in ()).throw(RuntimeError("payment failed")),
    compensate=lambda: print("refund payment"),
)

saga.run()
```

### Python Usage

Use this shape when explaining:

- A saga is not one database transaction.
- Each step commits locally.
- Compensation is domain-specific, not automatic rollback.

## 6. Where It Comes Handy in Real Life

- Checkout across order, inventory, payment, and shipping.
- Travel booking across flight, hotel, and vehicle services.
- Money transfer with ledger and notification services.
- Subscription signup with billing and provisioning.
- Fulfillment workflows.

## 7. Advantages Over Normal Code Without Pattern

Without Saga:

```text
service A updates data, service B fails, service C has no clear recovery
```

With Saga:

```text
every step has a planned compensation or retry path
```

Benefits:

- Avoids distributed database transactions.
- Keeps service databases independent.
- Handles partial failure explicitly.
- Supports long-running workflows.
- Makes recovery observable.

## 8. Where It Excels

- Business process spans multiple services.
- Strong immediate consistency is not required.
- Compensation is possible.
- Workflow steps can be retried safely.
- Each service owns its data.

## 9. Where It Fails

- Compensation cannot actually undo the business effect.
- Workflow requires immediate global consistency.
- Steps are not idempotent.
- Events can be lost or processed out of order.
- Saga state is not persisted.

For high-value irreversible operations, add manual review or hold states.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Axon Framework, Eventuate Tram Sagas, Camunda, Temporal Java SDK |
| Workflow | Temporal, Cadence, Netflix Conductor, Zeebe |
| Messaging | Kafka, RabbitMQ, Pulsar |
| Spring | Spring State Machine, Spring Cloud Stream |
| Observability | OpenTelemetry, distributed tracing, structured audit logs |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids two-phase commit. | Eventual consistency is harder to reason about. |
| Fits service-owned databases. | Compensation logic is domain-specific. |
| Makes long workflows explicit. | Requires careful retry and idempotency. |
| Improves partial failure recovery. | Debugging spans multiple services. |

## 12. Real-World Identification Example

Scenario: Checkout service must create order, reserve inventory, charge card, and schedule shipping.

Saga fit:

- Order service creates pending order.
- Inventory reserves items.
- Payment charges user.
- Shipping creates shipment.
- If payment fails, inventory is released and order is canceled.

Without it:

- Partial updates remain.
- Customers see inconsistent order states.
- Teams invent ad hoc cleanup jobs.

## 13. MAANG Interview Triggers

Use Saga when you hear:

- "Distributed transaction across services."
- "Each microservice has its own database."
- "How do we handle payment failure after inventory reservation?"
- "We cannot use two-phase commit."
- "Need eventual consistency."
- "Long-running business workflow."

Strong answer keywords:

- local transaction
- compensation
- orchestration
- choreography
- idempotency
- retry
- eventual consistency
- saga log
- outbox

## 14. Common Mistakes

### Mistake 1: Calling compensation rollback

- Why it is wrong: compensation is not a database rollback; it is a business action.
- Better approach: design explicit undo or counter-action behavior.

### Mistake 2: Forgetting idempotency

- Why it is wrong: retries can duplicate charges, reservations, or shipments.
- Better approach: use idempotency keys for every step.

### Mistake 3: Not persisting saga state

- Why it is wrong: orchestrator restart can lose progress.
- Better approach: store saga state and resume after crash.

### Mistake 4: Using choreography for very complex flows

- Why it is wrong: event chains become hard to understand.
- Better approach: use orchestration when global workflow visibility matters.

## 15. Saga vs Similar Patterns

| Pattern | Difference |
|---|---|
| Saga | Coordinates local transactions with compensation. |
| Two-Phase Commit | Locks participants until all agree to commit. |
| Outbox | Reliably publishes events from one local transaction. |
| Process Manager | Tracks and controls a long-running process; often implements saga orchestration. |
| CQRS | Separates reads and writes; may be used with saga. |

## 16. Saga Design Checklist

- What are the local transaction steps?
- What is the compensation for each successful step?
- Which steps are retryable?
- Which operations need idempotency keys?
- Where is saga state persisted?
- Is orchestration or choreography clearer?
- What is the final state after compensation fails?
- What is shown to the user during eventual consistency?

## 17. Quick Revision Notes

- One-line summary: Saga manages cross-service workflows through local commits and compensation.
- Three keywords: compensation, orchestration, eventual consistency.
- Interview trap: assuming compensation is the same as ACID rollback.
- Memory trick: do steps forward, undo completed steps backward.

## 18. Mini Exercise

Design a saga for food delivery checkout.

Answer these:

1. What are the steps?
2. Which service owns each step?
3. What compensation exists for each step?
4. Which steps need idempotency keys?
5. What user-visible state appears while the saga is running?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/saga/README.md](../../github-repo/saga/README.md)
- [github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/Saga.java](../../github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/Saga.java)
- [github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/SagaOrchestrator.java](../../github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/SagaOrchestrator.java)
- [github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/OrchestrationChapter.java](../../github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/OrchestrationChapter.java)
- [github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/Service.java](../../github-repo/saga/src/main/java/com/iluwatar/saga/orchestration/Service.java)
- [github-repo/saga/src/main/java/com/iluwatar/saga/choreography/Saga.java](../../github-repo/saga/src/main/java/com/iluwatar/saga/choreography/Saga.java)
- [github-repo/saga/src/main/java/com/iluwatar/saga/choreography/ServiceDiscoveryService.java](../../github-repo/saga/src/main/java/com/iluwatar/saga/choreography/ServiceDiscoveryService.java)
- [github-repo/saga/src/main/java/com/iluwatar/saga/choreography/SagaApplication.java](../../github-repo/saga/src/main/java/com/iluwatar/saga/choreography/SagaApplication.java)
