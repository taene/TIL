## 학습 키워드
- `FindComponentByClass`
- `GetComponentByClass`
- `TSubclassOf`
- 언리얼 엔진 C++
- 블루프린트
- 템플릿(Template)
- 타입 캐스팅(Type Casting)

## 학습 내용
### 배운 개념 요약
- **`FindComponentByClass<T>()`**: **C++ 네이티브 전용** 함수. 템플릿을 사용하여 컴파일 타임에 타입을 결정하므로 **타입 캐스팅이 필요 없고**, 코드가 간결하며 성능상 이점이 있다. 블루프린트에서는 호출할 수 없다.
- **`GetComponentByClass(UClass* Class)`**: **블루프린트 호환용** 함수. `TSubclassOf` 파라미터를 통해 런타임에 클래스 정보를 받아 동작한다. 항상 `UActorComponent*`를 반환하므로 **`Cast<>`를 통한 타입 변환이 필수**이다. 내부적으로는 `FindComponentByClass`를 호출하는 래퍼(Wrapper) 함수다.

### 핵심 차이점

| 구분 | `FindComponentByClass<T>` | `GetComponentByClass` |
| :--- | :--- | :--- |
| **주요 사용처** | C++ 네이티브 코드 | 블루프린트 노출 및 동적 클래스 처리 |
| **타입 결정 시점** | 컴파일 타임 | 런타임 |
| **반환 타입** | `T*` (구체적인 컴포넌트 타입) | `UActorComponent*` (부모 타입) |
| **타입 캐스팅** | **불필요** | **필수** |
| **C++ 문법** | `FindComponentByClass<UMyComponent>()` | `GetComponentByClass(UMyComponent::StaticClass())` |
| **블루프린트** | 호출 불가 | **호출 가능** (`BlueprintCallable`) |


### 구현 과정
**1. `FindComponentByClass<T>`: 이상적인 C++ 사용법**
- C++ 코드 내에서 컴포넌트에 접근할 때 가장 먼저 고려해야 할 표준 방식이다.
- 반환값이 명확한 타입으로 정해져 있어 바로 멤버 함수나 변수에 접근할 수 있다.

```cpp
// AMyCharacter.cpp
void AMyCharacter::SomeFunction()
{
    // UHealthComponent 타입을 컴파일 시점에 알고 있음
    if (UHealthComponent* HealthComponent = FindComponentByClass<UHealthComponent>())
    {
        // Cast가 필요 없이 바로 사용 가능
        HealthComponent->Heal(100.0f);
    }
}
```

**2. `GetComponentByClass`: 블루프린트 호환 및 동적 처리**

  - C++에서 사용하면 `Cast<>`가 필요해 코드가 길어진다.
  - 컴포넌트 타입을 변수(TSubclassOf)로 받아 동적으로 처리해야 하거나, 블루프린트에 기능을 노출할 때 사용한다.

```cpp
// AMyActor.cpp
// 헤더 파일에 UPROPERTY(EditAnywhere) TSubclassOf<UActorComponent> TargetComponentClass; 선언 필요

void AMyActor::SomeDynamicLogic()
{
    // 데이터로 지정된 컴포넌트 클래스를 런타임에 가져온다.
    if (TargetComponentClass)
    {
        UActorComponent* FoundComponent = GetComponentByClass(TargetComponentClass);

        // 찾은 컴포넌트를 구체적인 타입으로 캐스팅해야 한다.
        if (UMySpecificComponent* SpecificComponent = Cast<UMySpecificComponent>(FoundComponent))
        {
            SpecificComponent->DoSomething();
        }
    }
}
```

## 느낀점

  - 단순히 "블루프린트용"과 "C++용"으로 구분하는 것을 넘어, **컴파일 타임 타입 안정성**과 **런타임 유연성**이라는 핵심 차이를 이해하게 되었다.
  - `FindComponentByClass<T>`는 타입 안전성과 코드 간결성을 보장해주므로 **C++ 환경에서는 무조건 이 함수를 사용하는 것이 좋다.**
  - `GetComponentByClass`는 블루프린트와의 소통을 위한 '다리' 역할이거나, 라이라(Lyra)처럼 데이터 중심 아키텍처에서 컴포넌트 타입을 동적으로 결정해야 할 때 사용하는 **예외적인 경우**에 해당한다는 것을 깨달았다.
  - 잘못된 파라미터(`TSubclassOf` 대신 `StaticClass()`)를 사용했던 실수는 두 함수의 근본적인 설계 철학을 이해하지 못했기 때문이었다. 이제는 각 함수의 목적과 사용법을 명확히 구분할 수 있게 되었다. AAA급 프로젝트에서는 사소해 보이는 이런 부분에서부터 코드의 품질과 안정성이 결정된다는 것을 명심해야겠다.
