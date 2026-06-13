# TOP-Net

> Task-oriented Prior Adapter of Vision Foundation Models for Optical-SAR Fusion Detection

[![Paper](https://img.shields.io/badge/Paper-%202026-blue)](#citation)
[![Task](https://img.shields.io/badge/Task-Optical--SAR%20Fusion%20Detection-green)](#)
[![Backbone](https://img.shields.io/badge/Backbone-DINOv3-orange)](#)
[![License](https://img.shields.io/badge/License-TBD-lightgrey)](#license)

This repository contains the official implementation of **TOP-Net**, a task-oriented prior adapter network for adapting vision foundation models to **Optical-SAR Fusion Detection (OSFD)**.

Vision foundation models provide strong general representations, but they are not naturally aware of cross-modal optical-SAR alignment, SAR physical imaging priors, or the extreme scale variation in remote sensing scenes. TOP-Net bridges this gap by injecting task-specific priors into a frozen VFM and performing balanced optical-SAR feature fusion.

## News
- The code will be made public once the paper is accepted (2026.06.13).
- Code, pretrained models, and scripts will be released in this repository (2026.06.08).

## Highlights

- **Task-oriented VFM adaptation.** TOP-Net adapts frozen vision foundation models to optical-SAR fusion detection with task-specific priors.
- **Physical and multi-scale priors.** The Task-oriented Prior Adapter (ToPA) models SAR scattering characteristics with PPEM and strengthens scale awareness with MKEM.
- **Reliable prior injection.** GFM adaptively balances generic VFM features and task-specific prior features to reduce harmful interference.
- **Balanced multimodal fusion.** MBFM performs deep fusion of heterogeneous optical and SAR features.
- **Strong performance.** TOP-Net achieves state-of-the-art results on OGSOD-1.0, OGSOD-2.0, and M4-SAR.

## Framework

![TOP-Net framework](https://github.com/wchao0601/TOP-Net/blob/main/network.png)

TOP-Net follows a two-stage design:

1. **Task-prior learning.** ToPA is pretrained on optical-SAR data to consolidate task-specific physical and spatial priors.
2. **VFM adaptation and fusion.** A frozen VFM extracts general optical/SAR features. ToPA priors are injected via GFM, then optical and SAR features are fused by MBFM and sent to an oriented bounding box detection head.

## Method
![Task-oriented Prior Adapter](https://github.com/wchao0601/TOP-Net/blob/main/adapter.png)
### Task-oriented Prior Adapter

**ToPA** serves as a task-specific knowledge carrier for OSFD. It contains:

- **PPEM: Physical Prior Enhancement Module**  
  Models SAR backscattering characteristics with ratio-of-averages style gradient priors to suppress multiplicative speckle noise.

- **MKEM: Multi-scale Knowledge Enhancement Module**  
  Decouples feature channels into heterogeneous branches for point, local, medium-range, and global attention, improving perception of objects with large-scale variation and irregular shapes.

### Gated Fusion Module

**GFM** adaptively injects task-specific priors into VFM features. This prevents physical/spatial priors from overwhelming the general representation learned by the foundation model.

### Modality-Balanced Fusion Module

**MBFM** learns to balance optical texture/color cues and SAR structural cues, producing discriminative fused features for oriented object detection.

## Main Results

### OGSOD-1.0 and OGSOD-2.0

| Method | Params | FLOPs | Time | OGSOD-1.0 mAP | OGSOD-2.0 mAP |
|---|:---:|:---:|:---:|:---:|:---:|
| E2E-OSDet | 23.332M | 10.080G | 21.3ms | 67.3 | 65.1 |
| ViT-Adapter-S | 36.316M | 52.752G | 70.8ms | 71.4 | 68.2 |
| BAT-S | 27.434M | 25.188G | 43.2ms | 69.9 | 66.5 |
| STA-S | 27.633M | 24.730G | 39.4ms | 68.6 | 67.0 |
| **TOP-Net-S** | **45.511M** | **33.926G** | **60.5ms** | **74.0** | **71.8** |
| **TOP-Net-B** | **118.800M** | **102.306G** | **61.1ms** | **74.6** | **72.3** |
| **TOP-Net-KD** | **23.332M** | **10.080G** | **21.4ms** | **73.0** | **71.0** |

### M4-SAR

| Method | Params | FLOPs | Time | AP50 | AP75 | mAP |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| MMIDet | 53.591M | 31.526G | 41.9ms | 74.9 | 68.6 | 59.8 |
| E2E-OSDet | 23.333M | 40.324G | 20.9ms | 77.7 | 70.3 | 61.4 |
| ViT-Adapter-S | 36.317M | 211.008G | 72.7ms | 78.3 | 70.8 | 61.6 |
| MoBA-S | 27.533M | 100.368G | 58.2ms | 77.8 | 69.6 | 61.0 |
| **TOP-Net-S** | **45.511M** | **134.740G** | **58.4ms** | **80.9** | **74.3** | **65.0** |
| **TOP-Net-B** | **118.800M** | **405.386G** | **59.2ms** | **82.7** | **76.0** | **66.5** |
| **TOP-Net-KD** | **23.333M** | **40.324G** | **21.1ms** | **82.0** | **74.5** | **65.5** |

## Qualitative Results

![Detection comparison](https://github.com/wchao0601/TOP-Net/blob/main/detect.png)

![Heatmap comparison](https://github.com/wchao0601/TOP-Net/blob/main/heatmap.png)

## Installation

```bash
git clone https://github.com/wchao0601/TOP-Net.git
cd TOP-Net

conda create -n topnet python=3.10 -y
conda activate topnet

pip install -r requirements.txt
```

## Dataset Preparation

Please organize the datasets as follows:

```text
datasets/
+-- OGSOD-1.0/
|   +-- rgb/
|       +-- images/
|           +-- train/
|           +-- test/
|       +-- labels/
|           +-- train/
|           +-- test/
|   +-- sar/
|       +-- images/
|           +-- train/
|           +-- test/
|       +-- labels/
|           +-- train/
|           +-- test/
+-- OGSOD-2.0/
+-- M4-SAR/
```

Supported benchmarks:

| Dataset | Resolution | Classes | Train / Val / Test |
|---|---:|---|:---:|
| OGSOD-1.0 | 256 x 256 | Bridge, Harbor, Oil-Tank | 14,665 / - / 3,666 |
| OGSOD-2.0 | 256 x 256 | Bridge, Harbor, Oil-Tank | 14,250 / 2,035 / 4,047 |
| M4-SAR | 512 x 512 | Bridge, Harbor, Oil-Tank, Playground, Airport, Wind-Turbine | 56,116 / 22,112 / 33,946 |

## Training

### Train TOP-Net

```bash
python tools/train.py
```

## Evaluation

```bash
python tools/test.py
```

## Model Zoo

| Model | Backbone | Dataset | AP50 | AP75 | mAP | Checkpoint |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| TOP-Net-S | DINOv3-Small | OGSOD-1.0 | 96.2 | 82.5 | 74.0 | Coming soon |
| TOP-Net-B | DINOv3-Base | OGSOD-1.0 | 96.4 | 83.2 | 74.6 | Coming soon |
| TOP-Net-S | DINOv3-Small | OGSOD-2.0 | 95.0 | 79.8 | 71.8 | Coming soon |
| TOP-Net-B | DINOv3-Base | OGSOD-2.0 | 95.0 | 80.8 | 72.3 | Coming soon |
| TOP-Net-S | DINOv3-Small | M4-SAR | 80.9 | 74.3 | 65.0 | Coming soon |
| TOP-Net-B | DINOv3-Base | M4-SAR | 82.7 | 76.0 | 66.5 | Coming soon |
| TOP-Net-KD | Lightweight student | M4-SAR | 82.0 | 74.5 | 65.5 | Coming soon |

## Citation

If this work is helpful for your research, please consider citing:

```bibtex
@article{wang2026topnet,
  title={TOP-Net: Task-oriented Prior Adapter of Vision Foundation Models for Optical-SAR Fusion Detection},
  author={Wang, Chao and Yu, Zhenbo and Sun, Yanguang and Yang, Jian and Luo, Lei},
  year={2026}
}
```

## Acknowledgements

This project builds on prior research in optical-SAR fusion detection, vision foundation models, parameter-efficient tuning, and oriented object detection. We thank the authors of the related open-source projects and datasets.

## License

The license will be released with the code.
