# requirements.md
**Last updated:** 2025-08-11

# Kids Storytelling PWA — Requirements (developer-focused)

## Purpose & scope
Short: a lightweight, offline-first Progressive Web App for children (4–10) to listen to stories (voice-only or voice+image), learn vocabulary and knowledge items, take simple assessments, and allow community-driven content (writers, narrators, illustrators, translators, reviewers). Works on low-capacity devices and in low/patchy connectivity environments; supports peer-to-peer content transfer and explicit, opt-in social storytelling.

## Stakeholders
- Kids (primary users)
- Parents & caregivers (supervision, consent, analytics)
- Teachers (optional classroom use)
- Community contributors (writers, narrators, illustrators, translators, reviewers)
- Admins / Moderators (content curation, feature control)

---

## Functional requirements (core)
1. **Story playback**
   - Voice-only mode.
   - Voice + image page mode (page-turn style).
   - Play / Pause / Seek / Rewind.
   - Word highlighting synchronized with audio timestamps (optional, for supported stories).
   - Download stories for offline playback.

2. **Learning tracking**
   - Extract and store vocabulary per story.
   - Per-story knowledge cards (1–4 simple facts).
   - Local progress store (which stories played, seconds listened, words learned).

3. **Assessments**
   - Simple interactive quizzes: matching (word↔image), multiple choice (1 correct), fill-in-the-blank (with hints).
   - Store quiz attempts offline and sync when online.
   - Gamified rewards (stars/badges) displayed locally.

4. **Community content & workflows**
   - Contributor roles: writer, narrator, illustrator, translator, reviewer, moderator.
   - Content submission (story text + metadata), audio upload, SVG/graphics upload, translation files.
   - Review/approval workflow with versioning and basic metadata (language, age range, tags).

5. **Offline-first & sharing**
   - PWA installable on mobile/desktop.
   - Service Worker for caching app shell, assets, and selected stories.
   - Peer-to-peer sharing (Bluetooth / WebRTC fallback) of story packages (manifest + media).
   - Background sync for queued uploads and analytics.

6. **Privacy & social features**
   - Explicit opt-in consent for any real-child content; parental approval required.
   - Admin-curated social stories gallery (opt-in only).
   - Share buttons for published, consented content (share only sanitized/approved items).

---

## Non-functional requirements
- **Performance:** App shell loads < 1.5s on 3rd-gen low-end devices; first story start < 3s when cached.
- **Storage:** Default on-device downloads must be conservative (e.g., prefer low/medium audio quality); allow user to remove packs; handle storage quotas and eviction gracefully.
- **Offline behavior:** Core features (playback for downloaded stories, quizzes, progress recording) must work fully offline.
- **Accessibility:** WCAG AA baseline: large readable fonts, voice guidance, tappable controls, high-contrast theme option.
- **Localization:** Support multilingual UI and content (RTL support where needed).
- **Security & privacy:** TLS for all network calls; local encryption optional for sensitive items (consents); parental authentication for settings & sharing.
- **Device compatibility:** Target low-memory Android devices (1–2 GB RAM) and modern browsers (Chromium-based mobile browsers + Firefox).

---
