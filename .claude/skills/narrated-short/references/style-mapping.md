# Style description → concrete knobs

The render engine is fixed: claude-animated-shorts, which draws 2D bean-head cartoon cards with ink outlines in a
painted three.js world, with lens blur, gliding cameras and synthesized audio. The user's `## Style` text decides
**what goes inside that engine**.

Write the translation into `projects/<slug>/STYLE.md` as a table: phrase from the brief → file → change.
`templates/bridge/` is already tuned (verifier scores 5-7/10 per area). Change it on purpose; don't re-tune it.

| Brief talks about | Where it lands | How |
|---|---|---|
| **Place / era** ("1830s mill town", "Roman harbour", "1960s NASA hangar") | `build.ts` (layout, buildings, props), `paint.ts` (albedos, painted textures), Kenney kit choice, `cast.ts` costumes | Research period-correct details first (flags, clothes, vehicles, signage) and write them in SCRIPT.md. Keep the template's muted albedo range. Never use lemon yellow or lavender lit faces. |
| **Indoor / non-town setting** | `build.ts`, `world.ts` (sky/fog/river toggles) | The template is built for outdoors. The earlier "aita" short had a `room` set, but its source is missing, so the room has to be rebuilt: boxes with painted textures, the sky shader and river off, subtle fog. Expect more verifier rounds. `$REMOTION/out/aita/*.jpg` shows how the room looked. |
| **Time of day / light** ("sunny", "sunset", "moody evening") | `lights.ts` `LIGHTS`/`LOOK`, the `light` per shot in `story.ts` `SHOTS` | Presets: `day`, `golden`. To add `dusk` or `night`, extend the `"day" \| "golden"` union in lights.ts and add a preset next to golden: low sun, violet-blue shade, emissive windows (bloom threshold 1.0). The shade must never go brown. |
| **Palette / mood** ("pastel", "gritty", "vivid") | `LOOK` sky colours, `grade.ts`, `paint.ts` albedos, `uSat` | Move in small steps: uSat 0.95-1.15, and sky top/horizon within ±15 % luma. "Darker / serious": golden or dusk light, saturation −10 %, slower camera. |
| **Tone** ("comic", "absurd", "dramatic", "eerie") | camera (`CAMS`, `KICKS`), expressions (`toon.ts` face library), music | Comic: flip-kicks on impacts, a white flash after the hook, deadpan holds with creeping push-ins. Dramatic: longer 30-40 f glides, fewer kicks, more DOF. Eerie: low sun, fog closer, sparse music. |
| **Camera energy** ("lots of energy", "calm") | `story.ts` `CAMS`, `XFADE`, `KICKS`, `dofAt` | Energetic: a cut every 2-3 s, 18-24 f glides, 4.6° rolls, kicks on every hit. Calm: a cut every 4-6 s, 30-40 f glides, crossfades at place changes. The house rule holds either way: every hold creeps (no static hold ≥ 4 f). |
| **Characters** ("old sailors", "kids", "scientists in lab coats") | `toon.ts` (Pen drawings, faces), `cast.ts` (roles, crowds, walk cycles) | Keep the proportions: bean heads h/w 0.94, 3.5 head-heights, plum ink. Change hats, clothes, props, hair, body width and count. Kids are 2.5-3 head-heights. |
| **Music feel** ("brass march", "lo-fi", "spooky strings", "no music") | `audio/music.py`, `audio/instruments.py` | The template is a Bb-major 120 bpm comic march with section changes keyed to story beats. Rewrite `build()` for the new beats. Instruments available: brass(tuba/…), glock, vibes, musicbox, triangle, fife, whistle, snare, snare_roll, cymbal, timpani, brush, pluck, pad. Pick tempo and key per mood. "No music": skip music.py and mix only voice + SFX. |
| **Sound world** ("busy harbour", "quiet lab") | `audio/sfx_gen.py`, `audio/sfx.py`, `audio/timeline.py` | Make an event table from the shot plan and the words.json times. Every visible impact, step, splash or click gets a sound on its frame. Add an ambience bed per location. |
| **Captions** ("captions on") | new caption layer in the composition | Off by default (house rule). If asked: word-by-word from `words.json`, bottom third, bold sans with an ink outline, and no description of what is already on screen. |
| **Length** ("under 60 s") | voice stage | Tighten `pause_after` values and raise the default tempo up to 1.1. Never cut words. If the length still can't be met, report it. |
| Something the engine can't do ("photoreal", "anime 2D only", "stop-motion") | none | Do the closest thing inside this engine, then name the gap in DECISIONS.md and in REPORT.md. Do not switch engines. |

## Defaults when `## Style` is missing
Use the template look:
- a sunny `day` light, with any reflective or explanatory part at `golden`
- comic tone
- medium camera energy
- an upbeat original score written for the story
- the SFX palette derived from the story
