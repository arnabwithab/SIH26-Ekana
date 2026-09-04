# Research - SIH 2026 Single-Pass Drone Video to Accurate 3D Model Generation System (SIH26158)

## Pinned Problem Statement

**Problem Statement 17 / SIH26158 - Single-Pass Drone Video to Accurate 3D Model Generation System**

**Organisation:** National Technical Research Organisation (NTRO) | **Category:** Software | **Theme:** Drone / Robotics

**Links:**
- SIH 2026 Problem Statements Portal: https://www.sih.gov.in/sih2026PS (PS ID: 26158)
- NTRO Additional Information: https://drive.google.com/file/d/119hjXkLhMW_AhQ4cyYz-XJgcVz4BA-hD/view?usp=drive_link

> **One-line summary:** Generate a georeferenced and metrically accurate textured 3D mesh/point cloud of terrain, buildings (facade + rooftop), roads, vegetation from a single straight-line drone video + GPS, with <1m accuracy and <15 min for 10 min video, without GCPs, viewable in a browser.

Full verbatim problem statement is in `problem_statement.md`.

---

## 1. SIH 2026 Context

### 1.1 Overall Scale (fetched 2026-09-04 from https://www.sih.gov.in/sih2026PS)
- Total problem statements: **229** (parsed from SIH portal HTML dump, 18637 lines)
- Software: **175**, Hardware: **54**
- Software Themes distribution (core 158 excluding Student Innovation):
  - Smart Automation: 43
  - Blockchain & Cybersecurity: 28
  - Disaster Management: 21
  - MedTech / BioTech / HealthTech: 12
  - Agriculture, FoodTech & Rural Development: 12
  - Miscellaneous: 11
  - Space Technology: 7
  - Smart Education: 7
  - Transportation & Logistics: 5
  - Clean & Green Technology: 4
  - Smart Vehicles: 3
  - Robotics and Drones: 3
- Organisations with most Software PS: Ministry of Earth Sciences (27), NTRO (22), ISRO (10), Ministry of Home Affairs (10), Ministry of Rural Development (9)
- Student Innovation Software PS: 17 (IDs 26193-26209) - generic open-ended prompts, excluded from core counts.

### 1.2 Problem Statement 26158 Position
- Category: Software (not Hardware)
- Theme: Robotics and Drones (only 3 Software PS under this theme among core 158)
- One of 22 NTRO software PS; 26158 is the only NTRO drone-video 3D reconstruction PS in the set.

---

## 2. Problem Statement Details - Objective Facts

### 2.1 Full Text
See `problem_statement.md` for verbatim text (Background, Description, Key Challenges i-viii, Input Data mandatory/optional, Desired Output table, Evaluation Criteria table, Potential Applications i-viii, Organisation/Category/Theme, Dataset Link status: "Will be provided real time. Additional Information regarding PS https://drive.google.com/file/d/119hjXkLhMW_AhQ4cyYz-XJgcVz4BA-hD/view?usp=drive_link").

### 2.2 Input Data Specification
- **Mandatory:** Drone video (1080p/4K), GPS coordinates, flight metadata
- **Optional:** IMU data, barometric altitude, camera intrinsic parameters, RTK/PPK corrections

### 2.3 Desired Output Specification
| Parameter | Target |
|---|---|
| Reconstruction Type | 3D Mesh / Point Cloud |
| Processing Time | < 15 minutes for 10-minute video |
| Spatial Accuracy | < 1m |
| Coverage | Entire visible scene |
| Output Formats | OBJ, PLY, LAS, GeoTIFF, .glb/.gltf, .fbx |
| Visualization | Web-based or Desktop Viewer |

### 2.4 Evaluation Criteria
| Criteria | Weightage |
|---|---|
| Reconstruction Accuracy | 30% |
| Model Completeness | 20% |
| Processing Speed | 20% |
| Innovation | 15% |
| Scalability | 10% |
| User Interface | 5% |

### 2.5 Key Challenges (verbatim)
(i) Limited viewing angles due to single flight path.
(ii) Motion blur and video compression artifacts.
(iii) Variable illumination and shadows.
(iv) Dynamic objects (vehicles, humans, animals).
(v) GPS inaccuracies and sensor noise.
(vi) Real-time or near-real-time processing requirements.
(vii) Reconstruction of occluded surfaces.
(viii) Maintaining metric accuracy without extensive Ground Control Points (GCPs).

### 2.6 Deliverable Formats
OBJ, PLY, LAS, GeoTIFF, .glb/.gltf, .fbx + Web-based or Desktop Viewer. Models must be suitable for visualization, measurement, and analysis.

### 2.7 Potential Applications Listed
Border and strategic area mapping, disaster damage assessment, urban planning and smart cities, infrastructure inspection, construction progress monitoring, archaeological documentation, digital twin generation, military reconnaissance and mission planning.

---

## 3. Technical Domain Research - Relevant Open-Source Tools

### 3.1 COLMAP and GLOMAP (Structure-from-Motion for pose)

**COLMAP:**
- Source: https://colmap.github.io/ (general-purpose SfM and MVS pipeline, free and open source, graphical and command-line interface)
- Supports ordered and unordered image collections.
- Documentation: https://colmap.github.io/tutorial.html
- Video input guidance: "If you use a video as input, consider down-sampling the frame rate."
- Matching modes documented: Exhaustive, Sequential Matching (useful if images are acquired in sequential order, e.g., by video camera; consecutive frames have visual overlap; file names must be ordered sequentially), Vocabulary Tree matching.
- Camera models supported: SIMPLE_RADIAL (default), PINHOLE, OPENCV, SIMPLE_RADIAL_FISHEYE, OPENCV_FISHEYE.
- Database: SQLite database file stores extracted data.
- Masking: Supports masking of keypoints via `mask_path` folder or `camera_mask_path` single mask.

**GLOMAP:**
- Repository: https://github.com/colmap/glomap (2,351 stars, BSD-3-Clause)
- Status: **Deprecated** - "This project is no longer maintained and has been fully migrated to COLMAP, where GLOMAP functionality is exposed as the 'global' mapper." (README, 2024-05-29)
- Paper: https://arxiv.org/pdf/2407.20219 (ECCV 2024) - "Global Structure-from-Motion Revisited"
- Description: General purpose global SfM pipeline, requires COLMAP database as input, outputs COLMAP sparse reconstruction. Provides 1-2 orders of magnitude faster reconstruction with on-par or superior quality vs COLMAP incremental.
- Dependencies: COLMAP, PoseLib (built via FetchContent), CMake >=3.28 (or 3.10 with self-installed deps), Ceres Solver, Eigen.
- Build: `mkdir build && cmake .. -GNinja && ninja && ninja install`
- Usage: `glomap mapper --database_path DATABASE_PATH --output_path OUTPUT_PATH --image_path IMAGE_PATH`
- Notes: Recommend sharing camera intrinsics when same physical camera (`--ImageReader.single_camera_per_folder` or `single_camera_per_image`). Retriangulation behavior documented.

### 3.2 3D Gaussian Splatting (3DGS) and gsplat

**Core Paper:**
- "3D Gaussian Splatting for Real-Time Radiance Field Rendering" - Kerbl et al., SIGGRAPH 2023 Best Paper Award. Project page: https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/ , GitHub: https://github.com/graphdeco-inria/gaussian-splatting (reference implementation)

**gsplat Library:**
- Repository: https://github.com/nerfstudio-project/gsplat (also https://docs.gsplat.studio/main, https://rocm.docs.amd.com/projects/gsplat/en/latest/reference/gsplat-api-reference.html)
- Description: Open-source CUDA-accelerated differentiable rasterization of 3D Gaussians with Python bindings.
- Performance claims (docs.gsplat.studio): Up to **4x less training memory footprint** and up to **15% less training time** on Mip-NeRF 360 vs official implementation. Supports extremely large scene rendering.
- API: `rasterization(means, quats, scales, opacities, colors, viewmats, Ks, width, height)` with options including `radius_clip`, `antialiased`, `absgrad`.
- Dependencies: NVIDIA GPU, Python 3.10, `nerfstudio` methods `ns-process-data`, `ns-train splatfacto`, `ns-export gaussian-splat`.

**Hardware Requirements Research:**
- TheFuture3D (2026-04-03): "Minimum requirement: 8GB VRAM (e.g., RTX 3060). Recommended: 16-24GB VRAM. Cloud tools (Polycam, Luma AI) require no local GPU."
- TheFuture3D Cost Estimator: "minimum 8GB VRAM, recommended 24GB RTX 4090, processing requires more GPU compute"
- Student Guide (jamesroha.substack.com, 2026-02-18): "Nerfstudio requires NVIDIA GPU (8GB+ VRAM recommended)". Notes original implementation recommends 24GB (RTX 3090/4090) but workarounds exist: RTX 3060 (12GB) viable with reduced iterations (7k-15k), RTX 2060 (6-8GB) limited to smaller scenes with `--data_device cpu` at speed cost. Use of `--data_device cpu` reduces VRAM.
- ml-sharp-pinokio (francescofugazzi): "Minimum 16GB Unified Memory for Apple Silicon; Windows/Linux: Recommended 10GB+, Minimum 8GB supported via Low VRAM Mode (FP16) and shared memory." Includes precompiled gsplat wheels for CUDA and MPS-Lite.

**Rendering Characteristics:**
- Represent scene as millions of 3D Gaussian ellipsoids; achieves 100+ FPS rendering.
- Moving objects produce "floaters" - stray Gaussians requiring cleanup (e.g., SuperSplat editor https://superspl.at/editor).

### 3.3 Segment Anything Model (SAM)

**Repository:** https://github.com/facebookresearch/segment-anything

**Model Variants (facebookresearch):**
- `vit_h` (ViT-H): 2.4GB checkpoint, file `sam_vit_h_4b8939.pth` - https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth
- `vit_l` (ViT-L): 1.2GB, `sam_vit_l_0b3195.pth` - https://dl.fbaipublicfiles.com/segment_anything/sam_vit_l_0b3195.pth
- `vit_b` (ViT-B): 375MB, `sam_vit_b_01ec64.pth` - https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth
- Apache 2.0 license. Also SA-1B dataset.

**VRAM / Performance Research:**
- Clore AI docs table: `SAM-H 8GB VRAM / Best quality / Slow | SAM-L 6GB / Great / Medium | SAM-B 4GB / Good / Fast | SAM2 8GB+`
- GitHub Issue #33 (2023-04-06): User reports "I use 3090 24GB but it only need 7GB to load ViT-H". CamilleMaurice: "decrease points_per_batch from 64 to 16 - I was able to use 8GB VRAM GPU on big images." KnitVikas: reducing image resolution worked. kadirnar: "You should try vit_b model. GTX 1050ti (2GB) worked." encouver: "With vit_l model, it works perfectly without any extra params. And gives results similar to vit_h."
- Performance table (clore.ai): SAM-H 1024x1024 on RTX 3090 ~0.5s, SAM-L ~0.3s, SAM-B ~0.2s.

**Usage:**
- `sam = sam_model_registry["vit_h"](checkpoint="sam_vit_h_4b8939.pth"); sam.to("cuda")`
- `SamAutomaticMaskGenerator` with params `points_per_side, pred_iou_thresh, stability_score_thresh, crop_n_layers, crop_n_points_downscale_factor, min_mask_region_area`
- `SamPredictor` with `predict(point_coords, point_labels, multimask_output)`

### 3.4 Depth Anything V2

**Repository:** https://github.com/DepthAnything/Depth-Anything-V2 (8,730 stars, Apache-2.0) - TikTok/HKU, NeurIPS 2024. Paper: https://arxiv.org/abs/2406.09414, Project: https://depth-anything-v2.github.io/, Benchmark: https://huggingface.co/datasets/depth-anything/DA-2K

**Models:**
| Model | Params | Checkpoint |
|---|---|---|
| Depth-Anything-V2-Small | 24.8M | https://huggingface.co/depth-anything/Depth-Anything-V2-Small/resolve/main/depth_anything_v2_vits.pth |
| Depth-Anything-V2-Base | 97.5M | https://huggingface.co/depth-anything/Depth-Anything-V2-Base/resolve/main/depth_anything_v2_vitb.pth |
| Depth-Anything-V2-Large | 335.3M | https://huggingface.co/depth-anything/Depth-Anything-V2-Large/resolve/main/depth_anything_v2_vitl.pth |
| Depth-Anything-V2-Giant | 1.3B | Coming soon |

- Metric depth variants also: `metric_depth` folder with `-Small` and `-Base` based checkpoints (2024-06-22).
- Transformers integration (2024-07-06): Supported via `transformers` pipeline. Usage: `pipeline(task="depth-estimation", model="depth-anything/Depth-Anything-V2-Small-hf")` or `AutoModelForDepthEstimation.from_pretrained(...)`.
- Code: `from depth_anything_v2.dpt import DepthAnythingV2` (see run.py, run_video.py)

**VRAM / Speed Research:**
- Clore AI table: `Small 25M / 2GB VRAM / Fastest | Base 98M / 4GB / Fast | Large 335M / 8GB / Best quality`
- GPUBattle measurements (depth-anything/Depth-Anything-V2-Large-hf): Large runs on cards with VRAM; A100 etc listed. Small variant also bench tested.
- Video-Depth-Anything (related): Table shows Small 7.5ms FP16 / 6.8GB VRAM, Large 14ms / 23.6GB on A100 with 1x32x518x518 input (2025-02-08).

**Video Variant:** Video Depth Anything (https://github.com/DepthAnything/Video-Depth-Anything) with Relative and Metric models (Small 28.4M, Base 113.1M, Large 381.8M), improves temporal consistency for super-long videos (>5 min).

### 3.5 Other Related 3D Tools Mentioned in Research
- DUST3R / MASt3R - dense stereo priors for occlusion filling (not webfetched in depth, but noted as alternative to Depth Anything for single-view hallucination)
- OpenSplat, SuperSplat (https://superspl.at/editor) - for cleanup of floaters and Ply editing.

---

## 4. Dataset Research

### 4.1 UrbanScene3D

**Sources:**
- Official website: https://vcc.tech/UrbanScene3D (Visual Computing Research Center, Shenzhen University)
- GitHub (yilinliu77): https://github.com/yilinliu77/UrbanScene3D (130 stars)
- GitHub (Linxius): https://github.com/Linxius/UrbanScene3D (145 stars)
- Paper: https://arxiv.org/abs/2107.04286 (ECCV 2022)

**Content:**
- Total: **Over 128k high-resolution images covering 16 scenes, 136 km2 total area** (stated on both project page and paper).
- Composition: 6 virtual synthetic cities (lightweight CAD models) + 5+ real-world reconstructed regions (with realistic detailed structures) + simulator scenes. Total described as 16 scenes in 2021 paper overview; 2022 ECCV paper lists similar breakdown.
- Data types: Images, high-precision LiDAR scans, hundreds of image sets with different observation patterns, depth maps, 2D/3D instance segmentation maps, 3D point cloud/mesh segmentations, radar point clouds, original aerial photos of real scenes with camera poses, 4K videos for some scenes.
- Quality: Real scenes reconstructed by real-world aerial images with detailed geometries and realistic textures; instance segmentation available.
- Simulator: Based on Unreal Engine 4 (4.24 recommended) and AirSim (Microsoft AirSim APIs, C++ or Python). Requires Unreal project `UrbanScene.uproject`. All scenes located in `Contents/Maps`. Ground truth textured meshes and poses provided in Unreal project. Requires AirSim client or Unreal Engine to capture desired data. Path format documented: `image_name,x,y,z,pitch,roll,yaw` in UE left-handed Z-Up cm coordinates. FOV=60, resolution 6000x4000 for synthetic scenes; real scenes have different cameras requiring calibration. Capture via `cg_3_zuizhong.exe` with `Game.ini` configuration.

**Downloads:**
- Primary: https://vcc.tech/UrbanScene3D - Dataset size **UrbanScene3D (1.43TB)** publicly accessible for non-commercial use only.
- Mirrors:
  - Baidu Netdisk: https://pan.baidu.com/s/1ft1_5kFckPv7BTdMPlC4oA (code: vccc) or https://pan.baidu.com/s/1nqurXpbMzFo_-Cmf6eheOw?pwd=7zdg / https://pan.baidu.com/s/1ft1_5kFckPv7BTdMPlC4oA (code for yilinliu77 repo)
  - Dropbox: https://www.dropbox.com/sh/tx8n48ayjxjp9su/AACoNqF8VOosMvHXL1sDl4Qaa?dl=0 and https://www.dropbox.com/sh/8g2urrij2fercko/AABi0GclI-f96uYsAdP0D0Yga?dl=0
  - Google Drive: https://drive.google.com/drive/folders/1e91lEw56DUBbQgRTo48T3lVjo53SzEOd
  - HTTP: https://nas.moutong.org:4430/UrbanScene3D-VCC.zip
  - FTP: ftp://nas.moutong.org/dataset/UrbanScene3D-VCC.zip
  - NASA direct per-scene: Residence, Campus, Sci-Art, Square, Hospital via szuvccnas.quickconnect.cn links.
- Includes separate image-capturing programs zip (https://www.dropbox.com/sh/pw09ebaa6k4phzr/AABsXdqRusZp7WEtQ7qWledOa?dl=0) for synthetic scenes.
- Evaluation tools: Compiled `Evaluation.zip` and `Evaluation_data.zip` with `evaluate_model.exe p1 path_to_recon p3 path_to_gt` (mesh/points), encrypted `.pointcloud` format, Requires pre-registration with polytech1k.ply or artsci1k.ply.

**License:** "All DATA and CODE are free for Research and Education Use ONLY." Conditions: Provide AS IS, cite paper, no dissemination of altered variations, no commercial use.

**Alternatives Noted Briefly:**
- UrbanScene3D-V1 available via Dropbox/Baidu per-scene links.
- Sampled points for virtual cities provided due to copyright: https://github.com/Linxius/UrbanScene3D/releases/download/v0.0.1/UrbanScene3D-virtual_cities-sampled.7z

### 4.2 Other Datasets Mentioned
- BlendedMVS, Mill19, AI City Challenge CityFlow - mentioned as potential public alternatives for testing but not webfetched in detail for this PS.

---

## 5. Hugging Face Spaces and ZeroGPU Research

### 5.1 Hugging Face Spaces Overview
- Documentation: https://huggingface.co/docs/hub/spaces-overview
- Types: Gradio SDK Spaces, Docker Spaces, Static Spaces.

### 5.2 ZeroGPU Specifics

**Source:** https://huggingface.co/docs/hub/spaces-zerogpu (including archived .md copy at https://github.com/huggingface/hub-docs/blob/main/docs/hub/spaces-zerogpu.md) and skill file https://github.com/huggingface/skills/blob/main/skills/huggingface-zerogpu/SKILL.md

**Definition:**
- ZeroGPU is a shared infrastructure that dynamically allocates and releases NVIDIA GPUs as needed for Hugging Face Spaces.
- Optimizes GPU usage, offers Multi-GPU support on single application.
- Alternative to single-GPU allocation; aims at resource utilization and power efficiency.

**Hardware:**
- Backing hardware: **NVIDIA RTX Pro 6000 Blackwell Workstation Edition** (verified in hub-docs file: large = Half GPU 48GB, xlarge = Full GPU 96GB)
- Sizes table:
  | GPU size | Backing hardware | VRAM | Quota cost |
  |---|---|---|---|
  | large (default) | Half RTX Pro 6000 Blackwell | 48GB | 1x |
  | xlarge | Full RTX Pro 6000 Blackwell | 96GB | 2x |

**Access:**
- Free GPU access for Spaces users.
- Using existing ZeroGPU Spaces free to all users (curated list: https://huggingface.co/spaces/enzostvs/zero-gpu-spaces)
- PRO users and Enterprise/Team plans can host ZeroGPU Spaces without restrictions. Organizations require Team/Enterprise subscription to enable ZeroGPU for members.

**Compatibility:**
- Currently exclusively compatible with **Gradio SDK** (Docker and Static cannot schedule onto ZeroGPU; Streamlit now runs as Docker, so excluded).
- Compatibility enhanced for Hugging Face libraries `transformers` and `diffusers`; may have limited compatibility vs standard GPU Spaces.
- Supported versions: Gradio 4+, PyTorch 2.8.0 to latest (2.8.0, 2.9.1, 2.10.0, 2.11.0 listed), Python 3.12.12 / 3.10.13.

**CUDA Availability Model (from skill file):**
- Real GPU access only inside `@spaces.GPU`-decorated functions.
- `import spaces` monkey-patches `torch` so `torch.cuda.is_available()` returns `True` globally and `.to("cuda")` at module scope succeeds.
- Intended workflow: Write `device="cuda"` at module scope - tensors registered and offloaded to disk at startup "pack" step; forked GPU worker streams weights from disk to VRAM via pinned-memory pipeline when `@spaces.GPU` call lands. Warm workers keep weights resident on GPU.

**Code Pattern:**
```python
import spaces
import torch
from transformers import pipeline

# module-scope - registered for offload
pipe = pipeline(task="depth-estimation", model="depth-anything/Depth-Anything-V2-Small-hf", device="cuda")

@spaces.GPU
def predict(image):
    return pipe(image)["depth"]
```

**Constraints:**
- Pickle-based process isolation; `gr.State` semantics across worker boundary.
- No `torch.compile` (use AoTI instead).
- CUDA wheel-only builds (no `nvcc` at build or runtime).
- Duration tuning required: `@spaces.GPU(duration=60)` etc.; errors like `PicklingError` or `illegal duration` or `flash-attn` wheel-build failures indicate ZeroGPU-specific issues.

---

## 6. Hardware Constraint Research - 8GB VRAM Local Development

### 6.1 Context Provided
- Local development constraint: 8GB VRAM available.
- Deployment intent: Hugging Face Space (now clarified as ZeroGPU-capable, with 48GB/96GB available).

### 6.2 8GB VRAM Feasibility (from webfetched requirements)
- **COLMAP/GLOMAP:** CPU-bound for feature extraction and matching; VRAM not primary constraint. Feasible on 8GB system. For video, down-sample frame rate and use sequential matching.
- **SAM:** Requires model selection:
  - SAM-H on 8GB reported OOM without tuning; needs `points_per_batch=16` or lower, or lower resolution. Default needs ~7GB.
  - SAM-B verified to work on GTX 1050Ti (2GB) and 4GB, with 375MB checkpoint - recommended for 8GB.
  - SAM-L (1.2GB checkpoint, ~6GB) is middle option.
- **Depth Anything V2 Small:** 2GB VRAM, feasible; Base 4GB, Large 8GB (borderline). Small is sufficient for prototype.
- **3DGS/gsplat:** Original implementation recommended 24GB, but gsplat 4x memory efficient. Verified minimum 8GB with Low VRAM Mode (FP16) and shared memory. Requires downscaling images to ~800p-1280p, limiting to 60-80 frames, reducing iterations from 30k to 7k-15k, and using `--data_device cpu` if needed at speed cost. RTX 3060 12GB viable with 7k-15k iterations; RTX 2060 6-8GB limited to smaller scenes.
- **UrbanScene3D Simulator:** Requires Unreal Engine 4.24, optional AirSim; 1.43TB dataset requires selective download (single scene).

### 6.3 Hugging Face ZeroGPU as Deployment Target
- ZeroGPU provides 48GB/96GB on demand inside `@spaces.GPU`, free, so final model training/inference can be offloaded to HF rather than local 8GB.
- Local training at reduced resolution/iterations matches SIH speed requirement (<15 min for 10 min video); final quality can be demonstrated on reduced scale or on Colab/ZeroGPU for full resolution.

---

## 7. Existing Open-Source End-to-End Repos (Directly Relevant to Single-Pass Drone 3D)

### 7.1 BrandonRobare/telemetry-frame-mapper

**Source:** https://github.com/BrandonRobare/telemetry-frame-mapper (fetched 2026-09-04)

**Metadata:**
- Stars: 4, Forks: 0, Watchers: 1, Commits: 1,019 (main branch), License: MIT
- Python 3.11+, Republished wheel `drone-video-geotagger`, Frontend: Vite + React, Backend: FastAPI (28 routers, self-documenting at `/docs`), Packaging: PyInstaller + Inno Setup (Windows installer), Docker image available.
- Topics: colmap, dji, drone, exif, fastapi, gaussian-splatting, geotagging, opendronemap, photogrammetry, webodm

**One-line description (README):** "Turn DJI drone video into GPS-registered 3D gaussian splats — geotag frames, map coverage, plan missions, and reconstruct, with WebODM/OpenDroneMap-ready output."

**Core Problem Solved:** DJI drones record GPS telemetry into a subtitle track inside the video file. Extracted still frames lose location; this project extracts telemetry, interpolates per-frame position, and writes GPS EXIF tags so photogrammetry software can use them.

**Pipeline (verbatim from README diagram):**
```
DJI video ──ffmpeg──> frames ──CLI──> geotagged JPGs ──import──> map/review/plan
                                                                      │
                                              COLMAP SfM ──> gsplat training ──> splat viewer,
                                                                                 LAS/mesh/GeoJSON export
```
Every stage also produces WebODM/OpenDroneMap-ready output (stop at any point and take frames elsewhere).

**CLI Details:**
- Wheel install: `pip install drone-video-geotagger` (CLI-only)
- Full web app requires cloned checkout: `uv sync --group backend --group dev` (PEP 735 groups, not pip extras) or Docker/Windows installer.
- Dependencies: `ffmpeg` and `exiftool` on PATH (or `--ffmpeg`/`--exiftool` flags); COLMAP + CUDA GPU only for reconstruction.
- Commands:
  - `ffmpeg -i flight.mp4 -vf fps=8 extracted/frame_%05d.jpg`
  - `drone-video-geotagger --video flight.mp4 --frames extracted --takeoff-altitude 236.94` (writes `extracted_geotagged/` with `frame_geotags.csv` per-frame index/time/coords/altitudes + `exiftool_geotags.args`)
  - Flags: `--video` (MP4 required), `--frames` (folder required), `--takeoff-altitude` (metres ASL required), `--output` (defaults `<frames>_geotagged`), `--srt` (existing SRT), `--frame-rate`, `--ffmpeg`, `--exiftool`, `--in-place`
  - Frame numbering uses last number in filename (`frame_00042.jpg` and `DJI_0081_frame_42.jpg` both resolve to 42); files with no digits skipped.
  - Batch: `dvg-pipeline job.yml [--dry-run] [--output-root] [--log-dir] [-v]` (geotag → GPS validation → coverage via YAML spec; reconstruction/export need COLMAP/gsplat + DB, else exits 1). Example spec: `docs/examples/pipeline-job-spec.yml`.

**Web App:**
- Launch: `./run.sh` (macOS/Linux) or `run.bat` (Windows), then `http://localhost:5173` ; `dev.sh`/`dev.bat` also create venv and install deps on first run.
- Tabs follow flight: import/review frames, coverage on map, plan next flight, run reconstruction, explore splat, export.
- Backend: FastAPI 28 routers covering import, quality scoring, footprint geometry, coverage analysis, mission planning, reconstruction job pipeline, export formats.
- Docs: `docs/WORKFLOW.md` (end-to-end tutorial), `USER-MANUAL.md`, `INSTALL.md`, `SETUP.md` (GPU/CUDA/gsplat), `TROUBLESHOOTING.md`, `ARCHITECTURE.md`, `WEBODM.md`, `CESIUM-ION.md`, `VENDOR-PROJECT-IMPORT.md`, `WINDOWS-INSTALLER.md`, `CHANGELOG.md`.

**Docker:**
- CPU-only image serves backend + built frontend in one container/one API process, binds to `127.0.0.1:8000:8000` (loopback intentionally). Requires auth via `config.yaml` + `DRONE_MAPPING_PIN_HASH` if exposing to LAN; respects `deployment.allow_unauthenticated_lan`. Bundles `ffmpeg`, `exiftool`, COLMAP; GPU training out of scope for Docker image - use manual setup.

**Repository Layout:** `src/drone_video_geotagger` (CLI), `backend/` (FastAPI DB models/services), `frontend/` (Vite/React), `tests/` (pytest `tests/cli/` and `tests/backend/`), `packaging/` (Windows installer), `docs/`, runtime dirs `data/`, `imports/`, `processed/`, `exports/` (gitignored).

**Relevance to SIH26158:** Directly implements the DJI视频+SRT → GPS EXIF → COLMAP → Gaussian Splatting chain required by SIH26158, including georeferencing and multi-format export (LAS/mesh/GeoJSON) and viewer. Explicitly handles the SIH-listed inputs (video + GPS + flight metadata) and optional RTK/PPK via SRT parsing.

### 7.2 ch1bo/drone-reconstruction

**Source:** https://github.com/ch1bo/drone-reconstruction (fetched 2026-09-04)

**Metadata:**
- Stars: 10, Forks: 1, Commits: 40 (master), Submodule: `nerfstudio @ ce71de2` (nerfstudio-project/nerfstudio)
- Environment: Nix flakes (`flake.nix`, `flake.lock`, `.envrc` via direnv), two shells: `nix develop` (lightweight: colmap, ffmpeg) and `nix develop .#full` (full pipeline including CUDA + nerfstudio via `pip install -e .` in postVenvCreation). Binary cache `cache.nixos-cuda.org`. Patch `patches/tiny-cuda-nn-cache-env.patch` respects `TCNN_CACHE_PATH` for tiny-cuda-nn JIT kernels.
- License: Not explicitly stated in fetched README; reference repos MIT/Apache-2.0 for components.
- Goal: Walkable 3D model with real-world scale (meters) and geographic positioning (GPS overlay on map) from monocular drone video, neighborhood-scale (few hundred meters across, moderate altitude); indoor/close-range out of scope.

**One-line description (GitHub header):** "A modern pipeline for creating high-quality 3D reconstructions of neighborhoods and outdoor environments from monocular drone video footage using Neural Radiance Fields (NeRF) and 3D Gaussian Splatting"

**Flight Planning Guidance Documented:**
- Works well: Grid (lawnmower) pattern with ~70% lateral overlap, double-grid (perpendicular passes) even better; consistent altitude and fixed camera angle (nadir or fixed oblique); separate passes for area vs facades; loops at two altitudes (e.g., 20m and 40m) to avoid corridor ambiguity.
- Causes problems: Omnidirectional panning/rotating in place (violates sequential matching, too few tie-points, degenerate focal length), descent followed by flat flight (scale/perspective mismatch), too few frames (<2 fps at normal speed, need 5 fps if slow/repetitive texture), single-altitude loops (identical loops causing vertical drift).
- Diagnosing bad reconstruction: Check camera trajectory in COLMAP GUI - spiral/corkscrew (scale drift), degenerate intrinsics (`fx ≫ fy` or `|k1|>0.1`), reprojection error >1 px, multiple sub-models instead of one.

**Pipeline (verbatim structure):**
```
DJI video (.MP4) + telemetry (.SRT)
        │
        ▼
  ffmpeg                    ← extract frames
  colmap                    ← feature extraction, matching, sparse SfM
        │
        ▼
  GPS alignment             ← srt_to_reference_poses.py + colmap model_aligner (Sim3 fit to GPS in ENU, metres)
        │
        ▼
  colmap MVS                ← dense point cloud: image_undistorter, patch_match_stereo (CUDA), stereo_fusion → fused.ply (georeferenced)
        │
        ▼  (optional)
  ns-train splatfacto       ← 3D Gaussian Splatting (30-60 min on RTX 4070 Ti)
        │
        ▼
  ns-viewer / ns-export     ← interactive viewing or mesh/splat export
```

**Key Commands (variables: `SCENE=data/flight1`, `VIDEO=input/DJI_*.MP4`):**
- Frame extraction: `ffmpeg -i $VIDEO -vf fps=2 -q:v 2 $SCENE/images/frame_%06d.jpg` (2 fps = ~600 frames for 5 min flight, near-lossless JPEG quality)
- Feature extraction: `colmap feature_extractor --database_path $SCENE/database.db --image_path $SCENE/images --ImageReader.camera_model OPENCV --ImageReader.single_camera 1 --FeatureExtraction.use_gpu 1` (OPENCV supports k1,k2,p1,p2; single_camera 1 reduces BA degrees of freedom for same lens)
- Matching: `colmap sequential_matcher --database_path $SCENE/database.db --FeatureMatching.use_gpu 1` (O(n²) exhaustive avoided; sequential matches neighbours only)
- Mapper: `colmap mapper --database_path $SCENE/database.db --image_path $SCENE/images --output_path $SCENE/sparse` (outputs `sparse/0`, `sparse/1` etc., ordered by registered images)
- GPS alignment: `python scripts/srt_to_reference_poses.py --srt ${VIDEO%.MP4}.SRT --video $VIDEO --fps 2 --output $SCENE/reference_poses.txt` → `colmap model_aligner --input_path $SCENE/sparse/0 --output_path $SCENE/aligned --ref_images_path $SCENE/reference_poses.txt --ref_is_gps 1 --alignment_type enu --alignment_max_error 10` (Sim3 fit for rotation+translation+scale; ENU = East-North-Up local, origin at first camera, metres; uses rel_alt (infrared altimeter ~0.1m precision) over abs_alt (GPS MSL ~10-20m) for better scale; consumer GPS horizontal accuracy ~5m dominant error, relative distances sub-meter accurate)
- Dense MVS: `colmap image_undistorter --image_path $SCENE/images --input_path $SCENE/aligned --output_path $SCENE/dense --output_type COLMAP` then `colmap patch_match_stereo --workspace_path $SCENE/dense --PatchMatchStereo.geom_consistency true` (CUDA, run in `nix develop .#full`) then `colmap stereo_fusion --workspace_path $SCENE/dense --input_type geometric --output_path $SCENE/dense/fused.ply` (CPU, sensitive to RAM - several GB for 1000 full-res frames; output same coordinate system as aligned model, already georeferenced if alignment done)
- Training: `ns-train splatfacto --data $SCENE --output-dir outputs/ colmap-data-parser-config --colmap-path colmap/aligned --assume-colmap-world-coordinate-convention False` (must skip default Y/Z swap for ENU already +Z-up; trains 30-60 min on RTX 4070 Ti; NeRF alternative Zip-NeRF noted as potentially better for fine detail but 2-8hr vs 30min)
- Export/Viewer: `ns-viewer --load-config outputs/flight1/splatfacto/<timestamp>/config.yml`, `ns-render camera-path`, `ns-export pointcloud`, `ns-export poisson`

**DJI SRT Telemetry Format Handled:** Modern firmware `[latitude: 47.327540] [longitude: 9.603926] [rel_alt: 4.000 abs_alt: 377.161]`, older `GPS(47.327540,9.603926,377.161)`; `scripts/srt_to_reference_poses.py` interpolates SRT one-per-second entries to sub-second frames via `srt` library + regex.

**Common Issues Documented:**
- Headless server: `QT_QPA_PLATFORM=offscreen`, `--FeatureExtraction.use_gpu 0`/`--FeatureMatching.use_gpu 0` or `--cpu` for no GPU (2-3x slower)
- Alignment fails "not enough inliers": check SRT→video pairing, increase `--max-error` 10→15-20m for noisy GPS
- OOM during training: reduce `--pipeline.datamanager.train-num-rays-per-batch` or re-extract at lower resolution via `ffmpeg -vf scale=`

**Relevance to SIH26158:** Direct end-to-end example for SIH26158 inputs (DJI video + SRT GPS) producing all required outputs (sparse + dense georeferenced point cloud in ENU metres `fused.ply` usable in MeshLab/CloudCompare, plus optional Gaussian splats). Explicitly solves the SIH challenges of single-pass limited overlap (recommends 70% grid but shows failure modes for non-grid), GPS inaccuracies (Sim3 ENU alignment with rel_alt), and scale without GCPs. Documented runtime fits the <15 min target when using reduced iterations/resolution; dense MVS is CPU/GPU split suitable for 8GB and HF ZeroGPU offload.

---

## 8. Verified Links and Sources

- SIH 2026 Problem Statements: https://www.sih.gov.in/sih2026PS
- NTRO Additional Info Drive: https://drive.google.com/file/d/119hjXkLhMW_AhQ4cyYz-XJgcVz4BA-hD/view?usp=drive_link
- COLMAP: https://colmap.github.io/ | https://colmap.github.io/tutorial.html | https://colmap.github.io/faq.html
- GLOMAP: https://github.com/colmap/glomap | https://colmap.github.io/ | https://lpanaf.github.io/eccv24_glomap/ | Paper https://arxiv.org/pdf/2407.20219
- 3D Gaussian Splatting original: https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/ | https://github.com/graphdeco-inria/gaussian-splatting | Paper Kerbl et al. SIGGRAPH 2023
- gsplat: https://github.com/nerfstudio-project/gsplat | https://docs.gsplat.studio/main | https://rocm.docs.amd.com/projects/gsplat/en/latest/reference/gsplat-api-reference.html | Nerfstudio docs https://docs.nerf.studio/nerfology/methods/splat.html
- Future3D comparisons: https://www.thefuture3d.com/blog-0/2026/4/3/gaussian-splatting-software-tools-compared-2026
- SAM: https://github.com/facebookresearch/segment-anything | SAM checkpoints via https://dl.fbaipublicfiles.com/segment_anything/sam_vit_{h,l,b}_*.pth | Issue #33 https://github.com/facebookresearch/segment-anything/issues/33
- Depth Anything V2: https://github.com/DepthAnything/Depth-Anything-V2 | Paper https://arxiv.org/abs/2406.09414 | Hugging Face docs https://huggingface.co/docs/transformers/v4.56.1/en/model_doc/depth_anything_v2 | Model cards https://huggingface.co/depth-anything/Depth-Anything-V2-Small-hf etc.
- UrbanScene3D: https://vcc.tech/UrbanScene3D | https://github.com/yilinliu77/UrbanScene3D | https://github.com/Linxius/UrbanScene3D | Paper https://arxiv.org/abs/2107.04286
- HF Spaces ZeroGPU: https://huggingface.co/docs/hub/spaces-zerogpu | https://huggingface.co/docs/hub/spaces-zerogpu.md | https://github.com/huggingface/hub-docs/blob/main/docs/hub/spaces-zerogpu.md | Skill https://github.com/huggingface/skills/blob/main/skills/huggingface-zerogpu/SKILL.md | Curated ZeroGPU spaces list https://huggingface.co/spaces/enzostvs/zero-gpu-spaces

---

*This file is pure objective research with no bias, recommendations, or plan of action. All claims are derived from webfetched sources and the SIH portal dump dated 2026-09-04. File last updated: 2026-09-04.*
