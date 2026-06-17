# Page Object Pattern

Category: Testing Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [page-object](../../github-repo/page-object)

---

## How to Study This Page

Study Page Object as "hide UI locator details behind page-specific methods."

Remember the core idea:

```text
Test says what user does; Page Object knows how UI does it.
```

In interviews, the important point is maintainability: when the UI changes, tests should not all need locator updates.

---

## 1. Technical Definition

Page Object is a test automation pattern that represents a UI page or component as an object, encapsulating locators and UI interactions behind meaningful methods used by tests.

### 30-Second Interview Answer

Page Object keeps UI tests maintainable by moving selectors and interaction details into page-specific classes. Tests call business-readable methods such as `login` or `selectAlbum` instead of directly finding elements everywhere. I use it for Selenium, Playwright, Cypress, or HtmlUnit tests when many tests interact with the same pages. The benefit is reduced duplication and easier UI maintenance; the trade-off is that page objects can become bloated or accidentally contain assertions and business test logic.

---

## 2. Layman and Easy to Understand Definition

Imagine a TV remote.

You press "volume up" without knowing the circuit details inside the TV. If the TV internals change, the remote button can still mean the same thing.

A Page Object is like that remote for UI tests: tests call clear actions, and the page object handles UI details.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

UI tests become fragile when each test directly uses selectors:

```text
find #username
type user
find #password
type password
find #loginButton
click
```

If an element ID changes, many tests break.

### Page Object Flow

1. Create one class for a page or meaningful UI component.
2. Store locators and UI access inside that class.
3. Expose methods that describe user actions.
4. Tests call page methods instead of raw selectors.
5. If UI changes, update the page object in one place.

### Core Participants

| Participant | Responsibility |
|---|---|
| Test class | Describes scenario and assertions |
| Page object | Encapsulates page locators and actions |
| Locator | Finds UI elements |
| Driver/client | Interacts with the browser or UI runtime |
| Navigation method | Moves to another page object |
| Assertion | Usually stays in the test, not the page object |

---

## 4. Java Coding Example

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

class LoginPage {
    private final WebDriver driver;

    private final By username = By.id("username");
    private final By password = By.id("password");
    private final By loginButton = By.id("loginButton");

    LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    LoginPage enterUsername(String value) {
        driver.findElement(username).sendKeys(value);
        return this;
    }

    LoginPage enterPassword(String value) {
        driver.findElement(password).sendKeys(value);
        return this;
    }

    HomePage login() {
        driver.findElement(loginButton).click();
        return new HomePage(driver);
    }
}

class HomePage {
    private final WebDriver driver;

    HomePage(WebDriver driver) {
        this.driver = driver;
    }

    boolean isAt() {
        return driver.getTitle().equals("Home");
    }
}
```

### Java Block by Block

`LoginPage` owns login locators.

The test does not know element IDs.

Methods return `this` for fluent entry or a new page object after navigation.

Assertions stay outside the page object in the test.

---

## 5. Python Coding Example

```python
class LoginPage:
    def __init__(self, page):
        self.page = page

    def enter_username(self, username):
        self.page.fill("#username", username)
        return self

    def enter_password(self, password):
        self.page.fill("#password", password)
        return self

    def login(self):
        self.page.click("#loginButton")
        return HomePage(self.page)


class HomePage:
    def __init__(self, page):
        self.page = page

    def is_at(self):
        return self.page.title() == "Home"


def test_login(page):
    home = LoginPage(page).enter_username("admin").enter_password("password").login()
    assert home.is_at()
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Login flows | Many tests reuse the same login interaction |
| Admin dashboards | Complex pages need stable test actions |
| Checkout flows | Tests should read like user behavior |
| Form-heavy apps | Field locators change often |
| Regression suites | Reduces duplicate selector maintenance |
| Component tests | Component objects can model reusable UI pieces |

---

## 7. Advantages Over Normal Code Without Pattern

Without Page Object:
- selectors are duplicated across tests
- UI changes break many files
- tests describe mechanics instead of intent
- long UI scripts are hard to review

With Page Object:
- locators live in one page class
- test code reads like a scenario
- common actions are reusable
- UI changes are easier to absorb

---

## 8. Where It Excels

It excels when:
- many tests use the same page
- UI selectors change over time
- workflows should be readable
- tests need reusable navigation/actions
- teams want test code separated from UI mechanics
- page/component boundaries are stable

---

## 9. Where It Fails

It fails when:
- page objects become huge utility classes
- assertions are hidden inside every page method
- page objects duplicate business logic
- one object models too many screens
- tests become unreadable through over-abstraction
- dynamic UI behavior needs better component-level modeling

Use component objects, screen fragments, or direct test code for very small one-off UI interactions.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Selenium WebDriver, HtmlUnit, Selenide, FluentLenium |
| Python | Selenium, Playwright, pytest fixtures |
| JavaScript | Playwright, Cypress, WebdriverIO |
| .NET | Selenium, Playwright .NET |
| Mobile | Appium screen objects |
| Test runners | JUnit, TestNG, pytest, Jest |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces selector duplication | Adds extra classes |
| Improves UI test readability | Page objects can become bloated |
| Centralizes UI maintenance | Poor boundaries cause confusion |
| Encourages reusable actions | Hidden assertions reduce test clarity |
| Supports fluent workflows | Over-abstraction can hide important test steps |

---

## 12. Real-World Identification Example

Question:

> A UI regression suite has 80 tests that all use raw Selenium locators for login and checkout. A button ID change breaks dozens of files. What pattern helps?

Strong answer:

Use Page Object. Create `LoginPage`, `CheckoutPage`, and related component objects that encapsulate locators and user actions. Tests should call methods like `loginAs` and `submitPayment`, while assertions remain in tests. When a selector changes, update the page object once instead of editing many tests.

---

## 13. MAANG Interview Triggers

Say Page Object when you hear:
- UI test automation
- Selenium tests are brittle
- selectors duplicated everywhere
- web page abstraction
- page object model
- test scripts hard to maintain
- UI locator changes
- readable user workflows in tests

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Assertions inside page objects | Test intent gets hidden | Keep assertions in tests unless checking page state helpers |
| Giant page object | Becomes unmaintainable | Split by page sections/components |
| Exposing raw elements | Tests still depend on UI details | Expose meaningful actions |
| One method per click only | Still too mechanical | Prefer user-intent methods |
| No waits/synchronization strategy | Tests become flaky | Centralize waits around stable page actions |

---

## 15. Page Object vs Similar Patterns

| Pattern | Difference |
|---|---|
| Facade | Page Object is a testing-specific facade over UI elements |
| Screenplay Pattern | Screenplay models actors/tasks; Page Object models pages/components |
| Arrange-Act-Assert | AAA structures test flow; Page Object abstracts UI interaction details |
| Service Stub | Stub replaces external dependency; Page Object wraps UI dependency |
| Component Object | Component Object is a smaller Page Object for reusable UI fragments |

---

## 16. Page Object Design Checklist

- What page or component does this object represent?
- Which locators should be private?
- What user-intent methods should be public?
- Does a method return the next page object after navigation?
- Are assertions kept in the test?
- Is synchronization handled consistently?
- Is the object small enough to maintain?
- Are selectors stable and meaningful?
- Can tests read like user scenarios?

---

## 17. Quick Revision Notes

- One-line summary: Encapsulate UI locators and actions inside page-specific test objects.
- Memory hook: "tests say what, page objects know how."
- Best for: UI regression and end-to-end test suites.
- Avoid when: a tiny one-off UI test does not need extra abstraction.
- Interview line: "I would keep selectors private, expose user-level actions, and leave assertions in the test."

---

## 18. Mini Exercise

Design Page Objects for a checkout flow:
- create `CartPage`, `ShippingPage`, and `PaymentPage`
- define two user-intent methods per page
- decide what each method returns after navigation
- identify one reusable component object
- explain where assertions should live

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/page-object/README.md)
- [Page.java](../../github-repo/page-object/src/test/java/com/iluwatar/pageobject/pages/Page.java)
- [LoginPage.java](../../github-repo/page-object/src/test/java/com/iluwatar/pageobject/pages/LoginPage.java)
- [AlbumListPage.java](../../github-repo/page-object/src/test/java/com/iluwatar/pageobject/pages/AlbumListPage.java)
- [AlbumPage.java](../../github-repo/page-object/src/test/java/com/iluwatar/pageobject/pages/AlbumPage.java)
- [LoginPageTest.java](../../github-repo/page-object/src/test/java/com/iluwatar/pageobject/LoginPageTest.java)
- [AlbumListPageTest.java](../../github-repo/page-object/src/test/java/com/iluwatar/pageobject/AlbumListPageTest.java)

The repo implementation models login and album pages as page objects. Tests call methods such as `enterUsername`, `enterPassword`, `login`, and `selectAlbum` instead of directly manipulating HTML elements everywhere.
