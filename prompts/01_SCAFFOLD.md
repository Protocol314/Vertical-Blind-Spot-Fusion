# PHASE 01 — 코드 스캐폴드와 타입 계약

## 그대로 복사할 구현 요청

`AGENTS.md`와 `docs/01_REQUIREMENTS_SCOPE.md`, `docs/03_SYSTEM_ARCHITECTURE.md`, `docs/10_DATA_AND_INTERFACE_CONTRACTS.md`, `docs/13_TESTING_AND_SIMULATION.md`를 읽고 다음을 구현하라.

**목표:** Pi/ROS 실장비 없이도 unit test를 수행할 수 있는 `vertical_fusion` Python 3.10 패키지 생성.

### 필수 구현
- `src/vertical_fusion/`에 `types.py`(또는 `schemas/`), `io.py`, `config.py`, `exceptions.py`, `timebase.py`, `mock/`와 `tests/unit/` 생성.
- `VisionDetection`, `ScanFrameRef`, `LidarCluster`, `FusionCandidate`, `FusedObject`, `EvaluationEvent`, `CalibrationRecord`의 typed structures 구현.
- `status`, `range_source`, `source_flags`, `reason_codes` Enum과 검증. `UNKNOWN` range는 `None`; 측정 범위가 없다고 `0.0`m 사용 금지.
- JSONL read/write 및 `.json` 예시 fixture. `schema_version`, `calibration_id`, `model_id`, `session_id`, `timestamp` 보존.
- `TestClock`, `MockCamera`, `MockLidar`, 재생 가능한 임의 seed fixture.
- 형식 검사/pytest/ruff 설정과 `README`의 실제 설치·시험 명령.
- 필수 의존성과 선택 GPU/ROS 의존성 분리; 버전 충돌을 줄이고 설치를 최소화.
- `data/`의 민감 녹화, raw videos, API keys 등 git 제외.

### 수락 시험
- 정상 스키마 왕복 serialization.
- `camera_only + range=None`가 강제되고 `range_source=UNKNOWN`인지 확인.
- 실측 아닌 영상 객체의 `LIDAR_MEASURED` 출처 지정 거부.
- 잘못된 타입·음수 age·시간 도메인 미정·확률 >1 검증 실패.
- 모든 핵심 모듈이 ROS2 import 없이 pytest 가능.

### 결과 보고
변경한 파일 트리/실행한 명령/테스트 로그/현재 미구현 기능/다음 PHASE 02까지 필요한 조건을 한국어로 제시. 이 단계에서 장비 연결·YOLO 다운로드·외부 API 호출은 하지 마라.
