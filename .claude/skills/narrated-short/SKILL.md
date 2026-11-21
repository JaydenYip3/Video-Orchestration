---
name: narrated-short
description: End-to-end, fully automated production of a narrated animated short from a user brief (script + notes on pauses/delivery + a style). Makes the narration with the local Breeze TTS (/root/code/breeze_tts/mod.py), transcribes it to get timings, picks an art style from /root/code/videos/styles, then hands the audio and its .srt to a new agent that follows that style's skill (/root/code/videos/.claude/skills/<style>/SKILL.md) to build and render the video. Use when the user says "make a short/video from briefs/<x>.md", "run the full pipeline", "here's my script, make the video", or invokes /narrated-short.
---

# Labels

Every video gets its own labelled folder in outputs, and every file in it carries the same label.

- The label is the brief's file name without `.md` (`briefs/lighthouse-keeper.md` → `lighthouse-keeper`). If there
  is no brief file, make a short lowercase-hyphenated label from the story's title or topic.
- If `outputs/<label>/` already exists from an earlier video, add a number: `<label>-2`, `<label>-3`, ...
- Folder: `/root/code/orchestration/outputs/<label>/`

| File | What it is |
|---|---|
| `<label>.txt` | the script that was spoken |
| `<label>.wav` | the narration |
| `<label>.srt` | the transcript with timings |
| `<label>.mp4` | the final video |

# Voice

To create the voice, use `/root/code/breeze_tts/mod.py` with these flags:

| Flag | Description | Default |
|---|---|---|
| `--file story.txt` | speak a text file instead of `--text` | none |
| `--voice <name>` | voice file, a name in `/root/code/breeze_tts/voices/` or a path; sets the description, cfg and seed | none |
| `--out out.wav` | output file, `.wav` or `.mp3` by extension | `out.wav` |

1. Create the folder `outputs/<label>/` and write the script from the brief to `outputs/<label>/<label>.txt`.
   Copy the words exactly; never change them. Leave out the pause/delivery notes themselves; they are directions,
   not words.
2. Choose the voice that best matches the story, unless I specifically tell you which one to use. List the
   options with `ls /root/code/breeze_tts/voices/`.
3. Run mod.py with the three flags and wait for it to finish. `mod.py` changes into its own folder, so always pass
   absolute paths:

   ```bash
   python /root/code/breeze_tts/mod.py \
     --file /root/code/orchestration/outputs/<label>/<label>.txt \
     --voice <name> \
     --out /root/code/orchestration/outputs/<label>/<label>.wav
   ```

4. `mod.py` stops the TTS server when it finishes. Confirm it is stopped (nothing listening on port 8090, e.g.
   `ss -ltn | grep 8090` prints nothing); if it is still running, stop it. The GPU is shared, so this must happen
   before any render.

# Transcribe

Run the transcribe alias on the file you just made:

```bash
cd /root/code/orchestration/outputs/<label> && transcribe <label>.wav
```

It writes `<label>.srt` next to it.

# Style

Every art style is one folder in `/root/code/videos/styles/`. Each style has a skill with the same name that teaches
how to make a video in it: `/root/code/videos/.claude/skills/<style>/SKILL.md`.

1. List the styles with `ls /root/code/videos/styles`.
2. Pick one, without asking:
   - If the brief names a style (a `Style: <name>` line, or the folder name anywhere in its style text), use it.
   - If only one style exists, use it.
   - Otherwise read the `description` at the top of each style's SKILL.md and take the best match for the brief.
3. The rest of the brief's style text (place, era, mood, characters, sound) is direction for the video inside that
   style. Where it asks for something the style's rules forbid, the style's rules win.

# Video

Spawn a new agent with `model: "opus"` (Opus 5.5) and `effort: "high"` that reads
`/root/code/videos/.claude/skills/<style>/SKILL.md` and follows it to make a new video in that style, using
`outputs/<label>/<label>.wav` and `outputs/<label>/<label>.srt` plus the brief's style text. Tell it:

- **Name.** The video is called `<label>`. The style's skill says where a new video's folder goes. If a folder with
  that name is already there, it belongs to another video: add a number, never overwrite it.
- **Timing.** The narration sets the timeline. Take every scene and beat time from the `.srt`, so each picture is on
  screen while its words are spoken. The video is as long as the `.wav` plus about one second.
- **Sound.** The narration is the main audio. The style's render gives the picture with the style's own sound
  effects and no voice, so mix the narration over it with the effects turned down:

  ```bash
  ffmpeg -y -i <the style's render>.mp4 -i /root/code/orchestration/outputs/<label>/<label>.wav -filter_complex \
    "[0:a]volume=-10dB[fx];[1:a]aresample=48000,pan=stereo|c0=c0|c1=c0[vo];[fx][vo]amix=inputs=2:duration=first:normalize=0,alimiter=limit=0.89:level=false[a]" \
    -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k /root/code/orchestration/outputs/<label>/<label>.mp4
  ```

- **Files.** The mixed file above, `outputs/<label>/<label>.mp4`, is the final video. Stills, scratch files and
  `TMPDIR` go in `outputs/<label>/build/`.
- **Format.** Frame size and shape are whatever the style renders. Do not force another one.
