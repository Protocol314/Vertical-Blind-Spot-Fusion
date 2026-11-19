# PHASE 06 — 구조화된 경량 AI의 학습·검증

## 구현 요청

`docs/08_LIGHTWEIGHT_FUSION_AI.md`, `docs/11_DATASET_AND_LABELING.md`, `docs/12_EVALUATION_PLAN.md`를 기준으로 실행하라.

1. 모듈 `feature_extractor.py`를 만들어 bbox/클래스/신뢰도/투영 ray 수/유효 거리 통계/각도/시간/보정 품질/결측 마스크를 추출한다.
2. 스키마 v1 고정, `feature_order`, class vocabulary, normalizer 및 null-imputation+mask 규칙을 버전으로 보존한다.
3. **정답 라벨은 물리적 측정·사건 주석**을 기준으로 사용하며 YOLO/라이다 예측을 그대로 재사용하지 않는다.
4. `GroupShuffleSplit`/그룹 K-fold 등으로 `session_id`/실물 ID 누수를 방지하는 assert를 만든다.
5. 규칙 기반 B3와 LR baseline, 선택적으로 작은 MLP(64→32)를 비교한다.
6. 모델/임계값은 train+validation에서 선택하고, test를 본 후 재조정하지 않는다.
7. 모델이 `camera_only` 물체를 false lidar-confirmed로 승격시키지 않도록 고정 안전 정책을 별도 구현한다.
8. 모델 artifact에 모든 환경·데이터 split·가중치 checksum·seed 기록.

### 실험

- B4 vs B3를 **동일 Recall** 조건에서 FA/min, time-to-first-alert, latency로 비교.
- 결측/드롭/배경-전경 불일치 사례별 혼동 행렬과 오류 분석.
- 데이터가 부족하면 LOOCV/그룹 검증 등 적용하고 **통계적 일반화 한계**를 설명.

### 수락 조건

- 기록이 있을 경우 수치와 출력 재현 가능.
- 기록이 없으면 training pipeline/test fixtures까지만 완료했다고 보고하고, 실측 성능은 N/A.
- 보정/실측/거리 생성 책임을 AI에 옮기지 않음.
