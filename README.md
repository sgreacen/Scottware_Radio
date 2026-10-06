# Scottware Radio

**A Scottware utility.** Internet radio player for macOS, Windows, and Linux.

Created by **Scottware Software** — Built clean. Runs smooth. 
Search and play thousands of internet radio stations from three live directories, play your own **FLAC, MP3, AAC and other local music files**, rip and play **audio CDs**, or drop in direct stream URLs. Builds with [Electron](https://www.electronjs.org/) and [React](https://react.dev/).

## Features

### Radio

- **Directory search** across:
  - **Radio Browser** — community index (free, no key, includes Shoutcast + Icecast stations)
  - **Icecast** — official Xiph directory (`dir.xiph.org/yp.xml`), cached locally for fast offline search
  - **Shoutcast** — official directory (requires a free Developer ID, stored in Settings)
- **Direct URL playback** with automatic `.pls` / `.m3u` playlist expansion
- **Live song titles & album art** — reads ICY `StreamTitle` metadata from Shoutcast/Icecast streams and resolves album art from the keyless iTunes Search API, shown in the player bar, mini player, and macOS media-session widget

### Local music (v2.0)

- **Music library** — add folders (scanned recursively) or pick individual files; folders are rescanned each time the view opens
- **Formats** — FLAC, MP3, AAC, M4A, WAV, OGG, Opus and AIFF
- **Tags and durations are read directly from the file** — FLAC (STREAMINFO + Vorbis comments), MP3 (ID3v2 + Xing), AAC (ADTS), WAV and Ogg/Opus, so sorting, track numbers and the scrubber are populated before playback starts
- **Cover art** — picks up `cover` / `folder` / `album` / `front` images sitting next to the audio
- **Seek, scrub and next/previous** for every finite file, with automatic advance to the next track when one ends
- Files are streamed from disk over a private `media://` protocol with byte-range support, so long FLAC files seek instantly

### Audio CDs (v2.0)

- Detects optical drives on macOS, Linux and Windows, with a manual override for drives that are not listed
- Reads the table of contents and lists every audio track with its running time
- Rips tracks with `cdparanoia`, `cdda2wav` or `cdda`, reports progress, and caches the rip so replays are instant
- Requires the [cdrtools](https://www.cdrtools.org/) utilities on `PATH` (`brew install cdrtools` on macOS, `sudo apt install cdrtools` on Linux, cdrtools for Windows on Windows); the Audio CD view tells you exactly what is missing

### Playlists (v2.1)

- **Local playlists** — `.m3u`, `.m3u8` and `.pls` files can be opened straight from the *Add files or playlists* dialog and expand into their tracks, with `#EXTINF` titles, artists and durations carried over
- **Playlist URLs** — paste a `.m3u` or `.pls` URL in *Add URL* and every track is imported into your library and favorites; pressing play then queues the whole list from the track you clicked
- `#EXTINF`, `.pls` `File1=`/`Title1=` blocks, relative paths, `file://` entries and Windows paths are all handled; a playlist you cannot read is left as-is instead of being dropped
- Media playlists (`.m3u8`) are added as a single stream — their segments cannot be queued individually, and Electron's bundled Chromium does not decode HLS audio

### Spotify (v2.1)

- **Browse and play** your playlists, liked songs, saved albums and followed artists from a dedicated sidebar view, plus catalog search
- Plays through Spotify's own **Web Playback SDK**, so Scottware Radio appears as a Connect device in the Spotify desktop and mobile apps
- Sign-in uses **OAuth Authorization Code + PKCE** — no client secret is stored, and tokens live in a `0600` file under the app's user-data directory and refresh automatically
- Progress, scrubbing, volume, next/previous, favorites and the player bar all work like the other sources

Setup: Spotify playback requires a **Premium** account. Register an app at the
[Spotify Developer Dashboard](https://developer.spotify.com/dashboard/), select **Web Playback SDK** when asked which
APIs you plan to use, and add `http://127.0.0.1:8898/callback` as a redirect URI. Paste the resulting Client ID into
the Spotify view (or Settings) — that is all the app needs.

### Everything else

- **Favorites** and **recently played**, persisted locally in `~/Library/Application Support/Scottware Radio`
- **Mini player** — compact always-on-top window (also a proper full-width bar in the main window)
- **Media keys** support (macOS keyboard controls / Media Session), including next/previous track
- Branded **Scottware Radio** app icon on every platform

## Stack

- Electron 33 + electron-vite (main / preload / renderer)
- React 18 + TypeScript (strict)
- electron-builder for installers
- No runtime dependencies: audio tags, durations, CD TOCs and range streaming are all handled in-process

## Development

```bash
npm install
npm run dev        # launch with hot reload
npm run typecheck  # type-check main + renderer
npm run build      # production build into out/
```

### Playing audio CDs locally

CD playback shells out to the standard cdrtools binaries. Install them once:

```bash
brew install cdrtools            # macOS
sudo apt install cdrtools        # Debian / Ubuntu
```

On Windows install cdrtools so that `cdda2wav.exe` (or Nero's `cdda.exe`) is on `PATH`. Scottware Radio uses whichever it finds first: `cdparanoia`, `cdda2wav`, `cdda`.

## Packaging

Icons are generated from `scripts/generate-icon.swift` and written to `build/`:

```bash
npm run icon
```

### macOS

```bash
npm run dist:mac
```

Produces in `release/`:

- `Scottware Radio-<version>-arm64.dmg` (drag-to-Applications installer)
- `Scottware Radio-<version>-arm64-mac.zip` (for auto-update / distribution)

Signing: builds unsigned by default when no team is configured in `package.json`.

### Windows

Cross-build from macOS or build on Windows:

```bash
npm run dist:win
```

Produces in `release/`:

- `Scottware Radio Setup <version>.exe` (NSIS installer)
- `Scottware Radio <version>.exe` (portable single-file build)

Note: building the NSIS installer on macOS requires [Wine](https://electron.build/multi-platform-build) for the final steps.

### Layout / outputs

| Command | Output |
| --- | --- |
| `npm run dist` | Unpacked app in `release/` to sanity-check |
| `npm run dist:mac` | `.dmg` + `.zip` |
| `npm run dist:win` | NSIS `.exe` + portable `.exe` |
| `npm run dist:linux` | AppImage |

## Project layout

```
scripts/generate-icon.swift   App icon generator
build/                        Icons (icon.png, icon.icns)
src/main/                     Electron main process: window, IPC, providers, store
src/main/providers/           radioBrowser.ts, icecast.ts, shoutcast.ts
src/main/media/               formats.ts, tags.ts, library.ts, protocol.ts, cd.ts
src/preload/                  Context-isolated bridge
src/renderer/src/             React UI
src/shared/types.ts           Shared TypeScript contracts
```

## Privacy

All directory data comes from the public APIs above. Favorites and settings never leave the machine.
