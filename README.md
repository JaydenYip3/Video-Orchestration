# Narrated animated shorts: how to use

You write a brief. Claude Code produces the finished video: a voice from Breeze TTS over an animation drawn in one
of your art styles, as one mp4.

## Make a video
1. Write `briefs/<name>.md`. Copy `briefs/EXAMPLE.md` as a starting point. It has three sections:
   - **Script**: the exact words. They are never changed. Notes in brackets such as `[pause]` are directions and
     are not spoken.
   - **Voice**: who is speaking, how fast, how to deliver particular lines. Plain English is fine.
   - **Style**: optionally a `Style: <name>` line naming a folder in `/root/code/videos/styles`, then a few
     sentences on place, era, mood, characters and sound.
2. Start Claude Code in this folder (`cd /root/code/orchestration && claude`) and say:
   > make the short from briefs/<name>.md
   or `/narrated-short briefs/<name>.md`.
3. Wait. The result is `outputs/<name>/<name>.mp4`. The script, narration and transcript are next to it as
   `<name>.txt`, `<name>.wav` and `<name>.srt`. If `outputs/<name>/` already exists, the new video goes to
   `<name>-2`, `<name>-3` and so on.

## What happens
| Stage | Who | Output |
|---|---|---|
| 1 Voice | main Claude, `/root/code/breeze_tts/mod.py` | `<name>.txt`, `<name>.wav` |
| 2 Transcribe | main Claude, `transcribe` | `<name>.srt` (when each line is spoken) |
| 3 Style | main Claude | the style folder to use |
| 4 Video | a new agent following the style's skill | scenes timed to the `.srt`, rendered, narration mixed in, `<name>.mp4` |

The steps are defined in `.claude/skills/narrated-short/SKILL.md`.

## A video with no voice
If you have no script and only want an animation, give a style and a direction (what should happen on screen):
> /voiceless-short sci-fi-sitcom-thinline: Vic explains why the sky is blue, about 12 seconds

You can also put the same two things in `briefs/<name>.md` and say "make the voiceless short from
briefs/<name>.md". No voice is made and nothing is transcribed. The style's own sound is the soundtrack. The result
is `outputs/<name>/<name>.mp4`, with your style and direction saved next to it as `<name>.txt`. The steps are in
`.claude/skills/voiceless-short/SKILL.md`.

## Styles
- Each art style is one folder: `/root/code/videos/styles/<style>/`. It holds the style's drawing code, its build
  and render tools and a README.
- Each style has a skill with the same name, `/root/code/videos/.claude/skills/<style>/SKILL.md`, which teaches
  Claude how to make a video in it. The video agent reads that file.
- Which style is used: the one the brief names. If the brief names none and only one style exists, that one. With
  several styles and none named, Claude picks the closest match to the brief's style text.
- To add a style, add its folder to `styles/` and its skill to `.claude/skills/` under the same name. Nothing in
  this folder needs to change.
- Frame size and shape come from the style. `sci-fi-sitcom-thinline` renders 1920x1080 (landscape).

## Voices
Voices live in `/root/code/breeze_tts/voices/`. Claude picks the one that fits the story unless the brief or you
name one. The file formats (`<name>.txt` descriptions, `<name>.breeze` clones) are described in
`/root/code/breeze_tts/README.md`.

## What the machine needs
- Breeze TTS set up in `/root/code/breeze_tts`, and the `transcribe` alias.
- For rendering: Node 22 or newer, ffmpeg, and Chrome or Chromium (on `PATH`, or `CHROME=/path/to/chrome`).
- The GPU has 8 GB, so the TTS server is stopped before rendering. `mod.py` does this by itself.
