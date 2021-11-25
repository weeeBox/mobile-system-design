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

Make sure not to overload the system requirements with unnecessary features. Think in terms of MVP (Minimum Viable Product) and pick features that have the biggest value. You can learn more about requirements gathering [here](https://github.com/weeeBox/mobile-system-design#gathering-requirements).

## Functional requirements
- Users should be able to load images from the network, filesystem, and app resources to UI/non-UI targets.

## Non-functional requirements
- Cache loaded images in memory and on the disk.
- Image loading respects view hierarchy lifecycle events.

## Out of scope
- Custom image loaders.
- Image placeholders, loading indicators, transition animations.

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

> **Interviewer**: "What would you use for the cache key?"  
> **Candidate**: "It's a good question. The first option is to use the image URI. However, we might also want to distinguish between images with the same URI but different dimensions and formats. To accomplish this - we can create a new `ImageKey` type to encapsulate image parameters."  

```
ImageKey:
+ init(source:Uri, width: Int, height: Int, format: ImageFormat)
```  

> **Interviewer**: "Why would you need to include image dimensions?"  
> **Candidate**: "Mostly to save memory and make drawing more efficient: it's better to re-scale the image to fit its target while loading."  

> **Interviewer**: "What do you mean by `format`?"  
> **Candidate**: "The image format describes the bit encoding for every pixel (for example, `RGBA8888`, `RGB888`, `RGB565`, etc). We can optimize our memory usage by reducing the bit depth or discarding the alpha channel."  
 
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
> **Candidate**: "It accepts a `Loader Request` and produces an image in return."  

> **Interviewer**: "What would you include in the request?"  
> **Candidate**: "An `ImageKey` and a callback function."  

```
LoaderRequest:
+ init(key: ImageKey, callback: (key: ImageKey, image: Image?, error: String?) → Void)
```

_NOTE: in this exercise, the API description is purposely left platform/language agnostic to reach a broader audience. During an actual interview, you would most likely design for either iOS or Android and select the API style most suitable for the target platform. For example, you can advocate for coroutines instead of callbacks, suggest Rx-extensions, etc._  

![High-level Diagram](/images/exercise-image-library-image-loader.svg)

> **Candidate**: "I would also decouple image data loading from the decoding. This way, we can easily add new decoders using the [Strategy](https://en.wikipedia.org/wiki/Strategy_pattern) design pattern without changing the loader itself."  

_NOTE: Android engineers might also want to mention a special [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap) cache to avoid excessive garbage collection with recycled bitmaps. For more information check [BitmapPool](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.java) from Glide._

> **Interviewer**: "I'm a little confused by your caching approach - why there's no cache functionality in the Image Loader component?"  
> **Candidate**: "Mostly due to the [Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle): the Image Loader should not know anything about caching - its single purpose is to ~~pass butter~~ load images."  
> **Candidate**: "However, the HttpClient component handles the disk cache for downloaded images. This is a trade-off between a 'clean' design and an 'ease of implementation' - might be a good short-term strategy but would scale poorly in a long run."  

> **Interviewer**: "What do you mean by `scaling`? Are we talking about any backend issues?"  
> **Candidate**: "No. By `scaling`, I mean library compatibility and long-run support. Depending on the platform, a 3rd-party dependency may have version conflicts with other libraries from the host app."  
> **Candidate**: "You also need to have a good plan on supporting and updating the dependency over time: for example, you need to decide on the update schedule (each release, each major release, critical bugfix-only, etc) or if you want to maintain a private fork."  
> **Candidate**: "There might be licensing issues for the customers, too."  

_NOTE: See "Chapter 21. Dependency Management" from the "Software Engineering at Google" book._  

> **Interviewer**: "Sounds tough - why would you want to have the dependency in the first place?"  
> **Candidate**: "This a good example of `implement` vs `re-use` trade-off. Personally, I think we should prioritize time-to-market first and then refine in future releases."  
> **Interviewer**: "Makes sense."  

_NOTE: The system design interview is not all about technology and engineering - it's always good to keep business needs in mind, too._  

> **Candidate**: "I forgot to mention this but we can also easily add authentication support on the network client level."  
> **Interviewer**: "That's fine - we can leave it out of scope."  

## Deep Dive: Image Request

> **Interviewer**: "Tell me more about `ImageRequest`."  
> **Candidate**: "It encapsulates the image loading parameters and the image target. We can use fluent chaining to create a cleaner interface."  

```
ImageRequest:
+ load(source: Uri): ImageRequest
+ load(format: ImageFormat): ImageRequest
+ into(target: ImageView): ImageRequest // we can overload this to support multiple targets
```

> **Candidate**: "We can register lifecycle callbacks with UI-targets (ImageView, Button, etc) for easier control of resources. For example, we can stop loading when the target is detached from the view hierarchy, becomes invisible, or gets recycled as a part of list scrolling."  

## Follow-up Questions
Some interviewers might ask follow-up questions that might change the original design and introduce new requirements.  

> **Interviewer**: "How would you change your design if you can't use the HttpClient caching?"  
> **Candidate**: "I would store image metadata in a relational database and store image bytes in the internal cache directory?"  
> **Candidate**: "Also, I would set a disk usage limit for the cache and provide a simple LRU eviction policy."  

> **Interviewer**: "Why would you select the internal cache directory? What other options do you have?"  
> **Candidate**: "The cache directory can be automatically cleaned up when the device disk space runs low. Also, the cached contents won't be backed up with the app."  
> **Candidate**: "I would prefer an internal storage for privacy reasons and to avoid using extra permissions (Android only)."  
> **Candidate**: "I don't think if I know a better storage option."  

_NOTE: It's ok to tell the interviewer that you don't have much experience with the storage options. Better be honest than get caught in a lie._  

> **Interviewer**: "What if you need to store sensitive images?"  
> **Candidate**: "I would support this as a flag in the `ImageRequest` and encrypt/decrypt corresponding files using encryption keys from Android Keystore or iOS Keychain."  
> **Candidate**: "A **better** option is not to store sensitive images at all."  

## Major Concerns and Trade-Offs

### Memory/Disk/CPU/Bandwidth
_TBD_

### Low Data Mode (iOS) and Data Saver mode (Android)
_TBD_

## Conclusion
- Keep this in mind while preparing for a system design interview:
- Don't try to make it perfect - provide a "signal" instead.
- Listen to your interviewer and keep track of the time.
- Try to cover as much ground as possible without digging too much into the implementation details.

## Looking for more content?
I’m thinking about creating an in-depth mobile system design course on top of the free articles. Please, fill out this [form](https://forms.gle/KfvmZhPNPMRBE8Jj9) to express your interest!
