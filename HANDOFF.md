# Resume z-Sampling

Copy the following into a new session opened at `C:\Users\XZNON\z-sampling`:

---

Continue my z-Sampling learning project in `C:\Users\XZNON\z-sampling`.

First read `AGENTS.md`, `state.md`, `learning.md`, and `plan.md`, then inspect the relevant cells in `01_neareest_neigh.ipynb`. The notebook may be ahead of the notes.

I'm professional in Python, intermediate in ML, and have an RTX 3050 with 4 GB VRAM. I want to write the project myself while you teach, review, and help me debug. Explain one concept and give one small exercise at a time. Don't build the next feature for me or restart already completed lessons.

Done so far: nearest-neighbor exercises, lerp/bilerp, neighbor lookup with border clamping, pixel-center mapping, and a grayscale bilinear upscaler (loop + vectorized). I also built the controlled experiment: reference -> 2x box downsample / point sub-sample -> reconstruct (bilinear, nearest, NNV-style) -> compare with MSE, PSNR, and my own SSIM (global + local map). Box -> bilinear scores best so far: MSE 24.41, PSNR 34.25, SSIM 0.959 with a 3x3 box window.

On 2026-11-11 a review found bugs, and at my request the assistant fixed them: SSIM plot cell, `calculate_mse` raise, checkerboard variable, NNV mode step, fast local SSIM via `sliding_window_view`. The notebook on disk runs top to bottom in about 5s. My editor still showed the old cells, so I ran the slow loop-SSIM version and thought it crashed. I was told to reload from disk and Restart + Run All. If I haven't confirmed that yet, ask me for my cell 86 numbers first. If you need to edit the notebook file, ask me to close it first and reload it afterwards.

Most important open issue: the sub-sampling comparison is confounded. Where my point-sampled pixels sit doesn't match where my upscaler assumes they sit.

Next: ask whether I want the two assistant-written vectorized functions explained. Then teach sample-position alignment as one concept with one exercise: I predict where sub-sampled pixel i sits, adjust the mapping, and recompute MSE. After that, demonstrate aliasing with the fixed checkerboard. Don't give me answers before I predict. Don't skip to neural networks, bicubic, or optimization.

The longer roadmap is V1 spatial upscaling lab -> V2 small neural SR -> V3 temporal reconstruction -> V4 frame interpolation -> V5 small real-time demo. Preserve the plan and record progress in the context files as we go. Don't commit or push unless I ask.

---
