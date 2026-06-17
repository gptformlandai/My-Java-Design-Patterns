# Service Stub Pattern

Category: Testing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [service-stub](../../github-repo/service-stub)

---

## How to Study This Page

Study Service Stub as "replace a slow or unavailable external service with a small predictable implementation for testing."

Remember the core idea:

```text
Code under test -> service interface -> stub implementation
```

The key interview maturity is knowing that a stub helps isolation, but it must stay realistic enough to avoid false confidence.

---

## 1. Technical Definition

Service Stub is a testing pattern where a lightweight substitute implements the same interface as an external service and returns controlled responses so tests can run quickly and deterministically.

### 30-Second Interview Answer

Service Stub replaces a real dependency with a predictable implementation during testing. It is useful when the real service is slow, flaky, costly, unavailable, or hard to control. The production code depends on an interface, and tests inject the stub. The benefit is fast deterministic tests; the trade-off is that the stub can drift from real service behavior, so contract or integration tests are still needed.

---

## 2. Layman and Easy to Understand Definition

Imagine practicing a payment flow without charging a real credit card.

Instead of calling the real payment processor, you use a fake processor that always returns "approved" or "declined" based on your test setup.

That fake predictable service is a stub.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

Tests become unreliable when they call real external systems:

```text
test -> network -> third-party service -> random latency/failure/cost
```

The test might fail because the external system is down, not because your code is wrong.

### Service Stub Flow

1. Define an interface for the external dependency.
2. Production code depends on the interface.
3. Real implementation calls the real external system.
4. Stub implementation returns controlled test responses.
5. Tests inject the stub.
6. Separate integration or contract tests verify the real integration.

### Core Participants

| Participant | Responsibility |
|---|---|
| Service interface | Contract used by production code |
| Real service | Calls external dependency |
| Stub service | Returns predictable responses |
| Code under test | Uses the interface |
| Test setup | Chooses stub responses |
| Contract tests | Catch drift from real service behavior |

---

## 4. Java Coding Example

```java
interface PaymentGateway {
    PaymentResult charge(String accountId, int cents);
}

record PaymentResult(boolean approved, String code) {}

class StubPaymentGateway implements PaymentGateway {
    @Override
    public PaymentResult charge(String accountId, int cents) {
        if (accountId.startsWith("fail")) {
            return new PaymentResult(false, "DECLINED");
        }
        return new PaymentResult(true, "APPROVED");
    }
}

class CheckoutService {
    private final PaymentGateway gateway;

    CheckoutService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    boolean checkout(String accountId, int cents) {
        return gateway.charge(accountId, cents).approved();
    }
}

public class ServiceStubDemo {
    public static void main(String[] args) {
        CheckoutService service = new CheckoutService(new StubPaymentGateway());
        System.out.println(service.checkout("acct-1", 5000));
    }
}
```

### Java Block by Block

`PaymentGateway` is the boundary between production code and the external service.

`StubPaymentGateway` returns deterministic responses.

`CheckoutService` does not know whether the gateway is real or stubbed.

Tests can verify checkout behavior without real payment calls.

---

## 5. Python Coding Example

```python
class StubPaymentGateway:
    def charge(self, account_id, cents):
        if account_id.startswith("fail"):
            return {"approved": False, "code": "DECLINED"}
        return {"approved": True, "code": "APPROVED"}


class CheckoutService:
    def __init__(self, gateway):
        self.gateway = gateway

    def checkout(self, account_id, cents):
        result = self.gateway.charge(account_id, cents)
        return result["approved"]


service = CheckoutService(StubPaymentGateway())
assert service.checkout("acct-1", 5000) is True
assert service.checkout("fail-1", 5000) is False
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Payment provider tests | Avoid real charges and external availability |
| Email/SMS tests | Avoid sending real messages |
| Sentiment/ML APIs | Avoid slow or nondeterministic predictions |
| Third-party APIs | Control success, failure, and timeout scenarios |
| Microservice development | Work before dependent service is ready |
| CI pipelines | Keep tests fast and repeatable |

---

## 7. Advantages Over Normal Code Without Pattern

Without Service Stub:
- tests depend on network and external uptime
- test data is hard to control
- CI becomes slow and flaky
- failure scenarios are difficult to reproduce

With Service Stub:
- tests run quickly
- responses are deterministic
- failure paths can be simulated
- development can continue before real service is available

---

## 8. Where It Excels

It excels when:
- external service is slow
- external service costs money
- failure scenarios need control
- CI must be deterministic
- service contract is stable
- dependency injection is already available

---

## 9. Where It Fails

It fails when:
- stub behavior drifts from the real service
- only happy paths are stubbed
- tests never exercise serialization or network behavior
- the external contract changes silently
- the stub becomes more complex than the real client
- teams mistake stubbed tests for full integration coverage

Use contract tests, integration tests, sandbox environments, or service virtualization when realism matters.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java | Mockito stubbing, WireMock, MockServer, Spring MockRestServiceServer |
| JVM contracts | Spring Cloud Contract, Pact JVM |
| Python | unittest.mock, responses, requests-mock, pytest fixtures |
| JavaScript | Jest mocks, MSW, Nock |
| HTTP stubs | WireMock, Mountebank, Hoverfly |
| Containers | Testcontainers for higher-realism integration tests |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Fast deterministic tests | Can drift from real service |
| Easy failure simulation | May hide integration bugs |
| Avoids external cost and rate limits | Requires stub maintenance |
| Supports offline development | Unrealistic behavior gives false confidence |
| Good for CI stability | Needs contract/integration tests as backup |

---

## 12. Real-World Identification Example

Question:

> Your order service tests call a real payment provider. CI fails randomly due to network issues, and developers are afraid of triggering real charges. What pattern helps?

Strong answer:

Use Service Stub. Define a `PaymentGateway` interface and inject a stub implementation in tests that returns controlled approved, declined, and timeout-like responses. Keep separate contract or sandbox integration tests to verify the real payment provider behavior.

---

## 13. MAANG Interview Triggers

Say Service Stub when you hear:
- external service in tests
- slow or flaky dependency
- deterministic test responses
- test double
- fake third-party API
- CI instability
- simulate failure paths
- service unavailable during development

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Stub only success paths | Failure handling remains untested | Add declined, timeout, malformed, and error cases |
| Stub differs from real contract | Tests pass while production fails | Use contract tests |
| Hardcode too much behavior | Stub becomes brittle | Make scenario setup explicit |
| Replace all integration tests | Real wiring is never verified | Keep a smaller integration suite |
| Stub has hidden logic | Test behavior becomes surprising | Keep stubs simple and predictable |

---

## 15. Service Stub vs Similar Patterns

| Pattern | Difference |
|---|---|
| Mock Object | Mock verifies interactions; Stub mainly supplies predefined responses |
| Fake | Fake has working simplified behavior, often with in-memory storage; Stub is usually scenario-specific |
| Service Virtualization | More advanced environment-level simulation of services |
| Adapter | Adapter wraps an external API for production use; Stub implements the same interface for tests |
| Contract Test | Contract test verifies stub/consumer expectations against provider behavior |

---

## 16. Service Stub Design Checklist

- What external dependency is being isolated?
- Is there an interface or adapter boundary?
- What success responses are needed?
- What failure responses are needed?
- Can tests choose responses clearly?
- Is the stub deterministic?
- How will contract drift be caught?
- Which tests still need the real service?
- Is the stub simple enough to trust?

---

## 17. Quick Revision Notes

- One-line summary: Replace an external service with a predictable test implementation.
- Memory hook: "fake service, real contract."
- Best for: fast unit/service tests around external dependencies.
- Avoid when: you need to verify actual wire-level integration.
- Interview line: "I would inject a stub for deterministic tests, then keep contract or integration tests to catch drift."

---

## 18. Mini Exercise

Design a Service Stub for an email provider:
- define the email service interface
- create a stub that records sent messages
- simulate provider failure
- test success and failure paths
- describe one contract test you would keep

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/service-stub/README.md)
- [SentimentAnalysisServer.java](../../github-repo/service-stub/src/main/java/com/iluwatar/servicestub/SentimentAnalysisServer.java)
- [RealSentimentAnalysisServer.java](../../github-repo/service-stub/src/main/java/com/iluwatar/servicestub/RealSentimentAnalysisServer.java)
- [StubSentimentAnalysisServer.java](../../github-repo/service-stub/src/main/java/com/iluwatar/servicestub/StubSentimentAnalysisServer.java)
- [App.java](../../github-repo/service-stub/src/main/java/com/iluwatar/servicestub/App.java)
- [StubSentimentAnalysisServerTest.java](../../github-repo/service-stub/src/test/java/com/iluwatar/servicestub/StubSentimentAnalysisServerTest.java)
- [RealSentimentAnalysisServerTest.java](../../github-repo/service-stub/src/test/java/com/iluwatar/servicestub/RealSentimentAnalysisServerTest.java)

The repo implementation defines a sentiment analysis interface, a slow random real implementation, and a deterministic stub that returns positive, negative, or neutral sentiment based on input keywords.
