## AttributeSet, HealthSet, CombatSet

## 1. `ULyraAttributeSet` - 모든 어트리뷰트 셋의 기반

- `ULyraAttributeSet`은 라이라 프로젝트에 사용되는 모든 어트리뷰트 셋의 **기반이 되는 부모 클래스**
- 직접적으로 체력이나 공격력 같은 속성을 가지진 않지만, 프로젝트 전반에 걸쳐 공통적으로 필요한 기능과 헬퍼 함수들을 제공하여 코드의 일관성과 편의성을 높이는 중요한 역할

### 📄 `LyraAttributeSet.h` 파일

### @매크로: `ATTRIBUTE_ACCESSORS`

```cpp
#**define** ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

이 매크로는 GAS에서 어트리뷰트를 편리하게 다루기 위해 꼭 필요한 **접근자(Accessor) 함수들을 자동으로 생성**해주는 매우 중요한 역할을 합니다. 예를 들어, `ATTRIBUTE_ACCESSORS(ULyraHealthSet, Health)` 라고 사용하면 내부적으로 다음과 같은 함수들이 만들어집니다.

- `static FGameplayAttribute GetHealthAttribute()`: `Health` 어트리뷰트 자체를 `FGameplayAttribute` 타입으로 반환합니다. 어빌리티 시스템 내부에서 특정 어트리뷰트를 식별할 때 사용됩니다.
- `float GetHealth() const`: 현재 `Health` 값을 `float`으로 가져옵니다.
- `void SetHealth(float NewVal)`: `Health` 값을 새로운 값으로 설정합니다.
- `void InitHealth(float NewVal)`: `Health` 값을 초기화합니다. `SetHealth`와 달리, 초기 설정 시에만 사용하기 위한 용도입니다.

이 매크로 덕분에 반복적인 Getter/Setter 코드를 작성할 필요 없이, 단 한 줄로 필요한 함수들을 모두 가질 수 있습니다.

### @델리게이트: `FLyraAttributeEvent`

```cpp
DECLARE_MULTICAST_DELEGATE_SixParams(FLyraAttributeEvent, AActor* /*EffectInstigator*/, AActor* /*EffectCauser*/, const FGameplayEffectSpec* /*EffectSpec*/, float /*EffectMagnitude*/, float /*OldValue*/, float /*NewValue*/);
```

어트리뷰트 값에 변화가 생겼을 때, 이 변화를 다른 시스템(UI, 사운드, 이펙트 등)에 알리기 위한 **멀티캐스트 델리게이트**입니다. 예를 들어 체력이 변경되었을 때, 이 델리게이트를 브로드캐스트하면 미리 이 델리게이트에 바인딩(연결)된 함수들이 모두 호출됩니다.

- `EffectInstigator`: 효과를 시작시킨 액터 (예: 스킬을 시전한 플레이어)
- `EffectCauser`: 효과를 물리적으로 유발한 액터 (예: 플레이어가 쏜 총알)
- `EffectSpec`: 적용된 게임플레이 이펙트의 모든 정보
- `EffectMagnitude`: 가공되지 않은 순수 변경 값
- `OldValue`: 변경 전 값
- `NewValue`: 변경 후 값

이처럼 델리게이트를 사용하면 어트리뷰트 셋이 UI나 다른 시스템을 직접 알 필요 없이, **"값이 바뀌었다"는 사실만 외부에 알리면 되므로 시스템 간의 결합도(Coupling)를 크게 낮출 수 있습니다.**

### @클래스 함수

- `UWorld* GetWorld() const override`: `UObject`의 가상 함수를 오버라이드하여, 이 어트리뷰트 셋이 속한 월드(레벨)의 포인터를 안전하게 가져옵니다.
- `ULyraAbilitySystemComponent* GetLyraAbilitySystemComponent() const`: 소유하고 있는 `UAbilitySystemComponent`를 라이라 전용인 `ULyraAbilitySystemComponent`로 캐스팅하여 반환하는 헬퍼 함수입니다. 프로젝트 고유의 기능을 사용하기 위해 필요합니다.

### 📄 `LyraAttributeSet.cpp` 파일

- `GetWorld()`: `GetOuter()`를 통해 자신을 소유한 액터를 찾고, 그 액터의 `GetWorld()`를 호출하여 월드를 반환합니다.
- `GetLyraAbilitySystemComponent()`: `GetOwningAbilitySystemComponent()`로 부모 ASC를 가져온 뒤, `Cast`를 통해 `ULyraAbilitySystemComponent` 타입으로 변환합니다.

---

## 2. `ULyraHealthSet` - 생명과 죽음의 관리자

- `ULyraHealthSet`은 캐릭터의 **체력(Health), 최대 체력(MaxHealth)과 관련된 모든 로직을 처리**하는 핵심 어트리뷰트 셋입니다.
- 단순히 값을 저장하는 것을 넘어, 데미지를 받거나 치유될 때의 복잡한 과정을 관리합니다.

### 📄 `LyraHealthSet.h` 파일

### @전역 게임플레이 태그

```cpp
LYRAGAME_API UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Gameplay_Damage);
```

`.cpp` 파일에서 정의될 게임플레이 태그를 다른 모듈에서도 사용할 수 있도록 외부에 선언하는 부분입니다. 문자열 대신 태그를 사용하면 오타를 방지하고 성능상 이점을 가질 수 있습니다.

### @어트리뷰트 변수

- **`FGameplayAttributeData Health`**: 현재 체력입니다. 가장 중요한 속성 중 하나죠. `Meta = (HideFromModifiers)` 지정자가 붙어있는데, 이는 일반적인 `GameplayEffect`로는 이 값을 직접 수정할 수 없음을 의미합니다. 오직 `GameplayEffectExecutionCalculation` (데미지 계산 실행 클래스)을 통해서만 변경이 가능하도록 강제하여, 체력 변경 로직을 한 곳에서 중앙 관리하기 위한 매우 중요한 설계입니다.
- **`FGameplayAttributeData MaxHealth`**: 최대 체력입니다. `Health`와 달리 `HideFromModifiers`가 없으므로, 버프나 아이템 효과 등으로 최대 체력을 직접 증가시키거나 감소시키는 것이 가능합니다.
- **`FGameplayAttributeData Damage`**: '메타(Meta) 어트리뷰트'입니다. 이 값은 영구적으로 저장되는 값이 아니라, 데미지 계산이 실행될 때 **잠시 동안 데미지 값을 담아두는 임시 변수** 역할을 합니다. `PostGameplayEffectExecute` 함수에서 이 `Damage` 값을 읽어 `Health`를 감소시킨 후, 다시 0으로 초기화합니다.
- **`FGameplayAttributeData Healing`**: `Damage`와 마찬가지로 치유량을 임시로 저장하는 메타 어트리뷰트입니다.

### @핵심 함수 (GAS 오버라이드)

`UAttributeSet`에서 상속받은 이 함수들은 **어트리뷰트 변경의 모든 과정에 개입하여 커스텀 로직을 실행**할 수 있는 강력한 हुक(Hook) 역할을 합니다.

- `PreGameplayEffectExecute()`: 게임플레이 이펙트가 어트리뷰트에 실제로 적용되기 **직전**에 호출됩니다. 라이라에서는 이 함수를 사용해 데미지 면역(`TAG_Gameplay_DamageImmunity`) 태그가 있는지, 혹은 무적 치트(`Cheat_GodMode`)가 활성화되었는지 등을 검사하여 데미지를 0으로 만들어버리는 '방어 로직'을 구현합니다.
- `PostGameplayEffectExecute()`: 게임플레이 이펙트가 적용된 **직후**에 호출됩니다. `Damage`나 `Healing` 같은 메타 어트리뷰트의 값을 실제 `Health`에 반영하고, `Health`가 0 이하로 떨어졌는지 검사하여 `OnOutOfHealth` 델리게이트를 호출하는 등 '결과 처리 로직'을 담당합니다.
- `PreAttributeChange()` / `PreAttributeBaseChange()`: `GameplayEffect`를 통하지 않고 어트리뷰트 값이 직접 변경되려고 할 때 호출됩니다. `Health`가 `MaxHealth`를 넘지 않도록, `MaxHealth`가 1 미만으로 떨어지지 않도록 값을 제한(Clamp)하는 역할을 합니다.

### 📄 `LyraHealthSet.cpp` 파일

### @복제(Replication)

```cpp
void ULyraHealthSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    // ...
	DOREPLIFETIME_CONDITION_NOTIFY(ULyraHealthSet, Health, COND_None, REPNOTIFY_Always);
	DOREPLIFETIME_CONDITION_NOTIFY(ULyraHealthSet, MaxHealth, COND_None, REPNOTIFY_Always);
}
```

서버에서 변경된 `Health`, `MaxHealth` 값을 클라이언트들에게 복제하기 위한 설정입니다. `REPNOTIFY_Always`는 값이 변하지 않았더라도 항상 `OnRep_` 함수를 호출하도록 하여, 클라이언트에서도 값 변경에 따른 로직(UI 업데이트 등)이 안정적으로 실행되도록 보장합니다.

### @`OnRep_Health()`

서버로부터 `Health` 값이 복제되었을 때 클라이언트에서 호출되는 함수입니다. 이 함수는 `OnHealthChanged` 델리게이트를 브로드캐스트하여 UI의 체력 바를 업데이트하는 등의 클라이언트 측 로직을 촉발시킵니다.

### @`PostGameplayEffectExecute()` 상세 로직

이 함수는 `ULyraHealthSet`의 심장부입니다.

1. **메시지 브로드캐스트**: 데미지가 발생하면, `UGameplayMessageSubsystem`을 통해 `FLyraVerbMessage`라는 구조체에 데미지 정보를 담아 게임 전체에 브로드캐스트합니다. 이 메시지를 구독하는 어떤 시스템(UI, 퀘스트, 통계 등)이든 데미지 발생 사실을 알 수 있게 됩니다. 이는 시스템 간의 의존성을 없애는 매우 세련된 설계 방식입니다.
2. **메타 어트리뷰트 처리**: `GetDamage()`로 임시 데미지 값을 가져와 `Health`에서 뺀 뒤, `SetDamage(0.0f)`를 호출하여 **메타 어트리뷰트를 즉시 0으로 초기화합니다.** 이 부분이 핵심입니다.
3. **델리게이트 브로드캐스트**: `Health` 값이 실제로 변경되었으면 `OnHealthChanged` 델리게이트를, `Health`가 0 이하가 되었으면 `OnOutOfHealth` 델리게이트를 호출하여 외부에 상태 변화를 알립니다.

---

## 3. `ULyraCombatSet` - 공격의 시작점

- `ULyraCombatSet`은 `ULyraHealthSet`과는 반대로, **피해를 주는 쪽(공격자)의 능력치**를 정의합니다. 예를 들어 무기의 기본 공격력, 스킬의 기본 치유량 등이 여기에 해당합니다.

### 📄 `LyraCombatSet.h` 파일

### @어트리뷰트 변수

- **`FGameplayAttributeData BaseDamage`**: 기본 데미지 값입니다. 실제 데미지 계산 `GameplayEffectExecutionCalculation`에서 이 값을 '소스(Source)' 즉, 공격자의 능력치로 가져가서 최종 데미지를 계산하는 데 사용합니다.
- **`FGameplayAttributeData BaseHeal`**: 기본 치유량 값입니다. `BaseDamage`와 동일한 원리로 치유량 계산에 사용됩니다.

### 📄 `LyraCombatSet.cpp` 파일

### 복제(Replication)

```cpp
void ULyraCombatSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    // ...
	DOREPLIFETIME_CONDITION_NOTIFY(ULyraCombatSet, BaseDamage, COND_OwnerOnly, REPNOTIFY_Always);
	DOREPLIFETIME_CONDITION_NOTIFY(ULyraCombatSet, BaseHeal, COND_OwnerOnly, REPNOTIFY_Always);
}
```

여기서 주목할 점은 복제 조건이 `COND_OwnerOnly`라는 것입니다. 이는 `BaseDamage` 같은 능력치는 **해당 액터를 소유한 클라이언트에게만 복제**하면 된다는 의미입니다. 다른 플레이어들은 내 캐릭터의 기본 공격력이 얼마인지 알 필요가 없으므로, 불필요한 네트워크 트래픽을 줄이는 최적화 기법입니다. 반면 `Health`는 모든 플레이어가 봐야 하므로 `COND_None`(모두에게 복제)을 사용했죠. 이런 작은 차이가 대규모 멀티플레이어 게임의 성능을 좌우합니다.

## Meta Attribute

정확한 지적입니다! `ReplicatedUsing` 속성이 없다는 것은 `Healing`과 `Damage`가 메타 어트리뷰트라는 강력한 **신호** 중 하나이지만, 그것이 **결정적인 이유**는 아닙니다.

메타 어트리뷰트인지 아닌지를 판단하는 가장 확실한 근거는 **그 속성이 코드 내에서 어떻게 사용되는지**를 보는 것입니다.

### 핵심 근거: `PostGameplayEffectExecute`에서의 "사용 후 초기화" 패턴

```cpp
// LyraHealthSet.cpp in PostGameplayEffectExecute

// ...

if (Data.EvaluatedData.Attribute == GetDamageAttribute())
{
    // 1. Damage 어트리뷰트의 값을 읽어서 Health를 변경
    SetHealth(FMath::Clamp(GetHealth() - GetDamage(), MinimumHealth, GetMaxHealth()));
    
    // 2. 사용이 끝난 Damage 어트리뷰트를 즉시 0으로 초기화
    SetDamage(0.0f);
}
else if (Data.EvaluatedData.Attribute == GetHealingAttribute())
{
    // 1. Healing 어트리뷰트의 값을 읽어서 Health를 변경
    SetHealth(FMath::Clamp(GetHealth() + GetHealing(), MinimumHealth, GetMaxHealth()));
    
    // 2. 사용이 끝난 Healing 어트리뷰트를 즉시 0으로 초기화
    SetHealing(0.0f);
}

// ...
```

위 코드에서 볼 수 있듯이, `Damage`와 `Healing`은 다음과 같은 패턴으로 사용됩니다.

1. **값 사용**: `GameplayEffect` 계산의 결과값이 임시로 `Damage`나 `Healing`에 담깁니다.
2. **실제 적용**: `PostGameplayEffectExecute`에서 이 임시 값을 가져와 실제 어트리뷰트인 `Health`에 더하거나 뺍니다.
3. **즉시 초기화**: 역할이 끝나자마자 해당 어트리뷰트의 값을 **`0.0f`로 리셋**합니다.

이처럼 **영구적인 상태를 저장하지 않고, 특정 계산이 실행되는 아주 짧은 순간에만 값을 담는 임시 변수(Scratchpad) 역할**을 하는 어트리뷰트를 '메타 어트리뷰트(Meta Attribute)'라고 부릅니다.

---

### 그렇다면 `ReplicatedUsing`이 없는 이유는?

바로 이것이 메타 어트리뷰트의 특성과 연결됩니다.

- **낭비**: `Damage`와 `Healing` 값은 서버에서 계산이 끝나는 즉시 0으로 돌아갑니다. 이 찰나의 순간을 위해 값을 클라이언트로 복제하는 것은 불필요한 네트워크 대역폭 낭비입니다.
- **결과만 중요**: 클라이언트는 '가해진 데미지'가 얼마였는지 아는 것보다, 그 결과로 '변경된 최종 Health' 값이 얼마인지만 알면 됩니다. 따라서 라이라는 `Health` 값만 `ReplicatedUsing`으로 복제하여 클라이언트의 체력 바를 갱신하는 것입니다.

## HealthSet (자세히)

### @생성자 및 네트워크 설정 함수

### `ULyraHealthSet()`

이것은 클래스의 **생성자**입니다. `ULyraHealthSet` 객체가 처음 생성될 때 호출되며, 멤버 변수들의 초기값을 설정하는 역할을 합니다.

- **Health와 MaxHealth**: 각각 `100.0f`로 초기화됩니다. 이는 캐릭터가 처음 스폰될 때 가지는 기본 체력과 최대 체력 값입니다.
- **bOutOfHealth**: 체력이 0이 되어 죽음 상태인지를 추적하는 bool 변수로, `false`로 초기화됩니다.
- **HealthBeforeAttributeChange, MaxHealthBeforeAttributeChange**: 속성값이 변경되기 직전의 값을 저장하는 임시 변수로, `0.0f`로 초기화됩니다.

### `void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const`

이 함수는 언리얼 엔진의 **네트워크 복제(Replication) 시스템**의 핵심적인 부분입니다. 서버에서 어떤 멤버 변수를 클라이언트에게 복제할지, 그리고 어떻게 복제할지를 설정합니다.

- `DOREPLIFETIME_CONDITION_NOTIFY` 매크로를 사용하여 `Health`와 `MaxHealth` 속성을 복제 대상으로 등록합니다.
- `COND_None`: 별도의 조건 없이 항상 복제함을 의미합니다.
- `REPNOTIFY_Always`: 값이 변경되지 않았더라도 서버가 알림(Notify)을 보내 `OnRep_` 함수가 클라이언트에서 항상 호출되도록 보장합니다. 이는 UI 업데이트 같은 로직이 안정적으로 실행되게 합니다.

---

### @복제 알림 (RepNotify) 함수

이 함수들은 서버에서 `Health` 또는 `MaxHealth` 값이 변경되어 클라이언트로 데이터가 복제되었을 때, **해당 클라이언트에서 자동으로 호출**됩니다. 주로 클라이언트 측의 시각적/청각적 피드백을 처리하는 데 사용됩니다.

### `void OnRep_Health(const FGameplayAttributeData& OldValue)`

`Health` 값이 복제되었을 때 호출됩니다.

1. `GAMEPLAYATTRIBUTE_REPNOTIFY` 매크로를 호출하여 어빌리티 시스템에 속성 복제가 일어났음을 알립니다.
2. `OnHealthChanged` 델리게이트를 브로드캐스트하여, 이 델리게이트에 연결된 다른 시스템(예: UI의 체력 바)에게 체력 변경 사실을 알립니다.
3. 만약 체력이 0 이하로 떨어졌다면, `OnOutOfHealth` 델리게이트를 추가로 브로드캐스트하여 죽음 관련 로직을 촉발시킵니다.

### `void OnRep_MaxHealth(const FGameplayAttributeData& OldValue)`

`MaxHealth` 값이 복제되었을 때 호출됩니다. `OnRep_Health`와 유사하게, `GAMEPLAYATTRIBUTE_REPNOTIFY` 매크로를 호출하고 `OnMaxHealthChanged` 델리게이트를 브로드캐스트하여 최대 체력 변경 사실을 외부에 알립니다.

---

### @게임플레이 이펙트 실행 흐름 (Execution Flow) 함수

이 함수들은 `UAttributeSet`의 핵심 기능으로, 게임플레이 이펙트(Gameplay Effect)가 적용되는 **생명주기의 각 단계에 개입**하여 커스텀 로직을 실행할 수 있게 해주는 강력한 हुक(Hook)입니다.

### `bool PreGameplayEffectExecute(FGameplayEffectModCallbackData& Data)`

게임플레이 이펙트가 속성 값을 실제로 **변경하기 직전**에 호출됩니다. 주로 방어 로직이나 조건부 무효화 처리에 사용됩니다.

1. 부모 클래스의 `PreGameplayEffectExecute`를 먼저 호출하여 기본 유효성 검사를 수행합니다.
2. 변경될 속성이 `Damage` 속성인지 확인합니다.
3. 대상이 데미지 면역 태그(`TAG_Gameplay_DamageImmunity`)를 가지고 있거나, 무적 치트(`Cheat_GodMode`) 상태인지 검사합니다.
4. 만약 면역/무적 상태라면, `Data.EvaluatedData.Magnitude` 값을 `0.0f`로 만들어 **데미지를 무효화**하고 `false`를 반환하여 이후 로직 실행을 막습니다.
5. 데미지가 유효하다면, 현재 `Health`와 `MaxHealth` 값을 `HealthBeforeAttributeChange`, `MaxHealthBeforeAttributeChange` 변수에 **백업**해둡니다.

### `void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data)`

게임플레이 이펙트가 속성 값을 **변경한 직후**에 호출됩니다. `Damage`나 `Healing` 같은 메타 어트리뷰트의 값을 실제 `Health`에 반영하고, 그 결과를 처리하는 핵심 로직을 담당합니다.

1. **데미지 처리**:
    - `Data.EvaluatedData.Attribute`가 `Damage`일 경우 실행됩니다.
    - `UGameplayMessageSubsystem`을 통해 게임 전체에 데미지 발생 사실을 알리는 `FLyraVerbMessage`를 브로드캐스트합니다.
    - `GetDamage()`로 임시 저장된 데미지 값을 가져와 현재 `Health`에서 뺀 뒤, `FMath::Clamp`로 값을 0과 `MaxHealth` 사이로 제한하여 다시 `Health`에 설정합니다.
    - 메타 어트리뷰트인 `Damage`를 `0.0f`로 초기화합니다.
2. **치유 처리**:
    - `Data.EvaluatedData.Attribute`가 `Healing`일 경우 유사하게 동작합니다. `Health`에 `Healing` 값을 더하고, `Healing`을 `0.0f`로 초기화합니다.
3. **결과 알림**:
    - 백업해 둔 `HealthBeforeAttributeChange`와 현재 `GetHealth()`를 비교하여, 체력에 실제 변화가 있었다면 `OnHealthChanged` 델리게이트를 브로드캐스트합니다.
    - 체력이 `0.0f` 이하로 떨어졌고, `bOutOfHealth`가 `false`라면 `OnOutOfHealth` 델리게이트를 브로드캐스트하여 죽음 이벤트를 발생시킵니다.

---

### @직접적인 속성 변경 처리 함수

이 함수들은 게임플레이 이펙트를 통하지 않고, `SetHealth()` 같은 함수로 속성 값이 **직접 변경될 때** 호출됩니다. 값의 범위를 제한하는 역할을 주로 합니다.

### `void PreAttributeBaseChange(const FGameplayAttribute& Attribute, float& NewValue) const`

속성의 '기본값(Base Value)'이 변경되기 직전에 호출됩니다. 내부적으로 `ClampAttribute`를 호출하여 값이 유효한 범위 내에 있도록 강제합니다.

### `void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)`

속성의 '현재값(Current Value)'이 변경되기 직전에 호출됩니다. `PreAttributeBaseChange`와 마찬가지로 `ClampAttribute`를 호출합니다.

### `void PostAttributeChange(const FGameplayAttribute& Attribute, float OldValue, float NewValue)`

속성 값이 **변경된 직후**에 호출됩니다.

- 만약 변경된 속성이 `MaxHealth`이고, 현재 `Health`가 새로운 `MaxHealth`보다 크다면, `Health` 값을 새로운 `MaxHealth` 값으로 강제로 낮춰줍니다. 이는 최대 체력이 감소했을 때 현재 체력이 최대치를 넘는 상황을 방지합니다.
- `bOutOfHealth`가 `true`였는데 `Health`가 `0`보다 커졌다면(부활 등), `bOutOfHealth`를 `false`로 되돌립니다.

### `void ClampAttribute(const FGameplayAttribute& Attribute, float& NewValue) const`

`PreAttribute...Change` 함수들에서 호출되는 **헬퍼 함수**입니다. 속성의 유효 범위를 강제합니다.

- `Health` 속성은 `0.0f`와 `GetMaxHealth()` 사이의 값으로 제한(Clamp)합니다.
- `MaxHealth` 속성은 최소 `1.0f` 이상이 되도록 강제합니다.

## CombatSet과 HealthSet 차이

`HealthSet`이 데미지를 **'받아서 처리하는'** 역할이라면, `CombatSet`은 데미지를 **'일으키는 원천'**의 데이터를 제공하는 역할입니다.

### @역할의 분리: 공격자 vs 방어자

가장 쉽게 이해하는 방법은 **공격자(Attacker)**와 **방어자/대상(Target)**의 관점으로 나누어 보는 것입니다.

- **`ULyraCombatSet`**: **공격자**의 능력치를 정의합니다.
    - `BaseDamage`는 "내가 한 번에 얼마의 기본 피해를 줄 수 있는가?"를 나타냅니다.
    - 이 값은 데미지 계산의 **시작점**이 되는 아주 중요한 입력값입니다.
- **`ULyraHealthSet`**: **대상**의 상태를 정의합니다.
    - `Health`는 "내가 얼마의 피해를 감당할 수 있는가?"를 나타냅니다.
    - 이곳의 `Damage` 메타 어트리뷰트는 계산이 완료된 **최종 결과값**을 잠시 받아두는 임시 저장소일 뿐입니다.

---

### @데미지가 처리되는 전체 과정

Player A가 Player B에게 총을 쏘는 상황을 예로 들어보겠습니다.

1. **1단계: 공격자 정보 확인 (CombatSet)**
    - Player A의 `AbilitySystemComponent`는 `ULyraCombatSet`을 가지고 있고, 여기에 `BaseDamage`가 `25.0f`로 설정되어 있습니다.
2. **2단계: 데미지 계산 (ExecutionCalculation)**
    - 총알이 Player B에게 맞으면, `GameplayEffect`가 Player B에게 적용됩니다.
    - 이 이펙트는 `GameplayEffectExecutionCalculation`(데미지 계산식)을 실행합니다.
    - 이 계산식은 먼저 **공격자인 Player A의 `CombatSet`에서 `BaseDamage` 값(25.0f)을 가져옵니다.** (Capture)
    - 여기에 헤드샷 배수, 거리 감쇠, 방어력 등 다른 변수를 추가로 계산하여 최종 피해량 `20.0f`를 도출합니다.
3. **3단계: 결과 전달 및 최종 처리 (HealthSet)**
    - 계산식은 도출된 최종 피해량 `20.0f`를 **대상이 되는 Player B의 `HealthSet`에 있는 `Damage` 메타 어트리뷰트에 전달**합니다.
    - 이제 Player B의 `HealthSet`에서 `PostGameplayEffectExecute`가 호출됩니다.
    - 자신의 `Damage` 어트리뷰트에 들어온 `20.0f`라는 값을 읽어서, 자신의 `Health`를 깎습니다.
    - 마지막으로 임시 저장소였던 `Damage` 값을 `0`으로 초기화하며 모든 과정이 끝납니다.

## OnRep 함수::`GAMEPLAYATTRIBUTE_REPNOTIFY`

- 그건 GAS의 **클라이언트 예측(Client-Side Prediction) 시스템**을 올바르게 동기화하기 위한 아주 중요한 절차입니다. 단순히 "값이 바뀌었다"고 알리는 것을 넘어, **"서버의 최종 결정이 도착했으니, 클라이언트의 예측과 비교하고 바로잡아라"**고 명령하는 역할을 합니다.

### @클라이언트 예측 시스템이란?

1. **클라이언트의 예측**: 플레이어가 스킬을 누르는 즉시, 클라이언트는 서버의 허락을 기다리지 않고 "아마 마나가 20 소모될 거야"라고 미리 예측하여 UI에 반영합니다.
2. **서버의 계산**: 그동안 서버는 실제로 스킬 사용이 유효한지, 마나가 얼마 소모되는지 정확하게 계산합니다.
3. **서버의 통보**: 계산이 끝나면 서버는 최종 결과를 클라이언트에게 보내줍니다. 이 최종 결과가 바로 `OnRep` 함수를 통해 클라이언트에 도착하는 것입니다.

### @`GAMEPLAYATTRIBUTE_REPNOTIFY`의 진짜 역할

- `OnRep` 함수가 호출되었다는 것은 서버의 '최종 정답'이 도착했다는 뜻입니다. 이때 `GAMEPLAYATTRIBUTE_REPNOTIFY` 매크로는 클라이언트의 예측 시스템에게 다음과 같은 일을 하라고 지시합니다.
1. **예측과 실제 값 비교**: "서버가 보내준 진짜 마나 값은 80이다. 네가 예측했던 값과 비교해봐라."
2. **예측이 맞았다면**: "네가 예측한 대로 80이 맞구나. 그럼 예측 시스템에서 관리하던 '임시 마나 소모' 기록을 깔끔하게 지우고 확정하자."
3. **예측이 틀렸다면 (중요!)**: "어라? 렉 때문에 스킬이 두 번 나간 걸로 예측했었네. 서버는 한 번만 처리해서 마나가 80이라고 한다. 네 예측은 틀렸으니, **서버 값을 기준으로 강제로 수정해라.**"
- 만약 이 매크로를 호출하지 않는다면, 클라이언트의 예측 시스템은 **자신이 예측했던 '임시 변경' 기록을 계속 붙들고 있게 됩니다.** 서버의 진짜 값이 도착했음에도 불구하고 이 사실을 통보받지 못했기 때문이죠.
- 결론적으로 `GAMEPLAYATTRIBUTE_REPNOTIFY`는 단순한 알림이 아니라, **서버가 최종 권한을 가지는(Server-Authoritative) 온라인 환경에서 클라이언트의 예측 오류를 바로잡고 데이터의 정합성을 보장하는 필수적인 안전장치**입니다.
