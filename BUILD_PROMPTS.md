# Interlude — Build Prompts
> Copy each prompt into your AI coding tool (Windsurf / Antigravity / etc.) in order.
> Complete and verify each phase before moving to the next.

---

## PROMPT 1 — Project Bootstrap & MusicKit Auth

```
Create a new SwiftUI iOS app called "Interlude" with the following setup:

XCODE PROJECT SETUP:
- Minimum deployment target: iOS 17.0
- Interface: SwiftUI
- Language: Swift
- Bundle ID: com.interlude.app (adjust if needed)
- Add the "com.apple.developer.musickit" entitlement to the target's .entitlements file
- Add NSAppleMusicUsageDescription to Info.plist: "Interlude needs access to Apple Music to play songs from your playlists."

APP STRUCTURE:
Create the following Swift files:

1. InterludeApp.swift — @main entry point, creates ContentView

2. ContentView.swift — A tab view with three tabs:
   - "Listen" tab (house icon) → HomeView
   - "Settings" tab (gear icon) → SettingsView (stub, just Text("Settings coming soon"))

3. HomeView.swift — The main screen. For now shows:
   - App title "Interlude" in large bold text at top
   - A card/section for "Podcast" with a "Choose Episode" button (no action yet)
   - A card/section for "Music" with a "Choose Playlist" button (no action yet)
   - A disabled "Start Interlude" button at the bottom

4. MusicAuthManager.swift — An @Observable class (or ObservableObject) that:
   - Has a published property: authStatus: MusicAuthorization.Status
   - Has a method: requestAuthorization() async that calls MusicAuthorization.request() and updates authStatus
   - Is injected into the environment from InterludeApp.swift

5. On HomeView appearance, call requestAuthorization(). If status is .denied or .restricted, show an alert explaining the user needs to enable Apple Music access in Settings, with a button that opens UIApplication.openSettingsURLString.

The UI should be clean and minimal — dark background (#0D0D0D), white text, rounded cards with a subtle border. No placeholder images needed yet.
```

---

## PROMPT 2 — Podcast Picker (RSS + iTunes Search)

```
Add podcast episode selection to the Interlude app. Reference PROJECT.md and REQUIREMENTS.md for context.

Create the following:

1. Models/PodcastModels.swift:
   - struct PodcastEpisode: Identifiable, Codable
     - id: UUID (generated)
     - title: String
     - audioURL: URL
     - duration: TimeInterval? (in seconds, parsed from RSS <itunes:duration>)
     - publishedDate: Date?
     - showName: String

   - struct iTunesSearchResult: Decodable — matches iTunes Search API JSON response
     - struct Result: Decodable with fields: trackName, feedUrl, collectionName, artworkUrl100

2. Services/PodcastService.swift — a class with:
   - func searchShows(query: String) async throws -> [iTunesSearchResult.Result]
     Uses: https://itunes.apple.com/search?term=\(query.urlEncoded)&entity=podcast&limit=20
   - func fetchEpisodes(feedURL: URL) async throws -> [PodcastEpisode]
     Uses URLSession + XMLParser (or a simple string-based RSS parser) to parse:
     - <item> elements
     - <title>, <enclosure url="..." type="audio/mpeg">, <itunes:duration>, <pubDate>
     - Returns up to 30 most recent episodes

3. Views/PodcastPickerView.swift — a Sheet view:
   - Search bar at top (bound to a @State searchText)
   - While user hasn't searched: show a TextField for pasting an RSS URL directly, with a "Load Feed" button
   - After search: show list of podcast shows from iTunes results, tapping a show loads its episodes
   - Episode list: show title, duration (formatted mm:ss or h:mm:ss), date
   - Tapping an episode: dismiss sheet, pass selected PodcastEpisode back via a binding or callback

4. Wire up the "Choose Episode" button in HomeView to present PodcastPickerView as a sheet.
   - After selection, show the episode title in the Podcast card on HomeView.
   - Store the selected episode in a @State var selectedEpisode: PodcastEpisode? on HomeView.

Use async/await throughout. Show a ProgressView while loading. Show an error message if the feed fails to parse.
```

---

## PROMPT 3 — Apple Music Playlist Picker (MusicKit)

```
Add Apple Music playlist selection to the Interlude app using MusicKit.

Create the following:

1. Models/MusicModels.swift:
   - A thin wrapper if needed. MusicKit's Playlist type can be used directly.

2. Services/MusicLibraryService.swift — an @Observable class:
   - func fetchUserPlaylists() async throws -> [Playlist]
     Uses: MusicLibrary.shared.request(MusicLibraryRequest<Playlist>())
     Or: var request = MusicLibraryRequest<Playlist>(); request.limit = 100; return try await request.response().items
   - Returns playlists sorted alphabetically by name

3. Views/PlaylistPickerView.swift — a Sheet view:
   - List of the user's Apple Music playlists
   - Each row: playlist name + track count (if available)
   - Tapping a playlist dismisses the sheet and returns the selected Playlist
   - Show ProgressView while loading
   - If MusicAuthorization status is not .authorized, show a message: "Apple Music access required" with a Settings link

4. Wire up the "Choose Playlist" button in HomeView to present PlaylistPickerView as a sheet.
   - After selection, show the playlist name in the Music card on HomeView.
   - Store the selected playlist in @State var selectedPlaylist: Playlist? on HomeView.

5. Update the "Start Interlude" button: enable it only when BOTH selectedEpisode and selectedPlaylist are non-nil.

Important notes:
- Use MusicKit's async APIs throughout
- Do not use deprecated MusicKit APIs — target the MusicKit framework (not MediaPlayer MPMediaQuery) for library access
- Handle the case where the user has no playlists gracefully (show "No playlists found")
```

---

## PROMPT 4 — Session Config & Playback Engine

```
Build the core playback engine for the Interlude app. This is the most important feature.

Reference: PROJECT.md and REQUIREMENTS.md.

CONFIGURATION SCREEN:

Create Views/SessionConfigView.swift — presented as a sheet when "Start Interlude" is tapped:
- "Talk for" section: a Slider from 2 to 10 minutes (step 1 min), label shows current value e.g. "4 minutes"
- "Then play" section: a Stepper 1–3, label shows e.g. "2 songs"
- "Start" button that dismisses the sheet and begins playback
- Store these as @State or pass them as bindings

PLAYBACK ENGINE:

Create PlaybackEngine/InterludePlayer.swift — an @Observable class (or ObservableObject).

State machine with enum InterludeSegment: { case talking, music, idle, finished }

Properties:
- currentSegment: InterludeSegment = .idle
- episode: PodcastEpisode
- playlist: [Track] (resolved from MusicKit Playlist)
- talkDuration: TimeInterval (in seconds)
- songsPerBreak: Int
- currentTrackIndex: Int = 0
- podcastTimeElapsed: TimeInterval = 0
- podcastSeekTime: TimeInterval = 0 (track where we are in the episode)
- isPlaying: Bool = false

Internals:
- private var podcastPlayer: AVPlayer
- private var musicPlayer: ApplicationMusicPlayer (= .shared)
- private var segmentTimer: Task<Void, Never>?

Methods:
- func start() async — resolves playlist tracks from MusicKit (load Playlist with .tracks relationship), sets up AVPlayer with episode audioURL, begins first TALKING segment
- func beginTalkSegment() async — resume AVPlayer from podcastSeekTime, start a Task that waits talkDuration seconds then calls beginMusicSegment()
- func beginMusicSegment() async — pause AVPlayer, store current playback time as podcastSeekTime, queue next N tracks in ApplicationMusicPlayer, play, await .finished notification or track count, then call beginTalkSegment()
- func pause() — pause both players, cancel segmentTimer
- func resume() async — resume based on currentSegment
- func skipSegment() async — cancel current segment timer and immediately start next segment
- func stop() — cancel everything, reset state

AUDIO SESSION:
In InterludeApp.swift or a dedicated AudioSessionManager:
- Configure AVAudioSession.sharedInstance() with category .playback, mode .default, options .allowBluetooth
- Activate the session on app launch

NOW PLAYING (lock screen):
In InterludePlayer, update MPNowPlayingInfoCenter.default().nowPlayingInfo whenever segment changes:
- MPMediaItemPropertyTitle: episode title (talking) or track name (music)
- MPMediaItemPropertyArtist: show name (talking) or artist name (music)
- MPNowPlayingInfoPropertyElapsedPlaybackTime, MPNowPlayingInfoPropertyPlaybackDuration

REMOTE COMMANDS:
Register in InterludePlayer.setupRemoteCommands():
- MPRemoteCommandCenter.shared().playCommand
- MPRemoteCommandCenter.shared().pauseCommand
- MPRemoteCommandCenter.shared().nextTrackCommand → skipSegment()

Wire up: when the Start button is tapped in SessionConfigView, create InterludePlayer with the selected episode, playlist, and config, then call player.start(). Inject the player into the environment.
```

---

## PROMPT 5 — Now Playing UI & Controls

```
Build the Now Playing screen for the Interlude app. Reference PROJECT.md.

Create Views/NowPlayingView.swift — a full-screen view that appears once a session starts (push navigation or fullScreenCover).

Layout (top to bottom):
1. Segment badge: a capsule pill saying "🎙 Podcast" (orange) or "🎵 Music" (blue/purple) — updates when segment changes
2. Title: large bold text — current episode title (during talking) or current track name (during music)
3. Subtitle: show name / artist name in smaller grey text
4. Progress section:
   - During TALKING: circular or linear progress showing how far through the current talk segment (e.g. 2:30 / 4:00)
   - During MUSIC: show current track number e.g. "Song 1 of 2"
5. "Up next" label: e.g. "Up next: 2 songs" or "Up next: podcast"
6. Controls row:
   - [Skip segment] button (forward.end icon) — calls interludePlayer.skipSegment()
   - [Play/Pause] button (large, play.fill / pause.fill) — calls interludePlayer.pause() / resume()
   - [Stop] button (stop.fill or xmark.circle) — calls interludePlayer.stop() and pops/dismisses view
7. Bottom area: subtle waveform animation (just 3–5 animated vertical bars using SwiftUI animation, no need for real audio data)

State:
- Subscribe to @Observable InterludePlayer from environment
- Update the progress bar using a Timer that fires every 0.5s and reads interludePlayer.podcastTimeElapsed during TALKING segments
- During MUSIC segment, no countdown needed — just show track count

Transitions:
- When segment changes (talking ↔ music), animate the badge and title change with .transition(.opacity) and .animation(.easeInOut(duration: 0.4))

Wire up:
- After the user taps Start in SessionConfigView, navigate to NowPlayingView (or present as fullScreenCover)
- Pass the InterludePlayer instance
```

---

## PROMPT 6 — Polish, Edge Cases & Persistence

```
Final polish pass for the Interlude app. Reference REQUIREMENTS.md Phase 6.

TRANSITIONS:
In InterludePlayer, add a 0.5s audio fade-out before switching segments:
- For AVPlayer: use a Task that decrements volume from 1.0 to 0.0 in 10 steps over 0.5s, then pauses
- For ApplicationMusicPlayer: same approach using ApplicationMusicPlayer.shared.state.playbackRate or a volume fade via AVAudioSession

EDGE CASES TO HANDLE:
1. Podcast ends mid-session: when AVPlayer receives .AVPlayerItemDidPlayToEndTime, call a finishSession() method that:
   - Plays remaining music queue if in a talking segment
   - Then shows a "Session complete 🎉" state in NowPlayingView instead of crashing or looping
2. Empty playlist: before starting, check if the resolved track list is empty. If so, show an alert "This playlist has no tracks."
3. Network failure loading podcast feed: already handled in Prompt 2 with error display. Ensure errors propagate to HomeView and show a dismissible banner.
4. MusicKit authorization changes mid-session: observe MusicAuthorization.Status changes and show a non-blocking banner if access is revoked.

PERSISTENCE (UserDefaults):
Create Services/PersistenceService.swift:
- Save/load lastPodcastFeedURL: String? 
- Save/load lastPlaylistID: String? (MusicItemID as String)
- Save/load lastTalkDuration: Int (minutes, default 4)
- Save/load lastSongsPerBreak: Int (default 2)
On HomeView appear: restore last-used values and pre-populate the UI.
On session start: save the current values.

APP ICON:
Create a simple SF Symbol-based icon or solid colour placeholder. 
The icon concept: a waveform (waveform SF symbol) split half orange / half purple to represent podcast + music.
Create Assets.xcassets/AppIcon with at minimum a 1024x1024 image. Use a solid dark background (#0D0D0D) with the split waveform in white/orange/purple. If you can generate this programmatically with SwiftUI rendered to UIImage, do so; otherwise note where to drop in the PNG.

LAUNCH SCREEN:
Use Info.plist UILaunchScreen with a dark background and the app name centred — no separate LaunchScreen.storyboard needed.

FINAL CHECKLIST — verify each item works before committing:
- [ ] Full session plays from start to finish without crashing
- [ ] Lock screen shows correct metadata for both segment types
- [ ] Play/Pause from AirPods works
- [ ] Skip segment from lock screen works
- [ ] Last-used podcast and playlist are remembered across app launches
- [ ] Podcast ending mid-session shows "Session complete" gracefully
```

---

## Keys & Credentials Your Friend Needs to Provide

Before any device testing (the simulator can fake MusicKit auth), your friend will need:

1. **Apple Developer Account** — $99/year at developer.apple.com. Required to sign the app and use MusicKit on a real device.
2. **MusicKit Developer Token** — Generated from the Apple Developer portal:
   - Create a MusicKit key (Certificates, IDs & Profiles → Keys)
   - Use it to generate a JWT token signed with your private key
   - This token goes in Info.plist as `SKCloudServiceAPIKey` or is passed to `MusicDataRequest`
   - There are open source helpers (e.g. `apple-music-token-generator`) that do this for you
3. **Apple Music Subscription** — The actual Apple Music account used for testing needs an active subscription. The free tier won't work with MusicKit playback.

The podcast side (RSS + iTunes Search) requires **zero credentials** — it's all free public APIs.
