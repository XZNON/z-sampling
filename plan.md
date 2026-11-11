# z-Sampling — project roadmap

## Objective

Understand reconstruction from fundamentals by writing, inspecting, and evaluating progressively more capable systems. First enlarge images; then improve reconstruction quality; later exploit temporal information and generate intermediate frames. Learning and implementation are equally important outcomes.

Naming: V1 is the image upscaling lab (also called 0.1 in early conversation). V1 is not complete just because 2x resizing works. Later version numbers below are project milestones, not NVIDIA DLSS versions.

## V1 — spatial image upscaling lab (current)

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
- [x] Extra: NNV-style upscaler (mode check, then closest-to-bilinear) implemented and measured. The code appears externally sourced. The assistant fixed the vectorized version to include the mode step (identical to the loop). Learner understanding not checked.
- [ ] Explain cubic interpolation/kernel and implement a manageable bicubic exercise after the bilinear baseline is understood.
- [ ] Extend from grayscale to RGB intentionally; preserve floating-point arithmetic and discuss color-space limitations.
- [ ] Review vectorized code and verify agreement with the loop implementation on meaningful small examples, borders, scale 1, and constant images. Explain performance/memory implications without a broad optimization detour. (Partial, assistant-side only: bilinear identical at scales 1-2, about 3e-5 at scale 3 from float32 coordinates; NNV and SSIM fast versions match their loops. The learner has not reviewed any vectorized code.)
- [ ] Summarize failure modes, findings, and a reproducible baseline for V2.

Order can adapt to the user's questions. As of 2026-11-11: confirm the learner reran the reloaded notebook, then sample-position alignment, then the aliasing demo. Do not do all unchecked items at once. RGB, bicubic implementation, and optimization are later substeps.

The learner's commit "V.1 complete" (`928a058`) is a label, not the V1 exit below; several exit items remain open.

V1 exit: learner can explain and implement mapping, interpolation, and borders; run comparable image experiments; distinguish larger images from recovered detail; interpret basic reconstruction measurements.

## V2 — small neural super-resolution model

- [ ] Define a supervised 2x reconstruction task and dataset permissions/provenance.
- [ ] Build high/low-resolution pairs with a documented degradation pipeline; split by source image before patch extraction to avoid leakage.
- [ ] Train a small model on patches with a loss the learner understands. Use 4 GB-conscious batches and mixed precision if supported and useful.
- [ ] Evaluate on unseen images against V1 baselines; inspect detail, artifacts, runtime, and memory.
- [ ] Study perceptual versus pixel accuracy, hallucinated detail, and synthetic-to-game degradation mismatch.

Architecture, dataset, and runtime setup remain undecided. Inspect hardware/software when this milestone is reached; do not install a full ML stack now.

## V3 — temporal reconstruction

- [ ] Start with a controlled sequence or scene; understand jitter, motion vectors versus optical flow, and reprojection/warping.
- [ ] Accumulate history with rejection or weighting; inspect occlusion/disocclusion, ghosting, flicker, and scene cuts.
- [ ] Compare temporal results with single-frame baselines and study open-source FSR 2 as a reference.

## V4 — frame interpolation

- [ ] Define intermediate-time reconstruction from surrounding frames; begin offline.
- [ ] Explore optical flow and warping before a small learned approach.
- [ ] Withhold middle frames from triplets; use sequence-separated training/evaluation data.
- [ ] Study RIFE as an open-source reference, then design a learning-sized implementation/training experiment.
- [ ] Inspect motion boundaries, occlusion, cuts, runtime, and buffering latency.

## V5 — small real-time demonstration

- [ ] Choose a controlled rendering/integration target only after offline components work.
- [ ] Measure reconstruction quality, inference time, memory, display pacing, and end-to-end latency.
- [ ] Explain why generated display frames do not raise simulation or input-update frequency.

## Working method

Explain -> learner predicts -> learner implements -> inspect results -> measure -> discuss failures -> record checkpoint. Start CPU-only for V1. Do not promise DLSS-level quality or speed on the available GPU. Avoid arbitrary schedules; progress follows understanding.
