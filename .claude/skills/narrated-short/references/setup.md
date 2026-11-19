# One-time setup (run by Claude when `tools/check_env.sh` reports MISSING)

Do the steps that are missing, in order. Afterwards, `bash tools/check_env.sh` must print **ALL OK**. The only
exception is the Kenney check: those models are optional, and missing ones just don't appear.

Machine facts (checked 2026-11-17): Ubuntu, Python 3.14, Node 22, ffmpeg present, RTX 3060 Ti 8 GB, 15 GB RAM,
6 cores, ~200 GB free on `/`. The shorts repo was written on Windows, so it needs the Linux deltas below.

## 1. Python venv: already done 2026-11-17
```bash
cd /root/code/orchestration
python3 -m venv .venv
.venv/bin/pip install faster-whisper numpy scipy pillow requests
```
- The first `align_words.py` run downloads whisper `small.en` (~480 MB) into `~/.cache/huggingface`.
- faster-whisper's own audio decoder (PyAV) breaks with recent `av` versions. `align_words.py` already decodes with
  ffmpeg itself, so keep it that way.

## 2. Remotion project at `/root/code/videos/remotion-shorts`: done 2026-11-17
The user brought this project over from their Mac:
- versions: remotion 4.0.290, React 18.3, @react-three/fiber 8, @react-three/postprocessing 2, three 0.170
- the Mac copy also included an earlier finished short, "aita": `voice/aita/`, `audio/aita_*.py`, `out/aita/`
- its `src/` did NOT come across

These steps were then done on Linux:
- `rm -rf node_modules && npm ci`. The copied node_modules held darwin-arm64 binaries (compositor, esbuild). **If
  node_modules is ever copied from the Mac again, redo this.**
- `npx remotion browser ensure`: chrome-headless-shell for linux64.
- Scaffolded files:
  - `src/index.ts`
  - `src/Root.tsx` (one `<Composition>` per short; BridgeShort is the template and smoke test)
  - `tsconfig.json` (non-strict, bundler resolution)
  - `remotion.config.ts` (GL = `$REMOTION_GL` or `angle-egl`)
  - `src/videos/bridge/` (copied from the template)
  - `scripts/` (copied from the template, Linux-patched)
- `npx tsc --noEmit` passes.
- `<project>/.venv` is a **Mac** venv and does not work here. Ignore it, and use `/root/code/orchestration/.venv`.

To rebuild from scratch, `npm ci` with the existing package.json/package-lock.json and recreate the files above.
Keep every `@remotion/*` package on one exact version (`npx remotion versions`).

## 3. Linux port of the shorts skill (apply wherever the shorts SKILL.md or its scripts assume Windows)
Copy the helper scripts first: `cp /root/code/videos/claude-animated-shorts/templates/scripts/* scripts/`

| Shorts skill says | On this machine |
|---|---|
| `browserExecutable = "C:/Program Files/Google/Chrome/..."` in `bridge-stills.mjs` | Delete that line and every `browserExecutable` argument. Remotion uses its own chrome-headless-shell. |
| `chromiumOptions = { gl: "angle" }` | `{ gl: process.env.REMOTION_GL \|\| "angle-egl" }` |
| `selectComposition({ id: "BridgeShort" })` | `id: process.env.COMP \|\| "BridgeShort"`, so each short renders its own composition |
| `<big drive>/remotion-tmp`, `TEMP`/`TMP`, `D:` | `export REMOTION_TMP=/root/remotion-tmp TMPDIR=/root/remotion-tmp TEMP=/root/remotion-tmp TMP=/root/remotion-tmp` before any bundle or render |
| `~/bridge-short/` | `/root/code/orchestration/projects/<slug>/` |
| `python scripts/...` | `/root/code/orchestration/.venv/bin/python scripts/...` |
| `SendUserFile` | not available: give the absolute path of the final mp4 in the final message |
| "Sub-agents can't write report files" | They can here, but keep the ≤150-word final message pattern anyway, since it is cheaper |
| `npx remotion render ... --scale 1.3333` | Use the scale from the brief's `Resolution:` (default `1` = 1080x1920), plus `--gl=$REMOTION_GL` if the config file is not picked up |

**GL backend check: done 2026-11-17.** `angle-egl` uses the GPU and takes ~4 s per 0.5-scale still after a
~10 s bundle, and the bridge frames 400 and 1500 render correctly. If a driver or Chrome update ever breaks it, redo
this check:
1. Render the smoke still:
   `COMP=BridgeShort node scripts/bridge-stills.mjs out/smoke/x_ 400 0.5`
2. Open `out/smoke/x_400.jpg` with Read and check that it shows the bridge town, not a black or blank frame.
3. A still should take 3-8 s after the ~40 s bundle.
   - If it takes 60 s or more, WebGL is falling back to software (SwiftShader).
   - Try `REMOTION_GL=vulkan`, then `egl`, then `angle`.
   - Keep the first one that is fast and correct, and write it into `remotion.config.ts`.
   - `swangle` (CPU) always works but makes a final render take hours. Use it only as a last resort, and say so in the
     report.

## 4. Kenney models (optional, CC0)
`world.ts` loads `public/bridge/kenney/<name>.glb` for these names: tree_oak, tree_default, tree_detailed, tree_fat,
tree_cone, tree-high-round, tree, plant_bushLarge, plant_bushDetailed, plant_bush, hedge, flower_redA,
flower_yellowA, flower_purpleA, grass_large, grass, lily_large, lantern, cart, stall-red, stall-green, stall-bench,
banner-red, banner-green, windmill, canoe.

Most come from Kenney's **Nature Kit**. The hyphenated town props (lantern, cart, stall-*, banner-*, windmill,
tree-high-round) come from the **Fantasy Town Kit**. Both are CC0 at kenney.nl/assets.

Steps:
1. Download the two zips into a new empty folder under `/root/remotion-tmp/kenney-dl/`.
2. Copy the needed `.glb` files from their `Models/GLTF format/` folders into `public/bridge/kenney/`.
3. Keep the path `bridge/kenney/` for every short, because the template's `staticFile` path points there.

The loader catches missing files (`.catch(() => null)`), so a short still renders without them, just with fewer
props.

## 5. Breeze TTS: already set up
- Binary: `/root/code/breeze_tts/Breeze-TTS-2.cpp/build/breeze-server`, built with Vulkan.
- Model: `/root/code/breeze_tts/models/breeze-tts-2-q8_0.gguf`.
- `voice_build.py` starts the server itself on 127.0.0.1:8090 and stops it afterwards.
- `/root/code/breeze_tts/voices/` holds only the user's hand-picked `<name>.txt` voice descriptions (the server
  ignores non-`.breeze` files). The pipeline never writes `.breeze` files. It caches auditioned clips in
  `/root/code/orchestration/.voice_cache/` and uploads them with each request; the server caches the encode in
  memory.
- The user's own `mod.py`/`run.sh` also use port 8090. If one of them is already running, voice_build.py reuses it
  and leaves it running.
- `models/ref_voice.wav` exists, but its transcript is unknown. Use it only if a brief asks for it and gives the
  transcript.
