# z-Sampling — learning record

## Learner context

Professional Python; intermediate ML; RTX 3050, 4 GB VRAM. Wants fundamentals and hands-on model building, not an assistant-built product. Prefers small explanations tied to their own code. Coordinate mapping needed several explanations; revisit its purpose or derive one sample when needed rather than assuming fluency.

## Concepts covered and evidence

### Images, types, and resolution

- Grayscale is a 2D sample array; RGB adds channels. NumPy indexing is `[row, col]` = `[y, x]`; shape is `(height, width)` for grayscale.
- Original exercise: `[[0, 64], [128, 255]]`. Shape `(2,2)` means four pixels. A 2x increase along both axes yields `(4,4)` and four times the pixel count.
- `np.uint8` stores 0..255; `np.int8` is signed -128..127 and was an early mistake.
- Repeat horizontally with `axis=1`, vertically with `axis=0`. `repeats=4` repeats each element four times; it does not mean target width 4.
- Nearest neighbor copies samples, creating blocks without additional observed information.

### Lerp and bilerp

- `lerp(a,b,t) = (1-t)*a + t*b`; between 0 and 64, quarter/mid/three-quarter values are 16, 32, 48.
- Notebook `lerp` now raises ValueError for t outside [0,1].
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

### Neighbor lookup and mapping

- For input position x=2.3,y=1.7, floor gives left/top indices 2,1; right/bottom are 3,2; tx=0.3,ty=0.7.
- Clamp x,y to the valid input center range BEFORE floor in `findNeigh`. Cast indices to ints.
- Actual signature: `findNeigh(x,y,height,width)`, NOT width,height. Return order: x0,y0,x1,y1,tx,ty.
- Use width for x limits; height for y limits. Call with keywords when debugging dimension order.
- For height=3,width=4 and x=3.2,y=-0.25, expected return is `(3,0,3,1,0,0)`.
- Mapping used in the upscaler: `x=(col+0.5)/scale-0.5`, `y=(row+0.5)/scale-0.5`.
- Meaning: output center -> same location expressed in input coordinates -> sample there. Formula selects a LOCATION; bilerp estimates the VALUE at that location.
- Half-pixel convention aligns full image boundaries. `linspace(0,1,N)` in the four-corner exercise aligns endpoint centers instead. Do not conflate these conventions or imply there is only one valid convention.
- 2->4 mapping before clamping: [-0.25,0.25,0.75,1.25]; after clamping: [0,0.25,0.75,1].

### General grayscale upscaler

- User implemented output allocation, nested loops, coordinate mapping, neighbor lookup, bilerp, and storage.
- Earlier bug: column variable `c` was overwritten by bottom-left brightness. Renamed output loop variables to `row,col`.
- On `[[0,40,80,120],[60,100,140,180],[120,160,200,240]]`, sample at x=1.25,y=0.5 gives 80.
- 2x output shape `(6,8)` and first row `[0,10,30,50,70,90,110,120]` were reported correctly.
- Scale-1 equality reported True. Notebook explicitly persists a True check for the vectorized implementation; conversation also records successful identity after wrapping the loop function.
- A vectorized function exists in the notebook; its derivation and performance claims are not established learning achievements. Review later.

## Current conceptual question

User asked whether resizing actually improves quality and whether keeping dimensions fixed is the desired goal. Clarified:

- Resizing changes dimensions; bilinear smooths transitions but does not uniquely recover lost detail.
- Restoration can reduce noise/blur at unchanged dimensions.
- Super resolution reconstructs at higher dimensions, trying to estimate finer detail. Learned detail can be incorrect.
- Traditional supersampling renders higher then downsamples; super resolution reconstructs from lower-resolution input. DLSS-style temporal inputs add evidence beyond a single image.
- Next experiment keeps the reference and reconstruction dimensions equal by first shrinking the reference. This creates a controlled way to see losses and compare reconstruction methods.

## Not yet covered in depth

Downsampling filters/anti-aliasing, frequency-domain sampling theory, MSE/PSNR, bicubic derivation, RGB/linear-light processing, neural SR training, temporal reconstruction, motion estimation, interpolation models, and real-time integration. These are planned, not completed.

References for later study: Pillow filter descriptions https://pillow.readthedocs.io/en/stable/handbook/concepts.html#filters ; FSR 2 https://github.com/GPUOpen-Effects/FidelityFX-FSR2 ; RIFE https://github.com/hzwer/ECCV2022-RIFE . Do not turn these into reading assignments before the learner needs them.
