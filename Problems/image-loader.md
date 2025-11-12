Problem: Image Loading Library
-

**Functional**
- Load image from app resources/local cache/remote into UI targets
- Download / preload image given by image url

**Non-functional**
- Fast loading
- Moderate memory usage
- Optimize network bandwidth usage

High level diagram
-
<img width="774" height="478" alt="Screenshot 2025-11-11 at 10 08 11 PM" src="https://github.com/user-attachments/assets/abb1813c-c7f0-475e-a719-6177fa7a8689" />


Module Deepdives
- 
- Cache module
  <img width="604" height="468" alt="Screenshot 2025-11-11 at 4 24 24 PM" src="https://github.com/user-attachments/assets/1eaa38b3-7ee4-450b-bf45-83002cf63393" />

  1. In-memory LRU cache. This caches the recently used bitmaps that are decoded & transformed, ready to bind to UI targets.
  2. Disk/File cache. This caches the encoded image bytes into files when they are fetched from remote.
  3. Metadata Database. This keeps track of the image metadata (format, size, encryption, ttl, etcs). This is used along with disk cache to assist encoding/decoding, processing, etcs. 
  
- Request parser:
  Responsible for normalizing user inputs (URL, File, Uri, resource id) into a canonical ImageRequest object.
- Request Coordinator:
  A layer to handle request dedupe, priority & scheduling, cancellation & lifecycle-aware, error handling
  1. [dedupe]fetch key, decode key, subsciber, make sure network request, decode job are performed once
  2. [priority] VISIBLE/HIGH/NORMAL(DEFAULT)/LOW/PREFETCH
  3. [scheduling] max concurrent size, IO pool, CPU pool
  4. View detach / Fragment onStop / Activity onDestroy → call handle.cancel()
  5. retry or handle IO/network/low disk space errors

- Several Key things:
  1. 3 layers of hierarchy in caching
  2. request deduplication
  3. lifecycle-aware (cancellation mechanism)
  4. threading (network, I/O, main)
  5. Save encoded (original) bytes to disk, save transformed bytes in-memory
  6. File Cache has proper TTL
  7. Memory Cache monitoring, onTrimMemory()
  8. BitmapPool mechanism to save bitmap init cost

