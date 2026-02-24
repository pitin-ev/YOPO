# YOPO: You Only Pose Once

Official implementation of **YOPO** from the paper:

**You Only Pose Once: A Minimalist's Detection Transformer for Monocular RGB Category-level 9D Multi-Object Pose Estimation**  
[Hakjin Lee](https://scholar.google.com/citations?user=GIdEXLsy77UC), [Junghoon Seo](https://mikigom.github.io/), [Jaehoon Sim](https://www.linkedin.com/in/jaehoon-sim-801404175/)

[\[arXiv\]](https://arxiv.org/abs/2508.14965) 
[\[Project page\]](https://mikigom.github.io/YOPO-project-page/)

This paper is accepted to **IEEE ICRA 2026**.

## Overview

YOPO is a single-stage transformer framework for RGB-only category-level 9D pose estimation.  
This repository is built on MMDetection and contains YOPO-specific configurations and model extensions.

Current YOPO entry points are in:

- `configs/yopo/`
- `yopo/models/detectors/sixd_pose/`
- `yopo/models/dense_pose_heads/`
- `yopo/datasets/pose_estimation/`
- `yopo/evaluation/metrics/`

## Model Zoo
### NOCS (REAL275)

| Model | Config | IoU50 | IoU75 | 10deg10cm | Checkpoint |
|---|---|---:|---:|---:|---|
| YOPO R50 | [file](configs/yopo/nocs_yopo_real_camera_r50.py) | 67.1 | 16.6 | 40.7 | [link]() |
| YOPO Swin-L | [file](configs/yopo/nocs_yopo_real_camera_swinl.py) | 71.6 | 16.4 | 52.8 | [link]() |
| YOPO Swin-L* | [file](configs/yopo/nocs_yopo_real_camera_swinl_finetune_real.py) | 79.6 | 19.6 | 54.1 | [link]() |

### HouseCat6D

| Model | Config | IoU25 | IoU50 | 10deg10cm | Checkpoint |
|---|---|---:|---:|---:|---|
| YOPO Swin-L | [file](configs/yopo/housecat6d_yopo_swinl.py) | 71.3 | 34.8 | 33.3 | [link]() |

## Environment

The target base environment is:

- Docker image: `pytorch/pytorch:2.4.0-cuda12.1-cudnn9-devel`

## Installation

```bash
# 1) clone
cd /path/to/YOPO

# 2) install OpenMMLab core deps with openmim
pip install -U pip setuptools wheel openmim
mim install "mmcv>=2.0.0rc4,<2.2.0" "mmengine>=0.7.1,<1.0.0"

# 3) install this repo (setup.py)
pip install -v -e .

# 4) (optional) install extra deps listed by the repo
#    use this if you need optional scripts/utilities beyond core train/test
pip install -r requirements.txt
```

## Datasets

If you prepared data following AG-Pose, keep AG-Pose preprocessing outputs but map them to this repository layout.

- AG-Pose reference: https://github.com/Leeiieeo/AG-Pose
- Use the dataset processing instructions and download links from AG-Pose README (`Data Processing` section), then place outputs under `data/`.

### NOCS (for `NOCSDataset`)
Expected root: `data/nocs/`

Required structure for this codebase:

```text
data/nocs/
  camera/
    train_list.txt
    val_list.txt
    <scene>/<frame>_color.png
    <scene>/<frame>_depth.png
    <scene>/<frame>_label.pkl
  camera_full_depths/
    <scene>/<frame>_composed.png
  real/
    train_list.txt
    test_list.txt
    <scene>/<frame>_color.png
    <scene>/<frame>_depth.png
    <scene>/<frame>_label.pkl
  segmentation_results/
    CAMERA25/
      results_val_<scene>_<frame>.pkl
    REAL275/
      results_test_<scene>_<frame>.pkl
```

Notes:

- The list files (`train_list.txt`, `test_list.txt`, etc.) should store frame IDs without suffixes (the loader appends `_color.png`, `_depth.png`, `_label.pkl`).
- `real_test` evaluation uses `segmentation_results/REAL275`.
- For `camera` samples, the loader looks for composed depth maps in `camera_full_depths`.

### HouseCat6D (for `HouseCat6DDataset`)
Expected root: `data/housecat6d/`

Required structure:

```text
data/housecat6d/
  scene*/            # train
  test_scene*/       # test
```

Each scene should include at least:

- `rgb/*.png`
- `depth/*.png`
- `labels/*_label.pkl`
- `intrinsics.txt`


## Training

### NOCS (R50)
```bash
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5 \
bash tools/dist_train.sh configs/yopo/nocs_yopo_real_camera_r50.py 6 --auto-scale-lr
```

### NOCS (Swin-L)
```bash
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5 \
bash tools/dist_train.sh configs/yopo/nocs_yopo_real_camera_swinl.py 6 --auto-scale-lr
```

### HouseCat6D (R50)
```bash
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5 \
bash tools/dist_train.sh configs/yopo/housecat6d_yopo_r50.py 6 --auto-scale-lr
```

### HouseCat6D (Swin-L)
```bash
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5 \
bash tools/dist_train.sh configs/yopo/housecat6d_yopo_swinl.py 6 --auto-scale-lr
```

## Fine-tuning (NOCS Real)

The repository includes a real-data fine-tuning setup:

- Config: `configs/yopo/nocs_yopo_real_camera_swinl_finetune_real.py`

Example:

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5 \
bash tools/dist_train.sh configs/yopo/nocs_yopo_real_camera_swinl_finetune_real.py 6 --auto-scale-lr \
  --cfg-options load_from=work_dirs/nocs_yopo_real_camera_swinl/epoch_12.pth
```

## Evaluation

```bash
python tools/test.py <CONFIG> <CHECKPOINT>
```

Examples:

```bash
python tools/test.py configs/yopo/nocs_yopo_real_camera_r50.py <path_to_ckpt>
python tools/test.py configs/yopo/nocs_yopo_real_camera_swinl.py <path_to_ckpt>
python tools/test.py configs/yopo/housecat6d_yopo_swinl.py <path_to_ckpt>
```


## Citation

```bibtex
@inproceedings{lee2026yopo,
  title     = {You Only Pose Once: A Minimalist’s Detection Transformer for Monocular RGB Category-level 9D Multi-Object Pose Estimation},
  author    = {Lee, Hakjin and Seo, Junghoon and Sim, Jaehoon},
  booktitle = {IEEE International Conference on Robotics and Automation (ICRA)},
  year      = {2026}
}
```

## Acknowledgement

This work was supported by Institute of Information & Communications Technology Planning & Evaluation (IITP) grant funded by the Korea government (MSIT) (RS-2025-02653113, High-Performance Research AI Computing Infrastructure Support at the 2 PFLOPS Scale)

This repository is based on MMDetection:

- https://github.com/open-mmlab/mmdetection
