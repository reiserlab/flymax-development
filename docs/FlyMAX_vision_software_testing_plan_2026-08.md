# FlyMAX vision software: Atlas10 bring-up and tracking-module testing plan

**Draft 2026-08-08 — for review.** Companion to the [FlyMAX camera, lens, and filter proposal](FlyMAX_camera_and_lens_plan_2026-07-31.md). Linear: **ROB-21** (atlas-bench pipeline bring-up, Phases 0–3), **ROB-16** (follow mode, Phases 3–4), **ROB-13** (target mode, Phase 5), **ROB-22** (pickability pilot, Phase 6), **ROB-12** (camera order + optical acceptance).

**Scope:** the software side of the picking-platform vision system — bringing up the new LUCID Atlas10 cameras on the new Windows workstation, benchmarking the acquisition → GPU processing path, and building/testing the two tracking modes needed for the robot-picking demos. Optical acceptance (FOV, MTF, vignetting, 850 nm contrast) is covered by the camera plan and ROB-12; this document covers everything from the sensor interface to the data the robot consumes.

---

## 1. Goals: two demo modes

Both modes run on the upward-looking stage camera (`stage_up_detail` or `stage_up_fast`), looking through the platform window at flies on the 45–50 mm plate.

| | **Follow mode** (smooth pursuit) | **Target mode** (go-to-pick) |
|---|---|---|
| Task | Track one fly continuously | Detect all flies (~30–35), select pick candidates |
| Output | Center-of-mass (x, y) + body orientation θ, per frame | Fly count + ranked top-3 pick candidates with coordinates and stationarity |
| Rate | As close to camera rate as practical (137/205 fps); **measure, then decide** where to stop optimizing. Robot loop is slower (≥100 Hz possible) and always reads the latest pose. | **50 Hz target**; benchmark the achievable upper bound. Pick cycle takes seconds, so latency is not critical — freshness and reliability are. |
| Consumer | Robot control PC (separate machine, for now) | Robot control PC + pickability-pilot logging |

**Performance philosophy (agreed):** build the pipeline so it *can* sustain full camera rate, but treat September as a measurement exercise — report achieved fps and end-to-end latency at every pipeline stage, then decide how much further optimization the robot demos actually need. A walking fly at ~30 mm/s moves only 0.15–0.3 mm between frames even at 100 Hz.

## 2. System inventory

| Item | Details |
|---|---|
| `stage_up_detail` | LUCID Atlas10 **ATX081S-MC**, 8.1 MP mono (IMX536, 2840 × 2840, 2.74 µm), 136.7 fps, 10GigE PoE+, RDMA-capable. Lens: Edmund #67-714 16 mm VIS–NIR. FOV 45–50 mm → 15.9–17.6 µm/px, **~142–158 px per 2.5 mm fly** (~57–63 px/mm) |
| `stage_up_fast` | LUCID Atlas10 **ATX051S-MC**, 5.0 MP mono (IMX537, 2448 × 2048), 205 fps, 10GigE PoE+, RDMA-capable. Lens: Edmund #27-554 12 mm VIS–NIR. FOV 53.8–59.8 mm wide → 22–24.4 µm/px, **~102–114 px/fly** (~41–45 px/mm) |
| | The two stage cameras are **A/B alternatives** — never simultaneous; they share the HCA port, GPIO cabling, and mount. |
| Filters | Edmund #67-853 850 nm bandpass (40 nm FWHM) per lens, #65-800 mounts. Illumination 850 nm; imaging through NIR-coated BOROFLOAT platform window (#12-193). |
| NIC | Dual-port 10G PoE+ HCA (Edmund #72-562, LUCID). **Verify at bring-up that this is the RDMA-enabled model** — RDMA requires an RDMA-capable HCA plus RDMA firmware on the camera. |
| Vision PC | **Dell Pro Max Tower T2** (FCT2250): NVIDIA **RTX PRO 4500** (Blackwell), 2 × 2 TB NVMe, Windows. Delivery ~Aug 7 2026. |
| Also on hand | 3 × Intel E810-XXVDA2 25 GbE adapters (fiber; for lab networking / a fast vision-PC ↔ robot-PC link if wanted — not for camera PoE+). ATX051S-CC 5 MP color (`inspection_color`) — not part of tracking. |
| Data rate | ATX081S-MC @ 136.7 fps Mono8 ≈ **1.10 GB/s**; ATX051S-MC @ 205 fps ≈ **1.03 GB/s**. Both near-saturate 10GigE. PCIe host→GPU is >20 GB/s, so the GPU copy is never the bottleneck — the NIC→host path is what needs care. |

**Topology (September):** vision PC (cameras + tracker) and robot PC are **separate machines** on the same switch. Tracker publishes over the network; later integration onto one PC is anticipated (see shared-memory prototype, Phase 4).

## 3. Software architecture: three stages

**Stage A — `atlas-bench`** (Phases 0–3): a standalone C++17 console program, no Qt, no BIAS. Arena SDK + CUDA (+ OpenCV for verification only). Each phase adds a pipeline stage behind a command-line switch, and every stage prints a standard metrics block. This is the intermediate test program: it de-risks the transfer/DMA/GPU path before any application code depends on it. It doesn't start from zero: Isabel's Arena backend (§13) already established working Arena C API usage — device enumeration by IP, node-map configuration, the buffer grab/requeue cycle, PFNC↔OpenCV pixel-format mapping, and a `FindArena.cmake` — all directly reusable here.

**Stage B — `flymax-tracker`** (Phases 4–6): the demo service. Reuses the atlas-bench pipeline; adds the two tracking modules, the network publication layer (UDP push + HTTP), summary logging, and the pickability-pilot tooling.

**Stage C — BIAS convergence** (§13): *not a September goal for the GPU path.* An Arena camera backend for BIAS **already exists** — Isabel's work on the [`lucid` branch of isabel-hess/bias](https://github.com/isabel-hess/bias/tree/lucid), verified end-to-end on a LUCID Phoenix — and porting it into FlyVRBIAS is a small, well-defined job that gives us BIAS GUI/recording with the Atlas cameras early. Moving the *GPU* path into BIAS remains substantial and should wait until Stages A/B tell us what the pipeline actually needs.

Rationale: BIAS's per-frame copies, unbounded queues, GUI-thread HTTP, and C++11 baseline make it the wrong place to prototype a >1 GB/s GPU pipeline (§13 for specifics). A small purpose-built harness gives clean measurements and becomes the production tracker service.

---

## 4. Phase 0 — PC and network bring-up

**Purpose:** a verified, documented machine configuration before any measurement.

**Procedure**
1. Windows setup on the Dell T2; latest NVIDIA driver (Studio/Enterprise branch) for the RTX PRO 4500; CUDA Toolkit (13.x); Visual Studio 2022 + CMake.
2. Install the dual-port 10G PoE+ HCA in a PCIe slot with full lane width; install Arena SDK (Windows x64) + ArenaView.
3. Connect one Atlas10 (M12-to-RJ45 cable, PoE+ powered by the HCA). Confirm enumeration in ArenaView.
4. Record camera firmware version. Check for and apply the **RDMA firmware** for Atlas10 if not present; confirm the HCA is the RDMA-enabled LUCID model. (LUCID's RDMA = RoCEv2 / GigE Vision 3.0; on Windows it uses the Network Direct provider.)
5. NIC tuning: 9000-byte jumbo frames, receive buffers maxed, interrupt moderation per LUCID's Windows tuning guide; note every setting in the bring-up log.
6. Stream in ArenaView: both cameras (one at a time), full frame, Mono8, maximum rate, ≥60 s each. Watch the delivered-fps and incomplete-image counters.
7. Repeat for the second Atlas10 and the color camera (enumeration only is fine for the color unit).
8. Optional cross-check: build Isabel's BIAS Arena backend (isabel-hess/bias, `lucid` branch — or the FlyVRBIAS port once ROB-23 lands) and confirm it enumerates and streams the Atlas10 — an independent software path through the same SDK, and it exercises the HTTP control API.

**Metrics:** delivered fps vs datasheet; incomplete/dropped image count; camera temperature at sustained full rate.

**Pass:** both mono cameras stream at datasheet full-frame rate in ArenaView with zero incomplete images over ≥60 s.

**Artifacts:** bring-up log (driver/SDK/firmware versions, NIC settings), ArenaView screenshots of the stream statistics.

## 5. Phase 1 — Acquisition benchmark (`atlas-bench --acquire`)

**Purpose:** measure what the *software* can actually pull from the camera, and quantify what RDMA buys over standard GigE Vision streaming on this machine.

**Procedure**
1. Minimal Arena SDK acquisition loop: configure (Mono8, full frame, max fps, continuous), start stream, requeue buffers, count frames. No processing, no display.
2. Run each condition for a **10-minute soak**: {camera: ATX051S-MC, ATX081S-MC} × {transport: standard GVSP sockets, RDMA} × {pixel format: Mono8, Mono10p}.
3. Record per-frame camera timestamps and host arrival times; detect gaps via the camera's frame counter.
4. Log CPU utilization (total and per-core) alongside.

**Metrics (the standard block every later phase reuses):** sustained fps; dropped-frame count and %; CPU % (total / hottest core); frame-interval jitter (p50/p99/max); resend/incomplete statistics.

**Pass:** ≥ 99.9 % of frames delivered at datasheet rate over 10 min; RDMA shows a clear CPU reduction vs sockets (LUCID advertises reliable 1.2 GB/s with near-zero CPU; verify on our hardware).

**Decision point:** if RDMA on Windows proves flaky, standard sockets at ~1 GB/s with a tuned NIC may still be acceptable — the frame-drop and CPU numbers decide, not the datasheet. (Linux fallback exists but is a last resort; see §15.)

## 6. Phase 2 — DMA → GPU path (`atlas-bench --gpu-copy`)

**Purpose:** land frames in GPU memory with near-zero CPU cost and measured latency.

**Design (Windows):** Arena SDK **user-defined buffers** pointing at a ring of **pinned host memory** (`cudaMallocHost`, 16–32 slots). The NIC (RDMA or sockets) writes into pinned memory; a dedicated CUDA stream issues `cudaMemcpyAsync` host→device per frame. True GPUDirect (NIC→VRAM, bypassing host RAM) is **Linux-only** (`nvidia-peermem`) — not available on Windows, and not needed: an 8 MP Mono8 H2D copy is ~0.4 ms on this machine's PCIe.

**Procedure**
1. Implement the pinned ring + async copy; a CUDA event marks "frame resident on GPU."
2. Re-run the Phase 1 soak matrix with the copy enabled.
3. Measure exposure-to-GPU-resident latency using camera timestamps (after measuring the camera↔host clock offset via LUCID's timestamp latch / PTP if available).

**Metrics:** standard block + H2D bandwidth; exposure→GPU-resident latency (p50/p99); pinned-ring high-water mark.

**Pass:** full camera rate sustained with the GPU copy in the loop; added CPU cost ≈ 0; exposure→GPU-resident p99 < 2 frame periods.

## 7. Phase 3 — GPU kernels + single-fly pipeline (`atlas-bench --track-one`)

**Purpose:** the core measurement of the whole project — can we go sensor → fly pose at camera rate, and what does each stage cost?

**Pipeline (all GPU-resident, matching the ROB-12 sketch: full-frame acquire → locate fly → crop → centroid/orientation):**
1. **Background model:** static background captured at startup (median/mean of N frames of empty or settled plate) with slow running update; stored on GPU.
2. **Subtract + threshold** kernel (fly is dark on backlit 850 nm background) + platform-ROI mask.
3. **Detect:** full-frame reduction to find the fly (block-level foreground statistics); then a small ROI (~512², ~10× the fly) around the last position for subsequent frames; full-frame search only on loss.
4. **Pose:** image moments over the ROI → centroid + orientation (second moments give θ mod π; head/tail resolved by motion direction / brightness asymmetry, as FlyTrack does today). ROI moments are tiny — this stage can even run on CPU after a small D2H; measure both.
5. **CPU reference:** the identical algorithm in OpenCV on the same recorded frames (§11 fixtures) — correctness check (pose agreement) and the CPU-vs-GPU comparison the "do we need the GPU?" question deserves.

**Metrics:** per-stage GPU time (CUDA events); end-to-end exposure→pose latency (p50/p99); sustained fps with tracking on; pose noise on a stationary target (std of centroid, px) — feeds the ROB-12 "centroid stability" acceptance item; CPU-only pipeline fps for comparison.

**Pass:** camera-rate tracking with exposure→pose p99 < 2 frame periods; centroid noise ≪ 1 px on a stationary high-contrast target; GPU vs CPU pose agreement within noise.

## 8. Phase 4 — Follow-mode module (`flymax-tracker --follow`)

**Purpose:** turn the Phase 3 pipeline into the service the robot consumes.

**Publication (agreed hybrid):**
- **UDP push**, one small datagram per processed frame to the robot PC: `{frame, t_capture, t_sent, x, y, theta, quality}` (packed struct or compact JSON — decide with Daniel). The robot samples the latest at its own loop rate.
- **HTTP endpoint** for status, configuration, and one-shot queries — keeping the existing FlyTrack semantics (`get-last-clear-track`-style latest-pose query, keep-alive supported) so current consumers/scripts port over.
- **Pose-sink interface** in the code (`publish(pose)` behind an interface) with UDP, HTTP, and a **shared-memory prototype** implementation (single-writer seqlock ring) — the shm sink is exercised by a local reader process now, and becomes the real path when tracker and robot control later share one PC.

**Test procedure**
1. Loopback + two-PC latency test: UDP receive timestamps on the robot PC vs `t_sent` (NTP/PTP-sync the two machines or measure offset).
2. Live single fly in a dish on the bench setup: 10-min tracking runs; log pose stream; count track losses / full-frame re-searches.
3. Overlay verification: sparse annotated frames (pose drawn on image) via the HTTP endpoint.
4. **Robot-in-loop demo (stretch):** Daniel's gantry — or another available robot, e.g. the Meca 6-axis arm — servos to the streamed pose above the platform; qualitative smooth-pursuit demo + logged pursuit error.

**Pass:** sustained camera-rate pose stream to the second PC; end-to-end (exposure→robot-PC-received) p99 latency documented; ≥10 min live-fly tracking without unrecovered loss.

## 9. Phase 5 — Target-mode module (`flymax-tracker --target`)

**Purpose:** count all flies on the plate and publish ranked pick candidates. Runs at **50 Hz** (decimated from camera rate; benchmark the upper bound of the detection stage for the record).

**Detection & association**
1. Reuse the GPU subtract+threshold output; run connected components with stats (single pass → area, centroid, bbox for every blob). Start with OpenCV `connectedComponentsWithStats` on the (downloaded, sparse) mask; measure; move CC to GPU only if 50 Hz demands it.
2. Size/geometry filter → fly detections (~30–35 expected; area window rejects debris and touching-fly clumps get flagged by area > single-fly max).
3. **Short-window association** across frames (nearest-neighbor gating at 50 Hz is nearly unambiguous — a fast fly moves <1 mm/frame; the repo's `fly_sorter` Hungarian tracker (`src/demo/fly_sorter/identity_tracker.*`) is available if needed). IDs are for continuity of motion statistics, not long-term identity.
4. Per fly, maintain a ~1 s sliding window of positions → `stationary_s` = seconds since the fly last moved > **0.25 mm**.

**Published payload** (both HTTP poll and UDP push, ~10 Hz is plenty; designed around a pick cycle that takes a few seconds):

```json
{
  "t": 1234567.89, "frame": 45678,
  "n_flies": 32, "n_pickable": 3,
  "candidates": [
    {"id": 7, "x_mm": 12.3, "y_mm": -4.1, "size_mm2": 1.9,
     "isolation_mm": 6.2, "stationary_s": 1.84},
    {"id": 21, "...": "..."}
  ]
}
```

- `candidates` = **top ~3** flies ranked by score = f(size ↑, isolation ↑, stationary_s ↑) with hard gates (isolation > threshold, stationary_s > 0.3–0.5 s).
- **Dropout semantics:** a fly that moves > 0.25 mm immediately loses its `stationary_s` and falls out of the candidate list — the robot detects target invalidation by dropout without extra round-trips.
- **Target-lock handshake (optional command):** robot claims a candidate ID over HTTP; the tracker then reports that ID's status explicitly (even after it moves) until released — so a mid-pick escape is reported unambiguously rather than inferred.
- Coordinates in mm in the camera/platform frame for now; the camera→robot transform is ROB-20's calibration deliverable.

**Summary logging (built-in — this is the pickability-pilot instrument):** compact JSONL/CSV time-series, ~1 row per second (configurable): `t, n_flies, n_moving, n_pickable, per-fly [id, x, y, size, moving]` — plus one annotated overlay snapshot per minute. **No bulk video.** A 10-minute run is a few MB.

**Test procedure:** recorded multi-fly fixtures first (§11) — count accuracy vs hand count, association stability; then live 30–35 flies: count accuracy, candidate-list stability (no flicker), a hand-poke test (disturb a candidate → verify immediate dropout), rate benchmark (max survey Hz with everything on).

**Pass:** count within ±1 of manual count on stills; 50 Hz sustained with logging on; candidate dropout latency < 100 ms after motion; 10-min live run with clean logs.

## 10. Phase 6 — Pickability pilot experiment

**Purpose (pre-robot):** will flies placed on the open platform *stay* and *settle* enough to be picked? Measure retention and activity under a few environmental conditions, using Phase 5's logging — the core worry is that flies simply escape.

**Protocol**
1. Place ~30 flies on the platform (standard transfer, note anesthesia method and recovery time).
2. Run target mode with summary logging for **10 minutes** (first pass — if escape is high there is no point in longer trials; extend to 20–30 min only for conditions that hold flies).
3. Conditions (one variable at a time, ≥2 replicates): baseline room temp; cooled platform (Peltier setpoints, e.g. ~10–15 °C); ambient light vs dark (850 nm illumination only); optionally time-of-day / starvation state.
4. Analysis script (Python, runs on any machine): count vs time (retention/escape rate), % moving vs time, pause-duration distribution, `n_pickable` vs time per condition.

**Deliverable:** a short results memo with the plots — directly answers "how many flies are available and how many are pickable at any moment, under which conditions" before any robot integration.

## 11. Data recording (fixtures — do this early)

Record a small library of raw sequences as soon as Phase 1 works (Arena SDK capture-to-NVMe; short clips, not the pilot data path):

| Fixture | Content | Use |
|---|---|---|
| `empty_plate` | 10 s, both cameras | background models, noise floor |
| `single_fly` | 60 s walking fly, both cameras | Phases 3–4 dev + regression |
| `multi_fly_30` | 60 s, 30–35 flies, both cameras | Phase 5 dev + count ground truth |
| `stationary_target` | 10 s printed dot target | centroid-noise measurement |

These enable **offline development on any machine (including the Mac)** and give every pipeline change a fixed regression target. Total ≲ 300 GB on the 2 TB NVMe; keep the keeper subset, archive to lab storage.

## 12. Tie-in to ROB-12 optical acceptance

atlas-bench provides the measurement tooling for the software-measurable ROB-12 acceptance items: **dropped frames** (Phase 1 soak), **exposure at rate** (Phase 1, 850 nm illumination at target fps), **centroid stability** (Phase 3 noise measurement, per aperture and Z per the camera plan §9). Record these in the same acceptance log as the optical items.

## 13. BIAS: how substantial is the re-architecting?

Survey of the current codebase (BIAS/FlyVRBIAS, `trackfly` branch):

**What's already good:** the camera-vendor abstraction is genuinely clean — `Camera` facade → `CameraDevice` base → per-SDK backends; nothing above the facade knows the vendor. Frames are refcounted `cv::Mat` after the backend, so grab→log→plugin→display is already copy-light. The FlyTrack HTTP contract (keep-alive, `get-last-clear-track`) measured ~500 Hz / 1–2 ms.

**The Arena (LUCID) backend — largely done, by Isabel.** The [`lucid` branch of isabel-hess/bias](https://github.com/isabel-hess/bias/tree/lucid) adds a complete `src/backend/arena/` mirroring the Spinnaker backend (~2,700 lines + facade wiring + `FindArena.cmake`), documented in its [`docs/lucid-arena-backend.md`](https://github.com/isabel-hess/bias/blob/lucid/docs/lucid-arena-backend.md), and **verified end-to-end on a LUCID Phoenix PHX004S-M at ~271 fps** — enumerate → connect → stream → clean stop, from both the GUI and the HTTP control API. Design decisions that carry straight over to the Atlas10s: it uses the **Arena C API** (`ArenaC`, typed by-name node accessors — no GenICam wrapper layer needed, and it links under both MinGW and MSVC); cameras are identified by **IP address** with numeric-IP ordering, so a camera keeps a stable camera number/HTTP port across runs; a reference-counted shared Arena system handles the SDK's single-open constraint; and it resolves the `Guid(std::string)` collision with Spinnaker via a tagged `Guid(std::string ip, CameraLib)` constructor — exactly the sharp edge our survey flagged. Her branch also fixed several latent BIAS bugs exposed by modern toolchains (missing returns causing UB, non-`const` set comparators, an out-of-range histogram index) — at least the histogram indexing bug is present in FlyVRBIAS too (`src/gui/camera_window.cpp:3608`), so her fix list is the checklist when porting.

**What remains for the Atlas10s (small, well-defined — see ROB-23):** port the patch from her branch into FlyVRBIAS/`trackfly` (the backend directory is self-contained; the facade edits mirror the existing `spin` blocks); validate at 10GigE rates — her testing was a 1GigE 0.4 MP camera at ~105 MB/s, an order of magnitude below Atlas10's ~1.1 GB/s, so stream-channel/packet-size tuning needs checking; and add a Mono8 fast path in `grabImage` — the current code follows the Spinnaker pattern (`acImageFactoryConvert` + `copyTo` per frame), which is two full-frame passes we don't want at 1 GB/s when Mono8 needs no conversion at all. RDMA stays out of BIAS scope (that's atlas-bench/tracker territory); BIAS uses standard sockets.

**Moving the GPU pipeline into BIAS — substantial (weeks):** C++11 → C++17 baseline bump (also prerequisite for OpenCV 5); unbounded frame queues need real backpressure (the existing plugin-queue cap is dead code); `StampedImage` needs a GPU/pinned-buffer handle and lifetime rules; HTTP must move off the GUI thread; assorted hot-path cleanups (per-frame debug file writes in the dispatcher, per-component full-frame scans in FlyTrack's connected-components step). None of it is exotic, but it touches the acquisition spine everywhere.

**Recommendation:** keep the GPU tracker as a **standalone service** (Stage B) — it is the production architecture for the robot, not a temporary scaffold. **Port Isabel's Arena backend into FlyVRBIAS** as a parallel, low-risk track (ROB-23) — it gives BIAS GUI/recording with the Atlas cameras and an independent sanity check on the cameras during bring-up. Merge the GPU path into BIAS only if a concrete need emerges (e.g., wanting BIAS's GUI/logging around the GPU tracker), informed by Stage A/B measurements.

## 14. CV libraries: what changes on the GPU?

- **BIAS today:** OpenCV 4, CPU-only, all classic ops (threshold, connected components, moments/PCA, morphology). OpenCV remains the right tool for this layer.
- **OpenCV 5.0** (released June 2026; drops the legacy C API, requires **C++17**, new CPU DNN engine, classic CUDA modules retained with CUDA 13 support): BIAS can't take it without the C++17 bump, but the **new standalone code is C++17 from day one and should be built against OpenCV 5** — low risk, since OpenCV there is used for verification, fixtures, and offline tools rather than the hot path. Evaluate during Stage A; fall back to 4.x trivially if anything bites.
- **GPU hot path — not OpenCV:** the per-frame ops we need (subtract, threshold, ROI moments, reductions) are small, simple **custom CUDA kernels (+ NPP where convenient)**. This avoids from-source OpenCV-CUDA builds on Windows (fragile, huge, and version-locked to the CUDA toolkit) and keeps the latency-critical code fully under our control and instrumentable.
- **ML detectors (TensorRT / OpenCV 5 DNN):** out of scope for September. Classic CV should be very robust here (backlit 850 nm, high contrast, ~100–160 px flies). ROB-15 (evaluating Mark & Seung Je's networks on the new images) is the future path if classic CV hits limits (touching flies, wing/leg detail).

## 15. Platforms and development workflow

- **Windows (the Dell T2) is the target and primary dev machine** — this is deliberate: we want to know if this system can be done simply on a modern Windows PC with a good-but-not-exotic NVIDIA card. All camera-attached work happens here (Arena SDK is Windows/Linux only — **there is no macOS Arena SDK**).
- **Mac:** offline algorithm development against the §11 fixtures (OpenCV + Python/C++ without Arena), analysis scripts, docs. BIAS's video-file mode (`--play-fps`) also runs on Mac for FlyTrack-side work.
- **Linux:** the documented fallback if Windows RDMA under-delivers — LUCID supports RDMA and (uniquely) GPUDirect there. Only reach for it if Phase 1/2 measurements force us to; the project goal is the simple Windows story.
- Suggested repo: a new `atlas-bench`/`flymax-tracker` repository (or a directory in `flymax-development`) — CMake, C++17, CUDA; CPU-only build flag so fixtures-based development works on machines without CUDA/Arena.

## 16. September schedule and risks

Assumes cameras + PC on site by late August (PC est. Aug 7; cameras ordered 8/7).

| Week | Milestones |
|---|---|
| Sep w1 | Phase 0 complete (both cameras at full rate in ArenaView); atlas-bench skeleton; Phase 1 first numbers |
| Sep w2 | Phase 1 soak matrix done (sockets vs RDMA decision); Phase 2 GPU copy path measured; §11 fixtures recorded |
| Sep w3 | Phase 3 single-fly pipeline on fixtures + live fly; CPU-vs-GPU table; Phase 4 UDP/HTTP publication, two-PC latency. Parallel track: port Isabel's Arena backend into FlyVRBIAS (ROB-23) |
| Sep w4 | Phase 5 target mode live (count + candidates + logging); first Phase 6 pilot runs; follow-mode robot demo if a robot is ready; write up results |

**Risks / open questions**

| Risk | Mitigation |
|---|---|
| HCA is not the RDMA-enabled model, or camera lacks RDMA firmware | Verify Phase 0 step 4 first thing; LUCID firmware upgrade is free; sockets path is the measured fallback |
| Windows RDMA (Network Direct) immaturity | Phase 1 A/B gives hard numbers; tuned sockets or Linux as fallbacks |
| PoE+ budget / thermals at sustained 10GigE | Camera temp logged in Phase 0/1 soaks; external supply via GPIO if needed |
| 850 nm contrast through BOROFLOAT window insufficient for clean thresholding | ROB-12 optical acceptance runs first; adjust illumination geometry (oblique IR per ROB-10 discussion) |
| Flies escape the open platform (project-level risk) | That's exactly what Phase 6 measures, cheaply, before robot integration |
| Camera/PC delivery slip | Fixtures can be recorded on any interim GigE camera to keep Phases 3–5 software moving |

---

*Repo pointers for reference (BIAS/FlyVRBIAS `trackfly` branch): camera facade `src/facade/camera.{hpp,cpp}`; Spinnaker backend template `src/backend/spin/`; FlyTrack pipeline `src/plugin/flytrack/flytrack_plugin.cpp`; reusable multi-blob detector `src/utility/blob_finder.*`; Hungarian identity tracker `src/demo/fly_sorter/identity_tracker.*`; HTTP server `src/utility/basic_http_server.*`. Isabel's Arena backend: [isabel-hess/bias `lucid` branch](https://github.com/isabel-hess/bias/tree/lucid) — `src/backend/arena/`, `cmake/Modules/FindArena.cmake`, docs in `docs/lucid-arena-backend.md` and `docs/building-on-windows.md`.*
