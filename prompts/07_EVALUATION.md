# PHASE 07 — 평가·지연·오류·현장 재현성

## 구현 요청

`docs/12_EVALUATION_PLAN.md`, `docs/13_TESTING_AND_SIMULATION.md`, `docs/14_PERFORMANCE_DEPLOYMENT.md`, `docs/18_ACCEPTANCE_CRITERIA.md`를 기준으로:

- `EvaluationEvent`의 독립 GT를 읽는 검증 파서, 세션별 시험 분리 검사, 모든 baseline의 동일 로우데이터 재생 도구 생성.
- 아래/위/평면 교차 사각지대별 이벤트 TP/FP/FN, Recall/FNR/Precision/F1, FA/min, unknown/stale 비율을 산출.
- 잘못된 range association 비율과 null 거리 보존 위반 횟수 계산.
- frame이 아닌 **세션/사건 단위** confidence interval 제시; 표본 적으면 불안정성 설명.
- `capture_time → vision_completion → fusion_completion → publish` 구간의 p50/p95/p99, 실제 FPS, drop, queue, GPU 측정의 동기화 문제를 해결.
- 고장 주입(F01~F12)을 자동화하고 다음 조건 확인: stale=미승격, background=거리 미확정, Jev network failure=주 경로 불변.
- 테스트가 통과한 실제 명령만 사용 설명서에 넣기.

### 결과 보고서 필드

`method, session_count, event_count, blind_spot_type, TP, FP, FN, recall, fnr, fa_per_min, wrong_range_count, unknown_count, p95_capture_to_output_ms, git_sha, model_hash, config_hash`.

### 수락 조건

실제 센서 기록이 없으면 합성 fixture로 metric 함수 검증 **까지만** 보고. 프로젝트 성공/개선 수치는 독립 실제 시험 기록이 있을 때만 쓰라.
