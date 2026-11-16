# z-Sampling: current state

Checkpoint: 2026-11-17. **V0 (classical reconstruction) closed by the learner; V1 (CNN super resolution) is next and not started.** Everything below the "Archived" divider is the older V0.1 checkpoint (2026-11-11), kept as the record of review findings and measurements. Where the two disagree, this top section wins.

## V0 closure (2026-11-17)

Evidence: `01_neareest_neigh.ipynb` (126 cells; grayscale work, then RGB images from cell 94 on) and `02_video_upscaling.ipynb` (34 cells; grayscale and RGB video). Commits `b73a334` (RGB), `e8ae166` (rgb video upscale). Numbers below were read from the notebooks' saved outputs; the assistant did not re-run the video notebook.

### What was built and measured

- Classical stack: nearest-neighbor, lerp/bilerp, half-pixel mapping with border clamping, loop and vectorized bilinear (grayscale and RGB), 2x box downsample (loop, reshape-mean vectorized, RGB), point sub-sampling, NNV-style upscaler.
- Metrics: MSE, PSNR, SSIM from mean/variance/covariance. Global and local (windowed) SSIM. RGB: per-channel and total MSE, PSNR from total MSE, per-channel SSIM averaged, per-channel SSIM maps. Grayscale SSIM map and absolute-error map (`02`, cells 13-14).
- Video: OpenCV decode, per-frame downsample + bilinear reconstruction, per-frame metric plots, timing, re-encode with `mp4v`, reopen check. Input: `ginue-pigs/gta5.mp4`, 1080x608, 29.5 FPS, 564 frames (about 19.1 s). Output `reconstructed_gta5_rgb.mp4` reopens with 29.5 FPS and 564 frames. The grayscale output `reconstructed_gray.mp4` also reopened with 29.5 FPS and 564 frames, but that file is not in the working directory or in git.

### Final V0 RGB video benchmark (the baseline for V1 and later)

First 30 consecutive frames (about 1 s) of the clip. Pipeline: 2x box downsample (float64) -> vectorized bilinear 2x (half-pixel mapping) -> compare with the original decoded frame. Source: cell 30 of `02`.

| Metric | Result |
|---|---|
| Average MSE (all channels) | 86.53897 |
| Average PSNR (mean of per-frame PSNR) | 28.79463 dB |
| Average SSIM (mean of per-channel SSIM, 11x11 uniform window) | 0.955572 |
| Min / max frame SSIM | 0.941393 / 0.969917 |
| Processing time | 194.58 ms/frame |
| Processing throughput | 5.139 FPS |

All values match the handoff the learner supplied. Timing covers downsample + reconstruct only (not metrics, not video I/O), CPU, NumPy. A baseline for comparison, not a general performance claim.

Earlier grayscale video numbers (30 frames, cell 18): MSE 86.548, PSNR 28.794 dB, SSIM 0.9340 (3x3 window, see discrepancy 1), 269.2 ms/frame (includes metric computation, see discrepancy 2). Single grayscale frame (cell 12): MSE 109.47, PSNR 27.74 dB, SSIM 0.9427 (11x11 window).

### Limitations observed in V0

- The reconstruction is smoother, not sharper. Bilinear cannot restore discarded detail (many-to-one downsampling; the learner wrote this in `01`).
- The learner reports visible blur around text, HUD elements, edges and fine texture. The error and SSIM maps were plotted for a grayscale frame; the assistant has not inspected the rendered images.
- Frames are processed independently; no temporal information (what V3 targets).
- Speed: 5.1 FPS on CPU for 1080x608 at 2x, before any neural network.
- Channels are processed independently in gamma-encoded sRGB; no linear-light handling or cross-channel modelling. The learner measured channel differences on frame 0 (cell 32: B-G 28.6, G-R 13.6, R-B 40.6 mean absolute difference); no conclusion was drawn.
- Metrics are simplified (uniform-window SSIM, not Gaussian 11x11 sigma 1.5), so scores will not match library or paper numbers. MSE/PSNR/SSIM do not fully track perception.
- The `.mp4` output is lossy (`mp4v`). Metrics were computed on reconstructed frames before encoding, so codec error is not in the numbers.
- `downsample_2x` and `downsample_rgb_vectorized` use `reshape`, so odd frame dimensions would fail. The clip is 1080x608 (both even).
- One clip, first 30 frames only; GTA footage may not generalize.

### Discrepancies between the handoff, the docs, and the repo

1. SSIM windows differ between the two video benchmarks. Grayscale video (cell 16) calls `calculate_local_ssim_optimized` with its default window 3 (valid region only). RGB video uses `calculate_ssim_rgb` with `uniform_filter` window 11 (edge-reflected). 0.9340 (gray) vs 0.9556 (RGB) is therefore NOT evidence that RGB reconstructs better.
2. Grayscale timing (269.2 ms/frame) includes MSE/PSNR/SSIM; the RGB timing (194.58 ms) does not. Not comparable. Cell 28 (267.2 ms, 3.74 FPS) is stale: it appended one extra single-frame sample onto the gray loop's list. The 194.58 ms / 5.139 FPS figures come from cell 30, which reset the lists.
3. In `02` cell 29 the variables `mse_r, mse_g, mse_b` and `ssim_r, ssim_g, ssim_b` hold B, G, R because OpenCV frames are BGR. Totals (the benchmark) are unaffected. `01` uses PIL RGB, so its labels are correct.
4. `02` cell 25 asserts `recon_bgr.shape` before `recon_bgr` is assigned in the loop. It only runs because cell 22 left that variable behind, and it checks the previous frame. Cell 25 alone after a restart would raise `NameError`. Left unchanged (learner's code).
5. The source clip is gitignored (`ginue-pigs/gta5.mp4`, `assets/gta5.mp4`), so the benchmark cannot be reproduced from the public repo alone. `reconstructed_gta5_rgb.mp4` (17.6 MB) is tracked.
6. Context docs lagged: they said `01` had 91 cells, "no video tooling installed", and "nothing about RGB or video taught". Now `01` has 126 cells and `.venv` has OpenCV (cv2 5.0.0 reported) and SciPy 1.17.1 (checked 2026-11-17).
7. "V0 complete" vs the V0.1 checklist: the sample-position alignment lesson, the aliasing lesson, bicubic, a matched-crop visual comparison, and the learner's review of the vectorized code have no notebook evidence (`01` has no bicubic and no alignment fix). V0 is closed by the learner's decision, not because every plan item was ticked. See "Carried forward".
8. The supplied handoff lists V0-V4 only. V5 (frame interpolation) and V6 (real-time demo) remain in `plan.md` from the earlier roadmap; nothing was removed.
9. The handoff lists "standard deviation". The notebooks compute variance and covariance; no separate standard-deviation step was found. Immaterial (square root of variance), noted for accuracy.

### Carried forward (not done in V0)

- Sample-position alignment (finding 5 below): the sub-sampling vs box numbers in `01` stay confounded. The video baseline is unaffected: box downsample (center 2i+0.5) + half-pixel bilinear is aligned. Do this before building V1 training pairs, because the degradation and the model's input/output grid must agree.
- Aliasing lesson with the checkerboard; filtering before decimation.
- Bicubic (optional). Matched-crop visual comparison. Learner walkthrough of the vectorized functions. The learner's interpretation of the SSIM and error maps.

## Resume point (V1 entry)

STOPPED AFTER: V0 closed with the benchmark above. Nothing in V1 has been taught or installed (no PyTorch yet; check hardware/software when V1 starts, and do not install a full ML stack unasked).

Suggested first V1 lesson (a bridge from what the learner already built): a convolution is a sliding weighted sum. The learner's windowed mean in SSIM (`sliding_window_view` / `uniform_filter`) is a convolution with a box kernel. One exercise: the learner predicts by hand, then writes a loop 2D convolution/correlation for a small kernel on a small grayscale array, checks that a 3x3 box kernel reproduces their window mean, then tries an edge kernel and a blur kernel and says what each does. Explain first, then ask for the implementation. After that: feature maps, learnable kernels, a single neuron/linear layer, gradient descent, backpropagation, then PyTorch. The learner writes the code; do not build the V1 lab.

---

# Archived: V0.1 checkpoint (2026-11-11)

(Original text follows unchanged. Its "Immediate next teaching step" is superseded by the V0 closure and the Carried forward list above.)

Original checkpoint: 2026-11-11, about 01:00 Asia/Calcutta (end of the review-and-fix session). Roadmap update recorded 2026-11-16 (no teaching progress that day). Update this document as work progresses.

## Resume point

STOPPED AFTER: the learner built the controlled experiment (reference -> 2x box downsample -> reconstruct -> compare). They added point sub-sampling, an NNV-style upscaler, nearest-neighbor upscaling, MSE, PSNR, and a from-scratch SSIM (mean/variance/covariance -> global SSIM -> sliding-window local SSIM). An assistant review on 2026-11-11 found several bugs and one experiment confound (listed below). The learner fixed the uint8 overflow in `image_stats`/`calculate_covariance` themselves during the review. Afterwards the user explicitly asked the assistant to fix the remaining bugs. The assistant applied fixes for findings 2, 3, 4, 6, 7 and the minor items, and left finding 5 (alignment) as the next lesson.

Verified state of the notebook ON DISK (mtime 2026-11-11 00:35:22): every code cell runs in order without errors, in 5.1s total, run outside Jupyter with the Agg backend. Cell 89 prints True for loop vs fast SSIM at windows 3, 7, 11.

Editor desync incident (same night): the learner's open editor kept showing the pre-fix cells after the disk edits. They ran the old metrics cell, which calls the pure-Python `calculate_local_ssim` four times on 736x736 images. That takes about 57s and looked like a crash. Their save from the stale editor did not reach disk (mtime unchanged). The assistant reproduced the old cell outside Jupyter: no exception, just slow. The learner was told to reload from disk (VS Code: `File: Revert File`; JupyterLab: File > Reload Notebook from Disk), then Restart + Run All. NOT YET CONFIRMED: the learner has not reported numbers from the reloaded notebook.

Git: the learner committed the notebook fixes and context-doc updates as `c32405a updates and fixing` (2026-11-11 13:32, `main`, tracks `origin/main`). Nothing was committed or pushed by the assistant. Earlier learner commits: `928a058 V.1 complete`, `aef17ca variance, covariance, SSIM`, `2e5f9ef SMIIS`. "V.1 complete" is the learner's label for what is now V0.1; its exit criteria are not all met (no bicubic, vectorized review open, aliasing not demonstrated).

## Roadmap change (2026-11-16)

The learner adopted a new roadmap and asked for it to be recorded (plan.md has the details and the old -> new mapping):

- V0 classical reconstruction, extended: V0.1 finish grayscale image evaluation (current), V0.2 RGB images, V0.3 grayscale video, V0.4 RGB video, V0.5 final evaluation and baseline.
- V1 CNN SR. V2 Vision Transformers for SR (build a small ViT from scratch; study SwinIR, HAT). V3 temporal video reconstruction (incl. temporal attention; VRT as reference). V4 hybrids and optimization. V5 frame interpolation and V6 real-time demo are kept from the earlier plan.
- Old "V1 lab" = new V0.1. Old "V2 neural SR" = new V1. Interpret older notes accordingly.

Consequence for V0.1: reviewing and trusting the vectorized implementations is now required before video (V0.3), because the loop versions are far too slow for frames. No video tooling is installed yet; nothing about RGB or video has been taught.

## Working rule learned this session

The learner usually has `01_neareest_neigh.ipynb` open in their editor. Disk edits stay invisible there until the learner reloads, and a save from the stale editor can silently revert them. Before editing the notebook on disk, ask the learner to save and close it. After editing, ask them to reload from disk and Restart + Run All. If they report "my version differs", compare the file mtime with your edit time.

## Files actually present at inspection

- `01_neareest_neigh.ipynb`: main learner notebook, 91 cells. Preserve its spelling and content.
- `.venv/`: Python 3.11 (uv-managed) with numpy 2.4.6, Pillow, matplotlib 3.11.2 (verified 2026-11-11). Call `.venv/Scripts/python.exe` explicitly. On Windows set `PYTHONIOENCODING=utf-8` when printing notebook text: the markdown contains non-cp1252 characters.
- `assets/`: lesson diagrams and outputs. Includes LowRes.png, subDownsample.png, subUpscaled.png, nvvUpscale.png, nvvUpscale1.png, nearestNeighImage.png, upscaled.png, and the tiny nearest/nearest2/bilerp/bilerp2.png. `nvvUpscale*.png` came from the old (pre-mode-step) NNV and are stale.
- `ginue-pigs/`: monkey.jpg (736x736 after `convert("L")`), cat.jpg, bw_monkey.png.
- Context docs (AGENTS/HANDOFF/state/learning/plan) were committed by the learner in `f7d6fed`. They are tracked in the public repo. The `.gitignore` "Local project context" section is empty.
- No other copy of the notebook exists under the user profile (checked 2026-11-11).

## Notebook map (zero-based; locate by content if numbers shift)

- 0-18: nearest-neighbor `np.repeat`, `lerp`, `biLerp`, 3x3 / 4x4 / 1000x1000 four-corner surfaces.
- 19-26: `findNeigh` (clamp before floor), half-pixel mapping markdown, loop `upscale_bilinear`, `upscale_bilinear_vectorized` (heading calls it "optimized as hell (production style almost)"; the learner has not reviewed it), scale-1 identity check.
- 27-31: monkey loaded as grayscale float, 2x upscale.
- 32-34: `downsample_2x` (2x2 box average, loop). Example verified: [[154.5,125.25],[116.75,97.25]].
- 36: pipeline `original` (uint8) -> `lowRes` (box, rounded to uint8) -> `reconstructed` (vectorized bilinear, uint8). 736 -> 368 -> 736.
- 38: ill-posed explanation (0 and 255 average to 127.5; many-to-one).
- 39-41: `subDownsample_2x` = top-left pixel of each 2x2 block (point decimation). Markdown corrected to say point sampling, not mode.
- 43-46: `upscale_nnv` (loop; mode check + closest-to-bilinear) and `upscale_nnv_optimized` (vectorized; assistant rewrote it to include the mode step; identical to the loop on all tests). Only the vectorized one produces results. The NNV mapping is `r/scale` (no half-pixel offset), unlike the bilinear upscaler.
- 48: learner markdown on why interpolation cannot recover discarded values and what learned SR adds (plausible vs exact detail).
- 49-54: MSE/PSNR markdown + `calculate_mse`, `calculate_psnr`.
- 55-57: `nearest_neigh_upscaling` (loop) + comparisons.
- 58-63: synthetic tests: block-aligned vertical edge; checkerboard (cell 61 now uses `low_res_sub`; cell 62's saved output is stale).
- 65-76: mean/variance (`image_stats`), covariance, global `calculate_ssim` with 8-bit C1/C2.
- 77-83: `calculate_local_ssim` (uniform window, default 3x3, loop) and `calculate_local_ssim_optimized` (assistant-written with `sliding_window_view`, float64, 8-bit C1/C2). 7x7 one-pixel demo: nine windows at about 0.0556, mean SSIM 0.66003.
- 85-86: monkey metrics (cell 86 uses the fast SSIM on disk). 87: duplicate `calculate_covariance`. 88: identical-image SSIM sanity check = 1.0. 89 (assistant-added): loop vs fast SSIM agreement check. 90: SSIM map plot.

## Verified results (independently reproduced 2026-11-11)

Against the reference monkey (L, float), 2x:

| pipeline | MSE | PSNR dB | mean local SSIM (3x3 box) |
|---|---|---|---|
| box -> bilinear | 24.41 | 34.25 | 0.9590 |
| box -> nearest | 41.39 | 31.96 | 0.9429 (assistant check only) |
| sub -> bilinear | 51.72 | 30.99 | 0.9500 |
| box -> NNV (with mode step) | 119.74 | 27.35 | 0.9057 |
| sub -> NNV (with mode step) | 95.59 | 28.33 | 0.9150 |

Before the NNV fix (closest-to-bilinear only): box 67.16 / 29.86 / 0.9333, sub 64.06 / 30.07 / 0.9393. The saved output of cell 86 still shows these old numbers until the learner reruns.

Loop vs vectorized bilinear: identical at scales 1 and 2 on random inputs. Max difference about 3e-5 at scale 3, because the vectorized version uses float32 coordinates. Border behavior correct (duplicated neighbors).

Performance: the loop local SSIM costs about 26 us per window. A 736x736 image at window 3 has 734 x 734 = 538,756 windows, so one call takes about 14s and the four-call metrics cell about 57s. The fast version takes about 0.1s per call.

## Review findings (2026-11-11)

1. FIXED by the learner during the review: uint8 overflow in `image_stats`/`calculate_covariance` gave SSIM of about 0.0065. Now cast to float64.
2. FIXED by the assistant on request. The plot cell would have crashed: `plt` was never imported, and `original` had been overwritten by 2x2 toy arrays in cells 67/75. It now uses `image`.
3. FIXED by the assistant on request. `calculate_mse` did `return ValueError(...)` instead of `raise`, so PSNR then failed with a confusing TypeError.
4. FIXED by the assistant on request. Cell 61 used `upscale_bilinear(low_res, 2)` where it should use `low_res_sub`. With the fix, the checkerboard point-samples to all zeros while the box average gives 127.5 everywhere: a clean aliasing demo that the bug had hidden. Not yet taught.
5. OPEN (next lesson). Experiment confound: `subDownsample_2x` takes pixel 2i, but the half-pixel upscaler assumes low-res pixel i is centered at 2i+0.5. Assistant check: sub -> bilinear MSE drops from 51.72 to 18.44 when the mapping matches the 2i sample position, which beats box -> bilinear (24.41) on this photo. So "averaging beats sub-sampling" is NOT established by the current numbers. The NNV mapping (r/scale) has the opposite mismatch: aligned for sub, shifted for box. The learner was told the 18.44 figure but has not derived the matched mapping.
6. FIXED by the assistant on request: `upscale_nnv_optimized` now includes the mode step and matches `upscale_nnv` exactly. Before, it differed on about 22% of pixels of a monkey crop. Paper not cited in the notebook; likely Rukundo & Cao 2012, "Nearest Neighbor Value Interpolation" (unverified).
7. FIXED by the assistant on request: `calculate_local_ssim_optimized` was rewritten. It had used [0,1]-range constants C1=1e-4, C2=9e-4 (7x7 demo gave 0.640 vs the loop's 0.660). It also applied A's strides to B, reading wrong memory for mixed dtypes.
8. OPEN (note only). The uniform 3x3 window, divide-by-N SSIM is a simplified variant. Standard SSIM (Wang et al. 2004) uses an 11x11 Gaussian window with sigma 1.5, and common libraries differ in details such as sample vs population covariance. Numbers will not match library or paper SSIM.
9. Minor. FIXED: cell 14 `res4`, cell 51 labels, cell 47 list, covariance raises on shape mismatch (cells 70 and 87). OPEN: duplicate covariance cell 87 kept; block-aligned edge test favors nearest (notebook edge [0,0,255,255]: nearest MSE 0 vs bilinear 2048; assistant check with a non-aligned edge [0,0,0,255]: nearest 8128.25 vs bilinear 6592.25); an MSE between two reconstructions measures how different they are, not quality.

## Open questions

- Has the learner reloaded and rerun the notebook? Cell 86 should now print the NNV numbers in the table.
- Does the learner want walkthroughs of the two assistant-written vectorized functions (`sliding_window_view` window means; mode counting via pairwise equality)? Do not assume they are understood.
- Does the learner want the roadmap doc updates (2026-11-16) committed? Ask; do not commit unprompted.

## Immediate next teaching step

1. Confirm the rerun numbers from cell 86.
2. Teach item 5, sample-position alignment, as one concept with one exercise. The learner predicts where sub-sampled pixel i sits in reference coordinates, adjusts the mapping, and recomputes MSE. Do not give the mapping before they predict.
3. After that: aliasing, using the fixed checkerboard demo (cells 61-62).
4. Remaining V0.1 items (SSIM map interpretation, vectorized walkthroughs, benchmark) before V0.2 RGB. Do not start RGB or video early.
