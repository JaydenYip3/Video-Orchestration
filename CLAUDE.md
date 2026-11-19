# Orchestration: brief → narrated animated short

Glues two systems into one automated video pipeline, with no approval gates:
- **Breeze TTS** (`/root/code/breeze_tts`): narration.
- **claude-animated-shorts** (`/root/code/videos/claude-animated-shorts`): Remotion + three.js cartoon shorts.

The user writes `briefs/<label>.md` (script, delivery notes, style). **To make a video, use the `narrated-short` skill**
(`.claude/skills/narrated-short/SKILL.md`) and follow it as written:
1. `breeze_tts/mod.py` → `outputs/<label>/<label>.wav`
2. `transcribe` → `<label>.srt`
3. A new agent follows the claude-animated-shorts skill and saves `<label>.mp4` in the same folder.

Do not use the `voice-producer` agent, `tools/voice_build.py` or `projects/` (old system, kept only for reference).

## Layout
| Path | Purpose |
|---|---|
| `briefs/` | user briefs; `EXAMPLE.md` shows the format |
| `outputs/<label>/` | one video: `<label>.txt/.wav/.srt/.mp4`; working files in `build/` |
| `/root/code/videos/remotion-shorts` | Remotion project; one `src/videos/<name>/` folder + composition per short |
| `.venv/` | Python for the shorts' audio/QA scripts |

## Hard constraints
- 8 GB GPU shared by TTS and rendering: breeze-server must be stopped before any Remotion render
  (`mod.py` stops it unless run with `--keep`).
- Never change the words of the user's script.
- Never overwrite another short's code or composition: check `src/videos/` names first.
- The reference-channel material in the shorts repo is private study material. Never publish it or name the channel.
