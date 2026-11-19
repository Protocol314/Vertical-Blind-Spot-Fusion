# 16. 2인 14일 실행 계획과 중간 점검

## 역할 분담

- 담당 A (ROS/센서): Pi·LiDAR·카메라 입력, timestamp, 캘리브레이션, 투영·리플레이, 실제 테스트.
- 담당 B (Vision/ML): YOLO26n, 라벨링/학습, 구조화 특징, 경량 모델/검증, 결과 시각화.
- 공동: 실험 프로토콜, 독립 정답, 오류 사례 검토, 통합 성능, 보고서.

| Day | 담당 A | 담당 B | 하루 종료 시 수락 조건 |
|---:|---|---|---|
| 1 | Pi/ROS/scan 상태 점검·기록 | YOLO 환경·입력 영상 확인 | 실행 환경과 버전 로그, 짧은 scan+image 샘플 |
| 2 | 카메라 캘리브레이션 | 고정 이미지 YOLO 테스트 | 원본 좌표계·탐지 결과 JSON 구조 통일 |
| 3 | LiDAR-카메라 rigid rig 및 외부 보정 | 실험 객체·이미지 라벨 정의 | 보정판 투영 확인, scene 정의 |
| 4 | time sync / rosbag replay | YOLO 입력·출력 모듈 | 재생 가능 세션, timestamp 유지 |
| 5 | 실제 laser endpoint projection | 데이터 품질/클래스별 오류 | 투영 검증 및 GT 라벨 시작 |
| 6 | LiDAR 군집 및 후보 연결 | 필요시 YOLO 파인튜닝 | B0/B1 원본 baseline |
| 7 | 규칙·OR 통합 | 특징 생성·training split | **GATE-1: B2/B3 재생 가능 + camera-only 보존** |
| 8 | misassociation / stale 방어 | LogReg baseline 학습 | 결과 스키마 + smoke tests |
| 9 | 세션별 오류 검토 | tiny MLP/GBDT 비교 | B4 예측·학습 재현 |
| 10 | 정지 실측 1 | 데이터 라벨 검증/정리 | 시험 세션 독립성 확보 |
| 11 | 통제 실측 2 / failure inject | 모델 평가·임계값 고정 | 4+ 비교군 동일 GT 평가 |
| 12 | latency·drift·drop 측정 | 통계/결과표 | **GATE-2: 측정값·조건·실패 사례 보고** |
| 13 | 데모 기록 / 재설치 시험 | 발표용 그래프·보고서 | clean run 설명서 |
| 14 | 재현성 최종 검사 | 연구 한계·발표 정리 | **GATE-3: 수락 체크리스트 통과** |

## 계획 변경 기준

- Day 3에 장착/보정이 실패하면 하드웨어 의존 로직을 계속 늘리지 말고 **정지 장면의 rosbag + synthetic calibration fixture**로 핵심 모델 개발을 먼저 진행.
- Day 7에 규칙 융합이 불안정하면 tiny AI 추가 학습보다 규칙·데이터 계약/정합 문제 해결 우선.
- Day 10 이후 현장 테스트 시간이 부족하면 평가 장면을 줄이더라도 독립 정답과 실패 조건을 유지.
- 실제 Jev API, 단안 깊이, Nav2 costmap, 1D CNN은 **필수 목표 모두 달성한 경우만** 추가.

## 주간 회고

- Day 7: 동작하는 B2/B3 replay와 사각지대 camera-only 표시가 없다면 연구 핵심 미완료.
- Day 12: 결과표가 실제 시험 데이터 없이 비어 있다면 발표에서 개선 주장 불가.
- Day 14: 해당 조건에서 긍정적 개선이 없어도 **왜 실패했는지 원인 분석**을 제출하면 학술적 의미를 지닐 수 있음.
