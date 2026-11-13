# 참고 자료

"확인"은 해당 소스나 페이지를 직접 읽었다는 뜻입니다. "미확인"은 제목과 주소만 확인했고 내용은 읽지 않았습니다.

## 소스 코드

| 저장소 | 내용 | 읽은 범위 |
|---|---|---|
| [orangeduck/Motion-Matching](https://github.com/orangeduck/Motion-Matching) | 원본 구현 (C++, raylib). 기본 모션 매칭과 Learned Motion Matching | `database.h`, `spring.h`, `quat.h`, `common.h`, `lmm.h`, `generate_database.py` 전체. `controller.cpp`의 로직 부분(186~2093행). 렌더링과 `train_*.py` 세부는 읽지 않음 |
| [JLPM22/MotionMatching](https://github.com/JLPM22/MotionMatching) | 유니티 패키지. BVH 입력, Burst 탐색, Mecanim 리타게팅 | `MotionMatchingController`, `FeatureSet`, `Inertialization`, `SpringCharacterController`, `MotionMatchingData`, `PoseExtractor`, Burst 탐색 전체. 에디터 창, `Tags.cs`, 군중·경로 컨트롤러, 환경 인식 탐색은 읽지 않음 |
| [pau1o-hs/Learned-Motion-Matching](https://github.com/pau1o-hs/Learned-Motion-Matching) | Learned Motion Matching의 PyTorch 학습 코드와 ONNX. 유니티 쪽 코드는 저장소에 없고 태그 릴리스로 배포 | 학습 스크립트 4개 전체 |

### 원본에서 찾아볼 위치

| 주제 | 파일 | 함수 |
|---|---|---|
| 특징 계산 | `database.h` | `compute_bone_position_feature`, `compute_bone_velocity_feature`, `compute_trajectory_*_feature` |
| 정규화 | `database.h` | `normalize_feature` |
| 탐색 | `database.h` | `motion_matching_search`, `database_build_bounds` |
| 순기구학 | `database.h` | `forward_kinematics`, `forward_kinematics_velocity` |
| 스프링, 관성 보간 기본형 | `spring.h` | 전체 |
| 포즈 관성 보간 | `controller.cpp` | `inertialize_pose_transition`, `inertialize_pose_update`, `inertialize_root_adjust` |
| 시뮬레이션, 궤적 예측 | `controller.cpp` | `simulation_*_update`, `trajectory_*_predict` |
| 보정, 클램핑 | `controller.cpp` | `adjust_character_*`, `clamp_character_*` |
| 발 고정 | `controller.cpp` | `contact_update`, `ik_two_bone`, `ik_look_at` |
| 데이터 전처리 | `resources/generate_database.py` | 전체 |
| 기본 파라미터 값 | `controller.cpp` | `main` 앞부분 (1256~1420행) |

## 글과 문서

### 확인

- [Code vs Data Driven Displacement](https://theorangeduck.com/page/code-vs-data-driven-displacement): 시뮬레이션 오브젝트와 캐릭터의 동기화, 보정, 클램핑. 이 프로젝트의 7단계가 이 글입니다.
- [Inertialization Transition Cost](https://theorangeduck.com/page/inertialization-transition-cost): 관성 보간으로 생기는 변위량을 전환 비용으로 쓰는 방법. 위치·속도 특징을 `2·pos/y + vel/y²` 하나로 합칠 수 있습니다. 저자는 충분히 시험하지 않았다고 밝힙니다.
- [Dead Blending](https://theorangeduck.com/page/dead-blending): 이전 포즈를 속도로 외삽하며 새 애니메이션과 섞는 방식. 외삽이 길면 포즈가 망가지므로 속도에 감쇠를 걸어야 합니다.
- [Motion Matching in O3DE](https://docs.o3de.org/blog/posts/blog-motionmatching/): 다른 엔진의 구현 사례. 탐색을 KD-트리(넓은 탐색)와 비용 계산(좁은 탐색) 2단계로 나눕니다. 과거 궤적도 특징에 넣습니다(예시: 과거 0.7초에 4점, 미래 1.2초에 6점). 디버깅 시각화 조언이 유용합니다.
- [JLPM22 패키지 문서](https://jlpm22.github.io/motionmatching-docs/): 사용법 위주입니다. 권장 특징 구성이나 가중치 수치는 없습니다.

### 부분 확인

- [Spring-It-On: The Game Developer's Spring-Roll-Call](https://theorangeduck.com/page/spring-roll-call): 스프링 수식의 유도. 앞부분만 원문으로 읽었고, 임계 감쇠와 컨트롤러 절은 `spring.h`와 `controller.cpp`의 코드로 확인했습니다.

### 미확인 (제목과 주소만)

- [Fitting Code Driven Displacement](https://theorangeduck.com/page/fitting-code-driven-displacement): 제목으로 보아 시뮬레이션 파라미터를 데이터에 맞추는 방법
- [Tags, Ranges and Masks](https://theorangeduck.com/page/tags-ranges-masks)
- [Simple Two Joint IK](https://theorangeduck.com/page/simple-two-joint)
- [Propagating Velocities through Animation Systems](https://theorangeduck.com/page/propagating-velocities-through-animation-systems)
- [Unrolling Rotations](https://theorangeduck.com/page/unrolling-rotations)
- [Creating Looping Animations from Motion Capture](https://theorangeduck.com/page/creating-looping-animations-motion-capture)
- [Learned Motion Matching (글)](https://theorangeduck.com/page/learned-motion-matching), [논문 PDF](https://www.theorangeduck.com/media/uploads/other_stuff/Learned_Motion_Matching.pdf)
- [전체 글 목록](https://theorangeduck.com/page/all)

## 발표

- Simon Clavet, "Motion Matching and The Road to Next-Gen Animation", GDC 2016: [GDC Vault](https://www.gdcvault.com/play/1022985/Motion-Matching-and-The-Road). 기법을 처음 소개한 발표. 영상은 보지 않았습니다.
- David Bollo, "Inertialization: High-Performance Animation Transitions in Gears of War", GDC 2018: [GDC Vault](https://gdcvault.com/play/1025165/Inertialization-High-Performance-Animation-Transitions). 관성 보간의 출처. 영상은 보지 않았습니다.
- Jose Luis Ponton, "Motion Matching for Character Animation and Virtual Reality Avatars in Unity" (석사 논문): [arXiv](https://arxiv.org/pdf/2310.05215). JLPM22 패키지의 설계 문서. 읽지 않았습니다.

## 데이터

### 이 프로젝트에서 쓰는 팩: Ultimate Traversal Anims 2.0

에셋 스토어 유료 팩입니다. `Assets/Universal_Traversal_Anims`에 있고 저장소에는 올리지 않습니다. 아래는 블렌더 5.0으로 `Art/Animations/New Animations 2.0`의 FBX를 열어 측정한 값입니다. 54개 이동 클립 중 15개만 측정했습니다.

| 항목 | 측정값 | 측정한 클립 |
|---|---|---|
| 프레임률 | FBX 헤더 `TimeMode: 6` (30fps) | 6개 모두 동일 |
| 본 | 52개 + 최상위 `root` 노드. 단위는 cm (스케일 0.01) | 6개 |
| 루트 이동 | 등속 직선. `RunForward`가 0 → 0.797 → 1.590m | 이동 클립 전부 |
| 루프 | 골반의 루트 기준 위치가 첫 프레임과 끝 프레임에서 동일 | `RunForward`, `WalkLeft`, `Idle` |
| 루트 회전 있음 | 0° → -107° | `TurnLeft90`, `RunForwardPivotLeft90` |
| 루트 회전 없음 | 0° 유지, 이동 0.047m | `RunTurn180` |

| 클립 | 길이 | 이동 거리 | 평균 속도 |
|---|---|---|---|
| `WalkForward` | 1.200초 | 1.029m | 0.86 m/s |
| `RunForward`, `RunBack`, `RunLeft` | 0.833초 | 1.590m | 1.91 m/s |
| `RunFasterForward` | 0.833초 | 2.662m | 3.19 m/s |
| `SprintForward` | 0.633초 | 4.744m | 7.49 m/s |
| `RunForwardPivotLeft90` | 1.033초 | 2.135m | 2.07 m/s |

확인하지 못한 것:

- 유니티에서 Generic 리그로 임포트했을 때 `root` Transform이 실제로 움직이는지
- 클립 길이. 블렌더는 `RunForward`를 25구간으로 읽었고 유니티 메타 파일에는 `lastFrame: 24`로 적혀 있음
- 옆걸음·뒷걸음 클립에서 몸이 향하는 방향
- 측정하지 않은 나머지 39개 클립

짧은 루프·출발·정지·전환 클립으로 DB를 구성하는 것은 언리얼의 모션 매칭 문서 예시와 같은 방식입니다: [Motion Matching in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/motion-matching-in-unreal-engine), [Game Animation Sample Project](https://dev.epicgames.com/documentation/en-us/unreal-engine/game-animation-sample-project-in-unreal-engine). 언리얼은 루프 클립에 더하는 `Looping Cost Bias`와 현재 포즈 유지를 유도하는 `Continuing Pose Cost Bias`를 둡니다. 루프 클립에서 미래 궤적을 되감는 동작은 이 문서들에서 확인하지 못했습니다.

### 참고용 공개 데이터 (원본 구현이 쓰는 것)

| 이름 | 형식 | 비고 |
|---|---|---|
| [lafan1-resolved](https://github.com/orangeduck/lafan1-resolved) | BVH, FBX, 스킨 캐릭터 `Geno.fbx` | LAFAN1을 60fps로 다시 풀어낸 것. 발가락·손가락 포함. 비상업용. 유니티 임포트 결과는 확인하지 못함 |
| [LAFAN1 원본](https://github.com/ubisoft/ubisoft-laforge-animation-dataset) | BVH, 30fps | 77개 시퀀스 약 4.6시간. 걷기 12, 달리기 4, 전력질주 2 등. CC BY-NC-ND 4.0 |

원본 데모가 쓰는 클립은 LAFAN1의 `walk1_subject5`, `run1_subject5`, `pushAndStumble1_subject5`(194~351프레임)입니다.

같은 Geno 스켈레톤으로 옮긴 다른 데이터셋(미확인): [100style-retarget](https://github.com/orangeduck/100style-retarget), [zeroeggs-retarget](https://github.com/orangeduck/zeroeggs-retarget), [motorica-retarget](https://github.com/orangeduck/motorica-retarget).

## 유니티 API

- [`TransformStreamHandle`](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Animations.TransformStreamHandle.html), [`AnimationStream`](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Animations.AnimationStream.html): Animator 그래프 안에서 포즈를 쓰려면 `AnimationScriptPlayable`과 `IAnimationJob`을 사용합니다. Transform을 직접 쓰는 방식으로 시작한 뒤, Animator와 섞어야 할 때 옮기면 됩니다.
