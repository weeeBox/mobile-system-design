# Robust RESTful API Design Principles

Designing a robust RESTful API is critical for mobile applications, where network bandwidth, latency, and reliability are variable factors. A well-designed API ensures scalability, maintainability, and a smooth developer experience.

## 1. Resource-Oriented Design

REST APIs are based on resources, which represent entities in your system.

*   **Use Nouns, Not Verbs:** URIs should represent resources (nouns), not actions (verbs).
    *   **Good:** `GET /users`, `POST /orders`
    *   **Bad:** `GET /getUsers`, `POST /createOrder`
*   **Use Plural Nouns:** Keep resource names consistent, typically plural.
    *   `GET /users/123` (Read user 123)
    *   `GET /users` (List all users)
*   **Hierarchy and Relationships:** Use nesting to indicate relationships.
    *   `GET /users/123/orders` (Get orders for user 123)
    *   `GET /users/123/orders/456` (Get specific order 456 for user 123)
    *   *Tip:* Avoid deep nesting (more than 2 levels). Prefer flattening if possible: `GET /orders/456`.

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

Don't invent your own error codes. Use standard HTTP status codes to communicate the result of a request.

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

Keep the base resource URL clean and use query parameters for data manipulation.

*   **Filtering:** `GET /users?role=admin&active=true`
*   **Sorting:** `GET /users?sort=-created_at` (`-` for descending, `+` for ascending)
*   **Searching:** `GET /users?q=alex`
*   **Field Selection (Partial Response):** `GET /users?fields=id,name,email` (Great for mobile to reduce payload size).

## 6. Error Handling

Return consistent, structured error responses. This makes parsing errors on the client side predictable.

```json
{
  "error": {
    "code": "USER_ALREADY_EXISTS",
    "message": "A user with this email already exists.",
    "details": [
      {
        "field": "email",
        "issue": "duplicate"
      }
    ]
  }
}
```

## 7. Rate Limiting

Protect your backend from abuse and spikes in traffic. Return appropriate headers so the client knows their limits.

*   `X-RateLimit-Limit`: The number of allowed requests in the current period.
*   `X-RateLimit-Remaining`: The number of remaining requests in the current period.
*   `X-RateLimit-Reset`: The time at which the current period resets.
*   **Response:** Return `429 Too Many Requests` when the limit is exceeded.

## 8. Idempotency for Non-Idempotent Operations

Mobile networks are unreliable. A client might send a `POST` request (e.g., "Pay $10"), the server processes it, but the response is lost. The client retries, and the server shouldn't charge the user twice.

*   **Solution:** Use an `Idempotency-Key` header.
*   The client generates a unique key (UUID) for the request.
*   The server checks if it has already processed a request with that key.
    *   If **yes**: Return the stored response.
    *   If **no**: Process the request and store the key + response.

## 9. Date and Time

*   Always use **ISO 8601** strings for dates: `2023-10-27T10:00:00Z`.
*   Always store and return dates in **UTC**. Let the mobile client handle local time formatting.

## 10. Security Best Practices

*   **HTTPS:** Always use TLS/SSL. No exceptions.
*   **Authentication:** Use `Authorization: Bearer <token>` (e.g., JWT). Avoid passing tokens in URL parameters.
*   **Input Validation:** Validate all input on the server side. Never trust the client.
