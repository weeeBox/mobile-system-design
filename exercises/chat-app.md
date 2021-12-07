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

### Components:
_TBD_

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

### Concerns
__TBD__: Keeping a persistent connection that must be re-established while moving between cellular towers.

## Deep Dive: Data Model
![High-level Diagram](/images/exercise-chat-application-data-model.svg)

_NOTE: This diagram loosely follows the entity-relationship diagram notation. The cardinality/ordinality relationships are purposely omitted to save the interview time. A good rule of thumb is to avoid unnecessary drawings altogether._

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
