---
name: voiceless-short
description: Fully automated production of an animated short with no narration, from only an animation style and an animation direction (what should happen on screen). There is no script, so no voice is made and nothing is transcribed. Picks the art style from /root/code/videos/styles, then hands the direction to a new agent that follows that style's skill (/root/code/videos/.claude/skills/<style>/SKILL.md) to build and render the video. Use when the user gives a style and a direction without a script, says "make a voiceless/silent short", "no voice", "no narration", "just animate this", "make an animation about X", or invokes /voiceless-short. If the user gives a script to be spoken, use narrated-short instead.
---

# Input

Two things, given in the message or in a brief file `briefs/<label>.md`:

- **Style**: which art style to use. Optional, see "Style" below.
- **Direction**: what the animation shows. It can be one sentence ("explain how a teleporter would work") or a
  beat-by-beat description with a setting, characters, on-screen text and a length.

There is no script. Do not write one, do not run `/root/code/breeze_tts/mod.py` or `transcribe`, and do not start
the TTS server. If the user did give words to be spoken, this is the wrong skill: use `narrated-short`.

# Labels

Every video gets its own labelled folder in outputs, and every file in it carries the same label.

- The label is the brief's file name without `.md` (`briefs/teleporter.md` → `teleporter`). If there is no brief
  file, make a short lowercase-hyphenated label from the direction's topic.
- If `outputs/<label>/` already exists from an earlier video, add a number: `<label>-2`, `<label>-3`, ...
- Folder: `/root/code/orchestration/outputs/<label>/`

| File | What it is |
|---|---|
| `<label>.txt` | the style and direction exactly as the user gave them |
| `<label>.mp4` | the final video |

Create the folder and write `<label>.txt` first.

# Style

Every art style is one folder in `/root/code/videos/styles/`. Each style has a skill with the same name that teaches
how to make a video in it: `/root/code/videos/.claude/skills/<style>/SKILL.md`.

1. List the styles with `ls /root/code/videos/styles`.
2. Pick one, without asking:
   - If the user names a style (a `Style: <name>` line, or the folder name anywhere in their text), use it.
   - If only one style exists, use it.
   - Otherwise read the `description` at the top of each style's SKILL.md and take the best match for the direction.
3. Where the direction asks for something the style's rules forbid, the style's rules win.

# Video

Nothing else uses the GPU in this skill, but a TTS server left running from earlier would. Confirm nothing is
listening on port 8090 (`ss -ltn | grep 8090` prints nothing); if something is, stop it before the render.

Spawn a new agent with `model: "opus"` (Opus 5.5) and `effort: "high"` that reads
`/root/code/videos/.claude/skills/<style>/SKILL.md` and follows it to make a new video in that style from the
direction in `outputs/<label>/<label>.txt`. Tell it:

- **Name.** The video is called `<label>`. The style's skill says where a new video's folder goes. If a folder with
  that name is already there, it belongs to another video: add a number, never overwrite it.
- **No voice.** The video has no narration, so the picture has to carry the whole meaning: staging, acting and the
  style's own on-screen labels and text. Where the direction gives exact on-screen words, use them unchanged.
- **Timing.** If the direction gives a length or beat times, keep them. Otherwise plan the beats first and make the
  video as long as they need to read clearly, with every label on screen long enough to read.
- **Sound.** The style's own sound (effects, and music if the style has it) is the whole soundtrack. Nothing is
  mixed in afterwards.
- **Files.** Copy the style's finished render to `outputs/<label>/<label>.mp4`. That is the final video. Stills,
  scratch files and `TMPDIR` go in `outputs/<label>/build/`.
- **Format.** Frame size and shape are whatever the style renders. Do not force another one.
