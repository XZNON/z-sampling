# Resume z-Sampling (V0 complete; start V1)

Copy the following into a new session opened at `C:\Users\XZNON\z-sampling`:

---

Continue my z-Sampling learning project in `C:\Users\XZNON\z-sampling`. V0 (classical reconstruction) is finished. Do not repeat it. We are starting V1 (CNN super resolution).

First read `AGENTS.md`, `state.md` (top section: "V0 closure" and "Resume point"), `learning.md`, and `plan.md`. The notebooks are `01_neareest_neigh.ipynb` (grayscale + RGB images), `02_video_upscaling.ipynb` (video), and `03_CNN_V1.ipynb` (V1; created 2026-11-18, empty on disk at the last check). Inspect them only as needed. Do not blindly run every cell (AGENTS.md lists the slow ones); the video notebook needs `ginue-pigs/gta5.mp4`, which is gitignored.

About me: professional in Python, intermediate in ML, RTX 3050 with 4 GB VRAM. I write the code; you teach, review and help me debug. One concept and one small exercise at a time. Explain a calculation before asking me to implement it. Don't build the lab for me, don't restart completed lessons, and don't commit or push unless I ask.

Method: understand the mathematics -> implement from scratch -> test -> experiment -> benchmark -> investigate improvements. No blindly copying pretrained models.

V0 done: nearest, lerp/bilerp, half-pixel mapping, bilinear (loop + vectorized, grayscale + RGB), box/point downsampling, MSE/PSNR/SSIM (global, local, per-channel RGB), SSIM and error maps, grayscale and RGB video reconstruction with OpenCV.

V0 baseline (RGB video, first 30 frames of 1080x608 GTA V clip, box 2x down -> bilinear up, CPU): MSE 86.53897, PSNR 28.79463 dB, SSIM 0.955572 (min 0.941393, max 0.969917; 11x11 uniform window, per-channel mean), 194.58 ms/frame = 5.139 FPS (down + up only). Treat as a baseline, not a general claim. Visible blur around text, HUD, edges, fine texture. RGB image (monkey 736x736, box 2x -> bilinear): MSE 24.67, PSNR 34.21 dB, SSIM 0.9698 (11x11), channels nearly equal.

Known caveats (details in `state.md`): gray and RGB video SSIM used different windows (3 vs 11) so they are not comparable; gray timing included metrics; RGB cell 29 channel names are BGR; the vectorized RGB upscaler truncates to uint8 instead of rounding (about 0.2 MSE); `01` cell 122 redefined my loop SSIM helpers with `uniform_filter` versions under the same names; sample-position alignment, aliasing, bicubic, and my line-by-line understanding of the vectorized code were NOT done in V0 (carried forward; the alignment lesson should happen before I build V1 training pairs).

Next: V1 starts with convolution as a sliding weighted sum, bridged from my windowed-mean SSIM code (box kernel). Explain it, then give me one exercise: predict a small convolution by hand, write a loop version, check a 3x3 box kernel against my window mean, then try edge and blur kernels. After that: feature maps, learnable kernels, a neuron, gradient descent, backpropagation, PyTorch (check hardware/software when we get there; do not install a full ML stack unasked).

Roadmap (details in plan.md; milestones, not fixed specs): V0 classical (COMPLETE) -> V1 CNN SR (NEXT) -> V2 Vision Transformer SR (a confirmed requirement: I build a small ViT, study SwinIR and HAT, compare with CNN and classical) -> V3 temporal video reconstruction -> V4 hybrids and optimization -> V5 frame interpolation -> V6 small real-time demo. Before editing a notebook on disk, ask me to save and close it.

---

# Superseded handoff (2026-11-16, V0.1 era)

Kept for history. Its resume point (alignment lesson after the V0.1 review) is now part of the carried-forward list in `state.md`; the V0.x numbering is closed.

## Old text

Copy the following into a new session opened at `C:\Users\XZNON\z-sampling`:

---

Continue my z-Sampling learning project in `C:\Users\XZNON\z-sampling`.

First read `AGENTS.md`, `state.md`, `learning.md`, and `plan.md`, then inspect the relevant cells in `01_neareest_neigh.ipynb`. The notebook may be ahead of the notes.

I'm professional in Python, intermediate in ML, and have an RTX 3050 with 4 GB VRAM. I want to write the project myself while you teach, review, and help me debug. Explain one concept and give one small exercise at a time. Don't build the next feature for me or restart already completed lessons.

Done so far: nearest-neighbor exercises, lerp/bilerp, neighbor lookup with border clamping, pixel-center mapping, and a grayscale bilinear upscaler (loop + vectorized). I also built the controlled experiment: reference -> 2x box downsample / point sub-sample -> reconstruct (bilinear, nearest, NNV-style) -> compare with MSE, PSNR, and my own SSIM (global + local map). Box -> bilinear scores best so far: MSE 24.41, PSNR 34.25, SSIM 0.959 with a 3x3 box window.

On 2026-11-11 a review found bugs, and at my request the assistant fixed them: SSIM plot cell, `calculate_mse` raise, checkerboard variable, NNV mode step, fast local SSIM via `sliding_window_view`. The notebook on disk runs top to bottom in about 5s. My editor still showed the old cells, so I ran the slow loop-SSIM version and thought it crashed. I was told to reload from disk and Restart + Run All. If I haven't confirmed that yet, ask me for my cell 86 numbers first. If you need to edit the notebook file, ask me to close it first and reload it afterwards.

Most important open issue: the sub-sampling comparison is confounded. Where my point-sampled pixels sit doesn't match where my upscaler assumes they sit.

Next: ask whether I want the two assistant-written vectorized functions explained. Then teach sample-position alignment as one concept with one exercise: I predict where sub-sampled pixel i sits, adjust the mapping, and recompute MSE. After that, demonstrate aliasing with the fixed checkerboard. Don't give me answers before I predict. Don't skip to neural networks, bicubic, or optimization.

Roadmap (renumbered 2026-11-16; details in plan.md): V0 classical reconstruction = V0.1 finish grayscale image evaluation (current) -> V0.2 RGB images -> V0.3 grayscale video -> V0.4 RGB video -> V0.5 final evaluation and baseline. Then V1 CNN SR -> V2 Vision Transformers for SR (I build a small ViT from scratch; SwinIR, HAT) -> V3 temporal video reconstruction (temporal attention, VRT) -> V4 hybrids and optimization -> V5 frame interpolation -> V6 small real-time demo. My old "V1 lab" / "V.1 complete" commit means the current V0.1. Preserve the plan and record progress in the context files as we go. Don't commit or push unless I ask.

---
