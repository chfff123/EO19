# EO19: A Large Dataset Construction for Insect Identification and Multi-Model Performance Assessment

> **Public Preview (Under Review)**  
> This repository is a **public preview** prepared for the EO19 paper.  
> Since the paper/dataset is **not yet published**, the following items are intentionally omitted from this public version:
> - Dataset download links / private storage  
> - Full author list & affiliations  
> - Trained checkpoints (`.pth`) and any private keys/tokens  
> - Machine-specific absolute paths and internal logs  
>
> Placeholders used in this README:
> - **TODO**: to be filled after publication (final info not ready yet)  
> - **TBA**: to be announced later  

- **Paper:** TBA (PDF / arXiv / project page)  
- **Dataset release:** TBA (may be annotations-only / partial release, depending on source licenses)  
- **Authors:** TODO  

---

## Contents
- [Introduction](#introduction)
- [Dataset Overview](#dataset-overview)
- [Taxonomy & Label Space](#taxonomy--label-space)
- [Image Collection](#image-collection)
- [Rename Rule](#rename-rule)
- [Annotations & Formats](#annotations--formats)
- [Quick Start](#quick-start)
- [Requirements (Software)](#requirements-software)
- [Pretrained Models](#pretrained-models)
- [Preparation for Testing](#preparation-for-testing)
- [Model Zoo & Results (3-run Average)](#model-zoo--results-3-run-average)
  - [Co-DINO (ViT-Large, 5-scale)](#co-dino-vit-large-5-scale)
  - [RT-DETR (ResNet-18)](#rt-detr-resnet-18)
  - [Co-DETR (ResNet-50)](#co-detr-resnet-50)
  - [DEIM v1 (HGNetv2)](#deim-v1-hgnetv2)
  - [D-FINE Large (HGNetv2)](#d-fine-large-hgnetv2)
  - [D-FINE Medium (HGNetv2)](#d-fine-medium-hgnetv2)
  - [YOLOv12](#yolov12-experimental)
- [Gradcam](#gradcam)
- [Download](#download)
- [Citation](#citation)
- [License](#license)
- [Contact](#contact)
- [Acknowledgements](#acknowledgements)

---

## Introduction
EO19 is an insect detection dataset designed for agricultural pest monitoring.  
It organizes categories at the **family** level (Latin names), and introduces a **life-stage-aware labeling rule**: for holometabolous families, **Adult** and **Larva** are annotated as **two separate categories** to reduce intra-class appearance conflict in training.

---

## Dataset Overview
- **Scope:** Insecta → **4 orders**, **19 families**, **30 categories**
- **Images:** **24,626** expert-validated images
- **Labeling principle:** family-level taxonomy with **life-stage-aware split** for holometabolous insects
- **Annotation formats:** Pascal VOC (`.xml`), YOLO (`.txt`), COCO-style JSON (`.json`)
- **Split ratio:** **train : val : test = 8 : 1 : 1** (stratified within each category)

### EO19 design rationale (why this taxonomy?)
EO19 is designed for **practical agricultural pest monitoring**, where family-level identification is often more actionable than species-level labels (e.g., pesticide decisions are commonly made at family/group level in practice).

To reduce visual ambiguity during training:
- **Holometabolous families** are split into `Family_Adult` and `Family_Larva`
- **Hemimetabolous families** remain a single category (nymph/adult morphology is relatively similar)

This yields:
- **11 holometabolous families × 2 = 22 categories**
- **8 hemimetabolous families × 1 = 8 categories**
- **Total = 30 categories**

---

## Taxonomy & Label Space

### Family list used in EO19 (19 families)

#### Complete metamorphosis (Holometabolous, split into Adult/Larva)
- Crambidae
- Noctuidae
- Limacodidae
- Nymphalidae
- Papilionidae
- Sphingidae
- Scarabaeidae
- Elateridae
- Coccinellidae
- Cerambycidae
- Chrysomelidae

#### Incomplete metamorphosis (Hemimetabolous, single category)
- Fulgoridae
- Flatidae
- Delphacidae
- Cicadellidae
- Miridae
- Aphididae
- Gryllotalpidae
- Acrididae

### Order coverage (4 orders)
EO19 families mainly span the following insect orders:
- **Lepidoptera**
- **Coleoptera**
- **Hemiptera**
- **Orthoptera**

---

## Image Collection
EO19 images were collected from two sources and then cleaned using a unified screening protocol:

### Source A: Existing datasets (major source: IP102)
EO19 uses IP102 as a major source for initial collection and category alignment.
The paper reports that approximately **10,000** images were obtained from IP102 after screening and relabeling checks.

### Source B: Public web images
Additional images were collected from public web sources using multiple query strategies:
- Latin family names
- English common names
- Chinese common names
- Colloquial names
- Representative species names (within target families)

After the same filtering strategy, the paper reports approximately **14,000** images collected from public sources.

### Four image styles intentionally included
EO19 intentionally preserves diversity in image style and acquisition conditions:
1. **Natural habitat scenes** (small insects + complex backgrounds)
2. **Field close-up images** (clearer subject, less background)
3. **Specimen images** (clean background, morphology-focused)
4. **A small number of AI-generated images** (for a few visually distinctive families, used as an exploratory supplement)

> Note: The AI-generated subset is intentionally small and used only as an exploratory data source for underrepresented categories.

---

## Rename Rule
Unified filename rule:
- Holometabolous families:
  - Adult: `A_<FamilyLatin>_<Index>.jpg`
  - Larva: `L_<FamilyLatin>_<Index>.jpg`
- Hemimetabolous families:
  - `<FamilyLatin>_<Index>.jpg`

Examples:
- `A_Cerambycidae_0001.jpg`
- `L_Cerambycidae_0001.jpg`
- `Acrididae_0008.jpg`

---

## Annotations & Formats
- Annotation tool: LabelImg
- Bounding boxes: tightly cover entire insect body
- Primary format: Pascal VOC XML
- Export formats: YOLO (`.txt`), COCO (`.json`)

---

## Quick Start

### 1) Prepare the dataset (COCO format)
```text
EO19/
  images/
    train/
    val/
    test/
  annotations/
    eo19_train.json
    eo19_val.json
    eo19_test.json
```

Set:
```bash
export EO19_ROOT=/path/to/EO19
```

### 2) Ensure class count matches
EO19 uses **30 categories**. Make sure `num_classes = 30` in your config.

### 3) Run evaluation
This repo reports **COCO-style metrics** on the EO19 validation split.  
Because different baselines use different codebases, please use each baseline’s native eval script and only change:
- dataset path (`data_root` / `${EO19_ROOT}`)
- COCO json (`ann_file`)
- image folder (`img_prefix`)
- class count (`num_classes`)

---

## Requirements (Software)

> Notes:
> - Different baselines may use different original frameworks; pin dependencies per model.
> - Versions below reflect one verified setup; they are not the only working combinations.

| Model | Python | PyTorch / TorchVision | CUDA / cuDNN | Framework / Key deps |
|---|---|---|---|---|
| Co-DINO (ViT-L, 5-scale) | 3.7.12 | 1.11.0 / 0.12.0 | 11.3 / 8.2 | OpenCV 4.x, MMCV 1.5.0, MMDetection 2.25.3 |
| Co-DETR (R50) | 3.7.12 | 1.11.0 / 0.12.0 | 11.3 / 8.2 | OpenCV 4.x, MMCV 1.5.0, MMDetection 2.25.3 |
| RT-DETR (ResNet-18) | 3.7.12 | 2.4.1 | 12.1 / 9.1.2 | Project native deps |
| DEIM v1 (HGNetv2) | 3.7.12 | 2.0.1 / 0.15.2 | 12.1 / 9.1.2 | faster-coco-eval, PyYAML, TensorBoard, SciPy, calflops, Transformers |
| D-FINE (M/L) (HGNetv2) | 3.11.9 | 2.1.2 / 0.16.2 | 12.1 / 9.1.2 | faster-coco-eval, PyYAML, tensorboard, scipy, calflops, transformers, loguru |

---

## Pretrained Models
This **public preview** does **not** include trained checkpoints (`.pth`).  
Checkpoint download links and SHA256 will be provided after publication (TBA).

Recommended layout (local only):
```text
pretrained/
  codino_vitl_5scale.pth
  codetr_r50.pth
  rtdetr_r18.pth
  deim_v1_hgnetv2.pth
  dfine_m_hgnetv2.pth
  dfine_l_hgnetv2.pth
```

| Model | Checkpoint (local path) | Download | SHA256 |
|---|---|---|---|
| Co-DINO (ViT-L, 5-scale) | `pretrained/codino_vitl_5scale.pth` | TBA | TBA |
| RT-DETR (ResNet-18) | `pretrained/rtdetr_r18.pth` | TBA | TBA |
| Co-DETR (R50) | `pretrained/codetr_r50.pth` | TBA | TBA |
| DEIM v1 (HGNetv2) | `pretrained/deim_v1_hgnetv2.pth` | TBA | TBA |
| D-FINE Medium (HGNetv2) | `pretrained/dfine_m_hgnetv2.pth` | TBA | TBA |
| D-FINE Large (HGNetv2) | `pretrained/dfine_l_hgnetv2.pth` | TBA | TBA |

---

## Preparation for Testing

### 1) Dataset structure (COCO format)
```text
EO19/
  images/
    train/
    val/
    test/
  annotations/
    eo19_train.json
    eo19_val.json
    eo19_test.json
```

### 2) Ensure class count matches
EO19 uses **30 categories**. Make sure `num_classes = 30` in your model config.

### 3) Point configs to EO19 paths
Examples (edit to your framework):
- `data_root = ${EO19_ROOT}`
- `ann_file = annotations/eo19_val.json`
- `img_prefix = images/val/`

### 4) Example commands (single-line)
Co-DINO (MMDetection-style):
```bash
python tools/train.py projects/configs/co_dino_vit/co_dino_5scale_vit_large_coco.py --work-dir /path/to/output/codino --launcher none --cfg-options data_root=${EO19_ROOT} data.samples_per_gpu=1 model.backbone.img_size=1024 load_from=/path/to/pretrained_or_resume.pth
```

Co-DETR (MMDetection-style):
```bash
python tools/train.py projects/configs/co_deformable_detr/co_deformable_detr_r50_1x_coco.py --work-dir /path/to/output/codetr --launcher none --cfg-options data_root=${EO19_ROOT}
```

RT-DETR:
```bash
python -u tools/train.py -c configs/rtdetrv2/rtdetrv2_r18vd_sp3_120e_coco.yml --output-dir /path/to/output/rtdetr -u print_freq=10
```

DEIM:
```bash
CUDA_VISIBLE_DEVICES=0 python train.py -c configs/deim_dfine/deim_hgnetv2_m_coco.yml --use-amp --seed=0
```

D-FINE:
```bash
CUDA_VISIBLE_DEVICES=0 python train.py -c configs/dfine/dfine_hgnetv2_l_coco.yml --use-amp --seed=0
```

---

## Model Zoo & Results (3-run Average)
All results are COCO-style metrics (AP, AP50, AP75, AP_S/M/L) on the same EO19 validation split.  
Numbers are in **[0, 1]** scale.

### Leaderboard (avg of 3 runs)
| Model | Backbone | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---|---|---:|---:|---:|---:|---:|---:|
| D-FINE Large | HGNetv2 | 0.701 | 0.900 | 0.762 | 0.321 | 0.534 | 0.799 |
| D-FINE Medium | HGNetv2 | 0.693 | 0.885 | 0.756 | 0.282 | 0.524 | 0.792 |
| DEIM v1 | HGNetv2 | 0.692 | 0.887 | 0.754 | 0.309 | 0.545 | 0.794 |
| RT-DETR | ResNet-18 | 0.666 | 0.866 | 0.729 | 0.277 | 0.492 | 0.770 |
| Co-DETR | ResNet-50 | 0.611 | 0.814 | 0.668 | 0.218 | 0.408 | 0.720 |
| Co-DINO | ViT-L (5-scale) | 0.733 | 0.928 | 0.798 | 0.411 | 0.580 | 0.817 |					
| Yolov12 | \ | 0.675 | 0.869 | \ | \ | \ | \ |					


---

## Co-DINO (ViT-Large, 5-scale)
| Run | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 0.734 | 0.932 | 0.799 | 0.428 | 0.596 | 0.800 |
| 2 | 0.733 | 0.928 | 0.798 | 0.406 | 0.578 | 0.825 |
| 3 | 0.731 | 0.925 | 0.798 | 0.398 | 0.566 | 0.825 |
| **avg** | 0.733 | 0.928 | 0.798 | 0.411 | 0.580 | 0.817 |

---

## RT-DETR (ResNet-18)
| Run | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 0.662 | 0.858 | 0.723 | 0.275 | 0.480 | 0.770 |
| 2 | 0.668 | 0.870 | 0.727 | 0.290 | 0.490 | 0.768 |
| 3 | 0.668 | 0.869 | 0.736 | 0.266 | 0.505 | 0.772 |
| **avg** | **0.666** | **0.866** | **0.729** | **0.277** | **0.492** | **0.770** |

---

## Co-DETR (ResNet-50)
| Run | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 0.612 | 0.815 | 0.667 | 0.226 | 0.411 | 0.718 |
| 2 | 0.617 | 0.819 | 0.678 | 0.235 | 0.380 | 0.727 |
| 3 | 0.605 | 0.807 | 0.660 | 0.194 | 0.433 | 0.714 |
| **avg** | **0.611** | **0.814** | **0.668** | **0.218** | **0.408** | **0.720** |

---

## DEIM v1 (HGNetv2)
| Run | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 0.692 | 0.889 | 0.755 | 0.297 | 0.542 | 0.794 |
| 2 | 0.691 | 0.884 | 0.752 | 0.305 | 0.549 | 0.793 |
| 3 | 0.692 | 0.888 | 0.755 | 0.325 | 0.545 | 0.795 |
| **avg** | **0.692** | **0.887** | **0.754** | **0.309** | **0.545** | **0.794** |

---

## D-FINE Large (HGNetv2)
| Run | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 0.701 | 0.899 | 0.763 | 0.331 | 0.528 | 0.798 |
| 2 | 0.697 | 0.894 | 0.758 | 0.309 | 0.544 | 0.797 |
| 3 | 0.704 | 0.906 | 0.765 | 0.323 | 0.529 | 0.801 |
| **avg** | **0.701** | **0.900** | **0.762** | **0.321** | **0.534** | **0.799** |

---

## D-FINE Medium (HGNetv2)
| Run | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 0.691 | 0.887 | 0.755 | 0.292 | 0.538 | 0.791 |
| 2 | 0.698 | 0.883 | 0.756 | 0.279 | 0.513 | 0.794 |
| 3 | 0.690 | 0.885 | 0.756 | 0.276 | 0.521 | 0.792 |
| **avg** | **0.693** | **0.885** | **0.756** | **0.282** | **0.524** | **0.792** |

---

## YOLOv12 (Experimental)

> YOLO-style metrics reported by the framework: **P**, **R**, **mAP@0.50**, **mAP@0.50:0.95**.  
> For consistency with the rest of this README:
> - **AP = mAP@0.50:0.95**
> - **AP50 = mAP@0.50**
> - **AP75 / AP_S / AP_M / AP_L** are **TBA** (not provided by current YOLO output)

### Results (3 runs)
| Group | Images | Instances | P | R | mAP@0.50 | mAP@0.50:0.95 |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 2462 | 5394 | 0.883 | 0.809 | 0.872 | 0.677 |
| 2 | 2462 | 5394 | 0.880 | 0.817 | 0.870 | 0.675 |
| 3 | 2462 | 5394 | 0.880 | 0.809 | 0.866 | 0.674 |
| **avg** | 2462 | 5394 | **0.881** | **0.812** | **0.869** | **0.675** |

---

## Gradcam
<p>
  <img src="https://github.com/user-attachments/assets/720eb01a-7370-4afa-9ae0-c3a35d7ef3ee" width="45%" />
  <img src="https://github.com/user-attachments/assets/b9fae2d4-4bfa-493c-b650-876a3388758c" width="45%" />
</p>

<p>
  <img src="https://github.com/user-attachments/assets/2b812fb6-4a1c-42bf-a0d2-773c49f43738" width="45%" />
  <img src="https://github.com/user-attachments/assets/36e18aea-68d2-4766-ae48-913e472e667e" width="45%" />
</p>

<p>
  <img src="https://github.com/user-attachments/assets/5f952303-8036-4663-99d1-9216aacb0b6e" width="45%" />
  <img src="https://github.com/user-attachments/assets/c667cab8-13a1-42d5-959a-7d30dea0911e" width="45%" />
</p>

---

## Download
- Dataset: TBA
- Checksums: TBA
- Release form: TBA (will comply with original source licenses and publication policy)

---

## Citation
```bibtex
@misc{EO19_2026,
  title   = {EO19: A Family-Level, Life-Stage-Aware Insect Detection Dataset for Agricultural Pest Monitoring},
  author  = {TODO},
  year    = {2026},
  note    = {Technical report / paper under review},
  url     = {TBA}
}
```

---

## License
- Paper/text: TODO
- Dataset: TODO (e.g., CC BY-NC 4.0 / research-only, subject to source licenses)
- Code: TODO (e.g., Apache-2.0 / MIT)

---

## Contact
- GitHub: https://github.com/chfff123
- Email: TODO

---

## Acknowledgements
- IP102 dataset (screened and cleaned as a major image source)
- Thanks to agriculture experts for verification and label auditing
