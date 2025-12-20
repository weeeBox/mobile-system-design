# In-App API Design & Library Architecture Deep Dive

In mobile system design interviews, you may be asked to design a **client-side library** rather than a full app. Examples include:
*   "Design an Image Loading Library" (like Glide/Kingfisher)
*   "Design an Analytics SDK" (like Segment/Firebase)
*   "Design a Networking Library" (like Retrofit/Alamofire)

The interviewer is testing your **"API Ergonomics"** - can you design a tool that other developers will love to use and find difficult to break?

## 1. The Core Philosophy: "Easy to Learn, Hard to Misuse"

Joshua Bloch (author of *Effective Java*) defined the gold standard for API design. In an interview, explicitly mentioning these two goals provides a strong signal.

### Discoverability: Is it easy to use?
The "Hello World" of your library should be achievable with a single line of code.
*   **Zero Configuration:** Users should not be required to manually define cache sizes, thread pools, or timeouts for standard requests. Sensible defaults are essential for a positive developer experience.
*   **Fluent Interfaces:** Chaining methods guides the developer through configuration, allowing IDE autocomplete to serve as a form of "living documentation."

**Example: Image Loading**
*   *Bad Design:*
    ```kotlin
    val loader = ImageLoader(context, cacheSize = 100, threadPool = Executors.newFixedThreadPool(4))
    loader.loadImage(url, imageView, null, null) // What are those nulls?
    ```
*   *Good Design (Glide/Picasso style):*
    ```kotlin
    Glide.with(context)
        .load(url)
        .placeholder(R.drawable.loading)
        .into(imageView)
    ```

### Safety: Is it hard to misuse?
If a developer provides invalid parameters, the library should handle the error gracefully or fail predictably.
*   **Fail Fast:** In development environments, the library should surface configuration errors immediately with descriptive messages rather than failing silently or causing inconsistent states.
*   **Type Safety:** Utilize `Enums` or `Sealed Classes` instead of raw Strings or Integers to restrict inputs to valid, compile-time checked values.

**Example: Network Methods**
*   *Bad Design:* `request("GET", url)` -> Developer can typo "get".
*   *Good Design (Alamofire/Retrofit):* `request(.get, url)` -> Compiler ensures validity.

---

## 2. Key Topics for the Interview

### 2.1 Error Reporting: How do you tell the developer something went wrong?

In mobile development, I/O operations (Network/Disk) fail frequently. How you expose these errors defines the quality of your API.

*   **Approach 1: Exceptions (Avoid for Async)**
    *   *Why to avoid:* In async callbacks, thrown exceptions often crash the app unexpectedly or get swallowed by the thread pool.
*   **Approach 2: Nullable Returns (Avoid)**
    *   *Why to avoid:* Returning `null` forces the developer to guess *why* it failed. Was it network? Parse error? Disk full?
*   **Approach 3: Callback Hell (Common but Dated)**
    *   `onSuccess(data)` / `onError(exception)`
*   **Approach 4: Result Types / Sealed Classes (Best Practice)**
    *   Forces the developer to handle both success and specific failure cases.

**Android Example (Kotlin Sealed Class):**
```kotlin
sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(val exception: Exception, val isTransient: Boolean) : NetworkResult<Nothing>()
}

// The compiler forces you to handle the Error case
when(result) {
    is Success -> showData(result.data)
    is Error -> if (result.isTransient) retry() else showError()
}
```

**iOS Example (Swift Result Type):**
```swift
func fetchUser(completion: (Result<User, NetworkError>) -> Void) { ... }

// Usage
switch result {
case .success(let user): display(user)
case .failure(let error): handle(error) // Error is strongly typed
}
```

### 2.2 Threading: The "Main Thread" Rule
Your library **must never** block the UI thread.
*   **Mistake:** Doing I/O in the constructor or `init` block.
*   **Best Practice:** All I/O should happen on a background dispatcher, and results should be delivered to the **Main Thread** (for UI libraries) or the caller's thread (for generic libraries).

---

## 3. Common Pitfalls ("Red Flags")

### The "God Config" Object
Passing a massive configuration object with 20 optional parameters into a constructor.
*   **Fix:** Use the **Builder Pattern**. It separates the construction of a complex object from its representation.

### Leaking Implementation Details
Exposing internal libraries (e.g., exposing `OkHttp` types in your custom networking library's public API).
*   **Why it's bad:** If you switch from `OkHttp` to `Cronet` later, you break every app using your library.
*   **Fix:** Wrap internal objects in your own Domain Models.

### Static Mutable State
Avoid global variables that control state. It makes testing impossible (tests can't run in parallel) and causes race conditions. Use Dependency Injection instead.

### Swallowing Errors
Avoid catching exceptions without appropriate handling. At a minimum, errors should be logged or surfaced to the caller to prevent silent failures in production.

### "Magic" Behavior
Avoid implementing hidden side effects, such as exhaustive automatic retry loops or implicit lazy initialization that may affect the caller's thread. These behaviors can mask underlying performance issues and lead to unpredictable execution states. Favor deterministic logic that requires explicit configuration for resource-intensive or state-altering operations.

---

## 4. Real-World Case Studies (For "Signal")

Mentioning these libraries shows you study the ecosystem.

*   **Retrofit (Android):** ([GitHub](https://github.com/square/retrofit))
    *   *Why it's great:* It uses **Annotations** (`@GET`, `@POST`) to turn an API definition into a declarative Interface. It abstracts away the complexity of parsing and request creation.
*   **Alamofire (iOS):** ([GitHub](https://github.com/Alamofire/Alamofire))
    *   *Why it's great:* It uses **Parameter Encoding** and **Response Validation** chaining.
    *   `AF.request(url).validate().responseJSON { ... }`
*   **Room (Android):** ([Documentation](https://developer.android.com/training/data-storage/room))
    *   *Why it's great:* **Compile-time verification** of SQL queries. It catches "Safety" issues (syntax errors) before the app even runs.

## 5. Summary Checklist for the Interview

1.  **Define the Interface:** Write the code you *wish* you could write as a developer using your library. (Read-Eval-Print Loop approach).
2.  **Choose the Scope:** Singleton (Global) vs. Instance (Scoped).
3.  **Handle Lifecycle:** How does the client lifecycle affect your API? (Auto-cancellation).
4.  **Mockability:** Can a user inject a test-double into your library for their own unit tests?