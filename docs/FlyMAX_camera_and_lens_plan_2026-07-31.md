# FlyMAX camera, lens, and filter proposal

**Review draft — updated 2026-08-05**

**Decision status:** current order-now proposal, synchronized to the procurement sheet on 2026-08-05. The cameras and functional requirements are stable; the S-mount production lenses will be selected from a bounded test kit after measurement at the real working distances and at 850 nm. Obtain a final live stock and delivery confirmation before submitting the PO.

**Scope:** define stable camera names, document the published FlyMAX baseline, select the six physical cameras for five functional roles, and propose optics for 850 nm monochrome imaging and visible-light color inspection.

## 1. Published FlyMAX camera baseline

The published system is the reference, but the new design replaces functions rather than copying it one-for-one. Published-source details and the derivation of non-telecentric FOV values are documented in [FlyMAX_Cameras_and_Lenses.md](FlyMAX_Cameras_and_Lenses.md) and ultimately in [Woo et al., bioRxiv 2024.08.21.607451](https://doi.org/10.1101/2024.08.21.607451). Add both Markdown files to the FlyMAX documentation repository so this relative link resolves there.

| # | Published station / view | Camera and sensor | Resolution | Pixel | Max fps | Shutter | Lens | WD | Object FOV | Sampling |
|---:|---|---|---:|---:|---:|---|---|---:|---:|---:|
| 1 | Platform localization | Imaging Source DMM 22BUC03-ML; MT9V024 mono | 744 × 480 (0.4 MP) | 6.0 µm | 76 | Global | Edmund #64-108, 16 mm f/2 M12 | ≈105 mm [D] | 24.83 × 16.02 mm [D] | 33.4 µm/px |
| 2 | Inspection, lateral | Imaging Source DFK 37BUX250; IMX250 color | 2448 × 2048 (5 MP) | 3.45 µm | 75 | Global | Opto TCLWD250, 2.5× telecentric | 132.3 mm | 3.38 × 2.83 mm [E] | 1.38 µm/px |
| 3 | Inspection, anterior | Imaging Source DFK 37BUX250; IMX250 color | 2448 × 2048 (5 MP) | 3.45 µm | 75 | Global | VS Technology VS-TCH2-110, 2× telecentric | 110.8 mm | 4.22 × 3.53 mm [E] | 1.73 µm/px |
| 4 | Collection/pre-inspection | Imaging Source DMK 37BUX273; IMX273 mono | 1440 × 1080 (1.6 MP) | 3.45 µm | 238 | Global | Edmund #59-780, 35 mm f/2 M12 + FEL0750 [I] | ≈117 mm [D] | 11.57 × 8.68 mm [D] | 8.03 µm/px |
| 5 | Behavior rig, XZ | Imaging Source DMM 37BUX273; IMX273 mono | 1440 × 1080 (1.6 MP) | 3.45 µm | 238 | Global | Edmund #59-780, 35 mm f/2 M12 + FEL0750 [I] | ≈94 mm [D] | 8.41 × 6.31 mm [D] | 5.84 µm/px |
| 6 | Behavior rig, YZ | Imaging Source DMK 37BUX290; IMX290 mono | 1920 × 1080 (2.1 MP) | 2.9 µm | 143 | **Rolling** | Edmund #59-780, 35 mm f/2 M12 + FEL0750 [I] | ≈108 mm [D] | 11.64 × 6.55 mm [D] | 6.06 µm/px |

**[E]** exact from sensor size ÷ telecentric magnification. **[D]** derived from the calibrated `pxPerMM` values in the published FlyMAX software. **[I]** lens assignment inferred from the three lenses, three filters, and three mono/IR stations in the BOM. Datasheet maximum fps is shown; the operating rates used in the paper were not reported.

## 2. Goals and architecture

1. Use stable, role-based names in documentation, code, CAD, calibration files, and Linear issues.
2. Retain two simultaneous cameras on the behavior rig: one for FicTrac ball motion and one for the fly.
3. Use a lightweight downward-looking robot-head camera for final targeting and the head-camera-to-picker calibration.
4. Test two **non-simultaneous** upward-looking stage cameras: an 8.1 MP / 136.7 fps detail option and a 5.0 MP / 205 fps speed option. Both cover a 45–50 mm platform. Do not purchase the previously considered 24.5 MP camera.
5. Replace the two published color inspection cameras with one fixed color inspection camera; the robot-head view supplies the second useful view. Do not add a dedicated pre-inspection camera.
6. Use 850 nm bandpass filters on every monochrome path. Do **not** add an 850 nm filter to the color camera; retain its factory IR-cut window and illuminate it visibly.
7. Treat FOV, focus, modulation transfer, vignetting, exposure, depth of field, and centroid stability at the actual working distance as acceptance measurements—not as facts guaranteed by a web calculator.

The result is **five functional roles and six physical cameras**, because the two stage-up cameras are A/B alternatives and will not run together.

## 3. Proposed camera names and specifications

| Canonical name | Camera | Status | Role | Sensor / acquisition | Interface / mount | Proposed FOV | Official specifications |
|---|---|---|---|---|---|---|---|
| `head_down` | XIMEA xiMU **MU051MG-SY**, mono | Owned | Downward robot-head targeting; include the picker and enough surrounding workspace for robust approach | IMX568; 2472 × 2064; 5.1 MP; 2.74 µm; global; 48.9 fps full frame; 1224 × 1032 up to 168.6 fps with 2×2 binning or decimation at 8 bit | USB3; S-mount | **≈60–70 mm horizontal** within the 50–75 mm mechanical range; final field is an acceptance measurement | [Product](https://www.ximea.com/products/miniature-compact/ximu-smallest-industrial-usb-cameras/sony-imx568-usb3-mono-ximu-smallest-camera) · [technical manual (PDF)](https://www.ximea.com/downloads/documents-downloads/ximu-smallest-usb-industrial-cameras_technical-manual-dwl_manual-pdf) |
| `behavior_ball` | LUCID Phoenix **PHX004S-MS-IX**, mono | Owned | FicTrac view of the complete 9 mm ball with surrounding margin | IMX287; 720 × 540; 0.4 MP; 6.9 µm; global; 286 fps | GigE PoE; S-mount; ix Industrial | **12–15 mm horizontal** at an available **50–75 mm WD**; approximately 17–21 px/mm and 432–540 px across the ball | [Phoenix family](https://thinklucid.com/phoenix-machine-vision/) · [datasheet (PDF)](https://dce9ugryut4ao.cloudfront.net/Phoenix-datasheet.pdf) |
| `behavior_fly` | LUCID Triton2 **TRT016S-MC**, mono | Owned; exact suffix to confirm from label/invoice | Fixed fly-position view on behavior rig; robustness to Z variation is important | IMX273; 1440 × 1080; 1.6 MP; 3.45 µm; global; 166.3 fps | 2.5GigE PoE; **C-mount** | **≈4.97 mm horizontal at 110 mm WD** with the selected 1× telecentric | [Product](https://thinklucid.com/product/triton2-16-mp-imx273/) · [Triton2 datasheet (PDF)](https://dce9ugryut4ao.cloudfront.net/Triton2-Datasheet.pdf) |
| `stage_up_detail` | LUCID Atlas10 **ATX081S-MC**, mono | **Buy** | Higher-detail upward imaging through the platform | IMX536; 2840 × 2840; 8.1 MP; 2.74 µm; global; 136.7 fps | 10GigE PoE+; C-mount; RDMA capable | 45 × 45 or 50 × 50 mm | [Product](https://thinklucid.com/product/atlas10-8-1mp-imx536/) · [datasheet (PDF)](https://dce9ugryut4ao.cloudfront.net/Atlas10-datasheet.pdf) · [Edmund #18-476](https://www.edmundoptics.com/p/lucid-vision-labs-atlas10-atx081s-mc-sony-imx536-81mp-monochrome-camera/45502/) |
| `stage_up_fast` | LUCID Atlas10 **ATX051S-MC**, mono | **Buy** | Faster alternative for the same upward stage position | IMX537; 2448 × 2048; 5.0 MP; 2.74 µm; global; 205 fps at 10-bit ADC | 10GigE PoE+; C-mount; RDMA capable | 53.8 × 45 or 59.8 × 50 mm, so the circular platform fits on the short sensor axis | [Product](https://thinklucid.com/product/atlas10-5-mp-imx537/) · [datasheet (PDF)](https://dce9ugryut4ao.cloudfront.net/Atlas10-datasheet.pdf) · [Edmund #18-478](https://www.edmundoptics.com/p/lucid-vision-labs-atlas10-atx051s-mc-sony-imx537-50mp-monochrome-camera/45504/) |
| `inspection_color` | LUCID Atlas10 **ATX051S-CC**, color | **Buy** | Fixed color inspection/phenotyping | IMX537; 2448 × 2048; 5.0 MP; 2.74 µm; global; 205 fps | 10GigE PoE+; C-mount; factory IR-cut window | 3.36 × 2.81 mm with 2× telecentric lens | [Product](https://thinklucid.com/product/atlas10-5-mp-imx537/) · [datasheet (PDF)](https://dce9ugryut4ao.cloudfront.net/Atlas10-datasheet.pdf) · [Edmund #18-477](https://www.edmundoptics.com/p/lucid-vision-labs-atlas10-atx051s-cc-sony-imx537-50mp-color-camera/45503/) |

The June 2026 purchase record confirms one Phoenix IMX287 S-mount mono camera and one Triton2 IMX273 mono camera. Confirm the Triton2's exact SKU from the physical label before this document becomes the configuration authority.

## 4. Closest published-to-proposed comparisons

This is deliberately not a one-for-one replacement table. A green-field design can improve the task metric while trading a datasheet maximum that the task does not need.

| Proposed role | Closest published camera | New versus published | Geometric sampling comparison | Assessment |
|---|---|---|---|---|
| `head_down` | Pre-inspection IMX273, 1.6 MP, 238 fps, global | 5.1 MP, 48.9 fps full / 168.6 fps with 2×2 binning or decimation | ≈24–28 µm/px for a 60–70 mm horizontal field, versus 8.03 µm/px for the published much tighter field | The new view intentionally trades local sampling for workspace and picker visibility. A 2.5 mm fly remains roughly 88–103 px long at full resolution. Evaluate cropped and binned modes only after the nominal field is established. |
| `behavior_ball` | Behavior YZ IMX290, 2.1 MP, 143 fps, rolling | 0.4 MP, 286 fps, global | 16.7–20.8 µm/px for 12–15 mm FOV, versus 6.06 µm/px | Much coarser spatial sampling, but still 432–540 px across a 9 mm ball; speed doubles and rolling-shutter distortion is removed. This is a task-specific FicTrac trade, not a general imaging upgrade. |
| `behavior_fly` | Behavior XZ IMX273, 1.6 MP, 238 fps, global | Same sensor format/resolution; 166.3 fps; housed 2.5GigE camera | 3.45 µm/px geometrically at 4.97 mm FOV, versus 5.84 µm/px | About 1.7× finer sensor sampling. Peak fps is lower, and the selected true-telecentric lens has a fixed f/20.9 aperture; usable resolution, 850 nm exposure, and Z tolerance therefore require bench measurement. |
| `stage_up_detail` | Platform MT9V024, 0.4 MP, 76 fps, global | 8.1 MP, 136.7 fps, global | 15.85–17.61 versus 33.4 µm/px while covering a much larger field | Clear upgrade: ≈20× pixels, 1.8× fps, and ≈1.9–2.1× finer linear sampling. |
| `stage_up_fast` | Platform MT9V024, 0.4 MP, 76 fps, global | 5.0 MP, 205 fps, global | 21.97–24.41 versus 33.4 µm/px while covering a much larger field | Clear speed upgrade: ≈12.5× pixels, 2.7× fps, and ≈1.4–1.5× finer sampling. It is the speed end of the A/B comparison. |
| `inspection_color` | Anterior inspection IMX250, 5 MP, 75 fps, global, 2× | 5 MP, 205 fps, global, 2× | 1.37 versus 1.73 µm/px geometrically | Same pixel count, 2.7× the maximum frame rate, and slightly finer sensor sampling. The selected inexpensive CompactTL lens does not fully resolve the camera, so geometric sampling must not be presented as achieved optical resolution. One fixed view replaces two published color views by design. |

## 5. Recommended lenses and alternatives

### 5.1 Lens decision table

| Camera | Ideal lens and starting aperture | Predicted geometry | Alternatives and rationale |
|---|---|---|---|
| `head_down` | Nominal first mount: [Commonlands CIL062 6.2 mm, no IR-cut](https://commonlands.com/products/no-distortion-6mm-m12-lens), fixed **f/4** | Thin-lens estimate: ≈59 mm horizontal at 60 mm WD and ≈70 mm at 70 mm WD, or ≈24–28 µm/px | [Commonlands CIL085 8.2 mm f/4.4](https://commonlands.com/products/8mm-low-distortion-m12-lens) is the higher-sampling, tighter-field fallback but does not reach the full 60–70 mm target within 50–75 mm WD. Edmund 6, 8, 12.5, 17.5, and 25 mm f/4 lenses map the trade space, while Commonlands CIL142 14.4 mm and CIL121 21.8 mm provide finite-conjugate short-WD anchors. |
| `behavior_ball` | Nominal first comparison: [Edmund #69-262, 8 mm f/4 Blue Series](https://www.edmundoptics.com/p/8mm-fl-f4-blue-series-m12-mu-videotrade-imaging-lens/23591/) versus matched [#83-951, 8 mm f/8](https://www.edmundoptics.com/p/8mm-fl-f8-blue-series-m12-mu-videotrade-imaging-lens/27052/) | Same nominal focal length and prescription isolate the aperture/DOF/exposure trade. Actual FOV is not accepted from the catalog and must be measured at 50–75 mm WD. | Commonlands CIL085 8.2 mm is the vendor comparator. CIL142 14.4 mm f/4.1 remains the finite-conjugate corrective option if the 8 mm lenses are too wide; thin-lens geometry predicts CIL142 is near the 12–15 mm target at 50–55 mm WD. CIL121 21.8 mm is the long-WD endpoint. |
| `behavior_fly` | [Edmund #63-731, 1× 110 mm WD CompactTL](https://www.edmundoptics.com/p/1x-110mm-wd-compacttl-telecentric-lens/18464/), fixed **f/20.9** | Exact ≈4.97 × 3.73 mm FOV, 3.45 µm/px, at 110 mm WD | This is the inexpensive, fixed, true-telecentric baseline. Edmund lists ±1.2 mm DOF at its stated contrast criterion. The lens is VIS-coated, so 850 nm transmission, refocus, exposure, contrast, and usable Z range remain bench acceptance tests. [#63-730, 0.75×](https://www.edmundoptics.com/p/0.75x-110mm-wd-compacttl-telecentric-lens/18462/) is the fallback only if ≈6.62 mm horizontal FOV is needed. |
| `stage_up_detail` | [Edmund #67-714, 16 mm C Series VIS–NIR](https://www.edmundoptics.com/p/16mm-c-series-vis-nir-fixed-focal-length-lens/22382/), provisional; start at **f/2.8** | ≈108.5 mm WD for 45 mm square FOV; ≈118.8 mm WD for 50 mm | Correct 2/3-inch/2.74 µm coverage and 425–1000 nm coating. Test f/4 and f/5.6 for DOF after the camera choice is confirmed. |
| `stage_up_fast` | [Edmund #27-554, 12 mm C Series VIS–NIR](https://www.edmundoptics.com/p/12mm-c-vis-nir-series-fixed-focal-length-lens/53828/), provisional; start near **f/2.8** | Thin-lens estimate ≈108.2 mm WD for 53.8 × 45 mm FOV; ≈118.9 mm WD for 59.8 × 50 mm | Keeps nearly the same optical-path length as the 8 MP/16 mm configuration; supports 1/1.8-inch, 2.74 µm sensors and 425–1000 nm. Its front filter-mount strategy still needs checking. |
| `inspection_color` | [Edmund #63-732, 2× 110 mm WD CompactTL](https://www.edmundoptics.com/p/2X-110mm-WD-CompactTL-Telecentric-Lens/18465/), fixed **f/33** | Exact ≈3.36 × 2.81 mm geometric FOV and 1.37 µm/camera-pixel on the IMX537 | Best order-now match for FOV, 110 mm clearance, compactness, price, and stock—but **not an ideal resolution match**. Edmund rates it for 4.5 µm pixels / 2.3 MP, while the camera has 2.74 µm pixels / 5 MP. See the comparison below. |

### 5.2 Inspection telecentric resolution decision

The 2× CompactTL is a **geometry-first baseline**. The camera oversamples it: the 1.37 µm/object-pixel number describes the sensor grid, not resolved object detail. Edmund's 4.5 µm sensor-pixel rating corresponds to about 2.25 µm per lens-rated object sample at 2×, and the fixed f/33 aperture further limits high-frequency contrast. This may still be entirely adequate for whole-fly inspection, color, silhouette, body-part classification, and robust centroid/orientation measurements, but it should not be described as a full 5 MP optical system until measured.

| Edmund alternative | Optical match | WD / mechanics | Price and current availability | Decision |
|---|---|---|---|---|
| [#58-431, 2× SilverTL](https://www.edmundoptics.com/p/20x-silvertl-telecentric-lens/15294/) | Rated for 2.74 µm pixels; variable f/6–f/22 | 75 mm WD; 280 g; 45 mm diameter | $1,295; **Contact Us**, not orderable from confirmed stock | Best moderate-price resolution match, but fails the current stock requirement and shortens the WD by 35 mm. |
| [#88-348, 2× in-line SilverTL](https://www.edmundoptics.com/p/20x-in-line-illumination-silvertl-telecentric-lens/30609/) | Rated for 2.74 µm pixels; f/10–f/22 | 75 mm WD; 314 g; unnecessary in-line port | $2,465; 10 in stock at the 2026-07-31 check | Optically stronger, but too expensive and wrong WD for the present design. |
| [#59-837, 2× 120 mm WD in-line telecentric](https://www.edmundoptics.com/p/2x-120mm-wd-in-line-illumination-telecentric-lens/16498/) | Rated for 4.2 µm pixels; fixed f/32.5 | 120 mm WD; compact 17 mm body | $1,520; 1 in stock at the 2026-07-31 check | Nearly the same resolution mismatch as #63-732, with an $815 premium and an unused in-line port. |
| [#65-028, 2× high-resolution in-line telecentric](https://www.edmundoptics.com/p/20x-high-resolution-inline-telecentric-lens/19709) | Rated for 2.6 µm pixels; f/8–closed | 100 mm WD; 250 mm long; 54 mm diameter | $4,205; 1 in stock at the 2026-07-31 check | True high-resolution match, but physically large and not justified for the first inspection build. |

**Current decision:** retain #63-732 in the base order and test it with a slanted-edge target and real flies. If it cannot deliver the required color/structural contrast, revisit a higher-resolution telecentric after stock and mechanics are known. There is no currently confirmed-in-stock Edmund option that simultaneously preserves approximately 110 mm WD, matches 2.74 µm pixels, stays compact, and remains near the CompactTL price.

### 5.3 Nominal S-mount choices and bounded test kit

The production design does **not** require all of these lenses. The order is a compact shared test kit for two lightweight S-mount cameras whose final close-focus geometry is not guaranteed by catalog data. We will mount the nominal choices first, then use the remaining focal lengths only to bracket the field and resolve failures quickly.

| Role | Nominal first mount | Most relevant alternatives | What determines the winner |
|---|---|---|---|
| `head_down` | Commonlands CIL062, 6.2 mm f/4 | CIL085 8.2 mm f/4.4; Edmund 6 and 8 mm f/4; tighter 12.5, 17.5, and 25 mm options if the required field changes | Achieve a usable 60–70 mm horizontal field at 50–75 mm WD, include the picker and approach region, retain acceptable corners at 850 nm, and leave enough pixels on the fly for robust centroiding. |
| `behavior_ball` | Edmund 8 mm f/4 | Matched Edmund 8 mm f/8; Commonlands CIL085 8.2 mm; CIL142 14.4 mm finite-conjugate; CIL121 21.8 mm at the long-WD end | Show the complete 9 mm ball with a measured 12–15 mm horizontal field, no clipping during motion, sufficient edge-of-ball focus, short exposure at 286 fps, and stable FicTrac output. |

The **8 mm f/4 versus 8 mm f/8 pair** is the only controlled aperture experiment: the matched focal length and prescription let us attribute the difference primarily to aperture. The broader kit is a focal-length and vendor/prescription comparison, not an aperture series. In particular, the ordinary Edmund Blue Series lenses are cataloged for WD ≥150 mm and visible wavelengths, so their ability to focus and preserve contrast at 50–75 mm and 850 nm is a bench question. Commonlands CIL142 and CIL121 are included as finite-conjugate anchors with a stronger short-WD rationale.

#### Ordered S-mount sweep

| Vendor | Lens | Primary reason it is in the kit |
|---|---|---|
| Edmund | 6 mm f/4 | Wide-field cross-check for `head_down`; may vignette the XIMEA because its listed image circle is smaller than the sensor diagonal. |
| Edmund | 8 mm f/4 | Nominal `behavior_ball` lens and second wide-field point for `head_down`. |
| Edmund | 8 mm f/8 | Matched aperture/DOF comparison for FicTrac. |
| Edmund | 12.5 mm f/4 | Mid-field bridge between the 8 mm and 17.5 mm choices. |
| Edmund | 17.5 mm f/4 | Useful intermediate field and bridge to the finite-conjugate Commonlands lenses. |
| Edmund | 25 mm f/4 | Tight endpoint; verifies whether the earlier small-field concept has any value. |
| Commonlands | CIL062 6.2 mm f/4 | Nominal `head_down` choice for the 60–70 mm goal. |
| Commonlands | CIL085 8.2 mm f/4.4 | Independent 8 mm-class comparator with a full XIMEA-sized image circle. |
| Commonlands | CIL142 14.4 mm f/4.1 | Finite-conjugate mid-field anchor; most plausible corrective choice if an 8 mm lens is too wide for the 12–15 mm FicTrac requirement. |
| Commonlands | CIL121 21.8 mm f/5.9 | Finite-conjugate tight endpoint and likely FicTrac option near 75 mm WD. |

The Edmund [#68-228 S-mount spacer kit](https://www.edmundoptics.com/p/s-mount-brass-spacer-ring-kit-2-of-each-size/22914/) is part of the test kit. It provides two each of 0.25, 0.5, 1.0, 1.25, 1.5, and 2.0 mm rings for controlled lens-to-sensor spacing. Record the exact spacer stack for every result; the focus lock ring alone is not a substitute for this adjustment.

The 4 mm Edmund and 4.4 mm Commonlands lenses are explicitly excluded: they add a very-wide endpoint with higher vignetting/distortion risk but little value after retaining the two approximately 6 mm lenses.

#### Test sequence

1. Build one rigid adjustable fixture that reproduces 50, 55, 60, 65, 70, and 75 mm lens-to-object distance. Use the real camera, 850 nm bandpass filter, illumination, and any protective window used in the final path.
2. For each lens and WD, establish best focus, then record spacer stack, thread position, horizontal and vertical FOV, pixels/mm, usable image circle, and whether focus has safe mechanical margin.
3. Capture the same flat field and resolution/distortion target at fixed illumination. Measure center/corner contrast, vignetting, distortion, exposure, gain, and motion blur at the intended frame rate.
4. Run a Z sweep through the expected object-height tolerance. For the 8 mm f/4 and f/8 pair, compare usable DOF and edge-of-ball focus at matched mean image brightness.
5. Run the task test: centroid/picker visibility for `head_down`, and FicTrac tracking stability with the moving 9 mm ball for `behavior_ball`.
6. Score each configuration against the gates below. Keep the raw images and a one-row result record for every tested configuration so the production choice is traceable.

| Gate | `head_down` | `behavior_ball` |
|---|---|---|
| Geometry | 60–70 mm horizontal field; picker and approach region visible | 12–15 mm horizontal field; full 9 mm ball never clips |
| Focus/field quality | Acceptable center and corners across expected hover-height range | Ball center and rim remain trackable across curvature and motion |
| 850 nm operation | Acceptable contrast without an IR cut; no filter-induced vignetting | Stable surface contrast at 286 fps with acceptable exposure/gain |
| Task metric | Stable fly centroid and picker localization | Stable FicTrac output with low dropouts and no systematic motion bias |
| Mechanical repeatability | Focus can be locked and spacer stack documented | Focus can be locked; chosen WD fits the rig envelope |

Select the simplest lens that passes all gates. Do not select the sharpest center image if it fails field coverage, corners, exposure, or task stability.

### 5.4 Aperture strategy

- `head_down`: begin with the fixed f/4 CIL062. The focal-length sweep is more important than an aperture sweep for this role; evaluate DOF and exposure at the nominal lens before buying additional apertures.
- `behavior_ball`: compare the matched Edmund 8 mm f/4 and f/8 lenses. Prefer f/4 unless f/8 materially improves rim focus, usable Z range, or FicTrac stability without forcing unacceptable exposure, gain, or motion blur. If neither 8 mm lens meets the field requirement, switch focal length before adding more apertures.
- `behavior_fly`: #63-731 is fixed at f/20.9. Measure usable Z range, 850 nm exposure, contrast, and centroid stability; use #63-730 only if the 4.97 mm FOV is too tight.
- `stage_up_fast`: begin at f/2.8, then f/4 and f/5.6. At 850 nm, closing the aperture increases DOF but increases diffraction.
- `stage_up_detail`: f/2.8–f/4 is the likely useful range. f/5.6 is a DOF stress test; the 8.1 MP camera already becomes increasingly diffraction-limited as the aperture closes at 850 nm.
- `inspection_color`: #63-732 is fixed at f/33. Its value is repeatable scale, silhouette, working clearance, compactness, and price—not photon efficiency or full use of the 5 MP sensor. Use bright visible illumination and measure MTF before claiming achieved resolution.

## 6. 850 nm filters

An IR-transparent lens or a “no IR-cut” option is not a spectral filter. Every monochrome path should receive an 850 nm bandpass to reject room light and visible displays. The LUCID color camera has a factory IR-cut window; do not add an external 850 nm filter to it ([LUCID optical-window guidance](https://support.thinklucid.com/knowledgebase/back-focal-distance/)).

| Camera(s) | Selected filter | Mounting plan | Notes |
|---|---|---|---|
| `head_down`, `behavior_ball` | [Commonlands CBP850](https://commonlands.com/products/850nm-bandpass-filter), choose **10 mm circular × 0.3 mm** | One per camera in a lightweight black front holder, or ask Commonlands to bond it into the eventual winning barrel; do not insert it behind the lens unless the focus shift is characterized | Standardize the experimental holder across the two native S-mount paths. Verify angle-dependent CWL shift and vignetting with every candidate lens. |
| `behavior_fly` | Existing [Commonlands CBP850](https://commonlands.com/products/850nm-bandpass-filter) | Put a 10 mm filter in a black front holder ahead of #63-731 | Avoid adding optical spacing behind the telecentric lens. Verify vignetting, stray light, and 850 nm contrast with the finished holder. |
| `stage_up_detail`, `stage_up_fast` | [Edmund #67-853, 850 nm CWL / 40 nm FWHM, 25 mm mounted](https://www.edmundoptics.com/p/850nm-cwl-40nm-fwhm-25mm-mounted-diameter/22520/) | [Edmund #65-800, M25.5 × 0.5 mount](https://www.edmundoptics.com/p/M255-x-05-Mount-for-25mm-Diameter-Filters/20318/) threads onto both provisional C Series lenses | Direct, documented mechanical match for the 16 mm and 12 mm lens threads. Use one filter per lens to avoid disturbing focus/calibration. |
| `inspection_color` | **No external IR filter** | Retain factory IR-cut window | Use visible white/color-balanced illumination so eye and body color remain meaningful. |

For prototyping, two Edmund filters are sufficient because the two stage cameras are not simultaneous; giving each lens its own filter and adapter is preferable to avoid changing focus/calibration between tests.

## 7. Predicted sampling and optical resolution

### 7.1 Definitions and assumptions

For non-telecentric lenses, the first-pass thin-lens estimate is:

```text
magnification m ≈ f / (WD − f)
object FOV = sensor size / m
object sampling = sensor pixel pitch / m
sensor Nyquist interval = 2 × object sampling
object-space Airy diameter ≈ 2.44 × wavelength × f-number / m
```

The **sampling** column is geometric and is the best answer to “µm per pixel.” The Nyquist and Airy columns are theoretical lower bounds, not guaranteed resolved feature sizes. Real performance may be worse because of lens MTF at 850 nm, aberrations, contrast, motion blur, the platform window, mirror flatness, focus error, and demosaicing. Geometric FOV changes negligibly with wavelength after refocus; 850 nm primarily changes focus position, throughput, MTF, and diffraction.

Calculations below assume a 2.5 mm fly. The ball rows instead report pixels across the actual 9 mm ball. The color camera uses 550 nm for its diffraction estimate because it is a visible-light camera.

| Camera / nominal setup | Predicted FOV | Sampling | Pixels across target | Nyquist interval | Ideal Airy diameter | First-order limiting scale |
|---|---:|---:|---:|---:|---:|---:|
| `head_down`: CIL062, 60–70 mm, f/4, 850 nm | ≈59–70 mm horizontal | **23.8–28.2 µm/px** | **≈89–105 px/fly** | 47.6–56.5 µm | ≈72–85 µm | ≈72–85 µm first-order diffraction scale; actual corner quality and 850 nm contrast require measurement |
| `behavior_ball`: Edmund 8 mm, 50 mm, f/4, 850 nm | Thin-lens estimate ≈26.1 × 19.6 mm | **36.2 µm/px** | **≈249 px/9 mm ball** | 72.4 µm | ≈43.6 µm | The estimate is much wider than the 12–15 mm requirement; this is a high-priority geometry check, not a claimed achieved field |
| `behavior_ball`: Edmund 8 mm, 50 mm, f/8, 850 nm | Same nominal geometry | **36.2 µm/px** | **≈249 px/9 mm ball** | 72.4 µm | ≈87.1 µm | Controlled DOF/diffraction/exposure comparison with the f/4 lens; actual close-WD performance is unknown |
| `behavior_ball`: CIL142, 50 mm, f/4.1, 850 nm | ≈12.28 × 9.21 mm | **17.06 µm/px** | **≈528 px/9 mm ball** | 34.11 µm | ≈21.0 µm | Finite-conjugate corrective option if the 8 mm pair is too wide; ample FicTrac sampling |
| `behavior_ball`: 15 mm FOV boundary | 15 × 11.25 mm | **20.83 µm/px** | **≈432 px/9 mm ball** | 41.67 µm | Lens-dependent | Requirement boundary; still ample ball sampling |
| `behavior_fly`: #63-731, 1×, f/20.9, 850 nm | ≈4.97 × 3.73 mm | **3.45 µm/px** | **≈725 px/2.5 mm fly** | 6.90 µm | ≈43.3 µm | Strongly diffraction/contrast limited at the fixed aperture; catalog performance is VIS, so 850 nm exposure, contrast, and usable DOF require bench acceptance |
| `stage_up_detail`: 16 mm, 45 mm FOV, f/2.8, 850 nm | 45 × 45 mm | **15.85 µm/px** | **≈158 px/fly** | 31.69 µm | 33.58 µm | ≈34 µm, balanced sampling/diffraction |
| `stage_up_detail`: 16 mm, 50 mm FOV, f/2.8, 850 nm | 50 × 50 mm | **17.61 µm/px** | **≈142 px/fly** | 35.21 µm | 37.31 µm | ≈37 µm, balanced sampling/diffraction |
| `stage_up_fast`: 12 mm, 45 mm short-axis FOV, f/2.8, 850 nm | 53.79 × 45 mm | **21.97 µm/px** | **≈114 px/fly** | 43.95 µm | 46.57 µm | ≈47 µm, balanced sampling/diffraction |
| `stage_up_fast`: 12 mm, 50 mm short-axis FOV, f/2.8, 850 nm | 59.77 × 50 mm | **24.41 µm/px** | **≈102 px/fly** | 48.83 µm | 51.72 µm | ≈52 µm, balanced sampling/diffraction |
| `inspection_color`: #63-732, 2×, f/33, 550 nm | 3.36 × 2.81 mm | **1.37 µm/camera px** | **≈1,825 camera px/fly** | 2.74 µm sensor-grid interval | ≈22.1 µm first-order object-space Airy diameter | Camera substantially oversamples the lens; Edmund rates the lens for 4.5 µm sensor pixels / 2.3 MP. Measure MTF and do not equate 1.37 µm/px with resolved detail. |

The two stage cameras make a clean speed/detail comparison: the 8.1 MP camera supplies about 1.39× more linear samples across a fly, while the 5 MP camera supplies about 1.50× the full-frame rate. Both remain substantially better sampled and faster than the published platform camera.

## 8. Order-now kit

The practical plan is to place one coordinated set of vendor orders now. The post-delivery measurements select the two production S-mount lenses; the additional small lenses are an inexpensive shared qualification kit, not intended for simultaneous deployment.

### Edmund Optics order

- Cameras: ATX081S-MC (`stage_up_detail`), ATX051S-MC (`stage_up_fast`), and **ATX051S-CC** (`inspection_color`), one each. Availability is a live-checkout requirement; confirm allocation and delivery at PO.
- Stage optics: #67-714 16 mm VIS–NIR and #27-554 12 mm VIS–NIR, plus two #67-853 850 nm filters and two #65-800 M25.5 filter mounts.
- Telecentric optics: #63-731 1× CompactTL for `behavior_fly` and #63-732 2× CompactTL for `inspection_color`. Both are inexpensive fixed true-telecentric baselines; the inspection lens is a geometry/price/stock choice rather than a full-resolution match to the 5 MP sensor.
- M12 test lenses: one each of #38-012 6 mm f/4, #69-262 8 mm f/4, #83-951 8 mm f/8, #69-264 12.5 mm f/4, #69-265 17.5 mm f/4, and #69-266 25 mm f/4. The f/4 series maps focal length; the matched 8 mm f/4/f/8 pair isolates aperture. Close-WD focus and 850 nm performance are experimental.
- S-mount spacing: one #68-228 brass spacer-ring kit, containing two each of 0.25, 0.5, 1.0, 1.25, 1.5, and 2.0 mm rings. Record the spacer stack with every test result.
- Camera interfaces: one #72-562 dual-port 10G PoE+ HCA; three #13-749 M12-to-RJ45 data/PoE cables; **two #17-112 GPIO cables**; and one #17-120 Atlas10 tripod adapter. The two stage-up cameras are swapped serially, so they share the HCA position, GPIO cabling, and qualification mount.
- Platform windows: two #12-193 50 mm × 3.3 mm NIR-coated BOROFLOAT windows (prototype plus spare). One #25-414 50 mm × 1.0 mm NIR-I-coated sapphire window is **optional**, requires a carrier shim, and is excluded from the base-order subtotal.
- Current Edmund base-order subtotal: **$12,205.50** before tax and shipping; the optional sapphire adds $502.44. This is the synchronized 2026-08-05 sheet value and must be reconfirmed at PO.

### Commonlands order

- One each of CIL062-F4.0-M12ANIR (6.2 mm), CIL085-F4.4-M12BNIR (8.2 mm), CIL142-F4.1-M12ANIR (14.4 mm finite-conjugate), and CIL121-F5.9-M12ANIR (21.8 mm finite-conjugate).
- Six CBP850 10 mm filters, eleven M12 lock rings, one cleaning kit, one focus-card set, and one holder/screw set for fixture development.
- Do not add the unpublished CIL159 to this order without a published/orderable part number and aperture data; the 14.4 and 17.5 mm samples already bracket that focal length.
- Current Commonlands subtotal: **$345.00** before tax and shipping; reconfirm at checkout.

There is no VS Technology order in the current plan because pricing and availability could not be confirmed through the selected procurement path.

Explicitly do not order the 24.5 MP Atlas10, the VariMagTL, the $300 C-mount 850 nm filter previously paired with it, the 4 mm Edmund or 4.4 mm Commonlands lenses, a third stage camera, or a dedicated pre-inspection camera.

## 9. Bench acceptance and calibration consequences

For every camera/lens/filter configuration, record:

1. Actual WD, sensor mode, FOV, pixels/mm, and usable circular field.
2. Center and corner contrast at 850 nm (visible light for `inspection_color`), with a slanted-edge or equivalent resolution target at the relevant object height.
3. Focus and contrast through the final NIR-coated platform window and 45° mirror, where applicable.
4. Vignetting and flat-field uniformity with the selected filter and illumination geometry.
5. Exposure, gain, motion blur, and dropped-frame rate at the intended acquisition rate.
6. Usable Z range and centroid/orientation stability across it for each aperture.
7. Distortion-calibration residuals and pixel-to-robot residuals over the whole usable field.

Changing camera, lens, aperture, focus, lens adapter, filter position, mirror, or platform window requires at least a calibration verification. A change that alters geometry requires a new intrinsic calibration and camera-to-robot registration.

## 10. Decisions made after receipt

The measurements select only:

1. The production focal length for `head_down`, with CIL062 6.2 mm f/4 as the nominal first mount.
2. The production lens for `behavior_ball`, beginning with the matched Edmund 8 mm f/4/f/8 comparison and moving to CIL142 or CIL121 if required to meet the measured 12–15 mm field.
3. Whether the fixed 1× `behavior_fly` lens has sufficient 850 nm contrast and usable Z range, or whether the 0.75× CompactTL fallback is needed for more field margin.
4. Which Atlas10 camera provides the better speed/detail trade for the stage.
5. Whether the inexpensive 2× CompactTL provides enough resolved color/structural detail for inspection; if not, specify a higher-resolution telecentric from measured MTF and mechanical constraints.

The camera roles, target FOVs, vendors, and prototype order are otherwise fixed by this proposal.

*Commonlands CIL159 note: Delilah Jacobsen identified this unpublished 16 mm finite-conjugate lens as a possible future short-WD option. It is not part of the current order because no public part number, aperture set, or orderability information is available.*
