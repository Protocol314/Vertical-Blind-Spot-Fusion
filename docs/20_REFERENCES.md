# 20. 참고 문헌·공식 문서·증거 수준

> 확인 기준일: 2026-11-17. 아래 외부 출처의 내용은 개발 시 최신 상태·API 호환성을 재확인한다. 공식 문서와 원 논문을 우선하고, 블로그/2차 자료는 보조 참고한다. 숫자가 비교 환경에 따라 달라질 수 있으므로 보고서에서 임의로 일반화 금지.

## 핵심 공식 문서

1. **SLAMTEC.** RPLIDAR C1 Specifications. <https://www.slamtec.com/en/c1/spec> — 전형적 10Hz, 8–12Hz, 5kHz 샘플링, 반사율 의존 측정 범위, pitch 관련 사양. `FACT`.
2. **SLAMTEC.** C1 Datasheet v1.1. <https://bucket-download.slamtec.com/2d4664be9f9f5c748f3b608f2cf1862962b168eb/SLAMTEC_rplidar_datasheet_C1_v1.1_en.pdf> — 실제 적용시 원본 데이터시트와 드라이버 설정 확인. `FACT`.
3. **Ultralytics.** YOLO26 models. <https://docs.ultralytics.com/models/yolo26> — `yolo26n.pt`, train/inference, detection heads. `FACT`.
4. **Ultralytics.** YOLO export. <https://docs.ultralytics.com/modes/export> — ONNX/TensorRT 등; 내보내기 모드마다 헤드·NMS 차이 주의. `FACT`.
5. **Ultralytics.** Detection & Segmentation. <https://docs.ultralytics.com/tasks/detect>, <https://docs.ultralytics.com/tasks/segment>. `FACT`.
6. **ROS2 Humble.** `LaserScan` message. <https://docs.ros.org/en/humble/p/sensor_msgs/msg/LaserScan.html> 또는 canonical source <https://github.com/ros2/common_interfaces/blob/rolling/sensor_msgs/msg/LaserScan.msg>. `header.stamp` 첫 광선 정의, `time_increment` 등. `FACT`.
7. **ROS2.** message_filters Approximate Synchronizer. <https://docs.ros.org/en/rolling/p/message_filters/doc/Tutorials/Approximate-Synchronizer-Cpp.html>. 구현 시 Humble 버전과 QoS 확인. `FACT`.
8. **Nav2.** Obstacle Layer parameters. <https://docs.nav2.org/rolling/configuration_and_development/configuration_guide/core_servers/costmap_2d/costmap_plugins/obstacle/> — `marking`과 `clearing` 별도; 추정 장애물의 공간 clearing 방지. `FACT`.
9. **TypeSafe.** Jev models. <https://docs.typesafe.ai/models> — 2026-11-17 확인 가능한 `jev-1.14.0`, Text/Images, Choice·Score·Noul·Box, hosted API. `FACT`.
10. **TypeSafe.** Images and experimental features. <https://docs.typesafe.ai/images> — 이미지/Box는 별도 header, 실험적 기능. `FACT`.
11. **TypeSafe.** Introducing System One Models & Jev (2026-09-15). <https://typesafe.ai/blog/introducing-system-one-models-and-jev> — 회사의 속도 주장 **벤치마크 일반화 금지**. `VENDOR CLAIM`.
12. **browser-use/jev-ultrafast.** Performance documentation. <https://github.com/browser-use/jev-ultrafast/blob/main/docs/performance.md> — 브라우저 작업 환경의 지연 기록. 센서 지연 근거가 아님. `OBSERVATIONAL, DOMAIN-SPECIFIC`.
13. **Ultralytics.** AGPL-3.0 compliance guidance. <https://docs.ultralytics.com/help/contributing> — 배포 법적 검토 필요. `FACT`.
14. **OpenCV.** Camera calibration / projectPoints. <https://docs.opencv.org/4.7.0/d9/d0c/group__calib3d.html> — 기하학. `FACT`.

## 핵심 선행연구

15. **Young, J.; Simić, M. (2015).** *LIDAR and Monocular Based Overhanging Obstacle Detection.* Procedia Computer Science 60, 1423–1432. <https://doi.org/10.1016/j.procs.2015.08.218>. 본 주제의 직접 선행연구. `PRIMARY PAPER`.
16. **Fu, C.; Dong, C.; Mertz, C.; Dolan, J. M. (2020).** *Depth Completion via Inductive Fusion of Planar LIDAR and Monocular Camera.* <https://arxiv.org/abs/2009.01875>. 평면 LiDAR+영상 특징 깊이 보완 참고. `PRIMARY PAPER`.
17. **Han, Y.; Zhan, I. H.; Zhao, W.; Pan, J.; Zhang, Z.; Wang, Y.; Liu, Y.-J. (2022).** *Deep Reinforcement Learning for Robot Collision Avoidance With Self-State-Attention and Sensor Fusion.* <https://cg.cs.tsinghua.edu.cn/people/~Yongjin/2022-Deep-Reinforcement-Learning-for-Robot-Collision-Avoidance-With-Self-State-Attention-and-Sensor-Fusion.pdf>. 1차원 거리/영상 융합 관련. `PRIMARY PAPER`.
18. **Benayed, W.; Mabrouk, I.; Masmoudi, M. S.; Ben Abdelaziz, W. (2025).** *LiDAR 2D and Camera Fusion for ADAS: A Practical Approach With YOLOv9 and ROS2 Framework.* IEEE Access 13, 155500–155519. <https://consensus.app/papers/benayed-mabrouk-lidar-2d-and-camera-fusion-for-adas-a-practical/w:b1023efe-afaa-45ce-bc41-116b7f5f030f/?utm_source=chatgpt>. `PRIMARY PAPER`.
19. **D’Antona, A.; Farsoni, S.; Rizzi, J.; Bonfè, M. (2026-10-26).** *A Depth-Centric Review of LiDAR–Camera Fusion: From 2D Sensors to High-End Perception for Autonomous Systems.* Electronics 15(21), 4945. <https://doi.org/10.3390/electronics15214945>. `PEER-REVIEWED REVIEW`.
20. **Vats, V.; Nizam, M. B.; Davis, J. (2024).** *VaLID: Verification as Late Integration of Detections for LiDAR-Camera Fusion.* <https://arxiv.org/html/2409.15529v1> — 작은 MLP의 후기 판단 참고. 단, **이 연구는 LiDAR 검출을 검증하는 방식이므로 없는 LiDAR 객체를 자동 복원하지 않음**. `PREPRINT`.
21. **Abaza, B. F.; Staicu, A.-A.; Doicin, C. V. (2026).** *Lightweight Semantic-Aware Route Planning on Edge Hardware for Indoor Mobile Robots: Monocular Camera–2D LiDAR Fusion with Penalty-Weighted Nav2 Route Server Replanning.* Sensors 26(7), 2232. <https://doi.org/10.3390/s26072232>. 2D LiDAR+YOLO26n 각도 섹터 결합 사례. `PRIMARY PAPER`.

## 출처 사용시 주의

- `FACT`: 출처가 밝힌 특정 조건의 관찰/사양; 그 사실을 다른 센서로 확대 해석하면 안 됨.
- `DESIGN`: 본 프로젝트 자체 선택. 논문 실증 결과처럼 인용하지 말 것.
- `TARGET`: 사용 전 측정해야 하는 목표. 수락 기준이 있더라도 성능을 달성했다고 주장할 수 없음.
- 논문 일부의 이름/저자·연도·권호 등은 **원문 PDF/출판사 메타데이터를 최종 보고서 제출 전에 재확인**한다. 부정확한 DOI·서지사항을 임의 생성하지 않는다.
