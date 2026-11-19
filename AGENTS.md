# AGENTS.md — Codex 구현 계약

## Project identity
- **Vertical Blind-Spot Fusion**: single-plane 2D LiDAR + monocular RGB + YOLO26n + **local tiny decision model**.
- Written specification is in `README.md` and `docs/`. Follow `prompts/` phase order.
- User context: two-person university final project, approximately 1–2 weeks, existing RPLIDAR C1 and Pi 4B with ROS 2 Humble, RTX 4060-class laptop. No unnecessary purchases.
- Documentation and user-facing content: **Korean**. Code names/types: English. Prefer boring, testable Python and ROS2 Humble practices.

## Mandatory invariants (MUST NOT violate)
1. Physical `sensor_msgs/LaserScan` is **immutable**, `/scan` is the source of truth. Never inject inferred distances into it. Use separate outputs.
2. `CAMERA_ONLY` **remains visible** in fusion output with `range_m=null`, `range_source=UNKNOWN` absent validated range estimator. Do not silently drop unmatched YOLO detections.
3. Background LiDAR hits inside an image box are not automatically the detected object's distance. Prefer `AMBIGUOUS` to incorrect range.
4. Keep original acquisition timestamps and calibration version. Detection completion time is **not** capture time. Respect `LaserScan.header.stamp` as first ray and `time_increment` when meaningful.
5. Do **intrinsic/extrinsic and time calibration deterministically** before learned fusion; no model may be presented as directly calibrating both coordinate systems without validation.
6. Learning models cannot suppress a known, valid, near-field physical obstacle without explicit testable safety rules. Failure/timeouts => degradations, never fabricated evidence.
7. Actual TypeSafe Jev API is **optional shadow mode**. Network-dependent decision making never blocks critical fusion, replay, or visual hazard notifications.
8. Do not claim true 3D recovery, verified safety, accuracy, latency, or measured thresholds without raw test logs.
9. No downloading multi-GB datasets, paid APIs, deleting Pi workspaces, modifying production ROS settings, motor control, or transmitting user recordings without specific permission.
10. Any ROS→Python or WSL/Windows boundary must be explicit. Do not assume DDS discovery or clock synchronization works without proving it.

## Delivery sequence
- Begin with `prompts/00_MASTER_START.md`; implement phase 01→08 with checkpoints.
- First create an offline replay and synthetic fixture path. Hardware-in-loop only after corresponding tests pass.
- Each phase must state **files changed, commands run, test evidence, unverified assumptions, next steps**. Never write 'complete' until objective exit criteria pass.
- On dependency/version ambiguity, inspect installed environment and authoritative upstream docs before deciding; record exact versions and rationale in `docs/21_ADR_AND_CHANGELOG.md`.
- Use schemas and acceptance tests as contracts. Make deterministic seeds and session-level train/test splitting standard.

## Recommended coding conventions
- Use Python 3.10 compatible with ROS 2 Humble; isolate non-ROS training packages where needed. `ruff`, `pytest`, type hints, dataclasses/pydantic only if dependency justified.
- Modules deterministic and usable without ROS where possible; ROS node should wrap pure core logic. Define a `MockVision`, `MockLidar`, and `ReplayAdapter`.
- Test per-ray projection, behind-camera rejection, missing/NaN ranges, timestamp wrap/skew, overlapping boxes, foreground/background, missing transform, camera-only retention, stale expiration, confidence/uncertainty tags.
- For configuration, keep numeric thresholds documented in YAML/config, **not hidden magic constants**.
- Each model artifact stores git commit, dataset split IDs, feature order, preprocessing, seed, framework version, calibrated threshold, license notes.

## Working style
- Do not ask the user to choose optional features before making progress. Use MVP defaults and put extras behind feature flags.
- If no hardware/weights available, implement tested interfaces and replay; never invent experimental results.
- Update `docs/17_ENGINEERING_BACKLOG.md` only with evidence when marking tasks done.
