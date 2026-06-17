# Mediator Pattern

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/mediator](../../github-repo/mediator)

## How to Study This Page

Use this page in three passes:

1. First pass: understand objects communicating through a coordinator instead of directly.
2. Second pass: rewrite the Java example and identify colleagues, mediator, and message flow.
3. Third pass: compare Mediator with Observer, Facade, Event Bus, and Controller.

By the end, you should be able to say:

> Mediator centralizes communication between related objects so they do not all depend directly on each other.

## 1. Technical Definition

Mediator is a behavioral design pattern that encapsulates how a set of objects interact by routing their communication through a mediator object.

Core idea:

- Colleagues do not call each other directly.
- Colleagues talk to the mediator.
- Mediator coordinates interactions.
- Coupling between colleagues is reduced.

### 30-Second Interview Answer

I would use Mediator when many objects have complex many-to-many communication. Instead of each object knowing every other object, each object talks to a mediator that coordinates the interaction. This simplifies colleague objects and centralizes communication policy. The trade-off is that the mediator can become a large god object if it accumulates too much business logic.

## 2. Layman and Easy to Understand Definition

Mediator is like a group chat coordinator.

Instead of everyone calling everyone separately, each person sends messages to one coordinator. The coordinator decides who should hear what.

In code:

- Participants are colleagues.
- Coordinator is mediator.
- Messages/actions flow through mediator.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Imagine five UI components all updating each other:

```java
searchBox.updateTable();
searchBox.updateFilters();
table.updateDetailsPanel();
filters.updateTable();
```

This becomes tangled.

Problems:

- Components know too much about each other.
- Reuse becomes hard.
- Changes ripple across many classes.
- Event ordering becomes unclear.

### 3.2 The Mediator Solution

Route interactions through a mediator:

```java
searchBox.changed("laptop");
mediator.componentChanged(searchBox);
```

The mediator coordinates updates.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Mediator | Coordinates communication. |
| Concrete mediator | Implements routing rules. |
| Colleague | Object that communicates through mediator. |
| Client | Creates mediator and registers colleagues. |

### 3.4 Message Flow

1. Colleague performs an action.
2. Colleague notifies mediator.
3. Mediator decides affected colleagues.
4. Mediator calls those colleagues.
5. Colleagues remain unaware of each other.

## 4. Java Coding Example

This example coordinates a checkout page.

```java
interface CheckoutMediator {
    void shippingAddressChanged(String country);
    void paymentMethodChanged(String method);
}

final class ShippingPanel {
    private final CheckoutMediator mediator;

    ShippingPanel(CheckoutMediator mediator) {
        this.mediator = mediator;
    }

    void selectCountry(String country) {
        System.out.println("Shipping country: " + country);
        mediator.shippingAddressChanged(country);
    }

    void showInternationalNotice() {
        System.out.println("Show customs notice");
    }
}

final class PaymentPanel {
    private final CheckoutMediator mediator;

    PaymentPanel(CheckoutMediator mediator) {
        this.mediator = mediator;
    }

    void selectMethod(String method) {
        System.out.println("Payment method: " + method);
        mediator.paymentMethodChanged(method);
    }

    void disableCashOnDelivery() {
        System.out.println("Cash on delivery disabled");
    }
}

final class CheckoutPageMediator implements CheckoutMediator {
    private ShippingPanel shippingPanel;
    private PaymentPanel paymentPanel;

    void register(ShippingPanel shippingPanel, PaymentPanel paymentPanel) {
        this.shippingPanel = shippingPanel;
        this.paymentPanel = paymentPanel;
    }

    @Override
    public void shippingAddressChanged(String country) {
        if (!country.equals("US")) {
            shippingPanel.showInternationalNotice();
            paymentPanel.disableCashOnDelivery();
        }
    }

    @Override
    public void paymentMethodChanged(String method) {
        System.out.println("Recalculate checkout rules for " + method);
    }
}

public final class MediatorDemo {
    public static void main(String[] args) {
        CheckoutPageMediator mediator = new CheckoutPageMediator();
        ShippingPanel shipping = new ShippingPanel(mediator);
        PaymentPanel payment = new PaymentPanel(mediator);
        mediator.register(shipping, payment);

        shipping.selectCountry("CA");
        payment.selectMethod("CARD");
    }
}
```

### Java Block by Block Explanation

`CheckoutMediator` defines communication events.

`ShippingPanel` and `PaymentPanel` are colleagues.

`CheckoutPageMediator` knows how changes in one colleague affect another.

The colleagues do not directly reference each other.

### Java Usage

Use Mediator when:

- Object interactions are many-to-many.
- Components should not depend directly on each other.
- Communication rules are cohesive.
- A coordinator naturally exists.

## 5. Python Coding Example

```python
class CheckoutMediator:
    def register(self, shipping, payment):
        self.shipping = shipping
        self.payment = payment

    def shipping_address_changed(self, country):
        if country != "US":
            self.shipping.show_international_notice()
            self.payment.disable_cash_on_delivery()

    def payment_method_changed(self, method):
        print(f"Recalculate checkout rules for {method}")


class ShippingPanel:
    def __init__(self, mediator):
        self.mediator = mediator

    def select_country(self, country):
        print(f"Shipping country: {country}")
        self.mediator.shipping_address_changed(country)

    def show_international_notice(self):
        print("Show customs notice")


class PaymentPanel:
    def __init__(self, mediator):
        self.mediator = mediator

    def select_method(self, method):
        print(f"Payment method: {method}")
        self.mediator.payment_method_changed(method)

    def disable_cash_on_delivery(self):
        print("Cash on delivery disabled")


mediator = CheckoutMediator()
shipping = ShippingPanel(mediator)
payment = PaymentPanel(mediator)
mediator.register(shipping, payment)
shipping.select_country("CA")
```

### Python Usage

Python mediators often show up as:

- Controller objects.
- Coordinators.
- Event dispatchers.
- Workflow orchestrators.
- UI screen models.

## 6. Where It Comes Handy in Real Life

- UI form coordination.
- Chat rooms.
- Workflow orchestration.
- Air traffic control style coordination.
- Wizard-style multi-step flows.
- Domain components with many interactions.
- Message routing inside a module.

## 7. Advantages Over Normal Code Without Pattern

### Without Mediator

```java
shippingPanel.setPaymentPanel(paymentPanel);
paymentPanel.setShippingPanel(shippingPanel);
```

Problems:

- Components are tightly coupled.
- Reuse is difficult.
- Changes ripple across components.
- Many-to-many communication gets messy.

### With Mediator

```java
shippingPanel.selectCountry("CA");
```

Benefits:

- Components only know mediator.
- Communication policy is centralized.
- Colleagues are simpler.
- Coordination is easier to test.

## 8. Where It Excels

- Related objects with complex communication.
- UI screens.
- Workflow coordinators.
- Modules with clear orchestration rules.
- Reducing many-to-many dependencies.
- Centralizing local interaction policy.

## 9. Where It Fails

- Simple one-to-one communication.
- Mediator becomes too large.
- Business rules get hidden inside coordination code.
- Global event bus would be more appropriate.
- Direct calls are simpler and clear.

## 10. Prebuilt Libraries and Packages

### Java

- `Executor` and `ExecutorService` coordinate task execution.
- GUI frameworks use controllers/listeners as mediators.
- Spring application events can mediate components.
- JMS/message brokers mediate producers and consumers.

### Python

- GUI controller patterns.
- `asyncio` event loop coordination.
- Pub/sub libraries.
- Workflow orchestration objects.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces colleague coupling. | Mediator can become a god object. |
| Centralizes interaction rules. | Adds indirection. |
| Makes colleagues reusable. | Can hide control flow. |
| Useful for UI coordination. | Mediator can be hard to test if too broad. |

## 12. Real-World Identification Example

Scenario:

You are building a checkout screen with shipping, payment, coupon, tax, and summary panels.

Without Mediator:

- Each panel knows several others.
- A country change triggers scattered updates.

With Mediator:

- Panels notify `CheckoutMediator`.
- Mediator recalculates dependent panels.
- Panels remain focused on display and local events.

## 13. MAANG Interview Triggers

Use Mediator when you hear:

- "Many objects communicate with each other."
- "Reduce many-to-many dependencies."
- "Central coordinator."
- "UI components affect each other."
- "Workflow orchestration."
- "Objects should not know each other directly."

### Interview-Ready Answer Format

1. Identify colleagues and interaction events.
2. Define mediator interface.
3. Make colleagues depend only on mediator.
4. Centralize routing/coordination in concrete mediator.
5. Keep business rules out if they belong elsewhere.
6. Mention god-object risk.

## 14. Common Mistakes

### Mistake 1: Mediator Becomes Business God Object

Keep domain rules in domain services when they are not communication rules.

### Mistake 2: Using Mediator for Simple Calls

Direct dependencies are fine when interaction is simple and stable.

### Mistake 3: Global Mediator for Everything

Prefer local mediators scoped to one workflow or module.

### Mistake 4: Hidden Event Ordering

Document and test the order of updates.

### Mistake 5: Colleagues Still Know Each Other

That defeats the coupling benefit.

## 15. Mediator vs Similar Patterns

| Pattern | Difference |
|---|---|
| Mediator | Centralizes communication among related objects. |
| Observer | One subject notifies subscribers, often one-to-many. |
| Facade | Simplifies external API to subsystem. |
| Controller | Handles user/system input; may act as mediator. |
| Event Bus | Broader pub/sub mechanism, often application-wide. |

## 16. Mediator Design Checklist

| Question | Why it matters |
|---|---|
| What objects are colleagues? | Defines mediator scope. |
| What events flow through mediator? | Keeps API clear. |
| Is mediator local? | Prevents global coupling. |
| Does mediator own business rules? | Avoids god object. |
| Are update orders tested? | Prevents subtle UI/workflow bugs. |

## 17. Quick Revision Notes

- Mediator routes communication.
- Colleagues do not call each other directly.
- Good for many-to-many interactions.
- Common in UI and workflows.
- Watch for god object.
- Keep scope local and cohesive.

## 18. Mini Exercise

Design Mediator for `SearchPage`.

Colleagues:

- Search box.
- Filter panel.
- Result table.
- Pagination.

Events:

- Query changed.
- Filter changed.
- Page changed.

## 19. Source Reference in This Repo

The repository's Mediator implementation uses a `Party` mediator and several party members communicating through it.

Useful files:

- [github-repo/mediator/README.md](../../github-repo/mediator/README.md)
- [github-repo/mediator/src/main/java/com/iluwatar/mediator/Party.java](../../github-repo/mediator/src/main/java/com/iluwatar/mediator/Party.java)
- [github-repo/mediator/src/main/java/com/iluwatar/mediator/PartyImpl.java](../../github-repo/mediator/src/main/java/com/iluwatar/mediator/PartyImpl.java)
- [github-repo/mediator/src/main/java/com/iluwatar/mediator/PartyMember.java](../../github-repo/mediator/src/main/java/com/iluwatar/mediator/PartyMember.java)
- [github-repo/mediator/src/main/java/com/iluwatar/mediator/App.java](../../github-repo/mediator/src/main/java/com/iluwatar/mediator/App.java)

