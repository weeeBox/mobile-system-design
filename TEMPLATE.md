# Mobile System Design Interview Template

Use this template to structure your thoughts and notes during the interview.

## 1. Requirement Gathering (5 min)

**Goal:** Clarify scope and identify the "MVP".

### Questions to Ask:
-   **Users:** DAU/MAU? Spike traffic patterns?
-   **Geography:** Target market? (Network speed, data costs, device capabilities).
-   **Platform:** iOS, Android, or both? Minimum OS version?
-   **Team:** Team size? (Influences modularity).
-   **Features:** What are the top 3-5 critical features?

### Functional Requirements:
1.  
2.  
3.  

### Non-Functional Requirements:
-   **Offline Support:** (Yes/No/Partial)
-   **Performance:** (Latency, startup time)
-   **Security:** (E2E encryption, local storage encryption)
-   **Reliability:** (Data consistency, conflict resolution)

### Out of Scope:
-   (e.g., Auth, Settings, Analytics, Web version)

---

## 2. High-Level Diagram (10-15 min)

**Goal:** visual "Big Picture". Identify key components.

**Core Components:**
-   **UI Layer:** (View, ViewModel/Presenter)
-   **Navigation:** (Coordinator/Router)
-   **Domain Layer:** (Use Cases/Interactors)
-   **Data Layer:** (Repository)
    -   **Remote:** (API Service -> Retrofit/Alamofire)
    -   **Local:** (Persistence -> Room/CoreData)
-   **DI Graph:** (Dependency Injection)
-   **Image Loader:** (Coil/SDWebImage)

**Sketch:**
(Draw boxes and arrows here. Focus on data flow: UI -> ViewModel -> Repository -> API/DB)

---

## 3. Deep Dives (20-30 min)

**Select 1-2 topics to explore.**

### A. API Design
-   **Protocol:** REST vs GraphQL vs WebSocket vs gRPC.
-   **Pagination:** Offset vs Cursor vs PageNumber.
-   **Endpoints:**
    -   `GET /resource`
    -   `POST /resource`
-   **Payloads:** (JSON structure, field types)

### B. Data Storage
-   **Schema:** (Tables, Columns, Relationships)
-   **Sensitive Data:** (Keychain/Keystore)
-   **File Storage:** (Images/Videos -> Internal vs External cache)

### C. Synchronization (Offline Support)
-   **Strategy:** Write-Through vs Write-Back?
-   **Conflict Resolution:** Last-Write-Wins vs Server-Side vs User-Manual?
-   **Queue:** Request Queue / Worker Manager for background sync.

### D. Media Handling
-   **Images:** Caching (Memory/Disk), Resizing, Formats (WebP).
-   **Uploads:** Resumable/Chunked uploads?

---

## 4. Signal Checklist
-   [ ] Did I mention **Trade-offs**? (e.g., "WebSockets are faster but harder to scale than polling").
-   [ ] Did I mention **Edge Cases**? (e.g., "What if the network drops in the middle of an upload?").
-   [ ] Did I mention **Platform Specifics**? (e.g., "On Android, we need to handle process death").
-   [ ] Did I check assumptions with the interviewer?
