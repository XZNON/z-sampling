# z-Sampling: instructions for future sessions

## Purpose and working directory

Work in `C:\Users\XZNON\z-sampling`. This is a learning-by-building project inspired by DLSS: classical image and video upscaling (V0), CNN super resolution (V1), Vision Transformer super resolution (V2), temporal video reconstruction (V3), hybrids/optimization (V4), then frame interpolation (V5) and eventually a small real-time demo (V6). Numbering changed on 2026-11-16; see `plan.md` for the old -> new mapping. The goal includes the learner's understanding, not only working software. This is not a reproduction of NVIDIA's proprietary models.

Read `state.md`, `learning.md`, and `plan.md` before teaching or modifying the project. `HANDOFF.md` provides a compact restart prompt. Then inspect the relevant notebook cells; these notes may lag behind actual work.

## Learner and collaboration rules

- Professional Python, intermediate ML; GPU is RTX 3050 with 4 GB VRAM. Graphics/sampling fundamentals are being learned from scratch.
- User explicitly said: "I dont want you to build while I sit. I need to learn bro". Treat this as a tutoring project. The user writes the implementation.
- Give one coherent concept and one small exercise at a time. Explain what a calculation does and why it is needed before asking for its implementation.
- Review the user's code and outputs concretely. Supply hints or small corrections first; provide direct examples when requested. Do not withhold an explanation or needed syntax in the name of teaching.
- Do not implement the next stage, create a finished lab, refactor the notebook, rename files, train models, or build a UI without the user's request. Documentation maintenance and explicitly requested setup are fine.
- Do not assume understanding or completion from "done" when the next step depends on an unseen result. Ask for the relevant result or offer a small check without repeating completed lessons.
- If the learner asks a question, pause progression and address it. Do not introduce several coordinate conventions or tasks at once.
- Keep the tone friendly, direct, and technically accurate. Distinguish dimension increases, smoothness, restoration, and detail reconstruction. Avoid claiming recovered detail from bilinear interpolation.
- The loop implementation is the understandable reference. Vectorization exists, but do not assume it has been understood or thoroughly verified. Do not describe it as production-ready merely because a notebook heading does.
- No sub-agents or parallel teaching tracks unless the user asks for them.

## Current boundary

Update 2026-11-17: V0 (classical image and video reconstruction, grayscale and RGB) is complete by the learner's decision; V1 (CNN super resolution) is next and not started. See `state.md` "V0 closure" for the baseline, limitations, discrepancies, and carried-forward items (alignment, aliasing, bicubic, vectorized-code review). The paragraph below describes the earlier V0.1 boundary and is kept for history. Video work lives in `02_video_upscaling.ipynb`; the source clip `ginue-pigs/gta5.mp4` is gitignored.

As of 2026-11-11 the learner has built the controlled original -> downsample (box and point) -> reconstruct (bilinear, nearest, NNV-style) -> compare experiment. They measure it with their own MSE, PSNR, and SSIM (global + local). Do not restart at NumPy basics, and do not skip to neural networks. Next: sample-position alignment between downsampler and upscaler (the current sub-sampling comparison is confounded), then aliasing with the checkerboard demo. See `state.md` for the exact resume point. Explain sampling loss and anti-aliasing as they become relevant; interpret scores only with the confounds in view.

## Files and preservation

- Main work: `01_neareest_neigh.ipynb` (existing spelling; preserve it).
- `assets/` contains lesson diagrams and output images; `ginue-pigs/` contains experiment photos. Preserve both and their names.
- `.venv/` is the local Python environment. Use explicit paths/workdir; do not assume terminal activation.
- Do not use the earlier assistant-created lab under `Documents/Codex/.../outputs/z-Sampling` as the learner's work or project source. That was an abandoned assistant-led start.
- Context documents (this file, `HANDOFF.md`, `state.md`, `learning.md`, `plan.md`) were committed by the learner in `f7d6fed` and are tracked in the public repo. Do not delete them. Git is on `main`, tracking `origin/main` at https://github.com/XZNON/z-sampling (public, as explicitly requested). Do not make commits or pushes unless requested.
- Preserve the notebook's cells, saved outputs, attachments, and the learner's experiments. Do not execute every cell blindly in Jupyter: cell 17 is a million-element loop, and the loop SSIM takes about 14s per full image. Commented-out `.save(...)` lines write images if uncommented; `01` cell 106 is NOT commented out and writes `RGB_BiLerp.png` to the working directory. In `02`, cells 22 and 25 re-encode all 564 frames and overwrite the output `.mp4` files, and every video cell needs the gitignored `ginue-pigs/gta5.mp4`. To verify, run cells in order with `.venv/Scripts/python.exe` outside Jupyter (about 5s with the fast SSIM).
- The learner usually has the notebook open in their editor. Before editing the `.ipynb` on disk, ask them to save and close it. Afterwards, ask them to reload from disk (VS Code: `File: Revert File`) and Restart + Run All. Otherwise the open editor keeps the old cells, and saving it can silently revert your edits.
- On Windows, set `PYTHONIOENCODING=utf-8` when printing notebook text from Python; the markdown contains characters cp1252 cannot encode.

## Continuity maintenance

At meaningful checkpoints or session end, update `state.md` with what was actually completed, observed outputs, open questions, and one next exercise. Update `learning.md` with concepts actually covered and remaining confusion; update `plan.md` checkboxes only with evidence. Keep all four context documents consistent, including the compact `HANDOFF.md` restart summary when the resume point changes. Do not replace honest uncertainty with claims of mastery.

These are continuity instructions, not permission to progress through exercises without the learner.
