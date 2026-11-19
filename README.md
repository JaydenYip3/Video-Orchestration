# Narrated animated shorts: how to use

You write a brief. Claude Code produces the finished vertical video: a voice from Breeze TTS, cartoon-3D animation,
sound effects, music, and a 1080x1920 mp4.

## Make a video
1. Write `briefs/<name>.md`. Copy `briefs/EXAMPLE.md` as a starting point. It has three sections:
   - **Script**: the exact words. Optional markers: `[beat]`, `[pause]`, `[long pause]`, `[pause 1.5]`, and vocal
     events like `(sigh)`.
   - **Voice**: who is speaking, how fast, where to pause, how to deliver particular lines. Plain English is fine.
   - **Style**: place, era, time of day, mood, music feel, camera energy. A few sentences.
2. Start Claude Code in this folder (`cd /root/code/orchestration && claude`) and say:
   > make the short from briefs/<name>.md
   or `/narrated-short briefs/<name>.md`.
3. Wait. The result is `projects/<name>/final/<name>.mp4`, plus `REPORT.md` next to it. Progress is tracked in
   `projects/<name>/STATUS.md`. If a session ends midway, start Claude here again and say "resume <name>".

The full format is in `.claude/skills/narrated-short/references/brief-format.md`.

## What happens
| Stage | Who | Output |
|---|---|---|
| 0 Preflight | main Claude, `tools/check_env.sh` | one-time setup of anything missing (Remotion project etc.) |
| 1 Intake | main Claude | SCRIPT.md, STYLE.md, DECISIONS.md, STATUS.md |
| 2 Voice | `voice-producer` agent | `voice/voice.mp3`, `voice/voice.wav` (48 kHz, used by the mix), `words.json` (per-word times), `segments.json` |
| 3 Picture + sound | main Claude following `claude-animated-shorts/SKILL.md`, with Opus verifier agents | shots timed to your words, world, characters, SFX, music, mix |
| 4 Deliver | main Claude | mp4 (1080x1920 by default), frame-by-frame QA, REPORT.md |

How the voice is made:
- Breeze has no pause syntax, and voice design invents a new voice on every request.
- So the voice stage turns your narrator's description into one short clip, then copies that clip's voice for
  every sentence, with exact silences between the sentences.
- Whisper checks every sentence against the script. Any wrong or missing word causes the line to be regenerated.

## Your voices
Keep hand-picked narrators as text files in `/root/code/breeze_tts/voices/`, one per voice, e.g.
`deep_warm_storyteller.txt`:
```
A deep, warm male storyteller, slow and calm.
seed: 42
```
- The text is the description. `seed` picks which of the many voices matching that description you get: change it
  for a different person with the same description. `cfg` (default 3) and `sample:` (the audition sentence) are
  optional too.
- Use a file in a brief with `Voice file: deep_warm_storyteller`. Name a file `default.txt` and it is used whenever
  a brief doesn't say.
- The same file always gives the same narrator. Edit the file and the narrator changes.
- The format is defined and read by breeze_tts itself (`file.read_voice`); the full reference is
  `/root/code/breeze_tts/README.md`. `cfg-3.5` and `cfg=3.5` work as well as `cfg: 3.5`.
- Generated clips live in `/root/code/orchestration/.voice_cache/`. They're disposable, and nothing generated is
  ever written into `voices/`.

## Status (2026-11-17)
- Voice stage: built and tested end-to-end. `bash tools/selftest_voice.sh` should print `SELFTEST OK`.
- Animation stage: the Remotion project at `/root/code/videos/remotion-shorts` is installed for Linux. Its
  node_modules was reinstalled because the Mac copy had macOS binaries. The GPU renders the bridge template
  correctly at ~4 s per preview still.
- Not yet done:
  - a full-length final render on this machine
  - the optional Kenney 3D props (trees, stalls): without them those props are just missing
- Time per video: the voice takes minutes. The animation takes hours, because of shot authoring, several
  look-verification rounds and a ~5-10 min final render.

## Resolution
The default is 1080x1920. To change it for one video, add a line to the brief's `## Notes`:
`Resolution: 720p`, `1080p`, `2K` or `4K`. To change the default for every video, edit the table in Phase 4 of
`.claude/skills/narrated-short/SKILL.md`.
