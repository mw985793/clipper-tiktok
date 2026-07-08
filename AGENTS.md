# Clipper TikTok — Agent / Orchestrator Instructions

You are an orchestrator agent. Whenever a user gives you a YouTube video URL, you must follow the exact pipeline below **in order**, folder by folder, without skipping or reordering steps.

---

## Core Rule — Directive Files

> **Before taking any action inside a folder, you MUST read that folder's directive file first.**
> The directive file is the authoritative instruction set for that step. Do not proceed until you have read and understood it.

---

## Pipeline Overview

```
YouTube URL
    │
    ▼
1  Transcript ──► 2  Clip Analysis ──► 3  Video Download ──► 4  Clip Export ──► 5  Vertical Reframe
```

---

## Folder Descriptions

### 1 — YouTube Transcript (`1/`)

**Directive file:** `1/youtube_transcript.directive.md`

**Purpose:** Fetch the transcript / subtitles for the given YouTube video.

- Detect the YouTube URL from the user's message.
- Use `yt-dlp` to download subtitles only — **never download the video in this step**.
- Prefer creator/manual subtitles (`--write-subs`); fall back to auto-generated captions (`--write-auto-subs`) if manual ones are unavailable.
- Convert the raw subtitle file into a clean, readable paragraph transcript by stripping cue numbers, timestamps, markup, duplicates, and extra whitespace.
- Use `1/youtube_transcript_agent.py` as the helper script.
- **Output:** a clean `.srt` transcript file saved inside `1/`.

---

### 2 — Transcript Clip Analysis (`2/`)

**Directive file:** `2/transcript_clip_analysis.directive.md`

**Purpose:** Analyse the transcript from folder `1` and identify the best moments to clip.

- Read the transcript produced in step 1 (must include timestamps).
- Use your own intelligence to find the most important, emotional, surprising, or standalone-worthy moments — no external APIs or sentiment services.
- Choose clean start and end times for each moment.
- **Output:** `2/clip_durations.txt` — a plain-text file listing only clip numbers and time ranges, with no extra commentary.

  ```
  Clip 1
  Time: 00:01:24 - 00:02:10

  Clip 2
  Time: 00:05:33 - 00:06:18
  ```

---

### 3 — Video Download (`3/`)

**Directive file:** `3/video_download.directive.md`

**Purpose:** Download the full source YouTube video at the best available quality.

- Use the same YouTube URL provided by the user.
- Download with `yt-dlp`, preferring up to `1080p`; fall back to `720p` if unavailable.
- Merge video and audio into a single `.mp4` file.
- Do **not** download subtitles in this step.
- Do **not** clip the video in this step.
- **Output:** a single `.mp4` file saved inside `3/downloads/`.

---

### 4 — Clip Export (`4/`)

**Directive file:** `4/video_clip_export.directive.md`

**Purpose:** Cut the downloaded full video into individual clips using the time ranges from folder `2`.

- Read `2/clip_durations.txt` for the start/end times.
- Locate the source `.mp4` in `3/downloads/`.
- Use FFmpeg (via `4/cut_clips.py`) to cut one output file per clip.
- Do **not** re-analyse the transcript or re-download the video in this step.
- **Output:** one `.mp4` per clip saved inside `4/clips/` (e.g. `clip_01.mp4`, `clip_02.mp4`, …).

---

### 5 — Vertical Reframe (`5/`)

**Directive file / README:** `5/README.md`

**Purpose:** Reframe each horizontal clip into a vertical `1080×1920` format suitable for TikTok / Shorts.

- Read `5/speakers.json` for the configured speaker crop positions (host and guest).
- Use `5/active_speaker_reframe.py` to render vertical clips that rotate the camera crop between the two speakers.
- Default rotation: guest focus for 3–4 s, then host focus for 2–3 s, repeating until the clip ends.
- Optionally use `5/speaker_timeline.csv` for manual frame-by-frame speaker timing control.
- **Output:** vertical `.mp4` files saved inside `5/vertical_clips/`.

---

## Summary — Execution Order

| Step | Folder | Action | Output |
|------|--------|--------|--------|
| 1 | `1/` | Fetch transcript | `1/*.srt` |
| 2 | `2/` | Identify best clip moments | `2/clip_durations.txt` |
| 3 | `3/` | Download full video | `3/downloads/*.mp4` |
| 4 | `4/` | Cut clips from video | `4/clips/clip_XX.mp4` |
| 5 | `5/` | Reframe clips to vertical | `5/vertical_clips/clip_XX.mp4` |

Always execute these steps **in order**. Do not skip a step. Do not start a step before reading its directive file.
