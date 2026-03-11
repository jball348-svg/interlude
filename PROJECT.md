# Interlude

## Vision
Interlude is a native iOS app that weaves Apple Podcasts episodes and Apple Music playlists into one seamless listening session. The user defines the rhythm — talk for 2–10 minutes, then 1–3 songs — and Interlude handles the rest, creating a bespoke radio-style flow that feels natural and easy to live with.

## The Problem
People love both podcasts and music but switching between apps constantly breaks immersion. Listening to a full 90-minute podcast feels like a commitment; music-only sessions miss depth. Interlude solves this by treating both as equal citizens in a single listening experience.

## Target User
The primary user is John's friend — an Apple ecosystem user who listens to Apple Podcasts and Apple Music daily and wants a smarter, more rhythmic way to consume both together.

## Platform
- Native iOS app (SwiftUI + Swift)
- Requires Apple Music subscription (MusicKit)
- Requires Apple Podcasts (local playback via AVFoundation / podcast feed parsing)
- Minimum iOS target: iOS 17

## Core Loop
1. User picks a podcast episode
2. User picks an Apple Music playlist
3. User sets talk duration (2–10 min) and songs per break (1–3)
4. Interlude plays: [talk segment] → [N songs] → [talk segment] → repeat
5. Playback survives lock screen, background, AirPods

## Success Criteria
- A complete listening session from start to finish with no manual intervention
- Lock screen controls work correctly for both podcast and music segments
- The transition between talk and music feels smooth (short fade or brief pause)
- The app is simple enough that the friend can use it without any instructions

## Tech Stack
- SwiftUI (UI)
- MusicKit (Apple Music catalog + playback via ApplicationMusicPlayer)
- AVFoundation / AVPlayer (podcast audio playback)
- URLSession + RSS/Atom feed parsing (podcast episode fetching)
- AVAudioSession (audio session management)
- MediaPlayer framework (Now Playing / lock screen integration)

## Out of Scope (v1)
- Spotify or other streaming services
- Android
- Cross-fade audio mixing at the sample level (simple pause/resume transitions only)
- Cloud sync or accounts
- Social features
- Downloading episodes for offline (streaming only)
