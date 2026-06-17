# Arrange-Act-Assert Pattern

Category: Testing Patterns  
MAANG interview meter: Medium  
Software usage meter: High  
Repository module: [arrange-act-assert](../../github-repo/arrange-act-assert)

---

## How to Study This Page

Study Arrange-Act-Assert as the default shape of a readable unit test.

Remember the three phases:

```text
Arrange -> Act -> Assert
```

In interviews and code reviews, this pattern helps you explain what a test prepares, what behavior it executes, and what result it verifies.

---

## 1. Technical Definition

Arrange-Act-Assert is a unit test organization pattern that divides a test into setup, execution, and verification phases so the test's intent is easy to read and maintain.

### 30-Second Interview Answer

Arrange-Act-Assert structures a test into three parts. Arrange creates the object, data, mocks, or state needed for the test. Act executes exactly the behavior being tested. Assert verifies the expected result or interaction. I use it because it makes tests readable, focused, and easier to debug. The main trade-off is that some complex integration scenarios need helper setup or multiple assertions, but the core idea still keeps the test disciplined.

---

## 2. Layman and Easy to Understand Definition

Think of testing a coffee machine:

- Arrange: put water and coffee powder in the machine.
- Act: press the brew button.
- Assert: check that coffee came out and the cup is full.

You prepare, you perform, you verify.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Tests become hard to understand when setup, execution, and verification are mixed:

```text
create object
call method
assert something
change object
assert another thing
call another method
assert again
```

When this fails, it is not obvious which behavior broke.

### AAA Flow

1. Arrange the required test data and dependencies.
2. Act by calling the method or behavior under test.
3. Assert the expected output, state, exception, or interaction.
4. Keep the test focused on one behavior.
5. Move repeated setup into helpers only when it improves readability.

### Core Participants

| Participant | Responsibility |
|---|---|
| System under test | Object or function being tested |
| Test data | Inputs and initial state |
| Dependencies | Real, fake, mock, or stub collaborators |
| Act call | The behavior being verified |
| Assertions | Expected outcome checks |
| Test name | Describes the scenario and expectation |

---

## 4. Java Coding Example

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

class Cart {
    private int total;

    void addItem(int price) {
        total += price;
    }

    int total() {
        return total;
    }
}

class CartTest {
    @Test
    void addsItemPriceToCartTotal() {
        // Arrange
        Cart cart = new Cart();

        // Act
        cart.addItem(25);

        // Assert
        assertEquals(25, cart.total());
    }
}
```

### Java Block by Block

Arrange creates the `Cart`.

Act calls `addItem`.

Assert checks the final total.

The test has one main reason to fail: item price was not added correctly.

---

## 5. Python Coding Example

```python
class Cart:
    def __init__(self):
        self.total = 0

    def add_item(self, price):
        self.total += price


def test_adds_item_price_to_cart_total():
    # Arrange
    cart = Cart()

    # Act
    cart.add_item(25)

    # Assert
    assert cart.total == 25
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Unit tests | Makes behavior under test obvious |
| TDD | Gives a repeatable structure for new tests |
| Bug regression tests | Clearly captures setup, trigger, and expected fix |
| API/service tests | Separates request setup, call, and response checks |
| Domain model tests | Verifies one business behavior at a time |
| Code reviews | Makes test quality easier to assess quickly |

---

## 7. Advantages Over Normal Code Without Pattern

Without Arrange-Act-Assert:
- test intent is harder to scan
- several behaviors may be tested in one method
- failures take longer to diagnose
- setup and verification become tangled

With Arrange-Act-Assert:
- tests have a predictable shape
- the behavior under test is obvious
- failures point to one scenario
- future maintainers can change tests confidently

---

## 8. Where It Excels

It excels when:
- each test verifies one behavior
- setup is small or can be named clearly
- assertions describe one expected outcome
- tests are read often in code reviews
- teams practice TDD or BDD
- regressions need clear reproduction steps

---

## 9. Where It Fails

It fails when:
- a test tries to verify too many behaviors
- setup is huge and hides the scenario
- Act contains multiple unrelated operations
- Assert checks random implementation details
- the pattern is followed as comments but not as discipline
- integration tests require richer lifecycle hooks

Use helper builders, fixtures, or nested test contexts when setup becomes heavy, but keep the test flow readable.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | JUnit, AssertJ, Hamcrest, Mockito |
| Python | pytest, unittest |
| JavaScript | Jest, Mocha, Vitest |
| .NET | xUnit, NUnit, MSTest |
| BDD | Cucumber, Behave, SpecFlow |
| Test data | Fixture builders, Object Mother, factory libraries |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Very readable test structure | Can feel repetitive |
| Makes failure diagnosis easier | Complex setup may need helpers |
| Encourages focused tests | Not every integration flow fits perfectly |
| Works across languages | Comments alone do not guarantee good tests |
| Supports TDD and BDD thinking | Multiple assertions need judgment |

---

## 12. Real-World Identification Example

Question:

> A unit test creates data, calls several methods, asserts in the middle, mutates state again, and asserts again. When it fails, nobody knows which behavior broke. What pattern helps?

Strong answer:

Use Arrange-Act-Assert. Split the test into one scenario per method: arrange the object and inputs, act by calling one behavior, and assert the expected result. If multiple behaviors are important, write separate tests so each failure points to one reason.

---

## 13. MAANG Interview Triggers

Say Arrange-Act-Assert when you hear:
- readable unit tests
- test has too many steps
- Given When Then
- setup execution verification
- one reason to fail
- TDD structure
- test readability
- hard-to-debug test failure

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Multiple Act phases | Test verifies several behaviors | Split into separate tests |
| Huge Arrange block | Scenario is hidden | Use builders or named fixtures |
| Assertions before Act | Flow becomes confusing | Reserve assertions for verification |
| Asserting implementation details | Test becomes brittle | Assert observable behavior |
| Vague test name | Reader cannot infer purpose | Name scenario and expected result |

---

## 15. Arrange-Act-Assert vs Similar Patterns

| Pattern | Difference |
|---|---|
| Given-When-Then | BDD wording for the same mental model |
| Test Fixture | Fixture provides setup data; AAA structures the full test |
| Service Stub | Stub can be part of Arrange; AAA organizes the test |
| Mock Object | Mock verifies interactions; AAA decides where setup, action, and verification live |
| Page Object | Page Object abstracts UI actions; AAA structures each UI test |

---

## 16. Arrange-Act-Assert Design Checklist

- What behavior is this test proving?
- What setup is truly required?
- Is there one clear Act call?
- Are assertions checking observable behavior?
- Does the test have one main reason to fail?
- Can repeated setup be moved into a helper?
- Is the test name specific?
- Would a future reader understand the scenario in 10 seconds?

---

## 17. Quick Revision Notes

- One-line summary: Prepare the scenario, execute behavior, verify outcome.
- Memory hook: "setup, do, check."
- Best for: unit tests and focused behavior tests.
- Avoid when: using it as superficial comments while testing many behaviors.
- Interview line: "I would split the test into Arrange, Act, and Assert so each test has one behavior and one clear failure reason."

---

## 18. Mini Exercise

Refactor a messy test for wallet withdrawal:
- arrange wallet balance and withdrawal amount
- act by calling withdraw once
- assert returned success value
- assert final balance
- split insufficient-funds behavior into a separate test

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/arrange-act-assert/README.md)
- [Cash.java](../../github-repo/arrange-act-assert/src/main/java/com/iluwatar/arrangeactassert/Cash.java)
- [CashAAATest.java](../../github-repo/arrange-act-assert/src/test/java/com/iluwatar/arrangeactassert/CashAAATest.java)
- [CashAntiAAATest.java](../../github-repo/arrange-act-assert/src/test/java/com/iluwatar/arrangeactassert/CashAntiAAATest.java)

The repo implementation tests a `Cash` class. `CashAAATest` separates each behavior into Arrange, Act, and Assert phases, while `CashAntiAAATest` shows a harder-to-maintain test that mixes many behaviors together.
