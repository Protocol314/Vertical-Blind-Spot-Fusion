# PHASE 10 — 독립 리뷰 / 레드팀 요청문

## 복사 프롬프트

이 저장소를 **구현하지 않은 제3자 연구 심사자**로 검토하라. `AGENTS.md`, `docs/18_ACCEPTANCE_CRITERIA.md`와 전체 구현/테스트를 읽고 문제를 심각도 순으로 보고하라. 수정 제안은 구현 가능한 파일/함수/테스트 단위로 적어라.

반드시 다음을 검토:

- low object behind wall: 1m 물체에 4m 배경 LiDAR 거리를 잘못 부여하는가?
- hanging object: 지면 접점 없는 물체에 ground-plane range 추정을 적용하는가?
- frame_id/calibration: camera_optical_frame 및 변환 방향을 섞었는가?
- time: 캡처 timestamp 대신 추론 완료 timestamp를 사용하여 동적 물체 오류가 생기는가?
- missing: NaN/Inf/0/None과 유효성 마스크가 섞이는가?
- status: camera_only/no lidar returns 상황에서 객체가 버려지는가?
- source: physical scan이 영상 추론으로 조작되는가?
- data leakage: 인접 프레임/실물/같은 session이 train/test에 같이 있는가?
- ML: 조그마한 test 집합에서 threshold를 조정하는가?
- benchmarking: YOLO만의 inference time을 전체 경고 latency라고 주장하는가?
- Jev: 원격 API로 실시간 판단을 블로킹하는가?
- license/privacy: 가중치·녹화·API 키를 무단 공개하는가?

결과를 `REVIEW_REPORT.md`에 `Critical / High / Medium / Low`, 재현 입력, 위반 계약, 수정 방향, 필요한 테스트 순으로 기록하고, 실제 수정이 필요한 High 이상만 우선 처리하라.
