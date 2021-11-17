# Mobile System Design Exercise: Caching Library [work in progress]
The task might be simply defined as "Design Caching Library". The definition is purposely vague and requires some clarification.

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

> **Interviewer**: "Do we need to share the library between multiple platforms?"  
> **Candidate**: "No. Let's leave it out of scope."  

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
- Cross-platform support.

## Client Public API (Optional)
Your interviewer might want to discuss the client public API. In the simplest form it might look like this:

```
Cache:
+ init(config: CacheConfig)
+ set(key: String, value: [byte]): CacheTask
+ get(key: String): CacheTask
+ clear(key: String): CacheTask
+ clearAll()

CacheConfig:
+ init(maxMemoryCacheSize: Int, maxDiskCacheSize: Int)

CacheTask:
+ isSuccessful(): Bool
+ getCachedData(): [byte]?
+ getErrorMessage(): String?
+ addOnCompleteCallback(callback: (CacheTask) → Void)
```

> **Interviewer**: "Why do you return a `CacheTask` from `get` and `set` methods?"  
> **Candidate**: "`get` and `set` calls may result in blocking I/O operations and should not be invoked on the main thread."  
> **Candidate**: "We should make both of them async and let users specify completion callbacks."  

_NOTE_: in this exercise, the API description is purposely left platform/language agnostic to reach a broader audience. During an actual interview, you would most likely design for either iOS or Android, and select the API style most suitable for the target platform. For example, you can advocate for coroutines instead of callbacks, suggest Rx-extensions, etc.  

## High-Level Diagram
A high-level diagram shows all major system components and their interactions (without implementation details). You can learn more about high-level diagrams [here](https://github.com/weeeBox/mobile-system-design#high-level-diagram).

![High-level Diagram](/images/exercise-caching-library-high-level-diagram.svg)

### Components:
- **Dispatcher** - coordinates async read/write operations between the client and the library.  
- **Journal** - maintains a structural record of metadata for cached items (access count, timestamps, size, etc).  
- **In-Memory Cache** - handles fast in-memory item storage for quicker sequential access.  
- **Persistent Store** - handles slow persistent disk item storage.  
- **Cache Eviction** - manages cache overflow and item eviction.  

## Deep Dive
After a high-level discussion, your interviewer might want to discuss some specific components of the system. Make sure to keep your explanation brief and don't overload it with details - let your interviewer guide the conversation and ask questions instead. You can learn more about deep-dive discussions [here](https://github.com/weeeBox/mobile-system-design#deep-dive-tweet-feed-flow).  

## Deep Dive: Dispatcher
> **Interviewer**: "I’m not sure what the Request Dispatcher component does…"  
> **Candidate**: "It simplifies handling read/write concurrency."  
> **Candidate**: "Using synchronous API calls might be a poor idea since item access might result in a blocking I/O operation. Doing so on the Main thread is not advisable and may result in jank, app termination, and _interview failures_."  
> **Candidate**: "To solve this issue, we need to use async API. There might be a few options (depending on the platform and the programming language) but the callback-base approach is the simplest one and can be extended to coroutines, futures, promises, etc."  

![Dispatcher Sequence Diagram](/images/exercise-caching-library-dispatcher-sequence.svg)

> **Candidate**: "1) Dispatcher would maintain a _limited_ pool of concurrent workers (via executor, dispatch queue, etc) to avoid creating too many background threads if the client makes a huge number of requests."  
> **Candidate**: "2) Each worker would perform a cache access operation and notify its dispatcher via a callback when complete."  
> **Candidate**: "3) After a worker completes - the dispatcher would notify the client by invoking the completion callback on the main thread (or user-specified executor/queue)."  

## Deep Dive: Journal
> **Interviewer**: "Would you explain how the Journal component works?"  
> **Candidate**: "The Journal component creates and updates metadata records for cached items."  

<div align="center">
 
|     name      |  type  |
|---------------|--------|
| key           | String |
| size_bytes    | Int    |
| access_count  | Int    |
| last_accessed | Date   |

</div>

> **Candidate**: "Every time a cached item is accessed - the Journal will increase the `access_count` and update the `last_accessed` timestamp. This metadata would allow selecting items for eviction and calculating the total cache size."  
> **Candidate**: "The Journal itself could be stored in a text/binary file, property list, or a relational database (ORM). I think a relational database might be the best approach since it allows partial updates, querying, and provides data integrity."  

> **Interviewer**: "So the Journal would only store the metadata - where would the actual bytes be stored?"  

### First Option: File System
> **Candidate**: "We can write binary files in the app 'cache' directory and use generated [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)'s as file names. The item key won't work as a filename since the user may pass illegal path characters."  
> **Candidate**: "One drawback of such approach - we would need to sync the Journal and the file system separately which might lead to race condition bugs. For example, the host app might crash after the Journal is updated but before the data is written to the disk."  
> **Candidate**: "We may overcome this by introducing items states: `CLEAN`, `DIRTY`, `REMOVE`, etc. When the item is being created or updated - it will be marked with `DIRTY` state; once the process completes - it will be marked with `CLEAN` state; same for `REMOVE` state. The library may check the Journal upon initialization and prune dirty items and their files."  

<div align="center">
 
|     name      |  type  |
|---------------|--------|
| key           | String |
| path          | String |
| size_bytes    | Int    |
| access_count  | Int    |
| last_accessed | Date   |
| state         | Int    |

</div>

> **Candidate**: "The introduction of states makes it more reliable but won't solve all potential concurrency issues. We might still run in situations where different threads are trying to write the same item concurrently. We may try to add more synchronization but it would greatly complicate the design."  

> **Candidate**: "The advantage of this option - the app cache (with all binary files) might be cleaned up by the user from the app settings when device storage is low."  

### Second Option: BLOBs
> **Candidate**: "We can store the binary data in the same database where the Journal data is stored."  

<div align="center">
 
|     name      |  type  |
|---------------|--------|
| key           | String |
| data          | BLOB   |
| size_bytes    | Int    |
| access_count  | Int    |
| last_accessed | Date   |

</div>

> **Candidate**: "This way we don't need to worry about synchronization and item states. We can also merge the Journal and the Persistent Store components."  

![High-level Diagram](/images/exercise-caching-library-high-level-diagram-simplified.svg)

_NOTE: It's ok to change some of your initial statements as the interview progresses - it's more about the thinking process and not pretty diagrams nor "making it right" on the first attempt._  

> **Candidate**: "The disadvantage of this option - the binary data can't be cleaned from the app settings without removing the database itself and losing all its metadata."  
> **Candidate**: "There are lots of discussions around BLOBs vs file system. I don't have enough experience to take any side but I think the second option is an easier one."  

## Deep Dive: Cache Eviction

> **Interviewer**: "How does the Cache Eviction component work?"  
> **Candidate**: "When a new cache item is created - the component checks the disk and in-memory storage sizes; if any of them exceed the maximum allowed size - the Cache Eviction runs a prepared query to purge a sub-set of items."  

> **Interviewer**: "Why do you need a separate component and won't include the logic directly into the Journal and the In-Memory Cache?"  
> **Candidate**: "I'm trying to follow a [Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle): the Journal and the In-Memory Cache components should only perform a single function and should not be aware of cache eviction existence. It's also easier to test different components in isolation."  

> **Interviewer**: "You've mentioned _prepared queries_ - can you provide a bit more details?"  
> **Candidate**: "For the Journal component, it could be a prepared SQL statement, a fetch/batch request, or any other ORM mechanism to delete multiple items. Alternatively, we can iterate using a cursor (if the vendor supports it) and delete items until the new size can accommodate an extra item."  
> **Candidate**: "The drawback of this approach is that the Cache Eviction components might _know_ too much about other components implementation. At the same time - this is an efficient and a simple-to-build approach. Everything is a trade-off."  

> **Interviewer**: "What method would you choose?"  
> **Candidate**: "I think, a SQL cursor might be the most efficient since we don't need to load unnecessary items."  

> **Interviewer**: "Do you know how cursors work under the hood?"  
> **Candidate**: "Not really. I only worked with them and don't know the implementation details."  

> **Interviewer**: "How about the In-Memory eviction?"  
> **Candidate**: "For the In-Memory component, it could be a custom iterator over the collection which would perform a similar function."  

> **Interviewer**: "What data structure would you use for the in-memory store?"  
> **Candidate**: "I think a self-balancing tree with a custom item comparator should work."  

## Follow-up Questions
Some interviewers might ask follow-up questions that might change the original design and introduce new requirements.  

### Sensitive Information
> **Interviewer**: "How would you change your design to support handling sensitive information?"  
> **Candidate**: "Is every item considered to be sensitive?"  
> **Interviewer**: "Not necessary."  
> **Candidate**: "In this case, we might want to provide users some flexibility. They can set the whole cache to be encrypted or only secure a subset of the items."  

> **Candidate**: "To my best knowledge, iOS/Android platforms do not provide a built-in secure database at the moment. As a workaround, we could use third-party providers (like [SQLCipher](https://www.zetetic.net/)) to _encrypt the whole database_ or only _encrypt data BLOB storage_ in the database (under a strong assumption that cache keys do not contain any sensitive information)".  
> **Candidate**: "For our general-case purpose, I would rather select the BLOB encryption since the vast majority of users won't store any sensitive information. Also, I'd be very cautious before adding a 3rd-party dependency into the library: it might introduce licensing issues and binary/version incompatibility with the host app." 
![High-level Diagram](/images/exercise-caching-library-encryption.svg)

> **Candidate**: "The encryption key would be generated and stored in the Keystore/Keychain."  
> **Candidate**: "We should also store an `encrypted` flag in the database to mark secure items."  

<div align="center">
 
|     name      |  type  |
|---------------|--------|
| key           | String |
| data          | BLOB   |
| size_bytes    | Int    |
| access_count  | Int    |
| last_accessed | Date   |
| encrypted     | Bool   |

</div>

> **Candidate**: "As a side note, many applications already have a mature encryption stack. For this case, we might provide them the ability to specify a custom encryption/decryption implementation."  

```
CacheEncryption:
+ encrypt(data: [Byte]): [Byte]
+ decrypt(data: [Byte]): [Byte]

CacheConfig:
+ init(maxMemoryCacheSize: Int, maxDiskCacheSize: Int)
+ setCacheEncryption(encryption: CacheEncryption)
```

> **Candidate**: "This way they can better control user data privacy: for example, download encryption key from the backend upon login, etc."  

### Cross-platform Support
> **Interviewer**: "Imagine, you need to write a cross-platform library that should run on mobile and desktop platforms. How would you change your design?"  
> **Candidate**: "Would we need to share the cached storage somehow?"  
> **Interviewer**: "What do you mean?"  
> **Candidate**: "Should the permanent cache storage be transferred between platforms?"  
> **Interviewer**: "No."  
> **Candidate**: "In this case, we don't need storage binary compatibility".  
> **Candidate**: "I would separate the library into two modules: 1) common code (written in C/C++); 2) native implementation/adapters (Java/Kotlin/Swift/Objective-C)."  
> **Candidate**: "The Journal, the Cache Eviction, and the Database interface would be included in the common module; the rest would be included in the implementation module."  
> **Candidate**: "However, I would be very cautious around using native code: 1) hard to debug; 2) hard to read crash logs; 3) need to compile for all supported architectures (arm7, arm64, x86, etc); 4) more likely to crash the host app (instead of just throwing an exception). 5) harder to develop compared to modern languages. 6) iOS/Android developers are more likely to submit pull-requests if the codebase is native to their platform (assuming open-source)."  

## Major Concerns and Trade-Offs

### Library responsiveness vs device resource usage
The biggest decision to make is Dispatcher's worker pool size. Having a larger amount of workers might make the library more responsive but would eventually spawn more threads.
  
### Data security
- No data is considered secure as long as its encryption key is stored on the same device. Some devices provide hardware-backed key storage but we can't guarantee this in a general case.  
- Encryption/Decryption also comes with a CPU and memory cost: this is especially important for performance-critical cases and low-level devices.  

## Conclusion
- Keep this in mind while preparing for a system design interview:
- Don't try to make it perfect - provide a "signal" instead.
- Listen to your interviewer and keep track of the time.
- Try to cover as much ground as possible without digging too much into the implementation details.

## Looking for more content?
I’m thinking about creating an in-depth mobile system design course on top of the free articles. Please, fill out this [form](https://forms.gle/KfvmZhPNPMRBE8Jj9) to express your interest!
