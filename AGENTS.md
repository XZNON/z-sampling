# z-Sampling: instructions for future sessions

## Purpose and working directory

Work in `C:\Users\XZNON\z-sampling`. This is a learning-by-building project inspired by DLSS: spatial upscaling, learned super resolution, temporal reconstruction, then frame interpolation and eventually a small real-time demo. The goal includes the learner's understanding, not only working software. This is not a reproduction of NVIDIA's proprietary models.

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

The learner has reached a working 2x grayscale bilinear upscaler. Do not restart at NumPy basics, and do not skip to neural networks. Next: a controlled original -> downsample -> reconstruct -> compare experiment. Explain sampling loss and anti-aliasing as they become relevant; teach quantitative evaluation before interpreting scores.

## Files and preservation

- Main work: `01_neareest_neigh.ipynb` (existing spelling; preserve it).
- `assets/` contains lesson diagrams and output images; `ginue-pigs/` contains experiment photos. Preserve both and their names.
- `.venv/` is the local Python environment. Use explicit paths/workdir; do not assume terminal activation.
- Do not use the earlier assistant-created lab under `Documents/Codex/.../outputs/z-Sampling` as the learner's work or project source. That was an abandoned assistant-led start.
- Context documents are intentionally ignored by `.gitignore` at the user's request. Ignored does not mean backed up; do not delete them. Git is initialized on `main`, tracking `origin/main` at https://github.com/XZNON/z-sampling (public, as explicitly requested). Do not publish local context documents or make further commits/pushes unless requested.
- Preserve the notebook's cells, saved outputs, attachments, and the learner's experiments. Do not execute every cell blindly: some create million-element loop experiments or write images.

## Continuity maintenance

At meaningful checkpoints or session end, update `state.md` with what was actually completed, observed outputs, open questions, and one next exercise. Update `learning.md` with concepts actually covered and remaining confusion; update `plan.md` checkboxes only with evidence. Keep all four context documents consistent, including the compact `HANDOFF.md` restart summary when the resume point changes. Do not replace honest uncertainty with claims of mastery.

These are continuity instructions, not permission to progress through exercises without the learner.
