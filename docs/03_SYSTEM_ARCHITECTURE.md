# 03. 시스템 아키텍처와 모듈 계약

## 상위 흐름

```mermaid
flowchart TD
    CAM[Camera /image_raw] --> VIS[YOLO26n Inference]
    LASER[RPLIDAR C1 /scan] --> LID[Laser Preprocessing]
    VIS --> SYNC[Temporal Gate + Transform Lookup]
    LID --> SYNC
    CAL[Calibration K D T_camera_lidar] --> SYNC
    SYNC --> ASSOC[Projection + Candidate Association]
    ASSOC --> FEATURE[Feature Builder + Validity Masks]
    FEATURE --> RULE[Deterministic Baseline]
    FEATURE --> MLP[Local Tiny Classifier]
    RULE --> POLICY[Safety/Uncertainty Policy]
    MLP --> POLICY
    POLICY --> OUT[/fusion/obstacles + /risk_sectors]
    POLICY --> REC[Recorder / Evaluation]
    POLICY -. optional telemetry .-> JEV[TypeSafe Jev Shadow]
    JEV -. asynchronous comparison only .-> REC
```

## 물리적 배치

- Pi 4B: `/scan` 발행, 필요 시 카메라 장치 원본 데이터 발행, ROS2 bag 수집. 기존 작업 공간 비침습적.
- RTX 4060 노트북: YOLO26n, 기하 융합, LR/MLP 학습/추론, 실험 UI 및 기록.
- **실시간 네트워크의 복잡성이 MVP 실패 원인이 되면:** rosbag를 옮겨 **오프라인 재생을 먼저 완성**하고 온라인 통신을 두 번째 목표로 수행.
- 실제 Jev 원격 API: 선택적 진단/비교 경로. API 비활성 시 기능 결과 동일해야 함.

## 출력 채널

| 채널 | 내용 | 변경/제약 |
|---|---|---|
| `/scan` | 실제 LiDAR 거리 측정 원본 | **불변** |
| `/camera/image_raw` | 원본 촬영 영상 | 촬영 시각 유지 |
| `/vision/detections` | YOLO 클래스·박스·점수·frame time | 실측 거리 없음 |
| `/fusion/obstacles` | 객체별 출처·거리 상태·위험점수·유효시간 | 핵심 결과 |
| `/fusion/risk_sectors` | 각도 구간별 위험/출처/미확정 | 선택적 1D 표현 |
| `/fusion/diagnostics` | 입력/동기화/추론/드롭 상태 | 필수 |
| `/fusion/virtual_scan` | 별도 추정 거리 표현 | 확장 전용, 실측으로 혼동 불가 |

## 순수 로직과 ROS 어댑터 분리

`src/vertical_fusion` 내부에서 처리 핵심을 dataclass+NumPy 기반 순수 함수로 설계한다. ROS `Node`는 메시지 수집/발행 및 시간 관리만 담당한다. `MockCamera` / `MockScan` / `ReplayAdapter`로 하드웨어가 없는 환경에서도 동일한 핵심 파이프라인을 테스트한다.

## 입력·출력 상태 전이

```text
raw capture -> validated -> synchronized -> projected -> candidates
 -> {confirmed_fused, lidar_only, camera_only, ambiguous, stale}
 -> risk-policy -> published/expired
```

- 같은 객체가 모든 상태를 동시에 가지지 않는다. 변경 이력은 `track_id`와 상태 전이 이벤트로 남긴다.
- 누락된 객체에 새 `range_m`를 붙이지 않는다. 이전 측정값의 재사용은 명시적 지연·만료 표시가 있어야 한다.
- 가시화 UI에서 ‘실측 거리’는 실선, ‘영상 전용’은 별도 마커, ‘추정 거리’는 불확실성을 함께 표시한다.

## 장애 격리

| 실패 | 즉시 반응 |
|---|---|
| YOLO 추론 오류 | LiDAR 원본·LiDAR 전용 장애물은 유지, 영상 상태 비활성 |
| LiDAR timeout | 카메라 단독 후보 유지, 거리 미확정 |
| tf2 조회 불가 | 센서 간 확인 불가능; 각 센서 단독 정보는 보존 |
| 보정 불일치 | 고신뢰 융합 중지, `ambiguous`/진단 |
| API 장애 | Jev 기록에 오류만 남김; 주 경로에 영향 없음 |
| 오래된 추론 결과 | 큐에서 만료시키고 이전 프레임을 새 관측으로 둔갑시키지 않음 |
