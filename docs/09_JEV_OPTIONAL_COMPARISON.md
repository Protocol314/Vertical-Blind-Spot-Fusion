# 09. TypeSafe Jev 선택적 비교 및 불가 범위

## 2026-11-17 기준 확인 내용

- TypeSafe Jev의 최신 문서상 모델: **`jev-1.14.0`** (출시 2026-10-14). 공식 `POST /v1/systemone` 호스팅 API 사용.
- 정형 질문 형식은 Choice / Score / Noul / Box. Jev 1.13/1.14는 PNG/JPEG 이미지 입력을 지원하며 Box는 1.14에서 지원.
- 이미지/Box API는 실험적이며 문서상 별도 header (`TypeSafe-Danger-Zone-Images: true`)를 요구. 구현 전 API 스키마를 최신 문서로 다시 검증.
- 공식 로컬/다운로드 가중치는 검증하지 못함. 따라서 Pi에 Jev 자체를 올려 실행하는 것을 MVP 요구사항으로 삼지 않음.
- TypeSafe 공개 소개의 **70–500ms** 대략적 API 응답 범위는 특정 인터넷·워크로드에서의 회사 측 주장. 실제 10Hz 센서 판단 보장 아님.
- `jev-ultrafast` 브라우저 실험에서 중앙 결정 지연 **약 178ms**라는 기록이 있지만 장애물 융합 벤치마크가 아니다.

출처: `https://docs.typesafe.ai/models`, `https://docs.typesafe.ai/images`, `https://typesafe.ai/blog/introducing-system-one-models-and-jev`, `https://github.com/browser-use/jev-ultrafast/blob/main/docs/performance.md`.

## 실험 모드

```text
Local fusion -> publish safety-independent result (never waits for Jev)
             -> telemetry copy -> async Jev API client
                                -> offline JSONL comparison only
```

### Jev에 보낼 수 있는 상태 예시(의사 계약, 실제 API 호출 payload 아님)

```json
{
  "state": {
    "detection_class": "box",
    "vision_conf": 0.83,
    "lidar_support_count": 0,
    "range_m": null,
    "range_valid": false,
    "observation_age_ms": 42,
    "calibration_status": "verified"
  },
  "question": "Should this be retained as a camera-only hazard candidate?",
  "allowed_answers": ["retain", "abstain", "discard"]
}
```

**주의:** 위 JSON은 시험 설계 설명용 추상 형태이다. 실제 Jev System One API는 `state`, `questions` 객체 및 각 질문 타입 등 **별도의 공식 스키마**를 사용하므로 그대로 호출하지 말 것. API가 허용한 Choice 질문 등으로 변환해야 한다.

## 선택 비교 평가

| 지표 | 로컬 LR/MLP | 실제 Jev |
|---|---|---|
| 조건 | 고정 모델·동일 특징 | 동일 정보·요청 문맥(프롬프트 포함) |
| 정확도 | 시험 세션 기반 | 동일 시험 세션 기반 |
| 지연 | on-device 전체 시간 | 직렬화+네트워크+API 왕복 |
| 실패 | process error | timeout, 429, auth, transport |
| 정보 전송 | 기본 로컬 | 원격 서버, 사용자 동의 필요 |

측정 주의: p95/p99, 시간대/지역, rate limit, 호출 수, 모델 버전을 남기고 API 호출량·비용·민감한 영상 처리 정책 확인. 개인 영상·연구 녹화 원본을 API에 업로드하지 않는 것을 기본값으로 한다.

## 결론

이 프로젝트에서 **Jev 자체를 사용했다**고 발표하려면 실제 API 호출과 결과 로그가 필요하다. 호출하지 않았다면 **“Jev와 유사한 구조화·경량 판단 방식에서 착안한 로컬 모델”**이라고 정확히 표현한다.
