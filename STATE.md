# State

## Current Phase
**Phase 1 — Not started**

## Decisions Made
- App name: **Interlude**
- Platform: iOS 17+, SwiftUI
- Podcast source: RSS feed parsing + iTunes Search API (no Apple auth required for search)
- Music source: MusicKit / ApplicationMusicPlayer (requires Apple Music subscription)
- Podcast playback: AVPlayer (streaming from episode URL in RSS feed)
- Audio mixing strategy: sequential (pause one player, start other) — no sample-level crossfade in v1
- Transition: 0.5s fade out of current segment before switching

## Open Questions
- Does the friend have an Apple Developer account? (Required to run on device with MusicKit entitlement)
- What's the Apple Music API / MusicKit developer token situation? (Needed at runtime)
- Should search use iTunes Search API or require manual RSS URL entry? (Plan: offer both)

## Blockers
- None yet. Apple API keys / developer account needed before Phase 1 device testing.

## Session Log
- 2026-03-11: Repo scaffolded, GSD docs written, build prompts created.
