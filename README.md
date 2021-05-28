# A Simple Framework For Mobile System Design Interviews (work in progress)
Below is a simple framework for Mobile System Design interviews. As an example, we are going to use "Design Twitter Feed" question. The proposed solution is far from being perfect but it is not the point of a system design interview round: no one expects you to build a robust system in just 30 min - the interviewer is mosly looking for specific "signals" from your thought process and communication. Everything you say or do should help the interviewer to evaluate you as a candidate.

## Disclaimer
This might not be the only and/or the best way of structuring your system design interview. The framework was mostly inspired by similar "Scalable Backend Design" articles. Feel free to contribute. Use it at your own risk.

## Interview Process (45–60 min)
- 2–5 min - acquaintances
- 5 min - defining the task and gathering requirements
- 10 min - high-level discussion
- 20-30 min - detailed discussion
- 5 min - questions to the interviewer

## Acquaintances
Your interviewer tells you about themselves and you tell about yourself. It's better to keep it simple and short. For example, _"My name is Alex, I've been working on iOS/Android since 2010 - mostly on frameworks and libraries. For the past 2.5 years, I've been leading a team on an XYZ project: my responsibilities include system design, planning and technical mentoring."_ The only purpose of the introduction is to break the ice and provide a gist of your background. More time you spend on this - less time you gonna have for the actual interview.

## Defining The Task
The interviewer defines the task. For example: _"Design Twitter Feed"_.  Your first step is to figure out the scope of the task:
- **Client-side only** -  just a client-app: you have the backend and API available.
- **Client-side + API**  -  likely choice for most interviews: you need to design a client app and API.
- **Client-side + API + Back-end** -  less likely choice since most mobile engineers would not have a proper backend experience. If the interviewer asks server-side questions - let them know that you're most comfortable with the client-side and don't have hands-on experience with backend infrastructure. It's better to be honest than trying to fake your knowledge. If they still persist - let them know that everything you know comes from books, YouTube videos, and blog posts.

## Gathering Requirements
Task requirements can be split into **functional**, **non-functional** and **out of scope**.  Will define them in the scope of "Design Twitter Feed" task.
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

## Providing the "signal"
System design questions are made ambiguous. The interviewer is more interested in seeing your thought process than the actual solution you produce:
- What features did you choose?
- What clarifying questions did you ask?
- What concerns and trade-offs did you mention?

Your best bet is to ask many questions and cover as much ground as possible. Make it clear that you're starting with the BFS approach and ask them which feature or component they want to dig in. Some interviewers would let you choose which topics you want to discuss :  pick something you know best.

## Clarifying Questions
Here some of the questions you might ask during the task clarification step
- **Do we need to support Emerging Markets?**  
Publishing in a developing country brings additional challenges. The app size should be as small as possible due to the widespread use of low-end devices and higher cost of cellular traffic. The app itself should limit the number and the frequency of network requests and heavily rely on caching.
- **What number of users do we expect?**  
This seems like an odd question for a mobile engineer but it can be very important: a large number of clients results in a higher back-end load  -  if you don't design your API right , you can easily initiate a DDoS attack on your own servers. Make sure to discuss an Exponential Backoff and API Rate-Limiting with your interviewer.
- **How big is the engineering team?**  
This question might make sense for senior candidates. Building a product with a smaller team (2–4 engineers) is very different from building it with a larger team (20–100 engineers). The major concern is project structure and modularization. You need to design your system in a way that allows multiple people to work on without stepping on each other's toes.

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
Represents a set of components responsible for displaying a single tweet details.
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
After a high-level discussion your interviewer might steer the conversation towards some specific component of the system. Let's assume it was "Tweet Feed Flow". Things you might want to talk about:
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
### Real-time notification
We need to provide real-time notifications support as a part of the design. Bellow are some of the approached you can mention during the discussion:
- **Push Notifications**:
  - pros:
    - easier to implement compared to a dedicated service
    - can wake the app in the background
  - cons:
    - not 100% reliable
    - may take up to a minute to arrive
    - relies on a 3rd-party service
    - users can opt-out easily
- **HTTP-polling**  
Polling requires the client to periodically ask the server for updates. The biggest concern is the amount of unnecessary network traffic and increased backend load.
  - **short polling**: the client samples the server with a predefined time interval.    
    - pros:
      - simple and not as expensive (if the time between requests is long).
      - no need to keep a persistent connection.
    - cons:
      - the notification can be delayed for as long as the polling time interval.
      - addition overhead due to TLS Handshake and HTTP-headers 
  - **long polling**:
     - pros:
       - instant notification (no additional delay)
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
Provide a bi-directional communication between client and server
  - pros:
    - can transmit both binary and text data
  - cons:
    - more complex to set-up compared to Polling/SSE
    - keeps a persistent connection.

The interviewer would expect you to **pick a concrete approach** most suitable for the design task at hand. One possible solution for the "Design Twitter Feed" question could be using a combination of SSE (a primary channel of receiving real-time updates on "likes") with Push Notifications (sent if the client does not have an active connection to the backend).


### Protocols
- REST _TBD_
- GraphQL _TBD_
- WebSocket _TBD_
- gRPC _TBD_

### Pagination


## Providing the "signal"
The interviewer might be looking for the following signals:
- The candidate is aware of the challenges related to poor network conditions and expensive traffic.
- The candidate is familiar with most common protocols for unidirectional and bi-directional communication.
- The candidate is familiar with REST-full API design.
- The candidate is familiar with authentication and security best practices.
- The candidate is familiar with network error handling and rate-limiting.

## Additional topic
### Pagination
### Offline and Opportunistic State
### Caching
### Quality Of Service
To make your system more energy efficient you can introduce the Quality Of Service classes for your network operations. The implementation is quite tricky but you can discuss it on a higher level:
- Limit the number of concurrent network operations (4-10).
- Assign a Quality Of Service class to each of your network requests:
  - **User-critical** - should be dispatched as fast as possible: fetching the next page of data for the Tweet Feed; requesting Tweet Details.
  - **UI-critical**: - should be dispatched after User-critical requests: fetching low-res thumb images for tweets on the Feed while scrolling. Cancelled if the user scrolls past the target tweet. May be delayed in case of fast scrolling.
  - **UI-non-critical**: should be dispatched after UI-critical requests: fetching high-res images for tweets on the Feed. Cancelled if the user scrolls past the target tweet. May be delayed in case of fast scrolling.
  - **Background**: should be dispatched after all the above is finished: posting "likes", analytics.
- Introduce a priority queue for scheduling network requests: dispatch requests in the order of their priority. Suspend low-priority requests if the max number of concurrent operations is reached and a high-priority request is scheduled.
### Prefetching
## Additional Information

