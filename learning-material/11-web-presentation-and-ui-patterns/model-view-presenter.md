# Model-View-Presenter Pattern

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [model-view-presenter](../../github-repo/model-view-presenter)

---

## How to Study This Page

Study MVP as a test-friendly UI separation pattern.

The key distinction:
- View is passive and exposes an interface.
- Presenter owns presentation logic.
- Model owns data/business behavior.

---

## 1. Technical Definition

Model-View-Presenter is a presentation architecture pattern where the presenter mediates between a passive view and the model, handling user actions and updating the view through an interface.

### 30-Second Interview Answer

MVP separates UI logic from UI rendering. The view exposes an interface and forwards user actions to the presenter. The presenter reads or updates the model and tells the view what to display. This makes presentation logic easy to unit test because the view can be mocked. MVP is useful in desktop, mobile, and UI-heavy apps where views should stay passive.

---

## 2. Layman and Easy to Understand Definition

Think of a presenter during a live demo:
- the screen only displays things
- the presenter decides what should happen next
- the data source provides the facts

The screen does not contain the decision logic.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

UI code becomes hard to test when event handling, validation, data loading, and rendering are all inside the view.

### MVP Flow

1. View receives user action.
2. View calls presenter.
3. Presenter asks the view for required input.
4. Presenter invokes model/service behavior.
5. Presenter decides what to show.
6. Presenter calls view methods such as `showError`, `displayData`, or `close`.

### Core Participants

| Participant | Responsibility |
|---|---|
| Model | Data and business behavior |
| View interface | Operations the presenter can call |
| Concrete view | UI implementation |
| Presenter | Presentation logic and coordination |
| User event | Trigger from the view |

---

## 4. Java Coding Example

```java
interface LoginView {
    String username();
    String password();
    void showMessage(String message);
}

class LoginService {
    boolean authenticate(String username, String password) {
        return "admin".equals(username) && "secret".equals(password);
    }
}

class LoginPresenter {
    private final LoginView view;
    private final LoginService service;

    LoginPresenter(LoginView view, LoginService service) {
        this.view = view;
        this.service = service;
    }

    void loginClicked() {
        if (service.authenticate(view.username(), view.password())) {
            view.showMessage("login ok");
        } else {
            view.showMessage("login failed");
        }
    }
}

public class MvpDemo {
    public static void main(String[] args) {
        LoginView view = new LoginView() {
            public String username() { return "admin"; }
            public String password() { return "secret"; }
            public void showMessage(String message) { System.out.println(message); }
        };

        new LoginPresenter(view, new LoginService()).loginClicked();
    }
}
```

### Java Block by Block

The view is represented by an interface.

The presenter reads input from the view and calls the model/service.

The presenter decides what message to show.

The view remains passive and easy to replace with a test double.

---

## 5. Python Coding Example

```python
class LoginService:
    def authenticate(self, username, password):
        return username == "admin" and password == "secret"


class LoginPresenter:
    def __init__(self, view, service):
        self.view = view
        self.service = service

    def login_clicked(self):
        if self.service.authenticate(self.view.username(), self.view.password()):
            self.view.show_message("login ok")
        else:
            self.view.show_message("login failed")


class LoginView:
    def username(self):
        return "admin"

    def password(self):
        return "secret"

    def show_message(self, message):
        print(message)


LoginPresenter(LoginView(), LoginService()).login_clicked()
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Desktop UI | Presenter can be tested without Swing/JavaFX |
| Mobile screens | View delegates events to presenter |
| Wizard flows | Presenter owns step logic |
| Form validation | Presenter coordinates validation and messages |
| Legacy UI refactor | Move logic out of view classes |
| Test-heavy UI code | Mock view interface in unit tests |

---

## 7. Advantages Over Normal Code Without Pattern

Without MVP:
- UI classes contain validation and workflow logic
- automated tests need real UI widgets
- changing view technology is hard
- user-event code becomes tangled

With MVP:
- presenter logic is testable
- view can be passive
- model is independent of UI
- UI framework details stay behind the view interface

---

## 8. Where It Excels

It excels when:
- presentation logic is complex
- view implementation should be easily mocked
- UI framework is difficult to test directly
- you want explicit control over view updates
- forms/wizards have many states

---

## 9. Where It Fails

It fails when:
- the presenter becomes too large
- view interface mirrors every widget too closely
- simple screens get too much ceremony
- presenter and view become tightly coupled anyway
- data binding frameworks would be simpler

Split presenters by screen/use case and keep view interfaces intention-based.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Swing/JavaFX with MVP manually, GWT MVP patterns |
| Android | Classic MVP architecture |
| .NET | WinForms MVP |
| Testing | Mockito, JUnit, fake views |
| UI architecture | Passive View, Supervising Controller variants |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Highly testable presenter | More interfaces/classes |
| Passive view is simpler | Presenter can grow large |
| Clear event handling | Manual view updates |
| UI framework is isolated | Boilerplate for simple screens |
| Good for forms/wizards | View interface design needs care |

---

## 12. Real-World Identification Example

Question:

> A desktop form has validation, file loading, and display decisions embedded inside the UI class. It is almost impossible to unit test. What pattern helps?

Strong answer:

Use MVP. Define a view interface exposing only needed UI operations, move presentation decisions into a presenter, and keep file/data access in the model/service. Unit tests can mock the view and verify that the presenter calls `displayData` or `showMessage` correctly.

---

## 13. MAANG Interview Triggers

Say MVP when you hear:
- passive view
- presenter handles UI logic
- test UI logic without real UI
- view interface
- form workflow
- presenter updates view
- MVC variant for UI testability

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| View interface exposes every widget | Presenter becomes UI-framework aware | Expose intentions and values |
| Presenter contains business rules | Domain logic is misplaced | Use model/service layer |
| Presenter too large | Hard to test and maintain | Split by use case or screen area |
| View makes decisions | Passive view is lost | Delegate user events to presenter |
| No tests for presenter | Main benefit is wasted | Unit test presenter with fake view |

---

## 15. MVP vs Similar Patterns

| Pattern | Difference |
|---|---|
| MVC | Controller handles input; MVP presenter more directly manages the view |
| MVVM | MVVM relies on binding to ViewModel properties/commands |
| Front Controller | Front Controller is request entry/routing, not screen presentation logic |
| Observer | View updates may use observer, but MVP defines full roles |
| Component | Component composes reusable UI parts; MVP separates screen logic |

---

## 16. MVP Design Checklist

- What is the model/service?
- What should the view interface expose?
- What user events call the presenter?
- What decisions belong in the presenter?
- Can presenter be tested with a fake view?
- Does the view remain passive?
- Is the presenter too large?
- Are business rules outside the presenter?

---

## 17. Quick Revision Notes

- One-line summary: Presenter mediates between passive view and model.
- Memory hook: "view displays, presenter decides."
- Best for: testable UI logic.
- Avoid when: simple binding or MVC is enough.
- Interview line: "I would move presentation logic into a presenter and interact with the UI through a narrow view interface."

---

## 18. Mini Exercise

Design MVP for a file upload screen:
- define view methods
- define model/service methods
- define presenter actions for select, upload, cancel
- write one unit test idea using a fake view

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/model-view-presenter/README.md)
- [FileSelectorPresenter.java](../../github-repo/model-view-presenter/src/main/java/com/iluwatar/model/view/presenter/FileSelectorPresenter.java)
- [FileSelectorView.java](../../github-repo/model-view-presenter/src/main/java/com/iluwatar/model/view/presenter/FileSelectorView.java)
- [FileLoader.java](../../github-repo/model-view-presenter/src/main/java/com/iluwatar/model/view/presenter/FileLoader.java)
- [FileSelectorJframe.java](../../github-repo/model-view-presenter/src/main/java/com/iluwatar/model/view/presenter/FileSelectorJframe.java)
- [FileSelectorStub.java](../../github-repo/model-view-presenter/src/main/java/com/iluwatar/model/view/presenter/FileSelectorStub.java)
- [App.java](../../github-repo/model-view-presenter/src/main/java/com/iluwatar/model/view/presenter/App.java)

The repo implementation uses `FileSelectorPresenter` to react to view events, call `FileLoader`, and update the `FileSelectorView` through an interface.

