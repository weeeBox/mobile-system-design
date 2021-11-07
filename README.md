# A Simple Framework For Mobile System Design Interviews (work in progress)
Below is a simple framework for Mobile System Design interviews. As an example, we are going to use the "Design Twitter Feed" question. The proposed solution is far from being perfect but it is not the point of a system design interview round: no one expects you to build a robust system in just 30 min - the interviewer is mostly looking for specific "signals" from your thought process and communication. Everything you say or do should showcase your strengths and help the interviewer to evaluate you as a candidate.

## Disclaimer
The framework was heavily inspired by the similar "Scalable Backend Design" articles. Learning the framework does not guarantee passing the interview. The structure of the interview process depends on the personal style of the interviewer. The dialog is really important - make sure you understand what the interviewer is looking for. Make no assumptions and ask clarifying questions.

## Interview Process (45–60 min)
- 2–5 min - acquaintances
- 5 min - defining the task and gathering requirements
- 10 min - high-level discussion
- 20-30 min - detailed discussion
- 5 min - questions to the interviewer

## Acquaintances
Your interviewer tells you about themselves and you tell them about yourself. It's better to keep it simple and short. For example, _"My name is Alex, I've been working on iOS/Android since 2010 - mostly on frameworks and libraries. For the past 2.5 years, I've been leading a team on an XYZ project: my responsibilities include system design, planning and technical mentoring."_ The only purpose of the introduction is to break the ice and provide a gist of your background. The more time you spend on this, the less time you will have for the actual interview.

## Defining The Task
The interviewer defines the task. For example: _"Design Twitter Feed"_.  Your first step is to figure out the scope of the task:
- **Client-side only** -  just a client-app: you have the backend and API available.
- **Client-side + API**  -  likely choice for most interviews: you need to design a client app and API.
- **Client-side + API + Back-end** -  less likely choice since most mobile engineers would not have a proper backend experience. If the interviewer asks server-side questions - let them know that you're most comfortable with the client-side and don't have hands-on experience with backend infrastructure. It's better to be honest than trying to fake your knowledge. If they still persist - let them know that everything you know comes from books, YouTube videos, and blog posts.

## Gathering Requirements
Task requirements can be split into **functional**, **non-functional** and **out of scope**.  We'll define them in the scope of "Design Twitter Feed" task.
### Functional requirements
Think about 3–5 features that would bring _the biggest value to the business_.
- Users should be able to scroll through an infinite list of tweets.
- Users should be able to like a tweet.
- Users should be able to open a tweet and see comments (read-only).

### Non-functional requirements
Not immediately visible to a user but plays an important role for the product in general.
- Offline support.
- Real-time notifications.
- Optimal bandwidth and CPU/Battery usage.

### Out of scope
Features which will be excluded from the task but would still be important in a real project.
- Login/Authentication.
- Tweet sending.
- Followers/Retweets.
- Analytics.

## Providing the "signal"
System design questions are made ambiguous. The interviewer is more interested in seeing your thought process than the actual solution you produce:
- What assumptions did you make and how did you state them?
- What features did you choose?
- What clarifying questions did you ask?
- What concerns and trade-offs did you mention?

Your best bet is to ask many questions and cover as much ground as possible. Make it clear that you're starting with the BFS approach and ask them which feature or component they want to dig in. Some interviewers would let you choose which topics you want to discuss :  pick something you know best.

## Clarifying Questions
Here are some of the questions you might ask during the task clarification step
- **Do we need to support Emerging Markets?**  
Publishing in a developing country brings additional challenges. The app size should be as small as possible due to the widespread use of low-end devices and higher cost of cellular traffic. The app itself should limit the number and the frequency of network requests and heavily rely on caching.
- **What number of users do we expect?**  
This seems like an odd question for a mobile engineer but it can be very important: a large number of clients results in a higher back-end load  -  if you don't design your API right , you can easily initiate a DDoS attack on your own servers. Make sure to discuss an Exponential Backoff and API Rate-Limiting with your interviewer.
- **How big is the engineering team?**  
This question might make sense for senior candidates. Building a product with a smaller team (2–4 engineers) is very different from building it with a larger team (20–100 engineers). The major concern is project structure and modularization. You need to design your system in a way that allows multiple people to work on it without stepping on each other's toes.

## High-Level Diagram
Once your interviewer is satisfied with the clarification step and your choice for the system requirements  - you should ask if they want to see a high-level diagram. Below is a possible solution for the "Twitter Feed" question.
![High-level Diagram](/images/twitter-feed-high-level-diagram.svg)

### Server-side components:
- **Backend**  
Represents the whole server-sider infrastructure. Most likely, your interviewer won't be interested in discussing it.
- **Push Provider**  
Represents the Mobile Push Provider infrastructure. Receives push payloads from the Backend and delivers them to clients.
- **CDN (Content Delivery Network)**  
Responsible for delivering static content to clients.
### Client-side components:
- **API Service**  
Abstracts client-server communications from the rest of the system.
- **Persistence**  
A single source of truth. The data your system receives gets persisted on the disk first and then propagated to other components.
- **Repository**  
A mediator component between API Service and Persistence.
- **Tweet Feed Flow**  
Represents a set of components responsible for displaying an infinite scrollable list of tweets.
- **Tweet Details Flow**  
Represents a set of components responsible for displaying a single tweet's details.
- **DI Graph**  
Dependency injection graph. _TBD_.
- **Image Loader**  
Responsible for loading and caching static images. Usually represented by a 3rd-party library.
- **Coordinator**  
Organizes flow logic between Tweet Feed and Tweet Details components. Helps decoupling components of the system from each other.
- **App Module**  
An executable part of the system which "glues" components together.

## Providing the "signal"
The interviewer might be looking for the following signals:
- The candidate can present the "big picture" without overloading it with unnecessary implementation details.
- The candidate can identify the major building blocks of the system and how they communicate with each other.
- The candidate has app modularity in mind and is capable of thinking in the scope of the entire team and not limiting themselves as a single contributor (this might be more important for senior candidates).

## Deep Dive: Tweet Feed Flow
After a high-level discussion, your interviewer might steer the conversation towards some specific component of the system. Let's assume it was "Tweet Feed Flow". Things you might want to talk about:
- **Architecture patterns**: MVP, MVVM, MVI, etc. MVC is considered a poor choice these days. It's better to select a well known pattern since it makes it easier to onboard new hires (compared to some less known home-grown approaches).
- **Pagination**: essential for an infinite scroll functionality. For more details see Pagination.
- **Dependency injection**: helps building an isolated and testable module.
- **Image Loading**: low-res vs full-res image loading, scrolling performance, etc.

![Details Diagram](/images/tweet-details.svg)
## Components
- **Feed API Service** - abstracts Twitter Feed API client: provides the functionality for requesting paginated data from the backend. Injected via DI-graph.
- **Feed Persistence** - abstract cached paginated data storage. Injected via DI-graph.
- **Remote Mediator** - triggers fetching the next/prev page of data. Redirects the newly fetched paged response into a persistence layer.
- **Feed Repository** - consolidates remote and cached responses into a Pager object through Remote Mediator.
- **Pager** - trigger data fetching from the Remote Mediator and exposes an observable stream of paged data to UI.
- **"Tweet Like" and "Tweet Details" use cases** - provide delegated implementation for "Like" and "Show Details" operations. Injected via DI-graph.
- **Image Loader** - abstracts image loading from the image loading library. Injected via DI-graph.
## Providing the "signal"
The interviewer might be looking for the following signals:
- The candidate is familiar with most common MVx patterns.
- The candidate achieves a clear separation between business logic and UI.
- The candidate is familiar with dependency injection methods.
- The candidate is capable of designing self-contained isolated modules.

## API Design
The goal is to cover as much ground as possible - you won't have enough time to cover every API call - just ask the interviewer if they are particulary interested in a specific part, or choose something you know best (in case they don't have a strong preference).
### Real-time notification
We need to provide real-time notifications support as a part of the design. Bellow are some of the approaches you can mention during the discussion:
- **Push Notifications**:
  - pros:
    - easier to implement compared to a dedicated service.
    - can wake the app in the background.
  - cons:
    - not 100% reliable.
    - may take up to a minute to arrive.
    - relies on a 3rd-party service.
    - users can opt out easily.
- **HTTP-polling**  
Polling requires the client to periodically ask the server for updates. The biggest concern is the amount of unnecessary network traffic and increased backend load.
  - **short polling**: the client samples the server with a predefined time interval.    
    - pros:
      - simple and not as expensive (if the time between requests is long).
      - no need to keep a persistent connection.
    - cons:
      - the notification can be delayed for as long as the polling time interval.
      - additional overhead due to TLS Handshake and HTTP-headers 
  - **long polling**:
     - pros:
       - instant notification (no additional delay).
     - cons:
       - more complex and requires more server-side resources.
       - keeps a persistent connection until the server replies.
- **Server-Sent Events**  
Allows the client to stream events over an HTTP connection without polling.
  - pros:
    - real-time traffic using a single connection.
  - cons:
    - keeps a persistent connection.
- **Web-Sockets**:  
Provide a bi-directional communication between client and server.
  - pros:
    - can transmit both binary and text data.
  - cons:
    - more complex to set-up compared to Polling/SSE.
    - keeps a persistent connection.

The interviewer would expect you to **pick a concrete approach** most suitable for the design task at hand. One possible solution for the "Design Twitter Feed" question could be using a combination of SSE (a primary channel of receiving real-time updates on "likes") with Push Notifications (sent if the client does not have an active connection to the backend).

### Protocols
#### REST
A text-based stateless protocol - most popular choice for CRUD (Create, Read, Update, and Delete) operations.
- pros:
  - easy to learn, understand, and implement.
  - easy to cache using built-in HTTP caching mechanism.
  - loose coupling between client and server.
- cons:
  - less efficient on mobile platforms since every request requires a separate physical connection.
  - schemaless - it's hard to check data validity on the client.
  - stateless - needs extra functionality to maintain a session.
  - additional overhead - every request contains contextual metadata and headers.

#### GraphQL
A query language for working with API - allows clients to request data from several resources using a single endpoint (instead of making multiple requests in traditional RESTful apps).
- pros:
  - schema-based typed queries - clients can verify data integrity and format.
  - highly customizable - clients can request specific data and reduce the amount of HTTP-traffic.
  - bi-directional communication with GraphQL Subscriptions (WebSocket based).
- cons:
  - more complex backend implementation.
  - "leaky-abstraction" - clients become tightly coupled to the backend.
  - the performance of a query is bound to the performance of the slowest service on the backend (in case the response data is federated between multiple services).

#### WebSocket
Full-duplex communication over a single TCP connection.
- pros:
  - real time bi-directional communication.
  - provides both text-based and binary traffic.
- cons:
  - requires maintaining an active connection - might have poor performance on unstable cellular networks.
  - schemaless - it's hard to check data validity on the client.
  - the number of active connections on a single server is limited to 65k.
#### gRPC
Remote Procedure Call framework which runs on top of HTTP/2. Supports bi-directional streaming using a single physical connection.
- pros:
  - lightweight binary messages (much smaller compared to text-based protocols).
  - schema-based - built-in code generation with Protobuf.
  - provides support of event-driven architecture: server-side streaming, client-side streaming, and bi-directional streaming
  - multiple parallel requests.
- cons:
  - limited browser support.
  - non human-readable format.
  - steeper learning curve.

The interviewer would expect you to **pick a concrete approach** most suitable for the design task at hand. Since the API layer for the "Design Twitter Feed" question is pretty simple and does not require much customization - we can select an approach based on REST.

### Pagination
Endpoints that return a list of entities must support pagination. Without pagination, a single request could return a huge amount of results causing excessive network and memory usage.
#### Types of pagination
- **Offset Pagination**  
Provides a `limit` and an `offset` query parameters. Example: `GET /feed?offset=100&limit=20`.
  - pros:
    - easiest to implement - the request parameters can be passed directly to a SQL query.
    - stateless on the server.
  - cons:
    - bad performance on large offset values (the database needs to skip `offset` rows before returning the paginated result).
    - inconsistent when adding new rows into the database (Page Drift).
- **Keyset Pagination**  
Uses the values from the last page to fetch the next set of items. Example: `GET /feed?after=2021-05-25T00:00:00&limit=20`.
  - pros:
    - translates easily into a SQL query.
    - good performance with large datasets.
    - stateless on the server.
  - cons:
    - "leaky abstraction" - the pagination mechanism becomes aware of the underlying database storage.
    - only works on fields with natural ordering (timestamps, etc).
- **Cursor/Seek Pagination**  
Operates with stable ids which are decoupled from the database SQL queries (usually, a selected field is encoded using base64 and encrypted on the backend side). Example: `GET /feed?after_id=t1234xzy&limit=20`.
  - pros:
    - decouples pagination from SQL database.
    - consistent ordering when new items are inserted.
  - cons:
    - more complex backend implementation.
    - does not work well if items get deleted (ids might become invalid).

You need to select a single approach after listing the possible options and discussing their pros and cons. We'll pick Cursor Pagination in the scope of the "Design Twitter Feed" question. A sample API request might look like this:  
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

Although we left it out of scope, it's still beneficial to mention HTTP authentication. You can include an `Authorization` header and discuss how to properly handle `401 Unauthorized` response scenario. Also, don't forget to talk about Rate-Limiting strategies (`429 Too Many Requests`).  
Make sure to keep it brief and simple (without unnecessary details): your primary goal during a system design interview is to provide "signal" and not to build a production ready solution.

## Providing the "signal"
The interviewer might be looking for the following signals:
- The candidate is aware of the challenges related to poor network conditions and expensive traffic.
- The candidate is familiar with most common protocols for unidirectional and bi-directional communication.
- The candidate is familiar with REST-full API design.
- The candidate is familiar with authentication and security best practices.
- The candidate is familiar with network error handling and rate-limiting.

## Data Storage
### Data Storage Options
Bellow are the most common options for local device data storage:
- **Key-Value Storage (UserDefaults/SharedPreferences/Property List)**:
  - pros:
    - easy to use built-in API.
    - works best for simple, unstructured, non-sensitive data (settings, flags, etc).
  - cons:
    - insecure (Android provides [EncryptedSharedPreferences](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences); 3rd party wrapper libraries available on iOS).
    - not suitable for storing large data.
    - no schema support nor ability to query data.
    - no data migration support.
    - poor performance.
- **Database/ORM (sqlite/Room/Core Data/Realm/etc)**:
  - pros:
    - object–relational mapping support. 
    - schema and querying support.
    - data migration support.
  - cons:
    - more complex setup. 
    - insecure (wrapper libraries available on iOS/Android).
    - bigger memory footprint.
- **Custom/Binary (DataStore/NSCoding/Codable/etc)**:
  - pros:
    - highly customizable.
    - performant.
  - cons:
    - no schema/migration support.
    - lots of manual effort.
- **On-Device Secure Storage (Keystore/Key Chain)**:
  - pros:
    - secure (not 100% unless provided by the hardware).
  - cons:
    - not optimized for storing anything but encryption keys.
    - encryption/decryption performance overhead.
    - no schema/migration support.
### Storage Location
- **Internal**  
  Sand-boxed by the app and not readable to other apps (with few exceptions).
- **External**  
  Publicly visible and most likely not deleted when your app is deleted.
- **Media/Scoped**  
  Special type of storage for media files.
### Storage Type
- **Documents (Automatically Backed Up)**  
  User-generated data that cannot be easily re-generated and will be automatically backed up.
- **Cache**  
  Data that can be downloaded again or regenerated. Can be deleted by user to free-up space.
- **Temp**  
  Data that is only used temporary and should be deleted when no longer needed. 

### Best Practices
- Store as little sensitive data as possible.
- Use encrypted storage if you can't avoid storing sensitive data.
- Do not allow your app storage grow uncontrollably. Make sure that cleaning up cached files won't affect app functionality.

### Approach
You need to select a single approach after listing options and discussing their pros and cons. Don't worry about building a complete solution - just try to lay out a high-level idea without going too deep into details.  
A possible breakdown for the "Design Twitter Feed" question might look like this:
#### Feed Pagination Table
Create a "feed" database table for storing paginated feed response:
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
- Limit the total number of entries to 500 in order to control local storage size.
- Flatten attachments into a comma separated list. Alternatively, you can create an `attachments` table and join it with the `feed` table on `item_id`.
- Explicitly store next/prev cursor id with each item to simplify paged navigation.

#### Attachments Storage
- Store attachments as files in Internal Cached storage.
- Use attachment URLs as cache keys.
- Delete attachments after deleting corresponding items from the `feed` table.
- Limit the cache size to 200-400 Mb.

A possible follow-up question might require you to handle sensitive media data (private accounts, etc). In this case, you can selectively encrypt image files with encryption keys stored in Keystore/KeyChain.

## Providing the "signal"
The interviewer might be looking for the following signals:
- The candidate is aware of mobile storage types, security, limitations, and compatibility.
- The candidate is capable of designing a storage solution for most common scenarios.

## Additional topics
### Major Concerns For Mobile Development
Here's a list of concerns to keep in mind while discussing your solution with the interviewer:
- **User Data Privacy** - leaking customer's data can damage your business and reputation.
- **Security** - protect your products against reverse-engineering (more important for Android).
- **Target Platform Changes** - each new iOS/Android release may limit an existing functionality and make Privacy rules more strict.
- **Non-reversibility of released products** - assume everything you ship is final and would never change. Make sure to use staged rollouts and server-side "feature" flags.
- **Performance/Stability**
  - Metered data usage - cellular network traffic can be very expensive.
  - Bandwidth usage - constant waking up of the cellular radio results in a faster battery drain.
  - CPU usage - higher computational load results in a faster battery drain and device overheating.
  - Memory Usage - higher memory usage increases the risk of the app being killed in the background.
  - Startup Time - doing too much work at the app start creates poor user experience.
  - Crashers/ANRs - doing too much of work on main thread can lead to app shutdowns and UI-jank. App crashes is the leading factor of poor store ratings. 
- **Geo-Location Usage**
  - Don't compromise user privacy while using location services.  
  - Prefer the lowest possible location accuracy. Progressively ask for increased location accuracy if needed.  
  - Provide location access rationale before requesting permissions.  

### Privacy & Security
- Keep as little of the user's data as possible - don't collect things you won't need.
  - Avoid collecting device IDs (prefer one-time generated pseudo-anonymous IDs).
  - Anonymize collected data.
- Minimize the use of permissions
  - Be prepared for the user to deny permissions and respect the user's choice when they deny permission the second time.
  - Be prepared for the system to auto-reset permissions.
  - Be prepared for app hibernation of unused apps.
  - Delegate functionality to the 1st party apps (Camera, Photo Picker, File Manager, etc).
- Assume that on-device storage is not secure (even while using KeyStore/KeyChain functionality).
- Assume that the backend storage is also not secure - discussed possible end-to-end encryption mechanisms.
- Assume that the target platform's (iOS/Android) Security & Privacy rules will change - make critical functionality controllable by remote "feature" flags.
- User's _perception_ of security is as important as the implemented security measures - make sure to discuss how you would educate your customers about data collection, storage, and transmission.

#### Cloud vs On-Device
At some point during the interview you might need to choose between running some functionality on a device and moving it to a cloud. The approach you select would have the biggest impact when it comes to _On-Device AI_ but can also be extended to any kind of data processing as well.  

**Advantages of running things in a cloud:**
- Device independent - your customers are not limited by the characteristics of their devices.
- Better usage of client system resources - any intensive computation can be executed on the backend to save the device's battery.
- Fast pace of changes - you don't need to release an update to all the clients in order to get the desired functionality available.
- Much bigger computational resources - your backend can autoscale as the load pressure grows.
- Better security - the client code can be tampered and reverse engineered while the backend applications tend to be more secure.
- Easier analytics and offline data analysis - you gather all the data you need in your data centers.

**Advantages of running things on a device:**
- Better privacy - the data does not leave the user's device and is not stored on the cloud.
- Real-time functionality - certain operations can run much faster on a user's device than being sent to a backend.
- Lower bandwidth usage - you don't need to send data over the network.
- Offline functionality - the client does not need a persistent network connection to operate.
- Lower backend usage - the load is splitted between all the clients and the backend.  

**Things you should never run on a device:**  
- New "resource" creation - generating coupons, tickets, etc.
- Transactions and Payment verification - unless it's delegated to a 3rd-party SDK.

### Offline and Opportunistic State
_TBD_
### Caching
_TBD_
### Quality Of Service
To make your system more energy efficient you can introduce the Quality Of Service classes for your network operations. The implementation is quite tricky but you can discuss it on a higher level:
- Limit the number of concurrent network operations (4-10). The number itself may depend on the device state (battery/wall charger, WiFi/cellular, doze/standby, etc).
- Assign a Quality Of Service class to each of your network requests:
  - **User-critical** - should be dispatched as fast as possible: fetching the next page of data for the Tweet Feed; requesting Tweet Details.
  - **UI-critical**: - should be dispatched after User-critical requests: fetching low-res thumb images for tweets on the Feed while scrolling. Cancelled if the user scrolls past the target tweet. May be delayed in case of fast scrolling.
  - **UI-non-critical**: should be dispatched after UI-critical requests: fetching high-res images for tweets on the Feed. Cancelled if the user scrolls past the target tweet. May be delayed in case of fast scrolling.
  - **Background**: should be dispatched after all the above is finished: posting "likes", analytics.
- Introduce a priority queue for scheduling network requests: dispatch requests in the order of their priority. Suspend low-priority requests if the max number of concurrent operations is reached and a high-priority request is scheduled.
### Resumable Uploads
A resumable (chunked) media upload breaks down a single upload request in three stages:
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
Prefetching improves app performance by hiding the data transfer latency over slow and unreliable networks. The biggest concern while designing your prefetching solution is the increase of battery and cellular data usage.
_TBD_
#### More Info:
- [Optimize downloads for efficient network access](https://developer.android.com/training/efficient-downloads/efficient-network-access)
- [Improving performance with background data prefetching](https://instagram-engineering.com/improving-performance-with-background-data-prefetching-b191acb39898)
- [Informed mobile prefetching](https://dl.acm.org/doi/10.1145/2307636.2307651)
- [Understanding prefetching and how Facebook uses prefetching](https://www.facebook.com/business/help/1514372351922333)
## Conclusion
There's a significant amount of randomness during a system design interview. The process and the structure can vary depending on the company and the interviewer.
### Things you can control
- **Your attitude** - always be friendly no matter how the interview goes. Don't be pushy and don't argue with the interviewer - this might provide a bad "signal".
- **Your preparation** - the better your preparation is, the bigger the chance of a positive outcome. Practice mock design interviews with your peers (you can find people on [Teamblind](https://www.teamblind.com/search/mock)).
- **Your knowledge** - the more knowledge you have, the better your chances are.
  - Gain more experience.
  - Study popular open source projects: [iOS](https://github.com/search?q=iOS&type=Repositories), [Android](https://github.com/search?o=desc&q=Android&s=stars&type=Repositories)
  - Read development blogs from tech companies:
    - [Uber](https://eng.uber.com/category/articles/mobile/)
    - Instagram: [iOS](https://instagram-engineering.com/tagged/ios), [Android](https://instagram-engineering.com/tagged/android)
    - [Trello](https://tech.trello.com/)
    - _TBD_
- **Your resume** - make sure to list all your accomplishments with measurable impact.
### Things you cannot control
- **Your interviewer's attitude** - they might have a bad day or simply dislike you.
- **Your competition** - sometimes there's simply a better candidate.
- **The hiring committee** - they make a decision based on the interviewers' report and your resume.
### Judging the outcome
You _can influence_ the outcome but you _can't control_ it. Don't let minor setbacks determine your self-worth.
## Additional Information
_TBD_
## Consider "starring" the repository to help other people discover the guide! Thank you!
