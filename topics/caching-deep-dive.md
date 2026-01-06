# Caching Deep Dive

Caching is a fundamental concept in system design, crucial for improving performance, reducing latency, and ensuring offline capability. In mobile development, it bridges the gap between the fast, reliable local environment and the slow, unreliable network.

## 1. Why Cache?
-   **Offline Support:** Allows users to interact with the app without an internet connection.
-   **Performance:** Loading data from local storage is orders of magnitude faster than network requests.
-   **Cost Efficiency:** Reduces data usage for the user and backend load for the server.
-   **Battery Life:** Radios are battery-hungry; fewer requests mean longer battery life.

## 2. Caching Layers

### Memory Cache (L1)
The fastest access, but volatile. Data is lost when the app is killed.
-   **Use Case:** Bitmaps (Images) currently being displayed, frequently accessed small objects (Configuration).
-   **Implementation:** `LruCache` (Android), `NSCache` (iOS).
-   **Constraints:** Strictly limited by available RAM. Must handle low-memory warnings to avoid OOM (Out Of Memory) crashes.

### Disk Cache (L2)
Slower than memory, but persistent. Data survives app restarts.
-   **Use Case:** JSON responses, large images, video chunks, user profile data.
-   **Implementation:**
    -   **Structured:** SQLite (Room), Core Data, Realm. Good for complex queries and relationships.
    -   **Unstructured/Key-Value:** DiskLruCache, DataStore (Android), simple File I/O. Good for raw blobs or simple preferences.
    -   **HTTP Cache:** Standard HTTP caching handled by networking clients (OkHttp, URLSession).

## 3. Caching Strategies

### Cache-Aside (Lazy Loading)
The application code is responsible for loading data.
1.  App asks Cache for data.
2.  **Hit:** Cache returns data.
3.  **Miss:** App fetches data from API, writes to Cache, then returns data.

### Read-Through / Write-Through
The application treats the cache as the main data source. The cache library handles the fetching logic.
-   **Repositories:** In mobile architecture, the Repository pattern often acts as this layer.

### Stale-While-Revalidate
Return stale data immediately from the cache (for UI responsiveness) while simultaneously fetching fresh data from the network in the background to update the view.

## 4. Cache Eviction Policies
Since storage is finite, we must decide what to delete when full.
-   **LRU (Least Recently Used):** Discards the least recently used items first. Most common for images and general data.
-   **FIFO (First In First Out):** Discards the oldest items first, regardless of usage.
-   **TTL (Time To Live):** Data expires after a specific time period (e.g., 5 minutes). Critical for volatile data like stock prices or news feeds.

## 5. HTTP Caching
Leverage standard headers to let the server control caching behavior.
-   **`Cache-Control`:** `max-age=3600` (valid for 1 hour), `no-cache` (must revalidate), `no-store` (never cache).
-   **`ETag` (Entity Tag):** A fingerprint of the resource. Client sends `If-None-Match: "tag"`. Server returns `304 Not Modified` (empty body) if data hasn't changed, saving bandwidth.
-   **`Last-Modified`:** Client sends `If-Modified-Since`. Similar to ETag but time-based.

## 6. Security Considerations
-   **Sensitive Data:** Never cache PII (Personally Identifiable Information), auth tokens, or payment details in plain text on disk.
-   **Encryption:** Use `EncryptedSharedPreferences` / `Jetpack Security` (Android) or `Keychain` / `File Protection` (iOS).
-   **Snapshots:** Be aware that the OS might take snapshots of the UI (which includes cached data) for the app switcher. Mask sensitive fields.

## 7. Common Pitfalls
-   **Over-Caching:** Caching everything fills up disk space and complicates debugging.
-   **Invalidation Logic:** The hardest problem in computer science. How do you know when to clear the cache? (User logout, explicit pull-to-refresh, push notification triggers).
-   **Race Conditions:** Reading/Writing to cache from multiple threads.
