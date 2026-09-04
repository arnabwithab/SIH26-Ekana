# Problem Statement - SIH26058 / SIH26158

**Title:** Single-Pass Drone Video to Accurate 3D Model Generation System

**Organisation:** National Technical Research Organisation (NTRO)

**Category:** Software

**Theme:** Drone / Robotics

**Problem Statement ID:** 26158 (also referenced as PS 17, SIH26058 in pagination)

---

### Background

Generation of accurate 3D models of buildings, infrastructure, terrain, and objects typically requires multiple drone passes, extensive image overlap, specialized flight planning, and significant post-processing time. In operational scenarios such as disaster response, surveillance, infrastructure inspection, military reconnaissance, and rapid mapping, there is often only a single opportunity to capture data over the target area. A solution capable of generating an accurate and textured 3D model from a single drone pass video would significantly reduce mission time, operator effort, data acquisition requirements, and processing complexity while enabling near real-time situational awareness.

### Description

Design and develop an AI-enabled system capable of generating a georeferenced and metrically accurate 3D model of a scene using only a single-pass drone video stream captured from a moving UAV. The system should process video frames captured during one flight path and reconstruct:

(i) 3D terrain and structures
(ii) Building facades and rooftops
(iii) Roads and infrastructure
(iv) Vegetation and obstacles
(v) Textured 3D meshes or point clouds

### Expected Solution / Deliverables

The generated model should be suitable for visualization, measurement, and analysis purposes.

### Key Challenges

(i) Limited viewing angles due to single flight path.
(ii) Motion blur and video compression artifacts.
(iii) Variable illumination and shadows.
(iv) Dynamic objects (vehicles, humans, animals).
(v) GPS inaccuracies and sensor noise.
(vi) Real-time or near-real-time processing requirements.
(vii) Reconstruction of occluded surfaces.
(viii) Maintaining metric accuracy without extensive Ground Control Points (GCPs).

### Input Data

**Mandatory**
(i) Drone video (1080p/4K)
(ii) GPS coordinates
(iii) Flight metadata

**Optional**
(i) IMU data
(ii) Barometric altitude
(iii) Camera intrinsic parameters
(iv) RTK/PPK corrections

### Desired Output

| Parameter | Target |
|---|---|
| Reconstruction Type | 3D Mesh / Point Cloud |
| Processing Time | < 15 minutes for 10-minute video |
| Spatial Accuracy | < 1m |
| Coverage | Entire visible scene |
| Output Formats | OBJ, PLY, LAS, GeoTIFF, .glb/.gltf, .fbx |
| Visualization | Web-based or Desktop Viewer |

### Evaluation Criteria

| Criteria | Weightage |
|---|---|
| Reconstruction Accuracy | 30% |
| Model Completeness | 20% |
| Processing Speed | 20% |
| Innovation | 15% |
| Scalability | 10% |
| User Interface | 5% |

### Potential Applications

(i) Border and strategic area mapping
(ii) Disaster damage assessment
(iii) Urban planning and smart cities
(iv) Infrastructure inspection
(v) Construction progress monitoring
(vi) Archaeological documentation
(vii) Digital twin generation
(viii) Military reconnaissance and mission planning

### Links

- **SIH 2026 Problem Statements Portal:** https://www.sih.gov.in/sih2026PS (Problem Statement ID 26158)
- **Additional Information (NTRO):** https://drive.google.com/file/d/119hjXkLhMW_AhQ4cyYz-XJgcVz4BA-hD/view?usp=drive_link
- **Youtube/Video Link:** Nil
- **Dataset Link:** Will be provided real time.

*Source: SIH 2026 official problem statement document, pages 37-39 of 53.*
