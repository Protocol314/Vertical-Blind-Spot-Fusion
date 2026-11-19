# CLAUDE.md — Claude Code 구현 지침

이 프로젝트의 표준 계약은 [`AGENTS.md`](AGENTS.md)이다. **AGENTS.md 전문과 README.md를 먼저 읽고 동일하게 준수**한다. 이 문서의 지침은 두 에이전트가 서로 다른 구현을 하더라도 결과 계약을 동일하게 유지하기 위한 것이다.

## Claude Code 첫 행동
1. `AGENTS.md` → `README.md` → `docs/00_PROJECT_BRIEF.md` → `docs/18_ACCEPTANCE_CRITERIA.md` 순으로 읽는다.
2. `prompts/00_MASTER_START.md`를 실행 계획으로 삼고 `prompts/01_SCAFFOLD.md`부터 단계별 구현한다.
3. 의사결정·데이터 형식 변경 전 `docs/10_DATA_AND_INTERFACE_CONTRACTS.md`, `docs/21_ADR_AND_CHANGELOG.md`를 확인한다.
4. 생성 코드·경로는 `README.md`의 권장 저장소 구조를 기본으로 한다.

## Hard constraints
- YOLO의 **학습 데이터셋**을 매 프레임 검색하지 않는다. 운용 중 처리할 것은 YOLO **추론 결과**다.
- Jev API 또는 LLM에 10Hz 실시간 안전 결정을 종속시키지 않는다.
- `camera_only` + `range_m=null`을 지원한다. 필요 없는 강제 3D 보간·가짜 거리 삽입 금지.
- 실제 기록 없이 '10Hz 성공', '미탐률 X% 개선', '실시간 검증 완료'라 쓰지 않는다.
- 사용자의 기존 Pi/ROS2 설정을 삭제하거나 덮어쓰지 않는다.

## 단계 완료 시 출력 형식
```text
구현 단계 / 변경 파일 / 실제 실행 명령 / 통과 테스트 / 알려진 미구현 / 재현 방법 / 다음 단계
```

## 요청 구문
기본 순서: `prompts/00_MASTER_START.md` → `prompts/01_SCAFFOLD.md` → ... → `prompts/08_RELEASE.md`.
