# AGENTS.md

EKANA (SIH26158: Single-Pass Drone Video → Accurate 3D)

## Project Overview

EKANA turns a single straight-line drone video clip into a georeferenced, metrically-scaled textured 3D
model (Gaussian splats + mesh) viewable in a browser. 

Built for the SIH 2026 NTRO problem statement 26158


demo at Shiv Nadar University: judges open one deployed page showing campus footage beside an orbitable 3D reconstruction, plus a containerized pipeline that reproduces the result from any new clip.

## Development Philosophy

- Static site, offline pipeline — there is no backend server, no database, no auth. Do not add one.
- Reuse over invention: the reconstruction flow follows `ch1bo/drone-reconstruction`
  (ffmpeg → COLMAP sequential SfM → Sim3-ENU GPS alignment → Nerfstudio `splatfacto` → export).
  Telemetry parsing reuses its `srt_to_reference_poses.py` verbatim. Never reimplement these.
- Explicit over clever — readable code beats smart code.
- If it isn't runnable via `make`, it isn't done.

## Tech Stack

- Website (`web/`): Vite 5 + TypeScript (strict) + `three@0.160.0` +
  `@mkkellogg/gaussian-splats-3d@0.4.7` (splat tab) + `@google/model-viewer@3.3.0` (mesh tab).
  No framework, no map library (GPS track ships as a static plot image).
- Pipeline (`scripts/`, `notebook/`): Python 3.10, COLMAP 3.8 (SfM), Nerfstudio 1.1.3
  (`ns-process-data video`, `ns-train splatfacto`, `ns-export`), Open3D 0.18.0 (decimation),
  laspy 2.5 (LAS export), `huggingface_hub` 0.23 (asset publishing), torch 2.3 (Kaggle preinstalled).
- Package managers: `uv` (Python), `npm` (web). Never `pip` directly, never committed binaries.
- Build/Task Runner: **Make** — root `Makefile` is the single entry point (see Key Commands).
- Compute: one-time hero run on Kaggle T4 (free); reproducibility via GPU Docker image on GHCR (free).
- Hosting: Vercel free (static site) + Hugging Face Dataset repo as asset CDN (free).

## Key Commands

All commands run via `make <target>` from the project root. Tool invocations (`vite`, `colmap`,
`ns-train`, `docker`, …) live inside the Makefile, never as tribal knowledge.

```bash
make setup                       # installs web + pipeline deps (npm ci, uv sync), idempotent
make dev                         # serves web/ locally against published asset URLs
make test                        # runs pipeline unit tests (scripts/tests) + web typecheck
make style                       # formats + lints all code (prettier/eslint, black/ruff)
make build                       # production build of web/
make clean                       # removes build artifacts, caches, exports/
make clip                        # cuts + previews the 60–90s hero clip from the source MP4
make docker-build                # builds the pipeline image
make docker-run                  # runs the pipeline on input/ → output/ (GPU; --cpu for mesh-only)
```

## Directory Structure

```
ekana/
├── web/                         # static site (Vercel)
│   ├── index.html               # layout: topbar, scene tabs, viewer panes, metrics strip
│   ├── src/main.ts              # tab switching, ?mesh=1, metrics.json render
│   ├── src/viewer.ts            # initSplat / initMesh, one mounted at a time
│   ├── vite.config.ts
│   ├── tsconfig.json            # strict: no implicit any, strict null checks
│   ├── package.json
│   └── .env.example             # six VITE_* asset URLs, values never committed
├── scripts/                     # offline pipeline (frozen notebook logic)
│   ├── srt_to_reference_poses.py# stolen verbatim from ch1bo/drone-reconstruction
│   ├── make_prior_srt.py        # synthetic straight-line DJI-style telemetry
│   ├── plot_track.py            # ENU prior-vs-aligned plot + fit numbers → metrics.json
│   ├── decimate.py              # voxel/triangle reduction, scale bake, ply/glb/obj/las export
│   ├── convert.py               # argparse CLI wrapping the full pipeline (Docker ENTRYPOINT)
│   └── tests/                   # unit tests mirroring the scripts above
├── notebook/
│   └── ekana_hero.ipynb         # the 5-cell Kaggle run (clip → recon → prior → publish)
├── docs/
│   ├── problem_statement.md     # verbatim SIH26158 statement
│   ├── research.md              # objective tool/dataset/source research, no recommendations
│   └── spec.md                  # design spec — takes precedence on all build decisions
├── data/hero/
│   └── README.md                # published HF Dataset URLs only, never binaries
├── Dockerfile                   # CUDA 12.1 runtime + COLMAP + ffmpeg + pipeline deps
├── Makefile                     # canonical control surface (see Key Commands)
└── .env.example                 # HF dataset names, no tokens
```

## Conventions

### Makefile (required)

- Required targets: `setup`, `dev`, `test`, `style`, `build`, `clean` (+ `clip`, `docker-build`, `docker-run`).
  Never remove from the required set; each gets a `## short description` comment.
- `make setup` is idempotent and safe to re-run.

### Python (scripts/)

- **Package manager: `uv`** — `uv add`, `uv run`, `uv sync`. Never `pip` directly.
- Formatter: `black`, linter: `ruff` (includes import sorting).
- snake_case for files, variables, functions. Stolen files keep upstream logic untouched;
  project-specific behavior goes in new files (`make_prior_srt.py`, `plot_track.py`, `decimate.py`).
- `convert.py` is pure argparse → function calls; no HTTP, no viewer knowledge.
- Env/config (HF dataset names, altitude knob) via CLI flags and `.env`, never hardcoded secrets.
- Logging via the stdlib `logging` module through one shared `get_logger` helper — never bare `print`
  in pipeline code (notebook cells may print URLs as their output contract).

### TypeScript (web/)

- Strict mode, typed component props, camelCase vars/functions, PascalCase components/types,
  kebab-case file names.
- Only one 3D representation mounted at a time (splat XOR mesh) — mobile RAM constraint.
- All asset URLs come from `import.meta.env` (`VITE_*`); no hardcoded CDN links in source.
- Formatter: Prettier, linter: ESLint.

### General

- Conventional commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`).
- Never commit binaries, `.env` values, or HF tokens. `.env.example` carries keys only.
- Never modify files in `docs/` unless explicitly asked — `docs/spec.md` takes precedence; flag conflicts.
- Always run `make test` after changes; fix failures before moving on. Always run `make style` before done.
- Any new setup/run/test/style/build step becomes a Makefile target, not prose.

## Deployment Philosophy

Free tiers only, two static surfaces — no servers anywhere:

- **Website** → Vercel (static). Six `VITE_*` asset URLs as env config. No functions, no rewrites.
- **Assets** (`*_720p.mp4`, `poster.jpg`, `snu.ply`, `snu.glb`, `snu.obj`, `snu.las`, `track.png`,
  `metrics.json`) → public Hugging Face Dataset repo as CDN, published from the notebook.
  Large binaries never enter git.
- **Pipeline** → Docker image on GHCR (CUDA runtime). `--cpu` flag falls back to mesh-only
  for machines without a GPU.

## Agent Guidelines

- Always check `docs/spec.md` before starting any task — it takes precedence on all build decisions.
- Never swap the pinned stack (COLMAP / Nerfstudio / splat-renderer versions) without updating
  both `docs/spec.md` and this file.
- Reuse upstream logic verbatim where the spec says stolen; write tests only for project-owned code
  (`make_prior_srt.py`, `plot_track.py`, `decimate.py`, `convert.py` arg wiring, `viewer.ts` tab logic).
- Keep the two scene tabs showing the same scene (Campus = geometry, Metric Fit = same geometry +
  prior fit). Never source a second footage clip without explicit user approval.
- Label every prior-derived number as fitted-against-prior, never as ground truth — in code strings,
  UI copy, and metric reports alike.
- If something feels out of scope, flag it rather than silently doing it.

## Project-Specific Notes

- Source footage: `shri-friend.mp4` (repo root, gitignored if over limits) — non-DJI SigmaStar SoC,
  re-encoded, no telemetry streams. Cut the 60–90s straight segment via `make clip`; record chosen
  timestamps in the notebook. Never ship the 4K original; the site plays only the 720p preview.
- Campus anchor for the synthetic prior: lat 28.5265, lon 77.5746 (Shiv Nadar University, Greater Noida).
  Heading is read off the COLMAP camera path per run; default altitude knob 40 m, cruise 8 m/s.
- Stolen file — do not modify logic: `scripts/srt_to_reference_poses.py`
  (upstream: `ch1bo/drone-reconstruction`). Handles both modern `[latitude:..]` and legacy `GPS(..)` SRT.
- SIH formats shipped: PLY, GLB, OBJ, LAS + metrics. GeoTIFF/FBX explicitly out of demo scope
  (see spec §6 for the one-line answer if judges ask).
- Poster fallback: every page works with `?mesh=1` (forces mesh tab, bypasses splat renderer).
- Known gotchas: `three.js` PLYLoader cannot render splat files (SH coefficients) — splat tab must use
  the Gaussian-splats-3D viewer; COLMAP straight-line runs can corkscrew in scale (absorbed by the
  Sim3 fit, noted in the notebook); Vercel file cap means assets live on HF, never in `web/`.
