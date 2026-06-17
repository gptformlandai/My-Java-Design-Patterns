# Flux Pattern

Category: Messaging, Eventing, and Reactive Flow Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [flux](../../github-repo/flux)

---

## How to Study This Page

Study Flux as unidirectional state flow for UI-heavy applications.

The core loop:

```text
View -> Action -> Dispatcher -> Store -> View
```

The same direction every time is the point.

---

## 1. Technical Definition

Flux is an application architecture pattern where user interactions create actions, actions flow through a dispatcher to stores, stores update state, and views re-render from store state.

### 30-Second Interview Answer

Flux enforces one-way data flow. A view emits an action, the dispatcher sends it to stores, stores update state, and views re-render from those stores. I would use it for complex UIs where state changes are hard to reason about. Its benefit is predictable state flow; its cost is boilerplate and extra structure for small apps.

---

## 2. Layman and Easy to Understand Definition

Imagine every UI change going through one lane:

```text
User clicks -> action is created -> central router sends it -> state updates -> screen refreshes
```

No random component secretly mutates another component.

That predictability is Flux.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Complex UIs can become confusing when components update each other directly:

```text
View A changes View B
View B changes Store C
Store C changes View A
```

This creates loops and hidden dependencies.

### Flux Flow

1. User interacts with a view.
2. View creates an action.
3. Dispatcher sends the action to registered stores.
4. Stores decide whether they care about the action.
5. Stores update their state.
6. Stores notify views.
7. Views read store state and render.

### Core Participants

| Participant | Responsibility |
|---|---|
| View | Renders state and emits actions |
| Action | Describes what happened |
| Dispatcher | Routes actions to stores |
| Store | Owns state and business logic |
| Change notification | Tells views to re-render |

---

## 4. Java Coding Example

```java
import java.util.ArrayList;
import java.util.List;

record Action(String type, String payload) {}

interface Store {
    void onAction(Action action);
}

class Dispatcher {
    private final List<Store> stores = new ArrayList<>();

    public void register(Store store) {
        stores.add(store);
    }

    public void dispatch(Action action) {
        for (Store store : stores) {
            store.onAction(action);
        }
    }
}

class CounterStore implements Store {
    private int count;

    public void onAction(Action action) {
        if ("increment".equals(action.type())) {
            count++;
            System.out.println("render count = " + count);
        }
    }
}

public class FluxDemo {
    public static void main(String[] args) {
        Dispatcher dispatcher = new Dispatcher();
        dispatcher.register(new CounterStore());

        dispatcher.dispatch(new Action("increment", ""));
        dispatcher.dispatch(new Action("increment", ""));
    }
}
```

### Java Block by Block

`Action` represents a UI event or user intent.

`Dispatcher` sends every action to stores.

`CounterStore` owns the state and decides whether to handle the action.

The view would render from the store after state changes.

---

## 5. Python Coding Example

```python
class Dispatcher:
    def __init__(self):
        self.stores = []

    def register(self, store):
        self.stores.append(store)

    def dispatch(self, action):
        for store in self.stores:
            store.on_action(action)


class CounterStore:
    def __init__(self):
        self.count = 0

    def on_action(self, action):
        if action["type"] == "increment":
            self.count += 1
            print("render count =", self.count)


dispatcher = Dispatcher()
dispatcher.register(CounterStore())
dispatcher.dispatch({"type": "increment"})
dispatcher.dispatch({"type": "increment"})
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Complex UI state | Predictable state updates |
| Dashboard filters | Actions update multiple stores |
| Admin tools | Views render from central state |
| Multi-panel apps | Avoids component-to-component mutation |
| Client-side apps | Similar to Redux-style flow |
| Form workflows | State transitions are explicit |

---

## 7. Advantages Over Normal Code Without Pattern

Without Flux:
- views may mutate each other directly
- state flow becomes circular
- debugging UI changes is hard
- state ownership is unclear

With Flux:
- data flows in one direction
- actions are explicit
- stores own state
- rendering follows state changes

---

## 8. Where It Excels

It excels when:
- UI state is complex
- many views depend on shared state
- debugging state transitions matters
- predictable rendering is more valuable than minimal code
- actions are meaningful domain events in the UI

---

## 9. Where It Fails

It fails when:
- the app is small and simple
- there is little shared state
- boilerplate overwhelms the value
- stores become too large
- action names are vague
- async effects are not separated clearly

Use simpler local state when state does not need global coordination.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| JavaScript | Redux, Flux, Zustand, NgRx |
| Java | Custom dispatcher/store patterns, Vaadin state patterns |
| Android | MVI, ReduxKotlin, Orbit MVI |
| Kotlin | StateFlow plus reducer-style stores |
| UI architecture | Elm architecture, MVU, MVI |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Predictable one-way data flow | Boilerplate |
| Easier debugging | More concepts to learn |
| Clear state ownership | Can feel heavy for small apps |
| Good for large UIs | Store design requires discipline |
| Actions document user intent | Async effects need careful modeling |

---

## 12. Real-World Identification Example

Question:

> A web admin console has many panels sharing filters, selected account, and content state. Components are directly updating each other and bugs are hard to trace. What pattern helps?

Strong answer:

Use Flux-style unidirectional data flow. Views dispatch actions such as `AccountSelected` or `FilterChanged`. The dispatcher routes actions to stores. Stores update state and notify views to re-render. This makes state transitions traceable and prevents direct component-to-component mutation.

---

## 13. MAANG Interview Triggers

Say Flux when you hear:
- complex UI state
- unidirectional data flow
- action dispatcher store view
- state is hard to trace
- Redux-like design
- views should not mutate each other
- predictable rendering

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Store contains all app logic | Store becomes huge | Split stores by state boundary |
| Vague action names | Hard to debug intent | Use domain-specific action names |
| Mutating state from views | Breaks one-way flow | Dispatch actions only |
| Too much boilerplate for small UI | Slows development | Use local state when enough |
| Async work hidden in views | State transitions become unclear | Model async effects explicitly |

---

## 15. Flux vs Similar Patterns

| Pattern | Difference |
|---|---|
| MVC | MVC can have bidirectional updates; Flux enforces one-way flow |
| Redux | Redux is a popular Flux-inspired implementation |
| Observer | Stores notify views, but Flux adds actions and dispatcher |
| Mediator | Dispatcher mediates action delivery, but Flux defines full UI state flow |
| Event-Driven Architecture | Flux is usually UI/application state flow, not whole distributed architecture |

---

## 16. Flux Design Checklist

- What actions can views dispatch?
- What store owns each piece of state?
- Are stores updated only through actions?
- How do views subscribe to store changes?
- Where do async effects live?
- Are action names meaningful?
- Is the state small enough to inspect?
- Can state transitions be logged or replayed?

---

## 17. Quick Revision Notes

- One-line summary: View creates action, dispatcher sends it, store updates, view renders.
- Memory hook: "one-way UI traffic."
- Best for: complex shared UI state.
- Avoid when: local component state is enough.
- Interview line: "I would model user intent as actions, route them through a dispatcher, keep state in stores, and render views from store state."

---

## 18. Mini Exercise

Design Flux for a shopping cart UI:
- define actions for add, remove, and change quantity
- define cart store state
- define which views subscribe
- add one async action for price refresh
- explain why views should not mutate the store directly

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/flux/README.md)
- [Dispatcher.java](../../github-repo/flux/src/main/java/com/iluwatar/flux/dispatcher/Dispatcher.java)
- [Store.java](../../github-repo/flux/src/main/java/com/iluwatar/flux/store/Store.java)
- [MenuStore.java](../../github-repo/flux/src/main/java/com/iluwatar/flux/store/MenuStore.java)
- [ContentStore.java](../../github-repo/flux/src/main/java/com/iluwatar/flux/store/ContentStore.java)
- [MenuView.java](../../github-repo/flux/src/main/java/com/iluwatar/flux/view/MenuView.java)
- [MenuAction.java](../../github-repo/flux/src/main/java/com/iluwatar/flux/action/MenuAction.java)
- [App.java](../../github-repo/flux/src/main/java/com/iluwatar/flux/app/App.java)

The repo implementation wires views to stores, registers stores with a singleton dispatcher, and routes menu actions through stores so views re-render from updated store state.

