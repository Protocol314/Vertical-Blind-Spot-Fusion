# PHASE 03 — 내부·외부 보정, 투영, 대응 후보

## 구현 요청

`docs/04_CALIBRATION_GEOMETRY.md`, `docs/07_LIDAR_ASSOCIATION.md`, `docs/19_ALGORITHMS_PSEUDOCODE.md`를 읽고 진행하라.

### 구현 항목

- `CameraCalibration(K,D,image_size,calibration_id)` 로드/저장 및 값 검증.
- `RigidTransform(T_camera_laser,source_frame,target_frame,calibration_id)`를 명시적으로 보존. ROS base_link/laser/camera_optical 축 방향 혼동 방지.
- 2D LaserScan 각 ray를 이상적 `xy0` 끝점으로 변환, tf/extrinsic 적용 및 `cv2.projectPoints` 혹은 수학적으로 동등한 함수로 영상 투영.
- 왜곡 보정 전/후 영상 좌표계를 일관되게 쓰고 YOLO 박스가 원본 해상도 기준이라는 계약을 지켜라.
- 유효 점의 군집화와 여러 bbox의 projection overlap 후보 생성. 배경 거리/동일 광선의 이중 할당/원점 차이/시차를 검증.
- `no ray supports this YOLO box`를 결함이 아닌 **별도 camera-only candidate**로 보존. 이때 위험 객체의 `measured_range_m=None`.
- 시각적 calibration debug overlay와 held-out 표적의 reprojection residual 보고 코드(실측 시 수치 입력 필요).

### 회귀 테스트(필수)

1. 단위 행렬에서 광선 투영 golden result.
2. 카메라 뒤 점, 이미지 외 좌표, 왜곡 적용 중복 방지.
3. LiDAR 뒤쪽 벽(4m)이 이미지의 1m 낮은 박스 안에 투영된 경우 **박스 거리를 4m로 확정하지 않는다**.
4. LiDAR 실측 없는 box, box 겹침, frame ID 오류, 장착 pitch 변경.
5. 실제 calibration 파일 없이도 fixture로 기하학 테스트 가능.

수락 전까지 detector와 matched ray 관계는 '후보'이며 '같은 객체를 실측'이라고 서술하지 말 것.
