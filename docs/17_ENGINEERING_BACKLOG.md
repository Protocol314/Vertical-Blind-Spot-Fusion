# 17. 구현 백로그: 작업 단위·의존 관계

> 상태는 **미착수**. 에이전트는 실제 구현·테스트 로그가 있어야 체크박스를 완료로 변경한다. 별도 이슈 트래커가 없어도 작업 지시로 사용 가능.

## Phase 1 — 기반(P0)

- [ ] `CORE-001` 패키지 스캐폴드/pyproject or requirements/pytest/ruff — 의존: 없음 — 수락: `pytest` 실행, import 성공.
- [ ] `CORE-002` 모듈 공통 dataclass + serialization + schema_version — 의존: CORE-001 — 수락: JSON round trip/invalid input 검증.
- [ ] `CORE-003` fixture generator / mock sensor / deterministic seed — 의존: CORE-002 — 수락: ROS 없는 Linux/Windows Python에서 재생.
- [ ] `CAP-001` `/scan` readonly ingestion / rosbag2 recording (optional adapter) — 의존: CORE-002 — 수락: source hash/order/values 유지.
- [ ] `CAP-002` camera image + camera_info ingestion — 의존: CORE-002 — 수락: acquisition stamp 유지.
- [ ] `SYNC-001` timestamp pairing / TTL / drop policy — 의존: CAP-001,CAP-002 — 수락: skew/drop simulation 테스트.

## Phase 2 — 공간 보정(P0)

- [ ] `CAL-001` camera K/D save/load/validation — 의존: CORE-002 — 수락: points/undistortion golden test.
- [ ] `CAL-002` sensor transform T_camera_laser save/load + tf2 adapter — 의존: CAL-001 — 수락: frame axes regression.
- [ ] `CAL-003` LiDAR endpoint projection & pixel visualization — 의존: CAL-002,CAP-001 — 수락: known fixture + held-out target log.
- [ ] `CAL-004` calibration report with sample count/quality/ID — 의존: CAL-003 — 수락: calibration quality metadata exists.

## Phase 3 — 관측 추출(P0)

- [ ] `VIS-001` YOLO26n adapter with stub + real runtime — 의존: CORE-002 — 수락: original coords, weights hash, outputs.
- [ ] `VIS-002` test on local obstacles; train only if needed — 의존: VIS-001 — 수락: independent test results.
- [ ] `LID-001` valid scan rays + circular clustering — 의존: CAP-001 — 수락: NaN/Inf/range/angle boundary tests.
- [ ] `ASC-001` candidate association by projection/time/overlap — 의존: CAL-003,LID-001,VIS-001,SYNC-001 — 수락: background wall counterexample remains unconfirmed.

## Phase 4 — 융합(P0)

- [ ] `FUS-001` B0 LiDAR-only / B1 YOLO-only / B2 OR fusion — 의존: VIS-001,LID-001 — 수락: 3 outputs same session.
- [ ] `FUS-002` B3 rule fusion with measured-vs-inferred range separation — 의존: ASC-001 — 수락: unmatched camera survives with null range.
- [ ] `FUS-003` safe-output policy, TTL, ambiguity, no fabricated range — 의존: FUS-002 — 수락: F01–F12 fault tests.
- [ ] `FUS-004` risk sectors and source provenance — 의존: FUS-003 — 수락: camera-relative bearing and lidar bearing not conflated.

## Phase 5 — 경량 학습(P1)

- [ ] `DATA-001` data session manifest+split integrity checker — 의존: CORE-002 — 수락: same session/physical object across train/test blocked.
- [ ] `DATA-002` feature generator + missingness masks — 의존: ASC-001,DATA-001 — 수락: feature order version hash, NaN/Inf handling.
- [ ] `ML-001` logreg, tiny MLP and optional GBDT train/eval — 의존: DATA-002 — 수락: repeatable fit, holdout metrics.
- [ ] `ML-002` threshold selection on val, no test tuning — 의존: ML-001 — 수락: frozen configuration + provenance.
- [ ] `ML-003` live inference fallback rules and latency — 의존: ML-002,FUS-003 — 수락: model absent ⇒ rule path remains.

## Phase 6 — 실험·공개(P0/P1)

- [ ] `EVAL-001` per-event GT schema and metrics — 의존: CORE-002 — 수락: synthetic expected TP/FP/FN.
- [ ] `EVAL-002` 5-way ablation replay — 의존: EVAL-001,FUS-002,ML-003 — 수락: same data/labels/version log.
- [ ] `EVAL-003` p50/p95/p99 instrumentation — 의존: SYNC-001 — 수락: capture age vs inference time separated.
- [ ] `EVAL-004` failure injection and source preservation tests — 의존: FUS-003 — 수락: no false measured ranges.
- [ ] `RELEASE-001` setup guide/recording demo/reproducible report — 의존: EVAL-002 — 수락: clean checkout instructions pass.

## Optional (P2)

- [ ] `EXT-001` segmentation mask / ground-contact-only range estimate, separate verified metrics.
- [ ] `EXT-002` 1D sector CNN, only if data size sufficient.
- [ ] `EXT-003` Jev remote API shadow comparison (explicit opt-in, no blocking).
- [ ] `EXT-004` Nav2 custom virtual hazard layer (never clear free-space from inferred rays).
- [ ] `EXT-005` Pi-only optimized YOLO inference, documented accuracy/latency cost.

## 개발 로그 요구

각 작업 완료 시 `task_id`, `git_sha`, `files_changed`, `tests_command`, `test_summary`, `input_fixture`, `limitations`를 작업 보고에 기록. 테스트 실패/미실행을 완료로 체크하지 않는다.
