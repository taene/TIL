## 학습 키워드

* Unreal Engine 5
* Niagara System (UNiagaraComponent)
* VFX Lifecycle Management
* `Deactivate` vs `DeactivateImmediate`

## 학습 내용

### 배운 개념 요약

투사체(Projectile)가 목표물에 충돌하거나 소멸할 때, 꼬리(Trail) 효과나 루핑(Looping) 이펙트가 즉시 사라지지 않고 허공에 남는 현상을 해결하기 위해 나이아가라 컴포넌트의 종료 함수 차이를 명확히 학습했다.

핵심은 "새로운 파티클 생성을 멈추고 기존 파티클이 자연스럽게 죽기를 기다릴 것인가(Deactivate)" 아니면 "모든 파티클을 강제로 즉시 제거할 것인가(DeactivateImmediate)"의 차이이다.

| 기능 | 함수명 | 동작 방식 | 주요 사용 사례 |
| --- | --- | --- | --- |
| **비활성화** | `Deactivate()` | Emitter가 새로운 파티클 생성을 멈춘다. <br> **이미 생성된 파티클은 수명(Lifetime)이 다할 때까지 남아있다.** | 횃불 끄기, 마법 영창 중단 (연기가 서서히 사라짐) |
| **즉시 종료** | `DeactivateImmediate()` | Emitter를 멈추고, **활성화된 모든 파티클 데이터를 즉시 메모리에서 날려버린다(Kill).** | 투사체 충돌 후 소멸, 텔레포트, 오브젝트 풀링 반환 시 |

### 구현 과정

#### 1. 문제 상황 분석

GAS와 오브젝트 풀링을 활용해 투사체를 발사하는 로직을 구현하던 중, 투사체가 벽에 부딪혀 `Destroy()` 되거나 오브젝트 풀(Object Pool)로 반환될 때 시각적 결함이 발생했다.

* **현상:** 투사체 액터는 사라졌는데, 투사체 뒤를 따라오던 나이아가라 트레일(Trail)이 충돌 지점에 멈춘 채 잠시 동안 남아있다가 사라짐.
* **원인:** `NiagaraComponent->Deactivate()`를 호출했기 때문. 이는 수도꼭지를 잠그는 것과 같아서, 파이프에 남아있던 물(이미 생성된 파티클)은 다 흘러나올 때까지 유지된다.

#### 2. 해결 방안 적용

투사체가 '충돌'이라는 물리적 사건에 의해 더 이상 존재하지 않게 된다면, 그에 종속된 시각 효과 또한 물리적으로 즉시 소멸해야 한다. 따라서 잔존 파티클을 허용하지 않는 `DeactivateImmediate()`로 로직을 변경했다.

무기 발사체나 임팩트 효과가 예상치 못하게 중단되어야 할 때(예: 게임플레이 태그에 의한 취소 등) 시각적 정합성을 위해 이러한 즉시 종료 처리를 엄격하게 관리한다.

#### 3. 코드 구현 (C++)

다음은 투사체 액터가 충돌(OnHit) 시 호출되는 로직의 개선 전후 코드이다.

```cpp
void AMyProjectileActor::OnImpact(const FHitResult& HitResult)
{
    // ... 충돌 로직 처리 (데미지 적용 등) ...

    if (ProjectileLoopEffectComponent)
    {
        // [Before]
        // 단순히 비활성화만 하면, 이미 방출된 트레일 파티클이 허공에 남는다.
        // ProjectileLoopEffectComponent->Deactivate();

        // [After]
        // 투사체가 파괴되거나 풀로 돌아가므로, 모든 파티클을 즉시 정리한다.
        ProjectileLoopEffectComponent->DeactivateImmediate();
    }

    // 투사체 액터 파괴 또는 풀 반환
    Destroy();
}

```

## 느낀점

단순히 "이펙트를 끈다"라는 행위에도 "자연스러운 소멸"과 "즉각적인 데이터 클리어"라는 두 가지 층위가 있음을 깨달았다. 특히 **오브젝트 풀링(Object Pooling)** 환경에서 이 차이는 치명적이다. 만약 `Deactivate()`만 하고 풀에 반환하면, 다음번에 이 투사체를 꺼내 쓸 때 이전 위치에서 사라지지 않은 파티클이 튀어나오는 버그(Ghost Effect)를 겪게 될 것이다.

이러한 이펙트의 생명주기(Lifecycle)를 게임플레이 로직(Gameplay Ability System)의 상태와 완벽하게 동기화하면 게임의 품질을 높일 수 있다. 앞으로는 `Deactivate`를 사용할 때, **"이 이펙트가 잔상을 남겨야 하는가? 남기면 논리적으로 어색한가?"** 를 항상 먼저 자문하고 코드를 작성해야겠다.

---

### 요약: 나이아가라 종료 함수 선택 가이드

1. **자연스러운 연출이 필요할 때**  `Deactivate()`
* *예: 캠프파이어 불 끄기, 엔진 시동 끄기*


2. **논리적 단절/삭제가 필요할 때**  `DeactivateImmediate()`
* *예: 투사체 충돌, 캐릭터 사망 후 즉시 시체 소멸, 맵 이동*
