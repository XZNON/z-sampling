# z-Sampling — current state

Checkpoint: 2026-10-29, Asia/Calcutta. Update this document as work progresses.

## Resume point

STOPPED AFTER: implementing a 2x grayscale bilinear upscaler and reaching a real-photo enlargement experiment. The learner has not started the controlled downsample/reconstruction quality experiment. V1 remains in progress. No neural model, temporal system, or frame generator has been built.

The latest discussion distinguished larger dimensions from improved reconstruction quality. The user wants both learning and building, and explicitly rejects having the assistant implement the project while they watch.

## Files actually present at inspection

- `01_neareest_neigh.ipynb`: main learner notebook; preserve its spelling and content.
- `.venv/`: local environment; exact active notebook interpreter not verified in this handoff.
- `assets/`: diagrams/results including nearestNei.png, lerp.jpeg, bi-lerp1.png, biLerpMax.png, biLerpMaxx.png, 4.png, dfs.png, bilerpMonkey.png, original.png, upscaled.png.
- `ginue-pigs/`: monkey.jpg, cat.jpg, bw_monkey.png. These paths are intentional existing names.
- Git was initialized and pushed at the user's request after handoff preparation. Public repository: https://github.com/XZNON/z-sampling . Local branch `main` tracks `origin/main`. Initial commit: `9e1eed4` (`Start z-Sampling image upscaling learning lab`). Notebook, assets, experiment photos, and `.gitignore` were committed; local context documents and `.venv` remain ignored.
- No dependency manifest existed at inspection. Notebook uses NumPy and Pillow; do not infer exact installed versions in `.venv` from another interpreter.

## Notebook map (zero-based cells at this checkpoint)

- 2: uint8 2x2 array and nearest enlargement via repeat; persisted shapes (2,2), (4,4).
- 4: `lerp(a,b,t)` with ValueError outside [0,1].
- 8: `biLerp(a,b,c,d,tx,ty)`.
- 12,14,17: four-corner interpolation at 3x3,4x4,1000x1000; uses floats and export conversion.
- 19: `findNeigh(x,y,height,width)`: clamp position, floor to int, bound next neighbor, compute fractional offsets.
- 20: synthetic 3x4 example; persisted interpolated value 80.0.
- 21: learner's coordinate mapping explanation.
- 22: `upscale_bilinear(image,scale)`, readable loop implementation, grayscale 2D arrays and integer scales.
- 24: `upscale_bilinear_vectorized(image,scale)` using meshgrid, gathered neighbors, and array operations.
- 25: persisted loop output `(6,8)` with values reproduced below.
- 27: vectorized scale-1 check with persisted True.
- 29: loads `ginue-pigs/monkey.jpg`, converts to L, casts to float; no saved output in notebook JSON.
- 30: vectorized photo upscale at scale 2 and round/clip/uint8 conversion; saving `doublt.png` is commented out; no saved output in notebook JSON.

Cell numbers can change. Locate by function names and content on subsequent sessions.

## Known correct synthetic result

Input:
```
  0  40  80 120
 60 100 140 180
120 160 200 240
```
Loop 2x output:
```
  0  10  30  50  70  90 110 120
 15  25  45  65  85 105 125 135
 45  55  75  95 115 135 155 165
 75  85 105 125 145 165 185 195
105 115 135 155 175 195 215 225
120 130 150 170 190 210 230 240
```

## Immediate next teaching step

1. Briefly acknowledge the completed upscaler and the distinction between resolution and detail. Do not restart the introductory lessons.
2. Explain the controlled experiment BEFORE giving an exercise: original/reference -> reduce each dimension by 2 -> use the learner's upscaler to restore reference dimensions -> compare with reference.
3. Ask the learner to choose a small existing photo (monkey.jpg is available) and inspect dimensions. Guide conversion to grayscale if needed. Preserve the original separately. For odd dimensions, teach one explicit even-size crop policy before halving; do not silently alter the reference.
4. Explain why reduction needs filtering, then choose one documented baseline (e.g. Pillow BOX at integer 2x reduction). Give one small coding step at a time. The assistant should not write the whole experiment.
5. After reconstruction works, compare matched crops at the same display scale. Then introduce MSE/PSNR. No metric scores for a photo have been computed yet.

The intended first assistant turn after resuming is a brief explanation plus a small task, not implementation, a broad roadmap recap, or automatic execution of all cells.

## Follow-up observations to handle when relevant

- Cell 14 creates `res4` but converts `res` to uint8, likely a copy/paste oversight. Flag gently if revisiting/saving that exercise; do not silently edit it during handoff.
- Vectorized code floors unclamped X/Y, clips indices, then computes tx/ty from unclamped coordinates. At clamped borders, fractions can fall outside [0,1], but duplicated neighbors make the current interpolation yield replicated edges. Do not assert a confirmed wrong output; compare with loop and consider consistent clamping during a later review.
- Vectorized inputs should be floats for this lesson; output tests, timings, edge cases, and generic dtype validation have not been thoroughly established.
- Notebook heading calls vectorization nearly production style; that is not evidence of production readiness or broad performance validation.
- Real-photo code is present and the learner reports reaching 2x enlargement, but its last cells have no persisted shape/output. Do not claim to have independently run or visually verified that particular result.

## Handoff preparation actions

Added only context documents and `.gitignore`; did not modify or run the learner's notebook or change its images. Context files are ignored as explicitly requested. Ignoring files does not provide backup or guarantee another session can find them: open the new session in this project and supply the HANDOFF prompt.

Subsequent repository setup: user authorized repository creation/push and explicitly requested public visibility. The initial commit was pushed and visibility set to public. No notebook implementation changes were made. Learning resume point is unchanged.
