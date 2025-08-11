
```markdown
# architecture.md
**Last updated:** 2025-08-11

# Architecture overview (high level)

## System components
1. **Client (PWA)**  
   - UI + logic (small JS footprint). Responsible for playback, quizzes, local storage, contributor uploads, and peer sharing UI.
2. **Service Worker (SW)**  
   - App-shell pre-cache, runtime caching strategies, background sync queue, cache management.
3. **Local storage**  
   - IndexedDB for structured data (manifests, vocab, quiz results, progress), caches for binary assets (Cache API).
4. **Backend & APIs**  
   - Auth, content, contributor endpoints, sync endpoints. Lightweight serverless functions or Node/Express.
5. **CDN / Media Storage**  
   - Serve static assets (audio, svg, thumbnails) efficiently.
6. **Content Management / Contributor Portal**  
   - Web dashboard for writers, narrators, reviewers, translators, and moderators.
7. **Build pipeline (SSG / asset pipeline)**  
   - Pre-render story pages & manifests; generate optimized assets and multiple audio bitrates; incremental builds.
8. **Peer-to-peer module**  
   - Bluetooth data transfer + WebRTC DataChannel fallback for larger packages.

---

## High-level data flow (user scenario: download + play offline)
1. User opens app (first time) → PWA shell loads from network; SW pre-caches core files.
2. App fetches story list (small JSON) and displays metadata.
3. User taps “Download story” → client requests manifest; SW/network fetch caches manifest + assets. Assets stored in Cache API and structured data in IndexedDB.
4. User plays offline: audio served from Cache API; vocabulary and progress saved to IndexedDB.
5. User completes quiz while offline → result saved to a local "outbox" queue.
6. When the device regains connectivity, the SW Background Sync attempts to POST queued items to `/sync/progress`. Server acknowledges; client marks items as synced.

---

## Caching strategy (recommended)
- **App shell** (`app-shell-v1`) — *install-time precache* (critical JS/CSS/manifest/icons).
  - Strategy: precache (cache-first).
- **Static UI assets** (`static-assets-v1`) — images, svgs, icons.
  - Strategy: stale-while-revalidate.
- **Story manifests** (`manifests-v1`) — small JSON per story.
  - Strategy: network-first with cache fallback.
- **Story media** (`audio-cache`, `pages-cache`) — audio files + SVG pages.
  - Strategy: cache-first (explicitly downloaded by user), with explicit size limits and LRU eviction.
- **API responses** (`api-cache`) — non-critical lists.
  - Strategy: network-first with short TTL.

**Eviction / quota policy**
- Use `navigator.storage.estimate()` to evaluate free space.
- Keep on-device default download limit (e.g., 200 MB per user) with user overrides.
- Implement LRU eviction for least recently used story packages.

---

## Offline sync & conflict handling
- **Outbox queue** (IndexedDB): store local events (quiz results, progress, contributions).
- **Background sync**: exponential backoff for retries.
- **Conflict resolution**:
  - Progress/quiz results: merge-additive (store all attempts); server picks authoritative best attempt.
  - Content updates: use `version` + `etag` headers; client fetches new manifest when versions mismatch.

---

## Peer-to-peer sharing (practical approach)
- **Package format**: ZIP or tgz containing `manifest.json` + files; include `sha256` checksums.
- **Transfer options**:
  - Web Bluetooth (limited throughput): best for very small packages / metadata.
  - WebRTC DataChannel (higher throughput): preferred where supported; may require signaling via local hotspot or using temporary local server for handshake.
- **Handshake & integrity**:
  - Validate manifest version and checksums post-transfer.
  - Offer user a preview before import (language, age range, size).

---

## Security & privacy
- All server communications over HTTPS/TLS.
- Parental controls: require parental OTP or authenticated guardian session for sharing or publishing real-child content.
- Consent records: store explicit consent objects with timestamp, parentId, and storyId; show in admin portal.
- Minimize analytics (default off); anonymize any telemetry and keep it optional.

---

## SSG & build pipeline
- Pre-render story pages and small JSON manifests at build time.
- Produce multiple audio bitrate files and optimized SVGs during build.
- Use incremental builds for content updates (build only changed stories).
- Output artifacts directly to CDN origin for fast fetch.

---

## Recommended minimal deployment pattern
- Static hosting (Netlify/Vercel/Cloud CDN) for SSG output + CDN for media.
- Serverless functions (or small Node API) for auth, content search, and sync endpoints.
- Contributor portal separate authenticated app with file storage (S3 or equivalent).

---
