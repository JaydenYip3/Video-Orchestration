# Brief format (what the user writes)

One Markdown file, `briefs/<slug>.md`. Only `## Script` is required. Everything else falls back to defaults.

```markdown
# The Bridge That Fell for a Song

## Script
Seventy-four soldiers knocked down an iron bridge just by walking across it. [pause 1.2]
It's 1831, and the brand new bridge over the river is the pride of the whole town.
...
(sigh) Luckily, the river was only two feet deep. [beat]
Unluckily, the river was only two feet deep.

## Voice
Warm, slightly amused male narrator in his forties, brisk YouTube-shorts pace.
Long dramatic pause after the first line. Deadpan on the two "Luckily / Unluckily" lines.
Slow down for the swing explanation.

## Style
England, 1831, a proud little mill town on a river. Bright sunny day, the explanation part at golden hour.
Comic and a bit absurd. Upbeat brass march. Lots of camera energy around the collapse.
```

## Sections

### `## Script` (required)
These are the exact words that get spoken. The pipeline never rewords them.

Inline markers are optional:

| Marker | Meaning |
|---|---|
| `[beat]` | short pause, 0.6 s |
| `[pause]` | 0.9 s |
| `[long pause]` | 1.4 s |
| `[pause 1.7]` | exactly 1.7 s |
| blank line between paragraphs | 0.6 s |
| `(sigh)`, `(laugh)`, `(gasp)`, `(clears throat)`, `(nervous chuckle)` | vocal events. Free-form tags in round brackets. They usually fire, but not always. |
| `{whispered}` at the start of a line | per-line delivery note, used as that line's TTS instruction |

Numbers and years can stay as digits. The voice agent spells them out ("1831" → "eighteen thirty-one").

If you want a different reading, spell it out yourself ("eighteen-thirty-one").

### `## Voice` (optional)
Write it in plain language. It sets three things:

- **Who** is speaking. Gender, age, warmth, accent, character, for example "gravelly old sea captain". The default
  is a warm, clear, lightly amused narrator.
- **Pace.** For example "brisk" or "slow and calm". The default is brisk shorts pace, ~3.6 words/s.
- **Where to pause, and how to deliver specific lines.** Refer to lines by their words: "pause after the first
  line", "whisper the line about the bolt".
- Optional: `Reference voice: /path/clip.wav — "exact transcript of the clip"` clones that voice instead of
  designing one.
- Optional: `Voice file: <name>` uses your hand-picked description in `/root/code/breeze_tts/voices/<name>.txt`.
  The same file always gives the same narrator, so use it to keep one narrator across a series of videos. Without
  any voice line, `voices/default.txt` is used if it exists.

### `## Style` (optional)
A few sentences about the look and feel. The engine is always the "2D cartoon characters in a painted 3D world" style
of claude-animated-shorts. The style text steers what goes inside it:

- place and era
- time of day and light
- palette and mood
- character types
- camera energy
- music feel
- sound world

See `style-mapping.md` for how each phrase becomes concrete settings. The default is the template look: sunny day, a
golden-hour section for any reflective part, and an upbeat original score.

### `## Notes` (optional)
Anything else. For example `Resolution: 720p | 1080p | 2K | 4K` (default 1080p, i.e. 1080x1920), "captions on" (off by default), "keep it under 60 s", "end on the bridge wide shot",
"no music, SFX only".

## What the user does NOT need to provide
- shot lists, camera moves or timings, which are derived from the voice's word timings
- sound effects or music cues, which are derived from the story and picture
- voice settings such as seeds, cfg or tempo
