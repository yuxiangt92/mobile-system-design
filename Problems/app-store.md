Problem: Design an App Store. 

**Functional requirements**
1. Browse, Search (category, ranking, details page)
2. Login & device credentials
3. Download & installation (resumeable download, verification, signature verify)
4. Update mechanism, uninstall
5. Purchase
6. Scoring, review

**Non-functional**
1. Offline support
2. Good performance (loading latency, time to download & install apps)
3. Error handling

**High level diagram**
<img width="1174" height="730" alt="Screenshot 2025-11-08 at 10 46 29 PM" src="https://github.com/user-attachments/assets/784f9782-1835-4ac7-8c67-efd3aa66f69e" />

**Module Deepdives**

- Download & Installation
1. Client makes a request to the backend, say: "I want to install app X. Here's who I am and what my device looks like. Please give me the right artifacts."
   Client will provide user credentials and device info (OS type, device profile, etcs.) 
2. Server authenticates the user, checks entitlement (free / purchased / region / policy)
3. Server computes which artifacts are compatible and return a list of singed download URLs.
4. Client downloads those URLs directly from a CDN or object store.
5. Check if there is enough disk space and buffer before download.  
6. We prefer streaming download. This allows downloaded bytes to be written to disk in a streaming way, without spiking memories usage.
7. We should enqueue a download request to WorkManager. It helps to manage the job queue, check network, battery disk space, etcs.
8. The actual download job is handled by a ForegroundService. It creates a notification channel and bind with foreground notification, showing download progress, allowing user to pause/cancel. 
9. If download is suspended due to network disconnection, we should have some mechanism to resume the download when network reconnects. 
10. Once download is done, we should perform an integrity check to make sure downloaded file is intact (file size check)
11. Make signature verify before installation. 
12. Once installation is successful, delete the disk copy of the downloaded apk.

<img width="617" height="910" alt="Screenshot 2025-11-11 at 9 11 01 AM" src="https://github.com/user-attachments/assets/5841b4ca-4787-4a3b-b198-d082aac4025d" />



- Review & Score
  Use Local-First Data Flow 
  - Post a review
    1. User submits review
    2. Repository writes review = PENDING -> DB
    3. UI refreshed immediately (Optimistic UI)
    4. WorkManager update to server
    5. Server updated successful, change DB status to APPROVED
    6. Server updated failed, change DB status to FAILED (allowing for automatic retries)
  - Fetch reviews
    1. UI observes DB data flow
    2. Repository fetches from remote
    3. server response returns, merge into local database
    4. db emit new data, UI refreshed
    5. apply some pagination mechanism
       
- Minor but Crucial Optimizations
  1. DiffUtil: only refresh part of the recyclerview to avoid excessive refresh
  2. ETag: used for delta fetch, only pull when server content changes
  3. Prefetching: improve scrolling performance, reduce load time
  4. Optimistic UI: write comment, improve UI responsiveness, better user experience
  5. WorkManager: write comment, ensure reliable retry when offline

