---
name: narrated-short
description: End-to-end, fully automated production of a narrated animated vertical short from a user brief (script + notes on pauses/delivery + a short style description). Makes the narration with the local Breeze TTS (/root/code/breeze_tts/mod.py), transcribes it to get timings, then hands the audio and its .srt to the claude-animated-shorts pipeline to build and render the video with Remotion (1080x1920 by default). Use when the user says "make a short/video from briefs/<x>.md", "run the full pipeline", "here's my script, make the video", or invokes /narrated-short.
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

# Video

Spawn a new agent with `model: "opus"` (Opus 5.5) and `effort: "high"` that reads `/root/code/videos/claude-animated-shorts/SKILL.md` and follows it to create the
video with Remotion, using `outputs/<label>/<label>.wav` and `outputs/<label>/<label>.srt` plus the brief's style
description. Tell it to save the final render as `outputs/<label>/<label>.mp4`.
