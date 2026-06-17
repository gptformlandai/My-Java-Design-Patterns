# Monitor Pattern

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [monitor](../../github-repo/monitor)

---

## How to Study This Pattern

Study Monitor as the concurrency pattern behind synchronized objects.

The key idea:
- Put shared state inside one object.
- Protect that state with a lock.
- Allow only one thread at a time to execute critical methods.

---

## 1. Technical Definition

Monitor is a concurrency pattern that encapsulates shared state and the synchronization used to access it, ensuring mutual exclusion and sometimes condition-based waiting.

### 30-Second Interview Answer

A Monitor wraps shared mutable state behind synchronized methods or locks so only one thread can modify it at a time. In Java, `synchronized` methods are a common monitor-style implementation. I would use it for small shared objects where correctness matters more than maximum parallelism, such as account transfers or bounded state updates.

---

## 2. Layman Explanation

Imagine a room with one key. Anyone can use the room, but only the person holding the key can enter. When they leave, the next person can use it.

The room is the shared object.

The key is the monitor lock.

---

## 3. Bit by Bit Explanation

### Problem

Multiple threads can corrupt shared state when they modify it at the same time.

Example:

```text
Thread A reads balance = 100
Thread B reads balance = 100
Both subtract 20
Final balance accidentally becomes 80 instead of 60
```

### Pattern Flow

1. Shared state is kept private inside an object.
2. Public methods acquire a lock before reading or writing state.
3. Only one thread can execute locked methods at a time.
4. The method preserves invariants.
5. The lock is released when the method finishes.

### Core Participants

| Participant | Responsibility |
|---|---|
| Monitor object | Owns the shared state and lock |
| Critical method | Reads/writes shared state safely |
| Client threads | Call methods concurrently |
| Condition waiters | Optional threads waiting for state changes |

---

## 4. Java Coding Example

```java
class BankAccount {
    private int balance;

    public BankAccount(int openingBalance) {
        this.balance = openingBalance;
    }

    public synchronized void deposit(int amount) {
        balance += amount;
    }

    public synchronized boolean withdraw(int amount) {
        if (balance < amount) {
            return false;
        }
        balance -= amount;
        return true;
    }

    public synchronized int balance() {
        return balance;
    }
}

public class MonitorDemo {
    public static void main(String[] args) throws InterruptedException {
        BankAccount account = new BankAccount(1000);

        Thread t1 = new Thread(() -> account.withdraw(300));
        Thread t2 = new Thread(() -> account.withdraw(400));

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(account.balance());
    }
}
```

### Java Block by Block

`balance` is private shared state.

Each public method is `synchronized`, so callers acquire the object's monitor lock.

The invariant is that withdrawals cannot make the account negative.

Concurrent callers cannot interleave inside the critical methods.

---

## 5. Python Coding Example

```python
from threading import Lock, Thread


class BankAccount:
    def __init__(self, opening_balance):
        self.balance = opening_balance
        self.lock = Lock()

    def withdraw(self, amount):
        with self.lock:
            if self.balance < amount:
                return False
            self.balance -= amount
            return True

    def get_balance(self):
        with self.lock:
            return self.balance


account = BankAccount(1000)
threads = [
    Thread(target=lambda: account.withdraw(300)),
    Thread(target=lambda: account.withdraw(400)),
]

for thread in threads:
    thread.start()
for thread in threads:
    thread.join()

print(account.get_balance())
```

---

## 6. When to Use It

Use Monitor when:
- several threads share mutable state
- state invariants must be protected
- the object is small enough to lock as a unit
- correctness is more important than fine-grained parallelism
- the critical section is short

Interview triggers:
- "race condition"
- "shared mutable state"
- "synchronized methods"
- "critical section"
- "protect invariant"

---

## 7. When Not to Use It

Avoid Monitor when:
- operations are long-running or blocking
- high throughput requires finer-grained concurrency
- state can be made immutable
- message passing or queues would remove sharing
- distributed state is involved

Use concurrent collections, actors, immutable objects, database transactions, or lock-free structures when they fit better.

---

## 8. Real-World Use Cases

| Use case | Protected invariant |
|---|---|
| Bank transfer | Total balance remains correct |
| Inventory decrement | Stock never goes negative |
| Connection pool | Available/borrowed counts stay consistent |
| In-memory rate counter | Count updates are atomic |
| Bounded buffer | Size remains within limits |
| Cache metadata | Map and statistics stay consistent |

---

## 9. Advantages Over Normal Code

Without Monitor:
- thread interleavings corrupt state
- invariants are scattered
- each caller must remember locking rules

With Monitor:
- locking is centralized
- object controls its own consistency
- methods are safer to call concurrently
- invariants live near the state they protect

---

## 10. Where It Excels

Monitor excels for small critical sections around one object:
- update balance
- move money between two internal accounts
- increment counter
- borrow/release a resource
- mutate an in-memory aggregate

---

## 11. Where It Fails

It fails when:
- a synchronized method calls external slow code
- locks are acquired in inconsistent order
- the monitor becomes a bottleneck
- callers need atomic operations across multiple monitor objects
- the object exposes mutable internal state

Keep monitor state private and critical sections small.

---

## 12. Frameworks and Libraries

| Ecosystem | Tools |
|---|---|
| Java | `synchronized`, `ReentrantLock`, `Condition` |
| Java collections | `ConcurrentHashMap`, `Collections.synchronizedList` |
| Python | `threading.Lock`, `threading.RLock`, `Condition` |
| C# | `lock`, `Monitor` |
| C++ | `std::mutex`, `std::lock_guard` |

---

## 13. Pros and Cons

| Pros | Cons |
|---|---|
| Simple mental model | Can reduce parallelism |
| Protects invariants | Deadlock risk with multiple locks |
| Encapsulates locking | Hard to debug lock contention |
| Native Java support | Not suitable for distributed state |
| Good for small shared objects | Blocking calls inside lock are dangerous |

---

## 14. Real-World Identification Scenario

Question:

> Multiple worker threads update balances in an in-memory bank object. Occasionally the total balance becomes wrong. What pattern helps?

Strong answer:

Use a Monitor. Keep account balances private inside a bank object and make transfer and balance methods synchronized or protected by a lock. The transfer method should check and update balances inside one critical section so no two threads can corrupt the invariant.

---

## 15. Interview Triggers

Say Monitor when you hear:
- shared mutable object
- only one thread should enter
- critical section
- synchronized method
- race condition
- data invariant
- mutual exclusion

---

## 16. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Exposing internal arrays/maps | Callers mutate without lock | Return copies or read-only views |
| Holding lock during I/O | Blocks all other callers | Move I/O outside the lock |
| Locking too broadly | Kills throughput | Keep critical section small |
| Multiple locks without order | Deadlocks | Define lock ordering |
| Assuming `volatile` replaces locking | Does not make compound updates atomic | Use lock or atomic classes |

---

## 17. Pattern Comparison and Design Checklist

| Pattern | Difference |
|---|---|
| Guarded Suspension | Monitor may include condition waiting; guarded suspension focuses on waiting for a condition |
| Actor Model | Actor avoids shared state by serializing messages |
| Active Object | Active Object decouples method call from execution thread |
| Semaphore | Controls permits; monitor protects object state |
| Transaction Script | Application-level business flow, not thread synchronization |

Design checklist:
- What shared state must be protected?
- Which invariants must always hold?
- Are all reads and writes protected by the same lock?
- Does any method expose mutable internal state?
- Are critical sections short?
- Could high contention require a different design?

---

## 18. Quick Revision Notes and Mini Exercise

- One-line summary: Shared state plus lock inside one object.
- Memory hook: "one key for the shared room."
- Best for: protecting small mutable objects.
- Avoid when: long work or high contention would make the lock a bottleneck.
- Interview line: "I would encapsulate the shared state and protect all state-changing methods with one consistent synchronization policy."

Mini exercise:

Create a `SafeInventory` class with `reserve(sku, quantity)` and `release(sku, quantity)`. Protect the stock map with a monitor and ensure stock never goes negative.

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/monitor/README.md)
- [Bank.java](../../github-repo/monitor/src/main/java/com/iluwatar/monitor/Bank.java)
- [Main.java](../../github-repo/monitor/src/main/java/com/iluwatar/monitor/Main.java)

The repo implementation uses `Bank` as the monitor object. `transfer`, `getBalance`, and `getBalance(accountNumber)` are synchronized so concurrent worker threads cannot corrupt account state.
