# 11. 학습 데이터 수집·주석·세션 분할

## 데이터의 두 층

1. **YOLO 학습용**: 이미지와 객체 분류/위치 정답. 외부 공개 영상은 적절한 사용권·촬영조건 확인 후 사용.
2. **융합 학습/평가용**: 시간 동기화된 영상·LiDAR 스캔, 센서 보정값, 실제 물체의 크기·높이·거리, 위험/비위험 이벤트 라벨. **YOLO 학습 이미지 자체만으로는 센서 융합을 평가할 수 없다.**

## 추천 표적

- 낮은 박스·책·작은 블록 등 쉽게 반복할 수 있는 실제 물체(재질/반사율 기록).
- LiDAR 스캔 평면을 가로지르는 중간 높이 물체.
- 로봇 높이 내에 들어오는 매달린·돌출 장애물(안전하게 고정).
- 충분히 높아 통과 가능한 가로 구조물(**충돌하지 않는 음성 사례**).
- 그림자·포스터·평면 그림·색 무늬·그림자를 이용한 영상 오탐 사례.
- 벽 뒤 거리 반사가 박스 안에 들어오는 배경 교란 장면.

## 각 장면마다 기록할 메타데이터

```text
session_id, scene_id, recording_start_utc,
operator, device_id, camera_model, resolution, exposure,
laser_mount_height_m, camera_mount_height_m, pitch_deg,
calibration_id, transform_hash, rosbag_path,
object_physical_id, object_class, object_height_m, object_width_m,
object_ground_contact_bool, object_true_distance_m,
robot_collision_envelope_m, scenario_label,
lighting_condition, surface_material, reflectivity_note,
test_or_train_split, annotation_version, annotator_id
```

- `object_true_distance_m`는 줄자·별도 교정 장비 등을 통한 **독립적인 현장 계측**. LiDAR가 놓친 객체의 정답을 LiDAR 거리로 만들지 않는다.
- `ground_truth_hazard`는 **시험 로봇의 점유 체적과 실제 충돌이 예상되는가**를 기준으로 정의하며 객체의 이름/높이만 보고 라벨링하지 않는다.
- `blind_spot_type`: `below_plane`, `above_plane`, `intersects_plane`, `other_missing`, `unknown`. 높이 기준은 **실제 LiDAR 관측 단면**을 시험해 판단한다.

## 권장 수집량(계획값, 필수 수량 아님)

- 실험 시나리오 6종 × 3개 거리 × 2개 조명 × 5회 = 180회 *short recordings*은 확대 목표.
- MVP는 가능한 조건부터 **장면 다양성/독립적인 세션 우선**으로 모집. 데이터가 부족할 땐 반복 프레임을 늘리기보다 다양한 객체·거리·배경 확보.
- 센서 전원 재시작, 서로 다른 촬영 시간/위치, 별도의 실물 표적 등 독립 세션을 만들 것.
- 세션당 적어도 몇 초간 카메라·LiDAR 동시 수집하고, 시작/종료·객체 등장 시간을 주석.

## 학습·검증·시험 분할

- **분할 단위는 session_id/scene_id/physical_object_id 단위**. 동일 장면의 프레임을 train/test로 임의 섞지 않음.
- 예시 split 60/20/20이지만 **세션 수와 다양성**이 부족하면 group k-fold를 우선 검토.
- YOLO 학습에 포함된 실물/배경이 융합 시험에도 과도하게 노출되지 않도록 그룹 겹침 확인.
- 세션별 산출물이 모델 가중치 또는 threshold 선택에 쓰였다면 해당 세션은 blind test가 아님.
- 실험 조건별 결과와 전체 micro/macro 평균을 모두 보고.

## 어노테이션 레이어

1. `VisionGT`: 영상에서 실제 보이는 객체 bounding box 또는 mask.
2. `ObjectGT`: 실물 높이/거리/충돌 가능성/스캔 평면 교차.
3. `EventGT`: 특정 물체의 장면 진입·첫 발견 가능 시점·알림 허용 윈도우.
4. `AssociationGT`: 특정 LiDAR 군집이 **어느 실제 표면을 측정했는가**. 대응 없으면 NONE.
5. `FailureGT`: 센서 장애·측정 누락 이유를 확정 가능할 때만; 원인 불명이면 UNKNOWN.

## 데이터 검증 규칙

- 필수 참조: `session_id` ↔ raw rosbag ↔ camera frames ↔ annotation JSONL 일치.
- 모든 주석 파일은 변경 이력과 작성자, QA 상태 저장.
- 라벨 충돌은 독립 재검토 후 확정; 모델 예측을 GT로 그대로 복사 금지.
- 파일 해시(SHA256), 원본 보존, 공개·배포 시 개인정보 비식별 및 라이선스 확인.
- 동기화 기록이 불확실하면 융합 시험에 무조건 포함하지 않고 품질 플래그 표시.

## 외부 공개 자료의 역할

- KITTI: 카메라/3D LiDAR 정합·투영의 **오프라인 검증 참고**; C1 단일 평면 실측 데이터와 동일하지 않음.
- RoadObstacle21/분할 데이터셋: **영상 탐지 모델/미지 객체 평가 참고**; 실제 C1 동기화 데이터 제공 여부 별도 확인.
- COCO 사전학습: 일반 객체 검출 초기 가중치; 낮은/매달린 장애물의 실제 탐지 성능은 현장 평가 필요.
