# EKANA — Design Spec (SIH26158: Single-Pass Drone Video → Accurate 3D)

## 0. Decisions

- The deployed site serves **preconverted 3D only**. There is no online conversion, no upload path, no backend server.
- All 3D on the site comes from **one campus drone clip** (plain MP4, no telemetry).
- The site has two views of the **same scene**: `Campus` (pure reconstruction, the wow) and
  `Metric Fit` (same geometry passed through the GPS-alignment stage against a synthetic prior, the rubric story).
- Reproducibility is via a containerized offline pipeline, not via the site.

## 1. System overview

Two halves, decoupled. The offline pipeline runs once and publishes static assets.
The website serves those assets and renders them client-side.

- **Offline pipeline:** video clip → frames → structure-from-motion → scale/alignment →
  Gaussian splats + mesh → decimated web assets + metric report.
- **Website:** static page. Video element on the left, WebGL 3D viewer on the right.
  All 3D rendering happens in the visitor's browser. The host does zero 3D compute.

## 2. Website

### 2.1 Experience

- One page, one scroll. Top bar with project name and container pull command.
- Hero line stating single-pass video to metric 3D, with capability chips.
- Main block, two scene tabs:
  - **Campus (default):** campus clip playing beside an orbitable 3D view. Auto-rotates until first drag.
  - **Metric Fit:** same 3D view plus a top-down track plot and the pipeline numbers,
    labeled as fitted against a synthetic prior.
- Inside each scene, sub-tabs for **Splat** (photorealistic) and **Mesh** (measurable geometry),
  with reset-view, wireframe, and fullscreen controls.
- Metrics strip with reconstruction statistics, plus downloads for the model files and the metric report.
- How-it-works strip: extract, align, fuse — plus the one-line container run command.

### 2.2 Viewer technology

- Splat view uses `@mkkellogg/gaussian-splats-3d` (renders `.ply` files carrying spherical-harmonic
  coefficients). A generic point-cloud loader such as `three.js` PLYLoader cannot render splats —
  the dedicated splat renderer is required.
- Mesh view uses the `<model-viewer>` element for the Draco-compressed `.glb` (orbit/zoom/pan built in).
- Interaction: drag to orbit, scroll to zoom, right-drag to pan, pinch on touch.
- Loading: poster frame first, video plays immediately, splat streams with a progress indicator,
  mesh lazy-loads when its tab opens. Only one 3D representation is mounted at a time (mobile RAM).
- Mobile: phones default to the mesh tab with auto-rotate off and capped pixel ratio,
  holding interactive frame rates on integrated GPUs. `?mesh=1` forces the mesh tab anywhere.

### 2.3 Performance budget

- Web assets are decimated to roughly ≤100k splat points / ≤200k mesh triangles,
  keeping first load to a few seconds on shared WiFi and rendering at 30–60 fps locally thereafter.
- Orbit and zoom are purely local once loaded — zero network latency per frame.

## 3. Reconstruction pipeline

Upstream base: `ch1bo/drone-reconstruction` (ffmpeg → COLMAP sequential SfM → Sim3-ENU GPS alignment
→ Nerfstudio `splatfacto` → export). Telemetry parsing reuses its `srt_to_reference_poses.py` verbatim
(handles modern `[latitude:..]` and legacy `GPS(..)` SRT by interpolating 1 Hz entries to frame times).

### 3.1 Input

A single 60–90s straight-line drone clip, 1080p or higher, with buildings visible and no fast panning.
No telemetry is assumed. A short straight segment is selected from the longer recording
(`make clip`: 1 fps thumbnails → eyeball the straight leg → `-ss/-t` cut with stream copy);
a 720p preview and poster frame are derived for the website.

### 3.2 Stages

1. **Frame extraction.** `ns-process-data video` (or ffmpeg directly): 2 fps, capped at 1280p,
   ~150 frames max. Sequential video frames give the overlap SfM needs; the caps bound compute
   (fits 8 GB VRAM / Kaggle T4 free).
2. **Sparse reconstruction.** COLMAP with a single shared `OPENCV` camera model: GPU feature extraction,
   `sequential_matcher` (each frame matched to neighbors only — O(n), correct for video),
   `mapper` to camera poses plus a sparse point cloud. Triage: reprojection error must stay ≤1 px;
   multiple `sparse/N` sub-models mean a disconnected run (raise fps or trim panning seconds);
   corkscrew trajectories are expected scale drift on straight-line paths (absorbed by stage 3).
3. **Scale and alignment.** SfM output is up-to-scale and unplaced. `colmap model_aligner` fits one
   similarity transform (rotation + translation + scale) between estimated camera positions and
   per-frame GPS positions in a local East-North-Up meter frame (`--ref_is_gps 1 --alignment_type enu`,
   `--alignment_max_error 15`). Scale comes from barometric relative altitude where available,
   never from GPS mean-sea-level altitude (±10 m noise).
4. **Dense appearance and surface.** `ns-train splatfacto` (7k iterations laptop / 15k hero;
   OOM fallback: halve rays-per-batch, `--data_device cpu`) yields the photorealistic splat model;
   `ns-export gaussian-splat` + `ns-export poisson` yield splat and mesh. ENU-aligned models train with
   `--assume-colmap-world-coordinate-convention False` (already +Z-up, no Y/Z remap).
5. **Decimation and export.** Open3D voxel (2 cm) + triangle reduction to the §2.3 budget, Draco GLB,
   OBJ via Open3D, LAS via laspy, scale knob baked in — plus `track.png` and `metrics.json`,
   published to the Hugging Face Dataset repo.

### 3.3 Why GPS, and what the synthetic prior is

GPS contributes two things the pixels cannot: **scale** (true meters between fixes divided by
model units between the same frames) and **placement** (rotating and translating the scaled model
onto real coordinates). Without it the model is shape-correct but sizeless and floating.

Our clip carries no telemetry, and a telemetry subtitle is plain text that is trivially synthesizable —
per-second latitude, longitude, and altitude entries in the drone's native subtitle format.
For the Metric Fit view, `make_prior_srt.py` generates such a prior for our own clip: anchored on the
campus coordinates (lat 28.5265, lon 77.5746), headed along the flight direction read off the COLMAP
camera path, at the assumed cruise height (40 m) and speed (8 m/s), constant-velocity since the pass
is straight (meters→degrees via `dLat = dy/111320`, `dLon = dx/(111320·cos(lat0))`). It flows through
the identical alignment stage as real telemetry would, so the metric machinery is genuinely exercised;
only the input coordinates are assumed. Every number derived from it is labeled as fitted against
the prior, never as ground truth. A future flight with real sidecar telemetry drops into stage 3
with no code change (`ffmpeg -map 0:s:0` extract → same aligner).

### 3.4 Scale calibration

With no barometer data, absolute scale rests on one knob: assumed flight altitude (default 40 m).
It is calibrated once by comparing a known ground distance (measured on satellite imagery)
against the same distance in mesh units, and the correction is baked into the export.
Relative geometry (the <1 m claim) is unaffected by this knob; absolute placement inherits GPS-grade tolerance.

## 4. Data contracts

Reconstruction emits the eight web assets plus a metric report with exactly this schema —
the website renders its metrics strip and downloads exclusively from it:

```json
{"scene":"snu","gps_mode":"none | synthetic_prior | real_srt","n_frames":150,"fps":2,
 "resolution":1280,"reproj_px":0.62,"scale_factor":1.0,"rmse_vs_prior_m":0.4,
 "proc_min":32,"assets":{"video":"…720p.mp4","splat":"…snu.ply","mesh":"…snu.glb"}}
```

`rmse_vs_prior_m` is a pipeline-exercise number (fit residual against the prior), never ground truth —
UI copy, code strings, and metric reports all label it as such.

## 5. Deployment

- **Website** → Vercel (static). Six `VITE_*` asset URLs as env config. No functions, no rewrites.
- **Assets** → public Hugging Face Dataset repo as CDN, published from the notebook's final cell.
- **Pipeline** → GPU Docker image on GHCR (CUDA 12.1 runtime + COLMAP + ffmpeg + pipeline deps,
  `convert.py` as ENTRYPOINT). `--cpu` falls back to mesh-only (`FeatureExtraction.use_gpu 0`,
  skip `splatfacto` → fused-ply mesh). The 5-cell Kaggle notebook is retained as the human-readable record.

## 6. Non-goals

No live conversion, accounts, queues, databases, or persistent storage. No masking, depth-fusion,
or live splat training. No full-length-video hero (the demo is a representative slice; the pipeline
scales linearly in frames). GeoTIFF/FBX explicitly out of demo scope: dense `fused.ply` plus a
one-line translate covers them if a judge asks.

## 7. Demo narrative

Campus view first: play the clip, drag the model, switch splat to mesh, zoom to a known building.
Metric view second: same scene with the track plot and pipeline numbers —
"metric path end-to-end; prior today, your telemetry tomorrow."
Close with the container command: any new 60–90s pass reproduces the same outputs.
