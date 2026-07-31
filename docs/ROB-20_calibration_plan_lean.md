# ROB-20 — lean calibration plan

**Goal:** map the bottom camera to the robot, then map the head camera to the physical picker so the robot can correct the final fly-COM-to-picker offset before each strike.

Source: slides 8–9 of `FlyMax_073026.pptx`.

## Commissioning: bottom camera ↔ robot

1. **Camera intrinsics**
   - At setup, with no glass plate.
   - Image a dot-grid target in multiple poses, including rotations and modest tilts.

2. **Glass surface map**
   - Detect the glass surface using very slow robot travel at the center and eight surrounding locations.
   - Fit `Z_surface(x,y)`.

3. **Bead correspondence**
   - Pick a retroreflective bead, or use a bead-tipped needle.
   - Image it with the bottom camera on a `5 × 5` XY grid at three heights above the surface: `0.5 / 1.5 / 3.0 mm`.
   - During the same procedure, image the bead or picker axis with the head camera at the operating hover heights.

4. **Fit and verify**
   - Fit the bottom-camera ↔ robot projection.
   - Validate it on held-out points, for example 15 of the 75 observations.
   - Store the picker-axis location in the head camera as a height-dependent `p_picker(z)`.
   - Return the bead to its nest.

This procedure should absorb lens distortion, mirror handedness, glass/refraction, stage pose or remount error, thermal bow, and height parallax.

## Runtime targeting

1. **Bottom camera:** detect fly COM and convert it to robot XY.
2. **Robot hover:** move above the selected fly at a calibrated height.
3. **Head camera:** measure fly COM in the calibrated picker axes.
4. **Lateral correction:** align fly COM with the picker axis.
5. **Strike:** descend using `Z_surface + fly height`, then verify capture from the vacuum signal.

## Routine verification

- At startup and after a stage remount, image the bead at the center and four quadrant locations at one validated height.
- Run the full `5 × 5 × 3` calibration if the verification residuals fail.
- Recalibrate after moving the fixed camera or mirror; changing the lens, focus, iris, filter, window, or optical stack; disturbing the head camera; changing hover height; or replacing/rotating the picker.
- After tool replacement, repeat the head-camera → picker calibration and refine the lateral offset from successful-pick residuals.

## Open simplifications to test

1. **Detect the needle tip directly.** Determine whether the bottom camera can reliably localize the picker tip without a bead. The expected image may be a bright annulus with a darker center. If repeatable across height and illumination, this could replace the bead for some calibration and verification steps.
2. **Use a fixed tool-calibration spot.** Place a small mechanically referenced calibration location near one corner of the bottom camera's field, outside the active fly area. Use it to recalibrate each newly installed tool quickly.

The fixed spot can update the local head-camera → picker/tool offset. It should not replace field-wide bottom-camera ↔ robot verification after the stage, camera, mirror, or optical stack has moved.

