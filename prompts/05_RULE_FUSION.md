# PHASE 05 — 단순·규칙 융합과 핵심 사각지대 보존

## 구현 요청

`docs/07_LIDAR_ASSOCIATION.md`, `docs/08_LIGHTWEIGHT_FUSION_AI.md`, `docs/10_DATA_AND_INTERFACE_CONTRACTS.md`를 읽고:

- B0 LiDAR-only, B1 YOLO-only, B2 OR-fusion, B3 기하+규칙 융합을 **동일 입력 계약**으로 구현.
- 후보 상태 `confirmed_fused|camera_only|lidar_only|ambiguous|stale`.
- 거리 출처 `LIDAR_MEASURED|IMAGE_ESTIMATED|UNKNOWN`을 `range_value`와 독립 분리. 이미지 기반 거리 추정은 MVP에서 비활성.
- `camera_only`의 `measured_range_m`, `estimated_range_m`은 null, 보정 범위밖 객체는 단순 범위 경고만.
- 물리 ray가 검출한 객체는 영상 부재만으로 사라지지 않음. 중복 bbox/point은 ambiguous로 보수 처리.
- `age_ms`, `max_age_ms`, `tf_valid`, `calibration_id`를 모든 상태 전이에 반영.
- `source_flags`, `reason_codes`와 debug visualizer의 범례를 구현.
- 가능하면 `/fusion/risk_sectors` 출력을 별도 구조로 생성. LiDAR 각도와 카메라 관측 각도를 혼동하지 말 것.

### 필수 단위/통합 테스트

- 낮은 박스+뒤쪽 벽: 박스는 visual hazard로 보존, 벽 거리 배정 X.
- 레이저-박스 실제 교차: 올바른 군집만 confirmed_fused.
- 렌즈 교란·중복 탐지·센서 드롭: 외관상 성공 처리 금지.
- `/scan` 원본 불변.
- Jev 사용/미사용에 상관없이 결과 동일 (Jev 아직 연결 안 됨).

### 보고

동일 fixture에서 B0~B3 검출 객체 리스트, 상태·출처·거리, 실패 사례들을 나란히 비교한 텍스트 표/이미지(이미지 출력은 구현 파일일 때만). **이 단계가 본 연구의 최소 핵심 기능**이다.
