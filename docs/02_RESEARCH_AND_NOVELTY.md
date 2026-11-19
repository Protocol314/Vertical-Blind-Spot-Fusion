# 02. 선행연구 지도와 신규성의 경계

## 주요 기존 연구

| 연구 | 공개 사실·의미 | 이번 프로젝트와 차이 |
|---|---|---|
| Young & Simić (2015), *LIDAR and Monocular Based Overhanging Obstacle Detection* | 2D LiDAR의 낮은·매달린 물체 사각지대를 카메라로 보완한 명백한 선행연구 | '2D LiDAR+카메라로 위·아래 물체 발견'만으로 신규성 주장 불가 |
| Fu et al. (2020), *Depth Completion via Inductive Fusion of Planar LIDAR and Monocular Camera* | 평면 LiDAR와 RGB 기반 깊이 보완을 학습 | 조밀한 깊이 복원과 본 프로젝트의 경량 의사결정은 목표 다름 |
| Han et al. (2022), *Deep Reinforcement Learning for Robot Collision Avoidance With Self-State-Attention and Sensor Fusion* | 영상에서 얻은 거리 표현과 2D LiDAR 정보 결합 | 본 프로젝트는 주행 정책이 아니라 검출 오류·결측을 관리 |
| Benayed et al. (2025), *LiDAR 2D and Camera Fusion for ADAS... YOLOv9 and ROS2* | 2D LiDAR+YOLO+ROS2 적용 사례 | 버전 교체 자체를 신규성으로 강조하지 않음 |
| Abaza et al. (2026), *Lightweight Semantic-Aware Route Planning...* | 2D LiDAR+단안 영상, 각도 섹터 대응 | LiDAR 반사가 없는 영상 전용 후보를 중심 목표로 유지 |
| D’Antona et al. (2026), *A Depth-Centric Review...* | 센서 깊이 밀도에 맞는 투영/후기 융합 선택의 중요성 | 후술한 최소 깊이 조건을 따르고 BEV·조밀한 3D 과장 금지 |

각 논문 세부 인용과 정확한 원문 링크는 `20_REFERENCES.md` 참고.

Abaza et al. (2026)은 Pi 5 CPU 기반 YOLO26n의 평균 약 167ms 추론과 결합 모듈의 매우 작은 연산 부담을 서로 구분해 보고했다. 이는 **영상 추론과 융합 결정의 소요시간을 분리 측정**해야 한다는 근거이지, Pi 4 또는 본 프로젝트의 성능 예측값은 아니다. 논문 원문: <https://doi.org/10.3390/s26072232>.

## 비교 연구 문제

1. 사각지대의 영상 객체 후보를 버리는 규칙이 미탐에 얼마나 영향을 주는가?
2. 모든 영상 객체를 보존하는 OR 방식이 오경고를 얼마나 증가시키는가?
3. LiDAR 군집·시간 정합·영상 신뢰도를 사용하는 작은 모델이 **동일 재현율에서** 오경고를 줄이는가?
4. 분명하지 않은 거리 대응을 `UNKNOWN`으로 남기면 잘못된 거리 부여가 줄어드는가?

## 논문형 주장 가이드

**허용:** '단일 스캔 평면 밖에 존재하는 객체의 탐지 누락에 대한 정량 비교', '별도 근거 없는 거리 부여를 거부하는 경량 판단 구조', '동일 로깅 조건의 규칙 대비 학습형 융합 비교'.

**금지:** '첫 2D LiDAR-YOLO 센서 융합', '3D LiDAR 수준의 정확도 입증', 'AI가 물리적으로 관측하지 않은 높이의 정확한 복원', '안전성이 검증된 자동제어'.

## 연구 제한

- 소규모 현장 데이터는 일반화 성능에 제한이 있다. 별도 환경에서의 성능을 미래 연구로 명시한다.
- C1의 실제 광선은 제조사 데이터시트의 피치 특성에 영향받을 수 있다. 이상적 z=0 모델과 실제 측정을 구분한다.
- 카메라 기반 미측정 거리·높이는 근거 없는 실측치로 승격할 수 없다.
- 2주짜리 시스템의 성능을 타 논문과 같은 벤치마크 숫자로 직접 비교하지 않는다.
