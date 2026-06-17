# State Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: High  
Software usage meter: High  
Repository module: [github-repo/state](../../github-repo/state)

## How to Study This Page

Use this page in three passes:

1. First pass: understand behavior changing because internal state changes.
2. Second pass: rewrite the Java example and identify context, state interface, concrete states, and transitions.
3. Third pass: compare State with Strategy and finite state machines.

By the end, you should be able to say:

> State moves state-specific behavior into separate state objects so the context changes behavior when its current state changes.

## 1. Technical Definition

State is a behavioral design pattern that lets an object alter its behavior when its internal state changes by delegating state-specific behavior to separate state objects.

Core idea:

- Context owns current state.
- State interface defines state-dependent operations.
- Concrete states implement behavior for one state.
- Transitions switch the context to another state.

### 30-Second Interview Answer

I would use State when an object has modes and many methods behave differently depending on the current mode, such as order lifecycle, document workflow, connection state, or UI mode. Instead of large conditionals, each state becomes a class. It makes transitions and behavior clearer, but can add many classes and must be designed carefully to avoid hidden transition logic.

## 2. Layman and Easy to Understand Definition

State is like a traffic signal.

The same signal behaves differently depending on its current state:

```text
green means go
yellow means prepare to stop
red means stop
```

The signal is the same object, but its behavior changes with state.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Suppose an order behaves differently by status.

```java
if (status == CREATED) {
    pay();
} else if (status == PAID) {
    ship();
} else if (status == SHIPPED) {
    deliver();
}
```

Problems:

- Conditionals spread across methods.
- Adding a state touches many places.
- Invalid transitions are easy to miss.
- State-specific behavior is not localized.

### 3.2 The State Solution

Move behavior into state objects:

```java
order.pay(); // delegates to current state
```

The current state decides what `pay()` means and whether a transition should happen.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Context | Object whose behavior changes. |
| State | Interface for state-specific behavior. |
| Concrete state | Implementation for one state. |
| Transition | Changing the context's current state. |
| Client | Uses context without managing all state rules. |

### 3.4 Mental Model

Think of State as mode objects.

1. Context starts in one state.
2. Client calls a normal method on context.
3. Context delegates to current state.
4. State performs behavior.
5. State or context transitions to another state.

## 4. Java Coding Example

This example models an order lifecycle.

```java
public interface OrderState {
    void pay(Order order);
    void ship(Order order);
    void cancel(Order order);
}

public final class CreatedState implements OrderState {
    @Override
    public void pay(Order order) {
        System.out.println("Payment accepted");
        order.changeState(new PaidState());
    }

    @Override
    public void ship(Order order) {
        throw new IllegalStateException("Cannot ship before payment");
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Order cancelled");
        order.changeState(new CancelledState());
    }
}

public final class PaidState implements OrderState {
    @Override
    public void pay(Order order) {
        throw new IllegalStateException("Order is already paid");
    }

    @Override
    public void ship(Order order) {
        System.out.println("Order shipped");
        order.changeState(new ShippedState());
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Refund issued and order cancelled");
        order.changeState(new CancelledState());
    }
}

public final class ShippedState implements OrderState {
    @Override
    public void pay(Order order) {
        throw new IllegalStateException("Order already shipped");
    }

    @Override
    public void ship(Order order) {
        throw new IllegalStateException("Order already shipped");
    }

    @Override
    public void cancel(Order order) {
        throw new IllegalStateException("Cannot cancel shipped order");
    }
}

public final class CancelledState implements OrderState {
    @Override
    public void pay(Order order) {
        throw new IllegalStateException("Order is cancelled");
    }

    @Override
    public void ship(Order order) {
        throw new IllegalStateException("Order is cancelled");
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Order already cancelled");
    }
}

public final class Order {
    private OrderState state = new CreatedState();

    void changeState(OrderState state) {
        this.state = state;
    }

    public void pay() {
        state.pay(this);
    }

    public void ship() {
        state.ship(this);
    }

    public void cancel() {
        state.cancel(this);
    }
}
```

### Java Block by Block Explanation

#### State Interface

```java
public interface OrderState {
```

The state interface defines operations whose behavior depends on state.

#### Concrete State

```java
public final class CreatedState implements OrderState {
```

This class owns behavior and transitions for the created state.

#### Context

```java
public final class Order {
```

The context delegates behavior to current state.

#### Transition

```java
order.changeState(new PaidState());
```

The state changes the context after a valid operation.

### Java Usage

```java
public class Demo {
    public static void main(String[] args) {
        Order order = new Order();
        order.pay();
        order.ship();
    }
}
```

## 5. Python Coding Example

```python
from __future__ import annotations
from typing import Protocol


class OrderState(Protocol):
    def pay(self, order: "Order") -> None:
        ...

    def ship(self, order: "Order") -> None:
        ...


class CreatedState:
    def pay(self, order: "Order") -> None:
        print("Payment accepted")
        order.change_state(PaidState())

    def ship(self, order: "Order") -> None:
        raise ValueError("Cannot ship before payment")


class PaidState:
    def pay(self, order: "Order") -> None:
        raise ValueError("Order is already paid")

    def ship(self, order: "Order") -> None:
        print("Order shipped")
        order.change_state(ShippedState())


class ShippedState:
    def pay(self, order: "Order") -> None:
        raise ValueError("Order already shipped")

    def ship(self, order: "Order") -> None:
        raise ValueError("Order already shipped")


class Order:
    def __init__(self) -> None:
        self._state: OrderState = CreatedState()

    def change_state(self, state: OrderState) -> None:
        self._state = state

    def pay(self) -> None:
        self._state.pay(self)

    def ship(self) -> None:
        self._state.ship(self)
```

### Python Usage

```python
order = Order()
order.pay()
order.ship()
```

## 6. Where It Comes Handy in Real Life

Examples:

- Order lifecycle.
- Document workflow.
- TCP connection states.
- Media player states.
- UI modes.
- Vending machine states.
- Authentication session state.
- Game character modes.

## 7. Advantages Over Normal Code Without Pattern

### Without State

```java
if (status == PAID) {
    ship();
}
```

Problems:

- State checks spread everywhere.
- Invalid transitions are hard to audit.
- Adding states is risky.
- Behavior is not localized.

### With State

```java
order.ship();
```

Benefits:

- Context stays cleaner.
- State-specific behavior is localized.
- Transitions become explicit.
- Conditionals shrink.

## 8. Where It Excels

State excels when:

- Object has clear modes.
- Behavior changes by state.
- State transitions are important.
- Conditionals are growing.
- Lifecycle rules need clarity.

## 9. Where It Fails

State is a poor fit when:

- There are only one or two simple states.
- State behavior is trivial.
- Transitions are better represented in a state machine table.
- The number of state classes becomes unmanageable.
- State objects need too much context data.

## 10. Prebuilt Libraries and Packages

### Java

Examples:

- Spring StateMachine.
- Akka FSM concepts.
- Workflow engines.
- Enum-based state machines for small cases.

### Python

Examples:

- `transitions` library.
- Enum-based state handling.
- Workflow/state machine libraries.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Localizes state behavior. | More classes. |
| Makes transitions explicit. | Transition logic can be scattered. |
| Removes large conditionals. | State objects may need context access. |
| Fits lifecycle-heavy domains. | Can be overkill for simple states. |
| Easier to test state rules. | State explosion is possible. |

## 12. Real-World Identification Example

Scenario:

You are designing a document approval workflow.

States:

- Draft
- Submitted
- Approved
- Rejected
- Archived

Should you use State?

Yes, if operations behave differently by status and transitions need enforcement.

## 13. MAANG Interview Triggers

Think State when you hear:

- Object changes behavior by state.
- Lifecycle.
- State transition.
- Large conditionals on status.
- Finite state machine.
- Invalid transitions.

### Interview-Ready Answer Format

1. Identify the context.
2. Identify states and valid transitions.
3. Define state interface.
4. Implement concrete states.
5. Delegate context behavior to current state.
6. Mention trade-offs: class count and transition complexity.

## 14. Common Mistakes

### Mistake 1: Confusing State with Strategy

State changes because internal lifecycle changes. Strategy is usually selected as a policy.

### Mistake 2: Scattered Transition Logic

Keep transitions easy to audit.

### Mistake 3: Too Many Tiny States

Do not create a state class when an enum and simple branch is clearer.

### Mistake 4: State Objects Mutating Too Much

State should manage state-dependent behavior, not become a second context.

### Mistake 5: No Invalid Transition Policy

Decide whether invalid transitions throw, ignore, or return errors.

## 15. State vs Similar Patterns

| Pattern | Difference |
|---|---|
| Strategy | Strategy is chosen policy. State changes with lifecycle. |
| Template Method | Template Method fixes algorithm skeleton. State changes behavior by current state. |
| Command | Command encapsulates actions. State decides which actions are valid. |
| Memento | Memento stores previous state snapshots. State models behavior per state. |
| State machine | State pattern is OO state machine style; table-driven state machines are another option. |

## 16. State Design Checklist

| Concern | Why it matters |
|---|---|
| State list | Defines lifecycle. |
| Valid transitions | Prevents illegal moves. |
| Entry/exit actions | Helps side effects stay explicit. |
| Shared data | Belongs in context. |
| Error policy | Defines invalid operation behavior. |

## 17. Quick Revision Notes

- Context delegates to current state.
- Each state owns state-specific behavior.
- Transitions change current state.
- Great for lifecycle rules.
- Do not confuse with Strategy.
- Consider enum/table state machine for simpler cases.

## 18. Mini Exercise

Design State for `Subscription`.

States:

- Trial
- Active
- PastDue
- Cancelled

Operations:

- `pay()`
- `cancel()`
- `expireTrial()`

## 19. Source Reference in This Repo

The repository's State implementation uses `Mammoth` as context and `PeacefulState` / `AngryState` as concrete states.

Useful files:

- [github-repo/state/README.md](../../github-repo/state/README.md)
- [github-repo/state/src/main/java/com/iluwatar/state/State.java](../../github-repo/state/src/main/java/com/iluwatar/state/State.java)
- [github-repo/state/src/main/java/com/iluwatar/state/Mammoth.java](../../github-repo/state/src/main/java/com/iluwatar/state/Mammoth.java)
- [github-repo/state/src/main/java/com/iluwatar/state/PeacefulState.java](../../github-repo/state/src/main/java/com/iluwatar/state/PeacefulState.java)
- [github-repo/state/src/main/java/com/iluwatar/state/AngryState.java](../../github-repo/state/src/main/java/com/iluwatar/state/AngryState.java)
