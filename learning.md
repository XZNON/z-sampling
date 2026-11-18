# z-Sampling: learning record

Last updated: 2026-11-18 (audit: RGB image results, RGB loop tests, SSIM helper redefinition, truncation added from notebook evidence; no new teaching). Previous: 2026-11-17 (V0 closed). Records what the learner has actually done and understood. Notebook evidence beats memory of conversations. Mark uncertainty honestly.

## Learner context

Professional Python; intermediate ML; RTX 3050, 4 GB VRAM. Wants fundamentals and hands-on model building, not an assistant-built product. Prefers small explanations tied to their own code. Coordinate mapping needed several explanations; revisit its purpose or derive one sample when needed rather than assuming fluency. Asks "why" questions mid-task (e.g. why local SSIM was slow, why the notebook "crashed"); pause and answer them.

Goals stated 2026-11-16: extend the classical stage (V0) to RGB images and grayscale/RGB video before neural networks; then CNN SR (V1) and Vision Transformers (V2). For transformers the learner wants to understand and build a small ViT from scratch and train it for SR, not just use a pretrained one. Roadmap and old -> new version mapping are in plan.md (old "V1 lab" = new V0.1).

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
- Learner markdown (cell 48) distinguishes learned SR (plausible, predicted detail) from exact recovery. Correct framing for V1/V2 (neural SR).

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

### RGB images (V0.2; evidence `01` cells 94-125)

- RGB is `(H, W, 3)`; bilinear is applied to each channel independently with the same mapping, so the math is unchanged. The loop version (`upscale_bilinear_rgb`) and the vectorized one (`upscale_bilinear_rgb_vectorized`, which adds a channel axis to `tx`, `ty` for broadcasting) both exist. The vectorized one clips and casts to uint8 internally.
- RGB metrics: per-channel MSE plus total MSE; PSNR from total MSE (and per channel); SSIM per channel, averaged; per-channel SSIM maps, plus their channel mean as a heatmap. The notebook states why per-channel matters: the total MSE does not show which channel causes the problem. The notebook's own comment explains why a 3-channel array must not go straight into `uniform_filter` (it would blend channels).
- Learner built the RGB bilerp from their own grayscale pieces: `bilerp_rgb` splits four RGB corners into R, G, B lists and calls `biLerp1` (lerp-based) per channel. Red/green/blue/white corners at (0.5, 0.5) give [127.5, 127.5, 127.5]. Loop `upscale_bilinear_rgb` reuses `findNeigh` and the half-pixel mapping; on the 2x2 color test it gives shape (4, 4, 3) and `up[1,1]` = [159.375, 63.75, 63.75]. Learner noted the loop is too slow for a full photo and switched to the vectorized version.
- Learner markdown (cell 112) works the RGB MSE definition (divide by 3HW) through a one-pixel example, and cell 115 motivates per-channel MSE ("what if R and G were 0 and B was 300"). Cell 118: PSNR follows from MSE, so the per-channel version is optional.
- Monkey RGB box -> bilinear: MSE 24.67 (R/G/B 24.56/24.67/24.77), PSNR 34.21 dB, SSIM 0.9698 (11x11 uniform). Channels nearly equal on this photo. Not comparable to gray SSIM 0.959 (3x3 window).
- Cell 122 replaced the learner's loop SSIM helpers with `uniform_filter` versions under the same names (state.md discrepancy 11). Same maths, different window default and border handling. Whether the learner sees why `uniform_filter` equals their window mean is not checked; it is the bridge into convolution for V1.
- Truncation came back: the vectorized RGB upscaler casts with `astype(np.uint8)` (floor), the same issue as 111.75 -> 111 in the precision section. Costs about 0.2 MSE on the monkey (state.md discrepancy 10). Not discussed with the learner yet.
- Not covered: SSIM on luminance (Y), linear-light vs gamma-encoded sRGB, NNV on color. Understanding of color spaces is not evidenced.

### Video (V0.3 and V0.4; evidence `02`)

- Video as frames at a frame rate: decoded with OpenCV (`VideoCapture`, FPS, frame count), processed frame by frame, re-encoded with `VideoWriter` (`mp4v`), and reopened to check FPS 29.5 and 564 frames. OpenCV frames are BGR, not RGB; `cvtColor(..., COLOR_BGR2GRAY)` and `COLOR_GRAY2BGR` were used for the grayscale path.
- Vectorized downsample as `reshape(h//2, 2, w//2, 2).mean(axis=(1,3))` replaced the loop version for speed. This was written for the video work; whether the learner can derive the reshape trick unaided is not checked. Odd frame sizes would break it.
- Per-frame MSE, PSNR and SSIM were stored and plotted over time. Timing was done with `time.perf_counter`; the final RGB timing excludes metrics and I/O. The earlier grayscale timing included metrics (see state.md discrepancies). Throughput = frames / total time; 194.58 ms/frame = 5.139 FPS.
- Evaluation was done on reconstructed frames before the lossy re-encode, so codec error is not mixed in (stated intent in plan.md V0.3; the re-encoded video itself was not scored).
- Grayscale per-frame SSIM over 30 frames ranged 0.9190 (frame 0, worst) to 0.9555 (frame 29, best); the learner printed worst/best frame indices. Frame 0 also gave the single-frame MSE 109.47, above the 30-frame mean of 86.55. Why frame 0 is harder (content, HUD, motion) was not investigated.
- Inspection: the learner plotted original / bilinear / SSIM map side by side and an absolute-error map for a full video frame (`02` cells 13-14), after doing the same on a 64x64 monkey crop (`01` cells 90-93). Interpretation not recorded.
- Learned the hard way: the SSIM window size and padding change the score (window 3 gray vs window 11 RGB), so scores from different SSIM settings are not comparable. Timing numbers are only comparable when they cover the same work.

### V0 as a whole: what the learner can credibly claim (2026-11-17)

Built from scratch and measured: nearest, bilinear (loop + vectorized), box and point downsampling, MSE/PSNR/SSIM (global, local, RGB), grayscale and RGB video reconstruction with a recorded baseline. Understood in their own words: many-to-one downsampling means exact recovery is impossible; interpolation smooths rather than restores; PSNR/SSIM are imperfect proxies. Not yet established: sample-position alignment, aliasing and pre-filtering, bicubic, the vectorized code line by line, SSIM-map interpretation, color-space issues. V0 closed on the learner's decision with these carried forward.

## Results to remember (monkey, 2x, vs reference)

RGB box -> bilinear (2026-11-16): MSE 24.67 / 34.21 dB / SSIM 0.9698 (11x11 window; not comparable to the gray SSIM below). Grayscale: box -> bilinear MSE 24.41 / 34.25 dB / SSIM 0.959. Sub -> bilinear 51.72 / 30.99 / 0.950, but this comparison is confounded by alignment: with matched alignment, sub -> bilinear MSE is 18.44 (assistant check). Box -> nearest 41.39 / 31.96. NNV: box 119.74, sub 95.59 MSE. Full table in state.md.

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
- Carried forward from V0 (not covered): alignment lesson, aliasing, bicubic, vectorized-code walkthrough, SSIM/error-map interpretation, linear-light color, SSIM on luminance.
- V1 prerequisites coming up: convolution, kernels, feature maps, neurons, gradient descent, backpropagation, PyTorch (none taught yet).
- Not covered in depth: frequency-domain sampling theory, Gaussian-window SSIM, bicubic derivation, linear-light color processing, CNNs and neural SR training (V1), patches/embeddings/positional encoding/self-attention/multi-head attention and SR transformers such as SwinIR and HAT (V2), temporal reconstruction, motion estimation, temporal attention (V3), hybrids (V4), frame interpolation (V5), real-time integration (V6). These are planned, not completed.

## References

Cite only what has been checked. Do not turn these into reading assignments before the learner needs them.

- SSIM: Z. Wang, A. Bovik, H. Sheikh, E. Simoncelli, "Image Quality Assessment: From Error Visibility to Structural Similarity", IEEE TIP 2004.
- NNV (unverified match to the notebook's source): O. Rukundo, H. Cao, "Nearest Neighbor Value Interpolation", 2012.
- Pillow filter descriptions: https://pillow.readthedocs.io/en/stable/handbook/concepts.html#filters
- NumPy `sliding_window_view`: https://numpy.org/doc/stable/reference/generated/numpy.lib.stride_tricks.sliding_window_view.html
- FSR 2: https://github.com/GPUOpen-Effects/FidelityFX-FSR2
- RIFE: https://github.com/hzwer/ECCV2022-RIFE
