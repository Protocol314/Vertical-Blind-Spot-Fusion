# 05. ROS 2 Humble 입력·시간 동기화·QoS

## 실제 환경

- Pi 4B: Ubuntu 22.04.5 LTS arm64, ROS 2 Humble; 기존 `sllidar_ros2` 드라이버 실행 환경을 우선 보존한다.
- RTX 4060 노트북: OS·ROS 설치 상태 확인 전까지 결정을 유보. Windows 기본/WSL2/Ubuntu에 따라 DDS 멀티캐스트 및 CUDA 동작이 달라지므로 확인 로그 필요.
- 원본 `/scan` 토픽 `sensor_msgs/msg/LaserScan` 타입·frame_id·rate·QoS부터 검사한다.
- 카메라 `sensor_msgs/msg/Image`와 `CameraInfo`를 동일 시간 기준으로 기록한다. 카메라 드라이버가 제공하는 타임스탬프의 의미(노출 시작/종료/읽기 완료)를 기록한다.

## ROS2 표준 메시지 명세의 핵심

`LaserScan.header.stamp`는 **첫 광선의 측정 시각**으로 정의된다. `time_increment`는 광선 간 간격, `scan_time`은 스캔 사이의 간격이다. `ranges`는 부동소수점 배열이며, 드라이버에서 실제 시간 필드가 어떻게 채워지는지 확인해야 한다. 10Hz의 스캔 전체를 동일 순간으로 취급하면 움직임이 있는 장면에서 대응이 달라질 수 있다.

## 시계·스캔 시간 정책

1. 영상 취득 타임스탬프와 스캔의 첫 광선 시각을 복원한다.
2. 광선별 `t_ray_i = stamp + i*time_increment` 적용 가능 여부 검증; 드라이버의 필드가 0이거나 비정상인 경우 대체 정책을 로그에 표시한다.
3. tf2 변환 조회는 의도한 촬영/광선 시점 기준; 실패 시 최신 변환으로 **조용히 대체 금지**.
4. 정지 센서·정지 물체에서는 처음에 전체 스캔의 대표시각을 사용하는 간략 모드를 허용하되, `static_scene_only=true`라고 명시.
5. 장치 간 시계 동기화는 PTP/NTP/동일 호스트 시계 등 환경에 맞게 선택하고, **관측한 오프셋 및 지터**를 기록한다.

## 동기화 방식

- 녹화/재생: 개별 타임스탬프가 있는 raw 토픽과 `rosbag2` 사용.
- 온라인: 이미지와 스캔의 후보 페어를 작은 시간 창에서 검색하고 실제 `Δt`를 기록.
- 초기 `max_pair_skew_ms=50` 등은 **테스트 시작용 설정값**, 검증된 보장이 아님. 스캔 내부의 광선 시각 차이가 50ms보다 클 수 있으므로 물체·레이저의 실제 시점을 고려.
- ROS `message_filters` ApproximateTime은 시작점일 뿐, 물체별 대응 검증과 이동 보정을 대체하지 않는다.
- 입력 QoS 불일치로 콜백이 전혀 작동하지 않을 수 있다. SensorDataQoS/RELIABLE·BEST_EFFORT 조합을 확인하고 문서화.

## 토픽 / 서비스 예시

```text
/scan                   sensor_msgs/msg/LaserScan      [DRIVER; immutable]
/camera/image_raw       sensor_msgs/msg/Image
/camera/camera_info     sensor_msgs/msg/CameraInfo
/tf_static             tf2_msgs/msg/TFMessage
/vision/detections     custom or vision_msgs (selected by implementation)
/fusion/obstacles      custom msg or JSONL adapter (MVP)
/fusion/risk_sectors   custom msg or JSONL adapter
/fusion/diagnostics    diagnostic_msgs/msg/DiagnosticArray
```

실제 ROS 메시지 패키지를 생성하려면 `ros2_ws/src/.../msg/*.msg`와 설치/빌드가 필요하다. **MVP에서는 Python dataclass(JSONL 저장)로 계약을 먼저 통과하고, 이후 custom ROS message 어댑터를 생성**하는 방식을 권장.

## diagnostics

```text
camera_fps, lidar_scan_hz, yolo_fps, fusion_hz,
clock_offset_ms, pair_skew_ms, tf_age_ms,
queue_depth, dropped_frames, stale_ratio,
invalid_ranges, unmatched_camera, unmatched_lidar
```

## 현장 커맨드 (실제 작업 시 확인)

```bash
source /opt/ros/humble/setup.bash
ros2 topic list
ros2 topic info -v /scan
ros2 topic hz /scan
ros2 topic echo /scan --once
ros2 run tf2_ros tf2_echo base_link laser
```

`base_link`·`laser` 이름은 예시이며 실제 tf tree와 일치해야 한다. 기존 노드 실행이 없는 경우는 문서화된 기존 launch를 확인한 다음 **새 창**에서 실행한다. 이미 동작하는 ROS workspace는 정리/삭제하지 않는다.

## 수락 조건

- 동일 기록을 두 번 재생했을 때 pair 선택·stale 상태가 같아야 한다.
- 100ms 과거 영상과 최신 라이다를 무조건 현재 관측으로 승격시키지 않는다.
- tf missing, clock skew, LaserScan `NaN/+Inf/-Inf/0` 및 연결 끊김 테스트.
