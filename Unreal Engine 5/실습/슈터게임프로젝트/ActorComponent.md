# TIL: 언리얼 엔진의 핵심, 컴포넌트(Component) 파헤치기

- 컴포넌트(Component)는 액터(Actor)에 부착하여 특정 기능이나 데이터를 제공하는 '부품'과 같습니다. 
- 이 부품들을 조립하여 캐릭터, 무기, 문 등 게임 세상의 모든 오브젝트를 완성할 수 있습니다.
- 오늘은 언리얼 엔진 아키텍처의 근간을 이루는 컴포넌트의 개념과 동작 방식에 대해 깊이 있게 학습한 내용을 공유합니다.

-----

### 왜 컴포넌트를 알아야 할까요?

- 게임 클라이언트 개발에서 **컴포넌트 기반 아키텍처**(Component-Based Architecture)를 이해하는 것은 필수적입니다.
- 단순히 기능을 구현하는 것을 넘어, **재사용성**, **유연성**, **유지보수성**을 크게 향상시킬 수 있기 때문입니다.

- 예를 들어, '공격 기능'을 컴포넌트로 만들어두면 플레이어 캐릭터, NPC, 심지어는 공격하는 포탑에도 동일한 컴포넌트를 부착하여 기능을 쉽게 추가할 수 있습니다.
- 이는 상속(Inheritance)만으로는 해결하기 어려운 다중적인 특징을 객체에 부여할 때 강력한 해법이 됩니다.
- 면접에서도 "액터와 컴포넌트의 관계"는 단골 질문일 만큼, 그 중요성은 아무리 강조해도 지나치지 않습니다.

-----

### 컴포넌트, 무엇이 다른가요? - 상속 구조 파헤치기

- 언리얼 엔진의 모든 컴포넌트는 `UActorComponent`로부터 시작하지만, 역할에 따라 크게 세 가지 핵심 클래스로 나뉩니다.

####  `UActorComponent`: 보이지 않는 기능의 조립

  * **정의**: 모든 컴포넌트의 **최상위 부모 클래스**입니다.
  * **특징**: **물리적인 형태나 위치(`Transform`)가 없습니다.** 오직 추상적인 데이터나 기능만을 담당합니다.
  * **예시**:
      * 캐릭터의 인벤토리 시스템 (`UInventoryComponent`)
      * 체력, 마나 같은 스탯 관리 (`UStatComponent`)
      * AI의 특정 행동 패턴 로직
  * **핵심**: 눈에 보이지 않는 '**기능**'이나 '**데이터**' 덩어리를 액터에 붙이고 싶을 때 사용합니다.

#### `USceneComponent`: 월드 내 '위치'를 가지다

  * **정의**: `UActorComponent`를 상속받아 **`Transform`(위치, 회전, 크기)을 추가**한 클래스입니다.
  * **특징**: 월드 공간에 물리적인 위치를 가질 수 있습니다. 덕분에 다른 `USceneComponent`에 **계층 구조**(Hierarchy)로 붙일 수 있습니다.
  * 액터는 반드시 하나의 `USceneComponent`를 **루트 컴포넌트**(Root Component)로 가져야 하며, 이 루트 컴포넌트의 `Transform`이 곧 액터의 `Transform`이 됩니다.
  * **예시**:
      * 카메라를 플레이어 뒤에서 일정 거리 유지시켜주는 `USpringArmComponent`
      * 파티클이나 사운드를 발생시킬 위치를 지정하는 역할
  * **핵심**: 액터 내에서 무언가를 '**배치**'하고 싶을 때 사용합니다. 그 자체로 보이지는 않지만, 위치 정보를 가지는 모든 것의 기반이 됩니다.

#### `UPrimitiveComponent`: '모양'을 가지고 상호작용하다

  * **정의**: `USceneComponent`를 상속받아 **렌더링될 수 있는 기하학적 형태(Geometry)나 물리적 충돌**(Collision) 기능을 추가한 클래스입니다.
  * **특징**: 화면에 직접 그리거나 다른 오브젝트와 부딪히는 등, 실질적인 물리적 상호작용이 가능합니다.
  * **예시**:
      * 3D 모델을 보여주는 `UStaticMeshComponent`, `USkeletalMeshComponent`
      * 보이지는 않지만 충돌 판정에 사용되는 `UBoxComponent`, `UCapsuleComponent`, `USphereComponent`
  * **핵심**: 게임 월드에서 **'보이거나' '부딪히는'** 모든 것은 `UPrimitiveComponent`의 자식 클래스입니다.

-----

### 컴포넌트는 어떻게 일하나요? - 생명주기와 업데이트

컴포넌트는 액터에 붙어있다고 해서 저절로 동작하지 않습니다. 엔진이 그 존재를 인지하고 관리하는 과정, 즉 **생명주기**(Lifecycle)를 가집니다.

#### 등록과 해제: 컴포넌트를 씬(Scene)에 알리기

컴포넌트가 제 기능을 하려면(업데이트, 렌더링, 물리 시뮬레이션 등) 반드시 씬에 **등록**(Register)되어야 합니다.

  * **자동 등록**: 액터가 생성될 때 함께 생성된 컴포넌트들은 자동으로 등록됩니다.
  * **수동 등록**: 게임 플레이 중에 동적으로 컴포넌트를 생성했다면, `RegisterComponent()` 함수를 직접 호출해야 합니다.
      * **주의!**: 동적 등록/해제(`UnregisterComponent`)는 성능 부하를 유발할 수 있으므로 꼭 필요할 때만 사용해야 합니다.

#### 업데이트 (틱): 매 프레임 살아 움직이게 하기

액터처럼 컴포넌트도 매 프레임 특정 로직을 수행하는 **틱(Tick)** 기능을 가질 수 있습니다.

  * `TickComponent()` 함수를 오버라이드하여 프레임마다 실행할 코드를 작성합니다.
  * **활성화 방법**: 틱 기능은 기본적으로 비활성화되어 있습니다. 사용하려면 생성자에서 `bCanEverTick`을 `true`로 설정해야 합니다.



```cpp
// MyAwesomeComponent.h
public:
    virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;

// MyAwesomeComponent.cpp
UMyAwesomeComponent::UMyAwesomeComponent()
{
    // 이 컴포넌트가 틱을 사용할 수 있도록 설정합니다.
    PrimaryComponentTick.bCanEverTick = true;
}

void UMyAwesomeComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
    Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

    // 매 프레임 실행될 로직을 여기에 작성합니다.
    // 예: 일정 시간마다 레이저 발사, UI 업데이트 등
}
```

  * **성능 팁**: `TickComponent`가 필요 없는 컴포넌트는 `bCanEverTick = false` (기본값)로 두어 불필요한 연산을 줄이는 것이 좋습니다.

-----

### 오늘 새롭게 배운 점 및 오해 바로잡기

  * **오해**: "모든 컴포넌트는 액터의 위치를 따라간다."

      * **바로잡기**: **오직 `USceneComponent`와 그 자식들만 `Transform`을 가집니다.** `UActorComponent`는 위치 개념이 없는 순수한 로직/데이터 컨테이너입니다. 이 차이를 모르면 컴포넌트를 잘못 설계하기 쉽습니다.

  * **새로운 발견**: "에디터에서만 보이는 컴포넌트를 만들 수 있다."

      * 개발 편의를 위해 액터의 특정 범위나 정보를 에디터에서만 시각적으로 표시하고 싶을 때가 있습니다. 예를 들어 AI의 시야 범위 같은 경우죠.
      * 이때 **시각화 컴포넌트**(Visualization Component)를 사용하고, `#if WITH_EDITORONLY_DATA` 전처리기 지시문으로 감싸주면 게임 빌드에는 포함되지 않아 성능에 영향을 주지 않습니다.

    

    ```cpp
    #if WITH_EDITORONLY_DATA
        // 개발자만 에디터에서 볼 수 있는 디버그용 컴포넌트
        UPROPERTY()
        class UDebugSphereComponent* SightRadiusSphere;
    #endif
    ```

-----

###  실제 프로젝트 적용 및 다음 학습 계획

  * **프로젝트 적용 방안**:

      * 플레이어 캐릭터(`ACharacter`)를 설계할 때, 이동과 충돌은 내장된 `UCapsuleComponent`와 `UCharacterMovementComponent`를 활용합니다.
      * 무기 시스템은 별도의 `UWeaponComponent`(`UActorComponent` 상속)로 만들어 장착/해제 기능을 구현합니다.
      * 실제 무기 모델과 발사 효과는 `UWeaponComponent`가 소유하는 `UStaticMeshComponent`와 `UParticleSystemComponent`로 처리하는 구조를 구상해볼 수 있습니다.

  * **향후 학습 방향**:

      * **커스텀 컴포넌트 제작**: 프로젝트에 특화된 `UActorComponent`와 `USceneComponent`를 직접 설계하고 구현해보기.
      * **컴포넌트와 네트워크**: 멀티플레이 환경에서 컴포넌트의 프로퍼티와 함수 호출이 어떻게 복제(Replication)되는지 심도 있게 학습하기.
      * **렌더링 파이프라인과 컴포넌트**: `UPrimitiveComponent`가 `FPrimitiveSceneProxy`를 통해 렌더 스레드와 어떻게 데이터를 주고받는지 원리 파악하기.

-----

###  참고 자료

  * [언리얼 엔진 공식 문서: 컴포넌트](https://www.google.com/search?q=https://docs.unrealengine.com/5.3/ko/components-in-unreal-engine/)
  * [언리얼 엔진 공식 문서: 액터와 컴포넌트](https://www.google.com/search?q=https://docs.unrealengine.com/5.3/ko/actors-and-components-in-unreal-engine/)
