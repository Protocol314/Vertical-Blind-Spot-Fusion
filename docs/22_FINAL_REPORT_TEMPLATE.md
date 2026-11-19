# 22. 기말 연구 보고서 템플릿

> 최종 발표·과제 보고서 작성용. 이 파일의 **실험 결과란은 실제 기록이 없으므로 비워 둔 템플릿**이다. 학습/판단/측정에서 실제 수행하지 않은 일을 수행했다고 표현하지 않는다.

## 제목

**단일 2D LiDAR의 수직 관측 사각지대 보완을 위한 YOLO26n 기반 영상·거리 센서 융합과 경량 의사결정 AI의 실험적 평가**

## 초록 (실험 후 작성)

문제: 단일 LiDAR 스캔 평면 밖 장애물 누락. 방법: RGB YOLO26n 검출 + LiDAR projection + 규칙 융합 + 로컬 경량 모델. 데이터: [실제 세션 수/장면/표적]. 주요 결과: [미탐/오탐/지연 실제 값]. 한계: [관측 누락과 실측 거리 부재]. 결론: [측정한 것만].

## 1. 연구 배경

- 2D LiDAR 비용·전력·장착 용이성의 장점.
- 단일 스캔 평면 외 낮은/높은 물체의 관측 한계.
- RGB 객체 탐지의 강점과 metric depth 불확실성.
- 관련 연구: Young & Simić 2015, Fu et al. 2020, Han et al. 2022, Abaza et al. 2026 등을 역할에 따라 인용.

## 2. 연구 질문과 가설

H1/H2/H3/H4를 `docs/00_PROJECT_BRIEF.md` 문구와 일치시킴. 개선이 없더라도 측정 근거를 보고.

## 3. 구현 환경

| 항목 | 실제 사용값 |
|---|---|
| LiDAR 모델/스캔율 | TODO |
| RGB camera/FOV/해상도 | TODO |
| Raspberry Pi/OS/ROS2 | TODO |
| 노트북/GPU/전원 모드 | TODO |
| YOLO 가중치/코드 버전 | TODO |
| 융합 모델 아키텍처/학습 seed | TODO |
| 교정 행렬·오차 및 설치 높이 | TODO |

## 4. 실험 방법

- 바운딩박스와 LiDAR 스캔 기하학적 대응/시간 정합.
- background return, camera-only, ambiguous, stale 구분.
- B0/B1/B2/B3/B4 비교군과 입력/평가 공정성.
- 독립 GT와 세션 단위 데이터 분할.

## 5. 결과

`docs/12_EVALUATION_PLAN.md` 결과표를 실제 csv에서 생성하여 삽입. 사건별/조건별 재현율, FN, FA/min, 잘못된 거리 대응, p95 전체 지연. 단순 YOLO mAP는 별도 표로 구분.

## 6. 실패 및 제한

- 센서 모두에서 관측되지 않은 물체.
- 저조도·반사율·광원·대비·오클루전.
- 영상 박스의 높이/면적이 물리 거리·높이와 다름.
- calibration drift와 장치 간 시계 오프셋.
- 모델이 단순 규칙 대비 개선하지 않는 조건.
- 실험실/저속/시험 객체의 결과를 실도로로 일반화할 수 없음.

## 7. 결론과 확장

측정한 조건에서 어떤 경로가 미탐/오경고/지연에 어떤 영향을 미쳤는지 정리. 향후 ground-contact estimator, segmentation, range uncertainty, Pi-only, Nav2 virtual layer 등 검증 가능한 후속 연구 제시.

## 부록·재현성

- Git 커밋과 dependencies hash.
- 데이터 manifest와 적용한 데이터/모델 라이선스.
- 수치표 산출 명령/버전/실험 조건.
- 실제 Jev API 사용 여부(사용 안 했다면 모델 구분 문구 포함).
