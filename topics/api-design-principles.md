# Robust RESTful API Design Principles

Designing a robust RESTful API is critical for mobile applications, where network bandwidth, latency, and reliability are variable factors. A well-designed API ensures scalability, maintainability, and a smooth developer experience.

## 1. Resource-Oriented Design

REST APIs are based on resources, which represent entities in your system.

*   **Do Use Nouns, Not Verbs:** URIs should represent resources (nouns), not actions (verbs).
    *   **Good:** `GET /users`, `POST /orders`
    *   **Bad:** `GET /getUsers`, `POST /createOrder`
*   **Do Use Plural Nouns:** Keep resource names consistent, typically plural.
    *   `GET /users/123` (Read user 123)
    *   `GET /users` (List all users)
*   **Do Use Hierarchy and Relationships:** Use nesting to indicate relationships.
    *   `GET /users/123/orders` (Get orders for user 123)
    *   `GET /users/123/orders/456` (Get specific order 456 for user 123)
    *   **Don't:** Avoid deep nesting (more than 2 levels). Prefer flattening if possible: `GET /orders/456`.

## 2. HTTP Methods

Use standard HTTP methods to perform actions on resources.

| Method | Action | Safety | Idempotency | Description |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | Read | Safe | Yes | Retrieve a representation of a resource. Should not modify state. |
| **POST** | Create | Unsafe | No | Create a new resource. Not idempotent (retrying might create duplicates). |
| **PUT** | Update/Replace | Unsafe | Yes | Replace a resource entirely. |
| **PATCH** | Partial Update | Unsafe | No* | Update specific fields of a resource. (*Can be designed to be idempotent). |
| **DELETE** | Delete | Unsafe | Yes | Remove a resource. |

*   **Safe:** The operation does not modify the resource.
*   **Idempotent:** Making the same request multiple times has the same effect as making it once.

## 3. Standard HTTP Status Codes

**Don't** invent your own error codes. **Do** use standard HTTP status codes to communicate the result of a request.

*   **2xx: Success**
    *   `200 OK`: Request succeeded.
    *   `201 Created`: Resource successfully created (usually returns a `Location` header).
    *   `204 No Content`: Request succeeded, but there is no content to return (common for DELETE or PUT).

*   **3xx: Redirection**
    *   `301 Moved Permanently`: Resource has moved to a new URL.
    *   `304 Not Modified`: Client's cached version is still valid (saves bandwidth).

*   **4xx: Client Error**
    *   `400 Bad Request`: General client error (e.g., malformed JSON, validation error).
    *   `401 Unauthorized`: Authentication required (missing or invalid token).
    *   `403 Forbidden`: Authenticated, but not authorized to access this resource.
    *   `404 Not Found`: Resource does not exist.
    *   `429 Too Many Requests`: Rate limit exceeded.

*   **5xx: Server Error**
    *   `500 Internal Server Error`: Something went wrong on the server.
    *   `503 Service Unavailable`: Server is down (maintenance or overloaded).

## 4. Versioning

APIs evolve. Versioning ensures backward compatibility for existing mobile clients (which users might not update immediately).

*   **URI Versioning (Recommended for Mobile):**
    *   `GET /v1/users`
    *   *Pros:* Explicit, easy to test, easy to cache.
    *   *Cons:* "Pollutes" the URI.
*   **Header Versioning:**
    *   `Accept: application/vnd.myapi.v1+json`
    *   *Pros:* Cleaner URIs.
    *   *Cons:* Harder to test in browser, cache fragmentation.

## 5. Filtering, Sorting, and Searching

**Do** keep the base resource URL clean and use query parameters for data manipulation.

*   **Filtering:** `GET /users?role=admin&active=true`
*   **Sorting:** `GET /users?sort=-created_at` (`-` for descending, `+` for ascending)
*   **Searching:** `GET /users?q=alex`
*   **Field Selection (Partial Response):** `GET /users?fields=id,name,email` (Great for mobile to reduce payload size).

## 6. Error Handling

**Do** return consistent, structured error responses. This makes parsing errors on the client side predictable.

### Standardization
*   **Code:** Use a machine-readable string code (e.g., `USER_ALREADY_EXISTS`) rather than relying on HTTP status codes alone.
*   **Message:** Provide a human-readable message, preferably localized if the backend supports it, or generic enough for the client to map to a localized string.
*   **Details:** Include specific validation errors or additional metadata to help the user fix the issue.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "There were errors with your submission.",
    "details": [
      {
        "field": "email",
        "issue": "invalid_format",
        "description": "Please enter a valid email address."
      },
       {
        "field": "age",
        "issue": "too_young",
        "description": "User must be at least 18 years old."
      }
    ]
  }
}
```

## 7. Rate Limiting

**Do** protect your backend from abuse and spikes in traffic. Return appropriate headers so the client knows their limits.

*   `X-RateLimit-Limit`: The number of allowed requests in the current period.
*   `X-RateLimit-Remaining`: The number of remaining requests in the current period.
*   `X-RateLimit-Reset`: The time at which the current period resets.
*   **Response:** Return `429 Too Many Requests` when the limit is exceeded.

## 8. Idempotency for Non-Idempotent Operations

Mobile networks are unreliable. A client might send a `POST` request (e.g., "Pay $10"), the server processes it, but the response is lost. The client retries, and the server shouldn't charge the user twice.

*   **How to Generate Keys:** The client should generate a unique key (typically a UUID v4) for every critical mutation request.
*   **Where to Include:** Pass this key in a custom HTTP header, e.g., `Idempotency-Key` or `X-Request-ID`.
*   **Server Logic:**
    1.  The server receives the request.
    2.  It checks a shared cache (like Redis) or database to see if this `Idempotency-Key` has been seen before *for this user*.
    3.  If **seen and processed**: Return the *cached response* (including the original status code and body). Do not re-execute logic.
    4.  If **seen and processing**: Return a `409 Conflict` or a "Processing" status, telling the client to wait.
    5.  If **not seen**: Execute the logic, store the result (key + response) with a TTL (Time To Live, e.g., 24 hours), and return the response.
*   **Error Handling:** If the client sends a different request body with the *same* key, the server should return a `422 Unprocessable Entity` or `409 Conflict` to indicate misuse.

## 9. Date and Time

*   **Do** always use **ISO 8601** strings for dates: `2023-10-27T10:00:00Z`.
*   **Do** always store and return dates in **UTC**. Let the mobile client handle local time formatting.

## 10. Security Best Practices

*   **Do** use **HTTPS**: Always use TLS/SSL. No exceptions.
*   **Do** use **Authentication**: Use `Authorization: Bearer <token>` (e.g., JWT). Avoid passing tokens in URL parameters.
*   **Do** use **Input Validation**: Validate all input on the server side. Never trust the client.

### API Key Placement
When using API keys (typically to identify the client application rather than the user):
*   **Do Use Headers:** Pass the key in a standard header (e.g., `X-API-Key` or `X-Client-ID`). Headers are less likely to be inadvertently logged than URLs.
*   **Don't Use Query Parameters:** Avoid `/resource?api_key=xyz`. URLs are often logged in cleartext by proxies, servers, and browsers, leaking your keys.
*   **Mobile Reality:** Remember that any API key embedded in a mobile app binary can be extracted. Treat these keys as **public**. Use them for rate limiting and identification, not for granting sensitive permissions.

### Protecting Against Scraping
*   **Rate Limiting:** Aggressive rate limiting (per IP and per User) makes scraping inefficient.
*   **User-Agent Filtering:** Block known bot User-Agents, though this is easily spoofed.
*   **App Attestation:** Use platform-specific integrity checks (Google Play Integrity API, Apple App Attest) to ensure requests are coming from your genuine, unmodified mobile app, not a script.
*   **WAF (Web Application Firewall):** Use a WAF to detect and block suspicious traffic patterns.

## 11. Resource Management (Over-fetching vs. Under-fetching)

Finding the balance between "one request for everything" and "many small requests" is key for mobile performance.

*   **Over-fetching:** Getting too much data.
    *   *Problem:* Wastes bandwidth, slows down parsing, uses more memory.
    *   *Solution:* **Field Selection**. Allow clients to specify exactly what they need: `GET /users/123?fields=id,name,avatar_url`.
*   **Under-fetching:** Getting too little data, requiring subsequent requests (N+1 problem).
    *   *Problem:* Increases latency due to multiple round-trips.
    *   *Solution:* **Resource Embedding** or **Compound Resources**. Allow clients to include related resources in a single call: `GET /orders/123?embed=items,payment_details`.
*   **The "Backend for Frontend" (BFF) Pattern:** Ideally, create a dedicated API layer specifically for the mobile app that aggregates data from multiple microservices into a single, mobile-optimized response.

## 12. Common Mistakes & Anti-Patterns

Even experienced developers can fall into these traps. Avoid these anti-patterns to build a truly robust API.

### 12.1. Tunneling Everything Through GET or POST
**Mistake:** Ignoring HTTP semantics and using `GET` to modify data or `POST` for everything.
*   **Bad:** `GET /deleteUser?id=123` (GET should be safe and idempotent).
*   **Bad:** `POST /getUser?id=123` (POST is not cacheable by default; use GET for retrieval).
*   **Fix:** Use `DELETE /users/123` and `GET /users/123`.

### 12.2. Returning 200 OK for Errors
**Mistake:** The HTTP status is 200, but the body contains an error code. This breaks HTTP-aware clients and monitoring tools.
*   **Bad:**
    ```http
    HTTP/1.1 200 OK
    {
      "success": false,
      "error": "Not Found"
    }
    ```
*   **Fix:** Use 4xx codes for client errors.
    ```http
    HTTP/1.1 404 Not Found
    {
      "error": { "message": "Resource not found" }
    }
    ```

### 12.3. Putting Actions in URIs (Verbs instead of Nouns)
**Mistake:** Using verbs like `create`, `update`, `delete` in the URL path.
*   **Bad:** `POST /users/create`, `POST /orders/123/cancel`
*   **Fix:** Use the HTTP method to describe the action and the URI to describe the resource.
    *   `POST /users`
    *   `POST /orders/123/cancellations` (Treat "cancellation" as a resource created on the order) OR `PATCH /orders/123` with body `{"status": "cancelled"}`.

### 12.4. The "Chatty" API (N+1 Requests)
**Mistake:** Designing endpoints that require the client to make separate requests for related data, draining mobile battery and increasing latency.
*   **Scenario:**
    1.  `GET /feed` -> returns list of 20 post IDs.
    2.  Client loops and fires 20 requests to `GET /posts/{id}`.
    3.  Client loops again to `GET /users/{id}` for author info.
*   **Fix:**
    *   **Compound Documents:** Embed related resources. `GET /feed?embed=posts,posts.author`.
    *   **Partial Response:** Allow requesting specific fields.

### 12.5. Inconsistent Naming and Formatting
**Mistake:** Mixing conventions makes the API unpredictable and hard to consume.
*   **Bad:**
    *   `userId` (camelCase) in one endpoint, `user_id` (snake_case) in another.
    *   Returning boolean as `true` (JSON bool) in one place and `"true"` (string) in another.
*   **Fix:** Pick one convention (e.g., snake_case for JSON fields, camelCase for params) and stick to it strictly. Define a style guide.

### 12.6. Exposing Internal Implementation Details
**Mistake:** Returning raw database errors or internal identifiers that have no meaning to the client.
*   **Bad:** `{ "error": "SQLIntegrityConstraintViolationException: Duplicate entry 'alex'..." }`
*   **Fix:** Catch exceptions and return sanitized, helpful error messages. `{ "error": "Username 'alex' is already taken." }`

### 12.7. Missing Pagination on Collections
**Mistake:** Returning all records in a single response (`GET /users`).
*   **Impact:** If the database grows to 10,000 users, this request will time out or crash the mobile app with an OutOfMemory error.
*   **Fix:** Always implement pagination (Cursor or Offset) for any endpoint that returns a list.
