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
    - Easier to implement.
    - Can wake the app in the background.
  - Cons:
    - Not 100% reliable.
    - May have delays.
    - Relies on 3rd-party service.
    - Users can opt-out.
- **HTTP-polling:**
  - Periodic requests for updates.  Can create excessive traffic and backend load.
    - **Short polling:** Fixed time interval.
      - Pros:
        - Simple, less expensive (if infrequent).
        - No persistent connection.
      - Cons:
        - Potential for significant delays.
        - Overhead from TLS handshake and HTTP headers.
    - **Long polling:**
      - Pros:
        - Instant notifications.
      - Cons:
        - More complex, requires more server resources.
        - Keeps a persistent connection.
- **Server-Sent Events (SSE):**
  - Stream events over HTTP without polling.
    - Pros:
      - Real-time updates with a single connection.
    - Cons:
      - Keeps a persistent connection.
- **WebSockets:**
  - Bi-directional communication.
    - Pros:
      - Can transmit binary and text data.
    - Cons:
      - More complex to set up.
      - Keeps a persistent connection.

Choose an appropriate approach for the task. For "Design Twitter Feed", a combination of SSE (primary channel for "like" updates) and Push Notifications (for clients without active connections) could be suitable.

### Protocols
#### REST
Text-based, stateless protocol for CRUD operations.
- Pros:
  - Easy to learn, understand, implement.
  - Easy to cache using HTTP caching.
  - Loose coupling.
- Cons:
  - Less efficient on mobile due to separate connections.
  - Schemaless – data validation is harder.
  - Stateless – requires extra session management.
  - Overhead from metadata and headers.

#### GraphQL
Query language allowing clients to request specific data from multiple resources with a single endpoint.
- Pros:
  - Schema-based, typed queries – data integrity checks.
  - Highly customizable – reduces HTTP traffic.
  - Bi-directional communication with GraphQL Subscriptions (WebSocket-based).
- Cons:
  - More complex backend.
  - "Leaky abstraction" – tight client-backend coupling.
  - Query performance depends on the slowest backend service (for federated data).

#### WebSocket
Full-duplex communication over a single TCP connection.
- Pros:
  - Real-time, bi-directional communication.
  - Supports text and binary data.
- Cons:
  - Requires maintaining an active connection.
  - Schemaless – data validation is harder.
  - Limited number of active connections per server (65k).

Learn more:
- [WebSocket Tutorial - How WebSockets Work](https://www.youtube.com/watch?v=pNxK8fPKstc)

#### gRPC
Remote Procedure Call framework over HTTP/2. Supports bi-directional streaming.
- Pros:
  - Lightweight binary messages (smaller than text-based).
  - Schema-based – built-in code generation with Protobuf.
  - Supports event-driven architecture: server/client/bi-directional streaming.
  - Multiple parallel requests.
- Cons:
  - Limited browser support.
  - Non-human-readable format.
  - Steeper learning curve.

Select an appropriate protocol. REST is suitable for "Design Twitter Feed" due to its simplicity.

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
