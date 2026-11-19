# voice/plan.json: the input to tools/voice_build.py

The voice-producer agent writes this file from the brief. voice_build.py is deterministic: the same plan gives the
same audio, because the seeds are fixed.

```json
{
  "slug": "bridge-song",
  "voice": {"mode": "file", "file": "deep_warm_storyteller"},
  "voice_alt_design_example": {
    "mode": "design",
    "name": "short_bridge-song",
    "instruction": "A warm, clear male narrator in his forties, lightly amused, crisp diction.",
    "cfg": 3, "seed": 7,
    "audition_text": "optional: a neutral 1-2 sentence line used to create the voice"
  },
  "defaults": {
    "instruction": "Brisk, punchy, fast-paced short-video narration with energy; quick delivery, no dragging.",
    "cfg": 1.5, "seed": 42, "pause_after": 0.35, "tempo": 1.0
  },
  "lead_in": 0.25,
  "tail": 1.0,
  "segments": [
    {"id": 1, "text": "Seventy-four soldiers knocked down an iron bridge just by walking across it.", "pause_after": 1.2},
    {"id": 2, "text": "(sigh) Luckily, the river was only two feet deep.", "cfg": 2.5, "pause_after": 0.6,
     "instruction": "Deadpan, dry, understated."},
    {"id": 3, "text": "Unluckily, the river was only two feet deep.", "tempo": 0.95}
  ]
}
```

## Voice modes
| mode | fields | what happens |
|---|---|---|
| `file` | `file`: a name (`deep_warm_storyteller`) or a path to a `.txt` | Reads the user's hand-picked description from `/root/code/breeze_tts/voices/<name>.txt`. |
| `design` | `instruction`, `cfg` (3), `seed`, `audition_text`? | Takes the description written straight into the plan. |
| `clone` | `ref_audio` (wav path), `ref_text` (exact transcript) | Uses that recording as the voice directly. |

A description becomes a voice like this:
1. It is auditioned once: one sentence is spoken from it.
2. That clip is cached in `/root/code/orchestration/.voice_cache/<name>-<hash>.wav`.
3. Every segment clones the clip, so the narrator is identical across segments and across videos that use the same
   `.txt`.

The hash covers the description, seed, cfg and sample sentence. Editing any of them gives a new voice, and every
segment regenerates automatically. `--new-voice` re-auditions even when the clip is cached. The clip is also copied
into the project as `voice/voice_ref.wav`.

Nothing is written to `/root/code/breeze_tts/voices/`. That folder holds only the user's `.txt` files, and the
server ignores anything there that is not `.breeze`. `.voice_cache/` can be deleted at any time; a deleted clip is
simply auditioned again. The same seed gives the same clip, and so the same voice.

### Voice `.txt` format
```
# comments start with #
A deep, warm male storyteller, slow and calm.
seed: 42        (optional, default 42. Same description + another seed = another person.)
cfg: 3          (optional, default 3. How strictly the description is followed.)
sample: The harbour lights came on one by one.   (optional: the audition sentence)
```
Every line that is not a setting is part of the description.

## Segment fields
| field | default | notes |
|---|---|---|
| `id` | required | unique int; segment takes are cached as `voice/segments/seg_<id>.wav` |
| `text` | required | exact script words; numbers spelled out; vocal-event tags allowed |
| `pause_after` | `defaults.pause_after` | silence in seconds after this segment. The last segment gets `tail` instead. |
| `instruction` | `defaults.instruction` | delivery direction for this line (voice identity comes from the voice clip) |
| `cfg` | `defaults.cfg` | 1.0-1.5 for plain reading, 2.5-3 so vocal events fire. Above 3 the voice turns harsh. |
| `seed` | `defaults.seed` | change it to get a different take |
| `tempo` | `defaults.tempo` | pitch-preserving speed change 0.5-2.0, applied at assembly time with no regeneration. Keep it within 0.88-1.15. |

The cache is keyed on (text, voice id, instruction, cfg, seed). Editing any of those for one segment regenerates
only that segment. `tempo`, `pause_after`, `lead_in` and `tail` only re-assemble the track, and the server is not
started. `--only 3,7` forces a redo.

## Pause vocabulary → seconds
| Brief says | `pause_after` |
|---|---|
| (nothing, normal sentence end) | 0.35 |
| mid-sentence split at a comma | 0.15 |
| "beat", "short pause", `[beat]`, paragraph break | 0.6 |
| "pause", `[pause]` | 0.9 |
| "long pause", "dramatic pause", `[long pause]` | 1.4 |
| "let it land", before a punchline or deadpan double | 0.6-0.8 |
| `[pause N]` | N |

A pause *inside* a sentence ("Then... a single bolt snaps") means splitting the sentence into two segments at
that point, with the pause on the first one.

## Pace (measured on this model, the Q8_0 GGUF on an RTX 3060 Ti)
| default instruction | words/s |
|---|---|
| "Steady, warm storytelling pace." | ~2.2 (too slow for shorts) |
| "Brisk, punchy, fast-paced short-video narration with energy; quick delivery, no dragging." | ~3.6 |

Short lines (under ~8 words) with the brisk instruction can rush to 5+ w/s. Fix that with `tempo: 0.88` or a calmer
per-line instruction.

## Outputs (voice/)
- `voice.wav`: 48 kHz mono s16. This is what the shorts `mix.py` reads; it asserts 48 kHz.
- `voice_24k.wav`: the native model rate, kept for reference.
- `segments.json`:
  - `{duration, voice, segments: [{id, text, start, end, wps}]}`
  - `start`/`end` are seconds on the final track.
  - the pause after a segment is the gap from its `end` to the next `start`.
- `words.json` (from align_words.py): `[{"w": "Seventy-four", "s": 0.15, "e": 1.15}, ...]`, one entry per
  script word, with the script's spelling.
- `align_report.json`: per segment `match`, `missed` (script words not heard) and `heard`, plus `bad_segments`.
  By default any missed word flags the segment.
- `voice_ref.wav`: the voice clip every segment cloned.
- `breeze_server.log`: log of the server, when voice_build.py started it.
