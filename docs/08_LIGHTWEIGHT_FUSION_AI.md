# 08. Jev형 경량 의사결정 융합 AI

## 설계 철학

- **Jev형**은 '정형 상태 → 제한된 선택지/점수'라는 의사결정 패턴을 의미한다. 실제 TypeSafe Jev 모델을 다운로드하거나 동일 내부 알고리즘을 재현한다는 뜻이 아니다.
- 주 연산 경로에는 **로컬 모델**을 배치한다. API 응답시간·네트워크 실패를 안전 판단에 종속시키지 않는다.
- 모델이 맡는 것은 **후보 연결의 신뢰도/경고 우선순위 판단**이지, 물리적으로 측정되지 않은 높이·거리를 추론 확정하는 일이 아니다.

## 입력 특징 v1

특징 순서·정규화 방법·단위·결측 마스크를 **고정**하고 모델 파일과 함께 저장한다.

| 그룹 | 특징 | 타입/취급 |
|---|---|---|
| Vision | class_id(원-핫), yolo_conf | 범위 확인; class vocab 고정 |
| Vision | bbox cx,cy,w,h (normalized) | [0,1] 이미지 좌표 |
| Vision | track_age_frames | 추적 없으면 mask=0 |
| LiDAR | valid_return_count | 0은 실제 0개, 누락과 구분 |
| LiDAR | nearest_range_m, median_range_m | 미측정 시 null+mask=0 |
| LiDAR | range_iqr_m, range_jump_m | 유효 그룹이 있을 때만 |
| Geometry | projected_hit_count_in_box | 영상 좌표계 정합 후 |
| Geometry | overlap_ratio, point_box_residual | 유효한 경우에만 |
| Temporal | abs_skew_ms, camera_age_ms, scan_age_ms | 캡처 시각 기준 |
| Quality | calibration_quality, tf_available | 정량 품질/불리언 |
| Missingness | image_valid, lidar_valid, range_valid | 결측 명시 |

**금지:** 없는 LiDAR 값을 `0.0m`로 대체해서 신경망이 '근접 충돌'로 학습하게 만드는 방식. 결측 대체는 훈련시 정한 스케일/마스크가 반드시 따라야 한다.

## 모델 옵션

1. `rules`: 센서별 후보 유지 + 유효시간·대응 규칙. 재현 가능한 베이스라인.
2. `logreg`: 로지스틱 회귀, 작은 데이터에서 비교 지표 확보.
3. `gbdt`: 데이터가 작고 비선형 관계가 필요할 때 트리 계열(추가 라이브러리 필요성 평가).
4. `mlp`: 입력 24–40차원(클래스 인코딩에 따라 변동), 은닉층 64→32, 1~2개 확률 출력 예시.
5. `1d_cnn`: 각도별 연속 스캔 특징을 사용할 경우만 후속 실험.

처음부터 딥러닝 모델을 학습시킬 데이터가 부족한 경우, 규칙 모델의 성능이 더 좋을 수 있다. 이 가능성도 결과로 받아들인다.

## 출력 인터페이스

- `p_hazard`: 정의된 평가 환경에서 실제 충돌 위험 장애물일 점수(**학습된 위험 확률이라는 주장은 calibration 평가 후**).
- `p_association_valid`: 동일 객체에 대한 유효 LiDAR 대응 가능성 점수(별도 GT 필요; 모델 추가 시에만).
- `status`: `confirmed_fused|camera_only|lidar_only|ambiguous|stale`는 **모델 단독으로 조작 불가**, 기하·검증 게이트가 최종 결정.
- `decision`: `WARNING|UNKNOWN|CLEAR_IN_TESTED_AREA` (시험 범위 외에서 전방 전체 안전 선언 금지).
- `source_flags`, `range_source`, `range_m`, `reason_codes`.

## 모델 학습 레이블

- `is_true_hazard`: 시험 플랫폼의 주행 체적과 겹쳐 위험한 객체인가? 물리적 정답과 시나리오 정의로 결정.
- `is_valid_association`: 영상 객체와 특정 LiDAR 거리군집이 실제 같은 물체 표면을 관측했는가? 독립 정답으로 결정.
- `blind_spot_type`: `below_plane|above_plane|intersects_plane|other_missing|unknown`.

하나의 학습 타깃에 세 문제를 혼합하지 않는다. **최소 MVP는 위험 점수 1개에 집중**하고, 연관 판단은 규칙+후보 선택으로 처리.

## 학습 및 누수 방지

- `session_id`, `scene_id`, `physical_object_id` 기준으로 학습/검증/시험 분리; 인접 프레임 무작위 분할 금지.
- 이미지 모델은 테스트 세션을 보지 않은 상태로 동결; 융합 학습의 정답은 YOLO 예측값이 아니라 독립 주석.
- 클래스 불균형이 높을 수 있으므로 Precision/Recall tradeoff로 임계값 선택. **검증 split**에서만 threshold 조정.
- 확률 보정이 필요하면 검증 데이터에서 Platt/isotonic 등의 선택을 시험; 데이터가 적으면 과적합 주의.
- 추론시간과 메모리도 동일 장치/입력 크기에서 측정.

## 안전 게이트

```text
if input stale or missing calibrated transform:
    retain independent observations; output AMBIGUOUS/STALE
elif valid lidar near-field hazard:
    do not suppress due to low vision or ML score
elif camera-only visible but no validated range:
    warn as visual candidate; range=null
else:
    model score can prioritize/attenuate alert within evaluated policy
```

`CLEAR_IN_TESTED_AREA`는 다중 관측이 유효한 정의된 범위에서만 사용하고, 카메라 FOV 밖이나 LiDAR가 미관측인 3차원 공간 전체의 안전을 보장하지 않는다.

## 성능 기준

- **후처리 목표(안)**: 로컬 모델 포함 융합 계산 p95 5ms 미만. **측정 전인 TARGET**.
- 입력 주기: LiDAR 일반 10Hz, 영상은 가능한 입력 FPS에서 최근 관측만 소비.
- 실제 측정: p50/p95/p99, CPU/GPU 점유, 큐 길이, 프레임 드롭, 데이터 age, FNR·오탐.

## Jev 대비 공정한 비교

- 동일한 structured features를 전달하고, API/모델의 요청 지연과 정확도를 따로 기록.
- 로컬 모델은 on-device, Jev는 원격 API를 사용했다면 **시스템 아키텍처의 차이**를 명시한다.
- 실시간 필수 경로에 Jev를 대기시키지 않는다. `09_JEV_OPTIONAL_COMPARISON.md` 참고.
