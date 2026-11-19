# 12. 실험·Ablation·통계 평가 프로토콜

## 평가 전 고정할 것

- 시험 세션 목록과 SHA256, 전체 장면 조건, 각 실험의 로봇 충돌 체적, 이벤트 진입/경고 시간 정의.
- 모든 모델은 **같은 raw recordings**를 입력으로 사용하고 같은 GT로 점수를 계산.
- 모델 학습/하이퍼파라미터/임계값은 train+val로만. Test를 본 뒤 임계값 수정 금지.
- 기록 재생은 가능한 동일 순서·속도. 런타임 latency는 replay와 online을 분리 보고.

## 비교군

| ID | 시스템 | 무엇을 확인하는가 |
|---|---|---|
| B0 | LiDAR-only | 단일 수평 스캔 관측의 한계 |
| B1 | YOLO-only | 영상으로 사각지대 인식 가능한가 |
| B2 | OR-fusion | 둘 중 하나라도 감지하면 후보 유지할 때 Recall/오탐 tradeoff |
| B3 | Geometric+rule fusion | 시간/투영/거리 게이트로 줄일 수 있는 잘못된 대응과 오탐 |
| B4 | Tiny-AI fusion | 구조화 특징 학습이 B3보다 개선하는가 |
| B5 | Jev shadow (선택) | 비동기 원격 분류와 B4의 정확도/지연 비교, 주 경로는 불변 |

**정의 주의:** B0에서 LiDAR의 점 검출과 B1에서 YOLO 객체 검출을 같은 '객체 탐지'로 평가하려면 **실물 위치와 이벤트 수준으로 연결하는 독립 정답**이 필요하다. LiDAR 점 1개를 YOLO box 1개와 직접 동일 단위로 세지 않는다.

## 이벤트 평가

- 객체가 로봇 앞 시험 영역으로 진입한 시점을 `first_observable`; 실제 위험 영역 내 객체를 `hazard` 라벨로 정의.
- 최초 경고가 평가 창 안에 있으면 해당 이벤트 TP, 누락이면 FN. 중복 경고는 최소 갱신 간격 기준으로 합산.
- 비위험 장면의 분당 경고 횟수(FA/min); 그림자/안전 통과 가능 장애물 오경고는 별도 목록.
- **카메라로도 보이지 않는 구간**은 방법별 검출 가능성 분리; '모든 물체를 잡을 수 있다'는 평가 금지.

## 기본 지표

\[
Recall=\frac{TP}{TP+FN},\quad Precision=\frac{TP}{TP+FP},\quad FNR=\frac{FN}{TP+FN}
\]

`F1=2PR/(P+R)` (분모가 0이면 정책에 따라 N/A). 이벤트 TP/FP/FN의 판정 윈도우 정의를 저장한다. `false_alarm_per_minute = false_alert_events / verified_negative_duration_minutes`.

**중점 지표:**
- 전체/하부 사각지대/상부 사각지대 Recall·FNR.
- **같은 Recall 목표**에서 B2/B3/B4 분당 오경고.
- 검증한 객체에 잘못된 **물리 거리**를 할당한 비율.
- 거리 추정(옵션)에는 MAE/MedianAE/표본 수/거부율/조건별 오차.
- `unknown|ambiguous|stale`의 빈도와 실제 위험 포함률.
- first-warning latency와 e2e capture-to-publish p50/p95/p99.

## 통계

- 프레임 수가 아니라 **독립 세션/이벤트 수**를 기본 표본 수로 공개.
- 세션 단위 재표본 bootstrap CI 예시를 사용해 불확실성 보고(충분한 표본 없으면 신뢰구간의 불안정성 명시).
- 학습 seed별 변동, 클래스·거리·조명·재질별 층화 결과 제시.
- 0% 미탐은 시험 조건 내에서만 말할 수 있다. 일반적인 안전성 보장이 아님.

## 지연 측정 정의

- `t_capture`: camera 프레임 관측 시각. 광선 시각도 별도 기록.
- `t_vision_complete`, `t_fusion_start`, `t_fusion_complete`, `t_publish`.
- YOLO 처리 지연과 전체 객체의 **관측 나이** 분리.
- 실제 두 장치 시계가 맞지 않으면 e2e 지연을 잘못 계산할 수 있으므로 동기화 검증 필수.
- 측정 환경: host OS / GPU/CPU / 전원 모드 / 프레임 사이즈 / model hash / 입력률 / 네트워크 / 데이터 녹화 여부.

## 결과표 템플릿

| 방법 | N events | Below FNR | Above FNR | FA/min | Wrong-range | p95 e2e ms | Unknown % |
|---|---:|---:|---:|---:|---:|---:|---:|
| B0 | TODO | TODO | TODO | TODO | TODO | TODO | TODO |
| B1 | TODO | TODO | TODO | TODO | TODO | TODO | TODO |
| B2 | TODO | TODO | TODO | TODO | TODO | TODO | TODO |
| B3 | TODO | TODO | TODO | TODO | TODO | TODO | TODO |
| B4 | TODO | TODO | TODO | TODO | TODO | TODO | TODO |

모든 `TODO`는 실측 전 채우지 않는다.
