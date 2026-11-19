# 환경·장비 인수 체크리스트

- [ ] OS 버전(PC/Pi), CPU·RAM·GPU·드라이버/CUDA, Python 버전 확인.
- [ ] Pi: Ubuntu22.04 arm64/ROS2 Humble 실행 여부, 기존 `sllidar_ros2` 워크스페이스 경로 및 launch 기록.
- [ ] RPLIDAR C1: `/scan` 발행·토픽 타입·QoS·평균 scan Hz·frame_id·angle/range 범위 확인.
- [ ] 카메라: 장치 모델·렌즈·해상도·FOV·프레임레이트·timestamp의 의미 확인.
- [ ] 두 센서의 물리 장착 높이·방향·상대 위치 기록.
- [ ] `tf2` tree에서 `base_link`, `laser`, `camera_optical_frame` 정합 확인.
- [ ] PC/Pi 시계 오프셋·네트워크 연결/ROS DDS 발견 여부 확인.
- [ ] 사용자 파일/기존 프로젝트 백업 및 변경 금지 범위 확인.
- [ ] 가중치·공개 데이터셋 다운로드 용량·라이선스 확인(무단 설치/다운로드 금지).
- [ ] API 키 유무는 공개하지 말고, Jev off가 기본값인지 확인.
- [ ] 실제 하드웨어 접근 불가 시 동일 구조의 가상 fixture/replay 테스트 가능.

## 실측 기록 템플릿

```text
Timestamp:
Host OS/Python:
Pi OS/ROS:
ROS Workspace (do not overwrite):
LiDAR model/firmware:
Scan topic/frame/hz/QoS:
Camera device/resolution/fps:
PC GPU/CUDA/PyTorch:
Clock sync and observed offset:
Physical rig measurement:
Calibration ID:
Unknown fields:
```
