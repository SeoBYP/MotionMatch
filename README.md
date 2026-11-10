# MotionMatch

유니티에서 모션 매칭 시스템을 직접 구현하기 위한 프로젝트입니다.

## 문서

| 문서 | 내용 |
|---|---|
| [docs/GUIDE.md](docs/GUIDE.md) | 단계별 구현 가이드. 단계마다 완료 기준이 있습니다 |
| [docs/PORTING.md](docs/PORTING.md) | 원본 C 구현을 유니티 C#으로 옮긴 참고 코드 (스프링, 정규화, 탐색, 관성 보간, 보정, 발 고정) |
| [docs/REFERENCES.md](docs/REFERENCES.md) | 참고한 소스·글·발표·데이터와 각각의 확인 범위 |

## 구현 순서

1. 포즈 DB 굽기
2. DB 재생기
3. 특징 DB
4. 시뮬레이션 오브젝트와 궤적 예측
5. 탐색과 전환
6. 관성 보간
7. 보정과 클램핑
8. 탐색 가속
9. 발 고정

## 기반 자료

- [orangeduck/Motion-Matching](https://github.com/orangeduck/Motion-Matching) (MIT)
- [JLPM22/MotionMatching](https://github.com/JLPM22/MotionMatching) (MIT)
- [Code vs Data Driven Displacement](https://theorangeduck.com/page/code-vs-data-driven-displacement)

## 주의

모션캡처 데이터(LAFAN1 계열)는 비상업용이며 재배포 조건이 있습니다. 데이터 파일과 그것으로 만든 DB는 이 저장소에 커밋하지 않습니다.
