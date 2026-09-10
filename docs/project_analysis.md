# Project Analysis: Post-Hoc Hallucination Detector for SAR-to-Optical Translation

## till 10th sept/2026 5:00 pm

## What Has Been Built — The Full Pipeline 

Your project has successfully implemented all 4 stages of the proposed hallucination-detection pipeline, and has **exceeded the original scope** by training 3 generators instead of 1.

```mermaid
graph LR
    A["Stage 0\nData Prep"] --> B["Stage 1\nGenerator Training"]
    B --> C["Stage 1.5\nFake Image Generation"]
    C --> D["Stage 2\nGround-Truth Labels"]
    D --> E["Stage 3\nDeepLab Judge Training"]
    E --> F["Stage 4\nHallucination Benchmark"]

    style A fill:#4CAF50,color:white
    style B fill:#4CAF50,color:white
    style C fill:#4CAF50,color:white
    style D fill:#4CAF50,color:white
    style E fill:#4CAF50,color:white
    style F fill:#FF9800,color:white
```

---

## Inventory of Completed Work

### ✅ Stage 0 — Data Preparation
| Script | Status | Purpose |
|--------|--------|---------|
| [autorun.sh](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/autorun.sh) | ✅ Done | Downloads all 4 seasons of SEN12MS (~180GB) via rsync |
| [00_prepare_splits.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/00_prepare_splits.py) | ✅ Done | 80/10/10 geographic ROI-based train/val/test split (seed=42) |
| [dataset.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/dataset.py) | ✅ Done | Shared `SEN12MSDataset` with SAR dB normalization and optical RGB extraction |

Split sizes are stored in JSON:
- `splits/train_files.json` (~9.7 MB → ~144K patches)
- `splits/val_files.json` (~1.3 MB → ~18K patches)
- `splits/test_files.json` (~1.3 MB → ~18K patches)

---

### ✅ Stage 1 — Generator Training (3 models, all 30 epochs)

| Model | Script | Architecture | Key Design Choice |
|-------|--------|-------------|-------------------|
| **Pix2Pix** | [01b_train_pix2pix.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/01b_train_pix2pix.py) | U-Net Generator + PatchGAN Discriminator | Paired supervision with L1 pixel loss (λ=100) |
| **CycleGAN** | [01c_train_cyclegan.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/01c_train_cyclegan.py) | ResNet-9 Generator + PatchGAN (×2 each) | Unpaired training with cycle consistency (λ=10) |
| **Palette** | [01d_train_palette.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/01d_train_palette.py) | Conditional UNet with self-attention | DDPM diffusion (T=1000), DDIM sampling, CFG (scale=2.0) |

The [models.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/models.py) file contains the shared Pix2Pix `GeneratorUNet` and `Discriminator`. CycleGAN and Palette define their architectures inline.

Palette training log shows 30 epochs completed (Sep 4–6), best MSE loss = **0.001093** at epoch 27.

---

### ✅ Stage 1.5 — Fake Image Generation

| Script | Status | Purpose |
|--------|--------|---------|
| [generate_new_fakes.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/generate_new_fakes.py) | ✅ Done | Universal inference script supporting all 3 models |

Generates fake optical GeoTIFFs for each test-set SAR patch. Output dirs (gitignored): `fake_optical_test_set_pix2pix/`, `fake_optical_test_set_cyclegan/`, `fake_optical_test_set_palette/`.

Features: resume-safe (skips existing), NaN detection/cleanup, multithreaded GeoTIFF writing, DDIM sampling for Palette.

---

### ✅ Stage 2 — Ground-Truth Label Generation (WorldCover 10m)

> [!IMPORTANT]
> The project **pivoted from MODIS IGBP (500m, 17 classes) to ESA WorldCover (10m, 11 classes)** — a major improvement to the original proposal. Legacy MODIS scripts were removed.

| Script | Status | Purpose |
|--------|--------|---------|
| [05a_download_worldcover.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/05a_download_worldcover.py) | ✅ Done | Downloads required ESA WorldCover 3°×3° tiles from AWS S3 |
| [05b_generate_worldcover_labels.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/05b_generate_worldcover_labels.py) | ✅ Done | Reprojects WorldCover into exact SEN12MS patch grids, saves `_wc_` label files |

The 11 WorldCover classes: Tree Cover, Shrubland, Grassland, Cropland, Urban/Built-up, Barren, Snow/Ice, Water Bodies, Wetlands, Mangroves, Moss/Lichen.

---

### ✅ Stage 3 — DeepLab Judge Training

| Script | Status | Purpose |
|--------|--------|---------|
| [02b_train_deeplab_worldcover.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/02b_train_deeplab_worldcover.py) | ✅ Done | Fine-tunes DeepLabV3-ResNet50 on real optical images with WorldCover labels (15 epochs) |

Saves `deeplabv3_finetuned_worldcover.pth`. Uses `ignore_index=255` for unmapped pixels.

---

### ✅ Stage 4 — Hallucination Benchmark Evaluation

| Script | Status | Purpose |
|--------|--------|---------|
| [03b_evaluate_hallucinations_worldcover.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/03b_evaluate_hallucinations_worldcover.py) | ✅ Done | Double-condition filter + 2-pixel boundary erosion |

Implements the core methodology:
1. DeepLab segments **real** optical → `pred_real`
2. DeepLab segments **fake** optical → `pred_fake`
3. Ground truth from WorldCover → `gt_mask`
4. **Hallucination** = `(pred_real == gt) AND (pred_fake != gt) AND valid_mask`

---

## Current Results

### Pix2Pix Hallucination Benchmark

| Class | Detector Accuracy | Hallucination Rate |
|-------|:-:|:-:|
| 🌲 Tree Cover | 95.67% | **2.80%** |
| 🌊 Water Bodies | 92.19% | **3.42%** |
| 🏙️ Urban / Built-up | 97.04% | **26.03%** |
| 🏜️ Barren | 58.17% | **40.03%** |
| 🌾 Grassland | 52.96% | **64.24%** |
| 🌾 Cropland | 81.88% | **73.22%** |
| 🌿 Shrubland | 36.40% | **87.89%** |
| 💧 Wetlands | 36.17% | **100.00%** |
| 🌴 Mangroves | 0.01% | **100.00%** |
| ❄️ Snow/Ice | N/A | N/A |
| 🪨 Moss/Lichen | N/A | N/A |

### Palette (Diffusion) Hallucination Benchmark

| Class | Detector Accuracy | Hallucination Rate |
|-------|:-:|:-:|
| 🌊 Water Bodies | 92.18% | **0.59%** |
| 🏜️ Barren | 58.61% | **99.45%** |
| Everything else | — | **~100.00%** |

> [!WARNING]
> **The Palette results are almost certainly invalid.** Nearly 100% hallucination across ALL land cover classes (except Water) is not a genuine finding — it indicates that Palette's fake images are so out-of-distribution for the DeepLab judge that the segmentation fails entirely. This is flagged in the commit message `e692194` ("incorrect pallete run needs fix"). The Palette normalization pipeline may be mismatched between training/inference and the evaluation script.

### CycleGAN Benchmark
**Not yet produced.** No `hallucination_benchmark_worldcover_cyclegan.csv` exists, nor any CycleGAN evaluation log.

---

## Supporting Scripts

| Script | Purpose |
|--------|---------|
| [plot_training_curve.py](file:///Users/ananyakarn/Desktop/dev_ananya/Post-Hoc-Hallucination-Detector-for-SAR-to-Optical-Image-Translation_2/plot_training_curve.py) | Parses training logs → convergence curve PNGs for all 3 models |

Visual artifacts in the repo root: `pix2pix_sample.png`, `palette_*.png`, `real_ground_truth.png` — appear to be manual diagnostic screenshots.

---

## What Needs To Be Done Next

### 🔴 Critical (Blocking Capstone Deliverables)

#### 1. Fix the Palette Hallucination Benchmark
The ~100% hallucination rate indicates a **normalization mismatch**. The Palette model trains in `[-1.0, 1.0]` space and `generate_new_fakes.py` maps output back to `[0.0, 1.0]` reflectance (saved as `uint16 × 10000`). But the evaluation script's `read_rgb()` function has two paths:
```python
if img.dtype != np.uint8:
    img = img.astype(np.float32) / 10000.0  # ← expects uint16 reflectance
else:
    img = img.astype(np.float32) / 255.0    # ← expects uint8
```
**Verify** that the Palette fakes are being saved and read back correctly — inspect a few `.tif` files from `fake_optical_test_set_palette/` to check value ranges and data types.

#### 2. Generate & Evaluate CycleGAN Benchmark
Run:
```bash
python3 generate_new_fakes.py --model cyclegan
python3 03b_evaluate_hallucinations_worldcover.py --model cyclegan
```
This completes the 3-model comparison.

#### 3. Compute Standard Generator Metrics (PSNR, SSIM, FID)
The proposal's **Evaluation Plan** explicitly requires PSNR, SSIM, and FID to validate each generator as representative of state-of-the-art quality **before** the hallucination audit. No script for this exists yet. You'll need:
- **PSNR / SSIM**: `torchmetrics` or `skimage.metrics` over the test set (real vs. fake optical pairs)
- **FID**: `pytorch-fid` or `torchmetrics.image.fid` using real vs. fake optical distributions

---

### 🟡 Important (Capstone Report Quality)

#### 4. Manual Spot-Check Validation
The proposal requires *"manually reviewing a stratified sample of flagged hallucination pixels against human annotation."* Create a visualization script that:
- Overlays the hallucination mask (from the double-condition filter) on real/fake image pairs
- Randomly samples N patches per land-cover class
- Saves a visual grid for human review

#### 5. Statistical Validation Across Geographic Splits
The proposal mentions *"per-class rates reported alongside sample counts so low-frequency classes aren't over-interpreted."* Consider:
- Confidence intervals (bootstrap) for per-class hallucination rates
- Per-ROI hallucination variance to show geographic stability

#### 6. Comparative Analysis Table
Create a final side-by-side table: **Pix2Pix vs. CycleGAN vs. Palette** — per-class hallucination rates, PSNR, SSIM, FID, and DeepLab detector accuracy. This is the thesis centerpiece.

---

### 🟢 Nice-to-Have (Strengthens the Paper)

#### 7. Hallucination Heatmap Visualization
Generate spatial heatmaps showing where hallucinations concentrate geographically (e.g., coastal regions, mountain areas). Use patch GPS metadata.

#### 8. Cross-Model Confusion Matrices
For each generator, show which class gets hallucinated *into* which other class (e.g., Water → Land, Cropland → Grassland).

#### 9. Code Cleanup
- The CycleGAN and Palette architectures are **duplicated** across training scripts and `generate_new_fakes.py`. Extract them into shared modules.
- `autorun.sh` still references `python train.py` (line 33) which doesn't exist.
- The `dataset.py` original module is not used by any of the current training scripts — they all define their own inline `SAR2OptDataset`.

---

## Project Timeline Mapping

| Proposal Month | Milestone | Current Status |
|:-:|---------|:-:|
| M1 | Literature survey; SEN12MS download & preprocessing | ✅ Complete |
| M2 | Train pix2pix baseline; validate convergence | ✅ Complete (exceeded: trained 3 models) |
| M3 | Synthetic generation, double-condition filter, per-class table | ⚠️ Partially complete (Palette broken, CycleGAN missing, no PSNR/SSIM/FID) |
| M4 | Statistical validation; Capstone 1 report | 🔴 Not started |
