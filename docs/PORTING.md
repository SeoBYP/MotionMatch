# 포팅 레퍼런스 (C → Unity C#)

[orangeduck/Motion-Matching](https://github.com/orangeduck/Motion-Matching)의 핵심 함수를 `UnityEngine` 타입으로 옮긴 참고 코드입니다. 직접 구현할 때 대조용으로 쓰세요.

**검증 상태**

- 스프링 수식(`SimpleSpringDamper`, `CharacterPositionUpdate`)은 파이썬으로 수치 적분과 비교해 소수 5자리까지 일치하는 것을 확인했습니다.
- C# 코드 자체는 **유니티에서 컴파일·실행해 보지 않았습니다.** 원본 C 코드를 줄 단위로 옮긴 것이므로 오타가 있을 수 있습니다. 의심되면 원본 파일을 기준으로 삼으세요.

## 0. 옮길 때 걸리는 차이

| 항목 | orangeduck (C) | Unity |
|---|---|---|
| 쿼터니언 생성자 순서 | `quat(w, x, y, z)` | `new Quaternion(x, y, z, w)` |
| 곱 `quat_mul(q, p)` | q · p (p 먼저 적용) | `q * p` (동일) |
| `quat_mul_inv(q, p)` | q · p⁻¹ | `q * Quaternion.Inverse(p)` |
| `quat_inv_mul(q, p)` | q⁻¹ · p | `Quaternion.Inverse(q) * p` |
| `quat_mul_vec3(q, v)` | 회전 적용 | `q * v` |
| 전방 축 | +Z | +Z (동일) |
| 좌표계 | 오른손 | 왼손. FBX를 유니티가 임포트하면 자동 변환되므로 신경 쓸 필요 없음. BVH를 직접 파싱할 때만 X 반전 필요 |
| 프레임 | 60fps 고정 가정 (`frame_index++`) | 가변. 누적 시간으로 프레임 인덱스 계산 필요 |

## 1. 수학 헬퍼

원본: `common.h`, `quat.h`

```csharp
using UnityEngine;

public static class MMath
{
    public const float Ln2 = 0.69314718056f;

    // exp(-x)의 근사. x >= 0 에서만 사용
    public static float FastNegExp(float x)
        => 1f / (1f + x + 0.48f * x * x + 0.235f * x * x * x);

    public static float HalflifeToDamping(float halflife, float eps = 1e-5f)
        => (4f * Ln2) / (halflife + eps);

    // w가 음수면 부호를 뒤집어 최단 경로 회전으로 만든다
    public static Quaternion Abs(Quaternion q)
        => q.w < 0f ? new Quaternion(-q.x, -q.y, -q.z, -q.w) : q;

    public static Vector3 Log(Quaternion q, float eps = 1e-8f)
    {
        float len = Mathf.Sqrt(q.x * q.x + q.y * q.y + q.z * q.z);
        if (len < eps) return new Vector3(q.x, q.y, q.z);
        float halfAngle = Mathf.Acos(Mathf.Clamp(q.w, -1f, 1f));
        return halfAngle * (new Vector3(q.x, q.y, q.z) / len);
    }

    public static Quaternion Exp(Vector3 v, float eps = 1e-8f)
    {
        float halfAngle = v.magnitude;
        if (halfAngle < eps)
            return Quaternion.Normalize(new Quaternion(v.x, v.y, v.z, 1f));
        float c = Mathf.Cos(halfAngle);
        float s = Mathf.Sin(halfAngle) / halfAngle;
        return new Quaternion(s * v.x, s * v.y, s * v.z, c);
    }

    // 스케일드 앵글 액시스: 방향 = 회전축, 길이 = 회전각(라디안)
    public static Vector3 ToScaledAngleAxis(Quaternion q) => 2f * Log(q);
    public static Quaternion FromScaledAngleAxis(Vector3 v) => Exp(v * 0.5f);

    // 각속도 (rad/s). 데이터 베이킹 때 사용
    public static Vector3 AngularVelocity(Quaternion next, Quaternion curr, float dt)
        => ToScaledAngleAxis(Abs(next * Quaternion.Inverse(curr))) / dt;
}
```

## 2. 스프링

원본: `spring.h`. 이론은 [spring-roll-call](https://theorangeduck.com/page/spring-roll-call).

모두 임계 감쇠 스프링의 정확한 해입니다. `dt`가 커도 안정적이므로 미래 궤적 예측에 `dt = 20/60초`를 그대로 넣을 수 있습니다.

`halflife`는 엄밀한 반감기가 아닙니다. 속도 0에서 시작하면 한 반감기 뒤 남는 오프셋은 약 0.60입니다(검산값 0.5966).

```csharp
public static class Spring
{
    // 목표값 x_goal을 향해 x를 움직인다. v는 호출 사이에 유지해야 하는 상태
    public static void SimpleSpringDamper(
        ref Vector3 x, ref Vector3 v, Vector3 xGoal, float halflife, float dt)
    {
        float y = MMath.HalflifeToDamping(halflife) / 2f;
        Vector3 j0 = x - xGoal;
        Vector3 j1 = v + j0 * y;
        float eydt = MMath.FastNegExp(y * dt);

        x = eydt * (j0 + j1 * dt) + xGoal;
        v = eydt * (v - j1 * y * dt);
    }

    // 회전 버전. v는 각속도 (rad/s)
    public static void SimpleSpringDamper(
        ref Quaternion x, ref Vector3 v, Quaternion xGoal, float halflife, float dt)
    {
        float y = MMath.HalflifeToDamping(halflife) / 2f;
        Vector3 j0 = MMath.ToScaledAngleAxis(MMath.Abs(x * Quaternion.Inverse(xGoal)));
        Vector3 j1 = v + j0 * y;
        float eydt = MMath.FastNegExp(y * dt);

        x = MMath.FromScaledAngleAxis(eydt * (j0 + j1 * dt)) * xGoal;
        v = eydt * (v - j1 * y * dt);
    }

    // 오프셋을 0으로 감쇠. 관성 보간에서 사용
    public static void DecaySpringDamper(
        ref Vector3 x, ref Vector3 v, float halflife, float dt)
    {
        float y = MMath.HalflifeToDamping(halflife) / 2f;
        Vector3 j1 = v + x * y;
        float eydt = MMath.FastNegExp(y * dt);

        x = eydt * (x + j1 * dt);
        v = eydt * (v - j1 * y * dt);
    }

    public static void DecaySpringDamper(
        ref Quaternion x, ref Vector3 v, float halflife, float dt)
    {
        float y = MMath.HalflifeToDamping(halflife) / 2f;
        Vector3 j0 = MMath.ToScaledAngleAxis(x);
        Vector3 j1 = v + j0 * y;
        float eydt = MMath.FastNegExp(y * dt);

        x = MMath.FromScaledAngleAxis(eydt * (j0 + j1 * dt));
        v = eydt * (v - j1 * y * dt);
    }

    // 캐릭터 이동: 속도가 목표 속도를 스프링으로 따라가고, 위치는 그 적분.
    // 원본: controller.cpp 의 simulation_positions_update
    public static void CharacterPositionUpdate(
        ref Vector3 x, ref Vector3 v, ref Vector3 a,
        Vector3 vGoal, float halflife, float dt)
    {
        float y = MMath.HalflifeToDamping(halflife) / 2f;
        Vector3 j0 = v - vGoal;
        Vector3 j1 = a + j0 * y;
        float eydt = MMath.FastNegExp(y * dt);

        x = eydt * ((-j1) / (y * y) + (-j0 - j1 * dt) / y)
            + j1 / (y * y) + j0 / y + vGoal * dt + x;
        v = eydt * (j0 + j1 * dt) + vGoal;
        a = eydt * (a - j1 * y * dt);
    }

    // 차이 g 중 이번 프레임에 줄일 양. 상태 없음
    public static Vector3 DampAdjustment(Vector3 g, float halflife, float dt, float eps = 1e-5f)
        => g * (1f - MMath.FastNegExp((MMath.Ln2 * dt) / (halflife + eps)));

    public static Quaternion DampAdjustment(Quaternion g, float halflife, float dt, float eps = 1e-5f)
        => Quaternion.Slerp(Quaternion.identity, g,
            1f - MMath.FastNegExp((MMath.Ln2 * dt) / (halflife + eps)));
}
```

`DampAdjustment`의 쿼터니언 버전은 원본이 `quat_slerp_shortest_approx`를 쓰지만 `Quaternion.Slerp`로 대체했습니다. `g`에 `MMath.Abs`를 먼저 적용해서 넘기면 최단 경로가 보장됩니다.

## 3. 특징 정규화

원본: `database.h`의 `normalize_feature`

특징 그룹(예: 왼발 위치 3차원) 단위로 호출합니다. 그룹 안의 모든 차원이 **같은 scale**을 공유하는 것이 핵심입니다. 차원별로 따로 나누면 거의 안 변하는 축(예: 발 높이)의 잡음이 증폭됩니다.

```csharp
// features: [nFrames * nFeatures] 평탄화 배열
public static void NormalizeFeature(
    float[] features, float[] featuresOffset, float[] featuresScale,
    int nFrames, int nFeatures, int offset, int size, float weight = 1f)
{
    // 평균
    for (int j = 0; j < size; j++) featuresOffset[offset + j] = 0f;
    for (int i = 0; i < nFrames; i++)
        for (int j = 0; j < size; j++)
            featuresOffset[offset + j] += features[i * nFeatures + offset + j] / nFrames;

    // 차원별 분산
    var vars = new float[size];
    for (int i = 0; i < nFrames; i++)
        for (int j = 0; j < size; j++)
        {
            float d = features[i * nFeatures + offset + j] - featuresOffset[offset + j];
            vars[j] += d * d / nFrames;
        }

    // 그룹의 표준편차 = 차원별 표준편차의 평균
    float std = 0f;
    for (int j = 0; j < size; j++) std += Mathf.Sqrt(vars[j]) / size;
    Debug.Assert(std > 0f, "변화가 없는 특징은 거의 항상 버그");

    // 가중치를 scale에 구워 넣는다: 가중치가 클수록 scale이 작아져 거리에 크게 반영됨
    for (int j = 0; j < size; j++) featuresScale[offset + j] = std / weight;

    for (int i = 0; i < nFrames; i++)
        for (int j = 0; j < size; j++)
        {
            int k = i * nFeatures + offset + j;
            features[k] = (features[k] - featuresOffset[offset + j]) / featuresScale[offset + j];
        }
}
```

가중치를 바꾸면 DB 전체를 다시 정규화해야 합니다. 런타임에 슬라이더로 조절하고 싶다면 JLPM22처럼 `weight = 1`로 정규화해 두고 탐색 시 `diff * diff * weight[j]`를 곱하는 방식이 있습니다. 이 경우 AABB 거리 계산에도 같은 가중치를 곱해야 합니다.

## 4. 탐색

원본: `database.h`의 `motion_matching_search`, `database_build_bounds`

먼저 가속 구조 없이 전수 탐색으로 구현하고, 동작을 확인한 뒤 AABB를 붙이세요. 결과가 같아야 하므로 전수 탐색이 테스트 기준이 됩니다.

```csharp
public const int BoundSm = 16;
public const int BoundLr = 64;

// 16프레임, 64프레임 단위로 특징별 최소·최대를 저장
public static void BuildBounds(
    float[] features, int nFrames, int nf,
    out float[] smMin, out float[] smMax, out float[] lrMin, out float[] lrMax)
{
    int nSm = (nFrames + BoundSm - 1) / BoundSm;
    int nLr = (nFrames + BoundLr - 1) / BoundLr;
    smMin = new float[nSm * nf]; smMax = new float[nSm * nf];
    lrMin = new float[nLr * nf]; lrMax = new float[nLr * nf];
    System.Array.Fill(smMin, float.MaxValue); System.Array.Fill(smMax, float.MinValue);
    System.Array.Fill(lrMin, float.MaxValue); System.Array.Fill(lrMax, float.MinValue);

    for (int i = 0; i < nFrames; i++)
    {
        int iSm = i / BoundSm, iLr = i / BoundLr;
        for (int j = 0; j < nf; j++)
        {
            float f = features[i * nf + j];
            smMin[iSm * nf + j] = Mathf.Min(smMin[iSm * nf + j], f);
            smMax[iSm * nf + j] = Mathf.Max(smMax[iSm * nf + j], f);
            lrMin[iLr * nf + j] = Mathf.Min(lrMin[iLr * nf + j], f);
            lrMax[iLr * nf + j] = Mathf.Max(lrMax[iLr * nf + j], f);
        }
    }
}

// bestIndex: 현재 프레임을 넣고 호출 (클립 끝이면 -1). 더 좋은 프레임이 있으면 갱신된다.
// query는 이미 정규화된 상태여야 한다: (raw - offset) / scale
public static void Search(
    ref int bestIndex, ref float bestCost,
    int[] rangeStarts, int[] rangeStops,
    float[] features, int nf,
    float[] smMin, float[] smMax, float[] lrMin, float[] lrMax,
    float[] query,
    float transitionCost = 0f, int ignoreRangeEnd = 20, int ignoreSurrounding = 20)
{
    int currIndex = bestIndex;

    // 현재 프레임의 비용이 기준선. 전환 비용을 더하지 않으므로 현재 프레임이 유리하다
    if (bestIndex != -1)
    {
        bestCost = 0f;
        for (int j = 0; j < nf; j++)
        {
            float d = query[j] - features[bestIndex * nf + j];
            bestCost += d * d;
        }
    }

    for (int r = 0; r < rangeStarts.Length; r++)
    {
        int i = rangeStarts[r];
        int rangeEnd = rangeStops[r] - ignoreRangeEnd;   // 클립 끝은 미래 궤적이 없으므로 제외

        while (i < rangeEnd)
        {
            int iLr = i / BoundLr;
            int iLrNext = (iLr + 1) * BoundLr;

            // 큰 박스까지의 거리. 박스 안 어떤 프레임도 이보다 가까울 수 없다
            float cost = transitionCost;
            for (int j = 0; j < nf; j++)
            {
                float d = query[j] - Mathf.Clamp(query[j], lrMin[iLr * nf + j], lrMax[iLr * nf + j]);
                cost += d * d;
                if (cost >= bestCost) break;
            }
            if (cost >= bestCost) { i = iLrNext; continue; }

            while (i < iLrNext && i < rangeEnd)
            {
                int iSm = i / BoundSm;
                int iSmNext = (iSm + 1) * BoundSm;

                cost = transitionCost;
                for (int j = 0; j < nf; j++)
                {
                    float d = query[j] - Mathf.Clamp(query[j], smMin[iSm * nf + j], smMax[iSm * nf + j]);
                    cost += d * d;
                    if (cost >= bestCost) break;
                }
                if (cost >= bestCost) { i = iSmNext; continue; }

                while (i < iSmNext && i < rangeEnd)
                {
                    // 현재 프레임 주변은 건너뛴다 (같은 자리로의 무의미한 전환 방지)
                    if (currIndex != -1 && Mathf.Abs(i - currIndex) < ignoreSurrounding) { i++; continue; }

                    cost = transitionCost;
                    for (int j = 0; j < nf; j++)
                    {
                        float d = query[j] - features[i * nf + j];
                        cost += d * d;
                        if (cost >= bestCost) break;
                    }
                    if (cost < bestCost) { bestIndex = i; bestCost = cost; }
                    i++;
                }
            }
        }
    }
}
```

전수 탐색 버전은 위에서 두 개의 박스 검사 블록만 지우면 됩니다.

Burst로 옮길 때는 `float[]`를 `NativeArray<float>`로 바꾸고 `[BurstCompile] struct : IJob`으로 감쌉니다. 구조는 JLPM22의 `Runtime/Core/Burst/BVHMotionMatchingSearch.cs`를 참고하세요.

## 5. 관성 보간 (Inertialization)

원본: `spring.h`의 `inertialize_*`, `controller.cpp`의 `inertialize_pose_*`

원리: 전환 순간 "이전 포즈 − 새 포즈"를 오프셋으로 기록하고, 새 애니메이션에 그 오프셋을 더해 출력합니다. 오프셋은 매 프레임 스프링으로 0까지 감쇠시킵니다. 크로스페이드와 달리 애니메이션을 하나만 평가합니다.

오프셋이 `ref`인 이유는 감쇠가 끝나기 전에 다음 전환이 일어나면 남은 오프셋 위에 누적해야 하기 때문입니다.

```csharp
public static class Inertializer
{
    public static void Transition(
        ref Vector3 offX, ref Vector3 offV,
        Vector3 srcX, Vector3 srcV, Vector3 dstX, Vector3 dstV)
    {
        offX = (srcX + offX) - dstX;
        offV = (srcV + offV) - dstV;
    }

    public static void Update(
        out Vector3 outX, out Vector3 outV,
        ref Vector3 offX, ref Vector3 offV,
        Vector3 inX, Vector3 inV, float halflife, float dt)
    {
        Spring.DecaySpringDamper(ref offX, ref offV, halflife, dt);
        outX = inX + offX;
        outV = inV + offV;
    }

    public static void Transition(
        ref Quaternion offX, ref Vector3 offV,
        Quaternion srcX, Vector3 srcV, Quaternion dstX, Vector3 dstV)
    {
        offX = MMath.Abs((offX * srcX) * Quaternion.Inverse(dstX));
        offV = (offV + srcV) - dstV;
    }

    public static void Update(
        out Quaternion outX, out Vector3 outV,
        ref Quaternion offX, ref Vector3 offV,
        Quaternion inX, Vector3 inV, float halflife, float dt)
    {
        Spring.DecaySpringDamper(ref offX, ref offV, halflife, dt);
        outX = offX * inX;
        outV = offV + offX * inV;
    }
}
```

`src`는 전환 직전까지 재생하던 DB 프레임의 **원본 값**(오프셋이 더해지기 전)입니다. 오프셋이 이미 포함된 출력 포즈를 넣으면 오프셋이 두 번 더해집니다.

### 루트 처리

루트(시뮬레이션 본)를 뺀 나머지 본은 부모 기준 로컬 값이라 위 함수를 그대로 쓰면 됩니다. 루트는 DB 안의 위치가 클립마다 제각각이라 "DB 공간 → 월드 공간" 변환이 필요합니다.

**단순한 방법 (JLPM22 방식, 먼저 이걸로 시작 권장)**: 루트는 관성 보간하지 않습니다. 전환할 때 기준점 두 쌍을 기록하고, 이후 DB 루트를 그 기준으로 변환합니다.

```csharp
// 전환(또는 시작) 시점에 호출
void UpdateAnimationSpaceOrigin(int frame)
{
    animOriginPos = db.Position(frame, 0);                    // DB 안에서의 루트 위치
    invAnimOriginRot = Quaternion.Inverse(db.Rotation(frame, 0));
    worldOriginPos = root.position;                           // 그 순간 월드에서의 루트
    worldOriginRot = root.rotation;
}

// 매 프레임
void ApplyRoot(int frame)
{
    Vector3 localPos = invAnimOriginRot * (db.Position(frame, 0) - animOriginPos);
    Quaternion localRot = invAnimOriginRot * db.Rotation(frame, 0);
    root.SetPositionAndRotation(
        worldOriginRot * localPos + worldOriginPos,
        worldOriginRot * localRot);
}
```

보정(7절)으로 루트를 옮길 때는 `root.position`을 직접 바꾸지 말고 `worldOriginPos += delta`, `worldOriginRot = deltaRot * worldOriginRot`로 기준점을 옮깁니다. 직접 바꾸면 다음 프레임 `ApplyRoot`가 덮어씁니다.

**원본 방식**: 루트의 속도·각속도도 관성 보간합니다. `controller.cpp`의 `inertialize_pose_transition`, `inertialize_pose_update`, `inertialize_root_adjust` 세 함수가 한 세트입니다. 전환 시 루트 속도가 튀는 것이 눈에 띌 때 옮기세요.

## 6. 쿼리 벡터와 궤적 예측

원본: `controller.cpp`의 `query_compute_*`, `trajectory_*_predict`

```csharp
// 궤적 위치 특징: 미래 3개 지점을 현재 루트 기준 로컬 XZ로
static void QueryTrajectoryPosition(
    float[] query, ref int offset,
    Vector3 rootPos, Quaternion rootRot, Vector3[] trajPos /* [0]=현재, [1..3]=미래 */)
{
    Quaternion inv = Quaternion.Inverse(rootRot);
    for (int i = 1; i <= 3; i++)
    {
        Vector3 t = inv * (trajPos[i] - rootPos);
        query[offset++] = t.x;
        query[offset++] = t.z;
    }
}

// 궤적 방향 특징: 미래 3개 지점의 전방 벡터를 로컬 XZ로
static void QueryTrajectoryDirection(
    float[] query, ref int offset, Quaternion rootRot, Quaternion[] trajRot)
{
    Quaternion inv = Quaternion.Inverse(rootRot);
    for (int i = 1; i <= 3; i++)
    {
        Vector3 d = inv * (trajRot[i] * Vector3.forward);
        query[offset++] = d.x;
        query[offset++] = d.z;
    }
}
```

`rootPos`, `rootRot`은 **캐릭터 루트**(애니메이션이 움직이는 쪽)이고, `trajPos`, `trajRot`은 **시뮬레이션 오브젝트**에서 예측한 값입니다. 둘이 벌어져 있으면 쿼리가 "시뮬레이션을 따라잡는 방향"을 가리키게 되어 자연스럽게 수렴합니다.

궤적 예측:

```csharp
const float PredDt = 20f / 60f;   // 20프레임 간격

// 위치: 이전 예측값에서 이어서 적분
trajPos[0] = simPos; trajVel[0] = simVel; trajAcc[0] = simAcc;
for (int i = 1; i < 4; i++)
{
    trajPos[i] = trajPos[i - 1]; trajVel[i] = trajVel[i - 1]; trajAcc[i] = trajAcc[i - 1];
    Spring.CharacterPositionUpdate(ref trajPos[i], ref trajVel[i], ref trajAcc[i],
        desiredVelocity, velocityHalflife, PredDt);
}

// 회전: 현재 상태에서 i * PredDt 만큼 한 번에 적분 (정확한 해라서 가능)
for (int i = 1; i < 4; i++)
{
    trajRot[i] = simRot; trajAngVel[i] = simAngVel;
    Spring.SimpleSpringDamper(ref trajRot[i], ref trajAngVel[i],
        desiredRotation, rotationHalflife, i * PredDt);
}
```

원본은 미래 시점마다 카메라 회전까지 예측해 `desiredVelocity`를 다시 계산합니다(`trajectory_desired_velocities_predict`). 스트레이프가 없다면 위처럼 현재 값을 그대로 써도 됩니다.

쿼리의 포즈 부분(앞 15차원)은 계산하지 않습니다. 현재 프레임의 DB 특징을 역정규화해서 복사합니다.

```csharp
for (int j = 0; j < 15; j++)
    query[j] = features[frame * nf + j] * featuresScale[j] + featuresOffset[j];
```

이후 궤적 12차원을 채우고, 전체를 `(query[j] - offset[j]) / scale[j]`로 정규화해 탐색에 넘깁니다.

## 7. 보정과 클램핑

원본: `controller.cpp`의 `adjust_character_*`, `clamp_character_*`

```csharp
// 캐릭터를 시뮬레이션 쪽으로 당기되, 캐릭터가 움직이는 속도의 일정 비율까지만
public static Vector3 AdjustPositionByVelocity(
    Vector3 charPos, Vector3 charVel, Vector3 simPos,
    float maxRatio, float halflife, float dt)
{
    Vector3 adj = Spring.DampAdjustment(simPos - charPos, halflife, dt);
    float maxLen = maxRatio * charVel.magnitude * dt;
    if (adj.magnitude > maxLen) adj = maxLen * adj.normalized;
    return charPos + adj;
}

public static Quaternion AdjustRotationByVelocity(
    Quaternion charRot, Vector3 charAngVel, Quaternion simRot,
    float maxRatio, float halflife, float dt)
{
    Quaternion diff = MMath.Abs(Quaternion.Normalize(simRot * Quaternion.Inverse(charRot)));
    Quaternion adj = Spring.DampAdjustment(diff, halflife, dt);

    float maxLen = maxRatio * charAngVel.magnitude * dt;
    Vector3 saa = MMath.ToScaledAngleAxis(adj);
    if (saa.magnitude > maxLen) adj = MMath.FromScaledAngleAxis(maxLen * saa.normalized);

    return adj * charRot;
}

public static Vector3 ClampPosition(Vector3 charPos, Vector3 simPos, float maxDistance)
{
    Vector3 d = charPos - simPos;
    return d.magnitude > maxDistance ? simPos + maxDistance * d.normalized : charPos;
}

public static Quaternion ClampRotation(Quaternion charRot, Quaternion simRot, float maxAngle)
{
    Quaternion diff = MMath.Abs(charRot * Quaternion.Inverse(simRot));
    float angle = 2f * Mathf.Acos(Mathf.Clamp(diff.w, -1f, 1f));
    if (angle <= maxAngle) return charRot;

    Vector3 axis = new Vector3(diff.x, diff.y, diff.z).normalized;
    return Quaternion.AngleAxis(maxAngle * Mathf.Rad2Deg, axis) * simRot;
}
```

원본 기본값: 위치 반감기 0.1초, 회전 반감기 0.2초, `maxRatio` 0.5, 최대 거리 0.15m, 최대 각도 90°.

`maxRatio` 제한의 효과: 캐릭터 속도가 0이면 보정량도 0입니다. 서 있는 캐릭터가 미끄러지지 않는 이유가 이것입니다.

## 8. 발 고정

원본: `controller.cpp`의 `contact_update`, `ik_two_bone`, `ik_look_at`

```csharp
public struct FootContact
{
    public bool State;          // 직전 프레임의 접촉 여부
    public bool Lock;           // 현재 고정 중인지
    public Vector3 Position;    // 출력: IK가 목표로 삼을 발끝 위치
    public Vector3 Velocity;
    public Vector3 Point;       // 고정된 지점
    public Vector3 Target;      // 직전 프레임의 입력 위치 (속도 계산용)
    public Vector3 OffsetPos, OffsetVel;
}

public static void ContactUpdate(
    ref FootContact c, Vector3 inputPos, bool inputContact,
    float unlockRadius, float footHeight, float halflife, float dt)
{
    Vector3 inputVel = (inputPos - c.Target) / (dt + 1e-8f);
    c.Target = inputPos;

    // 고정 중이면 고정점을, 아니면 애니메이션 값을 따라간다
    Inertializer.Update(out c.Position, out c.Velocity, ref c.OffsetPos, ref c.OffsetVel,
        c.Lock ? c.Point : inputPos,
        c.Lock ? Vector3.zero : inputVel,
        halflife, dt);

    bool unlock = c.Lock && (c.Point - inputPos).magnitude > unlockRadius;

    if (!c.State && inputContact)
    {
        // 접촉 시작: 현재 발 위치를 바닥 높이로 내려 고정점으로 삼는다
        c.Lock = true;
        c.Point = c.Position;
        c.Point.y = footHeight;
        Inertializer.Transition(ref c.OffsetPos, ref c.OffsetVel,
            inputPos, inputVel, c.Point, Vector3.zero);
    }
    else if ((c.Lock && c.State && !inputContact) || unlock)
    {
        // 접촉 종료 또는 애니메이션과 너무 벌어짐: 고정 해제
        c.Lock = false;
        Inertializer.Transition(ref c.OffsetPos, ref c.OffsetVel,
            c.Point, Vector3.zero, inputPos, inputVel);
    }

    c.State = inputContact;
}
```

원본 기본값: `unlockRadius` 0.2m, `footHeight` 0.02m, 반감기 0.1초.

`c.Position`을 발끝 목표로 두 관절 IK(엉덩이–무릎–발목)를 풉니다. 해법은 [Simple Two Joint IK](https://theorangeduck.com/page/simple-two-joint)와 원본 `ik_two_bone`을 참고하세요. 유니티 구현 예시는 JLPM22의 `Runtime/IK/TwoJointIK.cs`에 있습니다.

저자는 이 발 고정을 "토이 수준"이라고 평가합니다. 다른 방법(데이터 보강, 보정 파라미터 조정)을 먼저 시도하고 마지막에 붙이는 것을 권합니다.
