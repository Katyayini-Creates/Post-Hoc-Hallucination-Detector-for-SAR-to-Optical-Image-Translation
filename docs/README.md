# Quantifying Semantic Hallucination in SAR-to-Optical Image Translation: A Per-Land-Cover-Class Ground-Truth Benchmark

**B.Tech Capstone Research Proposal**
**Capstone 1: Analysis**
**Reference:** CAP-CSE-2025-SAR-01
**Issuing Centre:** School of Computer Science(SOCS), UPES
**Project Context:** Reliable AI-Translated Satellite Imagery for All-Weather Disaster Response and Defence Surveillance
**Research Area:** Computer Vision | Generative AI Safety | Remote Sensing
**Dataset:** SEN12MS (Schmitt et al., 2019) — 180,662 co-registered Sentinel-1/2 patch triplets, 17 IGBP land-cover classes, ~180 GB

**TEAM MEMBERS:**
1. Ananya Karn(500125205) CCVT B3
2. Hiten Gupta(500123599) DevOps B1
3. Katyayini Singh(500123234) CSF B4
4. Vyansh Sinha(500120123) DevOps B1

## Abstract
SAR-to-Optical translation converts all-weather radar imagery into human-intuitive optical-style images, but current GAN-based generators hallucinate semantic content — a bridge that does not exist, dry land over floodwater — with the same photorealistic confidence as genuine features. Standard metrics (SSIM, PSNR, FID) are global scores that are structurally blind to these localized failures, creating a false sense of security in high-stakes disaster-response and defence workflows. This research trains a baseline pix2pix SAR-to-Optical generator and, using a novel double-condition DeepLabV3 ground-truth filter, produces the first quantitative, per-land-cover-class hallucination benchmark — a reproducible, externally validated pipeline that isolates genuine generator errors from segmentation-model limitations and establishes a numerical reference point for evaluating semantic trustworthiness in SAR-to-Optical translation.

## 1. Introduction and Motivation
Sentinel-1 SAR imaging is unaffected by cloud cover, smoke, or darkness, but its backscatter physics — speckle, layover, foreshortening, radar shadow — make it unreadable without specialist training. Sentinel-2 optical imagery is intuitive but unavailable roughly two-thirds of the time due to cloud cover. GAN- and diffusion-based SAR-to-Optical translation promises to close this gap, but a generator faced with ambiguous input produces its most statistically plausible completion, not an admission of uncertainty. Consider a flood-response team routing vehicles along a road the generator hallucinated over submerged terrain, invisible to a global SSIM score that stays high because the rest of the image is correct — or a coastal analyst missing a vessel signature the generator resolved as calm, empty sea. A reliability framework capable of flagging untrustworthy pixels is therefore a precondition, not an enhancement, for operational deployment of translated satellite imagery.

## 2. Problem Statement
- P1 — Optical imagery is human-intuitive but unavailable exactly when cloud cover, smoke, or darkness make it most needed.
- P2 — SAR imagery is always available but requires years of specialist training to interpret correctly.
- P3 — SAR-to-Optical generators produce photorealistic but semantically unreliable output; SSIM/PSNR/FID aggregate over the whole image and cannot detect localized hallucination.
- P4 — No standardized, quantitative benchmark exists that measures where — and how often — SAR-to-Optical generators hallucinate, broken down by land-cover class.
- P5 — The scope of this study is restricted to land-cover-level semantic hallucinations (e.g., water classified as land); fine-grained object-level hallucinations (e.g., vehicle type misidentification) require higher-resolution datasets and object-detection ground truth and are outside the scope of this study.

## 3. Literature Review
Surveys representative post-2021 work spanning GAN-based, diffusion-based, and confidence-aware SAR-to-Optical translation, alongside adjacent NLP hallucination-detection research. Across all approaches, evaluation remains dominated by global perceptual metrics; none establishes a model-agnostic, externally validated, pixel-level hallucination benchmark — the gap this proposal addresses.

| Reference | Year | Data Used | Method / Algorithm | Key Findings | Limitations |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Yang et al. [5] | 2022 | SEN1-2 (Sentinel-1/2 pairs) | Improved conditional GAN with structure + gradient constraints | Improves structural/texture fidelity of translated optical imagery over baseline CGAN | Evaluated with SSIM/PSNR only; no localized or semantic-error analysis |
| Hu et al. [6] | 2023 | Sentinel-1/2, 8,669 patch pairs, 304 Canadian wildfires | ResNet-based Pix2Pix SAR→optical translation | Enables burn-severity assessment from SAR when optical is cloud-obscured | Application-specific (fire scenes); no hallucination quantification for other land-cover classes |
| Zhang et al. [10] | 2023 | N/A — survey of LLM outputs | Taxonomy of hallucination detection: factual consistency, uncertainty quantification, retrieval verification | Establishes that fluency/plausibility is a poor proxy for factual correctness | NLP-specific; no equivalent taxonomy yet exists for remote-sensing image translation |
| Bai et al. [7] | 2024 | GF-3 and SEN12 SAR-optical pairs | Conditional diffusion model conditioned on SAR input | Diffusion backbone surpasses prior GAN baselines on SSIM/FID | Visual-quality metrics only; no pixel-level semantic verification of content |
| Do et al. [8] | 2024 | QXS-SAROPT, SAR2Opt, SpaceNet6 | Latent diffusion with a confidence-guided (C-Diff) loss | Jointly predicts EO output and a confidence map; improves PSNR/SSIM/FID over prior SET methods | Confidence map is a training-time uncertainty proxy, not an externally validated hallucination ground truth |
| Lee et al. [9] | 2026 | Multi-dataset SAR-optical benchmarks | ControlNet-guided diffusion with an uncertainty-aware (aleatoric) objective | Explicitly names "structural hallucinations" as a failure mode; suppresses artefacts via class-aware prompts | Improves the generator itself; no post-hoc, model-agnostic detector usable with third-party generators |

*Table 1. Literature review — SAR-to-Optical translation and hallucination-adjacent research, 2022–2026.*

## 4. Research Objectives
- Implement and validate a baseline pix2pix SAR-to-Optical generator (U-Net + PatchGAN) on SEN12MS.
- Design a double-condition DeepLabV3 filter that generates externally validated, pixel-level hallucination ground truth.
- Produce the first per-land-cover-class hallucination rate benchmark.

## 5. Proposed System and Methodology
SEN12MS (180,662 SAR–optical–land-cover triplets) supplies every modality this pipeline needs. The pipeline (Fig. 1) trains a G_pix2pix baseline and generates fake-optical test images. Crucially, it then applies a double-condition filter using a DeepLabV3 model fine-tuned on Sentinel-2 imagery with SEN12MS IGBP land-cover labels. A pixel is labeled as a ground-truth "hallucination" only if this model classifies it correctly on the real optical image but incorrectly on the fake optical image. This isolates genuine generator errors from segmentation-model limitations, yielding an externally validated, per-class hallucination rate table.

*Fig. 1 Hallucination ground-truth generation pipeline (Stages 1–4).*

## 6. Evaluation Plan
- The baseline pix2pix generator is evaluated using PSNR, SSIM, and FID to confirm it is representative of state-of-the-art SAR-to-Optical translation quality before hallucination auditing begins.
- The double-condition filter's ground-truth quality is spot-checked by manually reviewing a stratified sample of flagged hallucination pixels against human annotation, and by testing sensitivity to the DeepLabV3 boundary-erosion margin.
- The final per-land-cover-class hallucination rate table is computed on a geographically held-out test split — avoiding spatial-autocorrelation leakage between train and test tiles — with per-class rates reported alongside sample counts so low-frequency classes aren't over-interpreted.

## 7. Capstone 1 Timeline
| Month | Milestone / Deliverable |
| :--- | :--- |
| M1 | Literature survey; SEN12MS download & preprocessing pipeline |
| M2 | Train pix2pix baseline generator (G_pix2pix); validate convergence |
| M3 | Run Stages 2–4: synthetic generation, double-condition filter, per-class failure table |
| M4 | Statistical validation across geographic splits; Capstone 1 report |

## 8. Risks and Mitigation
- **Sparse ground truth** — Low measured hallucination rates are unlikely given the physical (not statistical) SAR–optical information gap; a lower rate still yields a publishable, tighter empirical bound.
- **Compute limits** — Generator training (~10–12 GPU-hrs on a T4) and frozen-model inference are modest; a stratified 20% subset is the fallback if full-dataset training exceeds budget.
- **GANs are outdated** — Diffusion-based generators are increasingly common, but pix2pix remains a standard, well-understood baseline for establishing a first quantitative benchmark; extension of the ground-truth pipeline to diffusion generators is direct future work.
- **Co-registration tolerance** — Sentinel-1 and Sentinel-2 acquisitions are not pixel-synchronous; sub-pixel misalignment at class boundaries (riverbanks, building edges) can generate false-positive hallucination labels. Mitigation: a 2-pixel morphological erosion is applied to class boundaries in the segmentation maps before the double-condition comparison, excluding edge-adjacent pixels from the ground truth.
- **Binary ground-truth simplification** — The hallucination mask is binary (hallucinated / faithful). Partial hallucinations — spectrally plausible but incorrect vegetation, or blended artefacts — are not captured by this labeling scheme. Continuous uncertainty scoring is an explicit direction for future work.

## 9. Expected Contributions and Impact
**The Empirical Benchmark** — This research establishes the first quantitative, per-land-cover-class hallucination benchmark in the domain of SAR-to-Optical image translation. By systematically categorizing semantic failures across distinct terrain types (e.g., urban, water, dense forest), this benchmark provides the research community with a definitive, numerical reference point. Future generative models can be evaluated against this baseline to prove genuine semantic improvements, moving the field beyond subjective visual inspections.

**The Ground-Truth Methodology** — We introduce a highly reproducible, automated pipeline for generating pixel-level hallucination ground truth without the need for exhaustive manual annotation. By utilizing a double-condition validation filter with a fine-tuned DeepLabV3 segmentation model, this methodology successfully isolates generator-induced hallucinations from segmentation errors. This robust, externally validated approach is designed to be fully architecture-agnostic, meaning it can be universally applied to audit any existing or future generative model.

**Paradigm Shift and Operational Impact** — This project provides hard empirical evidence that current community evaluation standards (SSIM, PSNR, FID) are structurally insufficient for deployment-grade assessment in safety-critical environments. By shifting the evaluation paradigm from "photorealism" to "semantic trustworthiness," this research directly enables the cautious, real-world adoption of AI-translated satellite imagery. This provides a critical safety layer for organizations making high-stakes, time-critical decisions, such as NDMA/NDRF teams during cloud-obscured disaster response operations or DRDO analysts conducting defense surveillance.

## References
[1] P. Isola, J.-Y. Zhu, T. Zhou, and A. A. Efros, "Image-to-Image Translation with Conditional Adversarial Networks," CVPR, 2017.
[3] L.-C. Chen, G. Papandreou, I. Kokkinos, K. Murphy, and A. L. Yuille, "DeepLab: Semantic Image Segmentation with Deep Convolutional Nets, Atrous Convolution, and Fully Connected CRFs," IEEE TPAMI, vol. 40, no. 4, pp. 834–848, 2018.
[4] M. Schmitt, L. H. Hughes, C. Qiu, and X. X. Zhu, "SEN12MS — A Curated Dataset of Georeferenced Multi-Spectral Sentinel-1/2 Imagery for Deep Learning and Data Fusion," ISPRS Annals, vol. IV-2/W7, 2019.
[5] X. Yang, J. Zhao, Z. Wei, N. Wang, and X. Gao, "SAR-to-optical image translation based on improved CGAN," Pattern Recognition, vol. 121, p. 108208, 2022.
[6] X. Hu, P. Zhang, Y. Ban, and M. Rahnemoonfar, "GAN-based SAR and optical image translation for wildfire impact assessment using multi-source remote sensing data," Remote Sensing of Environment, vol. 289, p. 113522, 2023.
[7] X. Bai, X. Pu, and F. Xu, "Conditional Diffusion for SAR to Optical Image Translation," IEEE Geoscience and Remote Sensing Letters, vol. 21, pp. 1–5, 2024.
[8] J. Do, J. Lee, and M. Kim, "C-DiffSET: Leveraging Latent Diffusion for SAR-to-EO Image Translation with Confidence-Guided Reliable Object Generation," arXiv:2411.10788, 2024.
[9] H. Lee, S. M. Kim, H. K. Shin, T. Kim, and W.-J. Nam, "OSCAR: Optical-aware Semantic Control for Aleatoric Refinement in SAR-to-Optical Translation," arXiv:2601.06835, 2026.
[10] Y. Zhang et al., "Siren's Song in the AI Ocean: A Survey on Hallucination in Large Language Models," arXiv:2309.01219, 2023.
