---
name: screen-video
description: >-
  Record the screen and edit screen recordings into polished videos with ScreenForge. Use when
  the user wants to record their screen, a demo, walkthrough, tutorial, bug repro or product
  video, or to edit, tighten, caption, narrate, restyle or export a screen recording, a
  .screenforge or .screenstudio project, a video file or a Loom link.
---

# Screen videos with ScreenForge

The `screenforge` MCP server records the screen and edits ScreenForge projects (`.screenforge`
packages). Every editing tool takes `project`, the package path, and saves straight away. If
the project is open in the ScreenForge app, the user sees each edit there live.

If the screenforge tools are missing or fail to start, ScreenForge is not installed or is
older than 0.1.66: point the user to https://screenforge.studio (or
`brew install zot24/tap/screenforge`) and ask them to restart the agent.

## 1. Get a project

- An existing one: `list_projects` (newest first), or the path the user gives.
- A video file, a Screen Studio project or a public Loom link: `import_video`.
- A new recording:
  1. `list_capture_sources` shows the displays, windows and microphones, and whether Screen
     Recording and Input Monitoring are granted. Without Input Monitoring there are no click
     zooms; tell the user it is granted to the app that runs the agent (their terminal), in
     System Settings → Privacy & Security, and that app must restart afterwards.
  2. Ask what to record when it is unclear: the whole display, or one window (pass a word of
     its title or app name, e.g. `"Safari"`). Ask whether to include the microphone.
  3. `start_recording`, then tell the user it is recording and to tell you when they are done.
     Do not stop it yourself unless they asked for a fixed length.
  4. `stop_recording` makes the project and returns its path. Automatic zooms around the
     clicks are added unless `auto_zooms` is false.

## 2. Edit

- Start with `get_project`. Read again whenever the user may have edited in the app.
- Times are milliseconds on the edited timeline. Find moments with:
  - `get_transcript`, after `transcribe` (Whisper, on this computer)
  - `get_activity` for typing, clicks and idle stretches
  - `get_motion`, then `view_frames` to see the raw screen
- `cut` takes every range of one edit in one call. Leave about 100 ms around neighbouring words.
- Zoom gently: 1.3–1.5× for text and typing, about 2× only for a small detail. Zooms, masks and
  camera layouts cannot overlap: update or remove the one in the way.
- Look settings (background, padding, corners, shadow, cursor, captions, aspect ratio) go
  through `patch_project`. `get_options` lists the values and where each setting lives.
- "Tighten it" / auto edit:
  - With a transcript: cut long pauses and filler words.
  - Speed up stretches where nothing is said or clicked, 2–4×.
  - Add zooms around clicks if there are none (`add_auto_zooms`).
- Narration: `add_narration` with an on-device voice (am_fenrir, am_michael, am_puck,
  af_heart, af_bella, af_nicole).
- Titles and transitions: `add_title`, `set_transition`.

## 3. Check, then deliver

- Use `render_frame` at the moments you changed to see the finished look before saying it is
  done.
- Use `export` when the user wants the file. It writes mp4 by default (gif or webm on
  request), next to the project unless `out` is given, plus a `.details.txt` with a title,
  description and tags.
- Report what changed with times as m:ss. `undo_answers` reverts your edits if the user does
  not like them.
