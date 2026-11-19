# 14. 성능 예산·배포 방식·경량화

## 플랫폼

- Pi 4B 8GB: LiDAR 드라이버/ROS2 송신·진단과 로깅이 주 책임.
- RTX 4060 노트북: YOLO26n inference / 경량 모델 / evaluation.
- ROS Humble + Ubuntu22.04-arm64 환경과 RTX 노트북(Windows/WSL2 등)은 **연결 전 실제 상태 확인**. DDS discovery·시계 동기화 문제가 발생할 수 있음.
- 권장 순서: **동일 호스트 offline replay** → 네트워크 기록/재생 → 실시간 다중 장치.

## 목표값과 측정값을 구분

| 항목 | 초기 TARGET(실측 아님) | 반드시 보고할 MEASURED |
|---|---|---|
| LiDAR 수집 | 약 10Hz (C1 전형적인 스펙) | 평균·분포·drop |
| 융합 갱신 | 10Hz 수준을 지향 | 유효 result rate |
| 경량 판단 및 융합 후처리 | p95 5ms 이내 목표 | 실제 p50/p95/p99 |
| 전체 경고 지연 | 운영 시나리오에 따른 예산 설정 | 카메라 관측→경고 출력 p50/p95/p99 |
| `camera_only` range 정확도 | `null` 유지 | 의미상 위반 0건을 회귀테스트 |
| 실시간 오류 처리 | bounded queue와 TTL | max backlog, stale ratio |

10Hz LiDAR라고 해서 모든 광선이 같은 순간에 획득되거나, YOLO와 합쳐진 전체 판단이 반드시 100ms 이내라고 단정할 수 없다.

## 추론 최적화 순서

1. **동작하는 PyTorch 기준선 확보**: 정확도·시간 로그부터 확보.
2. 입력 이미지 크기 최적화: 작은 저형 장애물의 재현율 손실도 함께 평가.
3. 최신 프레임 우선 bounded queue: 오래된 영상 처리 backlog 금지.
4. YOLO 추론과 LiDAR 처리 병렬화, 단 결과 결합은 캡처 시각 기준.
5. 필요시 ONNX Runtime / TensorRT로 변환; `nms` head, 박스 좌표, class score 결과 회귀 비교.
6. 학습형 융합은 CPU small model 우선; GPU↔CPU 전송 비용 포함 측정.
7. 이미지 모델을 Pi에 올리는 최적화는 부가 실험으로 분리.

## 로그 필드와 구간

```text
t_capture, t_sensor_receive, t_vision_queue,
t_vision_start, t_vision_end,
t_association_start, t_association_end,
t_ml_start, t_ml_end, t_publish,
scan_first_ray, scan_last_ray,
clock_offset_estimate_ns, queue_depth, source_age_ms
```

GPU 추론 측정 시 asynchronous CUDA kernel launch 때문에 host-side `time.perf_counter`만 재면 실제 실행시간을 과소평가할 수 있다. 별도 CUDA event나 명시적 동기화 구간을 둔 정확한 벤치마크 방법을 사용.

## 안전 관점 예산

알림이 필요한 거리는 플랫폼 속도·운전자/제어 반응·제동거리·센서 최대 가시거리를 통해 결정되어야 한다. 수치가 정의되지 않았다면 '충분히 실시간'으로 표시하지 않는다. MVP는 **경고·시각화 시스템**이며 실제 제동 제어는 범위 외.

## 기록파일/버전 잠금

- `requirements-lock` 또는 실측 환경 보고서로 package version, Python, CUDA/torch, Ultralytics, ROS2 배포판, OS 버전 저장.
- `git rev-parse HEAD`, YOLO weights SHA256, tiny AI model SHA256, hardware calibration SHA256 기록.
- rosbag2 기록과 model inference 시간 등 원본 데이터는 버전·세션 정보와 연결.
