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
_TBD_: Socket connection active only while the app is in the foreground. Disconnected in the background. Push notification to notifiy user about new messages.

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
