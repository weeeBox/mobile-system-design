# Mobile System Design Exercise: Chat Application
The task might be simply defined as "Design Chat Application". The definition is purposely vague and requires some clarification.

## Gathering Requirements
You are expected to ask clarifying questions and narrow down the scope of the task. This should not take more than 5 minutes.

> **Candidate**: "Are we designing a general usage app (like WhatsApp) or something more specific (like an internal chat app for a company)?"  
> **Interviewer**: "Something like WhatsApp."  

> **Candidate**: "Do we have any information on the number of monthly active users?"  
> **Interviewer**: "Why is this important for a mobile app?"  
> **Candidate**: "We need to make sure that our clients won't accidentally create a DDoS attack on our backend services."  

> **Interviewer**: "In what cases a DDoS _attack_ is possible?"  
> **Candidate**: "The most likely reason is a frequent HTTP-polling that produces large volumes of network traffic."  
> **Candidate**: "Another reason - not using an exponential backoff while retrying failing requests. Million of clients retrying their requests at the same time can produce an unnecessary load to our backend services."  
> **Interviewer**: "Got it. Let's say we're expecting to have 1,000,000 monthly active users."  

> **Candidate**: "Do we know anything about our target market and the audience?"  
> **Interviewer**: "Why do you think this is important?"  
> **Candidate**: "An app targetted primarily for North America and Europe might have a different set of performance requirements compared to a similar app targetted to developing countries. The users in developed countries usually have more powerful phones compared to their peers in developing countries. The cost of cellular traffic in Europe and North America is much lower than in India."  
> **Interviewer**: "Let's design an app for North America."  

> **Candidate**: "Do we want to support group chats?"  
> **Interviewer**: "No, let's leave it out of scope."  

> **Candidate**: "Do we need to support sign-up and log-in?"  
> **Interviewer**: "We can leave it out of scope."  

Make sure not to overload the system requirements with unnecessary features. Think in terms of MVP (Minimum Viable Product) and pick features that have the biggest value. You can learn more about requirements gathering [here](https://github.com/weeeBox/mobile-system-design#gathering-requirements).

### Functional requirements
- User should be able to see the list of recent chats sorted by date.
- User should be able to open a 1-on-1 chat, send and receive messages.
- User should be able to add photo attachments to chat messages.
- User should be able to see chat message status (sending, sent/failed, received) and read receipts.

### Non-functional requirements
- Offline support (reading chat messages).
- Secure message storage.
- Real-time notifications.

### Out of scope
- Video and voice attachments.
- Sign-up and log-in.
- Group chats.

_NOTE: It is tempting to include additional requirements such as 'last seen`, message replies/forwarding, reactions, etc. Proper time management is crucial during the interview - try keeping everything closer to an MVP and move on to the next section._

## High-Level Diagram
A high-level diagram shows all major system components and their interactions (without implementation details). You can learn more about high-level diagrams [here](https://github.com/weeeBox/mobile-system-design#high-level-diagram).

![High-level Diagram](/images/exercise-chat-application-high-level-diagram.svg)

_NOTE: This diagram looks familiar to other cases from the guide. The main reason is the adopted architectural pattern combination MVx + repository + coordinator._

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
A single source of truth. The data your system receives gets persisted on the disk and then propagated to other components.
- **Repository**  
A mediator component between API Service and Persistence.
- **Chat Lobby**  
Represents a set of components responsible for displaying a list of recent chats.
- **Chat Room**  
Represents a set of components responsible for displaying a single chat room.
- **DI Graph**  
Dependency injection graph. See: [Dependency injection in Android](https://developer.android.com/training/dependency-injection) and [Dependency injection in iOS](https://www.raywenderlich.com/14223279-dependency-injection-tutorial-for-ios-getting-started).
- **Image Loader**  
Responsible for loading and caching static images. Usually represented by a 3rd-party library.
- **Coordinator**  
Organizes flow logic between Chat Lobby and Chat Room components. Helps decouple components of the system from each other.
- **App Module**  
An executable part of the system which "glues" components together.

## Deep Dive
After a high-level discussion, your interviewer might want to discuss some specific components of the system. Make sure to keep your explanation brief and don't overload it with details - let your interviewer guide the conversation and ask questions instead. You can learn more about deep-dive discussions [here](https://github.com/weeeBox/mobile-system-design#deep-dive-tweet-feed-flow).  

## Deep Dive: API Service

> **Interviewer**: "Can you tell me more about the API Service component?"  
> **Candidate**: "It's an abstraction for the network communication layer."  
> **Candidate**: "The idea is to isolate the low-level transport primitives from the rest of the app. This way, we can better support modularity and testability of the project."  

> **Interviewer**: "What protocols would you use for the network communication?"  
> **Candidate**: "I would split everything into three major components:"  

> **Candidate**: "1) Real-time bi-directional communication layer for sending/receiving chat messages and status. We need to decide between connection-based (TCP) and connectionless (UDP) protocols."  
> **Candidate**: "TCP-based clients establish a virtual connection and provide an ordered delivery guarantee by re-transmitting lost packets. It might be more expensive in terms of battery life (especially on flaky networks where the participants need to frequently repeat handshakes to restore lost connection). Another disadvantage is a 64k limit for the connection count for any given host port and a bigger packet header size compared to UDP.  Some examples of TCP-based protocols: WebSocket (Slack), XMPP (WhatsApp, Zoom, Google Talk), MQTT (IoT, Smart Home, etc)."  
> **Candidate**: "UDP-based clients are more lightweight and don't require any handshakes. As a result, UDP can't provide an ordered delivery guarantee and has no error checking beyond simple checksums. Some examples of UDP-based protocols: WebRTC (Discord, Google Hangouts, Facebook Messenger) - also works over TCP."  

_NOTE: You can learn more about the differences between TCP and UDP [here](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html)._

> **Candidate**: "I don't have much experience working with real-time protocols so I would select something I know best - WebSockets. Some disadvantages of this choice - the protocol is schemeless and does not provide automatic reconnection. There are also several [security flaws](https://www.neuralegion.com/blog/websocket-security-top-vulnerabilities). Alternatively, we can use HTTP-polling - might be a poor choice since it would generate an excessive amount of backend traffic."  
> **Candidate**: "We can also use gRPC (bi-directional streaming) or GraphQL (subscriptions), but I don't have enough experience with them so let's stick with WebSockets."  

_NOTE: It is better not to bring unfamiliar technologies to the discussion. You can mention some of their advantages - but don't go too deep unless you're prepared to discuss implementation-specific details._  

> **Interviewer**: "How would you keep a socket connection when the app goes to the background?"  
> **Candidate**: "I don't need to keep it alive while the app is not active - a push notification can be used to notify the user about new messages and bring the app back to the foreground."  
> **Interviewer**: "Sounds good."  

_NOTE: It's tempting for iOS engineers to mention `URLSessionWebSocketTask` as a solution for bi-directional communication. But it's also important to remember that the API is not available until iOS 13. Make sure to keep OS version compatibility in mind._  

> **Candidate**: "2) Request-response layer for fetching a paginated list of chats or message history. REST protocol could be a good choice since we don't need much of the request customization to bring up GraphQL (and have an extra complexity on the backend side)."  

_NOTE: Don't over-complicate your design - aim to cover more ground (unless the interviewer wants to dig deeper into the protocols)._  

> **Candidate**: "3) Cloud Messaging layer to receive push notifications while the app is not active."

![API Service Diagram](/images/exercise-chat-application-api-service-diagram.svg)

> **Candidate**: "I would introduce separate interfaces (protocols) to abstract communication between the network layer and the business logic: `ChatLobbyService` (receive the list of chats, create/delete chats) and `ChatRoomService` (receive the message history, send/receive messages, download/upload attachments)."  

> **Interviewer**: "Why do you need to have separate interfaces?"  
> **Candidate**: "To decouple different modules of the app. For example, the `Chat Lobby` flow can receive the `ChatLobbyService` instance from DI and only use the public API contract without knowing anything about the underlying network module implementation. It also makes development/testing easier since developers can swap real implementation with fake instances and serve predefined data from disk or memory."  

> **Candidate**: "I would also use a `Model Converter` layer to turn network response classes into model classes to hide protocol/implementation details and perform simple data transformations."  
> **Interviewer**: "Not sure if I'm following - can you elaborate?"  

> **Candidate**: "Imagine that you received a data frame containing a chat message and deserialized it into an object."  

```
ChatMessageData:
+ id: String
+ user_id: String
+ text: String
+ status: String
+ created_at: Long
+ attachments: String // comma-separated list
```

> **Candidate**: "You can convert it into a model object."  

```
ChatMessage
+ id: String
+ userId: String
+ text: String
+ status: ChatMessageStatus
+ createdAt: Date
+ attachments: [Attachments]
```

> **Candidate**: "This way the business logic does not need to know about the network layer and the data format of the transport protocols. You can change the networking implementation without affecting the rest of the app."  

## Deep Dive: Data Model

> **Interviewer**: "How would you organize your data storage?"  
> **Candidate**: "I would use a relational database (ORM) to store chats, messages, users, and attachments."  
> **Candidate**: "Other alternatives might be less robust since we're looking for querying support and data integrity."  

_NOTE: Don't waste time describing approaches that obviously won't fit the task at hand (like app preferences, text/binary files, etc)._

> **Candidate**: "I would use a relational database (ORM) and create tables for chats, messages, users, and attachments."  

![High-level Diagram](/images/exercise-chat-application-data-model.svg)

_NOTE: This diagram loosely follows the entity-relationship diagram notation. The cardinality/ordinality relationships are purposely omitted to save the interview time. A good rule of thumb is to avoid unnecessary drawings altogether._

> **Candidate**: "The `User` table is pretty straightforward: we would store user id, name, and profile picture URL."  

> **Candidate**: "The `Chat` table would contain the list of active chats: we would store the chat id, the id of the user who sent the last message, and the last message id."  

> **Candidate**: "The `Message` table would contain all messages from all chats: we would store the message id, the user id, the chat id, the message status (`PENDING`, `SENT`, `READ`, `FAILED`), and a comma-separated list of attachment ids."  

> **Candidate**: "The `Attachment` table would contain all attachments for all messages from all chats: we would store the attachment id, the image URLs (full-size and the low-res), the cached local versions paths, the chat id, and the attachment status (`UPLOADING`, `DOWNLOADING`, `FAILED`, `READY`)

> **Candidate**: "We can join the `Chat` table with the `Message` table (on the `last_message_id` column) and with the `User` table (on the `last_user_id` column) to get a list of recent chats. Each result could be represented by the following model class:"  

```
ChatInfo:
+ chatId: String
+ lastUsername: String
+ lastUserProfileUrl: Url
+ lastMessageText: String
+ lastMessageTimestamp: Date
```

> **Candidate**: "We can get the list of messages for a specific chat by selecting on the `chat_id` column."  

> **Interviewer**: "What would you use for message and attachment ids?"  
> **Candidate**: "What do you mean?"  

> **Interviewer**: "Are those local ids or server ids?"  
> **Candidate**: "We can use client-generated 128-bit UUIDs. The outgoing messages can be identified by `user_id` and the outgoing attachment by empty `url` columns: once an attachment is uploaded - it's undistinguished from a remote attachment."  
> **Candidate**: "The advantage of such approach is its simplicity and built-in [idempotency](https://stripe.com/docs/idempotency); the disadvantage - clients would generate resource ids which is less reliable and less secure compared to the backend generation."  

> **Candidate**: "Alternatively, we can maintain separate local and server ids: all local operations would be using local ids while the backend communication would use server ids. We would also need to build a bijection between them."  

_NOTE: Learn more about Local and Servier ids [here](https://tech.trello.com/sync-two-id-problem/)._

## Follow-up Questions
Some interviewers might ask follow-up questions that might change the original design and introduce new requirements.  

_TBD_

## Major Concerns and Trade-Offs
_TBD_

## Conclusion
Keep this in mind while preparing for a system design interview:
- Don't try to make it perfect - provide a "signal" instead.
- Listen to your interviewer and keep track of the time.
- Try to cover as much ground as possible without digging too much into the implementation details.

## Looking for more content?
I’m thinking about creating an in-depth mobile system design course on top of the free articles. Please, fill out this [form](https://forms.gle/KfvmZhPNPMRBE8Jj9) to express your interest!
