# ICPARK-StructRecon Dataset

> A Multi-Modal Parking Lot Dataset for Structured Reconstruction

Due to anonymous review requirements, we are unable to upload the full raw dataset at this stage. The complete sensor data (~60 GB, including RGB images, depth maps, LiDAR point clouds, IMU, and wheel-odometry poses) will be open-sourced upon paper acceptance.

---

## Overview

The StructRecon dataset is a multi-modal sequence collected in a real-world underground parking lot, used to support the training and evaluation of the structured reconstruction method presented in the paper.

- **Duration:** ~4,593 s (~76.6 min)
- **RGB/Depth Frames:** 137,724 frames @ 30 Hz, resolution 1280 × 720
- **LiDAR Frames:** 45,950 frames @ 10 Hz
- **Coordinate System:** ROS/KITTI right-handed (X-forward, Y-left, Z-up)

---

## Directory Structure

The core data directories used by the method are as follows:

```
dataset/
├── calib/                     # Sensor calibration files
│   ├── calib_cam_to_cam.txt   # Camera intrinsics (KITTI format)
│   ├── calib_velo_to_cam.txt  # LiDAR → Camera extrinsics
│   └── zed_calib.yaml         # ZED raw calibration (authoritative intrinsics)
├── depth/                     # Depth-related images (PNG RGBA, 1280×720)
├── image/                     # RGB images (PNG RGBA, 1280×720)
├── pose/                      # Wheel-odometry poses (KITTI format)
│   ├── poses.txt              # 229,597 rows, 12 floats per row (3×4 row-major matrix)
│   └── timestamps.txt
├── pose_lidar_f2f/            # LiDAR frame-to-frame odometry (45,950 frames)
│   └── poses.txt
├── velodyne/                  # LiDAR point clouds (KITTI BIN format, float32)
└── matched_frames.txt         # LiDAR-centric cross-modal matching index (45,950 × 19)
```

---

## Data Description

### 1. RGB Images (`image/`)

- Format: PNG (RGBA, alpha channel always 255 — treat as RGB)
- Naming: `left000001.png` – `left137724.png` (1-indexed)
- Resolution: 1280 × 720, ~30 Hz
- Sensor: ZED stereo camera (left eye)

### 2. Depth-Related Images (`depth/`)

- Format: PNG (RGBA, 1280×720×4, uint8)
- Naming: `depth000001.png` – `depth137724.png` (1-indexed)
- Count: 137,724 frames, ~30 Hz
- Note: When read normally, appears as RGBA with identical values in the first three channels. Physical depth interpretation requires calibration files — do not assume standard 16-bit depth encoding.

### 3. LiDAR Point Clouds (`velodyne/`)

- Format: BIN (float32 binary, KITTI format)
- Naming: `000000.bin` – `045949.bin` (0-indexed)
- Count: 45,950 frames, ~10 Hz
- Data: 4 float32 per point — `[x, y, z, intensity]`

```python
import numpy as np
pc = np.fromfile("velodyne/000000.bin", dtype=np.float32).reshape(-1, 4)
# (N, 4): x, y, z, intensity
```

### 4. Pose Data

Two pose sources are used in the paper:

**Wheel-Odometry Poses (`pose/`)**
- `poses.txt`: 229,597 rows, 12 floats per row forming a 3×4 homogeneous transformation matrix (row-major), ~50 Hz
- Derived from wheel odometry integration — motion estimates, **NOT high-precision ground truth**

**LiDAR Frame-to-Frame Odometry (`pose_lidar_f2f/`)**
- `poses.txt`: 45,950 rows (one per LiDAR frame), relative poses between consecutive LiDAR sweeps

### 5. Calibration Files (`calib/`)

| File | Content |
|------|---------|
| `zed_calib.yaml` | Authoritative camera intrinsics: fx=fy=527.525 px, cx=636.298, cy=357.787, baseline≈120 mm |
| `calib_cam_to_cam.txt` | P0/P1 projection matrices (consistent with zed_calib.yaml) |
| `calib_velo_to_cam.txt` | R=I, T=(0.6, 0.0, −0.07) m |

```python
# Transform LiDAR point to camera frame:
# [x_c, y_c, z_c, 1]^T = [R|T]_velo_to_cam · [x_l, y_l, z_l, 1]^T
```

### 6. Cross-Modal Temporal Alignment (`matched_frames.txt`)

- Shape: 45,950 rows × 19 columns
- LiDAR-centric nearest-neighbor matching within a 0.1 s time window
- Per row: 1 LiDAR index + 3 image indices + 10 IMU indices + 5 pose indices
- Mean absolute time difference between each modality and LiDAR: ~5–25 ms

---

## Quick Start

```python
import cv2
import numpy as np

# RGB image
img = cv2.imread("image/left000001.png")[..., :3]

# Depth image
depth = cv2.imread("depth/depth000001.png", cv2.IMREAD_UNCHANGED)

# LiDAR point cloud
pc = np.fromfile("velodyne/000000.bin", dtype=np.float32).reshape(-1, 4)

# Wheel-odometry poses
poses = np.loadtxt("pose/poses.txt").reshape(-1, 3, 4)

# Cross-modal matching
matches = np.loadtxt("matched_frames.txt", skiprows=1)
```

---

## Citation

```bibtex
@article{structrecon2026,
  title={StructRecon: ...},
  author={...},
  year={2026}
}
```
