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
- [Dataset at a Glance](#dataset-at-a-glance)
- [Taxonomy & Label Space](#taxonomy--label-space)
- [Image Collection & Filtering](#image-collection--filtering)
- [Annotation Protocol](#annotation-protocol)
- [Data Split](#data-split)
- [Annotations & Formats](#annotations--formats)
- [Quick Start](#quick-start)
- [Baselines & Evaluation](#baselines--evaluation)
- [Model Zoo & Results](#model-zoo--results)
- [Eigen-CAM Visualizations](#eigen-cam-visualizations)
- [Reproducibility Checklist](#reproducibility-checklist)
- [Download](#download)
- [Citation](#citation)
- [License](#license)
- [Contact](#contact)
- [Acknowledgements](#acknowledgements)

---

## Introduction
EO19 is an insect detection dataset designed for agricultural pest monitoring.
It organizes categories at the **family** level (Latin names), and introduces a **life-stage-aware labeling rule**:
for holometabolous families, **Adult** and **Larva** are annotated as **two separate categories** to reduce intra-class appearance conflicts.

---

## Dataset at a Glance
- **Taxonomy:** Class Insecta → **4 orders**, **19 families**, **30 categories**
- **Images:** **24,626**
- **Life-stage split:** **11 holometabolous families** split into **Adult / Larva** (→ 22 categories)
- **Hemimetabolous:** **8 families** kept as single categories (→ 8 categories)
- **Annotations:** bounding-box detection labels
- **Formats:** VOC (XML) as the primary source, exported to YOLO (TXT) and COCO-like JSON (TBA for exact schema wording)
- **Baseline coverage:** DETR-family and YOLO-family detectors + interpretability via Eigen-CAM

> Note: Final dataset release form will follow original source licenses and publication policy (TBA).

---

## Taxonomy & Label Space

### Why “Family” and Latin names?
EO19 adopts **family-level** labels to reduce ambiguity and align with practical pest-management decisions.
Latin family names provide stable, standardized naming (common names can be ambiguous).

### Developmental-stage-aware labeling (EO19 rule)
- Complete metamorphosis (holometabolous): split into `Family_Adult` and `Family_Larva`
- Incomplete metamorphosis (hemimetabolous): keep one category `Family` (nymph + adult together)

### Families used in EO19 (19 total)
**Complete metamorphosis (11 families → 22 categories):**
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

**Incomplete metamorphosis (8 families → 8 categories):**
- Fulgoridae
- Flatidae
- Delphacidae
- Cicadellidae
- Miridae
- Aphididae
- Gryllotalpidae
- Acrididae

### Filename convention (recommended)
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

## Image Collection & Filtering
EO19 images come from:
1) screened/cleaned images from existing datasets
2) public web collection via Latin/English/Chinese/common names + species names (followed by the same screening strategy)

Typical EO19 image types include:
- natural habitat images (small instances, heavy background)
- high-definition field images (one/few insects, clearer subject)
- specimen images (clean backgrounds, strong morphological visibility)
- a **small number of AI-generated images** (used only for feasibility exploration of generative augmentation)

**Filtering principles (high level):**
- remove extremely low-resolution images
- remove watermark/occlusion-heavy images that hide key morphology
- remove category-mismatch / empty-sample images
- remove images with severe blur or stacked instances that are not labelable

---

## Annotation Protocol
EO19 uses **LabelImg** for bounding-box annotation.

Key protocol principles:
- annotators first learn the key morphological traits for assigned categories
- (recommended) one annotator per category to reduce cross-person inconsistency
- bounding boxes should fully cover the insect body while minimizing background inclusion
- during annotation, unqualified images are removed and replaced to keep category continuity
- final annotations are verified by agricultural entomology experts

---

## Data Split
- Split ratio: **train : val : test = 8 : 1 : 1**
- Split is performed **within each category** to preserve per-category proportions across subsets.

Suggested structure (COCO format):
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

---

## Annotations & Formats
- Primary annotation format: **Pascal VOC XML**
- Export formats: **YOLO TXT**, **COCO-like JSON**
- Conversion is done via scripts to ensure consistency across formats (TBA: scripts will be published after acceptance).

> Practical note: keep class ID mapping fixed once released. Any change in category ordering will break reproducibility.

---

## Quick Start

### 1) Set dataset root
```bash
export EO19_ROOT=/path/to/EO19
```

### 2) Ensure class count matches
EO19 uses **30 categories**.
Set `num_classes = 30` (or equivalent) in your config.

### 3) Evaluate using each framework’s native pipeline
Because baselines come from different codebases, use the native scripts and only modify:
- dataset root/path (`data_root` / `${EO19_ROOT}`)
- annotation file (`ann_file`)
- image folder (`img_prefix`)
- number of classes (`num_classes`)

---

## Baselines & Evaluation

### Models covered
**DETR-family (COCO AP-style metrics):**
- Co-DINO (ViT-Large, 5-scale)
- D-FINE-L / D-FINE-M (HGNetv2)
- DEIM v1 (HGNetv2)
- RT-DETR v2 (ResNet-18)
- Co-DETR (ResNet-50)

**YOLO-family (YOLO-style metrics):**
- YOLOv8n
- YOLO11n
- YOLOv12n
- YOLOv13n

### Metrics
- **DETR-family:** COCO-style **AP, AP50, AP75, AP_S/M/L**
- **YOLO-family:** **Precision, Recall, F1, mAP50, mAP50-95**
  - For consistency, we map:
    - `AP := mAP50-95`
    - `AP50 := mAP50`

### Two-round taxonomy comparison (paper ablation summary)
EO19’s life-stage-aware taxonomy generally improves recognition boundaries.
However, long-tailed distributions can mask this advantage; after controlling long-tail influence, the majority of categories show improved performance under the refined taxonomy (details in paper).

---

## Model Zoo & Results
All values are reported in **[0, 1]** scale on the EO19 validation split.
This public preview lists the headline numbers; detailed logs/configs are TBA.

### DETR-family (COCO AP metrics)
| Model | Backbone | AP | AP50 | AP75 | AP_S | AP_M | AP_L |
|---|---|---:|---:|---:|---:|---:|---:|
| **Co-DINO (ViT-L, 5-scale)** | ViT-L | 0.733 | 0.928 | 0.798 | 0.411 | 0.580 | 0.817 |
| D-FINE Large | HGNetv2 | 0.701 | 0.900 | 0.762 | 0.321 | 0.534 | 0.799 |
| D-FINE Medium | HGNetv2 | 0.693 | 0.885 | 0.756 | 0.282 | 0.524 | 0.792 |
| DEIM v1 | HGNetv2 | 0.692 | 0.887 | 0.754 | 0.309 | 0.545 | 0.794 |
| RT-DETR v2 | ResNet-18 | 0.666 | 0.866 | 0.729 | 0.277 | 0.492 | 0.770 |
| Co-DETR | ResNet-50 | 0.611 | 0.814 | 0.668 | 0.218 | 0.408 | 0.720 |

> Note: Some numbers may be updated in the camera-ready / errata release. Treat the final paper + official release as the source of truth.

### YOLO-family (YOLO metrics)
| Model | Precision | Recall | F1 | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|---:|
| YOLOv8n  | 0.807 | 0.740 | 0.772 | 0.806 | 0.608 |
| YOLO11n  | 0.847 | 0.736 | 0.788 | 0.813 | 0.612 |
| **YOLOv12n** | **0.881** | **0.812** | **0.845** | **0.869** | **0.675** |
| YOLOv13n | 0.879 | 0.811 | 0.844 | 0.869 | 0.674 |

---

## Eigen-CAM Visualizations
We use **Eigen-CAM** to qualitatively inspect where detectors attend on EO19 images.
Compared to gradient-based methods (Grad-CAM/Grad-CAM++), Eigen-CAM is:
- gradient-free → typically more stable on small objects
- captures global morphology cues (e.g., wing venation, dorsal plates, antennae, segmentation)
- tends to suppress background interference and is more consistent across architectures

### DETR result
<p>
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

## Reproducibility Checklist
If you want to reproduce the baseline numbers:
- [ ] Use the exact EO19 split (8:1:1 per-category stratified)
- [ ] Keep `num_classes = 30` and a fixed class ID mapping
- [ ] Run **3 seeds** (e.g., 0/1/2) and report mean (and std if possible)
- [ ] For YOLO-family, report `P/R/F1/mAP50/mAP50-95` directly from the framework
- [ ] For DETR-family, use COCO evaluation scripts producing `AP/AP50/AP75/AP_S/M/L`
- [ ] Record hardware + versions (CUDA/cuDNN/PyTorch) for transparency

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
