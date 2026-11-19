# 13. 테스트 전략: 합성 → 녹화 재생 → 통제된 실측

## 원칙

실장비 없이도 실패 케이스를 검증할 수 있도록 핵심 모듈은 ROS 의존 없이 테스트 가능해야 한다. 가상 테스트 결과를 실제 물리 성능 근거로 사용하지 않는다. 모든 시험은 독립된 입력/기대 출력/허용 오차가 있어야 한다.

## 테스트 피라미드

### A. 단위 테스트

- 이미지 내 점 투영과 렌즈 왜곡 처리, frame convention, `Z<=0` 거부.
- `LaserScan` index→angle/time 변환, `range_min/max`와 NaN/Inf 처리.
- `bbox_xyxy`/letterbox 변환, 검출 신뢰도 범위, `calibration_id` 일치.
- 구간 cluster 검출, 원형 각도 래핑(-π/π 경계).
- 두 박스와 하나의 광선이 겹칠 때 중복 확정 배정 방지.
- Lidar cluster가 배경에 있고 camera low box는 전경에 있을 때 **range=null**.
- camera-only와 lidar-only 후보가 동시에 있어도 둘 다 유지.
- 오래된 이미지, transform 없음, 뒤틀린 시간 기준, Jev API 실패 시 주 경로 동작.

### B. 통합/리플레이 테스트

- 동기화 데이터의 영상 투영과 시각화.
- rosbag2 재생 시 같은 `session_id`/입력/모델 버전이면 같은 상태 전이.
- 룰/B0–B4별 동일 입력에 대해 출력 비교.
- 장면별 `first_alert` 시간과 FA/min 계산값이 사람이 만든 골든 이벤트 표에 일치.

### C. 실제 환경 수락 테스트

1. 정지 카트/로봇에 LiDAR·카메라 부착, 전원/고정 확인.
2. 낮은 장애물 vs laser plane 교차 장애물 vs 매달린 장애물 준비.
3. 같은 조건을 최소 3개 거리에서 반복, 독립 거리·높이 측정.
4. 센서 프레임·보정 파일·기록파일·정답 라벨 저장.
5. 실험 결과를 수치로 요약한 뒤에만 연구 개선 주장.

## 필수 고장 주입 사례

| ID | 주입 | 기대 결과 |
|---|---|---|
| F01 | camera frame dropout | LiDAR-only 유지; 카메라 데이터 stale |
| F02 | LiDAR scan dropout | camera-only 보존; 실측 거리 없음 |
| F03 | old YOLO result(> TTL) | 새 관측으로 취급 금지 |
| F04 | tf2 lookup failure | `ambiguous`/독립 관측만 |
| F05 | invalid calibration hash | 결합 불가, 명시적 상태 |
| F06 | 물체 뒤 벽 레이저 | 배경 거리를 물체 거리로 배정하지 않음 |
| F07 | 그림자(물리 장애물 X) | 오경고 이벤트 계산 |
| F08 | 상부 물체가 실제 안전 높이 | 위험 과탐 여부 평가 |
| F09 | API timeout/429 | Jev off와 동일 주 결과 |
| F10 | 중복 bbox / 클래스 변경 | track ID 교체·중복 경고 처리 |
| F11 | 영상 비율/레터박스 변경 | 좌표 변환 검증 또는 명시적 실패 |
| F12 | 레이저 frame 반전 | 검증 실패를 감지하고 대응 억제 |

## 테스트 데이터 종류

- `fixtures/synthetic/`: 수학적으로 참값이 있는 카메라 행렬, 광선, 가짜 검출(실측 아님).
- `fixtures/golden/`: 수작업 검증한 작은 실제 샘플(수집 전에는 없음).
- `data/recordings/`: 현장 전체 rosbag2 (gitignore, 배포 시 비식별 확인).
- `results/`: 요약 CSV/JSON, confusions, latency histograms (실제 생성 전 없음).

## 자동화 수락 테스트 예

```bash
pytest tests/unit -q
pytest tests/integration -q
python -m vertical_fusion.evaluation.cli --help
```

위 명령의 모듈 경로는 **Codex가 구현할 예정**이며, 이 ZIP 자체에는 아직 실행 모듈이 없다.
