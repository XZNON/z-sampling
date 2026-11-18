# z-Sampling — project roadmap

## Objective

Understand reconstruction from fundamentals by writing, inspecting, and evaluating progressively more capable systems. First enlarge images; then improve reconstruction quality; later exploit temporal information and generate intermediate frames. Learning and implementation are equally important outcomes.

Method for every stage: understand the mathematics -> implement from scratch -> train/run -> experiment -> benchmark -> improve. The aim is a knowledge and experimental foundation for eventually investigating our own architectures, not a reproduction of DLSS or of existing SR models.

## Naming (renumbered 2026-11-16)

On 2026-11-16 the learner adopted a new roadmap: V0 classical (images + video), V1 CNN SR, V2 Vision Transformer SR, V3 temporal video reconstruction, V4 hybrids and optimization. Frame interpolation and the real-time demo from the earlier plan are kept as V5 and V6.

Old -> new mapping, for reading older notes and commits:

- Old "V1 spatial image upscaling lab" (also "0.1" in early conversation) = new **V0.1**. The learner's commit "V.1 complete" (`928a058`) refers to this; it is a label, not the exit criteria below.
- Old "V2 small neural SR" = new **V1** (CNN).
- Old "V3 temporal reconstruction" = part of new **V3**.
- Old "V4 frame interpolation" = new **V5**; old "V5 real-time demo" = new **V6**.

Version numbers are project milestones, not NVIDIA DLSS versions.

## V0 — classical image and video reconstruction (COMPLETE, closed 2026-11-17)

Status: closed by the learner on 2026-11-17 after the RGB video benchmark (see `state.md` for numbers, limitations and discrepancies). V0.1 items still unchecked below were not completed in V0; they are deliberately carried forward, not silently dropped. The checkboxes record evidence only.

Grayscale and RGB images, grayscale and RGB video; nearest, bilinear, NNV (bicubic optional); MSE, PSNR, SSIM; performance benchmarks. V0 becomes the classical reconstruction engine and baseline for every later version. V0 processes each frame independently; no information from neighboring frames.

### V0.1 — finish grayscale image evaluation (closed; unchecked items carried forward)

- [x] Explain grayscale/RGB arrays, shapes, uint8 range, and resolution versus information.
- [x] Implement integer nearest-neighbor enlargement with `np.repeat` along both axes.
- [x] Implement `lerp` and `biLerp`; verify midpoint and corner values.
- [x] Explore 3x3, 4x4, and 1000x1000 sampling of a surface defined by four samples.
- [x] Select neighboring samples using floor, integer indices, fractional offsets, and border clamping.
- [x] Apply half-pixel output-to-input mapping in a loop-based grayscale upscaler.
- [x] Verify the synthetic 3x4 -> 6x8 output and scale-1 identity in the conversation.
- [x] Reach a 2x grayscale-photo upscaling experiment (user reports completion; notebook includes loading/scaling code, without persisted output for those last cells).
- [x] Explain and perform a controlled reconstruction experiment: reference -> low resolution -> reconstruction at reference size. (Notebook cells 32-36; 736 -> 368 -> 736, box average.)
- [ ] Align sample positions: match the upscaler's coordinate mapping to where each downsampler's samples actually sit (box center 2i+0.5 vs point sample 2i), then recompute the sub-sampling metrics. (NEXT LESSON. Current sub vs box numbers are confounded; see state.md finding 5. The learner derives the mapping.)
- [ ] Explain downsampling, filtering before decimation, aliasing, and why missing information cannot generally be recovered uniquely. (Partial: box vs point decimation built, many-to-one explained by the learner. The checkerboard demo is fixed and ready, but aliasing has not been taught yet.)
- [ ] Compare nearest, learner's bilinear, and library bicubic on the same low-resolution input using identical output dimensions and crops. (Partial: nearest, bilinear, and NNV compared by metrics; no bicubic; no matched-crop visual comparison.)
- [x] Learn MSE and PSNR, compute them against the reference, and discuss why they do not fully measure perceived quality. (Cells 49-57; limitations discussed in markdown.)
- [x] Extra (learner-initiated): SSIM from mean/variance/covariance, global SSIM, local SSIM map. Loop versions written by the learner, who also fixed the uint8 overflow. On request (2026-11-11) the assistant rewrote the fast `sliding_window_view` version and fixed the plot cell; the fast version agrees with the loop at windows 3/7/11. Walkthrough of the fast version pending. Simplified variant (uniform 3x3 window), not standard Gaussian SSIM.
- [ ] Interpret the SSIM map visualization (cell 90) on the real reconstructions: where errors concentrate and why. (Plots exist: `01` cells 90-93 SSIM map, side-by-side, absolute-error map on a 64x64 crop; `02` cells 13-14 on a video frame. Learner interpretation not yet seen.)
- [x] Extra: NNV-style upscaler (mode check, then closest-to-bilinear) implemented and measured. The code appears externally sourced. The assistant fixed the vectorized version to include the mode step (identical to the loop). Learner understanding not checked.
- [ ] Explain cubic interpolation/kernel and implement a manageable bicubic exercise after the bilinear baseline is understood. (Optional within V0; can move to V0.5.)
- [ ] Review vectorized code and verify agreement with the loop implementation on meaningful small examples, borders, scale 1, and constant images. Explain performance/memory implications. (Partial, assistant-side only: bilinear identical at scales 1-2, about 3e-5 at scale 3 from float32 coordinates; NNV and SSIM fast versions match their loops. The learner has not reviewed any vectorized code.) This becomes required before V0.3: loop versions are far too slow for video, so the learner must understand and trust the vectorized ones.
- [ ] Establish a reliable grayscale benchmark: fixed images, fixed degradation, aligned mappings, recorded MSE/PSNR/SSIM and runtime per method.

Carried forward from V0.1 (2026-11-17): sample-position alignment (do it before building V1 training pairs), aliasing demo, optional bicubic, matched-crop comparison, learner review of the vectorized code. Take them one at a time, when they serve V1 or the learner asks.

V0.1 exit: learner can explain and implement mapping, interpolation, and borders; run comparable image experiments; distinguish larger images from recovered detail; interpret basic reconstruction measurements with confounds in view.

### V0.2 — RGB image reconstruction (done; evidence `01` cells 94-125)

- [x] Explain RGB shape `(H, W, 3)` versus grayscale `(H, W)`; one pixel becomes three values. (`01` cells 94-97.)
- [x] Apply the existing upscalers per channel, then recombine. Keep float arithmetic; clip/round only at export. (Loop and vectorized RGB bilinear, RGB box downsample. Note: the vectorized RGB upscaler clips and casts to uint8 inside the function.)
- [ ] Check what happens to NNV per channel (mode chosen independently per channel can produce color combinations). (Not done; NNV was not extended to RGB.)
- [x] RGB metrics: MSE/PSNR over all channels; per-channel MSE/PSNR/SSIM. (Per-channel SSIM averaged; SSIM on luminance (Y) not done. Say which one each number is.) Monkey 2x result: MSE 24.67, PSNR 34.21 dB, SSIM 0.9698 (11x11); per channel nearly identical (`01` cells 114-124; table in `state.md`).
- [ ] Verify loop vs vectorized RGB bilinear agree (only the 2x2 loop test and the vectorized full-image run exist; no agreement check). Note the vectorized version truncates to uint8 (`state.md` discrepancy 10).
- [ ] Discuss color-space limitations: sRGB gamma vs linear light, where interpolation "should" happen, and per-channel independence. Later versions may model cross-channel relationships. (Listed as a limitation in `state.md`; no lesson or experiment yet.)

### V0.3 — grayscale video reconstruction (done; evidence `02` cells 8-23)

- [x] Video = sequence of frames at a frame rate. Choose a short clip at manageable resolution; keep the same frame rate throughout. (1080x608, 29.5 FPS, 564 frames; output FPS and frame count verified.)
- [x] Decode to frames, downsample each frame, reconstruct each frame, reassemble. (OpenCV; `reconstructed_gray.mp4` was written but is not in the repo.)
- [x] Per-frame MSE/PSNR/SSIM; plot the score over time. (30 frames; window-3 SSIM for gray, see state.md discrepancy 1.)
- [ ] Observe single-frame limitations in motion: flicker, unstable or crawling edges, aliasing that moves. (Learner reports blur on text/HUD/edges; flicker and crawling not documented.)
- [x] Separate reconstruction error from codec error: evaluate on decoded frames before re-encoding, and/or store results losslessly (PNG frames or a lossless codec). Compare against a lossy re-encode only deliberately. (Metrics use raw reconstructed frames; the lossy `mp4v` re-encode was not scored.)
- [x] Measure processing speed (frames per second) and relate it to the vectorization work. (Gray timing includes metrics; RGB timing does not.)

Tooling: OpenCV (cv2) and SciPy are installed in `.venv` (checked 2026-11-17). ffmpeg not used.

### V0.4 — RGB video reconstruction (done; evidence `02` cells 25-32)

- [x] Combine V0.2 and V0.3 into a color video upscaling pipeline. (`reconstructed_gta5_rgb.mp4`, 564 frames, 29.5 FPS.)
- [x] Test processing speed and reconstruction quality (30 frames). Motion and compression effects were only discussed, not isolated.

### V0.5 — final evaluation and V0 completion (baseline recorded; comparisons partial)

- [ ] Controlled comparisons across methods, images, and clips. (Video baseline is bilinear only, one clip; method comparisons exist only for the monkey image in `01`.)
- [ ] Visualize artifacts (difference images, SSIM maps, zoomed matched crops). (Difference and SSIM maps done; matched crops not.)
- [x] Document limitations and failure modes. (`state.md`, 2026-11-17.)
- [x] Record a baseline (data, degradation, method, metrics, runtimes) for V1 and later versions. (`state.md`. Not fully reproducible: the source clip is gitignored; SSIM variants differ between gray and RGB; RGB output is truncated, not rounded, to uint8.)
- [ ] Fix one evaluation protocol for all later comparisons: one SSIM setting, one uint8 export rule (round), aligned degradation. (Carried into V1; see `state.md` discrepancies 1 and 10.)

V0 exit: a working classical image and video upscaler the learner understands, with trustworthy measurements and a documented baseline. Closed 2026-11-17 on the learner's decision, with the carried-forward items above.

## V1 — CNN-based super-resolution (NEXT; not started)

Entry point: convolution as a sliding weighted sum, bridged from the learner's own windowed SSIM statistics (box kernel). Then feature maps, learnable kernels, a single neuron, gradient descent, backpropagation, PyTorch. The learner writes the code. Learning objectives: convolution mathematics, image kernels, feature maps, neural networks, backpropagation, PyTorch, datasets, training, evaluation, and a first trained image SR model compared against the V0 baseline.

- [ ] Convolution and feature extraction; backpropagation; PyTorch training loop.
- [ ] Define a supervised 2x reconstruction task and dataset permissions/provenance.
- [ ] Build high/low-resolution pairs with a documented degradation pipeline (reuse V0's); split by source image before patch extraction to avoid leakage.
- [ ] Loss functions and optimization the learner understands. 4 GB-conscious batches; mixed precision if supported and useful.
- [ ] Build and train a first small neural image upscaler.
- [ ] Evaluate on unseen images against V0 baselines; inspect detail, artifacts, runtime, and memory.
- [ ] Study perceptual versus pixel accuracy, hallucinated detail, and synthetic-to-real degradation mismatch.

Single-frame only: learn how a model reconstructs one frame before any temporal neural reconstruction. Architecture, dataset, and runtime setup remain undecided. Inspect hardware/software when this milestone is reached; do not install a full ML stack now.

## V2 — Vision Transformers for super-resolution (confirmed project requirement, not optional)

Goal: understand Vision Transformers, build a small one from scratch, train it for SR, and compare it against the V1 CNN and V0 classical methods. Not merely using a pretrained transformer.

- [ ] Image patches, patch embeddings, positional encoding.
- [ ] Self-attention and multi-head attention; transformer block (attention, MLP, normalization, residuals).
- [ ] The standard ViT (Dosovitskiy et al. 2020) was designed for image understanding (classification), not dense pixel reconstruction. Learn why that matters for SR: token-to-pixel output, cost quadratic in token count.
- [ ] Architectures adapted for restoration/SR: SwinIR (shifted-window attention), HAT (window attention plus channel attention and cross-window communication).
- [ ] Build and train a small transformer-based image upscaler within 4 GB VRAM (small windows/patches, few blocks).
- [ ] Compare CNN vs Transformer vs classical interpolation on the V0 benchmark: quality, artifacts, runtime, memory, parameter count.

## V3 — temporal video reconstruction

Go beyond independent frames.

- [ ] Start with a controlled sequence or scene; understand jitter, motion vectors versus optical flow, and reprojection/warping.
- [ ] Accumulate history with rejection or weighting; inspect occlusion/disocclusion, ghosting, flicker, and scene cuts.
- [ ] Use information from neighboring frames; temporal attention and alignment.
- [ ] Transformer-based video SR: study VRT (Video Restoration Transformer) as a reference. Full VRT is far beyond 4 GB VRAM; plan a small learning-sized variant.
- [ ] Compare temporal results with single-frame baselines (V0, V1, V2) and study open-source FSR 2 as a reference.

## V4 — hybrid architectures and optimization

- [ ] CNN-Transformer hybrids (HAT is one reference point).
- [ ] Efficiency and inference speed on the available GPU.
- [ ] Temporal stability and reconstruction quality trade-offs.

## V5 — frame interpolation (kept from the earlier plan)

- [ ] Define intermediate-time reconstruction from surrounding frames; begin offline.
- [ ] Explore optical flow and warping before a small learned approach.
- [ ] Withhold middle frames from triplets; use sequence-separated training/evaluation data.
- [ ] Study RIFE as an open-source reference, then design a learning-sized implementation/training experiment.
- [ ] Inspect motion boundaries, occlusion, cuts, runtime, and buffering latency.

## V6 — small real-time demonstration (kept from the earlier plan)

- [ ] Choose a controlled rendering/integration target only after offline components work.
- [ ] Measure reconstruction quality, inference time, memory, display pacing, and end-to-end latency.
- [ ] Explain why generated display frames do not raise simulation or input-update frequency.

V1-V6 are directional learning milestones, not finalized implementation specifications.

## Working method

Explain -> learner predicts -> learner implements -> inspect results -> measure -> discuss failures -> record checkpoint. CPU-only through V0. Do not promise DLSS-level quality or speed, or paper-level transformer results, on the available GPU. Avoid arbitrary schedules; progress follows understanding.

## References for later versions

Cite when the stage is reached; not reading assignments now.

- ViT: A. Dosovitskiy et al., "An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale", ICLR 2021 (arXiv 2010.11929).
- SwinIR: J. Liang et al., "SwinIR: Image Restoration Using Swin Transformer", ICCV Workshops 2021.
- HAT: X. Chen et al., "Activating More Pixels in Image Super-Resolution Transformer", CVPR 2023.
- VRT: J. Liang et al., "VRT: A Video Restoration Transformer", arXiv 2201.12288 (2022).
