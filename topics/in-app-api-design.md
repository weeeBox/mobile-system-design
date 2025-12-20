# In-App API Design & Library Architecture

In mobile system design interviews, you may be asked to design a **client-side library** rather than a full app. Examples include:
*   "Design an Image Loading Library" (like Glide/Kingfisher)
*   "Design an Analytics SDK" (like Segment/Firebase)
*   "Design a Networking Library" (like Retrofit/Alamofire)

The interviewer is testing your **"API Ergonomics"**—can you design a tool that other developers will love to use and find difficult to break?

## 1. The Core Philosophy: "Easy to Learn, Hard to Misuse"

Joshua Bloch (author of *Effective Java*) defined the gold standard for API design. In an interview, explicitly mentioning these two goals provides a strong signal.

### Discoverability: Is it easy to use?
The "Hello World" of your library should be a single line of code.
*   **Zero Config:** Users shouldn't need to configure a cache size, thread pool, or timeout just to make a simple request. Smart defaults are key.
*   **Fluent Interfaces:** This guides the developer through the configuration process. IDE autocomplete becomes the documentation.

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
If a developer passes `null` or an invalid combination of parameters, what happens?
*   **Fail Fast:** Crash immediately (in debug mode) with a clear error message rather than failing silently or showing a blank UI.
*   **Type Safety:** Use `Enums` or `Sealed Classes` instead of Strings or Integers to restrict inputs to valid values.

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

## 3. Common Mistakes (Red Flags)

### The "God Config" Object
Passing a massive configuration object with 20 optional parameters into a constructor.
*   **Fix:** Use the **Builder Pattern**. It separates the construction of a complex object from its representation.

### Leaking Implementation Details
Exposing internal libraries (e.g., exposing `OkHttp` types in your custom networking library's public API).
*   **Why it's bad:** If you switch from `OkHttp` to `Cronet` later, you break every app using your library.
*   **Fix:** Wrap internal objects in your own Domain Models.

---

## 4. Things to Avoid

*   **Static Mutable State:** Avoid global variables that control state. It makes testing impossible (tests can't run in parallel) and causes race conditions. Use Dependency Injection instead.
*   **Swallowing Errors:** Never catch an exception and do nothing. At minimum, log it. Ideally, expose it to the caller.
*   **"Magic" Behavior:** Don't do unexpected things like automatically retrying 10 times without telling the developer, or implicitly initializing on the first call. Be explicit.

---

## 5. Real-World Case Studies (For "Signal")

Mentioning these libraries shows you study the ecosystem.

*   **Retrofit (Android):**
    *   *Why it's great:* It uses **Annotations** (`@GET`, `@POST`) to turn an API definition into a declarative Interface. It abstracts away the complexity of parsing and request creation.
*   **Alamofire (iOS):**
    *   *Why it's great:* It uses **Parameter Encoding** and **Response Validation** chaining.
    *   `AF.request(url).validate().responseJSON { ... }`
*   **Room (Android):**
    *   *Why it's great:* **Compile-time verification** of SQL queries. It catches "Safety" issues (syntax errors) before the app even runs.

## 6. Interview Checklist

1.  **Define the Interface:** Write the code you *wish* you could write as a developer using your library. (Read-Eval-Print Loop approach).
2.  **Choose the Scope:** Singleton (Global) vs. Instance (Scoped).
3.  **Handle Lifecycle:** How does the client lifecycle affect your API? (Auto-cancellation).
4.  **Mockability:** Can a user inject a test-double into your library for their own unit tests?