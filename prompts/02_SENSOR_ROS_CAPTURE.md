# PHASE 02 — 센서 어댑터, 기록, 재생, 타임스탬프

## 구현 요청

Phase 01 완료를 전제로 `docs/05_ROS2_TIMING_AND_QOS.md`, `docs/10_DATA_AND_INTERFACE_CONTRACTS.md`, `docs/13_TESTING_AND_SIMULATION.md`를 읽고:

1. ROS 없이 작동하는 `ReplayAdapter`를 먼저 구현해 가짜 `LaserScan`과 이미지 detection 입력을 timestamp 기반으로 전달.
2. 실제 장비 사용 가능한 경우에만 `rclpy`/ROS2 Humble 어댑터를 작성하여 `/scan`, `/camera/image_raw`, `/camera/camera_info`, `/tf_static`을 구독/기록하는 코드 추가.
3. 원본 `LaserScan`은 immutable; 내부에서 invalid range를 정규화할 때도 복사.
4. `header.stamp=first ray`, `time_increment`, `scan_time`의 의미를 코드 주석/테스트로 반영.
5. `time_domain`, 다중 장치 오프셋·스큐, 오래된 이미지/스캔, tf lookup 누락의 상태를 추적. 초기 시간 게이트/TTL은 config에 둬라.
6. ROS QoS를 하드코딩하지 말고 `/scan` QoS 감지/설정 경로를 준비. 네트워크 DDS 동기화가 안 되는 경우 rosbag 오프라인 경로로 폴백.
7. 짧은 rosbag 저장/재생 도우미와 기록 세션 메타데이터 구현. 하드웨어 불가 상태에서는 모의 입력으로 일관된 테스트 출력.

### 필수 테스트
- first-ray vs per-ray timestamp 변환, 비정상적인 `time_increment` 품질 플래그.
- `NaN`, `Inf`, `0`, 음수, max 밖 range, 시간 역행, 100ms 이상 오래된 프레임.
- `/scan` raw sample/hash가 전처리로 변하지 않는지.
- offline replay가 연속 두 번 동일 output log를 만드는지.

실제 Pi에 접근하지 못하면 **장비에서 동작 확인했다고 주장하지 말고**, 코드/리플레이 테스트만 보고하라.
