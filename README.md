# EO19: A Large-Scale Insect Detection Dataset and Multi-Model Performance Assessment

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
- [Data Split](#data-split)
- [Annotations & Formats](#annotations--formats)
- [Quick Start](#quick-start)
- [Baselines & Evaluation](#baselines--evaluation)
- [Model Zoo & Results](#model-zoo--results)
- [Eigen-CAM Visualizations](#eigen-cam-visualizations)
- [Download](#download)
- [Citation](#citation)
- [License](#license)
- [Contact](#contact)
- [Acknowledgements](#acknowledgements)

---

## Introduction
EO19 is an insect detection dataset designed for agricultural pest monitoring.

It organizes categories at the **family** level (Latin names) and introduces a **life-stage-aware labeling rule**:
for holometabolous families, **Adult** and **Larva** are annotated as **two separate categories** to reduce intra-class appearance conflicts.

### EO19 summary
- **Taxonomy:** Class Insecta → **4 orders**, **19 families**, **30 categories**
- **Images:** **24,626**
- **Life-stage split:** **11 holometabolous families** split into **Adult / Larva** (→ 22 categories)
- **Hemimetabolous:** **8 families** kept as single categories (→ 8 categories)
- **Annotations:** bounding-box detection labels
- **Formats:** Pascal VOC (XML, primary source), exported to YOLO (TXT) and COCO JSON
- **Baseline coverage:** DETR-family and YOLO-family detectors + interpretability via Eigen-CAM

EO19 images are collected from screened/cleaned public sources (including IP102 as a major source) and web collection using Latin/English/Chinese/common names and species names, followed by a unified filtering strategy. Final annotations are verified with agricultural entomology expertise.

> Note: Final dataset release form will follow original source licenses and publication policy (TBA).

---

## Data Split
- Split ratio: **train : val : test = 8 : 1 : 1**
- Split is performed **within each category** to preserve per-category proportions across subsets.

### COCO Format (JSON)
```text
EO19_JSON/
  images/
    train/
    val/
    test/
  annotations/
    train.json
    val.json
    test.json
```

### XML Format (Pascal VOC)
```text
EO19_XML/
  training/
    images/
    labels/
  val/
    images/
    labels/
  test/
    images/
    labels/
```

> In XML format, each image should have a corresponding annotation file in the `labels/` folder with the same filename stem.

### TXT Format (YOLO-style)
```text
EO19_TXT/
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

> In TXT format, each image should have a corresponding `.txt` annotation file in the `labels/` folder with the same filename stem.  
> Each line in a label file follows the YOLO format: `<class_id> <x_center> <y_center> <width> <height>` (normalized).

---

## Annotations & Formats
- Primary annotation format: **Pascal VOC XML**
- Export formats: **YOLO TXT**, **COCO JSON**
- Annotation tool: **LabelImg**
- Conversion is done via scripts to ensure consistency across formats (TBA: scripts will be published after acceptance)
- Final annotations are verified by agricultural entomology experts

> Practical note: keep class ID mapping fixed once released. Any change in category ordering will break reproducibility.

---
## Quick Start

### 1) Set dataset root
```bash
export EO19_ROOT=/path/to/EO19
```

### 2) Update framework configurations
Whether you are using DETR-based (e.g., MMDetection) or YOLO-based frameworks, please use their native evaluation pipelines. Keep all model-specific settings unchanged and **only modify the following dataset settings**:

- **Dataset Path:** Point to your dataset root (e.g., `data_root: ${EO19_ROOT}` for DETR, or `path: ${EO19_ROOT}` for YOLO).
- **Images & Annotations:** Update the paths to your image folders and annotation files (e.g., `img_prefix` & `ann_file` for DETR, or `train`/`val`/`test` split references for YOLO).
- **Class Configuration:** EO19 uses **30** categories. Ensure the class count is updated (e.g., `num_classes = 30` for DETR, or `nc: 30` for YOLO). Update class names if required by the framework.

> **Note:** For YOLO-specific details, refer to the [official Ultralytics documentation](https://docs.ultralytics.com/). For DETR-based baselines, see your specific model's config files.
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
| YOLOv11n | 0.847 | 0.736 | 0.788 | 0.813 | 0.612 |
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

<table width="100%">
  <tr>
    <th width="16%" align="center">Input</th>
    <th width="16%" align="center">Co-DINO</th>
    <th width="16%" align="center">Co-DETR</th>
    <th width="16%" align="center">RT-DETR</th>
    <th width="16%" align="center">D-FINE-L</th>
    <th width="16%" align="center">DEIM</th>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/63ed81de-4d4f-481f-8aa8-18f1fd4f8eb7" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/704c36d4-9d54-4540-9e28-df9b461f4046" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/4e7b2f38-c06f-439a-a966-5b36c1e75f78" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/26b0ed55-c927-4888-8612-055a20075209" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/bfd2cf75-301f-493d-9adc-736a5c114ab4" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/0aa783c8-3068-4512-bfda-dcc69919cdb2" width="100%" /></td>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/c0f23b81-6b42-432c-9fc9-7fc99c6cb75c" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/76a56231-7bb9-4f36-a860-bf096cd5dd91" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/7398aa5b-a3bf-4b80-91df-0f4da75aa270" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/ca109e1f-aba7-44be-8f81-19ed6f0d086e" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/8adab51b-633f-468d-a5f5-dda02d024e55" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/ea289851-a18b-4265-94e2-eef8ccffb52e" width="100%" /></td>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/eba5deb5-f5fa-441c-a15b-19b7d7797f18" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/b363a23a-074f-4088-9510-1c8a42f25fad" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/f352f7fd-573a-47af-8e7e-e95e40d51763" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/913d2f23-4613-420c-91b9-c7cafc6c5d70" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/830ae023-94d9-4578-bc73-dfc615982ae2" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/8b8be1ed-d469-4d75-9814-7b08a721f1d4" width="100%" /></td>
  </tr>

  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/f9832c66-60ee-4338-a968-2977d2a0bd9f" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/526c660d-8db4-431a-a486-252fa96f5f51" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/8e2f2624-726f-4d1b-999a-48ee9394f758" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/f1c51f8f-d4d9-490a-8b38-2b0e66769480" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/e67e858b-4d19-4daa-bf41-39028385a60e" width="100%" /></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/10a1a634-666a-4133-bf89-da480a86d164" width="100%" /></td>
  </tr>
</table>

### YOLO result

<table width="100%">
  <tr>
    <th width="20%" align="center">Input</th>
    <th width="20%" align="center">YOLOv11n</th>
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
