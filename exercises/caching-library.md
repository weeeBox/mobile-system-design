# Mobile System Design Exercise: Caching Library
The task might be simply defined as "Design Caching Library Library". The definition is purposely vague and requires some clarification.

## Gathering Requirements
You are expected to ask clarifying questions and narrow down the scope of the task. This should not take more than 5 minutes.

> **Candidate**: "Are we designing a part of an application or a general-purpose library?"  
> **Interviewer**: "A general-purpose library."  
 
> **Candidate**: "What kind of items are we going to cache?"  
> **Interviewer**: "Raw bytes."  

> **Candidate**: "Do we have a size limit for each entry and an upper bound for the total number of items?"  
> **Interviewer**: "Generally, no. But, for the sake of simplicity, let's assume image-like data up to 10 Mb per item. The max number of items is in order of thousands."  

> **Candidate**: "Do we need to handle sensitive data? Should we support encrypted storage?"  
> **Interviewer**: "Great question! What's your recommendation?"  
> **Candidate**: "Having access to encrypted storage might be useful for specific use-cases. However, doing it in a general case might lead to an unnecessary overhead (encryption/decryption comes with CPU and memory cost)."  
> **Candidate**: "We can allow turning encryption on/off on a library level or for certain specific items and let users decide based on their user-cases."  
> **Interviewer**: "Sounds good. Let's leave it out of scope for now."  

> **Candidate**: "How are we going to identify each cached item? Would string keys suffice?"  
> **Interviewer**: "String keys are fine."  

> **Candidate**: "Do we need to support an in-memory cache, a disk cache, or a hybrid memory-and-disk cache?"  
> **Interviewer**: "Good question. How would the 'hybrid' approach work?"  
> **Candidate**: "The cache would use the disk as a primary storage and keep a subset of items in memory for a quicker access."  
> **Interviewer**: "Let's use the 'hybrid' approach."  

> **Candidate**: "Should we limit the size of the cache?"  
> **Interviewer**: "What's your recommendation?"  
> **Candidate**: "I think, we should limit the number of bytes stored in memory and on the disk. Once the specific limit is reached - we should remove a portion of items according to the eviction policy."  

> **Interviewer**: "What eviction policy are you going to use?"  
> **Candidate**: "Since we're designing a general-purpose library - we should let the user select an eviction policy based on app-specific needs."  
> **Candidate**: "We can start with pre-defined policies: Least Recently Used (LRU), Least Frequently Used (LFU), First In First Out (FIFO), Random, etc."  
> **Candidate**: "We can also allow user-defined eviction policies using the [Strategy](https://en.wikipedia.org/wiki/Strategy_pattern) design pattern. Although, we should probably leave it out of scope for now."  
> **Interviewer**: "Sounds good. Let's leave user-defined policies out of scope."  

Make sure not to overload the system requirements with unnecessary features. Think in terms of MVP (Minimum Viable Product) and pick features that have the biggest value. You can learn more about requirements gathering [here](https://github.com/weeeBox/mobile-system-design#gathering-requirements).

## Functional requirements
- Users should be able to cache and retrieve raw byte data using strings as keys.
- Users should be able to configure disk and memory usage limits as a part of library initialization.
- Users should be able to configure the cache eviction policy as a part of library initialization.

## Non-functional requirements
- The cached data should be persistent on the disk.
- A small subset of items should be kept in memory for quicker access.
- Once the cache is full - a portion of items should be deleted according to the eviction policy.

## Out of scope
- User-defined eviction policies.
- Secure item storage.

## Client Public API (Optional)
Your interviewer might want to discuss the client public API. In the simplest form it might look like this:

```
TBD
```

## High-Level Diagram
A high-level diagram shows all major system components and their interactions (without implementation details). You can learn more about high-level diagrams here.
_TBD_

## Deep Dive
After a high-level discussion, your interviewer might want to discuss some specific components of the system. Make sure to keep your explanation brief and don't overload it with details - let your interviewer guide the conversation and ask questions instead. You can learn more about deep-dive discussions [here](https://github.com/weeeBox/mobile-system-design#deep-dive-tweet-feed-flow).

_TBD_

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
