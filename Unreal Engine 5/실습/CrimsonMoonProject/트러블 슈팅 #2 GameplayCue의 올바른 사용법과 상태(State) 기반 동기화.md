## 학습 키워드

* Unreal Engine 5
* GAS(Gameplay Ability System)
* GameplayCue
* GameplayEffect
* Networking
* Replication

## 학습 내용

### 배운 개념 요약

멀티플레이어 게임 개발 시, 사운드(Sound)나 VFX 같은 시각/청각적 피드백을 다룰 때 가장 중요한 것은 네트워크 신뢰성(Reliability)이다. 기존에 사용하던 '명령 기반' 방식의 문제점을 파악하고, 언리얼이 **'상태(State) 기반'** 처리 방식을 학습했다.

#### 1. 문제점: 명령 기반 (Imperative) 관리

기존에는 어빌리티 시작 시 `AddGameplayCue`, 종료 시 `RemoveGameplayCue`를 직접 호출했다.

* **방식:** "소리를 켜라(RPC)" -> "소리를 꺼라(RPC)"
* **위험성:** 네트워크 렉이나 패킷 손실(Packet Loss)로 인해 `Remove` 명령이 클라이언트에 도달하지 못하면, 게임이 끝날 때까지 사운드가 무한 재생되는 치명적인 버그(Ghost Sound)가 발생한다.

#### 2. 해결책: 상태 기반 (Declarative) 관리 (GAS 표준)

GameplayEffect(GE)를 사용하여 캐릭터의 '상태'를 정의하고, 엔진이 이를 동기화하게 만든다.

* **방식:** "캐릭터는 지금 [공격 중] 상태이다." (GE 적용)
* **안전성:** 서버에서 GE를 제거하면, 리플리케이션 시스템이 이를 감지하여 클라이언트의 연결된 GameplayCue를 강제로 정리(Garbage Collection)한다. 인터넷이 끊겼다 재연결되어도 현재 상태(GE 없음)를 확인하여 즉시 소리를 끈다.

#### 3. Loop vs Burst 처리 기준

* **지속형(Looping):** 레이저, 차징, 버프 오라 등. **Duration이 Infinite인 GE**를 사용하며, `GameplayCueNotify_Actor`로 처리한다.
* **단발성(Burst):** 총구 화염, 타격음 등. **Instant GE**나 **ExecuteCue**를 사용하며, `GameplayCueNotify_Static`으로 처리한다.

### 구현 과정

기존의 `CMPGAbility_BaseAttack` 클래스에서 태그를 직접 관리하던 코드를 **GameplayEffect 핸들**을 사용하는 방식으로 리팩토링했다.

#### 1. 헤더 파일 수정 (`CMPGAbility_BaseAttack.h`)

직접적인 `FGameplayTag` 대신, 상태를 정의할 GE 클래스와 이를 추적할 핸들을 선언한다.

```cpp
protected:
    /* [수정 전]
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "GameplayCue", meta = (AllowPrivateAccess = "true"))
    FGameplayTag SoundCueTag;
    */

    // 기존의 단순 Tag 대신, Cue가 포함된 지속형 GE 클래스를 사용한다.
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "CM|Effects")
    TSubclassOf<UGameplayEffect> LoopSoundEffectClass;

    // 적용된 GE를 나중에 끄기 위해 핸들을 저장한다.
    FActiveGameplayEffectHandle ActiveLoopSoundHandle;
```

#### 2. 소스 파일 수정 (`CMPGAbility_BaseAttack.cpp`)

`ActivateAbility`에서 GE를 적용하고, `EndAbility`에서 해당 GE를 제거한다. 이렇게 하면 중간에 어떤 사고가 발생해도 엔진이 GE의 수명에 맞춰 사운드/VFX를 관리해 준다.

```cpp
void UCMPGAbility_BaseAttack::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
    Super::ActivateAbility(Handle, ActorInfo, ActivationInfo, TriggerEventData);

    // ... (유효성 검사 생략) ...

    /*
    // [수정 전] 사운드 GameplayCue 추가 (지속형 - 스킬 취소 시 중지 가능)
	  if (SoundCueTag.IsValid())
	  {
		  UAbilitySystemComponent* SourceASC = GetAbilitySystemComponentFromActorInfo();
		  if (SourceASC)
		  {
			  FGameplayCueParameters CueParams;
			  CueParams.Instigator = GetAvatarActorFromActorInfo();
  
			  SourceASC->AddGameplayCue(SoundCueTag, CueParams);
		  }
	  }
    */

    // [변경] 사운드/VFX가 포함된 GE 적용 (State 기반 관리)
    if (LoopSoundEffectClass)
    {
        UAbilitySystemComponent* ASC = GetAbilitySystemComponentFromActorInfo();
        if (ASC)
        {
            FGameplayEffectContextHandle ContextHandle = ASC->MakeEffectContext();
            ContextHandle.AddSourceObject(this);

            FGameplayEffectSpecHandle SpecHandle = ASC->MakeOutgoingSpec(LoopSoundEffectClass, GetAbilityLevel(), ContextHandle);
            
            if (SpecHandle.IsValid())
            {
                // GE를 적용하고 핸들을 저장한다.
                ActiveLoopSoundHandle = ASC->ApplyGameplayEffectSpecToSelf(*SpecHandle.Data.Get());
            }
        }
    }
    
    // ... (몽타주 재생 로직) ...
}

void UCMPGAbility_BaseAttack::EndAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, bool bReplicateEndAbility, bool bWasCancelled)
{
    /*
    // [수정 전] 사운드 GameplayCue 제거 (스킬 취소 시 사운드도 중지)
    if (SoundCueTag.IsValid())
    {
		  GetAbilitySystemComponentFromActorInfo()->RemoveGameplayCue(SoundCueTag);
	  }
    */

    // [변경] 저장해둔 핸들로 GE 제거
    // 서버에서 제거되면 클라이언트의 Cue도 자동으로 종료된다.
    if (ActiveLoopSoundHandle.IsValid())
    {
        UAbilitySystemComponent* ASC = GetAbilitySystemComponentFromActorInfo();
        if (ASC)
        {
            ASC->RemoveActiveGameplayEffect(ActiveLoopSoundHandle);
        }
        ActiveLoopSoundHandle.Invalidate();
    }

    Super::EndAbility(Handle, ActorInfo, ActivationInfo, bReplicateEndAbility, bWasCancelled);
}

```

## 느낀점

단순히 "기능이 작동한다"는 것과 "네트워크 환경에서 안전하다"는 것은 완전히 다른 차원의 문제임을 깨닫게 되었다. 이전 방식(직접 Cue 관리)은 로컬 테스트에서는 완벽해 보였지만, 실제 멀티플레이어 환경에서는 언제든 버그를 일으킬 수 있는 시한폭탄과 같았다.

오늘 학습을 통해 "명령(Event)"보다는 **"상태(State)"를 신뢰해야 한다**는 GAS의 핵심 철학을 깊이 이해하게 되었다. 앞으로의 개발에서는 사운드나 VFX 하나를 넣더라도, 이것이 '상태'인지 '사건'인지를 먼저 고민하고 설계하는 습관을 들여야 한다.

---

### 요약: GameplayCue 관리 방식 비교

| 특징 | 기존 방식 (명령형) | 개선된 방식 (선언형/상태기반) |
| --- | --- | --- |
| **코드** | `AddGameplayCue` / `RemoveGameplayCue` | `ApplyGameplayEffect` / `RemoveActiveGameplayEffect` |
| **원리** | 일회성 명령(RPC) 전송 | 데이터(State) 동기화 (Replication) |
| **안전성** | 패킷 유실 시 **무한 재생 버그** 발생 | 상태 변경 감지로 **자동 복구 및 종료** |
| **Lyra** | 사용 지양 (Instant에만 제한적 사용) | **모든 지속형(Looping) 효과의 표준** |
