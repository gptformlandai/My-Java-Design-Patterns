# Money Pattern

Category: Domain Modeling and Business Rule Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/money](../../github-repo/money)

## How to Study This Page

Use this page in three passes:

1. First pass: understand money as amount plus currency, not just a number.
2. Second pass: rewrite the Java example and identify precision, currency checks, arithmetic, and rounding policy.
3. Third pass: compare Money with Value Object, primitive numbers, BigDecimal, and currency conversion services.

By the end, you should be able to say:

> Money is a domain value object that combines amount and currency so financial operations stay precise and currency-safe.

## 1. Technical Definition

Money is a domain modeling pattern that represents monetary values as a dedicated object containing amount, currency, and financial operations with explicit precision and rounding rules.

Core idea:

- Amount and currency belong together.
- Avoid floating-point errors.
- Prevent accidental cross-currency arithmetic.
- Centralize rounding and formatting rules.
- Treat money as a value object.

### 30-Second Interview Answer

I would use Money whenever the domain performs financial calculations. Instead of passing `double amount` and `String currency`, I create a `Money` value object backed by `BigDecimal` or minor units. It validates currency compatibility for addition/subtraction and centralizes rounding. The trade-off is needing explicit currency conversion and rounding policy, but that is exactly the safety financial code needs.

## 2. Layman and Easy to Understand Definition

Money is not just a number.

`10 USD` and `10 EUR` are not the same thing. If code stores only `10`, it loses important business meaning. A Money object keeps the amount and currency together.

In code:

- `Money(10, "USD")`
- `Money(10, "EUR")`
- Adding different currencies is blocked unless conversion is explicit.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Primitive money code often looks like this:

```java
double price = 19.99;
String currency = "USD";
```

Problems:

- Floating-point rounding errors.
- Currency can be forgotten.
- USD and EUR can be accidentally added.
- Rounding rules are scattered.
- Formatting and tax logic duplicate.

### 3.2 The Money Solution

Use a dedicated type:

```java
Money price = Money.usd("19.99");
Money tax = Money.usd("1.50");
Money total = price.add(tax);
```

The object protects financial correctness.

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Amount | Numeric value, often `BigDecimal` or minor units. |
| Currency | ISO code such as USD, INR, EUR. |
| Money object | Value object combining amount and currency. |
| Rounding policy | Defines scale and rounding behavior. |
| Exchange service | Converts currencies explicitly when needed. |

### 3.4 Precision Rule

Avoid binary floating point for money:

```java
double bad = 0.1 + 0.2;
```

Prefer:

```java
BigDecimal amount = new BigDecimal("0.30");
```

or store minor units:

```java
long cents = 30;
```

## 4. Java Coding Example

This example models immutable money with `BigDecimal`.

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.Currency;
import java.util.Objects;

public final class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        if (amount == null || currency == null) {
            throw new IllegalArgumentException("amount and currency are required");
        }
        this.currency = currency;
        this.amount = amount.setScale(currency.getDefaultFractionDigits(), RoundingMode.HALF_UP);
    }

    public static Money of(String amount, String currencyCode) {
        return new Money(new BigDecimal(amount), Currency.getInstance(currencyCode));
    }

    public Money add(Money other) {
        requireSameCurrency(other);
        return new Money(amount.add(other.amount), currency);
    }

    public Money subtract(Money other) {
        requireSameCurrency(other);
        return new Money(amount.subtract(other.amount), currency);
    }

    public Money multiply(int factor) {
        if (factor < 0) {
            throw new IllegalArgumentException("factor must be non-negative");
        }
        return new Money(amount.multiply(BigDecimal.valueOf(factor)), currency);
    }

    private void requireSameCurrency(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("currency mismatch");
        }
    }

    @Override
    public boolean equals(Object other) {
        if (this == other) {
            return true;
        }
        if (!(other instanceof Money that)) {
            return false;
        }
        return amount.compareTo(that.amount) == 0 && currency.equals(that.currency);
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount.stripTrailingZeros(), currency);
    }
}
```

### Java Block by Block Explanation

`BigDecimal` avoids floating-point precision bugs.

`Currency` keeps currency explicit.

`add` and `subtract` require the same currency.

The object is immutable: operations return new `Money` values.

Rounding is centralized in the constructor.

### Java Usage

Use Money in Java when:

- Prices, balances, invoices, fees, or taxes exist.
- Currency matters.
- Financial arithmetic must be safe.
- Rounding policy must be consistent.
- You want domain language instead of raw numbers.

## 5. Python Coding Example

```python
from dataclasses import dataclass
from decimal import Decimal, ROUND_HALF_UP


@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str

    @staticmethod
    def of(amount, currency):
        rounded = Decimal(amount).quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)
        return Money(rounded, currency)

    def add(self, other):
        self._require_same_currency(other)
        return Money.of(self.amount + other.amount, self.currency)

    def subtract(self, other):
        self._require_same_currency(other)
        return Money.of(self.amount - other.amount, self.currency)

    def multiply(self, factor):
        if factor < 0:
            raise ValueError("factor must be non-negative")
        return Money.of(self.amount * factor, self.currency)

    def _require_same_currency(self, other):
        if self.currency != other.currency:
            raise ValueError("currency mismatch")


total = Money.of("19.99", "USD").add(Money.of("1.50", "USD"))
print(total)
```

### Python Usage

Python money code should use:

- `Decimal`, not float.
- Explicit currency codes.
- Immutable dataclasses.
- Centralized rounding.
- Separate exchange-rate service for conversion.

## 6. Where It Comes Handy in Real Life

- E-commerce pricing.
- Payments.
- Refunds.
- Taxes.
- Subscriptions.
- Wallet balances.
- Accounting ledgers.
- Invoices.
- Foreign exchange conversion.

## 7. Advantages Over Normal Code Without Pattern

### Without Money

```java
double amount = 10.00;
String currency = "USD";
```

Problems:

- Amount and currency can separate.
- Floating-point issues appear.
- Wrong-currency arithmetic is possible.
- Rounding policy is inconsistent.

### With Money

```java
Money total = price.add(tax);
```

Benefits:

- Currency is always present.
- Arithmetic is controlled.
- Precision is safer.
- Domain code is clearer.

## 8. Where It Excels

- Financial systems.
- Any pricing/tax logic.
- Multi-currency domains.
- Domain-driven design.
- Ledger and payment code.
- Systems requiring correctness over convenience.

## 9. Where It Fails

- Non-financial approximate calculations.
- Cases where exchange rates are incorrectly embedded inside Money.
- Performance hot loops with huge volumes and careless object allocation.
- Systems that still use float internally.
- Domains without clear rounding rules.

## 10. Prebuilt Frameworks and Packages

### Java

- JSR 354 JavaMoney.
- Moneta implementation.
- `BigDecimal`
- `Currency`
- Bean Validation for amount constraints.

### Python

- `decimal.Decimal`
- `babel` for formatting.
- `moneyed` / `py-moneyed`
- Custom immutable dataclasses.

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Prevents currency mismatch. | Requires explicit conversion service. |
| Avoids floating-point errors. | Adds domain type overhead. |
| Centralizes rounding. | Rounding rules can be subtle. |
| Makes financial code readable. | Persistence mapping may need converters. |
| Works as a value object. | Object creation can matter at scale. |

## 12. Real-World Identification Example

Scenario:

You are designing checkout totals.

Money fit:

- Item price is Money.
- Tax is Money.
- Discount is Money.
- Total is Money.
- Currency conversion happens before arithmetic or through a dedicated exchange service.

What would go wrong without it:

- USD and EUR may be added.
- Floating-point rounding causes incorrect totals.
- Tax rounding differs by service.

## 13. MAANG Interview Triggers

Use Money when you hear:

- "Financial calculations."
- "Amount plus currency."
- "Avoid floating point for money."
- "Rounding rules."
- "Currency mismatch."
- "Value object."
- "Payments or wallet balance."

### Interview-Ready Answer Format

1. Represent amount and currency together.
2. Use `BigDecimal`/minor units, not floating point.
3. Make Money immutable.
4. Validate same currency for arithmetic.
5. Centralize rounding.
6. Use separate service for currency conversion.

## 14. Common Mistakes

### Mistake 1: Using `double` for Money

Binary floating-point values are unsafe for exact financial arithmetic.

### Mistake 2: Adding Different Currencies

Conversion must be explicit.

### Mistake 3: Rounding Everywhere

Rounding should follow a clear centralized policy.

### Mistake 4: Mutable Money

Immutable money values are safer and easier to reason about.

### Mistake 5: Money Object Does Exchange Rates

Currency conversion usually belongs in an exchange/pricing service.

## 15. Money vs Similar Patterns

| Pattern | Difference |
|---|---|
| Money | Specific value object for amount and currency. |
| Value Object | General pattern; Money is a classic example. |
| BigDecimal | Numeric type, not a domain concept by itself. |
| Quantity | Represents measured amount, usually with unit. |
| Exchange Service | Converts between currencies using rates. |

## 16. Money Design Checklist

| Question | Why it matters |
|---|---|
| What numeric representation is used? | Prevents precision bugs. |
| Is currency mandatory? | Prevents meaningless amounts. |
| Is Money immutable? | Keeps values safe. |
| What rounding policy applies? | Ensures consistent totals. |
| Can currencies be mixed? | Should usually be rejected. |
| Where does conversion happen? | Keeps Money focused. |

## 17. Quick Revision Notes

- Money is amount plus currency.
- Avoid `double`.
- Use `BigDecimal`, Decimal, or minor units.
- Make it immutable.
- Reject cross-currency arithmetic.
- Conversion belongs elsewhere.

## 18. Mini Exercise

Design Money for `Invoice`.

Operations:

- `add`
- `subtract`
- `multiply`
- `allocateAcross(n)`

Rules:

- Same currency required.
- Rounding strategy is explicit.
- No negative invoice total unless domain allows it.

## 19. Source Reference in This Repo

The repository's Money implementation encapsulates amount and currency, plus arithmetic operations and currency checks.

Useful files:

- [github-repo/money/README.md](../../github-repo/money/README.md)
- [github-repo/money/src/main/java/com/iluwatar/Money.java](../../github-repo/money/src/main/java/com/iluwatar/Money.java)
- [github-repo/money/src/main/java/com/iluwatar/CannotAddTwoCurrienciesException.java](../../github-repo/money/src/main/java/com/iluwatar/CannotAddTwoCurrienciesException.java)
- [github-repo/money/src/main/java/com/iluwatar/CannotSubtractException.java](../../github-repo/money/src/main/java/com/iluwatar/CannotSubtractException.java)
- [github-repo/money/src/main/java/com/iluwatar/App.java](../../github-repo/money/src/main/java/com/iluwatar/App.java)

