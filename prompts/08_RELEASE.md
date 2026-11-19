# PHASE 08 — 시연 패키지·연구 보고서·선택 확장

## 구현 요청

`docs/15_RISKS_SAFETY_LICENSE.md`, `docs/16_14_DAY_PLAN.md`, `docs/18_ACCEPTANCE_CRITERIA.md`를 검토하고 마무리하라.

1. clean checkout 기준 설치 및 실행 가이드(라이브/오프라인/replay) 제공.
2. `config`별 모델 경로/보정/데이터 출처/threshold 설명.
3. **B0~B4** 동일 조건 수치표·실패 사례·해석과 하위/상위 사각지대별 정확도 비교를 보고서에 작성.
4. 실제 촬영·모델 학습·API 호출 여부와 미검증 항목을 명시.
5. 데모 화면은 raw camera+YOLO detections, actual laser points, fusion status+range source, diagnostics를 한눈에 보여주되 **거리 미확정 객체는 거리 숫자를 표시하지 않음**.
6. 사용자 실험 원본/개인정보/수십 MB+ 가중치/비밀키는 패키지에 넣지 않고 제공 방법과 라이선스를 정리.
7. 발표용 요약: 선정 배경, 연구 가설, 기존 연구 차이, 데이터·보정, 결과, 실패/확장.
8. `PROJECT_STATUS.md`, `docs/21_ADR_AND_CHANGELOG.md`, 테스트 체크리스트를 실제 증빙으로 업데이트.

## 확장 조건

G0-G3를 모두 통과했을 때만 (a) `yolo26n-seg`, (b) 단안/바닥접점 거리 추정, (c) Jev 비동기 비교, (d) ROS2 Nav2 custom virtual layer, (e) Pi-only 최적화를 우선순위대로 제안.

## 선택적인 Jev 비교 작업(별도 사용자 허용 필요)

- 사용자에게 실제 원격 API 호출, 전송할 자료 범위와 비용을 확인 받는다.
- 인증키는 환경변수, 이미지는 기본 전송 금지; structured features로만 시작.
- `POST /v1/systemone`의 **공식 요청 스키마를 검증**하고 타입을 `Choice`/`Score` 등으로 제한.
- API timeout/429/실패는 로그만 남기고 real-time fusion은 동작 지속.
- 브라우저 워크플로의 Jev 속도를 로봇 측정값으로 주장하지 않는다.

## 최종 인수 선언 형식

```text
완료된 Gate: G0/G1/G2/G3
미완료 및 원인:
실제 실험 세션 수 및 조건:
실측 결과 파일 및 해시:
테스트 명령과 실행 환경:
실시간 지연 p50/p95/p99:
성능 주장에 대한 제한:
```
