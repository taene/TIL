## 학습 키워드

* Unreal Engine 5
* Gameplay Ability System (GAS)
* Ability Lifecycle & EndAbility
* World Timer vs Ability Task
* Actor Spawning for Persistent Effects

## 학습 내용

### 배운 개념 요약

GAS(Gameplay Ability System)에서 `UGameplayAbility`는 기본적으로 **실행(Activation)부터 종료(End)까지의 생명주기**를 가진다. 어빌리티가 `K2_EndAbility` 혹은 `EndAbility`를 호출하여 종료되면, 해당 어빌리티 인스턴스에 종속된 모든 Ability Task와 Timer는 즉시 정리(Cleanup)된다.

따라서 몽타주 재생과 같이 어빌리티의 수명과 묶여있는 로직은 Task로 처리하는 것이 적합하지만, 장판형 스킬(AoE)이나 설치물처럼 **어빌리티 시전은 끝났으나 월드에 남아 지속적인 영향을 주어야 하는 로직**은 어빌리티 내부의 타이머나 Task로 관리해서는 안 된다.

라이라(Lyra) 프로젝트에서도 수류탄 투척이나 설치형 아이템을 사용할 때, 어빌리티는 오직 **투사체(Projectile)나 액터(Actor)를 스폰하는 역할**만 수행하고 즉시 종료된다. 이후의 물리 연산, 충돌, 지속 효과(GameplayEffect 적용) 등은 스폰된 액터(`ALyraProjectile` 등)가 독자적인 수명을 가지고 처리한다. 이는 **Fire-and-Forget** 패턴으로, 어빌리티와 이펙트의 결합도를 낮추고 로직의 안전성을 보장하는 핵심 설계이다.

### 구현 과정

#### 1. 문제 상황 분석

Advent 스킬 구현 중, 일정 시간 동안 바닥에 지속 피해를 주는 장판을 생성해야 했다. 초기에는 `AbilityTask`나 `GetWorld()->GetTimerManager()`를 사용해 어빌리티 내부에서 틱마다 데미지를 주도록 구현했다.

그러나 스킬 모션(몽타주)이 끝나면 `EndAbility`가 호출되도록 설계되어 있었고, 이때 어빌리티가 소멸하면서 등록된 타이머와 태스크가 강제로 취소되는 문제가 발생했다. 결과적으로 장판의 지속 시간(Duration)이 남았음에도 불구하고 데미지 판정이 중단되었다.

#### 2. 해결 방안: 책임의 분리 (Decoupling)

어빌리티가 끝나면서 타이머도 같이 끝나는 것의 이유를 찾기 위해 어빌리티에 중단점을 걸어 디버깅을 한 결과, GAS의 EndAbility 코드 내에 해당 어빌리티에 존재하는 모든 타이머를 정리하는 코드를 발견했다.
따라서 지속 피해 로직의 주체를 어빌리티에서 별도의 액터(AOE Field Actor)로 이관하였다.

1. **AOE Actor 클래스 생성:** `AActor`를 상속받는 `AAdventFieldActor`를 생성하고, 이 액터가 생성될 때(BeginPlay) 스스로 타이머를 돌며 `GameplayEffect`를 주변 적에게 적용하도록 변경한다.
2. **Ability의 역할 축소:** 어빌리티는 이제 데미지를 직접 주는 것이 아니라, `AAdventFieldActor`를 알맞은 위치에 `SpawnActor` 하는 역할만 수행한다.

#### 3. 코드 구현 (C++)

어빌리티에서 액터를 스폰하고, 이후 로직은 액터에게 위임하는 방식이다.

```cpp
// AAdventFieldActor.h
// 지속 피해를 담당하는 별도의 액터
UCLASS()
class ADVENT_API AAdventFieldActor : public AActor
{
    GENERATED_BODY()

public:
    AAdventFieldActor();

protected:
    virtual void BeginPlay() override;

private:
    void ApplyDamageLoop();

    UPROPERTY(EditDefaultsOnly, Category = "GAS")
    TSubclassOf<UGameplayEffect> DamageEffectClass;

    FTimerHandle TimerHandle_Damage;
};

// AAdventFieldActor.cpp
void AAdventFieldActor::BeginPlay()
{
    Super::BeginPlay();

    // 액터가 생성되면 자신의 수명주기에 맞춰 타이머를 실행한다.
    // 어빌리티가 종료되어도 이 액터는 월드에 존재하므로 타이머는 유지된다.
    GetWorld()->GetTimerManager().SetTimer(
        TimerHandle_Damage,
        this,
        &AAdventFieldActor::ApplyDamageLoop,
        1.0f,
        true
    );
    
    // 예시: 5초 뒤 액터 자체 소멸
    SetLifeSpan(5.0f);
}

void AAdventFieldActor::ApplyDamageLoop()
{
    // Overlap된 타겟들에게 GameplayEffect를 적용하는 로직 수행
    // ...
}

```

```cpp
// UAdventGameplayAbility.cpp
// 어빌리티는 단순히 액터를 배치하는 역할만 수행 (Fire and Forget)

void UAdventGameplayAbility::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
    if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
    {
        EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
        return;
    }

    // 장판 액터 스폰
    if (HasAuthority(&ActivationInfo))
    {
        FTransform SpawnTransform = GetTargetTransform(); // 타겟 위치 계산 함수 가정
        
        AAdventFieldActor* SpawnedField = GetWorld()->SpawnActorDeferred<AAdventFieldActor>(
            FieldActorClass, 
            SpawnTransform, 
            GetOwningActorFromActorInfo(), 
            nullptr, 
            ESpawnActorCollisionHandlingMethod::AlwaysSpawn
        );

        if (SpawnedField)
        {
            // 필요한 데이터 전달 (예: Instigator, Effect Spec 등)
            SpawnedField->FinishSpawning(SpawnTransform);
        }
    }

    // 몽타주 재생 등 시전 동작 수행
    // ...

    // 시전 동작이 끝나면 EndAbility 호출 -> 어빌리티는 끝나도 SpawnedField는 월드에 남아 제 할 일을 함.
}

```

## 느낀점

이번 트러블 슈팅을 통해 GAS 어빌리티의 생명주기(Lifecycle)가 로직 설계에 얼마나 큰 영향을 미치는지 깨닫게 되었다. 단순히 "기능이 작동하는가?"를 넘어, "이 기능이 언제 시작되고 언제 소멸하는가?"를 고려해야 한다.

특히 전 GG 프로젝트 때 학습한 Lyra 프로젝트에서 왜 복잡한 로직을 어빌리티 내부에 다 때려 넣지 않고, `GameplayEffect`, `Cue`, `Actor` 등으로 잘게 쪼개어 관리하는지 이해할 수 있었다. "어빌리티는 이벤트를 시작하는 트리거일 뿐, 월드에 남는 현상은 독립적인 객체가 되어야 한다"는 데이터 중심, 객체 지향적 설계 원칙을 내 프로젝트에도 확실히 적용할 수 있게 되었다.

---

### 요약

| 구분 | 변경 전 (Ability Internal) | 변경 후 (Actor Spawning) |
| --- | --- | --- |
| **구현 방식** | Ability Task / World Timer 사용 | 별도의 `AActor` 생성 및 위임 |
| **타이머 주체** | `UGameplayAbility` 인스턴스 | 생성된 `AActor` |
| **문제점** | `EndAbility` 호출 시 타이머 강제 종료 | 어빌리티 종료와 무관하게 동작 |
| **Lyra 철학** | 로직의 강한 결합 (지양) | **Fire-and-Forget**, 책임 분리 (지향) |
