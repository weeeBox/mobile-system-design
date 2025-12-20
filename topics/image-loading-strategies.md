# Mobile System Design Interview: Image Loading Deepdive

Image loading is a classic system design topic. While "downloading a file" and "loading an image in a list" seem similar, the constraints are vastly different.

**Your Goal:** Demonstrate that you understand **memory management**, **scrolling performance**, and **caching hierarchies**.

## 1. Image Loading vs. File Downloading

Candidates often conflate these two. Clarify the distinction immediately.

*   **File Downloader (e.g., PDF, Zip):**
    *   **Goal:** Reliability & Integrity.
    *   **Constraint:** Completing the download even if the app backgrounds or the network flakes.
    *   **Storage:** Disk (permanent).
    *   **UI:** Often just a progress bar.

*   **Image Loader (e.g., Feed, Gallery):**
    *   **Goal:** Responsiveness & Frame Rate.
    *   **Constraint:** Decoding speed and RAM usage.
    *   **Storage:** Multi-tier Cache (RAM + Disk).
    *   **UI:** Bound to the UI lifecycle.

## 2. The "Build vs. Buy" Signal

The interviewer will likely ask: *"Would you build this from scratch?"*

*   **The Practical Answer:** "For a production app, I would almost always use an industry-standard library (Glide/Coil for Android, Kingfisher/Nuke for iOS)."
*   **The "Why":**
    *   **Edge Cases:** Libraries handle complex edge cases (Exif rotation, color spaces, progressive JPEGs) that are very expensive to implement.
    *   **Performance:** They have highly optimized caching engines.
    *   **Focus:** It allows the team to focus on *product features* rather than low-level infrastructure.

## 3. Best Practices

### 3.1 Caching Hierarchy (The "L1/L2" Strategy)

The most critical decision is **sizing** your caches. Too small = constant network calls. Too big = Out Of Memory (OOM) crashes.

#### Memory Cache (L1: Fast & Expensive)
*   **Content:** Decoded Bitmaps (Ready to draw).
*   **Sizing Strategy:** Do not use hardcoded constants (e.g., "50MB"). Use a percentage of the *available* application memory.
    *   **Rule of Thumb:** **15% - 25%** of available RAM.
    *   **Calculation:** `TargetMemoryCacheSize = (Screen Width * Screen Height * BytesPerPixel) * 4 Screens`. This ensures you can hold about 3-4 screens worth of images in memory.
*   **Library Defaults:**
    *   **Glide (Android):** Uses `MemorySizeCalculator`. defaults to **2 screens** worth of bitmaps + a buffer, capped at a % of the memory class.
    *   **Coil (Android):** Defaults to **25%** of the application's available memory.
    *   **Kingfisher (iOS):** Defaults to **25%** of the device's *total* memory (be careful with this on older devices).

#### Disk Cache (L2: Slow & Cheap)
*   **Content:** Encoded bytes (JPEG/PNG).
*   **Sizing Strategy:** Disk space is cheap, but not infinite.
    *   **Rule of Thumb:** **250MB - 1GB**.
*   **Library Defaults:**
    *   **Glide:** 250MB.
    *   **Coil:** 0.5% - 2% of the file system (min 10MB, max 250MB).
    *   **Kingfisher:** No default size limit (time-based expiration of 7 days).
    *   **SDWebImage:** Time-based expiration (7 days).

**The Signal:** You understand that "Available RAM" varies drastically between a low-end Android device and an iPhone 15 Pro, and your sizing logic adapts dynamically.

### 3.2 View Lifecycle Integration
*   **Problem:** User scrolls fast. The view for "Image A" is recycled and now needs to show "Image B".
*   **Solution:** **Request Cancellation**.
    *   Before starting a request for a View, cancel any pending request attached to that View (using Tags or View IDs).
    *   Prevents "Image A" from flashing onto the wrong cell 2 seconds later.

### 3.3 Sampling & Downsampling
*   **Problem:** Loading a 4K image (12MB decoded) into a 100x100 thumbnail view (40KB needed).
*   **Solution:** Decode bounds first (`inJustDecodeBounds` on Android), calculate sample size, and decode only the necessary pixels.
*   **Signal:** Shows you care about crashing the app with OOM (Out Of Memory) errors.

## 4. Common Pitfalls ("Red Flags")

*   **"I'll decode on the main thread."** -> Immediate fail. Blocks the UI, causing frame drops (jank).
*   **"I'll cache the raw response in memory."** -> Inefficient. You want to cache the *decoded* bitmap so you don't burn CPU decoding it every time the user scrolls back up.
*   **Ignoring Request Coalescing:** If 3 widgets request the same URL simultaneously, the system should make only *one* network call and fan out the result to all 3 listeners.
*   **Forgetting Priorities:** Thumbnails in the visible viewport should have higher priority than prefetching images for the next screen.

## 5. Summary Checklist for the Interview

1.  **Differentiate:** It's about RAM and FPS, not just downloading bytes.
2.  **Architecture:** Request Manager -> Memory Cache -> Disk Cache -> Network.
3.  **Optimization:** Downsampling, Request Coalescing, Cancellation on View Recycle.
4.  **Modern Approach:** Acknowledge libraries (Coil/Nuke) but explain the *concepts* they implement.
