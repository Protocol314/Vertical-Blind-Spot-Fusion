# 19. 구현 가능한 수식·핵심 의사코드

> 여기의 코드는 **계약을 설명하는 pseudocode**이다. 실제 ROS 타입·호스트 환경/검증·예외처리는 구현 에이전트가 작성해야 한다.

## 1. LaserScan to points

```python
def valid_laser_points(scan):
    points = []
    for i, r in enumerate(scan.ranges):
        if not isfinite(r) or not (scan.range_min <= r <= scan.range_max):
            continue
        theta = scan.angle_min + i * scan.angle_increment
        time_ns = scan.header_stamp_ns + round(i * scan.time_increment * 1e9)
        p_laser = (r * cos(theta), r * sin(theta), 0.0)  # ideal plane only
        points.append((i, p_laser, time_ns, r))
    return points
```

C1 실제 스캔 피치·장착 기울기는 외부 보정과 실제 시험으로 검증한다. 드라이버 `time_increment`가 0 또는 무효하면 `timing_quality=false`로 표시하고 강한 시간 정합을 주장하지 않는다.

## 2. Laser 3D endpoint to image

```python
def project(point_laser, T_camera_laser, K, D, image_shape):
    p_cam = transform(T_camera_laser, point_laser)
    if p_cam.z <= 0: return None
    uv = project_with_camera_model(p_cam, K, D)
    if not in_image(uv, image_shape): return None
    return uv
```

이미지를 미리 `undistort`한 경우 `D`를 중복 적용하지 않도록 주의. 원본 bbox 좌표계로 일치시켜야 함.

## 3. 후보 연결과 불일치 처리

```python
def make_candidates(detections, clusters, calibration, time_gate):
    candidates = []
    for det in detections:
        if det.is_stale:
            candidates.append(Uncertain(det, reason='EXPIRED'))
            continue
        cluster_candidates = []
        for clu in clusters:
            if not time_gate(det, clu): continue
            if not valid_transform(det, clu, calibration): continue
            projected = project_cluster(clu, calibration)
            if not overlaps_bbox_or_mask(projected, det): continue
            cluster_candidates.append((clu, compute_features(det, clu)))
        if not cluster_candidates:
            candidates.append(CameraOnly(det, measured_range=None,
                                         range_source='UNKNOWN'))
        else:
            candidates.append(ResolveOrAbstain(det, cluster_candidates))
    candidates.extend(LidarOnlyForUnusedClusters(clusters))
    return candidates
```

`ResolveOrAbstain`이 box-overlap만으로 확정하지 않도록 foreground/background 검사, 여러 객체 공유 점 중복 방지, 격자·거리 점프 확인을 넣는다.

## 4. 부정확한 거리 배정 차단

```python
def safe_fused_range(candidate):
    if candidate.status == 'confirmed_fused' and candidate.association_validated:
        return MeasuredRange(candidate.cluster_distance_m, source='LIDAR_MEASURED')
    if candidate.has_independently_validated_image_metric_estimator:
        return EstimatedRange(candidate.estimated_m, source='IMAGE_ESTIMATED',
                              std_m=candidate.estimated_std_m)
    return UnknownRange(value=None, source='UNKNOWN')
```

**경고:** 현재 알고리즘이 완벽히 전경/배경을 구별할 수 있다는 가정을 하지 말 것. 불명확하면 UNKNOWN이 정답에 가까울 수 있다.

## 5. 판단/TTL 규칙

```python
def fuse_with_policy(candidate, ml_score, now_ns, config):
    if candidate.is_too_old(now_ns, config.max_age_ms):
        return status('stale', decision='UNKNOWN')
    if candidate.has_verified_near_field_lidar_hit():
        return warn_measured(candidate)  # never override to CLEAR by ML
    if candidate.status == 'camera_only':
        # local model may prioritize the alert but cannot invent distance
        return warn_unknown_range(candidate, risk_score=ml_score)
    if candidate.status == 'ambiguous':
        return mark_ambiguous(candidate, risk_score=ml_score)
    return decision_for_validated_candidate(candidate, ml_score)
```

## 6. 학습 특징(열 고정)

```python
features_v1 = [
    'yolo_conf', 'box_cx', 'box_cy', 'box_w', 'box_h',
    'projected_hit_count', 'lidar_cluster_size',
    'nearest_range_m', 'range_spread_m', 'angle_overlap',
    'pixel_residual', 'pair_skew_ms', 'camera_age_ms',
    'lidar_age_ms', 'calibration_error_px',
    'has_lidar_support', 'has_valid_range', 'has_tf',
    'class_one_hot_*'  # vocabulary expanded with stable ordered list
]
```

실제 계산 전에 마스크·스케일·class vocab 명세가 필요. `unknown` 상태를 다른 물체 클래스로 강제 변환하지 않는다.

## 7. 연산 병렬화

```text
Time t0: camera capture ------------------> YOLO inference ---┐
Time t1: lidar ray + full scan -----------> preprocessing ----┤
                                                         sync/projection
                                                               ↓
                                                         tiny model
                                                               ↓
                                                         decision/output
```

병렬 처리한다고 `t0`와 `t1`이 일치하는 것은 아니다. 데이터 시각 정합과 오래된 프레임 처리 정책을 별도로 적용한다.

## 8. 레이저 복사 원칙

원본 `LaserScan`은 그대로 저장/발행. 각도별 **위험 스코어 벡터**는 별도 `/fusion/risk_sectors`에 저장. 이것을 `/scan`으로 발행하거나 LiDAR의 물리 거리로 오해하게 하지 않는다.
