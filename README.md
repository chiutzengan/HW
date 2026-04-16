# MERT-based Residual BiGRU Beat Tracking

This project implements a high-performance beat tracking system using MERT semantic features and a memory-optimized BiGRU architecture.

## Highlights
- **Architecture**: Residual BiGRU with a linear projection layer for VRAM efficiency.
- **Features**: MERT-v1-330M (1024 dimensions, 75 FPS).
- **Optimization**: Dynamic 30s random cropping and gradient accumulation (steps=4).
- **Data Augmentation**: Time-stretching (0.82x, 1.18x) and Harmonic-Percussive Source Separation (HPSS).
- **Performance**: Achieved **0.85 F-Measure** on the overall evaluation.

## Installation
```bash
pip install torch transformers librosa numpy madmom tqdm pandas mirdata

python train2.py

python generate_prediction_json_new.py
