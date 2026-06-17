# Component Pattern

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [component](../../github-repo/component)

---

## How to Study This Page

Study Component as composition over inheritance.

The key idea:
- entity/object owns small behavior components
- each component handles one capability
- new combinations are built by swapping components

---

## 1. Technical Definition

Component is a structural pattern where behavior or data is split into independent reusable components that can be attached to objects to compose capabilities without deep inheritance hierarchies.

### 30-Second Interview Answer

The Component pattern builds objects by composing small capability objects instead of creating large inheritance trees. For example, an object can have input, rendering, and physics components. I would use it when entities need flexible combinations of behavior, such as UI widgets, dashboards, plugins, or simulation objects. The trade-off is indirection and coordination between components.

---

## 2. Layman and Easy to Understand Definition

Think of building a dashboard tile from plug-in parts:
- one part fetches data
- one part formats it
- one part renders it
- one part handles clicks

Different tiles can reuse the same parts in different combinations.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Inheritance can explode when objects need many combinations of features:

```text
ClickableChart
ClickableAnimatedChart
ClickableAnimatedChartWithTooltip
ReadOnlyAnimatedChartWithTooltip
```

This becomes hard to maintain.

### Component Flow

1. Define small component interfaces.
2. Implement each capability separately.
3. Compose an object from selected components.
4. The object delegates behavior to its components.
5. Components can be reused across many object types.
6. Components can sometimes be swapped at runtime.

### Core Participants

| Participant | Responsibility |
|---|---|
| Entity/object | Holds composed components |
| Component interface | Defines a capability |
| Concrete component | Implements one behavior |
| Update/render/input loop | Invokes components |
| Factory/builder | Assembles useful component combinations |

---

## 4. Java Coding Example

```java
interface RenderComponent {
    void render(String componentId);
}

interface ClickComponent {
    void click(String componentId);
}

class ButtonRenderer implements RenderComponent {
    public void render(String componentId) {
        System.out.println("render button " + componentId);
    }
}

class SaveClick implements ClickComponent {
    public void click(String componentId) {
        System.out.println("save from " + componentId);
    }
}

class UiComponent {
    private final String id;
    private final RenderComponent renderer;
    private final ClickComponent clicker;

    UiComponent(String id, RenderComponent renderer, ClickComponent clicker) {
        this.id = id;
        this.renderer = renderer;
        this.clicker = clicker;
    }

    void render() {
        renderer.render(id);
    }

    void click() {
        clicker.click(id);
    }
}

public class ComponentDemo {
    public static void main(String[] args) {
        UiComponent saveButton = new UiComponent("save", new ButtonRenderer(), new SaveClick());
        saveButton.render();
        saveButton.click();
    }
}
```

### Java Block by Block

`RenderComponent` and `ClickComponent` define separate capabilities.

`UiComponent` does not inherit all behavior. It delegates to components.

The same renderer or click behavior can be reused by other objects.

---

## 5. Python Coding Example

```python
class ButtonRenderer:
    def render(self, component_id):
        print("render button", component_id)


class SaveClick:
    def click(self, component_id):
        print("save from", component_id)


class UiComponent:
    def __init__(self, component_id, renderer, clicker):
        self.component_id = component_id
        self.renderer = renderer
        self.clicker = clicker

    def render(self):
        self.renderer.render(self.component_id)

    def click(self):
        self.clicker.click(self.component_id)


save_button = UiComponent("save", ButtonRenderer(), SaveClick())
save_button.render()
save_button.click()
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| UI widgets | Compose rendering, validation, input, behavior |
| Dashboards | Reuse data/render/action components |
| Games/simulations | Entities combine movement, rendering, input |
| Plugin systems | Add capabilities without inheritance |
| Design systems | Components are reusable building blocks |
| Workflow builders | Steps can be composed from capabilities |

---

## 7. Advantages Over Normal Code Without Pattern

Without Component:
- inheritance trees grow large
- duplicate behavior spreads across classes
- adding combinations requires new subclasses
- behavior changes ripple through parent classes

With Component:
- behavior is reusable
- combinations are flexible
- objects stay smaller
- features can be swapped or tested independently

---

## 8. Where It Excels

It excels when:
- objects need many feature combinations
- behavior should be reusable
- inheritance is becoming deep
- capabilities change independently
- runtime composition is valuable

---

## 9. Where It Fails

It fails when:
- objects have only one simple behavior
- components need excessive cross-communication
- performance-sensitive paths suffer from indirection
- ownership boundaries are unclear
- component lifecycle is unmanaged

Use simple classes when composition would add ceremony without flexibility.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| UI | React components, Vue components, Angular components |
| Java desktop | JavaFX controls, Swing components |
| Game/simulation | Entity Component System style frameworks |
| Backend | Spring beans as composable services |
| Design systems | Storybook, component libraries |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Avoids deep inheritance | More indirection |
| Reusable behaviors | Component coordination can be tricky |
| Flexible composition | Lifecycle management required |
| Better testability | Debugging jumps across objects |
| Supports runtime variation | Too many tiny components can fragment design |

---

## 12. Real-World Identification Example

Question:

> A UI framework has cards, charts, buttons, and tables. Each can independently support rendering, validation, click behavior, permissions, and loading state. Inheritance is exploding. What pattern helps?

Strong answer:

Use Component. Split capabilities into reusable components such as renderer, validator, permission checker, and action handler. Each UI element composes the capabilities it needs instead of inheriting every combination. I would define clear component contracts and avoid components reaching too deeply into each other.

---

## 13. MAANG Interview Triggers

Say Component when you hear:
- compose behavior dynamically
- avoid inheritance explosion
- reusable UI pieces
- entity component system
- plug-in capabilities
- independent UI modules
- design system components

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Components know too much about each other | Coupling returns through the side door | Use narrow interfaces/events |
| Too many tiny components | System becomes fragmented | Split only real variation points |
| No lifecycle ownership | Leaks or stale state | Define create/update/destroy rules |
| Using components for fixed behavior | Adds ceremony | Use simple classes |
| Sharing mutable component state carelessly | Cross-object bugs | Keep component state scoped or immutable |

---

## 15. Component vs Similar Patterns

| Pattern | Difference |
|---|---|
| Composite | Composite models tree structures; Component composes capabilities |
| Decorator | Decorator wraps an object; Component delegates to capability parts |
| Strategy | Strategy swaps one algorithm; Component can combine many capabilities |
| MVC | MVC separates presentation roles; Component structures reusable parts |
| Flyweight | Flyweight shares intrinsic state; Component focuses on capability composition |

---

## 16. Component Design Checklist

- What capability varies independently?
- What is the component interface?
- Who owns component lifecycle?
- Can the component be reused safely?
- Does it need access to the parent object?
- Is communication between components controlled?
- Is composition simpler than inheritance here?
- Are performance costs acceptable?

---

## 17. Quick Revision Notes

- One-line summary: Build objects from reusable capability parts.
- Memory hook: "capabilities as plug-in pieces."
- Best for: flexible UI/entity behavior combinations.
- Avoid when: a simple class is enough.
- Interview line: "I would split variable capabilities into components and compose objects instead of building a deep inheritance tree."

---

## 18. Mini Exercise

Design a dashboard card with components:
- renderer
- data loader
- permission checker
- click action
- loading indicator

Create two cards that reuse at least two components but differ in one behavior.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/component/README.md)
- [GameObject.java](../../github-repo/component/src/main/java/com/iluwatar/component/GameObject.java)
- [InputComponent.java](../../github-repo/component/src/main/java/com/iluwatar/component/component/inputcomponent/InputComponent.java)
- [PlayerInputComponent.java](../../github-repo/component/src/main/java/com/iluwatar/component/component/inputcomponent/PlayerInputComponent.java)
- [PhysicComponent.java](../../github-repo/component/src/main/java/com/iluwatar/component/component/physiccomponent/PhysicComponent.java)
- [ObjectPhysicComponent.java](../../github-repo/component/src/main/java/com/iluwatar/component/component/physiccomponent/ObjectPhysicComponent.java)
- [GraphicComponent.java](../../github-repo/component/src/main/java/com/iluwatar/component/component/graphiccomponent/GraphicComponent.java)
- [ObjectGraphicComponent.java](../../github-repo/component/src/main/java/com/iluwatar/component/component/graphiccomponent/ObjectGraphicComponent.java)
- [App.java](../../github-repo/component/src/main/java/com/iluwatar/component/App.java)

The repo implementation composes `GameObject` from input, physics, and graphics components, then delegates update behavior to those components.

