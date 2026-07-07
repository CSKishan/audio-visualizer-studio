# Audio Visualizer Studio

A single-file, browser-based audio visualizer for creating background videos for music, podcasts, and audio clips. Drop in an audio file, a background, and a profile picture, choose a look, and record a clean **1080p** video — no install, no build step, no server.

Everything the video shows is drawn on one `<canvas>`; the on-screen controls sit *on top* of it, so they're never captured. The result is a crisp, button-free clip in either **16:9** or **9:16**.

## Features

- **Circular profile image** centered over a background, ringed by an audio-reactive visualizer.
- **Three visualizer shapes** — radial bars, morphing solid blob, or multiple smooth curve lines.
- **Full color control** — preset solid colors, preset gradients, custom color, or custom multi-stop gradient.
- **Backgrounds** — upload your own image, or pick from 13 built-in presets (8 dark + 5 light).
- **Positioning** — zoom and pan both the profile picture and the background; resize the central circle.
- **Toggles** — rotate the visualizer on/off; enable/disable the background vignette & dimming.
- **Orientation** — one click switches between 16:9 landscape and 9:16 portrait.
- **Recording** — capture the composition at exactly **1920×1080** / **1080×1920** as **MP4 (H.264)** or **WebM**, at **24 / 30 / 60 fps**, with an option to embed the audio track or record silent visuals.
- **Persistence** — background, profile, and all appearance settings are remembered across reloads (via `localStorage`).

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
