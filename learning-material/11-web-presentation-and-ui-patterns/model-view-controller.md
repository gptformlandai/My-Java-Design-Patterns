# Model-View-Controller Pattern

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [model-view-controller](../../github-repo/model-view-controller)

---

## How to Study This Page

Study MVC as the core mental model for separating UI systems.

Remember the three roles:
- Model owns data and business rules.
- View renders information.
- Controller receives input and coordinates model/view updates.

---

## 1. Technical Definition

Model-View-Controller is a presentation architecture pattern that separates an application into model, view, and controller components so domain state, rendering, and input handling can evolve independently.

### 30-Second Interview Answer

MVC separates concerns in UI and web applications. The model represents application data and business rules, the view displays data to the user, and the controller handles input, invokes model operations, and selects or updates the view. It is heavily used in web frameworks such as Spring MVC. The benefit is testability and maintainability; the trade-off is more structure and the risk of fat controllers or anemic models.

---

## 2. Layman and Easy to Understand Definition

Imagine a restaurant ordering screen:
- The model is the menu and order data.
- The view is the screen the cashier sees.
- The controller reacts when the cashier clicks "add item" or "checkout."

The screen does not own business rules. The model does not decide how pixels look. The controller connects the two.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Without MVC, UI code often mixes:
- screen rendering
- user input
- validation
- business logic
- database calls

That makes the code hard to test and hard to change.

### MVC Flow

1. User sends input through the UI.
2. Controller receives the input.
3. Controller calls model behavior or updates model state.
4. Model holds the resulting state.
5. Controller selects or updates a view.
6. View renders model data.

### Core Participants

| Participant | Responsibility |
|---|---|
| Model | Business state and rules |
| View | Output and rendering |
| Controller | Input handling and coordination |
| User action | Request or UI event |
| View model/data transfer | Data shaped for rendering |

---

## 4. Java Coding Example

```java
class ProfileModel {
    private String displayName;

    public ProfileModel(String displayName) {
        this.displayName = displayName;
    }

    public String getDisplayName() {
        return displayName;
    }

    public void rename(String newName) {
        if (newName == null || newName.isBlank()) {
            throw new IllegalArgumentException("name is required");
        }
        this.displayName = newName;
    }
}

class ProfileView {
    public void render(ProfileModel model) {
        System.out.println("Profile: " + model.getDisplayName());
    }
}

class ProfileController {
    private final ProfileModel model;
    private final ProfileView view;

    public ProfileController(ProfileModel model, ProfileView view) {
        this.model = model;
        this.view = view;
    }

    public void renameProfile(String newName) {
        model.rename(newName);
        view.render(model);
    }
}

public class MvcDemo {
    public static void main(String[] args) {
        var model = new ProfileModel("Aravind");
        var view = new ProfileView();
        var controller = new ProfileController(model, view);

        controller.renameProfile("A. Aravind");
    }
}
```

### Java Block by Block

`ProfileModel` owns data and validation.

`ProfileView` only renders.

`ProfileController` handles user intent and coordinates model plus view.

The responsibilities are separated, so each part can be tested or changed independently.

---

## 5. Python Coding Example

```python
class ProfileModel:
    def __init__(self, display_name):
        self.display_name = display_name

    def rename(self, new_name):
        if not new_name:
            raise ValueError("name is required")
        self.display_name = new_name


class ProfileView:
    def render(self, model):
        print("Profile:", model.display_name)


class ProfileController:
    def __init__(self, model, view):
        self.model = model
        self.view = view

    def rename_profile(self, new_name):
        self.model.rename(new_name)
        self.view.render(self.model)


controller = ProfileController(ProfileModel("Aravind"), ProfileView())
controller.rename_profile("A. Aravind")
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Web applications | Routes/controllers coordinate models and templates |
| Admin dashboards | UI changes should not rewrite business logic |
| Desktop applications | Screens can change independently from state |
| REST APIs with views/templates | Request handling stays separate from rendering |
| Large teams | UI and domain logic can be developed in parallel |

---

## 7. Advantages Over Normal Code Without Pattern

Without MVC:
- UI and business logic are mixed
- controllers, templates, and data access can become tangled
- testing requires full UI setup
- small UI changes risk business regressions

With MVC:
- concerns are separated
- models can be tested without views
- views can change without rewriting domain logic
- controllers become clear request/input coordinators

---

## 8. Where It Excels

It excels when:
- UI and business logic change independently
- multiple views need the same model
- request handling must be testable
- frameworks already support MVC conventions
- teams need clear presentation boundaries

---

## 9. Where It Fails

It fails when:
- controllers become too large
- model becomes a passive data bag with all logic elsewhere
- view templates contain business rules
- every tiny screen gets over-architected
- data access leaks into controllers/views

Keep controllers thin and put business rules in domain/service layers.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Spring MVC, Jakarta MVC, JSF |
| Python | Django, Flask with blueprints/views |
| Ruby | Rails |
| PHP | Laravel, Symfony |
| JavaScript | Express with templates, older Backbone-style MVC |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Strong separation of concerns | More files/classes |
| Improves testability | Controllers can become bloated |
| Familiar framework model | Boundaries vary by framework |
| Supports parallel development | Small apps may not need full structure |
| Easier UI changes | Model/view synchronization needs care |

---

## 12. Real-World Identification Example

Question:

> You are building a web app where request handling, business validation, and HTML rendering are mixed in one class. It is hard to test and hard to change. What pattern helps?

Strong answer:

Use MVC. Put business state and rules in models/domain services, let controllers handle HTTP input and select responses, and keep views focused on rendering. In a Spring MVC app, a controller method would call a service/model, add data to a view model, and return the template name.

---

## 13. MAANG Interview Triggers

Say MVC when you hear:
- separate UI from business logic
- model view controller
- request maps to controller
- controller selects view
- web framework architecture
- test presentation logic
- avoid mixed UI/domain code

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Fat controller | Hard to test and maintain | Move business logic to model/service |
| Business logic in view | Templates become fragile | Keep views rendering-only |
| Model as only getters/setters | Domain rules get scattered | Put behavior near state |
| Controller talks directly to database everywhere | Coupling grows fast | Use service/repository layer |
| Overusing MVC for tiny scripts | Too much ceremony | Keep simple flows simple |

---

## 15. MVC vs Similar Patterns

| Pattern | Difference |
|---|---|
| MVP | Presenter controls the view more directly through an interface |
| MVVM | View binds to ViewModel properties/commands |
| Front Controller | Central web entry point that can dispatch to MVC controllers/actions |
| Component | Composes reusable UI/behavior parts, not necessarily a full app architecture |
| Layered Architecture | MVC is presentation-focused; layered architecture spans the whole app |

---

## 16. MVC Design Checklist

- What is the model?
- What business rules belong in the model/service layer?
- What user inputs does the controller handle?
- What data does the view need?
- Is the controller thin?
- Does the view contain only rendering logic?
- Can the model be tested without UI?
- Can the view change without changing business rules?

---

## 17. Quick Revision Notes

- One-line summary: Model owns data, View renders, Controller handles input.
- Memory hook: "data, screen, coordinator."
- Best for: web and UI apps with changing presentation.
- Avoid when: the app is too small to need the structure.
- Interview line: "I would keep controllers thin, views rendering-only, and business rules in the model/service layer."

---

## 18. Mini Exercise

Design MVC for a profile edit page:
- define the model fields
- define two controller actions
- define what the view renders
- decide where validation lives
- identify one possible fat-controller smell

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/model-view-controller/README.md)
- [GiantModel.java](../../github-repo/model-view-controller/src/main/java/com/iluwatar/model/view/controller/GiantModel.java)
- [GiantView.java](../../github-repo/model-view-controller/src/main/java/com/iluwatar/model/view/controller/GiantView.java)
- [GiantController.java](../../github-repo/model-view-controller/src/main/java/com/iluwatar/model/view/controller/GiantController.java)
- [App.java](../../github-repo/model-view-controller/src/main/java/com/iluwatar/model/view/controller/App.java)

The repo implementation has a model holding state, a view displaying it, and a controller that mutates model state and asks the view to redraw.

