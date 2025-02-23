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
- **Offset Pagination:**
  - `limit` and `offset` parameters: `GET /feed?offset=100&limit=20`.
  - Pros:
    - Easy to implement – parameters passed directly to SQL.
    - Stateless on the server.
  - Cons:
    - Poor performance with large offsets.
    - Inconsistent when adding new rows (Page Drift).
- **Keyset Pagination:**
  - Uses values from the last page: `GET /feed?after=2021-05-25T00:00:00&limit=20`.
  - Pros:
    - Translates easily to SQL.
    - Good performance with large datasets.
    - Stateless on the server.
  - Cons:
    - "Leaky abstraction" – pagination aware of the database.
    - Only works on naturally ordered fields (timestamps).
- **Cursor/Seek Pagination:**
  - Uses stable IDs decoupled from SQL (usually base64 encoded and encrypted): `GET /feed?after_id=t1234xzy&limit=20`.
  - Pros:
    - Decouples pagination from SQL.
    - Consistent ordering when inserting new items.
  - Cons:
    - More complex backend.
    - Does not work well with deletions (invalid IDs).

Choose an approach. Cursor Pagination is suitable for "Design Twitter Feed."

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

Mention HTTP authentication (Authorization header, 401 Unauthorized) and Rate-Limiting (429 Too Many Requests). Keep it brief and simple.

### Providing the "Signal"
The interviewer assesses:
- Awareness of network challenges and expensive traffic.
- Familiarity with communication protocols.
- Understanding of RESTful API design.
- Familiarity with authentication and security.
- Understanding of network error handling and rate-limiting.

## Data Storage
### Data Storage Options
Local device storage options:

- **Key-Value Storage (UserDefaults/SharedPreferences/Property List):**
  - Stores primitive data with string keys. Best for simple, unstructured data.
    - Pros:
      - Easy to use built-in API.
    - Cons:
      - Insecure (use EncryptedSharedPreferences on Android, third-party wrappers on iOS).
      - Not for large data.
      - No schema or querying.
      - No data migration.
      - Poor performance.
- **Database/ORM (sqlite/Room/Core Data/Realm):**
  - Relational database. For large, structured data requiring complex queries.
    - Pros:
      - Object-relational mapping.
      - Schema and querying.
      - Data migration.
    - Cons:
      - More complex setup.
      - Insecure (use wrapper libraries).
      - Bigger memory footprint.
- **Custom/Binary (DataStore/NSCoding/Codable):**
  - Low-level storage and loading. For custom data storage pipelines.
    - Pros:
      - Highly customizable.
      - Performant.
    - Cons:
      - No schema/migration.
      - Lots of manual effort.
- **On-Device Secure Storage (Keystore/Key Chain):**
  - OS-encrypted storage for keys and key-value data.
    - Pros:
      - Secure.
    - Cons:
      - Not optimized for general storage.
      - Encryption/decryption overhead.
      - No schema/migration.
### Storage Location
- **Internal:** Sandboxed and not readable by other apps.
- **External:** Publicly visible, often not deleted on app deletion.
- **Media/Scoped:** For media files.
### Storage Type
- **Documents (Automatically Backed Up):** User-generated data, automatically backed up.
- **Cache:** Regeneratable data, can be deleted to free space.
- **Temp:** Temporary data, deleted when no longer needed.

### Best Practices
- Store as little sensitive data as possible.
- Use encrypted storage for sensitive data.
- Prevent uncontrolled storage growth. Ensure cache cleanup does not affect functionality.

### Approach
Select an approach and discuss pros/cons.
For "Design Twitter Feed":

#### Feed Pagination Table
Create a "feed" table:
```
item_id:        String
author_id:      String
title:          String
description:    String
likes:          Int
comments:       Int
attachments:    String # comma separated list
created_at:     Date   # also used for sorting
cursor_next_id: String # points to the next cursor page
cursor_prev_id: String # points to the prev cursor page
```
- Limit entries to 500 to control storage size.
- Flatten attachments into a comma-separated list or create an `attachments` table and join it with the `feed` table on `item_id`.
- Store next/prev cursor IDs for paged navigation.

#### Attachments Storage
- Store attachments in Internal Cached storage.
- Use attachment URLs as cache keys.
- Delete attachments after deleting feed items.
- Limit cache size to 200-400 MB.

For sensitive media data, selectively encrypt images with keys stored in Keystore/KeyChain.

### Providing the "Signal"
The interviewer assesses:
- Awareness of mobile storage types, security, limitations, and compatibility.
- Ability to design a storage solution for common scenarios.

## Additional Topics
### Major Concerns For Mobile Development
Key concerns to keep in mind:

- **User Data Privacy:** Leaks can damage business reputation.
- **Security:** Protect against reverse engineering (more important for Android).
- **Target Platform Changes:** New OS releases may limit functionality and tighten privacy.
- **Non-reversibility of released products:** Assume releases are final. Use staged rollouts and server-side feature flags.
- **Performance/Stability:**
  - Metered data usage: Cellular data can be expensive.
  - Bandwidth usage: Constant radio usage drains battery.
  - CPU usage: High computation drains battery and causes overheating.
  - Memory Usage: High usage increases the risk of background termination.
  - Startup Time: Slow startup creates a poor user experience.
  - Crashers/ANRs: UI freezes and crashes lead to poor app ratings.
- **Geo-Location Usage:**
  - Don't compromise privacy with location services.
  - Prefer the lowest possible accuracy. Request increased accuracy progressively.
  - Provide a rationale before requesting location permissions.
- **3rd-Party SDKs Usage:**
  - SDKs can cause performance regressions and outages.
  - Guard each SDK with a feature flag.
  - Use A/B testing or staged rollout for new SDK integrations.
  - Have a long-term support and upgrade plan.

### Privacy & Security
- Minimize user data collection.
  - Avoid device IDs (use one-time generated pseudo-anonymous IDs).
  - Anonymize collected data.
- Minimize permission usage.
  - Be prepared for permission denials and respect the user's choice.
  - Be prepared for auto-reset permissions.
  - Be prepared for app hibernation of unused apps.
  - Delegate functionality to system apps (Camera, Photo Picker, File Manager).
- Assume on-device storage is not secure.
- Assume backend storage is also not secure.
- Assume platform security/privacy rules will change. Control critical functionality with remote feature flags.
- User _perception_ of security is crucial. Discuss how you would educate users about data collection, storage, and transmission.

#### Cloud vs On-Device
Choose between running functionality on a device or in the cloud. Especially relevant for On-Device AI.

**Advantages of running things in a cloud:**
- Device-independent.
- Better usage of client system resources.
- Fast pace of changes.
- Bigger computational resources.
- Better security.
- Easier analytics and offline data analysis.

**Advantages of running things on a device:**
- Better privacy.
- Real-time functionality.
- Lower bandwidth usage.
- Offline functionality.
- Lower backend usage.

**Things you should never run on a device:**
- New resource creation.
- Transactions and Payment verification (unless delegated to a 3rd-party SDK).

### Offline and Opportunistic State
Offline state ensures app usability without a network connection:
- User can like/comment/delete tweets offline.
- UI updates "opportunistically" assuming requests will be sent when online.
- App displays cached results.
- App notifies users about offline state.
- State changes are batched and sent when online.

#### Request de-duplication
Ensure retrying requests won't create duplicates on the server (idempotence). Use unique client-side request IDs and server-side de-duplication.

#### Synching Local and Remote States
Handle state synchronization across multiple devices.

**Local Conflict Resolution:**
Local device merges remote state with local state and uploads changes.

Pros:
- Easy to implement.

Cons:
- Insecure - gives a local device authority over the backend.
- Fails if multiple devices send updates simultaneously.
- Requires app update for merging logic changes.

**Remote Conflict Resolution:**
Local device sends local state to the backend, receives a new state, and overwrites the local state.

Pros:
- Moves conflict resolution authority to the backend.
- Does not require client updates.

Cons:
- More complex backend implementation.

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
### Quality Of Service
Improve energy efficiency by introducing Quality of Service (QoS) classes for network operations.
- Limit concurrent network operations (4-10, depending on device state).
- Assign a QoS class to each request:
  - **User-critical:** Fetching next page of data, tweet details.
  - **UI-critical:** Fetching low-res images while scrolling. Canceled if user scrolls past.
  - **UI-non-critical:** Fetching high-res images. Canceled if user scrolls past.
  - **Background:** Posting likes, analytics.
- Introduce a priority queue to schedule requests in order of priority. Suspend low-priority requests if the max concurrent operations is reached.
### Resumable Uploads
Break down uploads into three stages:
- Upload initialization
- Chunk bytes uploading (appending)
- Upload finalization

![Resumable-Uploads](/images/resumable-uploads.svg)

**Advantages of resumable uploads:**
- Allows you to resume interrupted data transfer operations without restarting from the beginning.

**Disadvantages of resumable uploads:**
- An overhead from additional connections and metadata.

**Note:** Resumable uploads are most effective with large uploads (videos, archives) on unstable networks. For smaller files (images, texts) and stable networks, a single-request upload should be sufficient.

#### More Info:
- [YouTube: Resumable Uploads](https://developers.google.com/youtube/v3/guides/using_resumable_upload_protocol)
- [Google Photos: Resumable Uploads](https://developers.google.com/photos/library/guides/resumable-uploads)
- [Google Cloud: Resumable Uploads](https://cloud.google.com/storage/docs/resumable-uploads)
- [Twitter: Chunked Media Upload](https://developer.twitter.com/en/docs/twitter-api/v1/media/upload-media/uploading-media/chunked-media-upload)

### Prefetching
Improve performance by hiding data transfer latency. Balance prefetching with battery and data usage.
_TBD_
#### More Info:
- [Optimize downloads for efficient network access](https://developer.android.com/training/efficient-downloads/efficient-network-access)
- [Improving performance with background data prefetching](https://instagram-engineering.com/improving-performance-with-background-data-prefetching-b191acb39898)
- [Informed mobile prefetching](https://dl.acm.org/doi/10.1145/2307636.2307651)
- [Understanding prefetching and how Facebook uses prefetching](https://www.facebook.com/business/help/1514372351922333)

## Conclusion
System design interviews involve randomness. Process varies by company and interviewer.

### Things you can control
- **Your attitude:** Be friendly, avoid being pushy.
- **Your preparation:** Practice with mock interviews.
- **Your knowledge:**
  - Gain more experience.
  - Study open-source projects: [iOS](https://github.com/search?q=iOS&type=Repositories), [Android](https://github.com/search?o=desc&q=Android&s=stars&type=Repositories)
  - Read tech company blogs:
    - [Uber](https://eng.uber.com/category/articles/mobile/)
    - Instagram: [iOS](https://instagram-engineering.com/tagged/ios), [Android](https://instagram-engineering.com/tagged/android)
    - [Trello](https://tech.trello.com/)
  	- Meta
  	  - [Engineering at Meta](https://engineering.fb.com/tag/mobile/)
  	  - [Meta Tech Podcast](https://open.spotify.com/show/1NlTm7OkZmcrOPfvlqlBMz)
    - [Square](https://code.cash.app)
    - [Dropbox](https://dropbox.tech/mobile)
    - [Reddit](https://www.redditinc.com/blog/topic/technology)
    - [Airbnb](https://medium.com/airbnb-engineering/tagged/mobile)
    - [Lyft Tech Podcast](https://podcasts.apple.com/us/podcast/lyft-mobile/id1453587931)
    - [Curated List of Blog Posts](/BLOGPOSTS.MD)
- **Your resume:** List accomplishments with measurable impact.

### Things you cannot control
- **Interviewer's attitude.**
- **Competition.**
- **Hiring committee decisions.**

### Judging the outcome
You can influence, but not control, the outcome. Don't let setbacks affect your self-worth.

## Frequently Asked Questions

### How do you know this approach works? Why is this necessary?
- There's no guarantee that the suggested approach would work well in many cases - the structure of the system design round depends on a personal interviewer style.
- Having a good interview plan at hand allows both the interviewer and the candidate to concentrate more on the content of the discussion and not organizational aspects of the actual round.

### Can you go a bit deeper at X?
This is not necessary since there might be lots of alternative solutions and the guide does not provide the ground truth. The implementation details should depend on the personal experience of the candidate and not on an opinionated approach of some random people from the Internet.

### I'm an interviewer - this ruins the process for all of us: now the candidates just memorize the solutions to cheat during the interview.
- The system design is much more broad compared to coding rounds. Learning a particular solution is not nearly enough to be successful. The interviewer can slightly tweak the requirements to make it a brand new question.
- It's really obvious when the candidate memorized a certain approach instead of relying on experience.
- Learning some patterns and approaches before the interview might help the candidate to ease the stress and deliver the solution in a more clear and structured way.

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
