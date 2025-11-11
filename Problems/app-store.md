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
- Minor but Crucial Optimizations

