# Server Session Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/server-session](../../github-repo/server-session)

## How to Study This Page

Use this page in three passes:

1. First pass: understand server-side session storage as state behind a session id.
2. Second pass: trace login, session creation, cookie/token return, lookup, expiration, and logout.
3. Third pass: compare Server Session with JWT, stateless auth, sticky sessions, and distributed cache.

By the end, you should be able to say:

> Server Session keeps user session state on the server and gives the client only a session identifier.

## 1. Technical Definition

Server Session is a web application pattern where session state is stored server-side and referenced by a client-held session id.

Core idea:

- Client authenticates.
- Server creates session id.
- Server stores session data.
- Client sends session id on later requests.
- Server looks up state and enforces expiration or logout.

### 30-Second Interview Answer

I would use Server Session when the server must control user session state, support immediate logout, store sensitive session attributes, or invalidate sessions centrally. The client holds only a session id, usually in a secure cookie. The trade-off is server-side storage and scaling complexity, so large systems often use distributed session stores like Redis.

## 2. Layman and Easy to Understand Definition

Server Session is like getting a claim ticket.

The ticket has only an id. The actual belongings are stored safely behind the counter.

In software:

- Client stores session id.
- Server stores actual session data.
- Every request sends the id.
- Server uses the id to load session state.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

HTTP is stateless:

```text
request 1 does not automatically remember request 2
```

But applications need continuity:

- Logged-in user.
- Shopping cart.
- Multistep form.
- User preferences.
- CSRF state.

### 3.2 The Server Session Solution

Store state on the server:

```text
client cookie: session_id=abc
server store: abc -> userId=42, role=ADMIN, expiresAt=...
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Session id | Opaque identifier held by client. |
| Session store | Server-side state storage. |
| Login handler | Creates session after authentication. |
| Session middleware | Loads session on each request. |
| Expiration policy | Removes old sessions. |
| Logout handler | Invalidates session. |

### 3.4 Request Flow

1. User logs in.
2. Server creates random session id.
3. Server stores session data.
4. Server sends session id in secure cookie.
5. Later request includes cookie.
6. Server loads session and authorizes request.
7. Session expires or logout deletes it.

## 4. Java Coding Example

This example shows a minimal session manager.

```java
import java.time.Instant;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

record Session(String userId, Instant expiresAt) {
}

class SessionManager {
    private final Map<String, Session> sessions = new ConcurrentHashMap<>();

    String createSession(String userId) {
        String sessionId = UUID.randomUUID().toString();
        sessions.put(sessionId, new Session(userId, Instant.now().plusSeconds(1800)));
        return sessionId;
    }

    Session findValidSession(String sessionId) {
        Session session = sessions.get(sessionId);
        if (session == null || session.expiresAt().isBefore(Instant.now())) {
            sessions.remove(sessionId);
            return null;
        }
        return session;
    }

    void logout(String sessionId) {
        sessions.remove(sessionId);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `sessionId` | Client-visible opaque id. |
| `Session` | Server-side state. |
| `createSession` | Login creates stored state. |
| `findValidSession` | Requests load and validate session. |
| `logout` | Server can invalidate immediately. |

### Java Usage

```java
SessionManager sessions = new SessionManager();
String sessionId = sessions.createSession("user-42");

System.out.println(sessions.findValidSession(sessionId));
sessions.logout(sessionId);
System.out.println(sessions.findValidSession(sessionId));
```

## 5. Python Coding Example

```python
from datetime import datetime, timedelta
from uuid import uuid4


class SessionManager:
    def __init__(self):
        self.sessions = {}

    def create(self, user_id):
        session_id = str(uuid4())
        self.sessions[session_id] = {
            "user_id": user_id,
            "expires_at": datetime.utcnow() + timedelta(minutes=30),
        }
        return session_id

    def find(self, session_id):
        session = self.sessions.get(session_id)
        if not session or session["expires_at"] < datetime.utcnow():
            self.sessions.pop(session_id, None)
            return None
        return session


manager = SessionManager()
sid = manager.create("user-42")
print(manager.find(sid))
```

### Python Usage

Use this shape when explaining:

- Client does not store full session data.
- Server can expire or delete sessions.
- Distributed systems need shared session storage.

## 6. Where It Comes Handy in Real Life

- Web login sessions.
- Shopping carts.
- Admin dashboards.
- Multistep forms.
- CSRF protection.
- Enterprise apps with immediate logout requirements.

## 7. Advantages Over Normal Code Without Pattern

Without Server Session:

```text
client must send all state or user logs in every request
```

With Server Session:

```text
client sends session id and server loads state
```

Benefits:

- Sensitive state stays server-side.
- Immediate logout is possible.
- Session expiration is centrally controlled.
- Client payload is small.
- State can be updated without changing client token.

## 8. Where It Excels

- Server must invalidate sessions centrally.
- Session data changes often.
- Sensitive state should not live on client.
- Browser cookie flow is acceptable.
- Distributed session store is available.

## 9. Where It Fails

- Stateless horizontal scaling is a hard requirement.
- Session store becomes bottleneck.
- Sticky sessions hide scaling problems.
- Cookies are not secured.
- Session ids are predictable.

Use strong random ids, secure cookies, and shared storage if multiple instances serve traffic.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java web | Servlet `HttpSession`, Spring Session, Spring Security |
| Stores | Redis, JDBC session store, Hazelcast, Memcached |
| Security | Secure, HttpOnly, SameSite cookies |
| Cloud | Managed Redis, database-backed session stores |
| Alternatives | JWT, opaque token introspection |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Keeps sensitive state server-side. | Requires server-side storage. |
| Supports immediate invalidation. | Needs shared store for scaling. |
| Simple client model. | Session store can become hot path. |
| Easy to change session attributes. | Cookie/session security must be strong. |

## 12. Real-World Identification Example

Scenario: Admin portal needs immediate logout and role changes.

Server Session fit:

- User logs in and receives session cookie.
- Server stores user id, role, and expiration.
- Admin role changes can invalidate active sessions.
- Logout deletes session from store.

Without it:

- Stateless tokens may remain valid until expiration.
- Sensitive state may be pushed to clients.

## 13. MAANG Interview Triggers

Use Server Session when you hear:

- "How do we manage login state?"
- "Need immediate logout."
- "Where do we store session data?"
- "How do sessions scale across servers?"
- "Stateful web application."

Strong answer keywords:

- session id
- secure cookie
- server-side store
- expiration
- logout invalidation
- Redis
- sticky sessions
- CSRF

## 14. Common Mistakes

### Mistake 1: Storing sessions only in one app instance

- Why it is wrong: requests routed to another instance lose session.
- Better approach: use shared store or sticky sessions with caution.

### Mistake 2: Predictable session ids

- Why it is wrong: attackers can guess valid sessions.
- Better approach: use cryptographically strong random ids.

### Mistake 3: Missing cookie security flags

- Why it is wrong: session theft risk increases.
- Better approach: set Secure, HttpOnly, and SameSite appropriately.

### Mistake 4: No expiration cleanup

- Why it is wrong: session storage grows forever.
- Better approach: enforce TTL and background cleanup.

## 15. Server Session vs Similar Patterns

| Pattern | Difference |
|---|---|
| Server Session | Session state stored server-side. |
| JWT | Claims stored client-side in signed token. |
| Opaque Token | Client stores id; auth server introspects token. |
| Sticky Session | Load balancer routes same user to same instance. |
| Cache | May store sessions but is broader than session management. |

## 16. Server Session Design Checklist

- What data is stored in session?
- Where is the session store?
- How is session id generated?
- What cookie flags are used?
- What is idle timeout and absolute timeout?
- How does logout invalidate state?
- How does scaling across instances work?
- How is session fixation prevented?

## 17. Quick Revision Notes

- One-line summary: Server Session stores user state on the server behind a session id.
- Three keywords: cookie, store, expiration.
- Interview trap: ignoring session scaling across multiple app instances.
- Memory trick: client holds claim ticket; server holds the state.

## 18. Mini Exercise

Design sessions for an admin dashboard.

Answer these:

1. What data goes in session?
2. What cookie flags are required?
3. Where are sessions stored?
4. How does logout work?
5. How are expired sessions cleaned?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/server-session/README.md](../../github-repo/server-session/README.md)
- [github-repo/server-session/src/main/java/com/iluwatar/sessionserver/App.java](../../github-repo/server-session/src/main/java/com/iluwatar/sessionserver/App.java)
- [github-repo/server-session/src/main/java/com/iluwatar/sessionserver/LoginHandler.java](../../github-repo/server-session/src/main/java/com/iluwatar/sessionserver/LoginHandler.java)
- [github-repo/server-session/src/main/java/com/iluwatar/sessionserver/LogoutHandler.java](../../github-repo/server-session/src/main/java/com/iluwatar/sessionserver/LogoutHandler.java)
