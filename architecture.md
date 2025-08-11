# Architecture

**Last updated:** 2025-08-11

## 1. System Components
1. **Client (PWA)**  
   - UI + logic (small JS footprint). Handles playback, quizzes, local storage, contributor uploads, and peer sharing UI.
2. **Service Worker (SW)**  
   - App-shell precache, runtime caching strategies, background sync queue, cache management.
3. **Local Storage**  
   - IndexedDB for structured data (manifests, vocab, quiz results, progress), Cache API for binary assets.
4. **Backend & APIs**  
   - Auth, content, contributor endpoints, sync endpoints. Lightweight serverless functions or Node/Express.
5. **CDN / Media Storage**  
   - Serve static assets (audio, SVG, thumbnails) efficiently.
6. **Content Management / Contributor Portal**  
   - Web dashboard for writers, narrators, reviewers, translators, and moderators.
7. **Build Pipeline (SSG / Asset Pipeline)**  
   - Pre-render story pages & manifests; generate optimized assets and multiple audio bitrates; incremental builds.
8. **Peer-to-Peer Module**  
   - Bluetooth data transfer + WebRTC DataChannel fallback for larger packages.

---

## 2. High-Level Data Flow  
**Scenario: Download + Play Offline**
1. User opens app (first time) → PWA shell loads from network; SW precaches core files.
2. App fetches story list (small JSON) and displays metadata.
3. User taps “Download story” → client requests manifest; SW/network fetch caches manifest + assets (Cache API + IndexedDB).
4. User plays offline: audio served from Cache API; vocabulary & progress saved to IndexedDB.
5. Quiz completed offline → result stored in an "outbox" queue.
6. On reconnect, SW Background Sync POSTs queued items to `/sync/progress`.

---

## 3. Caching Strategy
- **App Shell** (`app-shell-v1`) — Precache (cache-first).
- **Static UI Assets** (`static-assets-v1`) — Stale-while-revalidate.
- **Story Manifests** (`manifests-v1`) — Network-first with cache fallback.
- **Story Media** (`audio-cache`, `pages-cache`) — Cache-first (explicit download by user), with LRU eviction.
- **API Responses** (`api-cache`) — Network-first with short TTL.

**Eviction Policy**:
- Check storage space (`navigator.storage.estimate()`).
- Default limit: ~200 MB per user (configurable).
- LRU eviction for old story packages.

---

## 4. Offline Sync & Conflict Handling
- **Outbox Queue** (IndexedDB) for offline events.
- **Background Sync** with exponential backoff.
- **Conflict Resolution**:
  - Quiz/progress: merge-additive.
  - Content: version + ETag checks.

---

## 5. Peer-to-Peer Sharing
- **Package Format**: ZIP/TGZ with `manifest.json` + files + `sha256` checksums.
- **Transfer Methods**:
  - Web Bluetooth (small packages/metadata).
  - WebRTC DataChannel (larger packages).
- **Handshake**:
  - Validate manifest & checksums.
  - Preview before import (language, age, size).

---

## 6. Security & Privacy
- All server communications via HTTPS/TLS.
- Parental controls for content sharing/publishing.
- Consent records with timestamp, parentId, and storyId.
- Optional, anonymized analytics.

---

## 7. SSG & Build Pipeline
- Pre-render story pages + manifests at build.
- Multiple audio bitrates + optimized SVGs.
- Incremental builds for changed stories.
- Direct CDN output.

---

## 8. Deployment Pattern
- Static hosting (Netlify/Vercel/Cloud CDN) for PWA.
- CDN for media files.
- Serverless/Node API for auth & sync.
- Separate contribu
