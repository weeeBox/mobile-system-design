# Mobile System Design Exercise: File Downloader Library
The task might be simply defined as "Design File Downloader Library". The definition is purposely vague and requires some clarification.

## Gathering Requirements
You are expected to ask clarifying questions and narrow down the scope of the task. This should not take more than 5 minutes.

> **Candidate**: "Are we designing a part of an application or a general-purpose library?"  
> **Interviewer**: "A general-purpose library."  

> **Candidate**: "Are we downloading a file from the internet and saving it to the disk?"  
> **Interviewer**: "Yes."  

> **Candidate**: "What kind of files do we need to download? Any size constraints?"  
> **Interviewer**: "Any kind - let's assume that we're working with binary files without specific format and no size limit."  

> **Candidate**: "Should we support pausing/resume/canceling/listing downloads?"  
> **Interviewer**: "Yes."  

> **Candidate**: "Do we need to support simultaneous file downloads?"  
> **Interviewer**: "I don't know - what do you think?"  
> **Candidate**: "I think, there might be some good use-cases where simultaneous downloads would be useful. For example, downloading a video playlist."  

> **Candidate**: "Should we limit the number of active simultaneous downloads?"  
> **Interviewer**: "I don't know - what do you think?"  
> **Candidate**: "I think, having unrestricted parallel downloads might hurt application performance and quickly exhaust system resources. At the same time, we can't make any assumptions about app-specific use-cases, so the best approach might be selecting a sensible default value (for example, no more than `4` parallel downloads) and letting developers configure it based on their application's need".  

> **Interviewer**: "Why do you think `4` is a good number for parallel downloads?"  
> **Candidate**: "It's a tricky question. We can use different heuristics to select the best number. For example, we can limit the size to the number of CPU cores on the device."  
> **Candidate**: "On the other hand - each downloader would be blocked on the network I/O so we can use a higher number like `16`."  
> **Candidate**: "It's hard to figure this out for the general case. So `4`-`16` seems reasonable by default. The user may pick a better size depending on the use-case."  

> **Candidate**: "Should we support progress reporting for active downloads?"  
> **Interviewer**: "Might skip it for now and discuss it if we have time"  

> **Candidate**: "Do we need to handle authentication?"  
> **Interviewer**: "Let's skip it."  

> **Candidate**: "Do we need to support [HTTP ranges](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests) for resumable downloads?"  
> **Interviewer**: "We can leave it out of scope"  

Make sure not to overload the system requirements with unnecessary features. Think in terms of MVP (Minimum Viable Product) and pick features that have the biggest value. You can learn more about requirements gathering [here](https://github.com/weeeBox/mobile-system-design#gathering-requirements).
### Functional requirements
- Developers should be able to simultaneously download multiple files over HTTP to the disk.
- Developers should be able to pause/resume/cancel downloads.
- Developers should be able to list active downloads.

### Non-functional requirements
- Limited active simultaneous downloads.
- Unlimited download file size.

### Out of scope
- Login/Authentication.
- Resumable HTTP-downloads.

## Client Public API (Optional)
Your interviewer might want to discuss the client public API. In the simplest form it might look like this:
```
FileDownloader:
+ init(config: FileDownloaderConfig)
+ download(request: FileDownloadRequest): FileDownloadTask
+ pauseAll()
+ resumeAll()
+ cancelAll()
+ activeTasks(): [FileDownloadTask]

FileDownloadRequest:
+ init(sourceUrl: Url, destPath: String)

FileDownloadTask:
+ addDownloadCallback(callback: FileDownloadCallback)
+ pause()
+ resume()
+ cancel()

FileDownloaderConfig:
+ init(maxParallelDownloads: Int)

FileDownloadCallback:
+ onComplete(request: FileDownloadRequest)
+ onFail(request: FileDownloadRequest, error: String)
+ onCancel(request: FileDownloadRequest)
```

- **FileDownloader** - represents a single file downloader instance; schedules and maintains active downloads.
- **FileDownloadRequest** - encapsulates a single download request (source, dest, headers, etc).
- **FileDownloadTask** - a handle to an asynchronous file downloading operation.
- **FileDownloaderConfig** - encapsulates file downloader configuration. Simplifies downloader instance creation and future refactorings. As an alternative, you may suggest the Builder pattern.
- **FileDownloadCallback** - encapsulate completion/failure callbacks.

## High-Level Diagram
A high-level diagram shows all major system components and their interactions (without implementation details). You can learn more about high-level diagrams [here](https://github.com/weeeBox/mobile-system-design#high-level-diagram).

![High-level Diagram](/images/exercise-file-downloader-library-high-level-diagram.svg)

### Components
- **File Downloader** - the central component which provides the client API and brings all components together.
- **Download Request** - encapsulates a single file downloading request; accepted by file downloader as input.
- **Download Task** - represents an asynchronous download operation; produced by the file downloader as output.
- **Download Dispatcher** - schedules and dispatches download operations.
- **Network Client** - handles receiving bytes over HTTP.
- **File Store** - writes file contents to the disk.

## Deep Dive: Download Dispatcher
After a high-level discussion, your interviewer might want to discuss some specific components of the system. Make sure to keep your explanation brief and don't overload it with details - let your interviewer guide the conversation and ask questions instead. You can learn more about deep-dive discussions [here](https://github.com/weeeBox/mobile-system-design#deep-dive-tweet-feed-flow).

![Download Dispatcher Diagram](/images/exercise-file-downloader-library-deep-dive-download-dispatcher-diagram.svg)

> **Candidate**: "Download Dispatcher maintains a _concurrent_ dispatch queue of jobs. Each job consists of a download request, a download task, and a state (PENDING, ACTIVE, PAUSED, COMPLETED, FAILED). The active jobs are dispatched by download workers."  

> **Interviewer**: "Why do you need to maintain a job state?"  
> **Candidate**: "To keep track of _pending_, _active_, and _completed_ jobs: we limit the number of parallel downloads by design. Also, we need to maintain the `pause` state of the scheduled jobs."  

> **Interviewer**: "What's the difference between a `Job` and a `Download Worker`?"  
> **Candidate**: "A `Job` encapsulates a single downloading request from the user. A `Download Worker` is responsible for actual data transmission from the network."  
> **Candidate**: "A `Job` is cheap and doesn't do anything. A `Download Worker` is expensive and handles blocking I/O."  
> **Candidate**: "There might be an unlimited number of jobs and only a handful of workers (limited by the pool size)."  
> **Candidate**: "There might be _multiple jobs_ for the same URL but only _one worker_ per download. For example, the same video file can be included in multiple playlists. The user can download these playlists in parallel - as a result, there might be different jobs for the same worker."  
> **Candidate**: "If a single job gets canceled - its worker keeps downloading until done or all the associated jobs are canceled, too." 

![Download Job Diagram](/images/exercise-file-downloader-library-job-diagram.svg)

> **Interviewer**: "Why do you need Init and Complete/Fail operations in the File Store?"  
> **Candidate**: "This gives you an ability to pre-allocate disk space before downloading a file and perform post-processing/cleanup after a complete download."  

## Follow-up Questions
Some interviewers might ask follow-up questions that might change the original design and introduce new requirements.

> **Interviewer**: "How would you change your design if the library needs to keep track of downloaded files?"  
> **Candidate**:
>  - "We can add a component ("Progress Store") that maintains a structured record of scheduled jobs in a relational database."
>  - "Each job would have an associated "progress" callback to signal every time the next chunk of bytes is received from the network."
>  - "The Progress Store component will use the job callbacks to update the database."
>  - "Once the job is complete - the corresponding record is deleted."  

<div align="center">

|       name       |  type  | primary |
|------------------|--------|---------|
| _ID              | Int    | YES     |
| url              | String |         |
| path             | String |         |
| created_at       | Date   |         |
| total_bytes      | Int    |         |
| downloaded_bytes | Int    |         |
| state            | Int    |         |
  
</div>

> **Interviewer**: "Why won't you store files in the database?"  
> **Candidate**: "It's easier to access a file on the disk than from the database. We can partially read BLOBs and provide the content as a stream but it might not be sufficient in the general case."  

> **Interviewer**: "How would you handle downloading sensitive information?"  
> **Candidate**: "We might introduce an "encrypted" File Store implementation and encrypt a fully-received file in post-processing. Not sure if we can do it on the fly - don't have much experience working with encryption libraries."  

> **Interviewer**: "How would you change your design to support download requests of different priorities? For example, user-critical, UI-critical, UI-non-critical, low-priority?"  
> **Candidate**: "The first option: update Download Dispatcher to use a priority queue; the second option: configure on the File Downloader instance level and maintain different instances based on the priority (would also require synchronization between instances). The priority information could be included in a Download Request."  

## Foreground and Background Downloads (Optional)
You may ask your interviewer if they want to discuss foreground/background downloads.
### Foreground Download
Uses a worker thread in the host application process. Works best for short-running and urgent downloads.
#### pros:
- Immediate execution within the host app.
- Limited only by system resources.
- Easy to set up and troubleshoot.

### cons:
- Stopped when the application is killed or suspended (works slightly different between iOS and Android).

### Background Download 
Might run in a separate process while the app is suspended. Best for long-running and nonurgent downloads.

#### pros:
- Guaranteed execution (iOS - survives backgrounding the app; Android - survives device restart).

#### cons:
- Hard to set up and debug.
- The host OS might wait for optimal conditions to perform the transfer, such as when the device is plugged in or connected to Wi-Fi.
- Must comply with Background Transfer Limitations.

### Implementation Details
- A preferred download method is selected with the File Downloading request.
- The Download Dispatcher selects appropriate Download Worker implementation based on request configuration.

## Major Concerns and Trade-Offs

### Network/CPU/Battery Usage
- Having too many parallel downloads might negatively impact device battery life and increase cellular network usage. A possible workaround is to adjust the concurrency settings depending on the device state. For example, limit the number of parallel downloads to `1` when using the cellular network or having a low battery charge, etc.

### Disk Space Allocation
- Another major decision is whether to pre-allocate disk space or not. For example, the library can create place-holder files for every download in the queue to ensure enough space regardless of the file system state. It's hard to argue about this in a general case - as a workaround this can be configurable while initializing a File Downloader instance.

## Conclusion
- Keep this in mind while preparing for a system design interview:
- Don't try to make it perfect - provide a "signal" instead.
- Listen to your interviewer and keep track of the time.

Try to cover as much ground as possible without digging too much into the implementation details.

## Looking for more content?
I’m thinking about creating an in-depth mobile system design course on top of the free articles. Please, fill out this [form](https://forms.gle/KfvmZhPNPMRBE8Jj9) to express your interest!
