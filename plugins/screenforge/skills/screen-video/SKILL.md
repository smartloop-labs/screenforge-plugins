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

`ask_editor`, `set_format`, `join_project`, `open_project` and the templates (`list_templates`,
`new_from_template`) come with ScreenForge 0.1.69. If they are missing, the user's ScreenForge
is older: tell them to update, and edit with the other tools meanwhile.

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

Nothing to record (an explainer, a feature video, an announcement, a how-to)? Call
`list_templates`, then `new_from_template` with a hook, the beats (a two-line headline, one
spoken sentence, and a card you draw as SVG following the guide `list_templates` returns) and
an end card. It returns an ordinary project: add music, check each part with `render_frame`,
then continue with step 3.

## 2. Edit

**The short way: `ask_editor`.** When the user describes the video they want ("a 30 s vertical
TikTok of this with narration by Eric, a mountain photo behind and music from my favourites"),
pass the whole description as `request`. ScreenForge's own AI editor looks through the
recording, cuts, zooms, narrates and styles it, and answers with a short report, so the
transcript and the frames never enter your context. Put every wish of the user in the request
(length, format such as 9:16, voice, background, music): the editor cannot ask back. Call it
again for changes; the chat continues. It runs on the user's ScreenForge plan: if the tool says
the plan does not include it, tell the user and edit with the tools below, which are free.

**By hand**, for a small change or without the plan:

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
- Narration: `add_narration`. Leave `voice` out to read with the narrator chosen in the app;
  when the user names one, pass it: on-device (am_fenrir, am_michael, am_puck, af_heart,
  af_bella, af_nicole), ElevenLabs by name (Eric, Brian, Sarah …, studio quality, free) or their own voice (get_options myVoices).
- Music: the user's favourites first (get_options: `music` entries with favorite: true go to
  `set_music`, `favoriteMusic` Jamendo ids to `use_music`); otherwise `search_music`.
- Backgrounds: a wallpaper or gradient through `patch_project`, or an Unsplash photo with
  `search_photos` then `use_photo`.
- Titles and transitions: `add_title`, `set_transition`.

## 3. Check, then deliver

- Use `render_frame` at the moments you changed to see the finished look before saying it is
  done.
- Use `export` when the user wants the file. It writes mp4 by default (gif or webm on
  request), next to the project unless `out` is given, plus a `.details.txt`, and for mp4
  uploads it and returns `link`, a page anyone can watch: give the user that link and the
  file path. Export is the one paid step: a computer without a license has two free exports;
  when the tool says they are used, pass on how to get Pro or Creator instead of retrying. Write its
  `title`, `description` (a few sentences on what the video shows) and `tags` yourself and pass
  them to `export`: ScreenForge runs no model here, so without them the file is only a template.
- `export` returns `credits` when the video uses Creative Commons music or an Unsplash photo
  (it also writes them under the description). Give the user those lines as they are for the
  caption or description wherever they publish; do not write credits of your own.
- For TikTok, Reels or Shorts call `set_format` with the platform before placing any text:
  it sets 1080×1920 and keeps captions and titles inside the safe area the app leaves clear.
  Place titles within the `titleY` range it returns.
- A promo or feature short (TikTok, Reels, Shorts): `set_format`, a title card as the hook, short
  headlines over the footage timed to the narration, highlight captions, more shots with
  `join_project` and a transition at each cut, music kept low, a closing title card with the
  call to action. `ask_editor` builds all of it from one description.
- `open_project` opens the project in the ScreenForge app when the user wants to watch it or
  go on editing by hand.
- Report what changed with times as m:ss. `undo_answers` reverts your edits if the user does
  not like them.
