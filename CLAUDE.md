# Orchestration: brief → narrated animated short

Glues two systems into one automated video pipeline, with no approval gates:
- **Breeze TTS** (`/root/code/breeze_tts`): narration.
- **Video styles** (`/root/code/videos`): one self-contained art style per folder in `styles/<style>/`, and one
  skill per style in `.claude/skills/<style>/SKILL.md` that teaches how to make a video in it.

The user writes `briefs/<label>.md` (script, delivery notes, style). **To make a video, use the `narrated-short` skill**
(`.claude/skills/narrated-short/SKILL.md`) and follow it as written:
1. `breeze_tts/mod.py` → `outputs/<label>/<label>.wav`
2. `transcribe` → `<label>.srt`
3. Pick a style from `/root/code/videos/styles`. A new agent follows that style's skill and saves `<label>.mp4` in
   the same folder.

For a video with no narration (the user gives only a style and a direction, no script), use the `voiceless-short`
skill (`.claude/skills/voiceless-short/SKILL.md`) instead: no voice and no transcript, only step 3, driven by the
direction.

Do not use the `voice-producer` agent, `tools/voice_build.py` or `projects/` (old system, kept only for reference).
The old Remotion engine (`claude-animated-shorts`, `remotion-shorts`) has been removed from `/root/code/videos`.

## Layout
| Path | Purpose |
|---|---|
| `briefs/` | user briefs; `EXAMPLE.md` shows the format |
| `outputs/<label>/` | one video: `<label>.txt/.wav/.srt/.mp4` (voiceless: `.txt/.mp4` only); working files in `build/` |
| `/root/code/videos/styles/<style>/` | one art style: source, build and render tools, README; each video is its own folder inside it |
| `/root/code/videos/.claude/skills/<style>/SKILL.md` | how to make a video in that style |
| `.venv/` | Python of the old voice system; the current pipeline does not use it |

## Hard constraints
- 8 GB GPU: breeze-server must be stopped before any render (`mod.py` stops it unless run with `--keep`).
- Never change the words of the user's script.
- Never overwrite another video's code: check the names already in the style's folder first.
- A style's own source and tools are shared by every video in it. Change them only when the user asks for a change
  to the style.
- Reference material in the videos repo (such as `style_creator/references/`) is private study material. Never
  publish it or name its source. Characters in videos are original designs.
