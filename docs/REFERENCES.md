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

에셋 스토어 유료 팩입니다. `Assets/Universal_Traversal_Anims`에 있고 저장소에는 올리지 않습니다. 아래는 유니티 6000.6.7f1에서 `Art/Animations/New Animations 2.0`의 FBX 79개를 모두 Generic 리그로 임포트하고 `AnimationClip.SampleAnimation`으로 프레임마다 `root`를 읽은 값입니다. 팩 원본의 설정은 바꾸지 않고 복사본으로 측정했습니다.

이전에 블렌더 5.0으로 잰 값은 클립 길이가 1구간씩 길었고(예: `RunForward` 25구간), 그래서 속도가 약 4% 낮았으며 회전도 -107°로 잘못 읽혔습니다. 아래 값이 맞는 값입니다.

**공통**

| 항목 | 측정값 |
|---|---|
| 프레임률 | 79개 모두 30fps |
| 계층 | Transform 55개. 모델 루트 아래에 `root`가 자식으로 있고 그 아래 `pelvis`. 단위는 cm (`fileScale` 0.01) |
| `root` 이동 | 샘플링하면 실제로 이동. Root node 설정과 Preserve Hierarchy에 무관 |
| 클립 길이 | 메타 파일의 `lastFrame`과 같음. N구간이면 샘플은 0~N번, N+1개 |
| 팩 원본 리그 | Humanoid. 구울 클립은 Generic으로 바꿔야 함 |

**루프 클립** (첫·끝 프레임에서 골반의 `root` 기준 위치 차이 0.0001m 이하, `root` 회전은 처음부터 끝까지 0°)

| 클립 | 길이 | 이동 거리 | 평균 속도 | 구간 속도 범위 |
|---|---|---|---|---|
| `Idle` | 7.000초 (210구간) | 0m | 0 | |
| `WalkForward`, `WalkLeft`, `WalkRight`, `WalkForwardLeft`, `WalkForwardRight` | 1.167초 (35구간) | 1.029m | 0.88 m/s | 0.88~0.88 |
| `WalkBack`, `WalkBackLeft`, `WalkBackRight` | 1.167초 (35구간) | 1.139m | 0.98 m/s | 0.79~1.17 |
| `WalkForwardAlt01` / `WalkForwardAlt02` | 1.167초 (35구간) | 1.529m / 1.320m | 1.31 / 1.13 m/s | 1.12~1.70 / 0.92~1.40 |
| `Run` 8방향 | 0.800초 (24구간) | 1.590m | 1.99 m/s | 1.82~2.12 |
| `RunFaster` 8방향 | 0.800초 (24구간) | 2.662m | 3.33 m/s | 2.99~3.64 |
| `SprintForward` | 0.600초 (18구간) | 4.744m | 7.91 m/s | 7.51~8.24 |
| `Movement_Sprint_LeftFootStart` | 0.600초 (18구간) | 4.080m | 6.80 m/s | 6.51~6.99 |

- 걷기는 후진 계열(0.98 m/s)이 전진·측면(0.88 m/s)보다 빠릅니다. 달리기와 빠른 달리기는 8방향이 같습니다.
- 이동 방향은 직선이고 대각선은 정확히 45°입니다(예: `RunForwardLeft`가 `root` 기준 (-1.124, 1.124)m). 직선에서 벗어나는 양은 최대 0.04m입니다. 속도는 걷기 전진·측면만 완전한 등속이고 나머지는 한 걸음 안에서 ±7~10% 흔들립니다.
- `RunFasterRight`만 첫·끝 골반 위치가 0.017m 어긋납니다.
- 걷기 8방향은 속도가 두 가지(전진·측면 5개, 후진 계열 3개)로 나뉩니다.

**몸이 향하는 방향** (양쪽 허벅지 본을 잇는 선에서 구한 골반 방향. `root` 전방 기준, 클립 평균)

| 이동 방향 | 걷기 | 달리기 | 빠른 달리기 |
|---|---|---|---|
| 전진 | 0.5° | -1.9° | 1.7° |
| 전진+왼쪽 / 전진+오른쪽 | -44.5° / 45.5° | -46.9° / 43.1° | -43.3° / 46.7° |
| 왼쪽 / 오른쪽 | -65.3° / 65.5° | -66.9° / 63.1° | -63.3° / 66.7° |
| 후진+왼쪽 / 후진+오른쪽 | 49.3° / -40.7° | 42.7° / -47.3° | 46.2° / -43.8° |
| 후진 | 4.3° | -2.3° | 1.2° |

`root`는 8방향 모두 같은 쪽(+Z)을 보고, 골반만 이동 방향 쪽으로 최대 약 65° 틀어집니다. 후진 계열은 뒷걸음입니다. 즉 이 8방향 세트는 "보는 방향을 고정하고 움직이는" 스트레이프용입니다. `Idle`은 같은 방법으로 재면 19.0°가 나오므로(선 자세에서 골반이 약간 돌아가 있음) 위 값에는 수 도 정도의 기준 오차가 있습니다.

**전환 클립**

| 클립 | 길이 | 이동(`root` 기준 x, z) | 평균 속도 | `root` 회전 변화 |
|---|---|---|---|---|
| `RunForwardStart` | 0.833초 (25구간) | (0, 1.370)m | 1.64 m/s (0.13~2.99) | 0° |
| `RunForwardStop` | 2.167초 (65구간) | (0, 1.951)m | 0.90 m/s (0.00~2.76) | 0° |
| `RunForwardPivotLeft90` / `Right90` | 1.000초 / 0.667초 | (-1.847, 1.071) / (0.704, 0.974)m | 2.14 / 1.80 m/s | -90° / +90° |
| `RunForwardAvoidancePivotLeft90` / `Right90` | 1.167초 / 0.600초 | (-1.823, 2.010) / (0.695, 1.020)m | 2.33 / 2.06 m/s | -90° / +90° |
| `WalkForwardStart` / `WalkForwardStop` | 1.600초 / 1.467초 | (0, 0.983) / (0, 0.196)m | 0.61 / 0.13 m/s | 0° |
| `WalkForwardStopInPlace` | 1.467초 | 0m | 0 | 0° |
| `WalkForwardStartAndStop` | 4.233초 | (0, 2.273)m | 0.54 m/s | 0° |
| `WalkForwardPivotLeft90` / `Right90` | 2.467초 / 1.200초 | (-1.436, 1.337) / (0.597, 0.540)m | 0.80 / 0.67 m/s | -90° / +90° |
| `WalkBackStart` / `WalkBackStop` | 2.033초 / 2.933초 | (0, -1.344) / (0, -0.845)m | 0.66 / 0.29 m/s | 0° |
| `WalkBackStopInPlace` / `WalkBackStartAndStop` | 2.067초 / 4.967초 | 0 / (0, -2.196)m | 0 / 0.44 m/s | 0° |
| `TurnLeft90` / `TurnRight90` | 2.667초 (80구간) | 0m | 0 | -90° / +90° |
| `TurnLeft180` / `TurnRight180` | 2.667초 (80구간) | 0m | 0 | 180° |
| `RunTurn180` | 1.167초 | (0, -0.047)m | 0.04 m/s | 0° (골반은 0° → -175°) |
| `RunTurn180FastSlide` | 1.500초 | (0, 1.102)m | 구간 속도 최대 107 m/s | 0° |

- 좌우 피벗은 길이와 이동 거리가 달라 서로 거울상이 아닙니다.
- `RunTurn180`은 몸이 도는 동안 `root`가 회전하지 않고, `RunTurn180FastSlide`는 `root`가 한 프레임에 3.6m 튑니다. 둘 다 그대로는 DB에 넣을 수 없습니다.
- 출발·정지 클립은 전진과 후진만 있습니다. 측면·대각선 출발·정지는 없습니다.

**그 밖의 클립** (이동용이 아니라서 수치는 생략): `AimSpace_*` 9개는 1구간짜리 포즈, `DuckUnderPass*` 6개, `StepUpToPlatform100cm*` 4개, `JumpDown`, `JumpDownHeroLanding`, `JumpRunForward`, `RunForwardJumpHurdle106cm`, `RunForwardTicTacLeftWall`/`RightWall`, `RunForwardShoveAway`, `DoorOpenWalkThrough`, `Climb_Leap_Behind_Down_Alternate_Height`.

확인하지 못한 것:

- 발 접촉을 판정할 본 이름과 임계값(1단계에서 정할 값)
- 에디터를 직접 열어 눈으로 본 재생 결과. 위 값은 모두 배치 모드에서 숫자로 읽은 것입니다

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
