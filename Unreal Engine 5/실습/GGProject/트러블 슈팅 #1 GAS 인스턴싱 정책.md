## 학습 키워드 <br>

  * Gameplay Ability System (GAS)
  * GameplayAbility
  * Instancing Policy
  * Instanced per Actor
  * Instanced per Execution
  * Class Default Object (CDO)

<br/>

## 학습 내용

### 배운 개념 요약 <br>

`GameplayAbility`의 **인스턴싱 정책**(Instancing Policy)은 어빌리티 객체의 생성 및 소멸 시점과 상태 관리 방식을 결정하는 핵심 설정이다. `Non-Instanced` 정책은 어빌리티를 실제 객체로 만들지 않고 클래스 기본 객체(CDO)에서 직접 호출하려 하기 때문에, 상태 관리가 어렵고 멀티플레이어 환경에서 문제를 일으켜 현재는 사용되지 않는다. 이로 인해 `operator () called on the CDO` 에러가 발생할 수 있다.

이를 해결하기 위한 두 가지 주요 인스턴싱 정책이 존재한다.

1.  **Instanced per Actor**

      * **생성/소멸**: 어빌리티가 액터에게 부여(Grant)될 때 생성되고, 박탈(Remove)될 때 소멸한다.
      * **상태 관리**: 인스턴스가 어빌리티를 소유하는 동안 계속 유지되므로, **쿨타임, 충전 횟수, 레벨** 등 영구적인 상태를 관리하는 데 적합하다.
      * **용도**: 가장 일반적이고 표준적인 방식으로, 대부분의 어빌리티에 사용된다.

2.  **Instanced per Execution**

      * **생성/소멸**: 어빌리티가 실행(Execute)될 때마다 새로운 인스턴스가 생성되고, 어빌리티가 종료(End)되면 즉시 소멸한다.
      * **상태 관리**: 어빌리티의 '단일 실행'에만 유효한 **일시적인 상태**(예: 채널링 시간, 특정 타겟 정보)를 관리하는 데 사용된다.
      * **용도**: 동시에 여러 개가 활성화되어야 하거나, 실행마다 독립적인 상태 추적이 필요한 특수한 경우(예: 차지 스킬, 다중 발사 투사체)에 사용된다.
      * 하지만 이번 문제에서는 Instanced Per Execution으로 지정해줬음에도 크래시가 떴다. 라이라에서는 Instanced Per Actor로 해야하는 것 같다...

### 구현 과정 <br>

`Non-Instanced` 정책으로 인해 발생한 에러를 해결하고, 어빌리티의 기획에 맞는 정책을 선택하는 과정은 다음과 같다.

1.  **문제 식별**: `Ensure condition failed: ... IsInstantiated()` 로그를 통해 어빌리티가 인스턴스화되지 않고 CDO에서 호출되었음을 확인한다.
2.  **정책 변경**: 문제가 발생한 `GameplayAbility` 블루프린트를 열고, `클래스 디폴트(Class Defaults)`에서 `Instancing Policy`를 `Non-Instanced`에서 다른 정책으로 변경한다.
3.  **정책 선택**:
      * 어빌리티가 쿨타임이나 스택 같은 영구적인 상태를 가져야 한다면 `Instanced per Actor`를 선택한다.
      * 어빌리티가 실행될 때마다 독립적인 상태(채널링, 지속 시간 등)를 가져야 한다면 `Instanced per Execution`을 선택한다.

C++ 코드에서는 어빌리티 클래스의 생성자에서 직접 정책을 설정할 수 있다.

```cpp
// ULyraGameplayAbility_Fire::ULyraGameplayAbility_Fire()

ULyraGameplayAbility_Fire::ULyraGameplayAbility_Fire()
{
    // 이 어빌리티는 쿨타임을 관리해야 하므로 '액터 당 인스턴스' 정책을 사용한다.
    InstancingPolicy = EGameplayAbilityInstancingPolicy::InstancedPerActor;
}
```

## 느낀점 <br>

어빌리티의 인스턴싱 정책은 단순히 에러를 해결하기 위한 기술적 설정을 넘어, 어빌리티의 동작 방식과 상태 관리의 근간을 결정하는 중요한 설계 요소임을 깨달았다. 특히 Lyra와 같은 복잡한 AAA 프로젝트에서 `Instanced per Actor`를 표준으로 삼는 이유는, 멀티플레이어 환경에서 모든 상태를 명확하고 예측 가능하게 관리하려는 철학이 담겨있기 때문이다.

과거에는 `Non-Instanced`가 단순한 최적화의 일환이라고 생각했지만, 이제는 왜 그것이 deprecated 되었고, 인스턴스화가 왜 필수적인지에 대해 깊이 이해하게 되었다. 앞으로 어빌리티를 설계할 때, 단순히 기능을 구현하는 것을 넘어 '이 어빌리티의 상태는 어떤 생명주기를 가져야 하는가?'를 먼저 고민하여 적절한 인스턴싱 정책을 선택하는 습관을 들여야겠다.
