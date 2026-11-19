# Vertical Blind-Spot Fusion | 연구·개발 명세 패키지

> 연구 주제: **단일 2D LiDAR의 수직 관측 사각지대 보완을 위한 YOLO 기반 카메라–LiDAR–경량 AI 센서 융합 장애물 탐지 시스템**  
> 기준일: **2026-11-17** · 문서 패키지 **v1.0.0** · 언어: 한국어 · 대상: **Codex / Claude Code**

## 이 ZIP에 들어 있는 것

이 패키지는 **소스 코드가 아니라, 소스 코드를 안전하고 재현 가능하게 만들기 위한 Markdown 설계·실험·작업 지시 문서**다. ZIP을 풀어 개발 저장소 루트로 사용하고, Codex는 `AGENTS.md`, Claude Code는 `CLAUDE.md`를 먼저 읽게 한다. 두 파일은 공통 문서를 참조한다.

### 즉시 시작

1. 프로젝트 폴더를 Codex 또는 Claude Code로 연다.
2. `AGENTS.md` / `CLAUDE.md`와 `docs/00_PROJECT_BRIEF.md`를 읽게 한다.
3. `prompts/00_MASTER_START.md`를 **첫 요청**으로 제공한다.
4. `prompts/01_SCAFFOLD.md`부터 `prompts/08_RELEASE.md`까지 **순서대로 진행**하되 각 단계의 수락 기준을 통과해야 다음 단계로 이동한다.
5. 실제 장비가 없으면 **재현 가능한 합성 입력 및 rosbag2 재생 경로를 먼저 구현**한다. 합성 시험은 실제 성능 결과로 보고하지 않는다.

### 필수 구현 목표(MVP)

- Raspberry Pi 4B / ROS 2 Humble / RPLIDAR C1의 실제 또는 기록된 `/scan` 데이터 수집.
- RGB 카메라 영상의 YOLO26n 객체 탐지. **영상 모델 학습 데이터와 실시간 추론 결과를 혼동하지 말 것.**
- 카메라 내부·외부 보정, 센서별 타임스탬프를 이용한 유효한 시간 정합.
- LiDAR 실측점의 영상 투영과 객체–점 대응의 **모호성 보존**.
- LiDAR에서 보이지 않는 객체도 `camera_only` 위험 후보로 유지; **실측되지 않은 거리 생성 금지**.
- 베이스라인 4종 비교: LiDAR / 카메라 / 규칙 융합 / 경량 학습 융합.
- 정량 평가: 높이별 미탐, 오탐, 잘못된 거리 대응, p50·p95·p99 지연.

### 명시적으로 제외(MVP)

- 단일 2D LiDAR와 RGB 카메라만으로 모든 장애물의 진짜 3D 형상 복원.
- 주행 제어·안전 인증·사람이 탑승한 이동장치에 바로 적용.
- 실제 Jev API의 응답을 매 스캔 기다리는 실시간 필수 경로.
- 근거 없이 특정 정확도·지연을 이미 달성했다고 주장.

## 문서 찾아보기

| 목적 | 문서 |
|---|---|
| 연구 목표·범위·핵심 가설 | [00](docs/00_PROJECT_BRIEF.md), [01](docs/01_REQUIREMENTS_SCOPE.md), [02](docs/02_RESEARCH_AND_NOVELTY.md) |
| 전체 흐름 및 ROS 2 노드 | [03](docs/03_SYSTEM_ARCHITECTURE.md), [05](docs/05_ROS2_TIMING_AND_QOS.md) |
| 센서 장착·캘리브레이션 | [04](docs/04_CALIBRATION_GEOMETRY.md) |
| YOLO·LiDAR 입력 파이프라인 | [06](docs/06_YOLO26_PIPELINE.md), [07](docs/07_LIDAR_ASSOCIATION.md) |
| 구조화된 융합·경량 AI | [08](docs/08_LIGHTWEIGHT_FUSION_AI.md), [09](docs/09_JEV_OPTIONAL_COMPARISON.md) |
| 메시지·데이터 계약 | [10](docs/10_DATA_AND_INTERFACE_CONTRACTS.md) |
| 데이터셋·실험·평가 | [11](docs/11_DATASET_AND_LABELING.md), [12](docs/12_EVALUATION_PLAN.md) |
| 테스트·성능·안전 | [13](docs/13_TESTING_AND_SIMULATION.md), [14](docs/14_PERFORMANCE_DEPLOYMENT.md), [15](docs/15_RISKS_SAFETY_LICENSE.md) |
| 일정·이슈·인수기준 | [16](docs/16_14_DAY_PLAN.md), [17](docs/17_ENGINEERING_BACKLOG.md), [18](docs/18_ACCEPTANCE_CRITERIA.md) |
| 구현용 수식·의사코드 | [19](docs/19_ALGORITHMS_PSEUDOCODE.md) |
| 연구 출처 | [20](docs/20_REFERENCES.md) |
| 결정 기록·변경 관리 | [21](docs/21_ADR_AND_CHANGELOG.md) |
| 연구 보고서 템플릿 | [22](docs/22_FINAL_REPORT_TEMPLATE.md) |
| 에이전트 작업 요청문·인수인계·독립 리뷰 | [`prompts/`](prompts/00_MASTER_START.md) |
| 체크리스트 | [`checklists/`](checklists/00_ENVIRONMENT.md) |

## 구현 후 권장 저장소 구조

```text
.
├── AGENTS.md                 # Codex 지침
├── CLAUDE.md                 # Claude Code 지침
├── docs/ prompts/ checklists/  # 이 문서 패키지
├── configs/                  # 장비·노드·모델·평가 파라미터
├── src/vertical_fusion/      # Python 패키지 / 핵심 로직
│   ├── capture/              # ROS·영상 입력 어댑터
│   ├── calibration/          # 카메라·LiDAR 변환, 검증
│   ├── vision/               # YOLO 추론·추적(선택)
│   ├── lidar/                # 스캔 정규화·군집화
│   ├── association/          # 지리·시간적 연관 후보
│   ├── fusion/               # 규칙 기반 / 학습 모델
│   ├── evaluation/           # 평가·지연·통계
│   └── visualizer/           # RViz / 오버레이
├── ros2_ws/src/             # 필요한 경우만 ROS 패키지
├── tests/{unit,integration,replay}/
├── tools/                    # 기록·학습·분석 실행 스크립트
├── data/                     # 로컬 데이터 (git에서 제외)
└── results/                  # 실험 결과 (요약만 버전 관리)
```

`src/`와 나머지 코드는 **구현 단계에서 생성**한다. 이 패키지가 완료된 제품 또는 실행 가능한 ROS 노드를 포함한다고 주장하지 않는다.

## 검증 표시 규칙

- `FACT`: 공식 문서·논문으로 확인한 사실. 정확한 출처가 `docs/20_REFERENCES.md`에 있어야 한다.
- `DESIGN`: 이번 프로젝트에서 선택한 설계 원칙.
- `TARGET`: 실험으로 검증할 공학적 목표. 달성한 성능이 아니다.
- `TODO`: 구현자가 확인·작성할 내용.
- `MEASURED`: 원본 로그·측정 환경·커밋 ID가 기록된 실제 실험값에만 사용.

## 주의 사항

실험은 **정지 플랫폼 또는 통제된 저속 환경**을 기준으로 한다. 사람이 타는 장치의 실제 안전 주행 판단이나 자동 비상 정지 기능을 이 MVP의 검증 범위로 주장하지 않는다. Ultralytics의 AGPL-3.0 / 엔터프라이즈 라이선스 선택은 배포 전에 별도로 검토한다.
