# Roadmap — Milestone 1 (MVP)

## Phase 1 — Project Bootstrap & Auth
Bootstrap the Xcode project, configure entitlements, and handle MusicKit authorisation. App launches, requests permission, and shows a skeleton UI.

## Phase 2 — Podcast Picker
RSS feed parsing + iTunes Search API integration. User can find and select a podcast episode. No playback yet.

## Phase 3 — Music Playlist Picker
MusicKit library query. User can browse and select an Apple Music playlist. No playback yet.

## Phase 4 — Session Config & Playback Engine
The heart of the app. InterludePlayer state machine drives the talk→music→talk loop using AVPlayer (podcast) and ApplicationMusicPlayer (music). Background audio and lock screen metadata wired up.

## Phase 5 — Playback UI & Controls
Now Playing screen with segment progress, skip controls, and Remote Command Centre integration so lock screen and AirPods work correctly.

## Phase 6 — Polish & Edge Cases
Transitions, error states, basic UserDefaults persistence, app icon, and launch screen. Ship-ready.

---

## Status
- [ ] Phase 1
- [ ] Phase 2
- [ ] Phase 3
- [ ] Phase 4
- [ ] Phase 5
- [ ] Phase 6
