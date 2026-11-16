# z-Sampling: current state

Checkpoint: 2026-11-11, about 01:00 Asia/Calcutta (end of the review-and-fix session). Roadmap update recorded 2026-11-16 (no teaching progress that day). Update this document as work progresses.

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
