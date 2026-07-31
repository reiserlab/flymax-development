# FlyMAX camera, lens, and filter proposal

**Review draft — 2026-07-31**  
**Decision status:** current order-now proposal, synchronized to the procurement sheet on 2026-07-31; obtain a final live stock and delivery confirmation before submitting the PO.  
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
| `head_down` | XIMEA xiMU **MU051MG-SY**, mono | Owned | Downward robot-head targeting; final hover and picker-offset calibration | IMX568; 2472 × 2064; 5.1 MP; 2.74 µm; global; 48.9 fps full frame; 1224 × 1032 up to 168.6 fps with 2×2 binning or decimation at 8 bit | USB3; S-mount | ≈10.3 × 8.6 mm at 55 mm WD; adjustable within the 50–75 mm mechanical range | [Product](https://www.ximea.com/products/miniature-compact/ximu-smallest-industrial-usb-cameras/sony-imx568-usb3-mono-ximu-smallest-camera) · [technical manual (PDF)](https://www.ximea.com/downloads/documents-downloads/ximu-smallest-usb-industrial-cameras_technical-manual-dwl_manual-pdf) |
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
| `head_down` | Pre-inspection IMX273, 1.6 MP, 238 fps, global | 5.1 MP, 48.9 fps full / 168.6 fps with 2×2 binning or decimation | ≈4.17 versus 8.03 µm/px at the proposed FOV | About 1.9× finer sampling full-frame; slower at full resolution. The 2×2 modes are ≈8.35 µm/output px—close to the published spatial sampling—and should be evaluated for the final approach loop. |
| `behavior_ball` | Behavior YZ IMX290, 2.1 MP, 143 fps, rolling | 0.4 MP, 286 fps, global | 16.7–20.8 µm/px for 12–15 mm FOV, versus 6.06 µm/px | Much coarser spatial sampling, but still 432–540 px across a 9 mm ball; speed doubles and rolling-shutter distortion is removed. This is a task-specific FicTrac trade, not a general imaging upgrade. |
| `behavior_fly` | Behavior XZ IMX273, 1.6 MP, 238 fps, global | Same sensor format/resolution; 166.3 fps; housed 2.5GigE camera | 3.45 µm/px geometrically at 4.97 mm FOV, versus 5.84 µm/px | About 1.7× finer sensor sampling. Peak fps is lower, and the selected true-telecentric lens has a fixed f/20.9 aperture; usable resolution, 850 nm exposure, and Z tolerance therefore require bench measurement. |
| `stage_up_detail` | Platform MT9V024, 0.4 MP, 76 fps, global | 8.1 MP, 136.7 fps, global | 15.85–17.61 versus 33.4 µm/px while covering a much larger field | Clear upgrade: ≈20× pixels, 1.8× fps, and ≈1.9–2.1× finer linear sampling. |
| `stage_up_fast` | Platform MT9V024, 0.4 MP, 76 fps, global | 5.0 MP, 205 fps, global | 21.97–24.41 versus 33.4 µm/px while covering a much larger field | Clear speed upgrade: ≈12.5× pixels, 2.7× fps, and ≈1.4–1.5× finer sampling. It is the speed end of the A/B comparison. |
| `inspection_color` | Anterior inspection IMX250, 5 MP, 75 fps, global, 2× | 5 MP, 205 fps, global, 2× | 1.37 versus 1.73 µm/px geometrically | Same pixel count, 2.7× the maximum frame rate, and slightly finer sensor sampling. The selected inexpensive CompactTL lens does not fully resolve the camera, so geometric sampling must not be presented as achieved optical resolution. One fixed view replaces two published color views by design. |

## 5. Recommended lenses and alternatives

### 5.1 Lens decision table

| Camera | Ideal lens and starting aperture | Predicted geometry | Alternatives and rationale |
|---|---|---|---|
| `head_down` | Leading candidate: [Commonlands CIL121 21.8 mm finite-conjugate, no IR-cut](https://commonlands.com/products/telephoto-22mm-m12-lens), **f/5.9** | At 55 mm WD: ≈10.32 × 8.61 mm, 4.17 µm/px | Test CIL121 at f/2.8 and f/8, plus Edmund 17.5 and 25 mm candidates. The Commonlands lens is the lower-risk starting point because it explicitly supports useful performance down to about 50 mm; the Edmund lenses supply an independent design/vendor comparison but are specified for WD ≥150 mm. |
| `behavior_ball` | Mount near the short end of the available **50–75 mm WD** range and use [Commonlands CIL142 14.4 mm finite-conjugate, no IR-cut](https://commonlands.com/products/telephoto-14mm-m12-lens), initially f/2.6 | Thin-lens prediction: ≈12.28 mm horizontal at 50 mm WD and ≈14.00 mm at 55 mm—correct for a 9 mm ball with margin. At 60 mm it grows to ≈15.73 mm and is already outside the preferred range. | Compare CIL142 f/4.1 for DOF. If mechanics force the camera to the far end, the CIL121 21.8 mm gives ≈12.12 mm at 75 mm WD. Avoid choosing an ordinary 17–18 mm M12 lens for the middle of the range unless its close-focus and 850 nm performance are verified. |
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

### 5.3 Work through the S-mount lenses one camera at a time

The earlier eight-lens list is withdrawn pending this review. The two native S-mount cameras have different targets and should not receive one combined focal-length kit by default.

#### First: `behavior_ball` / FicTrac

Confirmed requirement: image a 9 mm ball with a **12–15 mm horizontal field**, leaving 1.5–3 mm of lateral margin on each side. On the 720-pixel-wide Phoenix, this produces:

| Horizontal FOV | Sampling | Pixels across 9 mm ball | Margin on each side |
|---:|---:|---:|---:|
| 12 mm | 16.67 µm/px | 540 px | 1.5 mm |
| 13.5 mm | 18.75 µm/px | 480 px | 2.25 mm |
| 15 mm | 20.83 µm/px | 432 px | 3.0 mm |

This is ample sampling for FicTrac; full-ball visibility, surface contrast, exposure time, and lack of clipping are more important than filling the frame.

The Phoenix can be placed at 50–75 mm WD, but a fixed lens does not preserve the same FOV while its distance changes. The useful finite-conjugate combinations are:

| WD | CIL142, 14.4 mm predicted FOV | CIL121, 21.8 mm predicted FOV | Interpretation |
|---:|---:|---:|---|
| 50 mm | 12.28 mm | 6.43 mm | CIL142 is an excellent match |
| 55 mm | 14.00 mm | 7.57 mm | CIL142 is an excellent match |
| 60 mm | 15.73 mm | 8.71 mm | CIL142 is just wider than preferred |
| 65 mm | 17.45 mm | 9.85 mm | Neither is ideal |
| 70 mm | 19.18 mm | 10.99 mm | Neither is ideal |
| 75 mm | 20.90 mm | 12.12 mm | CIL121 becomes a good match |

Therefore, the simplest and lowest-risk design is to set the camera at approximately **50–55 mm** and use the CIL142. If mechanical clearance favors 75 mm, use the CIL121 instead. A roughly 17–18 mm lens would span the desired field around 60–70 mm, but Commonlands' published finite-conjugate series does not provide that intermediate focal length, so it would introduce a less-qualified lens prescription.

For the preferred 50–55 mm mounting position, the sensible first candidates are:

| Priority | Lens | Predicted horizontal FOV | Why test it |
|---:|---|---:|---|
| 1 | [Commonlands CIL142-F2.6-M12ANIR, 14.4 mm](https://commonlands.com/products/telephoto-14mm-m12-lens) | ≈12.28 mm at 50 mm WD; ≈14.00 mm at 55 mm | Finite-conjugate design, fast aperture for 286 fps, correct FOV range, 9.3 mm image circle covers the Phoenix. |
| 2 | CIL142-F4.1-M12ANIR | Same FOV | Controlled DOF/exposure comparison using the same prescription. |
| 3 | CIL142-F5.2-M12ANIR | Same FOV | Maximum-DOF comparison. At 850 nm its diffraction Airy diameter is ≈10.8 µm, still below the Phoenix's 13.8 µm Nyquist sample period. |
| 4 | [Edmund #58-205, 12.5 mm f/2.5 Blue Series, no IR-cut](https://www.edmundoptics.com/p/125mm-fl-f25-blue-series-m12-mu-videotrade-imaging-lens/15121/) | ≈14.90 mm at 50 mm WD | Independent vendor/design comparator with a geometrically useful focal length. It is high risk because Edmund specifies ≥150 mm WD and 400–700 nm; the latest check showed 20+ in stock. |

Do not buy the previous 25 mm FicTrac candidate on the basis of the old 5 mm FOV target. The 21.8 mm CIL121 is too tight at 50–55 mm, but remains the correct finite-conjugate fallback if the camera must be placed near 75 mm.

#### Next: `head_down`

Retain the CIL121 21.8 mm aperture series as the leading hypothesis for the approximately 10 mm head-camera field, but review it only after the FicTrac choice is settled.

### 5.4 Aperture strategy

- `head_down`: f/5.9 is the proposed production starting point because head-height variation matters; compare f/2.8 and f/8 on the same lens prescription.
- `behavior_ball`: compare CIL142 f/2.6, f/4.1, and f/5.2. Prefer f/2.6 if it keeps the full curved ball acceptably sharp because short exposure matters at 286 fps; use f/4.1 or f/5.2 if edge-of-ball focus/DOF improves without unacceptable exposure, gain, or motion blur.
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
| `head_down`: CIL121, 55 mm, f/5.9, 850 nm | 10.32 × 8.61 mm | **4.17 µm/px** | **≈599 px/fly** | 8.35 µm | 18.64 µm | ≈19 µm, diffraction-limited at this aperture; f/2.8 would recover detail at lower DOF |
| `behavior_ball`: CIL142, 50 mm, f/2.6, 850 nm | ≈12.28 × 9.21 mm | **17.06 µm/px** | **≈528 px/9 mm ball** | 34.11 µm | 13.33 µm | ≈34 µm, sensor-sampling limited; ample for FicTrac |
| `behavior_ball`: 15 mm FOV boundary | 15 × 11.25 mm | **20.83 µm/px** | **≈432 px/9 mm ball** | 41.67 µm | Lens-dependent | Requirement boundary; still ample ball sampling |
| `behavior_fly`: #63-731, 1×, f/20.9, 850 nm | ≈4.97 × 3.73 mm | **3.45 µm/px** | **≈725 px/2.5 mm fly** | 6.90 µm | ≈43.3 µm | Strongly diffraction/contrast limited at the fixed aperture; catalog performance is VIS, so 850 nm exposure, contrast, and usable DOF require bench acceptance |
| `stage_up_detail`: 16 mm, 45 mm FOV, f/2.8, 850 nm | 45 × 45 mm | **15.85 µm/px** | **≈158 px/fly** | 31.69 µm | 33.58 µm | ≈34 µm, balanced sampling/diffraction |
| `stage_up_detail`: 16 mm, 50 mm FOV, f/2.8, 850 nm | 50 × 50 mm | **17.61 µm/px** | **≈142 px/fly** | 35.21 µm | 37.31 µm | ≈37 µm, balanced sampling/diffraction |
| `stage_up_fast`: 12 mm, 45 mm short-axis FOV, f/2.8, 850 nm | 53.79 × 45 mm | **21.97 µm/px** | **≈114 px/fly** | 43.95 µm | 46.57 µm | ≈47 µm, balanced sampling/diffraction |
| `stage_up_fast`: 12 mm, 50 mm short-axis FOV, f/2.8, 850 nm | 59.77 × 50 mm | **24.41 µm/px** | **≈102 px/fly** | 48.83 µm | 51.72 µm | ≈52 µm, balanced sampling/diffraction |
| `inspection_color`: #63-732, 2×, f/33, 550 nm | 3.36 × 2.81 mm | **1.37 µm/camera px** | **≈1,825 camera px/fly** | 2.74 µm sensor-grid interval | ≈22.1 µm first-order object-space Airy diameter | Camera substantially oversamples the lens; Edmund rates the lens for 4.5 µm sensor pixels / 2.3 MP. Measure MTF and do not equate 1.37 µm/px with resolved detail. |

The two stage cameras make a clean speed/detail comparison: the 8.1 MP camera supplies about 1.39× more linear samples across a fly, while the 5 MP camera supplies about 1.50× the full-frame rate. Both remain substantially better sampled and faster than the published platform camera.

## 8. Order-now kit

The practical plan is to place one coordinated set of vendor orders now. The kit intentionally contains several finite-conjugate M12 apertures; the post-delivery measurements select the production lens/aperture rather than deciding whether to place the prototype order.

### Edmund Optics order

- Cameras: ATX081S-MC (`stage_up_detail`), ATX051S-MC (`stage_up_fast`), and **ATX051S-CC** (`inspection_color`), one each. The 8.1 MP mono and 5 MP color units each showed one unit in stock during the live 2026-07-31 check; confirm allocation and delivery at PO.
- Stage optics: #67-714 16 mm VIS–NIR and #27-554 12 mm VIS–NIR, plus two #67-853 850 nm filters and two #65-800 M25.5 filter mounts.
- Telecentric optics: #63-731 1× CompactTL for `behavior_fly` and #63-732 2× CompactTL for `inspection_color`. Both are inexpensive fixed true-telecentric baselines; the inspection lens is a geometry/price/stock choice rather than a full-resolution match to the 5 MP sensor.
- M12 cross-check lenses: one #58-205 12.5 mm f/2.5 Blue Series for `behavior_ball` and one #69-266 25 mm f/4 Blue Series for `head_down`. These are bounded vendor/design comparators, not a general Blue-Series kit; close-WD focus and 850 nm performance are experimental.
- Camera interfaces: one #72-562 dual-port 10G PoE+ HCA; three #13-749 M12-to-RJ45 data/PoE cables; **two #17-112 GPIO cables**; and one #17-120 Atlas10 tripod adapter. The two stage-up cameras are swapped serially, so they share the HCA position, GPIO cabling, and qualification mount.
- Platform windows: two #12-193 50 mm × 3.3 mm NIR-coated BOROFLOAT windows (prototype plus spare). One #25-414 50 mm × 1.0 mm NIR-I-coated sapphire window is **optional**, requires a carrier shim, and is excluded from the base-order subtotal.
- Current Edmund base-order subtotal: **$11,649.00** before tax and shipping; the optional sapphire adds $502.44.

### Commonlands order

- `head_down`: CIL121-F2.8-M12ANIR, CIL121-F5.9-M12ANIR, and CIL121-F8.0-M12ANIR, one each.
- `behavior_ball`: CIL142-F2.6-M12ANIR, CIL142-F4.1-M12ANIR, and CIL142-F5.2-M12ANIR, one each.
- Six CBP850 10 mm filters, eight M12 lock rings, one cleaning kit, one focus-card set, and one holder/screw set for fixture development.
- Add one or more CIL159 16 mm samples only if Commonlands confirms that they are ready to order, available without IR cut, and supplied in useful aperture variants.
- Current Commonlands subtotal excluding CIL159: **$521.50** before tax and shipping.

There is no VS Technology order in the current plan because pricing and availability could not be confirmed through the selected procurement path.

Explicitly do not order the 24.5 MP Atlas10, the VariMagTL, the $300 C-mount 850 nm filter previously paired with it, a broad Edmund Blue-Series M12 kit, a third stage camera, or a dedicated pre-inspection camera.

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

1. The production aperture for CIL121 and CIL142.
2. Whether the CIL159 is a better Phoenix/FicTrac choice than CIL142 at the final mounting distance.
3. Whether the fixed 1× `behavior_fly` lens has sufficient 850 nm contrast and usable Z range, or whether the 0.75× CompactTL fallback is needed for more field margin.
4. Which Atlas10 camera provides the better speed/detail trade for the stage.
5. Whether the inexpensive 2× CompactTL provides enough resolved color/structural detail for inspection; if not, specify a higher-resolution telecentric from measured MTF and mechanical constraints.

The camera roles, target FOVs, vendors, and prototype order are otherwise fixed by this proposal.

*Commonlands CIL159 note: Delilah Jacobsen identified this unpublished 16 mm finite-conjugate lens as another short-WD option. If its no-IR-cut version and apertures are orderable, it is especially interesting for the Phoenix at approximately 55–65 mm WD (estimated 12.1–15.2 mm horizontal FOV). CIL250, CIL350, and CIL051 remain lower priority because Commonlands warned of field curvature at short WD.*
