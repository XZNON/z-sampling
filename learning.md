# z-Sampling: learning record

Last updated: 2026-11-11. Records what the learner has actually done and understood. Notebook evidence beats memory of conversations. Mark uncertainty honestly.

## Learner context

Professional Python; intermediate ML; RTX 3050, 4 GB VRAM. Wants fundamentals and hands-on model building, not an assistant-built product. Prefers small explanations tied to their own code. Coordinate mapping needed several explanations; revisit its purpose or derive one sample when needed rather than assuming fluency. Asks "why" questions mid-task (e.g. why local SSIM was slow, why the notebook "crashed"); pause and answer them.

## Concepts covered and evidence

### Images, types, and resolution

- Grayscale is a 2D sample array; RGB adds channels. NumPy indexing is `[row, col]` = `[y, x]`; shape is `(height, width)` for grayscale.
- Original exercise: `[[0, 64], [128, 255]]`. Shape `(2,2)` means four pixels. A 2x increase along both axes yields `(4,4)` and four times the pixel count.
- `np.uint8` stores 0..255; `np.int8` is signed -128..127 and was an early mistake.
- Repeat horizontally with `axis=1`, vertically with `axis=0`. `repeats=4` repeats each element four times; it does not mean target width 4.
- Nearest neighbor copies samples, creating blocks without additional observed information.

### Lerp and bilerp

- `lerp(a,b,t) = (1-t)*a + t*b`; between 0 and 64, quarter/mid/three-quarter values are 16, 32, 48.
- Notebook `lerp` raises ValueError for t outside [0,1].
- Corner order: top-left a, top-right b, bottom-left c, bottom-right d.
- Bilerp: top = lerp(a,b,tx); bottom = lerp(c,d,tx); result = lerp(top,bottom,ty).
- Original 2x2 center is 111.75; corner tests yield 0,64,128,255.
- 3x3 exercise uses positions `[0,0.5,1]` on each axis. Expected rows: `[0,32,64]`, `[64,111.75,159.5]`, `[128,191.5,255]`.
- `row,col` select destination storage; `tx,ty` describe where to sample within a neighbor square. These were initially confusing; nested loops were demonstrated explicitly.
- Sampling the same four-corner surface at 1000x1000 produces a dense smooth surface, not newly recovered scene detail.

### Precision

- Keep interpolation in floating point. Direct assignment of a positive fractional value into uint8 truncates it; 111.75 becomes 111.
- At export: `np.clip(np.rint(result),0,255).astype(np.uint8)` rounds and bounds the values.
- `np.float64(111.75)` is a scalar type representation, not an error; 0.2999999999999998 is ordinary floating-point approximation.
- The user perceived improved appearance with floats. Do not attribute a large visible improvement to sub-integer precision without checking identical display scaling. Smoothness versus nearest is mainly interpolation; truncation versus rounding here differs by at most one output level.
- uint8 accumulation overflow (learner found and fixed, 2026-11-11): `mean = 0; mean += image[r][c]` stays uint8 under NumPy 2 rules (a Python int does not upcast a NumPy scalar), so sums wrap at 256. Symptom: SSIM of about 0.0065 on a good reconstruction. Fix: `np.asarray(x, dtype=np.float64)` before any statistics. Good independent debugging.

### Neighbor lookup and mapping

- For input position x=2.3,y=1.7, floor gives left/top indices 2,1; right/bottom are 3,2; tx=0.3,ty=0.7.
- Clamp x,y to the valid input center range BEFORE floor in `findNeigh`. Cast indices to ints.
- Actual signature: `findNeigh(x,y,height,width)`, NOT width,height. Return order: x0,y0,x1,y1,tx,ty.
- Use width for x limits; height for y limits. Call with keywords when debugging dimension order.
- For height=3,width=4 and x=3.2,y=-0.25, expected return is `(3,0,3,1,0,0)`.
- Mapping used in the upscaler: `x=(col+0.5)/scale-0.5`, `y=(row+0.5)/scale-0.5`.
- Meaning: output center -> same location expressed in input coordinates -> sample there. The formula selects a LOCATION; bilerp estimates the VALUE at that location.
- The half-pixel convention aligns full image boundaries. `linspace(0,1,N)` in the four-corner exercise aligns endpoint centers instead. Do not conflate these conventions or imply there is only one valid convention.
- 2->4 mapping before clamping: [-0.25,0.25,0.75,1.25]; after clamping: [0,0.25,0.75,1].
- NOT YET TAUGHT: the mapping silently assumes where each low-res sample came from in the reference. Box averaging of block i has its center at 2i+0.5 (matches the half-pixel mapping). Point sampling at 2i does not. This is the next lesson (state.md finding 5).

### General grayscale upscaler

- User implemented output allocation, nested loops, coordinate mapping, neighbor lookup, bilerp, and storage.
- Earlier bug: column variable `c` was overwritten by bottom-left brightness. Renamed output loop variables to `row,col`.
- On `[[0,40,80,120],[60,100,140,180],[120,160,200,240]]`, sample at x=1.25,y=0.5 gives 80.
- 2x output shape `(6,8)` and first row `[0,10,30,50,70,90,110,120]` were reported correctly.
- Scale-1 equality reported True for both loop and vectorized versions.
- `upscale_bilinear_vectorized` exists (meshgrid + fancy indexing). Assistant verified it matches the loop (identical at scales 1-2; about 3e-5 at scale 3 from float32 coordinates). The learner has not walked through it; its derivation is not an established learning achievement.

### Downsampling and the ill-posed inverse

- Built `downsample_2x` (2x2 box average, loop); verified on a 4x4 example -> [[154.5,125.25],[116.75,97.25]].
- Controlled experiment: reference (736x736) -> low-res (368x368) -> reconstruct at 736x736 -> compare against the reference. Dimensions match so errors are measurable.
- Learner explained in their own words that averaging is many-to-one (0 and 255 -> 127.5; infinitely many inputs give the same average), so no upscaler can uniquely invert it.
- Built `subDownsample_2x` (keep the top-left pixel of each block = point sampling/decimation). The learner first called this "mode"; the notebook markdown now says point sampling. Whether the learner's understanding has changed is unconfirmed.
- Learner markdown (cell 48) distinguishes learned SR (plausible, predicted detail) from exact recovery. Correct framing for V2.

### Error metrics: MSE and PSNR

- Implemented `calculate_mse` (float64, raises on shape mismatch) and `calculate_psnr` (10 log10(255^2/MSE); inf when MSE = 0).
- Notebook markdown covers why errors are squared (no cancellation; big errors penalized), dB scale, PSNR normalising for bit depth, and that a one-pixel shift wrecks PSNR while a human sees no difference. Much of this markdown reads as pasted reference text. The learner's understanding is evidenced by working code and correct interpretation of the scores, not by the prose.
- The "30-40 dB good, below 20 poor" bands in the markdown are rough rules of thumb that depend on content; do not treat them as thresholds.
- Synthetic tests: a block-aligned vertical edge gives nearest MSE 0 vs bilinear 2048. This is test-design bias (the edge sits exactly on block boundaries), not proof that nearest is better. Not yet discussed with the learner.
- Comparing two reconstructions to each other (cell 51 lower half) measures how different they are, not quality. Mentioned in review; not yet discussed in depth.

### SSIM (learner-initiated extension)

- Built from parts, by loops: mean, population variance (divide by N), covariance. Learner's own examples: A=[0,50,100,150] vs reversed B: same mean and variance, covariance -3125 vs +3125, so variance alone misses structure.
- Global SSIM = luminance term x contrast-structure term with 8-bit constants C1=(0.01*255)^2, C2=(0.03*255)^2. Uniform 2x2 example: brightness shift A gives 0.995; alternating pattern B gives 0.369.
- Local SSIM: slide a window, compute SSIM per window, average for a single score. Learner explained why one changed pixel at (3,3) affects nine 3x3 windows (start rows/cols 1..3) and verified mean SSIM 0.66003 by hand and in code. The map localises the neighborhood of an error, not the exact pixel.
- Identical-image sanity check returns 1.0.
- Why the loop version is slow (answered 2026-11-11): about 26 us per window in pure Python; 538,756 windows per full image; about 14s per call. The fast version (`sliding_window_view`, assistant-written) uses mean(x^2)-mean(x)^2 for variance and mean(xy)-mean(x)mean(y) for covariance. It matches the loop for windows 3/7/11. The learner has not had it explained.
- This is a simplified SSIM (uniform window, population statistics). Standard SSIM uses an 11x11 Gaussian window (sigma 1.5). Scores are internally comparable but will not match published or library numbers.

### Nearest Neighbor Value (NNV) interpolation

- The algorithm, per output pixel, looks at the four surrounding low-res values. If any value repeats (a mode exists), use it; otherwise pick the neighbor closest to the bilinear estimate. The output only contains values that existed in the input.
- The code appears externally sourced (different style; paper uncited, likely Rukundo & Cao 2012, unverified). Learner understanding not checked.
- With the real mode step, NNV scores worse than bilinear on the monkey (box -> NNV MSE 119.74). Its mapping `r/scale` lacks the half-pixel offset, so it is aligned for point-sampled input and shifted for box input. That confound is part of the alignment lesson.

### Debugging and workflow lessons

- `return ValueError(...)` returns an exception object instead of raising it; the failure then surfaces later as a confusing TypeError. Use `raise`.
- Reusing a global name (`original`) for toy arrays broke a later cell that expected the monkey. Prefer distinct names for toy data.
- Notebook editor vs disk (2026-11-11): edits made to the .ipynb on disk do not appear in an already-open editor; running or saving the stale copy undoes them. Reload from disk after external edits.
- A slow cell (`[*]` for a minute) is not a crash. Time a small crop first, then extrapolate.

## Results to remember (monkey, 2x, vs reference)

box -> bilinear MSE 24.41 / 34.25 dB / SSIM 0.959. Sub -> bilinear 51.72 / 30.99 / 0.950, but this comparison is confounded by alignment: with matched alignment, sub -> bilinear MSE is 18.44 (assistant check). Box -> nearest 41.39 / 31.96. NNV: box 119.74, sub 95.59 MSE. Full table in state.md.

## Earlier conceptual question (2026-10-29)

User asked whether resizing actually improves quality and whether keeping dimensions fixed is the goal. Clarified:

- Resizing changes dimensions; bilinear smooths transitions but does not uniquely recover lost detail.
- Restoration can reduce noise/blur at unchanged dimensions.
- Super resolution reconstructs at higher dimensions, trying to estimate finer detail. Learned detail can be incorrect.
- Traditional supersampling renders higher then downsamples; super resolution reconstructs from lower-resolution input. DLSS-style temporal inputs add evidence beyond a single image.
- The controlled experiment (now built) keeps reference and reconstruction dimensions equal by first shrinking the reference.

## Remaining confusion / not yet covered

- Next: sample-position alignment between downsampler and upscaler (learner must derive the matched mapping themselves).
- Then: aliasing, demonstrated with the fixed checkerboard (point sampling turns a 0/255 checkerboard into all 0; box gives 127.5 everywhere); filtering before decimation.
- Walkthroughs of the assistant-written vectorized code (bilinear, NNV, fast SSIM) on request.
- Test-design bias; difference-between-methods vs quality-against-reference.
- Not covered in depth: frequency-domain sampling theory, Gaussian-window SSIM, bicubic derivation, RGB/linear-light processing, neural SR training, temporal reconstruction, motion estimation, interpolation models, real-time integration. These are planned, not completed.

## References

Cite only what has been checked. Do not turn these into reading assignments before the learner needs them.

- SSIM: Z. Wang, A. Bovik, H. Sheikh, E. Simoncelli, "Image Quality Assessment: From Error Visibility to Structural Similarity", IEEE TIP 2004.
- NNV (unverified match to the notebook's source): O. Rukundo, H. Cao, "Nearest Neighbor Value Interpolation", 2012.
- Pillow filter descriptions: https://pillow.readthedocs.io/en/stable/handbook/concepts.html#filters
- NumPy `sliding_window_view`: https://numpy.org/doc/stable/reference/generated/numpy.lib.stride_tricks.sliding_window_view.html
- FSR 2: https://github.com/GPUOpen-Effects/FidelityFX-FSR2
- RIFE: https://github.com/hzwer/ECCV2022-RIFE
