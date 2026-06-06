# EO19: A Large Dataset Construction for Insect Identification and Multi-Model Performance Assessment

> **Public Preview · Under Review**  
> Dataset links, full author list, trained checkpoints, and private paths are withheld pending publication.  
> **Paper / Dataset release:** TBA

---

## Overview

EO19 is a family-level, life-stage-aware insect detection dataset for agricultural pest monitoring.

| Item | Value |
|---|---|
| Taxonomy | Class Insecta → 4 orders, 19 families, **30 categories** |
| Images | **24,626** |
| Life-stage split | 11 holometabolous families → Adult / Larva (22 categories); 8 hemimetabolous families kept as-is (8 categories) |
| Annotation type | Bounding-box detection |
| Primary format | Pascal VOC XML; also exported to YOLO TXT and COCO JSON |
| Annotation tool | LabelImg, verified by entomology experts |

EO19 is designed to support both biological-taxonomy-aware insect recognition and practical agricultural pest detection. The dataset adopts the insect family as the main taxonomic unit and further separates adult and larval stages for holometabolous insects, where the visual morphology differs substantially across life stages.

---

## Important Notice

This repository is intended to document EO19 and provide the dataset-related files, EO19-adapted configuration files, environment records, and experimental logs used in our study.

The evaluated detectors were **not** integrated into one unified training framework. Each detector was trained using its corresponding upstream implementation or official codebase. Users should install the corresponding upstream repository first, and then apply the EO19-adapted configuration files provided here.

The `.py`, `.yml`, and `.yaml` files in this repository are **configuration files**, not trained model weights. Trained checkpoints, if released, will be provided separately.

---

## Dataset Versions

| Version | Description |
|---|---|
| **EO19-Original** | Real-world collected distribution. This version preserves the natural long-tailed distribution across categories. |
| **EO19-Augmented** | Long-tail-mitigated version. Categories are augmented and then downsampled to **1,000 images per category**. |

> **Note:** EO19-Original is recommended for studying real-world data imbalance and category distribution. EO19-Augmented is recommended for controlled comparisons where category imbalance should be reduced.

---

## Data Split

Train : Val : Test = **8 : 1 : 1**, stratified within each category.

<details>
<summary>Directory structure</summary>

```text
EO19_JSON/  # COCO JSON
  images/
    train/
    val/
    test/
  annotations/
    train.json
    val.json
    test.json

EO19_XML/  # Pascal VOC XML
  training/
    images/
    labels/
  val/
    images/
    labels/
  test/
    images/
    labels/

EO19_TXT/  # YOLO TXT
  training/
    images/
    labels/
  val/
    images/
    labels/
  test/
    images/
    labels/
  classes.txt
```

TXT label format:

```text
<class_id> <x_center> <y_center> <width> <height>
```

The YOLO TXT coordinates are normalized. XML/TXT splits use `training/` rather than `train/` as the training folder name.

</details>

---

## Quick Start

```bash
export EO19_ROOT=/path/to/EO19   # point to EO19-Original or EO19-Augmented
```

General usage steps:

1. Install the corresponding upstream model repository.
2. Copy or reference the EO19-adapted configuration file for that model.
3. Update dataset paths to point to `$EO19_ROOT`.
4. Set the number of categories to 30.
5. Train and evaluate using the original instructions of the upstream repository.

Key configuration changes for any framework:

- **Dataset path** → `$EO19_ROOT`
- **Class count** → `num_classes = 30` for DETR-style models / `nc: 30` for YOLO
- **Split paths** → update `ann_file` / `img_prefix` for COCO-style detectors, or `train` / `val` / `test` keys for YOLO

---

## Upstream Implementations

The following baselines were trained using their own upstream implementations. The EO19 repository only provides dataset-adaptation files and experimental records.

| Model group | Upstream implementation | EO19 adaptation |
|---|---|---|
| Co-DETR | MMDetection-based Co-DETR implementation | COCO-format dataset paths, 30 classes, EO19 training/evaluation config |
| Co-DINO | MMDetection-based Co-DINO implementation | COCO-format dataset paths, 30 classes, EO19 training/evaluation config |
| RT-DETRv2 | Official PyTorch implementation | COCO-format dataset paths, 30 classes, EO19 YAML config |
| DEIMv1 | Official PyTorch implementation | COCO-format dataset paths, 30 classes, EO19 YAML config |
| D-FINE | Official PyTorch implementation | COCO-format dataset paths, 30 classes, EO19 YAML config |
| YOLO series | Ultralytics implementation | YOLO-format dataset YAML and training scripts |

---

## Model Zoo

All metrics are reported on the **EO19-Original validation split**. Values are shown on the scale **[0, 1]**.

### DETR-family Models

COCO-style AP metrics are reported for DETR-family models.

| Model | Backbone | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---|---|---:|---:|---:|---:|---:|---:|
| Co-DINO | ViT-L + 5-scale SFP | 0.733 | 0.928 | 0.798 | 0.411 | 0.580 | 0.817 |
| D-FINE-L | HGNetv2-B4 | 0.701 | 0.900 | 0.762 | 0.321 | 0.534 | 0.799 |
| D-FINE-M | HGNetv2-B2 | 0.693 | 0.885 | 0.756 | 0.282 | 0.524 | 0.792 |
| DEIMv1 | HGNetv2-B2 | 0.692 | 0.887 | 0.754 | 0.309 | 0.545 | 0.794 |
| RT-DETRv2 | PResNet-18 | 0.666 | 0.866 | 0.729 | 0.277 | 0.492 | 0.770 |
| Co-DETR | ResNet-50 | 0.611 | 0.814 | 0.668 | 0.218 | 0.408 | 0.720 |

### YOLO-family Models

| Model | Precision | Recall | F1 | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|---:|
| YOLOv12n | 0.881 | 0.812 | 0.845 | 0.869 | 0.675 |
| YOLOv13n | 0.879 | 0.811 | 0.844 | 0.869 | 0.674 |
| YOLO11n | 0.847 | 0.736 | 0.788 | 0.813 | 0.612 |
| YOLOv8n | 0.807 | 0.740 | 0.772 | 0.806 | 0.608 |

---

## Reproducibility Details

The main paper reports only the principal experimental settings to keep the method section concise. Detailed training configurations, optimizer settings, learning rates, data augmentation schedules, and software environments are documented here for reproducibility.

### Principal Training Settings

| Model | GPU | Implementation | Backbone | Epochs | Batch | Input Size |
|---|---|---|---|---:|---:|---|
| Co-DETR | V100 32GB | MMDetection | ResNet-50 | 36 | 2 | Multi-scale |
| Co-DINO | A100 80GB | MMDetection | ViT-Large + 5-scale SFP | 12 | 1 | Multi-scale |
| RT-DETRv2 | V100 32GB | Official PyTorch | PResNet-18 | 120 | 16 | 640×640 |
| DEIMv1 | V100 32GB | Official PyTorch | HGNetv2-B2 | 102 | 32 | 640×640 |
| D-FINE-M | V100 32GB | Official PyTorch | HGNetv2-B2 | 60 | 32 | 640×640 |
| D-FINE-L | V100 32GB | Official PyTorch | HGNetv2-B4 | 80 | 32 | 640×640 |
| YOLO series | RTX 4090 | Ultralytics | Nano variants | 100 | 16 | 640×640 |

### Multi-scale Input Settings

| Model | Training input setting | Validation / test input setting |
|---|---|---|
| Co-DETR | Multi-scale resizing with short side from 480 to 800 and maximum long side 1333 | 1333×800 |
| Co-DINO | Multi-scale resizing with short side from 480 to 1536 and maximum long side 2400 | 2048×1280 |
| RT-DETRv2 | 640×640 | 640×640 |
| DEIMv1 | 640×640 | 640×640 |
| D-FINE-M | 640×640 | 640×640 |
| D-FINE-L | 640×640 | 640×640 |
| YOLO series | 640×640 | 640×640 |

### Optimizer and Schedule Summary

| Model | Optimizer | Main LR | Backbone LR | Weight Decay | Schedule / Notes |
|---|---|---:|---:|---:|---|
| Co-DETR | AdamW | 2e-4 | 2e-5 for backbone-related parameters | 1e-4 | Step schedule at epochs 27 and 33; 36 epochs total |
| Co-DINO | AdamW | 5e-5 | Layer-wise decay | 1e-2 | Step schedule at epoch 7; linear warmup for 500 iterations; 12 epochs total |
| RT-DETRv2 | AdamW | 1e-4 | Same as main LR | 1e-4 | Linear warmup for 2000 iterations; EMA enabled; 120 epochs total |
| DEIMv1 | AdamW | 4e-4 | 4e-5 | 1e-4 | 102 epochs total; DEIMv1 augmentation and schedule follow official configuration |
| D-FINE-M | AdamW | 2e-4 | 2e-5 | 1e-4 | 60 epochs total; schedule follows official D-FINE-M configuration |
| D-FINE-L | AdamW | 2.5e-4 | 1.25e-5 | 1.25e-4 | 80 epochs total; schedule follows official D-FINE-L configuration |
| YOLO series | Ultralytics default optimizer setting | See YOLO logs | See YOLO logs | See YOLO logs | 100 epochs; 640×640 input |

> The optimizer and schedule table is a compact summary. Exact values should be checked in the released adapter files and training logs.

### Environment Details

Because the baseline detectors were trained using different upstream repositories, they were not run under a single unified software environment. The main environment versions and official requirement specifications are summarized below. For packages not pinned by the upstream repository, the table records the official minimum requirement or marks the item as not specified.

| Model group | Python | PyTorch | TorchVision | CUDA | cuDNN | MMCV | MMDetection | Ultralytics | Other official requirements / notes |
|---|---|---|---|---|---|---|---|---|---|
| Co-DETR | 3.7.12 | 1.11.0 | 0.12.0 | 11.3 | 8.2 | 1.5.0 | 2.25.3 | N/A | MMDetection-based implementation; versions recorded from the training environment |
| Co-DINO | 3.7.12 | 1.11.0 | 0.12.0 | 11.3 | 8.2 | 1.5.0 | 2.25.3 | N/A | MMDetection-based implementation; versions recorded from the training environment |
| RT-DETRv2 | 3.8.20 | 2.4.1 | 0.19.1 | 12.1 | 9.1.2 | N/A | N/A | N/A | Official PyTorch implementation; versions recorded from the training log |
| DEIMv1 | 3.11.9 | 2.0.1 | 0.15.2 | 11.7 | 8.5 | N/A | N/A | N/A | faster-coco-eval>=1.6.5, PyYAML, tensorboard, scipy, calflops, transformers |
| D-FINE | 3.11.9 | >=2.0.1 | >=0.15.2 | Not pinned | Not pinned | N/A | N/A | N/A | faster-coco-eval>=1.6.6, PyYAML, tensorboard, scipy, calflops, transformers, loguru |
| YOLOv8n | 3.8.10 | 2.0.0| 0.15.1 | 11.8 | 8.7 | N/A | N/A | 8.1.0 | Version recorded from YOLOv8 training environment |
| YOLO11n | 3.8.10 | 2.0.0| 0.15.1 | 11.8 | 8.7 | N/A | N/A | 8.3.27 | Version recorded from YOLOv11 training environment |
| YOLOv12n | 3.12.3 | 2.5.1 | 0.20.1 | 12.4 | 9.1 | N/A | N/A | 8.3.63 | Version recorded from YOLOv12 training environment |
| YOLOv13n | 3.12.3 | 2.3.0 | 0.18.0 | 12.1 | 8.9.2 | N/A | N/A | 8.3.63 | Version recorded from YOLOv13 training environment |

`N/A` indicates that the package is not required by that implementation.

---

## Training Commands

The commands below show how the EO19-adapted configurations were launched in their corresponding upstream repositories. Private machine paths from the original experiments have been replaced with generic paths. Users should adjust dataset paths, pretrained checkpoint paths, and output directories according to their local environment.

### Co-DETR

```bash
python tools/train.py \
  projects/configs/co_deformable_detr/co_deformable_detr_r50_1x_coco.py \
  --work-dir work_dirs/co_detr_r50_eo19 \
  --launcher none
```

### Co-DINO

```bash
python tools/train.py \
  projects/configs/co_dino_vit/co_dino_5scale_vit_large_coco.py \
  --work-dir work_dirs/co_dino_vit_l_5scale_eo19 \
  --launcher none \
  --cfg-options \
  load_from=/path/to/pretrained/pytorch_model.pth \
  data.samples_per_gpu=1 \
  model.backbone.img_size=1024
```

### RT-DETRv2

```bash
python -u tools/train.py \
  -c configs/rtdetrv2/rtdetrv2_r18vd_sp3_120e_coco.yml \
  --output-dir outputs/rtdetrv2_r18vd_sp3_120e_eo19 \
  -u print_freq=10 2>&1 | tee -a outputs/rtdetrv2_r18vd_sp3_120e_eo19/train.log
```

> For RT-DETRv2, `-u print_freq=10` is used to update the configuration value. Do not replace it with `--print-freq 10`, which is not recognized by the training script.

### DEIMv1

```bash
CUDA_VISIBLE_DEVICES=0 python train.py \
  -c configs/deim_dfine/deim_hgnetv2_m_coco.yml \
  --use-amp \
  --seed=0 > outputs/deim_hgnetv2_m_eo19.log 2>&1
```

### D-FINE

```bash
# D-FINE-M
CUDA_VISIBLE_DEVICES=0 python train.py \
  -c configs/dfine/dfine_hgnetv2_m_coco.yml \
  --use-amp \
  --seed=0

# D-FINE-L
CUDA_VISIBLE_DEVICES=0 python train.py \
  -c configs/dfine/dfine_hgnetv2_l_coco.yml \
  --use-amp \
  --seed=0
```

### YOLO Series

The exact YOLO commands will be finalized after the final training logs are organized. The following commands are placeholders based on the Ultralytics command-line interface and should be checked against the actual commands used in the experiments before release.

```bash
# YOLOv8n


# YOLO11n

# YOLOv12n


# YOLOv13n
```

---

## EO19 Configuration Adapters for Upstream Repositories

The files below are EO19-adapted configuration files and environment records. They are **not** a replacement for the original model repositories, and they are **not** intended to merge all detectors into one monolithic training codebase.

Users should first install the corresponding upstream repository, then copy or reference the provided configuration files in the appropriate location of that upstream repository.

```text
eo19_adapters/

  co-detr_mmdetection/
    README.md
    co_deformable_detr_r50_1x_coco.py   # Co-DETR configuration adapted to EO19
    environment.txt

  co-dino_mmdetection/
    README.md
    co_dino_5scale_vit_large_coco.py    # Co-DINO configuration adapted to EO19
    environment.txt

  rtdetrv2_official/
    README.md
    rtdetrv2_r18vd_sp3_120e_coco.yml    # RT-DETRv2 configuration adapted to EO19
    environment.txt

  deim_official/
    README.md
    deim_hgnetv2_m_coco.yml             # DEIMv1 configuration adapted to EO19
    environment.txt

  dfine_official/
    README.md
    dfine_hgnetv2_m_coco.yml            # D-FINE-M configuration adapted to EO19
    dfine_hgnetv2_l_coco.yml            # D-FINE-L configuration adapted to EO19
    environment.txt

  ultralytics_yolo/
    README.md
    eo19.yaml                           # YOLO dataset configuration
    train_yolov8n.sh
    train_yolo11n.sh
    train_yolov12n.sh
    train_yolov13n.sh
    environment.txt
```

Each subfolder corresponds to one upstream implementation. The adapter files mainly modify dataset paths, category numbers, input settings, training schedules, optimizer settings, and evaluation settings required for EO19.

If the actual released filenames differ from the placeholders above, the repository structure should use the real filenames that appear in the released repository.

---

## Configuration Files vs. Trained Checkpoints

Configuration files and trained model weights are different.

Configuration files:

```text
.py
.yml
.yaml
```

These files define dataset paths, category numbers, model architecture, training schedules, optimizer settings, input sizes, and evaluation settings.

Trained checkpoints:

```text
.pth
.pt
.ckpt
.safetensors
```

These files contain learned model parameters.

At the current preview stage, trained checkpoints are withheld pending publication. If released later, they should be provided separately, for example:

```text
checkpoints/

  co-detr/
    co_detr_r50_eo19.pth

  co-dino/
    co_dino_vit_l_5scale_eo19.pth

  rtdetrv2/
    rtdetrv2_r18vd_sp3_120e_eo19.pth

  deim/
    deim_hgnetv2_m_eo19.pth

  dfine/
    dfine_hgnetv2_m_eo19.pth
    dfine_hgnetv2_l_eo19.pth

  yolo/
    yolov8n_eo19.pt
    yolo11n_eo19.pt
    yolov12n_eo19.pt
    yolov13n_eo19.pt
```

---

## Eigen-CAM Visualizations

Eigen-CAM is used to inspect detector attention. It is gradient-free and tends to produce stable activation maps for small-object scenarios. All samples are from **EO19-Original**.

### DETR-family Models

<table width="100%">
  <tr>
    <th align="center">Input</th><th align="center">Co-DINO</th><th align="center">Co-DETR</th><th align="center">RT-DETR</th><th align="center">D-FINE-L</th><th align="center">DEIMv1</th>
  </tr>
  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/63ed81de-4d4f-481f-8aa8-18f1fd4f8eb7" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/704c36d4-9d54-4540-9e28-df9b461f4046" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/4e7b2f38-c06f-439a-a966-5b36c1e75f78" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/26b0ed55-c927-4888-8612-055a20075209" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/bfd2cf75-301f-493d-9adc-736a5c114ab4" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/0aa783c8-3068-4512-bfda-dcc69919cdb2" width="100%"/></td>
  </tr>
  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/c0f23b81-6b42-432c-9fc9-7fc99c6cb75c" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/76a56231-7bb9-4f36-a860-bf096cd5dd91" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/7398aa5b-a3bf-4b80-91df-0f4da75aa270" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/ca109e1f-aba7-44be-8f81-19ed6f0d086e" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/8adab51b-633f-468d-a5f5-dda02d024e55" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/ea289851-a18b-4265-94e2-eef8ccffb52e" width="100%"/></td>
  </tr>
  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/eba5deb5-f5fa-441c-a15b-19b7d7797f18" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/b363a23a-074f-4088-9510-1c8a42f25fad" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/f352f7fd-573a-47af-8e7e-e95e40d51763" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/913d2f23-4613-420c-91b9-c7cafc6c5d70" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/830ae023-94d9-4578-bc73-dfc615982ae2" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/8b8be1ed-d469-4d75-9814-7b08a721f1d4" width="100%"/></td>
  </tr>
  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/f9832c66-60ee-4338-a968-2977d2a0bd9f" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/526c660d-8db4-431a-a486-252fa96f5f51" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/8e2f2624-726f-4d1b-999a-48ee9394f758" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/f1c51f8f-d4d9-490a-8b38-2b0e66769480" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/e67e858b-4d19-4daa-bf41-39028385a60e" width="100%"/></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/10a1a634-666a-4133-bf89-da480a86d164" width="100%"/></td>
  </tr>
</table>

### YOLO-family Models

<table width="100%">
  <tr>
    <th width="20%" align="center">Input</th>
    <th width="20%" align="center">YOLO11n</th>
    <th width="20%" align="center">YOLOv8n</th>
    <th width="20%" align="center">YOLOv12n</th>
    <th width="20%" align="center">YOLOv13n</th>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/eba5deb5-f5fa-441c-a15b-19b7d7797f18" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/73fdb178-1f3e-4ab0-a979-5da3086807c9" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/9f2f9029-b28d-4743-bcf4-e875b01d2464" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/bbfe4ff3-3d8f-4fea-8570-6a6ca018833d" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/785e4852-ecbd-4a06-8008-68730eecf766" width="100%" /></td>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/63ed81de-4d4f-481f-8aa8-18f1fd4f8eb7" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/cf8e079e-ab7a-40fc-988a-78575067a061" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/469baba1-31ea-4ab3-ab35-6e40dfcfa301" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/61eb1daa-de30-4ecd-afdc-ca9a836e195b" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/adaeca80-4f9e-46d1-81c1-7ca9fb1f92b7" width="100%" /></td>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/f9832c66-60ee-4338-a968-2977d2a0bd9f" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/f2b73a9b-842e-4225-a5b0-832033b4e6bb" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/365cd1e2-d4d0-4deb-8cfb-e421d8de7b05" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/fe6ec5a2-7780-4272-838f-f2a6de76962f" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/08378aee-bfdf-4d35-9a32-b7244ee9cdbf" width="100%" /></td>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/c0f23b81-6b42-432c-9fc9-7fc99c6cb75c" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/aa637e1b-4a38-4fe7-93ae-8b5701fd3722" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/45c89be9-d6dc-45b0-91c8-bf8e44bb6b57" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/8f574757-1003-4393-919f-e336f3bf496a" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/5aa38ae3-5039-4cad-9ed2-2b77a28ebef6" width="100%" /></td>
  </tr>
</table>

---

## Raw vs Balanced Taxonomy Experiment

EO19's **life-stage-aware taxonomy** was compared against a traditional no-split taxonomy using **YOLOv12n**. Each experiment was repeated three times and averaged.

### Round 1 — EO19-Original

| Group | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| Experimental, life-stage-aware taxonomy | 0.881 | 0.812 | 0.869 | **0.675** |
| Control, no life-stage split | 0.894 | 0.837 | **0.898** | 0.670 |

### Round 2 — EO19-Augmented, 1,000 images per category

| Group | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| Experimental, life-stage-aware taxonomy | **0.920** | **0.898** | **0.940** | **0.734** |
| Control, no life-stage split | 0.874 | 0.821 | 0.881 | 0.660 |

Under balanced conditions, the life-stage-aware taxonomy yields clear gains across all metrics, indicating that the biological separation of adult and larval stages becomes more effective after long-tail imbalance is mitigated.

---

## Download

Dataset files, EO19-adapted configuration files, trained checkpoints, checksums, and release details: **TBA**.

Configuration files such as `.py`, `.yml`, and `.yaml` files are used to reproduce training in the corresponding upstream repositories. They are not trained model weights. If trained checkpoints are released, they will be provided separately as `.pth` or `.pt` files.

---

## Citation

```bibtex
@misc{EO19_2026,
  title  = {EO19: A Family-Level, Life-Stage-Aware Insect Detection Dataset for Agricultural Pest Monitoring},
  author = {TODO},
  year   = {2026},
  note   = {Under review},
  url    = {TBA}
}
```

---

## License

Paper / Dataset / Code: **TODO**.

---

## Contact & Acknowledgements

- Email: BDing81@outlook.com
- Built on [IP102](https://github.com/xpwu95/IP102), which was screened and cleaned as a primary image source.
- We thank the agricultural entomology experts involved in annotation verification.
