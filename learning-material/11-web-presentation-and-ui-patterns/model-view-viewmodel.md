# Model-View-ViewModel Pattern

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [model-view-viewmodel](../../github-repo/model-view-viewmodel)

---

## How to Study This Page

Study MVVM as presentation separation powered by data binding.

The key idea:
- View binds to ViewModel properties and commands.
- ViewModel exposes UI-ready state.
- Model/service owns data and business logic.

---

## 1. Technical Definition

Model-View-ViewModel is a presentation architecture pattern where the ViewModel exposes state and commands for the View to bind to, separating UI markup from presentation logic and model access.

### 30-Second Interview Answer

MVVM separates UI markup from presentation logic using a ViewModel. The view binds to ViewModel properties and commands, while the ViewModel talks to model/services and exposes UI-ready state. It is common in frameworks with data binding, such as WPF, Android data binding, JavaFX, and ZK. Its benefit is clean separation and reactive UI updates; its trade-off is binding complexity and hidden control flow.

---

## 2. Layman and Easy to Understand Definition

Think of a dashboard screen connected to a prepared data adapter.

The screen says:

```text
Show vm.bookList
Disable button when vm.selectedBook is empty
Call vm.deleteBook when clicked
```

The screen does not manually fetch or mutate the model. It binds to ViewModel state and commands.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

In UI code, views often do too much:
- fetch data
- transform model objects
- decide button state
- run commands
- update fields manually

MVVM moves that presentation state and command behavior into a ViewModel.

### MVVM Flow

1. View declares bindings to ViewModel properties.
2. ViewModel exposes UI-ready data.
3. User changes selection or clicks a button.
4. Binding framework invokes a ViewModel command or setter.
5. ViewModel updates state/model.
6. ViewModel notifies changed properties.
7. View updates automatically through binding.

### Core Participants

| Participant | Responsibility |
|---|---|
| Model | Domain data and business rules |
| View | UI markup/rendering |
| ViewModel | Presentation state and commands |
| Binding engine | Syncs view and ViewModel |
| Command | User action exposed by ViewModel |

---

## 4. Java Coding Example

```java
import java.util.ArrayList;
import java.util.List;

record Book(String name, String author) {}

class BookViewModel {
    private final List<Book> books = new ArrayList<>();
    private Book selectedBook;

    BookViewModel() {
        books.add(new Book("Design Patterns", "GoF"));
        books.add(new Book("PoEAA", "Martin Fowler"));
    }

    public List<Book> getBooks() {
        return books;
    }

    public Book getSelectedBook() {
        return selectedBook;
    }

    public void setSelectedBook(Book selectedBook) {
        this.selectedBook = selectedBook;
    }

    public void deleteSelectedBook() {
        if (selectedBook != null) {
            books.remove(selectedBook);
            selectedBook = null;
        }
    }
}

public class MvvmDemo {
    public static void main(String[] args) {
        BookViewModel vm = new BookViewModel();
        vm.setSelectedBook(vm.getBooks().get(0));
        vm.deleteSelectedBook();
        System.out.println(vm.getBooks().size());
    }
}
```

### Java Block by Block

The ViewModel exposes `getBooks` and `getSelectedBook`.

The View would bind to those properties.

The delete action is exposed as a command-like method.

The ViewModel owns presentation state, not visual rendering.

---

## 5. Python Coding Example

```python
class BookViewModel:
    def __init__(self):
        self.books = [
            {"name": "Design Patterns", "author": "GoF"},
            {"name": "PoEAA", "author": "Martin Fowler"},
        ]
        self.selected_book = None

    def select_book(self, book):
        self.selected_book = book

    def delete_selected_book(self):
        if self.selected_book:
            self.books.remove(self.selected_book)
            self.selected_book = None


vm = BookViewModel()
vm.select_book(vm.books[0])
vm.delete_selected_book()
print(len(vm.books))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Data-heavy forms | Bind fields to ViewModel properties |
| Tables/lists | Selection and commands are exposed cleanly |
| Desktop/mobile apps | Binding frameworks update UI automatically |
| Admin screens | ViewModel shapes model data for UI |
| Reactive UI | State changes notify the view |
| Testable presentation state | ViewModel can be unit tested |

---

## 7. Advantages Over Normal Code Without Pattern

Without MVVM:
- views manually fetch and transform data
- UI state is scattered
- button enable/disable logic lives in markup/code-behind
- testing requires real UI components

With MVVM:
- UI binds to ViewModel state
- commands are explicit
- ViewModel can be tested without UI
- model and view remain decoupled

---

## 8. Where It Excels

It excels when:
- the framework supports data binding
- UI has lots of state
- forms and lists need selection/commands
- views should be declarative
- presentation state should be testable

---

## 9. Where It Fails

It fails when:
- the framework has weak binding support
- binding expressions become complex business logic
- ViewModel mirrors the view too literally
- hidden binding flow makes debugging hard
- small screens do not need the extra structure

Keep ViewModels focused on presentation state and commands.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | JavaFX properties/binding, ZK binding |
| Android | ViewModel, LiveData, Data Binding, StateFlow |
| .NET | WPF, MAUI, Avalonia MVVM |
| JavaScript | Knockout, Vue-style binding, Angular components |
| Testing | JUnit/Mockito for ViewModel tests |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Great with data binding | Binding can hide control flow |
| Testable ViewModel | More concepts/classes |
| Declarative views | Debugging binding errors can be painful |
| Clear presentation state | ViewModel can become too view-specific |
| Supports reactive UI updates | Small apps may be over-structured |

---

## 12. Real-World Identification Example

Question:

> A book management screen has a list, selected item, delete button, and detail panel. The UI framework supports binding. You want markup to stay mostly declarative. What pattern fits?

Strong answer:

Use MVVM. The View binds to `bookList`, `selectedBook`, and `deleteBook` on a ViewModel. The ViewModel loads data through a service, exposes selection state, and notifies changes after commands. The View stays declarative and does not contain business or presentation decision logic.

---

## 13. MAANG Interview Triggers

Say MVVM when you hear:
- ViewModel
- data binding
- commands
- observable UI state
- declarative view
- selected item binding
- Android/WPF/JavaFX/ZK UI state

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Business logic in ViewModel | ViewModel becomes domain layer | Delegate to services/model |
| Complex binding expressions | Hard to debug and test | Move presentation logic into ViewModel methods |
| ViewModel knows concrete widgets | Coupling returns | Expose properties and commands |
| No change notification | UI becomes stale | Notify changed properties |
| Overusing MVVM on tiny screens | Adds ceremony | Use simpler state handling |

---

## 15. MVVM vs Similar Patterns

| Pattern | Difference |
|---|---|
| MVC | Controller handles input; MVVM uses binding and ViewModel commands |
| MVP | Presenter calls view methods; MVVM view binds to ViewModel |
| Observer | Binding often uses observer mechanics, but MVVM defines roles |
| Component | Component composes reusable UI parts; MVVM structures state flow |
| Front Controller | Front Controller routes web requests, not UI binding state |

---

## 16. MVVM Design Checklist

- What model/service provides data?
- What ViewModel properties does the view bind to?
- What commands does the ViewModel expose?
- How does the ViewModel notify changes?
- Is business logic outside the ViewModel?
- Are binding expressions simple?
- Can the ViewModel be tested without UI?
- Does the view avoid direct model mutation?

---

## 17. Quick Revision Notes

- One-line summary: View binds to ViewModel state and commands.
- Memory hook: "binding-friendly presenter state."
- Best for: data-bound UI frameworks.
- Avoid when: binding support is weak or screen is tiny.
- Interview line: "I would expose UI-ready properties and commands in a ViewModel, bind the view to them, and keep domain logic in services/models."

---

## 18. Mini Exercise

Design MVVM for a task list:
- ViewModel properties: task list, selected task, filter text
- commands: add, complete, delete
- define one change notification
- decide what belongs in task service versus ViewModel

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/model-view-viewmodel/README.md)
- [BookViewModel.java](../../github-repo/model-view-viewmodel/src/main/java/com/iluwatar/model/view/viewmodel/BookViewModel.java)
- [Book.java](../../github-repo/model-view-viewmodel/src/main/java/com/iluwatar/model/view/viewmodel/Book.java)
- [BookService.java](../../github-repo/model-view-viewmodel/src/main/java/com/iluwatar/model/view/viewmodel/BookService.java)
- [BookServiceImpl.java](../../github-repo/model-view-viewmodel/src/main/java/com/iluwatar/model/view/viewmodel/BookServiceImpl.java)
- [index.zul](../../github-repo/model-view-viewmodel/src/main/webapp/index.zul)
- [web.xml](../../github-repo/model-view-viewmodel/src/main/webapp/WEB-INF/web.xml)

The repo implementation uses `BookViewModel` to expose `bookList`, `selectedBook`, and a `deleteBook` command that the ZK view binds to from `index.zul`.

