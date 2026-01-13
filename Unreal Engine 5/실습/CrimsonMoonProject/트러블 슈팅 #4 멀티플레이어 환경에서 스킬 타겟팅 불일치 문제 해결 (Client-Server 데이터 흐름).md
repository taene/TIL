## 학습 키워드

* **Unreal Engine 5**
* **GAS (Gameplay Ability System)**
* **Network Replication (네트워크 복제)**
* **RPC (Remote Procedure Call)**
* **LocalPredicted & Prediction Key**
* **Lyra Architecture (Client-Server Data Flow)**

## 학습 내용

### 배운 개념 요약

### 1. LocalPredicted 어빌리티와 실행 흐름의 이해
`LocalPredicted` 정책을 사용하는 어빌리티는 반응성을 극대화하기 위해 클라이언트에서 즉시 실행되고, 동시에 서버에서도 실행된다. 하지만 **동시에 실행된다**는 것이 **동일한 데이터에 접근한다**를 의미하지는 않는다.

* **Client Context:** 로컬 플레이어의 `PlayerController`, `CameraManager` 등에 접근 가능하여 정확한 조준점(Crosshair)과 카메라 방향을 알 수 있다.
* **Server Context:** 클라이언트의 `PlayerController`는 존재하지만, 클라이언트의 화면 렌더링 정보(카메라 위치, 회전 등)는 서버에 실시간으로 동기화되지 않는다. 서버는 오직 복제된 `Pawn`의 위치와 회전(Control Rotation) 정보만 알 뿐, 클라이언트가 화면에서 정확히 어디를 보고 있는지(Camera Trace)는 알 수 없다.

### 2. 데이터 중심 설계 (Data-Driven Design)와 RPC
이러한 불일치를 해결하기 위해 **GameplayAbilityTargetData**라는 구조체를 사용한다. 클라이언트가 타겟 정보(위치, 히트 결과 등)를 생성(Generate)하고, 이를 서버로 전송하여 검증(Validate) 후 적용(Apply)하는 구조다.
이번 문제 해결에서 사용한 **Server RPC** 방식은 이 거대한 구조의 가장 기초적이고 핵심적인 원리인 **클라이언트가 계산하고, 서버가 확정한다**는 철학을 구현한 것이다.

### 3. 주요 기술 요소

* **Server RPC (`UFUNCTION(Server, Reliable)`):** 클라이언트의 중요 데이터(타겟 위치)를 서버로 전송하는 필수적인 통신 수단이다. 게임 플레이에 필수적인 정보이므로 `Reliable`을 사용하여 전송을 보장해야 한다.
* **FVector_NetQuantize:** 네트워크 대역폭은 한정된 자원이다. 일반 `FVector`(double/float 정밀도)를 그대로 보내는 대신, 소수점 이하를 적절히 잘라 용량을 줄인 `NetQuantize` 버전을 사용하는 것은 대규모 멀티플레이어 게임 최적화의 기본이다.
* **IsLocallyControlled():** 코드가 실행되는 주체가 로컬 플레이어(자신)인지 확인하여, 서버나 다른 클라이언트에서의 중복/오동작 계산을 방지하는 필터 역할을 한다.

### 구현 과정

### 1. 문제 상황 분석
Advent F 스킬 사용 시, 시전자의 화면(Local Client)에서는 정상적으로 스킬이 나가지만, 다른 클라이언트(Server, Remote Client) 화면에서는 데미지가 없거나 엉뚱한 곳에 스킬이 터지는 현상이 발생했다.
디버깅 결과, 서버에서는 `GetPlayerCameraManager()`가 유효하지 않거나, 기대와 다른 위치를 반환하여 Trace 계산이 실패하고 있었다. 이는 서버가 클라이언트의 카메라 렌더링 파이프라인에 관여하지 않기 때문이다.

### 2. 해결 전략 수립

* **Step 1 (Client):** `IsLocallyControlled()` 블록 내부에서 로컬 플레이어의 카메라를 기준으로 정확한 타겟 위치(Hit Location)를 계산한다.
* **Step 2 (Network):** 계산된 위치 벡터를 `Server RPC`를 통해 서버로 전송한다. 이때 `FVector_NetQuantize`를 사용해 패킷 비용을 최소화한다.
* **Step 3 (Server):** 서버는 클라이언트가 보낸 위치를 신뢰(혹은 검증)하여 스킬 로직(Gameplay Effect 적용, Projectile 생성 등)을 수행한다.

### 3. 코드 구현 (C++)

```cpp
// AdventGameplayAbility.h

#include "CoreMinimal.h"
#include "Abilities/GameplayAbility.h"
#include "AdventGameplayAbility.generated.h"

UCLASS()
class ADVENTPROJECT_API UAdventGameplayAbility : public UGameplayAbility
{
    GENERATED_BODY()

public:
    virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;

protected:
    // 클라이언트에서 계산한 타겟 위치를 서버로 전달하는 RPC
    // Reliable: 스킬 발동은 중요하므로 반드시 도달해야 함
    // WithValidation: 보안을 위해 검증 로직 추가 가능 (여기서는 생략)
    UFUNCTION(Server, Reliable)
    void ServerSetTargetLocation(FVector_NetQuantize TargetLocation);

private:
    // 실제 스킬 로직을 수행하는 함수 (서버에서 실행됨)
    void PerformSkillLogic(const FVector& TargetLocation);
};

```

```cpp
// AdventGameplayAbility.cpp

#include "AdventGameplayAbility.h"
#include "GameFramework/PlayerController.h"

void UAdventGameplayAbility::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
    Super::ActivateAbility(Handle, ActorInfo, ActivationInfo, TriggerEventData);

    // 1. 로컬 클라이언트인 경우에만 카메라 계산 수행
    if (IsLocallyControlled())
    {
        APlayerController* PC = ActorInfo->PlayerController.Get();
        if (PC)
        {
            FVector CameraLoc;
            FRotator CameraRot;
            PC->GetPlayerViewPoint(CameraLoc, CameraRot);

            FVector TraceStart = CameraLoc;
            FVector TraceEnd = CameraStart + (CameraRot.Vector() * 5000.0f);

            FHitResult HitResult;
            FCollisionQueryParams QueryParams;
            QueryParams.AddIgnoredActor(ActorInfo->AvatarActor.Get());

            FVector FinalTargetLocation = TraceEnd;

            // LineTrace 수행
            if (GetWorld()->LineTraceSingleByChannel(HitResult, TraceStart, TraceEnd, ECC_Visibility, QueryParams))
            {
                FinalTargetLocation = HitResult.Location;
            }

            // 2. 계산된 위치를 서버로 전송 (RPC 호출)
            ServerSetTargetLocation(FinalTargetLocation);
        }
    }
}

void UAdventGameplayAbility::ServerSetTargetLocation_Implementation(FVector_NetQuantize TargetLocation)
{
    // 3. 서버에서 수신된 위치를 바탕으로 실제 로직 수행
    PerformSkillLogic(TargetLocation);
}

void UAdventGameplayAbility::PerformSkillLogic(const FVector& TargetLocation)
{
    // 여기서 GameplayCue 발동, 데미지 적용, 장판 생성 등을 처리
    // 이 함수는 서버에서 실행되므로 모든 클라이언트로 결과가 복제됨
    
    // 예: MakeOutgoingSpec -> ApplyGameplayEffectToTarget 등
}

```

## 느낀점

이번 트러블슈팅을 통해 **멀티플레이어 게임 프로그래밍은 단순히 로직을 짜는 것이 아니라, 데이터의 소유권(Authority)과 흐름(Flow)을 설계하는 것**임을 깊이 깨달았다.

싱글플레이어 개발 습관대로 "카메라 위치 가져와서 쏘면 되겠지"라고 생각했던 것이 문제의 원인이었다. 언리얼 엔진의 `LocalPredicted` 어빌리티가 클라이언트와 서버 양쪽에서 돈다는 사실을 머리로는 알고 있었지만, **각 컨텍스트(Context)에서 접근 가능한 데이터가 다르다**는 점을 간과했다.

또한 `GameplayAbilityTargetData`가 단순 벡터 하나뿐만 아니라 히트 결과, 액터 정보 등 다양한 타겟 데이터를 범용적으로 처리하기 위해 구조체와 인터페이스를 활용하여 추상화해 놓은 것임을 알았다.

앞으로는 코드를 작성하기 전에 항상 다음 세 가지를 자문해야 한다.

1. **Who:** 이 코드는 어디서 실행되는가? (클라/서버/둘다)
2. **What:** 해당 컨텍스트에서 이 데이터(Camera, Input 등)에 접근 가능한가?
3. **How:** 접근 불가능하다면, 누구에게 요청(RPC)하여 받아와야 하는가?

---

### 요약

| 구분 | 로컬 클라이언트 (Owner) | 서버 (Authority) | 해결 방법 (Lyra 철학) |
| --- | --- | --- | --- |
| **실행 흐름** | `ActivateAbility` 실행됨 | `ActivateAbility` 실행됨 | `LocalPredicted` 특성 이해 |
| **데이터 접근** | 카메라, 입력 정보 **접근 가능** | 카메라 정보 **접근 불가** (모름) | 데이터 불일치 발생 확인 |
| **역할** | 타겟팅 계산 (Trace) 및 데이터 생성 | 데이터 검증 및 실제 게임 로직 수행 | **Client Compute, Server Execute** |
| **통신** | `Server RPC`로 결과 전송 | `Implementation`에서 수신 후 처리 | `FVector_NetQuantize`로 최적화 |
