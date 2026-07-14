# Audio Visualizer Studio

A single-file, browser-based audio visualizer for creating background videos for music, podcasts, and audio clips. Drop in an audio file, a background, and a profile picture, choose a look, and record a clean **1080p** video — no install, no build step, no server.

Everything the video shows is drawn on one `<canvas>`; the on-screen controls sit *on top* of it, so they're never captured. The result is a crisp, button-free clip in **16:9**, **9:16**, or **1:1**, up to **4K**.

## Features

- **Circular profile image** centered over a background, ringed by an audio-reactive visualizer.
- **Six visualizer styles** — radial bars, morphing solid blob, smooth curve lines, circular waveform, mirrored bottom spectrum, and a particle ring.
- **Full color control** — preset solid colors, preset gradients, custom color, or custom multi-stop gradient.
- **Backgrounds** — upload an image or a **video**, or pick from 13 built-in presets (8 dark + 5 light); background blur, video speed, **seamless crossfade looping** (the video's end blends into its start — no visible cut), and adjustable vignette density.
- **Particle overlay (FX)** — upload a particle / bokeh / light-leak clip that composites over the whole scene with a **dodge blend** (Screen / Add / Color Dodge — black becomes transparent, bright pixels add light), with opacity and speed controls, seamless crossfade looping, and it's captured in recordings.
- **Positioning** — zoom and pan both the profile picture and the background; resize the central circle. Sliders **snap** to their neutral/default positions (marked with tick marks) so it's easy to re-center.
- **Orientation & resolution** — **16:9 / 9:16 / 1:1**, output at **720p / 1080p / 1440p / 4K**.
- **Recording** — capture the composition at the chosen resolution as **MP4 (H.264)** or **WebM**, at **24 / 30 / 60 fps**, with optional embedded audio.
  - **Pause/resume** mid-recording (paused time is excluded from the video).
  - **Save destination** — normal browser download, **ask each time**, or **auto-save to a remembered folder** (File System Access API, with graceful fallback).
  - **On-screen recording progress** and a **3-2-1 countdown** — both helpers are excluded from the video.
  - **⚡ Fast export** — render the whole clip **offline, faster than real-time** (no waiting the full duration); the file still plays at normal **1× length**. It is **silent** (nothing plays through your speakers) and **keeps running in a background tab**, so you can start it and go back to other tabs / Spotify. Uses WebCodecs + an inlined muxer; works with **still backgrounds** (image/preset) in Chromium browsers, and falls back to real-time recording when a background/overlay video is in use or WebCodecs is unavailable.
  - **Mute audio while recording** — a toggle so real-time recording doesn't play through your speakers (the file still gets the audio). Only your loaded clip is ever recorded — other tabs (e.g. Spotify) are never captured, since audio is tapped in-page, not via screen/tab capture.
- **Look presets** — save/load named looks, and export/import all settings as a `.json` file.
- **Persistence** — background, profile, and all appearance settings are remembered across reloads (via `localStorage`).
- **Lightweight when idle/backgrounded** — the render loop is frame-rate capped (throttled further when idle), and the background video is paused while the tab is hidden, so the tool stays easy on your CPU/GPU and doesn't bog down other tabs.

## Getting started

No installation required.

1. Download or clone this repo.
2. Open **`index.html`** in Chrome or Edge (double-click, or drag it into the browser).

That's it.

## Usage

1. Click the **⚙ gear** (top-right) to open Settings.
2. Load your **Audio clip**, and optionally a **Background image** and **Profile picture** (or choose a background preset).
3. Pick your **orientation**, **visualizer shape**, and **color / gradient**.
4. Fine-tune with **circle size**, **profile zoom/position**, and **background zoom/position**.
5. Close Settings, then:
   - Press **▶ Play** (bottom-center) to preview live, or
   - Press the **● Record** button (bottom-right) to capture a video.

**Keyboard:** `Space` = play/pause · `Esc` = close settings.

## Recording notes

- Clicking **Record** plays your clip from the start and records the visual; it **auto-stops when the clip ends** (or click again to stop early). The file downloads automatically as `visualizer-<timestamp>.mp4` (or `.webm`).
- **Choose the format** in Settings → Recording: **MP4** (H.264, universally compatible — default) or **WebM** (VP9/VP8). MP4 is recorded natively by the browser, so there's no slow conversion step; if a browser can't record MP4, it falls back to WebM automatically.
- **Choose the frame rate** (24 / 30 / 60 fps). **30 fps is the default** and imports most reliably into editors.
- **Pause/resume** with the pause button that appears while recording — paused time is **not** added to the video, so you can step away and continue.
- **Where files save** (Settings → Recording → *Save recordings to*): **Browser** (default download), **Ask each time** (native Save dialog), or **Folder** (auto-save to a folder you pick once). The Ask/Folder modes use the File System Access API and fall back to a normal download when unavailable (e.g. when the page is opened directly as a `file://` file rather than over a local server).
- **Progress readout** and the **3-2-1 countdown** are on-screen helpers only — they're never in the recording. Both can be toggled off.
- **Look presets:** save a named look and reload it later, or **Export/Import** all settings as a `.json` file to move them between machines.
- **Fast export** (Settings → Recording → *⚡ Fast export*) renders offline as fast as your machine can encode and outputs a correct-length 1× file. MP4 fast-export needs the browser's AAC encoder (recent Chrome/Edge); where that isn't available it exports **WebM (VP9/Opus)**. It uses the MIT-licensed [`mp4-muxer`](https://github.com/Vanilagy/mp4-muxer) and [`webm-muxer`](https://github.com/Vanilagy/webm-muxer), inlined into `index.html`.

### Editor compatibility (DaVinci Resolve, etc.)

Browser (`MediaRecorder`) files are **variable frame rate (VFR)**, which some editors — notably DaVinci Resolve — can reject or misread (wrong duration, "media offline"), regardless of bitrate. If an import misbehaves:

1. Record at **30 fps** (Settings → Recording).
2. If it still won't import, re-encode to **constant frame rate** with ffmpeg:
   ```bash
   ffmpeg -i visualizer.mp4 -c:v libx264 -r 30 -pix_fmt yuv420p -c:a aac -b:a 192k visualizer_cfr.mp4
   ```
   (HandBrake's "Fast 1080p30" preset does the same.)
3. If only the audio fails, record with **"Include audio in video" off** and add the audio track in your editor.
- Use the **"Include audio in video"** toggle to record with sound baked in, or silent visuals only (handy if you'll add audio in an editor).
- The gear, play, record buttons, and the "REC" badge are HTML overlays — **never** part of the recording.
- **Keep the tab focused and visible while recording.** Browsers pause canvas animation in hidden/minimized tabs, which would freeze the captured frames.
- Output imports directly into Premiere, DaVinci Resolve, and CapCut — **MP4 (H.264/AAC)** is the most broadly compatible; **WebM (VP9/VP8)** is also available.

## Browser support

Works in **Chromium-based browsers (Chrome, Edge)**, which fully support `canvas.captureStream()` and `MediaRecorder`. Recent Chrome/Edge record **MP4 (H.264)** natively; where that isn't available the tool falls back to **WebM**. Other browsers may vary in recording support.

## How it works

- **Web Audio API** — a hidden `<audio>` element feeds an `AnalyserNode`; frequency data drives the visualizer, and bass energy pulses the profile's glow ring.
- **Canvas 2D** — background, visualizer, and profile are composited each frame via `requestAnimationFrame`.
- **MediaRecorder** — `canvas.captureStream(60)` provides the video track; the audio track (from a `MediaStreamAudioDestinationNode`) is added only when "include audio" is on.

The whole app is one self-contained `index.html` (HTML + CSS + JS inline), so it runs straight from the file system.

## License

Released under the [MIT License](LICENSE).
