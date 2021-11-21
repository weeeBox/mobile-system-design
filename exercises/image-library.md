# Mobile System Design Exercise: Image Library [work in progress]
The task might be simply defined as "Design Image Library". The definition is purposely vague and requires some clarification.

## Gathering Requirements
You are expected to ask clarifying questions and narrow down the scope of the task. This should not take more than 5 minutes.

> **Candidate**: "Are we designing a part of an application or a general-purpose library?"  
> **Interviewer**: "A general-purpose library."  

> **Candidate**: "Are we designing a library that loads images or handles a collection of photos?"  
> **Interviewer**: "The one that loads images."  

> **Candidate**: "Where do we need to load images from?"  
> **Interviewer**: "I don't know - you tell me."  

_NOTE: some interviewers may want you to "drive" the conversation and would "subtract points" if you try to ask them what you need to do._

> **Candidate**: "Network, filesystem, and app bundle would make sense. We can expose an API to create custom loaders but we could leave it out of scope for now."  
> **Interviewer**: "Yes, let's leave it out of scope."  

> **Candidate**: "Do we need to support UI-targets?"  
> **Interviewer**: "Not sure what you mean."  
> **Candidate**: "Do we want the library to load images directly to UI components (like ImageView)?"  
> **Interviewer**: "Yes."  
> **Candidate**: "I think, it might be beneficial to support non-UI targets - like saving the image to a file or providing a callback to access the image data when loading is complete."  

Make sure not to overload the system requirements with unnecessary features. Think in terms of MVP (Minimum Viable Product) and pick features that have the biggest value. You can learn more about requirements gathering [here](https://github.com/weeeBox/mobile-system-design#gathering-requirements).

## Functional requirements
- Users should be able to load images.

## Non-functional requirements
- _TBD_

## Out of scope
- _TBD_

## Client Public API (Optional)
Your interviewer might want to discuss the client public API. In the simplest form it might look like this:

```
TBD
```

## High-Level Diagram
A high-level diagram shows all major system components and their interactions (without implementation details). You can learn more about high-level diagrams [here](https://github.com/weeeBox/mobile-system-design#high-level-diagram).

![High-level Diagram](/images/exercise-image-library-high-level-diagram-simplified.svg)

### Components:
- _TBD_

## Deep Dive
After a high-level discussion, your interviewer might want to discuss some specific components of the system. Make sure to keep your explanation brief and don't overload it with details - let your interviewer guide the conversation and ask questions instead. You can learn more about deep-dive discussions [here](https://github.com/weeeBox/mobile-system-design#deep-dive-tweet-feed-flow).  

## Follow-up Questions
Some interviewers might ask follow-up questions that might change the original design and introduce new requirements.  

_TBD_

## Major Concerns and Trade-Offs

_TBD_

## Conclusion
- Keep this in mind while preparing for a system design interview:
- Don't try to make it perfect - provide a "signal" instead.
- Listen to your interviewer and keep track of the time.
- Try to cover as much ground as possible without digging too much into the implementation details.

## Looking for more content?
I’m thinking about creating an in-depth mobile system design course on top of the free articles. Please, fill out this [form](https://forms.gle/KfvmZhPNPMRBE8Jj9) to express your interest!
