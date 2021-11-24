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

> **Candidate**: "Network, filesystem, and app resources would make sense. We can expose an API to create custom loaders but we could leave it out of scope for now."  
> **Interviewer**: "Yes, let's leave it out of scope."  

> **Candidate**: "Do we need to support UI-targets?"  
> **Interviewer**: "Not sure what you mean."  
> **Candidate**: "Do we want the library to load images directly to UI components (like ImageView, Button, etc)?"  
> **Interviewer**: "Yes."  
> **Candidate**: "I think, it might be beneficial to support non-UI targets - like saving the image to a file or providing a callback to access the image data when loading is complete."  
> **Interviewer**: "Sounds good."  

> **Candidate**: "I would also add a caching support to store downloaded images on the disk to preserve the bandwidth and make it easier to support offline state."  
> **Interviewer**: "Sounds good."  

> **Candidate**: "It would be good to provide customizable placeholders, loading indicators, and transition animation for UI-targets - but we should probably leave it out of scope."  
> **Interviewer**: "Let's leave it out of scope."  

> **Candidate**: "We should also provide view lifecycle management for UI-targets."  
> **Interviewer**: "Not sure what you mean."  
> **Candidate**: "That means that the library would track view hierarchy lifecycle events to decide when to stop loading and free resources. For example, you should interrupt image loading when the target view becomes detached from the view hierarchy."  
> **Interviewer**: "Yes, it's a good feature."  

_TBD_: Low Data Mode (iOS) and Data Saver mode (Android)

Make sure not to overload the system requirements with unnecessary features. Think in terms of MVP (Minimum Viable Product) and pick features that have the biggest value. You can learn more about requirements gathering [here](https://github.com/weeeBox/mobile-system-design#gathering-requirements).

## Functional requirements
- Users should be able to load images from the network, filesystem, and app resources to UI/non-UI targets.

## Non-functional requirements
- Cache loaded images in memory and on the disk.
- Image loading respects view hierarchy lifecycle events.

## Out of scope
- Custom image loaders.
- Image placeholders, loading indicators, transition animations.

## Client Public API (Optional)
Your interviewer might want to discuss the client public API. In the simplest form it might look like this:

```
TBD
```

## High-Level Diagram
A high-level diagram shows all major system components and their interactions (without implementation details). You can learn more about high-level diagrams [here](https://github.com/weeeBox/mobile-system-design#high-level-diagram).

![High-level Diagram](/images/exercise-image-library-high-level-diagram.svg)

### Components:
- **Image Request** - encapsulates a single image request; accepted by request manager as input.  
- **Request Manager** - accepts and dispatches image requests to corresponding targets.  
- **Image Cache** - fast in-memory image storage for quicker access.
- **Image Loader** - coordinates image loading from different source (filesystem, app resources, network).
- **Image Target** - incapsulate image target destination (UI-element, in-memory image data, etc).

## Deep Dive
After a high-level discussion, your interviewer might want to discuss some specific components of the system. Make sure to keep your explanation brief and don't overload it with details - let your interviewer guide the conversation and ask questions instead. You can learn more about deep-dive discussions [here](https://github.com/weeeBox/mobile-system-design#deep-dive-tweet-feed-flow).  

## Deep Dive: Image Cache
> **Interviewer**: "What is the purpose of the `Image Cache` component?"  
> **Candidate**: "It's an in-memory LRU cache to keep a subset of loaded images for quicker access."  
 
> **Interviewer**: "Should this component also handle disk cache?"  
> **Candidate**: "The disk cache only makes sense for the images downloaded over the network. I think the network client can handle it better."  
 
> **Interviewer**: "Why do you think so?"  
> **Candidate**: "Most of the modern HTTP clients have built-in disk caching mechanisms that respect `Cache-Control` directives for responses."  
> **Candidate**: "This way, we only need to specify the cache size and the client can handle content expiration for us."  
 
> **Interviewer**: "How would you select the size of the cache?"  
> **Candidate**: "I would start with some default value. For example, the Google Drive app has `250`Mb disk cache size by default."  
> **Candidate**: "Then I would provide a library setting to override this."  
 
> **Interviewer**: "That makes sense but I was asking about in-memory cache size 🙂"  
> **Candidate**: "This part is harder since we can't make any strong assumptions on the memory usage patterns of the host app."  
> **Candidate**: "A simple heuristics could be allocating a chunk proportional to the max device memory. For example, Google [suggests](https://developer.android.com/topic/performance/graphics/cache-bitmap#memory-cache
) `1/8`th."  
> **Candidate**: "We can also use low-memory callbacks to purge the cache when the memory runs low."  

> **Interviewer**: "Can you see any drawbacks of using a 3rd party library?"  
> **Candidate**: "Depending on a 3rd party library is tricky since it can lead to semantic and binary incompatibilities with the host app. On the other hand, it handles lots of complexity for us and encapsulates the experience of many developers who worked on it. I think it's a trade-off between time-to-the-market and code ownership."  

_NOTE: If you don't have much experience with network stack internals - it's ok to mention a well-known library as a workaround._

## Deep Dive: Image Loader
> **Interviewer**: "Can we talk more about the `Image Loader` component?"  
> **Candidate**: "The component is responsible for loading images from different sources such as file system, app resources, and network."  
> **Candidate**: "It accepts a `Loader Request` and returns an image as a result."  

> **Interviewer**: "What would you include in the request?"  
> **Candidate**: "Source URI, image dimensions, and image format (like `RGBA_8888`, `RGB_888`, `RGB_565`, etc)"  

```
LoaderRequest:
+ init(source:Uri, width: Int, height: Int, format: ImageFormat)
```

> **Interviewer**: "Why would you need to include image dimensions and format?"  
> **Candidate**: "Mostly to save memory: 1) there's no need to load an image bigger than its target view; 2) we can drop alpha channel or lower the bit-depth."  

![High-level Diagram](/images/exercise-image-library-image-loader.svg)

> **Candidate**: "I would also decouple image data loading from the decoding. This way we can easily add new decoders without changing the loader itself."  

_NOTE: Android engineers might also want to mention a [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap) cache to avoid excessive garbage collection by recycling bitmaps. For more information check Glide's [BitmapPool](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.java)_

> **Candidate**: "I forgot to mention this but we can also easily add authentication support on the network client level."  
> **Interviewer**: "That's fine - we can leave it out of scope."  

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
