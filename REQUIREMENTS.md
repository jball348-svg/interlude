# Requirements

## Milestone 1 — Working MVP

### Phase 1: Project Bootstrap & Auth
- [ ] Xcode project created with SwiftUI, minimum iOS 17
- [ ] MusicKit entitlement added to the target
- [ ] `MusicAuthorization.request()` called on launch; graceful denial state shown
- [ ] Basic app structure: tab bar or nav with Home, Settings stubs

### Phase 2: Podcast Picker
- [ ] User can enter a podcast RSS feed URL OR search by show name (using iTunes Search API — free, no key required)
- [ ] App fetches and parses the RSS/Atom feed, displays a list of episodes (title, date, duration)
- [ ] User taps an episode to "select" it (stored in app state)
- [ ] Selected episode shown in a "Now Queued" strip at the bottom of the screen

### Phase 3: Music Playlist Picker
- [ ] MusicKit query for the user's Apple Music library playlists
- [ ] Simple list UI showing playlist name + track count
- [ ] User taps a playlist to select it (stored in app state)
- [ ] Selected playlist shown in the "Now Queued" strip alongside the podcast

### Phase 4: Session Config & Playback Engine
- [ ] Simple config screen: "Talk for" slider (2–10 min) + "Then play" stepper (1–3 songs)
- [ ] "Start Interlude" button enabled when both podcast + playlist are selected
- [ ] Playback engine (InterludePlayer) manages state machine:
  - TALKING → play podcast via AVPlayer for configured duration, then pause
  - MUSIC → play N songs from playlist via ApplicationMusicPlayer, then pause
  - Repeat until podcast episode ends or user stops
- [ ] AVAudioSession configured for background audio (category: `.playback`)
- [ ] MPNowPlayingInfoCenter updated for both segments so lock screen shows correct metadata

### Phase 5: Playback UI & Controls
- [ ] Full-screen "Now Playing" view showing:
  - Current segment type (Podcast / Music)
  - Title of what's playing
  - Progress bar with time elapsed / remaining in current segment
  - Next segment preview ("Up next: 2 songs")
- [ ] Play/Pause button
- [ ] Skip-to-next-segment button (jump straight to next music or talk segment)
- [ ] Stop / End Session button
- [ ] Remote command centre integration (play/pause/skip from lock screen / AirPods)

### Phase 6: Polish & Edge Cases
- [ ] Smooth transition between segments (short 0.5s audio fade out before switching)
- [ ] Handle podcast ending mid-session gracefully (go to music, then show "Session complete")
- [ ] Handle empty playlist edge case
- [ ] Handle network failure loading podcast feed
- [ ] Basic persistence: remember last-used podcast feed URL and playlist selection (UserDefaults)
- [ ] App icon + launch screen

## Out of Scope (v1)
- Spotify / other services
- Downloading episodes offline
- Cross-fade at sample level
- Multiple episodes queued
- Cloud sync / user accounts
- iPad / Mac Catalyst
