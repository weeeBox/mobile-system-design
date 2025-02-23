# A Simple Framework For Mobile System Design Interviews (iOS & Android)

This guide provides a framework for approaching mobile system design interviews, specifically for iOS and Android roles. The core idea is that interviewers focus on assessing your thought process and communication skills rather than expecting a perfect, production-ready solution within a limited time. By showcasing your strengths, you enhance your overall evaluation as a candidate.

Join the ["Mobile System Design"](https://discord.gg/AWDNrvqban) Discord server for general discussions and feedback.

<div align="center">
  
[<img height="60" alt="discord" src="https://user-images.githubusercontent.com/786644/91370247-cf4cea80-e7db-11ea-8330-dc29c0fe8969.png">](https://discord.gg/AWDNrvqban)
  
</div>

## Disclaimer
This framework is inspired by "Scalable Backend Design" resources. Understanding this guide doesn't guarantee success. Interview structure varies based on the interviewer's style. Active dialog is vital – ask clarifying questions and avoid assumptions.

**This guide does not reflect the interviewing policies from Google (or any other company).**

## Interview Process (45–60 min)
- 2–5 min: Introductions
- 5 min: Task definition and requirement gathering
- 10 min: High-level discussion
- 20-30 min: Detailed discussion
- 5 min: Questions for the interviewer

## Introductions
Briefly share your relevant experience. Example: _"Hi, I'm Alex. I've been developing for iOS/Android since 2010, focusing on frameworks and libraries. For the last 2.5 years, I led a team on the XYZ project, covering system design, planning, and technical mentorship."_ The purpose is to break the ice and highlight relevant skills.

## Defining The Task
The interviewer presents the task, like "Design Twitter Feed." Determine the task's scope:

- **Client-side Only:** Design the app, given a backend and API.
- **Client-side + API:** Design both the client app and API. (Most Common).
- **Client-side + API + Backend:** Design the app, API, and backend. (Less Likely). Be transparent if your backend experience is limited. Focus on your mobile expertise and mention your high-level understanding of backend concepts from books, videos, etc.

## Gathering Requirements
Categorize requirements into Functional, Non-Functional, and Out-of-Scope. Using "Design Twitter Feed" as an example:

### Functional Requirements
Focus on high-value features (3-5).
- Users can scroll through an infinite feed of tweets.
- Users can "like" a tweet.
- Users can view comments for a specific tweet (read-only).

### Non-Functional Requirements
These are critical for the product's overall success.
- Offline support: Users should be able to view cached tweets and interact with them even without internet connectivity.
- Real-time notifications: Notify users of new tweets, likes, and comments.
- Optimal resource usage: Minimize bandwidth, CPU, and battery consumption. Consider network optimization techniques and efficient data handling.

### Out of Scope
Exclude features important for a real project but not in the interview scope.
- Login/Authentication
- Sending tweets
- Following/Retweeting
- Analytics

### Providing the "Signal"
The interviewer evaluates your thought process. Show them:
- Your assumptions and how you stated them clearly.
- The rationale behind your feature choices.
- Relevant clarifying questions.
- Awareness of trade-offs and potential concerns.

Ask many questions and cover diverse aspects. Use a Breadth-First Search (BFS) approach, then ask the interviewer which area they want to explore in-depth. If they offer a choice, select a topic you know well.

## Clarifying Questions
Ask these questions during the task clarification step:

- **Do we need to support emerging markets?**
    - Consider app size optimization due to low-end devices and expensive data. Prioritize caching and minimize network requests.  On Android, look into using app bundles, on iOS investigate app thinning.
- **What number of users do we expect?**
    -  High user count impacts backend load. Discuss Exponential Backoff and API Rate Limiting to prevent unintended DDoS.
- **How big is the engineering team?**
    - Smaller teams (2-4) require different considerations than larger teams (20-100). Focus on modularity and project structure to enable parallel development. Think about using a monorepo versus a multi-repo strategy.
- **What are the target OS versions and device types?**
    - This will drive decisions about architectural patterns (e.g. Compose/SwiftUI vs. older view-based systems), available APIs (e.g. for background tasks, data persistence), and necessary compatibility layers.
- **Are there any specific performance benchmarks we need to achieve?**
    -  Understand acceptable latency for feed updates, image loading, and other key interactions.  This will influence caching strategies, network optimization, and data serialization techniques.
- **What level of data consistency is required?**
    -  Can we tolerate eventual consistency for likes and comments, or do we need strong consistency? This impacts decisions about offline behavior and data synchronization strategies.
- **How frequently is the data updated?**
    -  If tweets are very frequent, then a pull-based refresh strategy might overwhelm the server. Push notifications or SSE might be more appropriate.
- **What are the expected types of media?**
    -  Image sizes and formats will influence caching and networking strategies. Consider supporting modern formats such as webp or AVIF. Video will require different considerations, such as streaming protocols (HLS, DASH), transcoding, and CDN support.

## High-Level Diagram
If the interviewer is satisfied, ask if they want to see a high-level diagram.

![High-level Diagram](/images/twitter-feed-high-level-diagram.svg)

### Server-side Components:
- **Backend:** The overall server infrastructure.  Focus on client-side concerns.
- **Push Provider:** Infrastructure for mobile push notifications. Delivers payloads from the backend.
- **CDN (Content Delivery Network):** Delivers static content (images, videos) to clients efficiently.

### Client-side Components:
- **API Service:** Abstract client-server communication. Provides a clean interface for interacting with the backend. Consider using a library like Retrofit (Android) or Alamofire (iOS).
- **Persistence:**  The single source of truth for data on the device. Data is persisted locally first and then propagated to other components.  This enables offline access and reduces network dependency.
- **Repository:** Mediator between API Service and Persistence.  Decouples data sources from the UI.
- **Tweet Feed Flow:** Components responsible for displaying the scrollable tweet list.
- **Tweet Details Flow:** Components responsible for displaying a single tweet's details.
- **DI Graph:** Dependency injection graph. Enables loose coupling and testability (Dagger/Hilt on Android, Swinject/Typhoon on iOS).
- **Image Loader:** Loads and caches images (Glide/Coil on Android, SDWebImage/Kingfisher on iOS).
- **Coordinator:** Manages navigation and flow logic between Tweet Feed and Tweet Details, decoupling the components.
- **App Module:**  Executable part of the system that "glues" components together.  Responsible for application lifecycle management.
- **Analytics Service:** A module responsible for collecting and transmitting analytics data, such as user actions and performance metrics.

### Providing the "Signal"
The interviewer looks for:
- Ability to present the "big picture" without unnecessary details.
- Identification of major building blocks and their interactions.
- Consideration of app modularity and team collaboration.

### Frequently Asked Questions
#### Why is a high-level diagram necessary? Can I skip it?
Not mandatory, but it has advantages:
- **Time management:** Quickly outlines the system and raises discussion points.
- **Modularity:** Each component can be a separate module for team collaboration.
- **Best practices:** Mirrors backend system design and the C4 model for visualizing architecture ([https://c4model.com/](https://c4model.com/)).

_Still not sure?_ Ask the interviewer if they want a diagram or prefer to jump into specific components.

## Deep Dive: Tweet Feed Flow
The interviewer might focus on a specific component, like "Tweet Feed Flow". Discuss:

- **Architecture patterns:** MVP, MVVM, MVI, Clean Architecture, Redux. MVC is generally discouraged. Choose a well-known pattern for easier onboarding.
- **Pagination:** Essential for infinite scrolling. (See Pagination section).
- **Dependency injection:** Improves module isolation and testability.
- **Image Loading:** Discuss low-res placeholders, full-res loading, and scrolling performance.

![Details Diagram](/images/tweet-details.svg)
### Components
- **Feed API Service:** Abstracts the Twitter Feed API client, fetching paginated data. Injected via DI.
- **Feed Persistence:** Abstracts cached paginated data storage. Injected via DI.
- **Remote Mediator:** Fetches next/previous page data and saves it to the persistence layer.
- **Feed Repository:** Consolidates remote and cached responses into a `Pager` object using the `Remote Mediator`.
- **Pager:**  Triggers data fetching from the Remote Mediator and exposes an observable stream of paged data to the UI.
- **"Tweet Like" and "Tweet Details" use cases:**  Implement "Like" and "Show Details" operations.  Injected via DI.
- **Image Loader:** Abstracts image loading from the image loading library.  Injected via DI.

### Providing the "Signal"
The interviewer assesses:
- Familiarity with common MVx patterns.
- Separation of business logic and UI.
- Understanding of dependency injection.
- Ability to design self-contained modules.

### Frequently Asked Questions
#### How much detail should I provide?
No strict rule. Work with the interviewer; ask if they want more detail or to move on. Observe their body language. If they interrupt, stop and ask if they have questions. Collaboration is key.

#### Why no mention of specific classes (`RecyclerView`/`UICollectionView`) or vendors (Room, CoreData, Realm)?
- To keep the guide platform-agnostic. Libraries evolve rapidly. Abstracting functionality is more robust.
- Vendor selection is subjective and based on experience/trends.
- Large companies often build custom stacks.
- Implementation details are widely available online.

#### What drawing tool should I use?
[Excalidraw](https://excalidraw.com/), [Google Jamboard](https://jamboard.google.com/), and [Google Drawings](https://docs.google.com/drawings) are popular. Some prefer collaborative editors and verbal discussion. Privacy policies may limit tool choices.

## API Design
Cover as much ground as possible. Ask if the interviewer has preferences, or choose an area you know well.

### Real-time Notifications
Discuss approaches for real-time updates:

- **Push Notifications:**
  - Pros:
    - **Easier to implement:** Requires less client-side and server-side engineering effort compared to persistent connections.
    - **Can wake the app in the background:** Allows delivery of updates even when the app is not actively running (within OS limitations).
    - **Battery efficient (when used correctly):** The OS manages the connection and delivery, optimizing for battery life.
  - Cons:
    - **Not 100% reliable:** Delivery is not guaranteed due to network conditions, device state, or platform limitations (e.g., Doze mode on Android).
    - **May have delays:** Delivery latency can vary depending on network conditions and the push notification provider.
    - **Relies on 3rd-party service:** Dependent on Apple's APNs (iOS) or Google's FCM (Android), introducing a point of failure and potential vendor lock-in.
    - **Users can opt-out:** Users can disable push notifications at the OS level or within the app.
    - **Potential for abuse (spam):** Can be easily misused to send irrelevant or excessive notifications, leading to user frustration and app uninstalls.
- **HTTP-polling:**
  - Periodic requests for updates. Can create excessive traffic and backend load.
    - **Short polling:** Fixed time interval.
      - Pros:
        - **Simple to implement:** Relatively straightforward client-side and server-side logic.
        - **No persistent connection:** Avoids the complexities of managing and maintaining persistent connections.
        - **Compatible with firewalls and proxies:** Less likely to be blocked by restrictive network environments.
        - **Easy to scale horizontally:** The stateless nature of HTTP makes it easier to distribute load across multiple servers.
        - **Less expensive (if infrequent):** Lower server resource utilization if the polling interval is large.
        - **Can be useful for batch updates:** Polling can be used to retrieve multiple updates in a single request.
      - Cons:
        - **Potential for significant delays:** Notification delivery is limited by the polling interval.
        - **Overhead from TLS handshake and HTTP headers:** Repeated connections introduce significant overhead, especially for small updates.
        - **Inefficient bandwidth usage:** The client sends requests even when there are no updates available.
        - **High server load:** Frequent polling can strain server resources, especially with a large number of clients.
    - **Long polling:**
      - Pros:
        - **Instant notifications (almost):** Server holds the connection open until an update is available, minimizing latency.
      - Cons:
        - **More complex, requires more server resources:** Requires more sophisticated server-side logic to manage connections and track updates.
        - **Keeps a persistent connection (sort of):** Although not a *true* persistent connection like WebSockets, it simulates one, and managing timeouts is critical.
        - **Still introduces some latency:** Dependent on server response time even with updates readily available.
        - **Susceptible to connection timeouts and interruptions:** Requires robust error handling and reconnection logic.
        - **Can be more complex to scale:** Requires sticky sessions or other mechanisms to ensure that the client connects to the correct server instance.
- **Server-Sent Events (SSE):**
  - Stream events over HTTP without polling.
    - Pros:
      - **Real-time updates with a single connection:** Establishes a unidirectional (server-to-client) stream for efficient delivery of updates.
      - **Simpler than WebSockets:** Easier to implement and manage compared to bidirectional communication protocols.
      - **Standard protocol:** Leverages standard HTTP protocol, making it easier to integrate with existing infrastructure.
      - **Lower overhead than long polling:** Avoids the overhead of repeatedly establishing and tearing down connections.
      - **Text-based protocol:** Easier to debug and inspect compared to binary protocols.
    - Cons:
      - **Keeps a persistent connection:** Requires server resources to maintain the connection.
      - **Unidirectional:** Only supports server-to-client communication, making it unsuitable for applications that require frequent client-to-server updates.
      - **Limited browser support (older browsers):** May require polyfills for older browsers.
      - **Connection limits:** May be subject to connection limits imposed by browsers or proxies.
- **WebSockets:**
  - Bi-directional communication.
    - Pros:
      - **Can transmit binary and text data:** Supports both text-based and binary data, enabling a wide range of applications.
      - **Full-duplex communication:** Enables real-time, bidirectional communication between the client and the server.
      - **Lower latency than HTTP polling:** Avoids the overhead of repeatedly establishing and tearing down connections.
      - **Efficient use of bandwidth:** Reduces bandwidth consumption by maintaining a persistent connection.
    - Cons:
      - **More complex to set up:** Requires more complex server-side and client-side logic compared to HTTP-based protocols.
      - **Keeps a persistent connection:** Requires server resources to maintain the connection.
      - **More complex to scale:** Requires specialized infrastructure and techniques to handle a large number of concurrent connections.
      - **Firewall issues:** May be blocked by firewalls or proxies that do not support WebSockets.
      - **Increased security concerns:** Requires careful attention to security to prevent attacks such as cross-site scripting (XSS) and denial-of-service (DoS).
      - **Higher battery consumption:** Maintaining a persistent connection can drain battery life on mobile devices.
      - **Harder to debug compared to HTTP:** Binary messages can be difficult to inspect.

Choose an appropriate approach for the task. For "Design Twitter Feed", a combination of SSE (primary channel for "like" updates and potentially new tweet notifications if scale allows) and Push Notifications (for clients without active connections or less critical updates) could be suitable. Consider WebSockets for features like real-time chat or live video streaming (if in scope).

### Protocols

#### REST (Representational State Transfer)
Text-based, stateless protocol for CRUD (Create, Read, Update, Delete) operations.  Relies heavily on HTTP methods (GET, POST, PUT, DELETE).

- Pros:
  - **Easy to learn, understand, and implement:** Widely adopted and well-documented, with numerous libraries and frameworks available.
  - **Easy to cache using HTTP caching:** Leverages built-in HTTP caching mechanisms (e.g., `Cache-Control` headers) for improved performance. CDNs are very effective with RESTful APIs.
  - **Loose coupling:** Client and server are independent and can evolve separately as long as the API contract is maintained.
  - **Stateless:** Each request contains all the information needed for the server to understand and process it, simplifying server-side logic and scalability.
  - **Widely supported:** Supported by virtually all clients (browsers, mobile apps, etc.) and servers.
  - **Human-readable:** Text-based messages are easy to debug and inspect.

- Cons:
  - **Less efficient on mobile due to separate connections:** Each request requires a new TCP connection (although HTTP/2 mitigates this somewhat with connection multiplexing). Still can contribute to battery drain.
  - **Schemaless:** Difficult to validate data integrity and format on the client without custom validation logic or external schema definitions (e.g., OpenAPI/Swagger).  This also makes refactoring more risky.
  - **Stateless:** Requires extra functionality to maintain session state, such as cookies or tokens, adding complexity to authentication and authorization.
  - **Overhead from metadata and headers:** Each request contains contextual metadata and headers, which can add significant overhead, especially for small payloads.
  - **Over-fetching/Under-fetching:** Clients often receive more data than they need (over-fetching) or need to make multiple requests to retrieve all the required data (under-fetching).

#### GraphQL
A query language for your API and a server-side runtime for executing those queries. Allows clients to request specific data from multiple resources using a single endpoint.

- Pros:
  - **Schema-based, typed queries:** Clients can verify data integrity and format against a well-defined schema, improving developer experience and reducing errors. Enables introspection of the API.
  - **Highly customizable:** Clients can request only the specific data they need, reducing the amount of HTTP traffic and improving performance.  Eliminates over-fetching.
  - **Bi-directional communication with GraphQL Subscriptions (WebSocket-based):** Supports real-time updates via WebSockets, enabling features like live feeds and notifications.
  - **Strong typing:** Allows for compile-time checks, reducing runtime errors.
  - **Versioning is less critical:** Because clients only request specific data, backend changes are less likely to break existing clients.
  - **Developer tools:** Excellent tooling for development, including GraphiQL for exploring and testing queries.

- Cons:
  - **More complex backend:** Requires a more sophisticated server-side implementation compared to REST, including a GraphQL server and resolvers.
  - **"Leaky abstraction" – tight client-backend coupling:** Clients become tightly coupled to the backend schema, making it harder to evolve the API without breaking existing clients. Can complicate refactoring.
  - **Query performance depends on the slowest service (for federated data):** The performance of a query is bound to the performance of the slowest service on the backend, especially in a federated architecture where data is spread across multiple services. N+1 problem can occur without careful consideration.
  - **Complexity with caching:** HTTP caching is more difficult to implement compared to REST, requiring specialized solutions.
  - **Security considerations:** Requires careful attention to security to prevent malicious queries and unauthorized access.

#### WebSocket
Full-duplex communication over a single TCP connection.

- Pros:
  - **Real-time, bi-directional communication:** Enables continuous communication between the client and the server, ideal for applications that require low-latency updates.
  - **Supports text and binary data:** Can transmit both text-based and binary data, making it versatile for various use cases.
  - **Low latency:** Reduces latency by maintaining a persistent connection, avoiding the overhead of repeatedly establishing and tearing down connections.
  - **Efficient bandwidth usage:** Reduces bandwidth consumption by only sending data when there are updates.

- Cons:
  - **Requires maintaining an active connection:** Requires server resources to maintain the connection, potentially leading to scalability challenges.
  - **Schemaless:** Difficult to validate data integrity and format on the client without custom validation logic. This can also make debugging more difficult.
  - **Limited number of active connections per server (65k limitation is theoretical and OS-dependent):**  The actual limit depends on the server operating system and hardware, but managing a very high number of concurrent connections can be challenging. Connection pooling and proper resource allocation are crucial.
  - **More complex to implement than HTTP polling or SSE:** Requires more sophisticated client-side and server-side logic.
  - **Stateful:** The server needs to maintain state about each client connection, which can complicate scaling and fault tolerance.

Learn more:
- [WebSocket Tutorial - How WebSockets Work](https://www.youtube.com/watch?v=pNxK8fPKstc)

#### gRPC (gRPC Remote Procedure Calls)
A modern open source high performance Remote Procedure Call (RPC) framework that runs on top of HTTP/2. Supports bi-directional streaming using a single physical connection.

- Pros:
  - **Lightweight binary messages (smaller than text-based):** Uses Protocol Buffers (Protobuf) for message serialization, resulting in smaller message sizes and improved performance.
  - **Schema-based:** Built-in code generation with Protobuf, ensuring type safety and reducing errors.
  - **Supports event-driven architecture:** Provides support for server-side streaming, client-side streaming, and bi-directional streaming, enabling a wide range of use cases.
  - **Multiple parallel requests:** Leverages HTTP/2 multiplexing to send multiple requests over a single connection.
  - **High performance:** Optimized for low latency and high throughput.
  - **Strong typing:** Provides strong typing for both requests and responses, reducing runtime errors.
  - **Good performance on unreliable networks:** Protocol Buffers are designed to be resilient to data corruption and network disruptions.

- Cons:
  - **Limited browser support:** Requires gRPC-Web for browser clients, which adds complexity. While support has improved, it's still not as seamless as REST or GraphQL.
  - **Non-human-readable format:** Binary messages are difficult to debug and inspect without specialized tools.
  - **Steeper learning curve:** Requires familiarity with Protocol Buffers and gRPC concepts.
  - **Increased complexity compared to REST:** More complex to set up and configure compared to REST.
  - **Heavier on client:** The client library required for gRPC is generally larger compared to REST, which may impact app size.
  - **Not as widely understood as REST:** REST is much more widely understood, and developers are more likely to be familiar with it.

#### MQTT (Message Queuing Telemetry Transport)
A lightweight, publish-subscribe network protocol that transports messages between devices. Often used for IoT applications but can be suitable for mobile in specific scenarios.

- Pros:
    - **Lightweight:** Minimal bandwidth usage and low overhead, ideal for constrained networks.
    - **Publish-Subscribe:** Decouples message producers and consumers.
    - **Real-time communication:** Low latency and quick message delivery.
    - **QoS Levels:** Provides different Quality of Service (QoS) levels to ensure message delivery based on importance (at most once, at least once, exactly once).

- Cons:
    - **Binary protocol:** Difficult to debug and inspect.
    - **Not as widely used as HTTP-based protocols:** Less mature ecosystem compared to REST, GraphQL, and WebSockets.
    - **Stateful:** Requires managing client connections and subscriptions.
    - **Security Considerations:** Requires careful configuration to secure the MQTT broker and client connections.
    - **Broker Dependency:** Relies on a central MQTT broker for message routing, which can be a single point of failure.

Select an appropriate protocol. REST is suitable for "Design Twitter Feed" due to its simplicity for many of the initial tasks, though consider GraphQL for more complex data fetching requirements in the future. For truly real-time scenarios (if in scope), explore WebSockets or SSE. gRPC may be overkill for the initial feature set but beneficial if significant performance improvements are needed with more complex features. Consider MQTT only if you specifically need to interact with IoT devices or have very constrained network conditions.

### Pagination
Essential for endpoints returning lists. Prevents excessive network and memory usage.

#### Types of Pagination

- **Offset Pagination (also known as Limit-Offset Pagination):**
  - `limit` and `offset` parameters: `GET /feed?offset=100&limit=20`. The `offset` specifies the index from which to start returning results, and the `limit` specifies the maximum number of results to return.
  - Pros:
    - **Easy to implement:** Parameters can be passed directly to a SQL query (e.g., `SELECT * FROM tweets LIMIT 20 OFFSET 100`).
    - **Stateless on the server:** The server doesn't need to maintain any state about previous requests.
    - **Simple to understand:** Intuitive for both developers and API consumers.
  - Cons:
    - **Poor performance with large offsets:** The database needs to scan through a large number of rows before returning the requested page, leading to increased latency, especially for large tables.
    - **Inconsistent when adding/deleting rows (Page Drift):** If new items are inserted or deleted before the requested offset, the same item might appear on multiple pages, or items might be skipped entirely. This is also known as the "missing item" or "duplicate item" problem.
    - **Not suitable for real-time feeds:** Page drift makes it unsuitable for rapidly changing datasets.
    - **Vulnerable to Parameter Manipulation:** Clients can easily manipulate the offset parameter, potentially accessing unauthorized data or causing performance issues.

- **Keyset Pagination (also known as Seek Method or Time-Based Pagination):**
  - Uses values from the last item on the previous page to fetch the next set of items: `GET /feed?after=2021-05-25T00:00:00&limit=20`.  Instead of an offset, uses a value from the last item of the previous page (e.g., timestamp, ID) to determine where to start the next page.
  - Pros:
    - **Translates easily to SQL:** Can be implemented efficiently using `WHERE` clauses in SQL (e.g., `SELECT * FROM tweets WHERE created_at > '2021-05-25T00:00:00' ORDER BY created_at ASC LIMIT 20`).
    - **Good performance with large datasets:** Avoids the performance issues of offset pagination, as the database can use an index on the ordering field.
    - **Stateless on the server:** The server doesn't need to maintain any state about previous requests.
    - **More resilient to insertions:** Insertions after the "after" value won't cause items to be skipped or duplicated (though insertions *before* will affect the total number of pages).
  - Cons:
    - **"Leaky abstraction" – pagination aware of the database:** The API exposes details about the underlying data storage (the ordering field), increasing the risk of breaking changes if the database schema changes.  API consumers need to know *what* field to paginate on.
    - **Only works on fields with natural ordering (timestamps, IDs, etc.):** Requires a field that can be used to consistently order the results. Doesn't work well if there's no clear ordering.
    - **Can have issues with ties:** If multiple items have the same value for the ordering field, the pagination might not be consistent (addressed using compound keyset pagination).
    - **Sensitive to data modifications:** Deletions of items within a page can still cause issues.
    - **Less flexible for random access:** Difficult to jump to a specific page without iterating through all the preceding pages.

- **Cursor/Seek Pagination (also known as Token-Based Pagination):**
  - Uses opaque cursors or tokens that encapsulate the pagination state. These cursors are typically base64 encoded and encrypted: `GET /feed?after_id=t1234xzy&limit=20`. The server generates a cursor for the next (or previous) page and returns it in the response. The client then uses this cursor to request the next page.
  - Pros:
    - **Decouples pagination from SQL:** The API doesn't expose any details about the underlying data storage or ordering.
    - **Consistent ordering when inserting/deleting new items:** More robust to data modifications than offset pagination.
    - **More flexible:** Can support various ordering criteria and complex queries.
    - **Hides implementation details:** Clients don't need to know which field is being used for sorting/pagination.
  - Cons:
    - **More complex backend:** Requires generating and managing cursors, which involves encoding and encrypting the pagination state.  Also requires decoding and validating the cursor on subsequent requests.
    - **Does not work well if cursors expire or become invalid:** Requires careful management of cursor lifetime and invalidation. Deletions of *all* items between the 'after_id' and the next item can invalidate the cursor.
    - **Still potentially sensitive to item deletions:** If items are deleted and IDs re-used.
    - **Less transparent:** Can be harder to debug issues, as the cursor is an opaque token.
    - **Not easily bookmarkable:**  Cursors might expire, making bookmarking difficult.

- **Page Number Pagination:**
    - Uses page number and page size parameters: `GET /feed?page=3&pageSize=10`. Similar to offset pagination, but uses page numbers instead of offsets.
    - Pros:
        - **Easy for users to understand:** Intuitive for users familiar with traditional pagination.
        - **Simple to implement (initially):** Can be implemented using offset pagination on the backend.
    - Cons:
        - **Inherits all the cons of offset pagination:** Poor performance with large page numbers, page drift issues, etc.
        - **Not suitable for infinite scrolling:** Doesn't scale well for applications with a large number of pages.
        - **Not suitable for real-time data:** Page drift issues.

Choose an approach. Cursor Pagination is suitable for "Design Twitter Feed" as it decouples pagination from the underlying data store and can handle insertions/deletions reasonably well. However, consider the complexity of implementation. If the dataset doesn't change frequently and the API is read-heavy, Keyset pagination might be a simpler and more performant alternative. Offset pagination is generally discouraged unless the dataset is small and doesn't change frequently.

Example API request:
```
GET /v1/feed?after_id=p1234xzy&limit=20
Authorization: Bearer <token>
{
  "data": {
    "items": [
      {
        "id": "t123",
        "author_id": "a123",
        "title": "Title",
        "description": "Description",
        "likes": 12345,
        "comments": 10,
        "attachments": {
          "media": [
            {
              "image_url": "https://static.cdn.com/image1234.webp",
              "thumb_url": "https://static.cdn.com/thumb1234.webp"
            },
            ...
          ]
        },
        "created_at": "2021-05-25T17:59:59.000Z"
      },
      ...
    ]
  },
  "cursor": {
    "count": 20,
    "next_id": "p1235xzy",
    "prev_id": null
  }
}
```
#### Additional Information
- [Evolving API Pagination at Slack](https://slack.engineering/evolving-api-pagination-at-slack/)
- [Everything You Need to Know About API Pagination](https://nordicapis.com/everything-you-need-to-know-about-api-pagination/)

Mention HTTP authentication (Authorization header, 401 Unauthorized) and Rate-Limiting (429 Too Many Requests). Keep it brief and simple. Also, discuss potential API versioning strategies if the pagination scheme needs to be updated in the future.

### Providing the "Signal"
The interviewer assesses:
- Awareness of network challenges and expensive traffic.
- Familiarity with communication protocols.
- Understanding of RESTful API design.
- Familiarity with authentication and security.
- Understanding of network error handling and rate-limiting.
- Understanding of the trade-offs associated with different pagination strategies.
- Ability to choose a pagination strategy that is appropriate for the specific use case.

## Data Storage
### Data Storage Options
Local device storage options:

- **Key-Value Storage (UserDefaults/SharedPreferences/Property List/MMKV):**
  - Stores primitive data with string keys. Best for simple, unstructured, *non-relational* data. Consider `MMKV` on Android for increased performance.
    - Pros:
      - **Easy to use built-in API:** Simple APIs for storing and retrieving data.
      - **Fast for simple operations:** Efficient for small amounts of data.
      - **Low overhead:** Minimal memory footprint.
    - Cons:
      - **Insecure:** Data is stored in plain text (use `EncryptedSharedPreferences` on Android, third-party wrappers or `Keychain` on iOS for sensitive data).
      - **Not for large data:** Performance degrades significantly with large datasets.
      - **No schema or querying:** Limited querying capabilities.  No data integrity constraints.
      - **No data migration:** Difficult to manage schema changes.
      - **Poor performance for complex operations:** Inefficient for complex data structures or operations.
      - **Not ACID compliant:** No guarantee of atomicity, consistency, isolation, or durability.

- **Database/ORM (SQLite/Room/Core Data/Realm/ObjectBox):**
  - Relational or object database. For large, structured, *relational* data requiring complex queries, transactions, and data integrity. Consider `Room` on Android for a modern, type-safe SQLite abstraction.
    - Pros:
      - **Object-relational mapping (ORM):** Simplifies data access and manipulation using object-oriented paradigms.
      - **Schema and querying:** Supports structured data with defined schemas and powerful querying capabilities using SQL or ORM-specific query languages.
      - **Data migration:** Provides mechanisms for managing schema changes and data migration.
      - **ACID properties:** Guarantees atomicity, consistency, isolation, and durability.
      - **Efficient for complex operations:** Optimized for complex data structures and operations.
    - Cons:
      - **More complex setup:** Requires defining schemas, setting up database connections, and managing ORM configurations.
      - **Potentially Insecure:** (use SQLCipher or other encryption libraries, properly configured)
      - **Bigger memory footprint:** Database files can consume significant storage space.
      - **Performance overhead:** ORM can introduce performance overhead compared to raw SQL queries.
      - **Steeper learning curve:** Requires familiarity with SQL or ORM concepts.

- **Custom/Binary (DataStore/NSCoding/Codable/Protocol Buffers):**
  - Low-level storage and loading. For custom data serialization and storage pipelines where you need maximum control over performance and data format.  Consider `Protocol Buffers` for efficient serialization and deserialization.  DataStore on Android provides a performant solution without ORM overhead.
    - Pros:
      - **Highly customizable:** Allows fine-grained control over data serialization and storage.
      - **Performant:** Can be optimized for specific data structures and operations.
      - **Potentially smaller footprint:** If you carefully control serialization and avoid ORM overhead.
    - Cons:
      - **No schema/migration:** Requires manual management of schema changes and data migration.
      - **Lots of manual effort:** Requires writing custom serialization and deserialization code.
      - **Increased complexity:** Requires a deep understanding of data structures and serialization techniques.
      - **Error-prone:** Serialization and deserialization code can be complex and prone to errors.
      - **Harder to debug:** Binary formats are difficult to inspect and debug without specialized tools.

- **On-Device Secure Storage (Keystore/Key Chain/Android Biometrics):**
  - Hardware-backed or OS-encrypted storage for sensitive data, encryption keys, certificates, and user credentials.  Use the `Android Biometrics` API for secure authentication with biometric sensors.
    - Pros:
      - **Secure:** Provides hardware-backed or OS-level encryption for sensitive data. Protects against unauthorized access and data breaches.
      - **Integrates with device security:** Leverages device-level security features such as biometrics and PIN/password authentication.
    - Cons:
      - **Not optimized for general storage:** Designed for storing small amounts of sensitive data such as encryption keys and user credentials.
      - **Encryption/decryption overhead:** Can introduce performance overhead for encryption and decryption operations.
      - **No schema/migration:** Limited querying capabilities. Difficult to manage schema changes.
      - **Complex API:** Can be complex to use correctly.
      - **Limited storage:** Has restrictions on the amount of storage space available.
      - **User Dependency:** Reliant on the user configuring a lockscreen.

- **File Storage (Internal/External Storage):**
  - Storing data as files, suitable for larger unstructured data like images, videos, and documents.
    - Pros:
        - Simple to implement.
        - Suitable for storing large, unstructured data.
    - Cons:
        - No schema.
        - Limited querying capabilities.
        - Can be slow for random access.
        - Requires careful management of file paths and permissions.

### Storage Location
- **Internal:** Sandboxed, private to the app, and not readable by other apps (with few exceptions). Data is deleted when the app is uninstalled.
- **External:** Publicly visible and accessible to other apps (subject to permissions). Data persists even after the app is uninstalled. Generally discouraged for sensitive data.
- **Media/Scoped:** A restricted subset of external storage specifically for media files. Provides enhanced privacy and security compared to traditional external storage. Requires proper permissions and follows specific access patterns.

### Storage Type
- **Documents (Automatically Backed Up):** User-generated data that cannot be easily re-generated (e.g., user profiles, documents, settings). Automatically backed up to cloud storage (e.g., iCloud, Google Drive) unless explicitly excluded.
- **Cache:** Regeneratable data that can be downloaded again or recreated (e.g., downloaded images, cached API responses). Can be deleted by the system to free up space. Should not contain critical data.
- **Temp:** Temporary data that is only used for a short period and should be deleted when no longer needed (e.g., temporary files created during processing). Not backed up and can be deleted by the system at any time.

### Best Practices
- Store as little sensitive data as possible.
- Use encrypted storage for sensitive data (e.g., `EncryptedSharedPreferences`, `Keychain`, or SQLCipher).
- Prevent uncontrolled storage growth. Implement cache cleanup policies and manage file sizes.
- Use appropriate storage locations based on data privacy requirements (internal vs. external storage).
- Consider data backup and restore strategies for user-generated data.
- Use modern data storage APIs (e.g., `DataStore` on Android) for improved performance and security.
- Avoid storing secrets or API keys directly in the code or configuration files.
- Implement proper error handling and logging for data storage operations.
- Test data storage logic thoroughly to ensure data integrity and prevent data loss.

### Approach
Select an approach and discuss pros/cons.
For "Design Twitter Feed":

#### Feed Pagination Table
Use Room (Android) or CoreData (iOS) for storing paginated feed data. The schema could look like this:

```kotlin
// Kotlin (Android - Room)
@Entity(tableName = "feed")
data class FeedItem(
    @PrimaryKey val itemId: String,
    val authorId: String,
    val title: String,
    val description: String,
    val likes: Int,
    val comments: Int,
    val attachments: String, // comma separated list of URLs or JSON
    val createdAt: Date,
    val cursorNextId: String?,
    val cursorPrevId: String?
)
```

```swift
// Swift (iOS - Core Data)
// Add attributes for: itemID, authorID, title, description, likes, comments, 
// attachments, createdAt, cursorNextId, cursorPrevId
```

- Limit the total number of entries (e.g., 500) to control local storage size. Implement a Least Recently Used (LRU) eviction policy to remove older items.
- Flatten attachments into a comma-separated list of URLs or store them as JSON within a text field. Alternatively, create a separate `attachments` table and join it with the `feed` table on `itemId` for better normalization.
- Store `cursorNextId` and `cursorPrevId` with each item to simplify paged navigation. Use nullable types for the cursor IDs.

#### Attachments Storage
- Store attachments (images, videos) as files in Internal Cached storage.
- Use attachment URLs as cache keys. Implement an ImageLoader library like Coil (Android) or Kingfisher (iOS) to manage downloading, caching, and displaying images.
- Delete attachments when the corresponding feed items are deleted from the `feed` table. Implement a mechanism for deleting orphaned attachments.
- Limit the cache size (e.g., 200-400 MB) to prevent excessive storage usage. Use an LRU cache eviction policy to remove less frequently used attachments.

For sensitive media data (e.g., private accounts), selectively encrypt image files using encryption keys stored in the Keystore/KeyChain. Use Android Biometrics API or Touch ID/Face ID on iOS for user authentication to access the encrypted data.

### Providing the "Signal"
The interviewer assesses:
- Awareness of mobile storage types, security, limitations, and compatibility.
- Ability to design a storage solution for common scenarios, considering performance, security, and scalability.
- Understanding of data modeling and database design principles.
- Familiarity with data persistence libraries and frameworks on Android and iOS.
- Awareness of data encryption and secure storage practices.
- Understanding of cache management and eviction policies.

## Additional Topics

### Major Concerns For Mobile Development
Key concerns to keep in mind:

- **User Data Privacy:** Leaks can damage business reputation and result in legal penalties. Ensure compliance with privacy regulations such as GDPR and CCPA.
- **Security:** Protect against reverse engineering (more important for Android), data breaches, and unauthorized access. Implement security best practices such as data encryption, secure authentication, and authorization.
- **Target Platform Changes:** New OS releases may limit functionality and tighten privacy. Stay up-to-date with platform changes and adapt the app accordingly. Use compatibility layers or conditional compilation to handle differences between OS versions.
- **Non-reversibility of released products:** Assume releases are final. Use staged rollouts and server-side feature flags to safely deploy new features and minimize the impact of bugs.
- **Performance/Stability:**
  - Metered data usage: Cellular data can be expensive. Optimize network requests and minimize data transfer. Use data compression techniques.
  - Bandwidth usage: Constant radio usage drains battery. Batch network requests and use efficient network protocols.
  - CPU usage: High computation drains battery and causes overheating. Optimize algorithms and use background threads for computationally intensive tasks.
  - Memory Usage: High usage increases the risk of background termination. Use memory profiling tools to identify and fix memory leaks.
  - Startup Time: Slow startup creates a poor user experience. Optimize app initialization and lazy-load resources.
  - Crashers/ANRs: UI freezes and crashes lead to poor app ratings. Implement crash reporting and use error tracking tools to identify and fix bugs.
- **Battery Consumption:** Minimize battery drain by optimizing network requests, CPU usage, and background processing. Use battery profiling tools to identify and fix battery-related issues.
- **Geo-Location Usage:**
  - Don't compromise privacy with location services.
  - Prefer the lowest possible accuracy. Request increased accuracy progressively.
  - Provide a rationale before requesting location permissions. Use coarse location when fine location is not required. Obfuscate precise locations if possible.
- **3rd-Party SDKs Usage:**
  - SDKs can cause performance regressions, security vulnerabilities, and outages.
  - Guard each SDK with a feature flag.
  - Use A/B testing or staged rollout for new SDK integrations.
  - Have a long-term support and upgrade plan. Regularly review and update SDKs to address security vulnerabilities and performance issues.
- **App Responsiveness:** Ensure the app remains responsive to user input even when performing complex tasks. Use background threads and asynchronous operations to avoid blocking the main thread.
- **Testing on Real Devices:** Test the app on a variety of real devices to ensure compatibility and performance. Use cloud-based device testing services to access a wide range of devices.
- **Licensing and Legal Compliance:** Comply with all applicable licensing terms and legal regulations, including open source licenses, privacy policies, and data security laws.

### Privacy & Security

Privacy and security should be a central consideration throughout the entire development lifecycle, not an afterthought.

- **Minimize user data collection:** Only collect data that is absolutely necessary for the functionality of the app.
  - Avoid device identifiers (use resettable, pseudo-anonymous IDs).  Consider using Instance IDs instead.
  - Anonymize collected data whenever possible. Aggregate data and remove personally identifiable information (PII).
  - Explain what data you are collecting, how it is used, and who it is shared with, in easy to understand terms.
  - Provide users with transparency and control over their data. Allow users to access, modify, and delete their data.
  - Implement data retention policies to ensure that data is only stored for as long as it is needed.
- **Minimize permission usage:** Request only the permissions that are necessary for the app's functionality.
  - Be prepared for permission denials and respect the user's choice. Handle permission denials gracefully and provide alternative functionality if possible.
  - Be prepared for auto-reset permissions. Implement a mechanism for requesting permissions again if they are auto-reset by the system.
  - Be prepared for app hibernation of unused apps.
  - Delegate functionality to system apps (Camera, Photo Picker, File Manager) to reduce the need for permissions.
- **Secure Data Storage:**
  - Assume on-device storage is not secure, even when using KeyStore/KeyChain functionality. Implement encryption to protect sensitive data.
    - Encrypt all sensitive data at rest using strong encryption algorithms.
    - Store encryption keys securely in the KeyStore/KeyChain or a hardware security module (HSM).
    - Use secure data storage APIs such as EncryptedSharedPreferences on Android and the Keychain on iOS.
    - Implement proper access controls to restrict access to sensitive data.
  - Assume backend storage is also not secure.
    - Use end-to-end encryption to protect data in transit and at rest. Implement server-side encryption to protect data stored in the backend.
    - Use secure communication protocols such as HTTPS.
    - Implement proper authentication and authorization mechanisms to restrict access to backend resources.
- **Secure Communication:**
  - Use HTTPS for all network communication.
  - Validate server certificates to prevent man-in-the-middle attacks.
  - Implement proper authentication and authorization mechanisms.
  - Use secure data transfer protocols.
- **Code Security:**
  - Protect against reverse engineering by obfuscating the code and using anti-tampering techniques.
  - Validate user input to prevent injection attacks.
  - Use secure coding practices to avoid common security vulnerabilities.
  - Perform regular security audits and penetration testing.
- **Privacy by Design:**
    - Implement privacy considerations at every stage of the development process.
    - Conduct a Data Protection Impact Assessment (DPIA) to identify and mitigate privacy risks.
    - Use privacy-enhancing technologies (PETs) such as differential privacy and homomorphic encryption.
- **Regular Security Updates:**
    - Stay up-to-date with the latest security vulnerabilities and patches.
    - Regularly update dependencies and third-party libraries.
    - Implement a security incident response plan.
- **Privacy Compliance:**
    - Comply with all applicable privacy regulations, such as GDPR and CCPA.
    - Obtain user consent before collecting or processing personal data.
    - Provide users with the right to access, modify, and delete their data.
    - Implement data breach notification procedures.
- **Transparency and User Education:**
  - Assume platform security/privacy rules will change. Control critical functionality with remote feature flags to quickly adapt to new regulations.
  - User _perception_ of security is crucial. Discuss how you would educate users about data collection, storage, and transmission.
    - Be transparent about data collection practices.
    - Provide users with clear and concise information about how their data is used.
    - Educate users about security threats and best practices.
    - Implement a user-friendly privacy policy.

#### Cloud vs On-Device

Choose between running functionality on a device or in the cloud. Especially relevant for On-Device AI, Machine Learning, and complex processing.

**Advantages of running things in a cloud:**
- **Device-independent:** Customers are not limited by device capabilities (CPU, GPU, memory). Runs consistently across all devices.
- **Better usage of client system resources:** Offloads intensive computation, saving device battery and preventing overheating.
- **Fast pace of changes:** Updates and improvements can be deployed server-side without requiring app updates, enabling rapid iteration. A/B testing and experimentation are easier.
- **Bigger computational resources:** Access to virtually unlimited compute power, enabling complex tasks such as large-scale data processing and complex AI models.
- **Better security:** Server-side code is generally more secure than client-side code, as it is not exposed to the user. Easier to protect against reverse engineering and tampering. Secure environments and more sophisticated security tools are usually available.
- **Easier analytics and offline data analysis:** Centralized data collection enables comprehensive analytics and insights into user behavior and app performance. Easier to perform batch processing and machine learning on aggregated data.
- **Simplified development:** Can simplify mobile development by offloading complex logic and processing to the cloud. Developers can focus on building the user interface and user experience.

**Disadvantages of running things in a cloud:**
- **Increased Latency:** Requires network communication, which can introduce latency and negatively impact the user experience. Can be a major problem for real-time or interactive applications.
- **Network Dependency:** Requires a reliable network connection. Users may not be able to access certain features when they are offline or have a poor network connection.
- **Cost:** Cloud resources can be expensive, especially for computationally intensive tasks. Need to carefully manage cloud costs and optimize resource utilization.
- **Security Risks:** Requires careful attention to security to protect data in transit and at rest. Data breaches can have serious consequences.
- **Compliance Challenges:** May be subject to data privacy regulations such as GDPR and CCPA. Requires careful consideration of data residency and data transfer requirements.

**Advantages of running things on a device:**
- **Better privacy:** Data does not leave the user's device, providing enhanced privacy and security. Reduces the risk of data breaches and unauthorized access. No data is transmitted over the network.
- **Real-time functionality:** Certain operations can run much faster on a user's device, providing a more responsive and interactive user experience. Suitable for applications that require low latency.
- **Lower bandwidth usage:** Reduces bandwidth consumption by avoiding the need to send data over the network. Can be important for users with limited data plans or poor network connections.
- **Offline functionality:** Allows users to access certain features even when they are offline. Improves the user experience in areas with poor network coverage.
- **Lower backend usage:** Reduces the load on the backend servers, potentially saving costs. Distributes the processing load across all client devices. No server costs for computations performed on-device.
- **Reduced dependency on external services:** Makes the application more resilient to outages or disruptions of external services.

**Disadvantages of running things on a device:**
- **Device Limitations:** Performance is limited by the device's CPU, GPU, and memory. May not be suitable for computationally intensive tasks or large datasets.
- **Inconsistent Performance:** Performance can vary significantly depending on the device. Requires careful optimization to ensure consistent performance across a range of devices.
- **Larger App Size:** Can increase the app size due to the inclusion of machine learning models or other large libraries.
- **Difficult to Update:** Requires app updates to deploy new features or bug fixes. Can be difficult to ensure that all users are running the latest version of the app.
- **Security Risks:** Client-side code can be more vulnerable to reverse engineering and tampering. Requires careful attention to security to protect sensitive data.
- **Battery Consumption:** Can drain battery life, especially for computationally intensive tasks.

**Things you should never run on a device:**
- **New "resource" creation (sensitive):** Generating coupons, tickets, promo codes, or other sensitive resources should always be done server-side to prevent fraud.
- **Transactions and Payment verification:** Never process payment information directly on the device. Delegate to a 3rd-party SDK (e.g., Stripe, Braintree) or redirect the user to a secure payment gateway. Sensitive financial calculations should be performed on the server.
- **Complex Business Logic or sensitive Algorithms:** Sensitive algorithms, user authentication, or core business rules should be kept on the server for security and control. This protects valuable IP and allows for updates without app releases.

### Offline and Opportunistic State
Offline state ensures app usability without a network connection:

- User can like/comment/delete tweets offline.
- UI updates "opportunistically" assuming requests will be sent when online.
- App displays cached results.
- App notifies users about its Offline State, and any pending actions.
- State changes are batched and sent when the network goes back online.

#### Request de-duplication
Ensure retrying requests won't create duplicates on the server (idempotence). Use unique client-side generated request IDs and server-side de-duplication. The server should store a record of processed request IDs for a certain period.

Use the Idempotency-Key HTTP header to ensure a client can safely retry a request without accidentally performing the same operation twice. The client generates a unique key for each request and includes it in the header. If the server receives the same key again, it knows that the request has already been processed and can return the previous response.

Example (Kotlin):

```kotlin
// Client-side request
val idempotencyKey = UUID.randomUUID().toString()
val request = Request.Builder()
    .url("https://example.com/api/like")
    .header("Idempotency-Key", idempotencyKey)
    .post(requestBody)
    .build()
```

#### Synching Local and Remote States
Handle state synchronization across multiple devices. Implementing conflict resolution is crucial.

**Local Conflict Resolution (Last Write Wins with Timestamps - often NOT recommended):**
Local device merges remote state with local state and uploads changes.

Pros:
- Easy to implement (but often incorrect).

Cons:
- Insecure - gives a local device authority over the backend.
- Fails if multiple devices send updates simultaneously (the last update "wins," potentially losing data).
- Requires app update for merging logic changes.
- Can lead to data loss and inconsistencies.

**Remote Conflict Resolution (Recommended):**
Local device sends its local state to the backend, receives a *complete* new state, and overwrites the local state.

Pros:
- Moves conflict resolution authority to the backend.
- Does not require client updates for resolution logic.
- More reliable and consistent data synchronization.

Cons:
- More complex backend implementation.
- Potentially overwrites local, un-synced changes on the client, requiring careful UX design to manage.

**Conflict Resolution Strategies (on the server, using Remote Conflict Resolution):**

* **Last Write Wins with Timestamps (Not recommended in most cases):** The server simply accepts the latest update based on the timestamp.  This is the simplest approach, but it can lead to data loss if clocks are not synchronized or if updates are received out of order.
* **Conflict Detection and Resolution UI:** The server detects a conflict and returns an error to the client, prompting the user to manually resolve the conflict through a UI. This approach provides the most control over conflict resolution but can be cumbersome for the user.
* **Operational Transformation (OT):** A more sophisticated approach that transforms operations based on the history of changes. This allows for concurrent edits to be merged seamlessly. OT is more complex to implement but provides a better user experience.
* **Conflict-Free Replicated Data Types (CRDTs):** Data structures that are designed to be merged automatically without conflicts. CRDTs are a good choice for applications where data consistency is critical.

#### More Information
- Offline functionality for Trello’s mobile applications:
  - [Airplane Mode: Enabling Trello Mobile Offline](https://tech.trello.com/sync-architecture/)
  - [Syncing Changes](https://tech.trello.com/syncing-changes/)
  - [Sync Failure Handling](https://tech.trello.com/sync-failure-handling/)
  - [The Two ID Problem](https://tech.trello.com/sync-two-id-problem/)
  - [Offline Attachments](https://tech.trello.com/sync-offline-attachments/)
  - [Sync is a Two-Way Street](https://tech.trello.com/sync-downloads/)
  - [Displaying Sync State](https://tech.trello.com/sync-indicators/)

### Caching
_TBD_
### Quality Of Service (QoS)

Implementing Quality of Service (QoS) for network operations in mobile apps is critical to providing a smooth user experience while conserving device resources, especially battery life. QoS involves prioritizing network requests based on their importance to the user and the app's functionality.

- **Limit Concurrent Network Operations:** Restricting the number of simultaneous network requests is crucial for several reasons. Each active network connection consumes system resources (CPU, memory, radio), impacting performance and battery life. Aim for a limit of 4-10 concurrent operations, adjusting based on device state (battery level, charging status, network connectivity) and app usage patterns. Consider the network bandwidth available.
- **Adaptive Concurrency:** Monitor network conditions (Wi-Fi vs. cellular, signal strength) and adjust the concurrency limit dynamically. Reduce the limit on cellular networks or when the signal is weak.
- **Network Request Prioritization:** Assign a QoS class to each network request based on its importance. This allows the app to prioritize essential requests while deferring less critical ones.
- **QoS Classes and Examples:**

  - **User-Critical (High Priority):** Requests directly impacting the user's immediate experience and require minimal latency.

    - Fetching the next page of data for the Tweet Feed (infinite scrolling).
    - Requesting Tweet Details (when a user taps on a tweet).
    - Sending a direct message.
    - Authentication requests.

  - **UI-Critical (Medium Priority):** Requests contributing to the user interface's responsiveness, but can tolerate slightly higher latency.

    - Fetching low-resolution placeholder images for tweets while scrolling.
    - Loading thumbnails or previews.
    - Non-blocking UI updates.

  - **UI-Non-Critical (Low Priority):** Requests enhancing the UI but are not essential for its core functionality. Can be delayed or canceled if necessary.

    - Fetching high-resolution images for tweets on the Feed.
    - Preloading images for tweets that are not yet visible.
    - Downloading non-essential resources.

  - **Background (Lowest Priority):** Requests performed in the background without directly impacting the user experience. Can be deferred until the device is idle or charging.

    - Posting "likes" or comments.
    - Uploading analytics data.
    - Backing up data.
    - Syncing data with the server.

- **Priority Queue Scheduling:** Implement a priority queue to manage network requests based on their QoS class. Dispatch requests in the order of their priority, ensuring that user-critical requests are processed first.
- **Request Suspension and Resumption:** If the maximum number of concurrent operations is reached, suspend lower-priority requests to allow higher-priority requests to proceed. Resume suspended requests when resources become available. Use cancellation tokens to allow for quickly cancelling long-running tasks if needed.
- **Cancellation Support:** Implement cancellation support for network requests to prevent unnecessary data transfer and resource consumption. Cancel requests when they are no longer needed (e.g., when the user scrolls past a tweet).
- **Network Request Throttling:** Implement throttling mechanisms to prevent the app from overwhelming the network with too many requests. Use techniques such as exponential backoff and rate limiting.
- **Device State Awareness:** Consider the device's current state when scheduling network requests. Defer low-priority requests when the device is on battery or has a weak network signal.
- **Power Management Optimization:** Minimize the number of wake locks held by the app to prevent battery drain. Use the JobScheduler (Android) or BackgroundTasks framework (iOS) to schedule background tasks efficiently.
- **Network Change Monitoring:** Monitor network connectivity changes and adapt the app's behavior accordingly. Pause or cancel network requests when the device loses connectivity.
- **Error Handling and Retries:** Implement proper error handling and retry mechanisms for network requests. Use exponential backoff to avoid overwhelming the server with retries.
- **Testing and Monitoring:** Thoroughly test the app's network behavior under different network conditions and device states. Use network monitoring tools to identify performance bottlenecks and optimize network requests. Implement logging and analytics to track network usage and identify potential issues.

By implementing these QoS techniques, you can significantly improve the performance, responsiveness, and battery life of your mobile app, providing a better experience for your users.

### Resumable Uploads (Chunked Uploads)

Resumable uploads, also known as chunked uploads, break down a large file upload into smaller, manageable chunks. This technique allows the upload to be paused and resumed without losing progress, making it ideal for situations where network connectivity is unreliable or the upload size is significant. This also allows for more granular error reporting and potentially, faster recovery.

The resumable upload process typically involves these stages:

- **Upload Initialization:** The client initiates the upload by sending a request to the server, providing metadata about the file (name, size, content type). The server creates a temporary storage location and returns a unique upload ID or URL to the client.
- **Chunk Bytes Uploading (Appending):** The client divides the file into smaller chunks (e.g., 1MB, 5MB) and uploads each chunk separately to the server using the upload ID or URL. The client sends each chunk with a specific offset indicating its position in the overall file.
- **Upload Finalization:** Once all chunks have been successfully uploaded, the client sends a finalization request to the server. The server assembles the chunks into the complete file and performs any necessary processing (e.g., validation, transcoding).

![Resumable-Uploads](/images/resumable-uploads.svg)

**Advantages of Resumable Uploads:**

- **Allows you to resume interrupted data transfer operations without restarting from the beginning:** This is the primary advantage, especially crucial for users with unreliable network connections or large file uploads. Saves time, bandwidth, and improves the user experience.
- **Improved Reliability:** By dividing the upload into smaller chunks, it reduces the impact of network interruptions. If a chunk fails to upload, only that chunk needs to be retransmitted, not the entire file.
- **Bandwidth Optimization:** Resuming from a specific point avoids resending already transferred data, conserving bandwidth.
- **Progress Monitoring:** Allows for more accurate and granular progress tracking during the upload process, providing users with better feedback. This also allows for better prioritization of bandwidth within the device.
- **Parallel Uploading (Potentially):** Some implementations allow for uploading multiple chunks in parallel, further speeding up the upload process (however, this can increase complexity). This requires server-side support for out-of-order chunk arrival and assembly.
- **Improved Error Handling:** Easier to pinpoint and handle errors at a chunk level rather than the entire upload.
- **Resource Management:** Can help manage server-side resources more effectively by receiving and processing data in smaller increments.
- **Suitable for Large Files:** Essential for uploading files exceeding size limits imposed by HTTP servers or mobile operating systems.

**Disadvantages of Resumable Uploads:**

- **An overhead from additional connections and metadata:** Each chunk requires a separate HTTP request, adding overhead to the overall upload process. Requires careful management of HTTP headers.
- **Increased Complexity:** Implementing resumable uploads requires more complex client-side and server-side logic compared to single-request uploads.
- **Server-Side Requirements:** Requires server-side support for handling chunked uploads, storing temporary data, and assembling the complete file.
- **State Management:** Both client and server need to maintain state about the upload process (e.g., upload ID, chunk offsets), which can add complexity.
- **Error Handling:** Requires robust error handling to deal with network interruptions, server errors, and other potential issues.
- **Potential for Data Corruption:** If chunks are not assembled correctly, it can lead to data corruption. Requires careful validation and integrity checks.
- **Storage Requirements:** The server needs temporary storage space to hold the uploaded chunks until the upload is finalized.

**Additional Considerations:**

- **Chunk Size:** The optimal chunk size depends on network conditions, file size, and server capabilities. Experiment with different chunk sizes to find the best balance between performance and overhead.
- **Encryption:** Encrypt the uploaded chunks to protect sensitive data in transit and at rest.
- **Concurrency:** Consider the number of concurrent uploads allowed to prevent overwhelming the device or server.
- **Timeout Handling:** Implement timeouts to handle stalled uploads.
- **Background Uploads:** If the upload can be interrupted (app closed), consider using background upload tasks with OS-level support (WorkManager/BackgroundTasks)
- **Idempotency:** Ensure that resuming a failed upload does not result in duplicate data on the server. Use unique identifiers for each chunk.

**Note:** Resumable uploads are most effective with large uploads (videos, archives) on unstable networks. For smaller files (images, texts) and stable networks, a single-request upload should be sufficient. Carefully weigh the benefits and drawbacks before implementing resumable uploads. If targeting a specific cloud storage provider, utilize their SDK which may provide simplified upload mechanisms.

#### More Info:
- [YouTube: Resumable Uploads](https://developers.google.com/youtube/v3/guides/using_resumable_upload_protocol)
- [Google Photos: Resumable Uploads](https://developers.google.com/photos/library/guides/resumable-uploads)
- [Google Cloud: Resumable Uploads](https://cloud.google.com/storage/docs/resumable-uploads)
- [Twitter: Chunked Media Upload](https://developer.twitter.com/en/docs/twitter-api/v1/media/upload-media/uploading-media/chunked-media-upload)

### Prefetching

Prefetching is a technique used to improve application performance by proactively fetching data or resources that are likely to be needed in the future. By loading data in the background before it's explicitly requested, prefetching can significantly reduce latency and enhance the user experience. This creates the illusion of instant loading times and a more responsive application.

Prefetching can be applied to various types of content, including:

- **Data:** Pre-loading data for upcoming screens or features.
- **Images:** Caching images likely to be viewed by the user.
- **Code:** Downloading code modules or assets for features that might be used soon.
- **Configuration:** Fetching configuration updates in advance.

#### Types of Prefetching:

*   **Predictive Prefetching**: Uses machine learning or heuristics to predict which data or resources the user will need based on their past behavior and patterns.
*   **Cache-Aware Prefetching**: Takes into account what is already cached and prefetches only what is missing, optimizing bandwidth usage.
*   **Just-In-Time (JIT) Prefetching**: Preloads resources just before they are needed. E.g. when the user scrolls close to the end of the page (but before making a request), prefetch the next page.

#### Advantages of Prefetching:

- **Improved Performance:** Reduced latency and faster loading times.
- **Enhanced User Experience:** Smoother and more responsive application.
- **Increased User Engagement:** Users are more likely to interact with an app that feels fast and fluid.
- **Hides network latency:** Creates a better user experience, even on slower connections.

#### Disadvantages of Prefetching:

- **Increased Bandwidth Usage:** Unnecessary prefetching can consume bandwidth, especially on metered connections.
- **Increased Battery Consumption:** Downloading data in the background can drain battery life.
- **Increased Storage Consumption:** Caching prefetched data can consume device storage.
- **Potential for Stale Data:** Prefetched data may become stale if it's not updated frequently.
- **Resource Wastage:** Resources might be fetched that are never actually used, wasting bandwidth and battery.
- **Complex Implementation:** Requires careful planning and implementation to avoid negative impacts.

#### Considerations for Implementing Prefetching:

- **Network Conditions:** Adapt prefetching behavior based on network connectivity (Wi-Fi vs. cellular). Disable prefetching on metered connections or when the signal is weak.
- **Device State:** Consider the device's battery level and charging status. Disable prefetching when the battery is low.
- **User Behavior:** Use analytics to track user behavior and identify patterns. Prefetch content that is most likely to be needed.
- **Data Staleness:** Implement a mechanism for invalidating and refreshing prefetched data to ensure that it's up-to-date. Use ETags to avoid downloading the same content repeatedly.
- **Cache Management:** Implement a cache eviction policy to remove less frequently used data and prevent excessive storage consumption. Use Least Recently Used (LRU) or Least Frequently Used (LFU) algorithms.
- **QoS (Quality of Service):** Prioritize prefetching requests based on their importance. Defer prefetching tasks when user-critical requests are in progress.
- **User Control:** Provide users with control over prefetching settings. Allow them to disable prefetching or adjust the amount of data that is prefetched.
- **Testing and Monitoring:** Thoroughly test the app's prefetching behavior under different network conditions and device states. Monitor network usage, battery consumption, and storage consumption to identify potential issues.
- **Respect User Privacy:** Only prefetch data that is relevant to the user's current activity. Avoid prefetching sensitive or private data without explicit consent.

#### Mitigating Negative Impacts:

- **Use ETags**: Send conditional GET requests with `If-None-Match` to check if the prefetched content has been updated. This avoids downloading the same content repeatedly if it hasn't changed.
- **Adaptive Prefetching**: Monitor network conditions and device state and adjust prefetching behavior accordingly.
- **Time-Based Invalidation**: Set a time-to-live (TTL) for prefetched data and invalidate it after a certain period to prevent staleness.
- **User-Initiated Refresh**: Provide a way for users to manually refresh the data if they suspect it's stale.
- **Stop on User Action**: If the user performs an action that indicates they no longer need the prefetched content, stop the prefetching process immediately.

#### More Info:
- [Optimize downloads for efficient network access](https://developer.android.com/training/efficient-downloads/efficient-network-access)
- [Improving performance with background data prefetching](https://instagram-engineering.com/improving-performance-with-background-data-prefetching-b191acb39898)
- [Informed mobile prefetching](https://dl.acm.org/doi/10.1145/2307636.2307651)
- [Understanding prefetching and how Facebook uses prefetching](https://www.facebook.com/business/help/1514372351922333)

## Conclusion

Navigating system design interviews can feel like an overwhelming task. Remember that there's inherent randomness – the process can differ significantly across companies and even between interviewers within the same company. The key is to focus on what you *can* control and learn from what you can't.

### Things You Can Control:

- **Your Mindset:** Approach each interview with a positive and curious attitude. Be enthusiastic about solving the problem and demonstrate a willingness to learn. Be respectful and collaborative, even under pressure. A good attitude leaves a lasting impression.
- **Your Preparation:** Thorough preparation is your strongest weapon. Practice mock interviews with peers, mentors, or online services. The more you practice, the more comfortable and confident you'll become.
- **Your Knowledge Base:** Continuously expand your knowledge of mobile technologies, architectural patterns, and design principles. Stay updated with the latest trends and best practices in iOS and Android development.
    - Immerse yourself in open-source projects: [iOS](https://github.com/search?q=iOS&type=Repositories), [Android](https://github.com/search?o=desc&q=Android&s=stars&type=Repositories)
    - Follow tech company engineering blogs to gain insights into real-world challenges and solutions:
        - [Uber](https://eng.uber.com/category/articles/mobile/)
        - Instagram: [iOS](https://instagram-engineering.com/tagged/ios), [Android](https://instagram-engineering.com/tagged/android)
        - [Trello](https://tech.trello.com/)
        - Meta:
            - [Engineering at Meta](https://engineering.fb.com/tag/mobile/)
            - [Meta Tech Podcast](https://open.spotify.com/show/1NlTm7OkZmcrOPfvlqlBMz)
        - [Square](https://code.cash.app)
        - [Dropbox](https://dropbox.tech/mobile)
        - [Reddit](https://www.redditinc.com/blog/topic/technology)
        - [Airbnb](https://medium.com/airbnb-engineering/tagged/mobile)
        - [Lyft Tech Podcast](https://podcasts.apple.com/us/podcast/lyft-mobile/id1453587931)
        - [Curated List of Blog Posts](/BLOGPOSTS.MD)
- **How you articulate your thought process:** Clearly and concisely explain your reasoning behind design choices, and be prepared to justify your decisions with evidence and trade-offs. Walk the interviewer through your process step-by-step.
- **Your Resume:** Craft a compelling resume that highlights your accomplishments and demonstrates the impact you've made in your previous roles. Quantify your achievements whenever possible. Tailor your resume to match the specific requirements of the position you're applying for.

### Things Beyond Your Control:

- **The Interviewer's Style and Bias:** You might encounter interviewers with different personalities, preferences, or even unconscious biases. Focus on presenting your best self and don't take any perceived negativity personally.
- **The Competition:** There might be other highly qualified candidates with more experience or skills. Focus on showcasing your unique strengths and value proposition.
- **The Hiring Committee's Decision:** The final decision rests with the hiring committee, which considers various factors beyond your control, such as budget constraints, team dynamics, and organizational priorities.

### Embrace the Process, Learn from Every Experience:

Remember that every interview, regardless of the outcome, is a valuable learning opportunity. Analyze your performance, identify areas for improvement, and seek feedback from others. Don't be discouraged by rejections – view them as stepping stones toward your ultimate goal.

**Failure is a part of the process.** Even the most experienced engineers face rejection at some point in their careers. What matters is how you respond to setbacks. Use each experience to refine your skills, expand your knowledge, and strengthen your resolve.

**Keep practicing, keep learning, and keep pushing yourself.** The right opportunity will eventually come your way. Believe in your abilities, and never give up on your dreams. Good luck on your journey!

## Frequently Asked Questions

**Q: How do you know this approach works? Why is this guide necessary?**
- A: There's no one-size-fits-all approach to system design interviews. The structure of the interview heavily depends on the interviewer's style and the company's specific needs. However, having a structured framework like this helps both the candidate and the interviewer focus on the *content* of the discussion rather than getting bogged down in the organizational aspects of the interview. It provides a common language and a starting point for exploration.

**Q: Can you go a bit deeper into [specific technology/pattern/component]?**
- A: This guide aims to provide a broad overview of key concepts. It doesn't delve into extreme detail on any specific technology or implementation. The goal is to equip you with a foundational understanding, encouraging you to draw upon your personal experience and expertise to provide concrete solutions during the interview. Remember, the interviewer is assessing your problem-solving skills and your ability to adapt and apply your knowledge.

**Q: I'm an interviewer - won't this guide encourage candidates to memorize solutions and "cheat" during the interview?**
- A: System design is far more nuanced than coding challenges. Memorizing a specific solution is rarely sufficient, as interviewers can easily adapt the problem or introduce new constraints. This guide aims to provide a foundational understanding, but success hinges on a candidate's ability to reason critically, apply their knowledge creatively, and articulate their thought process clearly. It's usually quite apparent when a candidate is relying solely on memorization rather than demonstrating genuine understanding. Exposure to common patterns and approaches can reduce interview anxiety and allow the candidate to showcase their problem-solving skills more effectively.

**Q: What if the interviewer wants to focus on a technology I'm not very familiar with?**
- A: Be honest about your strengths and weaknesses. If the interviewer veers into an area where your knowledge is limited, acknowledge that and explain your general understanding of the concepts involved. Emphasize your willingness to learn and your ability to quickly acquire new knowledge. Suggest alternative solutions using technologies you are more comfortable with.

**Q: How important is it to draw a diagram? What if I'm not a visual thinker?**
- A: Visualizing the system architecture through diagrams can be extremely helpful for both you and the interviewer. It provides a clear and concise way to communicate the overall design and the interactions between components. If you're not naturally inclined to draw diagrams, practice beforehand! There are many online tools that can help. However, if you feel a diagram is not the best way to express yourself, communicate clearly why and offer an alternative representation, such as a well-structured verbal explanation.

**Q: How should I handle disagreements with the interviewer?**
- A: It's perfectly acceptable to have different opinions than the interviewer. However, approach the discussion respectfully and professionally. Clearly articulate your reasoning and be willing to consider alternative viewpoints. Avoid being argumentative or dismissive. If you strongly disagree, you can politely acknowledge the interviewer's perspective while reiterating your own rationale. The goal is to demonstrate your critical thinking skills and your ability to engage in constructive dialogue.

**Q: What if I get stuck and can't think of a solution?**
- A: Don't panic! It's okay to pause and take a moment to gather your thoughts. Communicate your thought process out loud. Explain what you're considering and what challenges you're facing. Ask clarifying questions to help you better understand the problem. If you're still stuck, ask the interviewer for a hint or guidance. The interviewer is more interested in seeing how you approach a difficult problem than in finding the perfect solution.

**Q: How much detail should I provide when discussing specific components?**
- A: The level of detail should depend on the interviewer's interest and the time available. Start with a high-level overview and then dive deeper into specific aspects as prompted by the interviewer. Pay attention to their body language and ask if they want you to elaborate or move on to another topic. Don't get bogged down in minute details unless the interviewer specifically requests it.

**Q: How do I choose between different architectural patterns (MVP, MVVM, MVI, etc.)?**
- A: There's no single "best" architectural pattern. Each pattern has its own advantages and disadvantages, and the best choice depends on the specific requirements of the project. Be prepared to explain the pros and cons of different patterns and justify your choice based on factors such as complexity, testability, and maintainability. Familiarity with common patterns such as MVVM with a unidirectional data flow (like MVI or Redux) is generally favored.

**Q: How can I improve my communication skills for system design interviews?**
- A: Practice explaining complex technical concepts in a clear and concise manner. Use diagrams and visual aids to illustrate your ideas. Practice active listening and ask clarifying questions to ensure that you understand the interviewer's perspective. Seek feedback from others on your communication style. Record yourself explaining technical concepts and analyze your performance. Focus on being clear, concise, and confident in your communication.

**Q: Should I memorize specific code snippets or API calls for the interview?**
- A: While having a general understanding of common APIs and libraries is helpful, it's generally not necessary (or advisable) to memorize specific code snippets. The focus should be on demonstrating your understanding of the underlying concepts and your ability to apply them to solve problems. If you need to refer to a specific API or code example, explain why you're using it and what it accomplishes.

**Q: Is it better to focus on a single area of mobile development (e.g., iOS or Android) or have broad knowledge across both platforms?**
- A: The ideal approach depends on the specific role and the company's needs. Some roles may require deep expertise in a single platform, while others may value broad knowledge across both platforms. Be honest about your skills and experience. If you have broad knowledge, highlight your ability to learn new technologies and adapt to different platforms. If you have deep expertise in a single platform, emphasize your ability to contribute immediately to the team and your willingness to learn about other platforms as needed.

## Additional Information
More System Design exericses [here](/exercises)!

### Junior, Middle, Senior, and Staff level interviews
The system design experience would be different depending on the candidate's target level. An approximate engineering level breakdown can be found [here](https://candor.co/articles/tech-careers/google-promotions-the-real-scoop-on-leveling-up).

_NOTE: There's no clear mapping between years of experience and seniority - some ranges might exist but it largely depends on the candidate's background._

#### Junior engineers
The system design round of junior engineers is optional since it's pretty unlikely they would have experience designing software systems.

#### Middle level engineers
The middle-level engineering design round might be heavy on the implementation side. The interviewer and the candidate would mostly talk about building a specific component using platform libraries.

#### Senior level engineers
The senior-level engineering design round could be more high-level compared to the previous levels. The interviewer and the candidate would mostly talk about multiple components and how they communicate with each other. The implementation details could be less important unless the candidate needs to make a decision that drastically affects the application performance. The candidate should also be able to select a technical stack and describe its advantages and trade-offs.

#### Staff level engineers
The staff-level engineering design round moves away from technical to strategic decisions. The candidate might want to discuss the target audience, available computational and human resources, expected traffic, and deadlines. Instead of thinking in terms of implementation tasks - the candidate should put business needs first. For example, being able to explain how to reduce product time-to-market; how to safely rollout and support features; how to handle OMG situations and large-scale outages. The user privacy topics and their legal implications become extremely important and should be discussed in great detail.

## Looking for more content?
### System Design Exercises
Check out the [collection](/exercises) of mobile system design exercises.
### Common System Design Interview Mistakes
Check the guide [here](/common-interview-mistakes.md).
### Mock Interviews
Check out the mock interview [archive](https://www.youtube.com/playlist?list=PLaMN-JyH50OYAfxJEpiQTYTD-gxTf7x9d) or [sign-up](https://forms.gle/Cez2R31wqtgp3RsX9) to be a candidate yourself!

## Consider "starring" the repository to help other people discover the guide! Thank you!
