# GLOMAP: Global Structure-from-Motion
## Overview

**GLOMAP** is a global Structure-from-Motion (SfM) tool for large-scale 3D reconstruction from unordered image collections. This guide walks you through installing and setting up GLOMAP (and its dependency COLMAP) on Google Colab, running SfM, and saving results.

---

## Installation Guide

### 1. Mount Google Drive

```python
from google.colab import drive
drive.mount('/content/drive')
```

### 2. Environment Check

Ensure CUDA, PyTorch, and other essential tools are available:

```python
!nvcc --version
import torch
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA device count: {torch.cuda.device_count()}")
!nvidia-smi
!python --version
!python -c "import torch; print(torch.__version__)"
!python -c "import torch; print(torch.version.cuda)"
!cmake --version
```

Minimum **CMake** version required: **3.28**

---

### 3. Install System Dependencies

```bash
!apt-get update
!apt-get install -y git build-essential libboost-all-dev libeigen3-dev libgoogle-glog-dev \
    libgflags-dev libfreeimage-dev libceres-dev libflann-dev libsuitesparse-dev libmetis-dev \
    ninja-build libgl1-mesa-dev libglu1-mesa-dev libglew-dev libx11-dev libxxf86vm-dev \
    libxrandr-dev libxinerama-dev libxcursor-dev libxi-dev libcgal-dev qtbase5-dev qtchooser \
    qt5-qmake qttools5-dev-tools wget
```

---

### 4. Clone and Build GLOMAP

```bash
!rm -rf /content/glomap
!git clone https://github.com/colmap/glomap.git /content/glomap
%cd /content/glomap
!mkdir -p build
%cd build
!cmake .. -GNinja
!ninja && ninja install
```

Copy the build to Google Drive:

```bash
!mkdir -p /content/drive/MyDrive/glomap_install/lib
!cp /usr/local/bin/glomap /content/drive/MyDrive/glomap_install/
!cp /usr/local/lib/libglomap*.a /content/drive/MyDrive/glomap_install/lib/ 2>/dev/null || echo "No libglomap*.a files found"
!cp /usr/local/lib/libglomap*.so* /content/drive/MyDrive/glomap_install/lib/ 2>/dev/null || echo "No libglomap*.so files found"
```

---

### 5. Install GLOMAP (from my saved build)

```python
def install_glomap():
    import subprocess
    try:
        result = subprocess.run(['glomap', '-h'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        if result.returncode == 0:
            print("GLOMAP is already installed")
            return True
    except:
        print("Installing GLOMAP...")

    !sudo mkdir -p /usr/local/bin /usr/local/lib
    !sudo cp /content/drive/MyDrive/glomap_install/glomap /usr/local/bin/
    !sudo cp /content/drive/MyDrive/glomap_install/lib/* /usr/local/lib/ 2>/dev/null || echo "No library files to copy"
    !sudo chmod +x /usr/local/bin/glomap
    !sudo ldconfig

    try:
        result = subprocess.run(['glomap', '-h'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        if result.returncode == 0:
            print("GLOMAP has been successfully installed")
            return True
        else:
            print("GLOMAP installation failed")
            return False
    except:
        print("GLOMAP installation failed")
        return False

# Usage
install_glomap()
```

Test GLOMAP installation:

```bash
!glomap -h
```

---

### 6. Setup Working Directories

```python
import os

IMAGE_PATH = "/content/drive/MyDrive/colmap_data/images"
LOCAL_IMAGE_PATH = "/content/glomap_data/images"
OUTPUT_PATH = "/content/glomap_output"
DATABASE_PATH = "/content/glomap_data/database.db"

os.makedirs(LOCAL_IMAGE_PATH, exist_ok=True)
os.makedirs(OUTPUT_PATH, exist_ok=True)

# Copy images locally for faster access
!cp -r {IMAGE_PATH}/* {LOCAL_IMAGE_PATH}/
```

---

## COLMAP Installation

If not already installed:

```bash
!apt-get update
!apt-get install -y git cmake build-essential libboost-all-dev libfreeimage-dev \
    libgoogle-glog-dev libgflags-dev libglew-dev libsuitesparse-dev libceres-dev \
    libeigen3-dev libflann-dev libcgal-dev libqt5opengl5-dev qtbase5-dev libmetis-dev
```

Install COLMAP from Drive(previously saved build):
here's the build in zip file:
https://github.com/DebBidhi/Colab-SfM-MVS-COLMAP-3D/blob/main/colmap_install_build.zip

```python
def install_colmap():
    import subprocess
    try:
        result = subprocess.run(['colmap', '-h'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        if result.returncode == 0:
            print("COLMAP is already installed")
            return True
    except:
        print("Installing COLMAP...")

    !sudo mkdir -p /usr/local/bin /usr/local/lib
    !sudo cp /content/drive/MyDrive/colmap_install/colmap /usr/local/bin/
    !sudo cp /content/drive/MyDrive/colmap_install/lib/* /usr/local/lib/
    !sudo chmod +x /usr/local/bin/colmap
    !sudo ldconfig

install_colmap()
```

---

## Structure from Motion (SfM) Pipeline

### Step 1: Feature Extraction

```bash
!colmap feature_extractor \
    --database_path {DATABASE_PATH} \
    --image_path {LOCAL_IMAGE_PATH} \
    --SiftExtraction.use_gpu 1
```

### Step 2: Feature Matching

```bash
!colmap sequential_matcher \
    --database_path {DATABASE_PATH} \
    --SiftMatching.use_gpu 1 \
    --SiftMatching.max_num_matches 8192 \
    --SequentialMatching.overlap 20 \
    --SequentialMatching.quadratic_overlap 1
```

---

## Running GLOMAP

```bash
!glomap mapper \
    --database_path {DATABASE_PATH} \
    --output_path {OUTPUT_PATH}/sparse
```

Save results:

```bash
!cp -r "/content/glomap_output/sparse" "/content/drive/MyDrive/golmap_sparse"
```

---

## Visualization

Install tools for visualization if needed:

```bash
!apt install -y xvfb
```

---

## Notes

- Ensure **CUDA driver** is compatible with runtime.
- Minimum **CMake 3.28** is required.
- COLMAP is required to prepare the database before running GLOMAP.
