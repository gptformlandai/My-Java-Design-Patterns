# Partial Response Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/partial-response](../../github-repo/partial-response)

## How to Study This Page

Use this page in three passes:

1. First pass: understand Partial Response as returning only the fields a client needs.
2. Second pass: trace field selection, response mapping, validation, and default full response.
3. Third pass: compare Partial Response with GraphQL, pagination, projection, and streaming.

By the end, you should be able to say:

> Partial Response lets clients request a subset of fields to reduce payload size and improve API efficiency.

## 1. Technical Definition

Partial Response is an API design pattern where a client specifies which fields or sections of a resource it wants, and the server returns only that subset.

Core idea:

- Resource may have many fields.
- Client may need only a few fields.
- Request includes field selection.
- Server validates and maps requested fields.
- Response payload becomes smaller.

### 30-Second Interview Answer

I would use Partial Response when APIs return large resources but clients often need only a subset. The client sends a field list like `fields=id,title,status`, and the server returns only those fields. The trade-off is added server-side mapping and validation complexity, but it reduces bandwidth, latency, and client parsing work.

## 2. Layman and Easy to Understand Definition

Partial Response is like ordering only the sections of a report you need.

If you need summary and totals, the system does not print appendix, charts, and raw data.

In code:

- Client asks for selected fields.
- Server filters the resource.
- Response contains only requested fields.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

APIs often return more data than clients need:

```json
{
  "id": "v1",
  "title": "Demo",
  "description": "Long text",
  "url": "https://example.com/demo",
  "metadata": {}
}
```

A list page may need only `id` and `title`.

### 3.2 The Partial Response Solution

Allow field selection:

```text
GET /videos/1?fields=id,title
```

Return only:

```json
{
  "id": "v1",
  "title": "Demo"
}
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Resource | Full object available on server. |
| Field selector | Client-supplied list of desired fields. |
| Mapper | Builds filtered response. |
| Allowlist | Valid fields clients may request. |
| Default response | Full or standard field set when no selector is provided. |

### 3.4 Request Flow

1. Client requests resource with `fields`.
2. Server parses requested field names.
3. Server validates fields against allowlist.
4. Server fetches resource.
5. Server maps selected fields.
6. Server returns partial JSON.

## 4. Java Coding Example

This example maps only requested fields from a video DTO.

```java
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Set;

record Video(String id, String title, String description, String url) {
}

class FieldMapper {
    Map<String, Object> map(Video video, Set<String> fields) {
        Map<String, Object> response = new LinkedHashMap<>();

        if (fields.contains("id")) {
            response.put("id", video.id());
        }
        if (fields.contains("title")) {
            response.put("title", video.title());
        }
        if (fields.contains("description")) {
            response.put("description", video.description());
        }
        if (fields.contains("url")) {
            response.put("url", video.url());
        }

        return response;
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `Video` | Full server-side resource. |
| `fields` | Client-requested projection. |
| `LinkedHashMap` | Response preserves field order. |
| `contains` checks | Only selected fields are returned. |
| `FieldMapper` | Mapping logic has one owner. |

### Java Usage

```java
Video video = new Video("v1", "Intro", "Long description", "https://example.com/v1");
Map<String, Object> response = new FieldMapper().map(video, Set.of("id", "title"));

System.out.println(response);
```

## 5. Python Coding Example

```python
def partial_response(resource, fields):
    allowed = {"id", "title", "description", "url"}
    selected = [field for field in fields if field in allowed]
    return {field: resource[field] for field in selected}


video = {
    "id": "v1",
    "title": "Intro",
    "description": "Long description",
    "url": "https://example.com/v1",
}

print(partial_response(video, ["id", "title"]))
```

### Python Usage

Use this shape when explaining:

- Validate requested fields.
- Return only allowed fields.
- Avoid exposing internal or sensitive fields.

## 6. Where It Comes Handy in Real Life

- Mobile APIs with bandwidth limits.
- List screens that need summary fields.
- Public APIs with large resources.
- Search result cards.
- Video or media metadata APIs.
- Graph-like resources with optional sections.

## 7. Advantages Over Normal Code Without Pattern

Without Partial Response:

```text
server sends full object even when client needs two fields
```

With Partial Response:

```text
server sends only requested fields
```

Benefits:

- Reduces bandwidth.
- Lowers client parsing cost.
- Improves perceived performance.
- Avoids over-fetching.
- Helps support multiple client needs.

## 8. Where It Excels

- Resources have many optional fields.
- Clients need different field sets.
- Network payload size matters.
- Public API consumers want control.
- Backward-compatible field additions are common.

## 9. Where It Fails

- Field selection is too complex.
- Query planning becomes expensive.
- Security allowlist is missing.
- Server still fetches all expensive data before filtering.
- Cache key design ignores field selections.

For complex graph selection, GraphQL may be a better fit.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java JSON | Jackson filters, JSON Views, JsonNode/ObjectNode |
| REST | Sparse fieldsets in JSON:API style APIs |
| Query APIs | GraphQL, OData `$select` |
| Persistence | JPA projections, Spring Data projections |
| API gateways | Response transformation policies |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces payload size. | Adds mapping complexity. |
| Gives clients flexibility. | Cache keys become more complex. |
| Avoids over-fetching. | Field validation is required. |
| Improves mobile performance. | Can hide expensive server-side fetching. |

## 12. Real-World Identification Example

Scenario: A video API serves detail pages and search cards.

Partial Response fit:

- Detail page requests all fields.
- Search card requests `id,title,thumbnail`.
- Server returns only selected fields.
- Mobile payload becomes smaller.

Without it:

- Search results download large descriptions and unused URLs.
- Mobile clients waste bandwidth.

## 13. MAANG Interview Triggers

Use Partial Response when you hear:

- "Clients need different fields."
- "How do we reduce API payload size?"
- "Avoid over-fetching."
- "Mobile API is too heavy."
- "Large resource but small screen needs."

Strong answer keywords:

- fields parameter
- projection
- allowlist
- sparse fieldset
- payload size
- cache key
- backward compatibility

## 14. Common Mistakes

### Mistake 1: No field allowlist

- Why it is wrong: clients may request sensitive or internal fields.
- Better approach: validate requested fields against a public allowlist.

### Mistake 2: Filtering after expensive fetch

- Why it is wrong: response is smaller but server work is unchanged.
- Better approach: push projection into database or service calls when possible.

### Mistake 3: Cache keys ignore fields

- Why it is wrong: a cached partial response may be served for a full request.
- Better approach: include normalized field selection in cache key.

### Mistake 4: Unstable field names

- Why it is wrong: clients break when fields are renamed.
- Better approach: version or preserve public field contracts.

## 15. Partial Response vs Similar Patterns

| Pattern | Difference |
|---|---|
| Partial Response | Client selects fields from one resource. |
| Pagination | Client selects result window, not fields. |
| GraphQL | Client selects nested graph shape with typed schema. |
| Projection | General term for selecting fields, often at database level. |
| Streaming | Sends data over time; partial response usually reduces field set. |

## 16. Partial Response Design Checklist

- Which fields are publicly selectable?
- What is the default field set?
- How are requested fields validated?
- Can projection reduce database or downstream work?
- How are field selections represented in cache keys?
- What happens when requested field is unknown?
- Are sensitive fields impossible to request?
- Is field naming stable?

## 17. Quick Revision Notes

- One-line summary: Partial Response returns only the fields requested by the client.
- Three keywords: fields, projection, allowlist.
- Interview trap: shrinking JSON after already doing all expensive backend work.
- Memory trick: do not ship the whole report when the client asked for one section.

## 18. Mini Exercise

Design partial response for a user profile API.

Answer these:

1. What fields are selectable?
2. What fields are never selectable?
3. What is the default response?
4. How is the cache key built?
5. Can database projection reduce work?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/partial-response/README.md](../../github-repo/partial-response/README.md)
- [github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/Video.java](../../github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/Video.java)
- [github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/FieldJsonMapper.java](../../github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/FieldJsonMapper.java)
- [github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/VideoResource.java](../../github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/VideoResource.java)
- [github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/App.java](../../github-repo/partial-response/src/main/java/com/iluwatar/partialresponse/App.java)
