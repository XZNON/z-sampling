# Resume z-Sampling

Copy the following into a new session opened at `C:\Users\XZNON\z-sampling`:

---

Continue my z-Sampling learning project in `C:\Users\XZNON\z-sampling`.

First read `AGENTS.md`, `state.md`, `learning.md`, and `plan.md`, then inspect the relevant cells in `01_neareest_neigh.ipynb`. These context files are intentionally gitignored, so read them directly rather than relying only on Git's file list.

I'm professional in Python, intermediate in ML, and have an RTX 3050 with 4 GB VRAM. I want to write the project myself while you teach, review, and help me debug. Explain one concept and give one small exercise at a time. Don't build the next feature for me or restart already completed lessons.

We have completed nearest-neighbor exercises, lerp/bilerp, local neighbor lookup with border clamping, pixel-center coordinate mapping, and a working grayscale bilinear upscaler. The 3x4 -> 6x8 synthetic result is correct. A vectorized variant and a real-photo 2x experiment are in the notebook. We are currently at the point where I have enlarged an image 2x, not trained a model or completed the whole upscaling lab.

My latest question was whether enlarging the image actually increases quality. We clarified that bilinear estimates smoother transitions but doesn't recover missing detail, and that my 1000x1000 four-corner experiment was also enlargement rather than new information.

Resume with a controlled reconstruction experiment: keep an original grayscale photo as reference, downsample it 2x, reconstruct to reference dimensions with my upscaler, then inspect matching crops and later learn MSE/PSNR. Explain the experiment and downsampling choice before asking me to implement it. Start with one small task. Don't skip to neural networks, bicubic implementation, or optimization.

The longer roadmap is V1 spatial upscaling lab -> V2 small neural SR -> V3 temporal reconstruction -> V4 frame interpolation -> V5 small real-time demo. Preserve the plan and record progress in the context files as we go.

---
