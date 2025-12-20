# RESTful API Design Deep Dive

This cheatsheet provides a concise list of design principles and mobile-specific optimizations for building robust RESTful APIs. In a system design interview, demonstrating these patterns shows your ability to build scalable systems that thrive under mobile constraints (high latency, limited bandwidth, and flaky reliability).

**Your Goal:** Proactively suggest these optimizations to show you prioritize the mobile user experience.

## 1. Resource-Oriented Design

REST APIs are based on resources (nouns), not actions (verbs).

*   **Do Use Nouns, Not Verbs:** URIs should represent entities.
    *   **Good:** `GET /users`, `POST /orders`
    *   **Bad:** `GET /getUsers`, `POST /createOrder`
*   **Do Use Plural Nouns:** Keep resource names consistent, typically plural.
    *   `GET /users/123` (Read user 123)
    *   `GET /users` (List all users)
*   **Do Use Hierarchy:** Use nesting to indicate relationships, but avoid deep nesting.
    *   **Good:** `GET /users/123/orders`
    *   **Avoid:** `GET /users/123/orders/456/items/789` (Too deep; prefer `GET /orders/456/items`).
*   **The Signal:** Shows you understand standard REST architectural patterns and can design intuitive, predictable interfaces.

## 2. HTTP Semantics (The Protocol Layer)

Use standard HTTP methods and status codes for their intended purpose.

### 2.1 HTTP Methods
| Method | Action | Safety | Idempotency | Description |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | Read | Safe | Yes | Retrieve data. Should never modify state. |
| **POST** | Create | Unsafe | No | Create a new resource. Not idempotent by default. |
| **PUT** | Replace | Unsafe | Yes | Replace a resource entirely. |
| **PATCH** | Update | Unsafe | No* | Partial update. (*Can be designed to be idempotent). |
| **DELETE** | Delete | Unsafe | Yes | Remove a resource. |

### 2.2 Standard HTTP Status Codes
**Avoid** inventing custom error codes. **Do** use standard status codes to control client flow.

*   **2xx (Success):** `200 OK`, `201 Created` (for POST), `204 No Content` (for DELETE).
*   **3xx (Redirection):** `301 Moved Permanently`, `304 Not Modified` (Critical for mobile caching).
*   **4xx (Client Error):** `400 Bad Request`, `401 Unauthorized` (auth missing), `403 Forbidden` (auth present but insufficient), `404 Not Found`, `429 Too Many Requests`.
*   **5xx (Server Error):** `500 Internal Server Error`, `503 Service Unavailable`.

*   **The Signal:** Demonstrates you know how to leverage the HTTP protocol correctly, which is crucial for caching and intermediate proxies.

## 3. Mobile-Specific Optimizations (The "Mobile" Signal)

Demonstrate your mobile expertise by explicitly optimizing for the realities of cellular networks: limited bandwidth, high latency, and battery constraints.

### 3.1 Versioning
APIs evolve, but mobile apps are not updated instantly.
*   **Strategy:** URI Versioning (`GET /v1/users`) is recommended for mobile.
*   **Pros:** Explicit, easy to test, easy to cache.
*   **Cons:** "Pollutes" the URI.
*   **The Signal:** You understand that old app versions will exist in the wild for years and must continue to function.

### 3.2 Resource Management (Over-fetching vs. Under-fetching)
Finding the balance between "one request for everything" and "many small requests" is key.

*   **Over-fetching:** Getting too much data.
    *   *Solution:* **Field Selection**. Allow clients to specify exactly what they need: `GET /users/123?fields=id,name,avatar_url`.
*   **Under-fetching:** Getting too little data, requiring subsequent requests (N+1 problem).
    *   *Solution:* **Resource Embedding**. Allow clients to include related resources: `GET /orders/123?embed=items,payment_details`.
*   **The "Backend for Frontend" (BFF) Pattern:** Ideally, mention creating a dedicated API layer specifically for the mobile app that aggregates data from multiple microservices into a single, mobile-optimized response.

### 3.3 Filtering, Sorting, and Searching
Keep the base resource URL clean and use query parameters.
*   **Filtering:** `GET /users?role=admin`
*   **Sorting:** `GET /users?sort=-created_at` (`-` for descending)
*   **Searching:** `GET /users?q=alex`

## 4. Reliability & Resilience

### 4.1 Idempotency for Non-Idempotent Operations
Mobile networks are flaky. A `POST` request (e.g., "Pay $10") might time out. The user retries.
*   **How to Generate Keys:** The client should generate a unique key (typically a UUID v4) for every critical mutation request.
*   **Where to Include:** Pass this key in a custom HTTP header, e.g., `Idempotency-Key` or `X-Request-ID`.
*   **Server Logic:**
    1.  Check shared cache/DB for the key.
    2.  If **Seen**: Return the cached response.
    3.  If **New**: Execute logic and store result.
*   **The Signal:** You know how to build resilient systems that handle network failures gracefully without duplicating transactions.

### 4.2 Rate Limiting
Protect your backend from abuse and spikes.
*   **Response Headers:** Return `X-RateLimit-Remaining` and `X-RateLimit-Reset` so the app can back off intelligently.
*   **Status Code:** Return `429 Too Many Requests`.

### 4.3 Date and Time
*   **Do** always use **ISO 8601** strings: `2023-10-27T10:00:00Z`.
*   **Do** always store and return dates in **UTC**. Let the mobile client handle local time formatting.

## 5. Security & Privacy

*   **HTTPS:** Mandatory. No exceptions.
*   **Authentication:** Use `Authorization: Bearer <token>` (e.g., JWT). Avoid passing tokens in URL parameters.
*   **Input Validation:** Validate all input on the server side. Never trust the client.

### 5.1 API Key Placement
When using API keys to identify the client application:
*   **Do Use Headers:** `X-API-Key`. Headers are less likely to be inadvertently logged than URLs.
*   **Avoid Query Parameters:** `/resource?api_key=xyz`. URLs are logged in cleartext by proxies.
*   **Mobile Reality:** Treat any key embedded in the app as **public**. Use them for rate limiting, not sensitive permissions.

### 5.2 Protecting Against Scraping
*   **Rate Limiting:** Aggressive limiting per IP/User.
*   **App Attestation:** Use Google Play Integrity / Apple App Attest to ensure requests come from your genuine app binary.
*   **WAF:** Use a Web Application Firewall to block suspicious traffic patterns.

## 6. Error Handling

Return consistent, structured errors.

```json
{
  "error": {
    "code": "USER_ALREADY_EXISTS", // Machine-readable
    "message": "A user with this email already exists.", // Human-readable
    "details": [
      { "field": "email", "issue": "duplicate" }
    ]
  }
}
```
*   **The Signal:** You prioritize the developer experience (DX). Structured errors allow the mobile app to parse specific issues and show localized, helpful UI messages.

## 7. Common Pitfalls ("Red Flags")

*   **Tunneling:** Sending everything via `POST`. (Breaks caching).
*   **Returning 200 OK for Errors:** Breaks HTTP-aware clients and monitoring tools. Use 4xx/5xx.
*   **Verbs in URIs:** `POST /createOrder`. (Use `POST /orders`).
*   **Chatty APIs:** Designing endpoints that require N+1 requests. Fix with Compound Documents.
*   **Inconsistent Naming:** Mixing `camelCase` and `snake_case`. Pick one.
*   **Leaking Internals:** Returning raw SQL exceptions to the client.
*   **Missing Pagination:** Returning all records in a single response (`GET /users`). Always paginate lists.

## 8. Real-World Case Studies

*   **Stripe (Idempotency):** Uses `Idempotency-Key` header to safely retry payment creation requests. ([Source](https://stripe.com/docs/api/idempotent_requests))
    *   *Signal:* Mentioning this shows you understand financial-grade reliability.
*   **Twitter (Versioning):** Uses distinct API versions (`/1.1/`, `/2/`) to allow radical architectural changes (like moving from Monolith to Microservices) without breaking clients. ([Source](https://developer.twitter.com/en/docs/twitter-api/versioning))
*   **GraphQL (Facebook):** While REST is standard, mentioning GraphQL for "Mobile-Specific Data Fetching" (solving over-fetching) shows breadth of knowledge. ([Source](https://graphql.org/learn/))

## 9. Summary Checklist for the Interview

1.  **Define Resources:** Identify the nouns (Users, Tweets).
2.  **Select Methods:** Map actions to HTTP verbs (GET, POST).
3.  **Optimize for Mobile:** Add Versioning, Pagination, and Partial Response (BFF).
4.  **Handle Failure:** Define Error formats and Idempotency keys.
5.  **Secure It:** HTTPS, Auth headers, Rate Limiting, and Attestation.
