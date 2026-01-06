# Resumable Uploads (Chunked Uploads)

Resumable uploads, also known as chunked uploads, break down a large file upload into smaller, manageable chunks. This technique allows the upload to be paused and resumed without losing progress, making it ideal for situations where network connectivity is unreliable or the upload size is significant. This also allows for more granular error reporting and potentially, faster recovery.

The resumable upload process typically involves these stages:

- **Upload Initialization:** The client initiates the upload by sending a request to the server, providing metadata about the file (name, size, content type). The server creates a temporary storage location and returns a unique upload ID or URL to the client.
- **Chunk Bytes Uploading (Appending):** The client divides the file into smaller chunks (e.g., 1MB, 5MB) and uploads each chunk separately to the server using the upload ID or URL. The client sends each chunk with a specific offset indicating its position in the overall file.
- **Upload Finalization:** Once all chunks have been successfully uploaded, the client sends a finalization request to the server. The server assembles the chunks into the complete file and performs any necessary processing (e.g., validation, transcoding).

![Resumable-Uploads](/images/resumable-uploads.svg)

**Advantages of Resumable Uploads:**

- **Allows you to resume interrupted data transfer operations without restarting from the beginning:** This is the primary advantage, especially crucial for users with unreliable network connections or large file uploads. Saves time, bandwidth, and improves the user experience.
- **Improved Reliability:** By dividing the upload into smaller chunks, it reduces the impact of network interruptions. If a chunk fails to upload, only that chunk needs to be retransmitted, not the entire file.
- **Bandwidth Optimization:** Resuming from a specific point avoids resending already transferred data, conserving bandwidth.
- **Progress Monitoring:** Allows for more accurate and granular progress tracking during the upload process, providing users with better feedback. This also allows for better prioritization of bandwidth within the device.
- **Parallel Uploading (Potentially):** Some implementations allow for uploading multiple chunks in parallel, further speeding up the upload process (however, this can increase complexity). This requires server-side support for out-of-order chunk arrival and assembly.
- **Improved Error Handling:** Easier to pinpoint and handle errors at a chunk level rather than the entire upload.
- **Resource Management:** Can help manage server-side resources more effectively by receiving and processing data in smaller increments.
- **Suitable for Large Files:** Essential for uploading files exceeding size limits imposed by HTTP servers or mobile operating systems.

**Disadvantages of Resumable Uploads:**

- **An overhead from additional connections and metadata:** Each chunk requires a separate HTTP request, adding overhead to the overall upload process. Requires careful management of HTTP headers.
- **Increased Complexity:** Implementing resumable uploads requires more complex client-side and server-side logic compared to single-request uploads.
- **Server-Side Requirements:** Requires server-side support for handling chunked uploads, storing temporary data, and assembling the complete file.
- **State Management:** Both client and server need to maintain state about the upload process (e.g., upload ID, chunk offsets), which can add complexity.
- **Error Handling:** Requires robust error handling to deal with network interruptions, server errors, and other potential issues.
- **Potential for Data Corruption:** If chunks are not assembled correctly, it can lead to data corruption. Requires careful validation and integrity checks.
- **Storage Requirements:** The server needs temporary storage space to hold the uploaded chunks until the upload is finalized.

**Additional Considerations:**

- **Chunk Size:** The optimal chunk size depends on network conditions, file size, and server capabilities. Experiment with different chunk sizes to find the best balance between performance and overhead.
- **Encryption:** Encrypt the uploaded chunks to protect sensitive data in transit and at rest.
- **Concurrency:** Consider the number of concurrent uploads allowed to prevent overwhelming the device or server.
- **Timeout Handling:** Implement timeouts to handle stalled uploads.
- **Background Uploads:** If the upload can be interrupted (app closed), consider using background upload tasks with OS-level support (WorkManager/BackgroundTasks)
- **Idempotency:** Ensure that resuming a failed upload does not result in duplicate data on the server. Use unique identifiers for each chunk.

**Note:** Resumable uploads are most effective with large uploads (videos, archives) on unstable networks. For smaller files (images, texts) and stable networks, a single-request upload should be sufficient. Carefully weigh the benefits and drawbacks before implementing resumable uploads. If targeting a specific cloud storage provider, utilize their SDK which may provide simplified upload mechanisms.

#### More Info:
- [YouTube: Resumable Uploads](https://developers.google.com/youtube/v3/guides/using_resumable_upload_protocol)
- [Google Photos: Resumable Uploads](https://developers.google.com/photos/library/guides/resumable-uploads)
- [Google Cloud: Resumable Uploads](https://cloud.google.com/storage/docs/resumable-uploads)
- [Twitter: Chunked Media Upload](https://developer.twitter.com/en/docs/twitter-api/v1/media/upload-media/uploading-media/chunked-media-upload)
