# 06. YOLO26n 영상 인식·학습 파이프라인

## 기본 모델 선택

- 1차 모델: **Ultralytics YOLO26n Detect (`yolo26n.pt`)**. 작은 크기와 학습·배포 생태계를 기준으로 선택.
- 선택적 세그멘테이션: **`yolo26n-seg.pt`**. 객체별 마스크가 거리 대응 개선에 유의미한지 확인했을 때만 추가.
- 별도의 단안 깊이 모델은 MVP 기본 입력이 아님. 객체 탐지 모델의 bounding box에는 실제 metric depth가 포함되지 않는다.
- 최신 Ultralytics는 NMS-free 헤드를 **옵션**으로 지원. 실제 API 기본값/내보내기 옵션은 고정 버전에서 확인한다.

## 추론 입력 계약

```text
image_id: string (session_id + frame_seq)
frame_stamp_ns: int (capture time, not processing completion)
frame_id: camera_optical_frame
image_width, image_height
image_data: BGR/RGB as explicitly declared
camera_calibration_id
```

## YOLO 출력 계약

```text
detection_id, frame_stamp_ns, class_id, class_name,
confidence, bbox_xyxy_in_original_image,
optional track_id, optional mask_ref,
inference_started_ns, inference_finished_ns,
weights_hash, model_version
```

바운딩박스는 `xyxy` 픽셀 좌표, **원본 촬영 영상 기준**으로 정의. 이미지 letterbox, crop, resize 후 결과의 좌표 되돌림을 단위 시험에서 검증한다.

## 초기 클래스 후보

- 현장 시험: `box_low`, `obstacle_medium`, `obstacle_overhang`, `person`, `chair`, `unknown_visual` 등. 단, COCO 사전학습 모델이 즉시 지원하지 않는 클래스는 별도 수집·라벨 필요.
- **형상과 높이는 서로 다름.** `box_low` 클래스가 곧 실측 높이 10cm를 의미하지 않음.
- 수집 표적 종류가 적고 시간이 촉박하면 COCO 사전학습의 `box`, `chair`, `person` 등을 먼저 적용하고, 탐지되지 않는 작은 장애물에 한해 현장 파인튜닝. 객체 클래스 불일치와 사각지대 효과를 분리 측정.

## 학습 및 검증 단계

1. 기존 가중치에서 **현장 20–30개 장면**에 대한 추론을 먼저 조사하여 탐지 성공/실패를 정리.
2. 실패가 많으면 새 라벨의 수집·라벨링 범위 확정; 이미지마다 동일한 클래스 정의 적용.
3. **촬영 세션/실물 표적 단위**로 train/val/test 분할; 인접 프레임 섞기 금지.
4. 가급적 작은 사전학습 모델 파인튜닝; 고정 seed, epochs, optimizer, batch, imgsz, 학습한 클래스 기록.
5. 원본 가중치와 fine-tuned 가중치 모두 독립적인 test split에서 평가.
6. 클래스별 PR·Recall·mAP50·mAP50:95 + **장애물 이벤트 탐지 성공률** 별도 제시.
7. 추론 결과로부터 융합 특징을 생성할 때 train/test 누수를 피함. 경량 융합 모델에는 YOLO가 학습에 사용한 영상을 직접 동일 조건으로 재사용하지 않도록 fold 구분.

## 초깃값 예시(성능 보장 아님)

```python
from ultralytics import YOLO

model = YOLO("yolo26n.pt")
results = model.predict(source="test.jpg", imgsz=640, conf=0.20)
# 결과 박스의 원본 좌표, class/conf, 모델 및 캡처 timestamp를 데이터 계약에 매핑.
# 모델 호출이 수행된 시각으로 촬영 시각을 덮어쓰면 안 됨.
```

학습:

```python
model.train(data="configs/obstacles.yaml", imgsz=640, epochs=30, seed=42)
```

`30 epoch`·`0.20`은 **예시 설정**이며, 실제 데이터 크기·과적합·재현율/오탐 곡선을 보고 조정한다.

## 성능/이식성

- RTX 4060에서 실측 `preprocess / inference / postprocess / decode / transfer` 구간의 시간을 따로 측정.
- ONNX/TensorRT 변환은 **모델 정확도 회귀 테스트 통과 후** 진행; 출력 헤드/NMS 처리 변경에 주의.
- Pi 4B에서 YOLO26n 자체를 실행하는 것은 MVP 필수가 아니다.
- 해당 모델과 라이브러리의 라이선스(AGPL-3.0 등)는 `15_RISKS_SAFETY_LICENSE.md` 준수.
