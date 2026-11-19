# PHASE 04 — YOLO26n 검출·분할 확장 옵션

## 구현 요청

`docs/06_YOLO26_PIPELINE.md`, `docs/11_DATASET_AND_LABELING.md`, `docs/12_EVALUATION_PLAN.md`를 읽고 아래 순서로 구현하라.

1. `VisionDetector` 프로토콜 정의; `StubVisionDetector`, `UltralyticsYOLO26Detector`를 구현하여 ROS 없이 단독 시험 가능하게 한다.
2. `yolo26n.pt`를 **탐지 모델**로 사용하고 bbox xyxy, confidence, class, frame original timestamp, model weights version을 출력.
3. Letterbox/resize 전후 좌표 역변환을 테스트하여 LiDAR projection과 같은 원본 영상 좌표가 되도록 한다.
4. 드라이버 타임스탬프를 유지; 추론 종료 시각은 별도 필드에 기록.
5. 사전학습만으로 낮은·매달린 물체 인식 가능한지 기록. 부족할 때만 사용자 수집 데이터에 대해 작은 fine-tuning 파이프라인 작성.
6. 데이터 분할은 물리 실물·세션·배경이 train/test에 반복되지 않도록 구현.
7. `yolo26n-seg.pt`는 **옵션 플래그**로만 포함하고, 실제 mask가 ROI 대응 오류 개선에 기여할 때 비교 결과를 제시.
8. Ultralytics 설치·모델 사용·배포 관련 AGPL-3.0 조건을 문서에 링크로 기록. 무단 대용량 데이터/가중치 다운로드 금지.

### 출력

- 고정 이미지에 대한 검출 JSONL과 debug overlay.
- model train/val/test config, seed, class list, actual path hash.
- 독립 시험 Recall/Precision/mAP와 에러 예시(실제 시험 전에는 N/A).

### 수락 기준

`StubVisionDetector` 없이도 로직 테스트 실행 가능. 단순 객체 탐지로 물리적 높이나 깊이를 '측정'했다고 표기하지 않음.
