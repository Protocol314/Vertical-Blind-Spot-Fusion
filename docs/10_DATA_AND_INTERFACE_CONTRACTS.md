# 10. 데이터 형식 및 모듈 인터페이스 계약 v1

## 공통 규칙

- 장치/촬영/레이저 시간은 **UTC epoch ns 또는 ROS clock 타입 하나로 명시**, 혼용 시 변환 로그 필수. `time_domain` 필드로 시스템/ROS/시뮬레이션 시간 구분.
- 거리 단위 `m`, 각도 `rad`, 픽셀 `px`; 모든 값의 단위는 필드명 또는 스키마에 반영.
- `null`은 측정 부재, `0.0`은 실제 숫자. `NaN`은 JSON에서 금지하고 `null+valid_flag=false`로 정규화.
- `model_id`, `calibration_id`, `session_id`, `config_hash`를 함께 저장.
- 원본 LiDAR와 RGB 프레임은 별도로 보존하고, 결합 메시지는 **참조 ID**를 통해 추적.

## 1. VisionDetection

```json
{
  "schema_version": 1,
  "session_id": "S001",
  "frame_id": "cam_000123",
  "camera_optical_frame": "camera_optical_frame",
  "stamp_ns": 1794890000123456789,
  "time_domain": "ROS_TIME",
  "image_size_px": [1280, 720],
  "model_id": "yolo26n-best-sha256-placeholder",
  "detections": [
    {"detection_id":"d001", "class_id":0, "class_name":"box", "confidence":0.82,
     "bbox_xyxy_px":[220,170,290,325], "track_id":null, "mask_id":null}
  ]
}
```

## 2. ScanFrame (raw reference)

```json
{
  "schema_version": 1,
  "session_id": "S001", "scan_id":"scan_000122", "frame_id":"laser",
  "stamp_first_ray_ns":1794890000100000000, "time_domain":"ROS_TIME",
  "angle_min_rad":-3.14159, "angle_increment_rad":0.01257,
  "time_increment_s":0.0002, "scan_time_s":0.1,
  "range_min_m":0.05,"range_max_m":12.0,
  "ranges_ref":"rosbag2:/scan@seq=122", "intensities_ref":null
}
```

이 JSON은 기록·비교용 래퍼 계약이며 ROS `LaserScan` 메시지를 재정의하려는 것이 아니다. C1의 파라미터는 기기의 실제 메시지를 우선 사용; 예시 숫자는 테스트용이며 항상 동일하지 않다.

## 3. FusedObject

```json
{
  "schema_version": 1,
  "session_id": "S001", "object_id": "o017", "frame_id": "base_link",
  "stamp_ns": 1794890000123456789, "time_domain": "ROS_TIME",
  "camera_detection_id":"d001", "lidar_cluster_id":null,
  "status":"camera_only", "reason_codes":["NO_VALID_LIDAR_ASSOCIATION"],
  "source_flags":["CAMERA"], "class_name":"box",
  "risk_score":0.74, "score_is_calibrated":false,
  "measured_range_m":null, "estimated_range_m":null,
  "range_source":"UNKNOWN", "range_std_m":null,
  "bearing_laser_rad":null,
  "bearing_camera_rad":0.14,
  "observation_age_ms":40, "pair_skew_ms":18,
  "calibration_id":"calib_001", "model_id":"fusion_mlp_v1",
  "expires_at_ns":1794890000323456789
}
```

**주의:** `risk_score=0.74`는 예시 수치이며 확률 보정이 검증되지 않았으므로 실제 사고 가능성 74%라는 의미로 쓰지 않는다. `bearing_camera_rad`는 카메라 기준이며 LiDAR 기준 방위와 직접 동일시하지 않는다.

### Enum 계약

```text
status = confirmed_fused | lidar_only | camera_only | ambiguous | stale
range_source = LIDAR_MEASURED | IMAGE_ESTIMATED | UNKNOWN
source_flags = CAMERA, LIDAR, TEMPORAL_HISTORY (0..n)
decision = WARNING | UNKNOWN | CLEAR_IN_TESTED_AREA
reason_codes = NO_VALID_LIDAR_ASSOCIATION | LOW_VISUAL_CONFIDENCE |
               BACKGROUND_RETURN_SUSPECTED | TIME_SKEW_EXCEEDED |
               TF_UNAVAILABLE | EXPIRED | CALIBRATION_INVALID |
               VALID_LIDAR_OBSTACLE | UNKNOWN_CAUSE
```

## 4. FeatureVector

```json
{
  "sample_id":"S001-d001-scan_000122", "schema_version":1,
  "feature_order_version":"fusion_features_v1",
  "features":{"yolo_conf":0.82,"projected_hit_count":0,"nearest_range_m":null,"age_ms":40},
  "masks":{"nearest_range_m":false,"image":true,"lidar_association":false},
  "label":{"is_true_hazard":true,"is_valid_association":false,"blind_spot_type":"below_plane"}
}
```

## 5. EvaluationEvent

```json
{
  "session_id":"S001", "event_id":"E007", "scenario":"below_plane",
  "object_physical_id":"T03", "ground_truth_hazard":true,
  "true_distance_m":1.5, "true_object_height_m":0.12,
  "first_visible_ns":1794890000000000000,
  "first_alert_ns":null,
  "alerted_in_window":false,
  "evaluated_method":"mlp_fusion",
  "notes":"illustrative only; not a measured record"
}
```

## 버전 관리 및 직렬화

- MVP는 JSONL + NumPy/rosbag2; API 호출의 입력은 별도 typed adapter.
- 실제 JSON Schema 생성은 `prompts/01_SCAFFOLD.md`에서 Codex가 생성하도록 지시한다.
- 위의 예시 수치/시각은 **가상 데이터**이며 실제 로그가 아니다.
- 스키마 변경 시 이전 결과를 덮어쓰지 않고 migration/버전 증가.
- 메타데이터에 `hardware`, `software_versions`, `git_commit`, `settings_hash` 저장.
