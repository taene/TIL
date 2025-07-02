## 학습 키워드

  - PlayerController
  - Enhanced Input System
  - 입력 액션 (Input Action, IA)
  - 입력 매핑 컨텍스트 (Input Mapping Context, IMC)
  - 모디파이어 (Modifier), 트리거 (Trigger)
  - AddMappingContext

<br/>

## 학습 내용

### 배운 개념 요약

  - **PlayerController**

      - 플레이어의 키보드, 마우스 등 입력 장치로부터 신호를 받아 해석하고, 현재 소유(Possess)하고 있는 폰(Pawn)에게 동작을 명령하는 핵심 클래스다. 
      - 언리얼 엔진은 "입력은 PlayerController에서, 동작은 Pawn에서" 처리하는 철학을 가지고 있어, 입력 로직과 실제 캐릭터의 동작 로직을 분리하여 코드를 구조적으로 관리할 수 있다. 

  - **Enhanced Input System**

      - 언리얼 엔진 5에서 사용되는 향상된 입력 시스템으로, 입력을 '입력 액션(IA)'과 '입력 매핑 컨텍스트(IMC)'라는 두 가지 개념으로 나누어 관리한다. 
      - **Input Action (IA)**: '점프', '이동'과 같이 플레이어의 행동을 추상화한 에셋이다. 이 액션이 어떤 종류의 값(Bool, Axis1D, Axis2D 등)을 반환할지 결정한다. 
      - **Input Mapping Context (IMC)**: IA와 실제 물리적 키(W, Spacebar, 마우스 움직임 등)를 연결하는 매핑 정보를 담고 있는 에셋이다. 게임 상황에 따라 특정 IMC를 활성화하거나 비활성화하여 입력 방식을 동적으로 제어할 수 있다.
      - **Modifiers & Triggers**:
          - **Modifier**: 입력 값을 변형하는 기능이다. (예: `Negate`로 축 값 반전, `Swizzle`로 축 순서 변경)
          - **Trigger**: 입력이 언제 활성화될지 조건을 정의한다. (예: `Pressed`는 키를 누르는 순간, `Hold`는 누르고 있는 동안)

<br/>

### 구현 과정

1.  **PlayerController 생성 및 적용**

    1.  `APlayerController`를 상속받는 C++ 클래스 `ASpartaPlayerController`를 생성한다.
    2.  `ASpartaGameMode`의 생성자에서 `PlayerControllerClass`를 방금 만든 `ASpartaPlayerController::StaticClass()`로 지정하여 게임 시작 시 이 컨트롤러를 사용하도록 설정한다.
    3.  C++ 클래스를 기반으로 `BP_SpartaPlayerController` 블루프린트를 생성하여 에디터에서 쉽게 편집할 수 있도록 한다.

2.  **Input Action (IA) 에셋 생성 및 설정**

    1.  콘텐츠 브라우저에서 `Input > Input Action`을 선택해 필요한 액션들을 생성한다. (예: `IA_Move`, `IA_Jump`, `IA_Look`, `IA_Sprint`)
    2.  각 IA를 열어 `Value Type`을 설정한다.
          - `IA_Move`, `IA_Look`: 앞/뒤와 좌/우 2개 축을 동시에 처리하므로 `Axis2D (Vector2D)`로 설정.
          - `IA_Jump`, `IA_Sprint`: 단순 On/Off 동작이므로 `Digital (bool)`로 설정.

3.  **Input Mapping Context (IMC) 생성 및 매핑**

    1.  `Input > Input Mapping Context`를 선택해 `IMC_Character` 에셋을 생성한다.
    2.  `IMC_Character`를 열고, Mappings 목록에 위에서 만든 IA들을 추가한 뒤 실제 키를 할당한다.
          - **IA\_Move**: W, A, S, D 키에 각각 매핑한다. S키와 A키는 반대 방향이므로 `Modifiers`에 `Negate`를 추가해 입력 값을 반전시킨다. 키보드 입력은 단일 축이지만 `IA_Move`는 2D 벡터이므로 `Swizzle Input Axis Values`를 사용해 올바른 축(Y축)으로 값을 보내준다.
          - **IA\_Jump**: `Space Bar` 키에 매핑한다.
          - **IA\_Look**: `Mouse XY 2D-Axis`에 매핑한다. 카메라 상하(Pitch) 움직임은 보통 반대이므로 Y축에 `Negate` 모디파이어를 추가한다. 
          - **IA\_Sprint**: `Left Shift` 키에 매핑한다. 

4.  **PlayerController에서 IMC 활성화 (C++)**

    1.  `SpartaPlayerController.h`에 `UPROPERTY`를 사용하여 `UInputMappingContext*`와 사용할 `UInputAction*` 변수들을 선언한다. 
    2.  에디터에서 `BP_SpartaPlayerController`를 열고, 방금 선언한 변수들에 `IMC_Character`와 각 IA 에셋들을 할당한다.
    3.  `SpartaPlayerController.cpp`의 `BeginPlay()` 함수에서 `EnhancedInputLocalPlayerSubsystem`에 IMC를 추가하여 활성화시킨다.

    ```cpp
    // SpartaPlayerController.cpp
    #include "EnhancedInputSubsystems.h"

    void ASpartaPlayerController::BeginPlay()
    {
        Super::BeginPlay();

        // Enhanced Input Subsystem을 가져온다
        if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(GetLocalPlayer()))
        {
            // 블루프린트에서 할당한 IMC를 Subsystem에 추가하여 활성화한다
            if (InputMappingContext)
            {
                Subsystem->AddMappingContext(InputMappingContext, 0); // 우선순위 0으로 추가
            }
        }
    }
    ```

<br/>

## 느낀점

  - Enhanced Input System은 확실히 기존 입력 방식보다 구조적이고 유연하다. '액션(IA)'과 '키 매핑(IMC)'을 분리함으로써, 코드 변경 없이 키 설정만 바꾸거나 게임 상황에 따라 입력 체계를 통째로 교체(`Add/RemoveMappingContext`)하는 것이 가능해진다.
  - 처음에는 IA, IMC 등 여러 에셋을 만들고 연결하는 과정이 다소 번거롭게 느껴질 수 있지만, 이 구조 덕분에 입력 관련 로직이 훨씬 깔끔해지고 확장성이 좋아진다. 특히 UI 입력과 캐릭터 조작 입력을 별도의 IMC로 분리하여 관리하는 방식은 매우 효율적인 접근법이다.
  - `Modifier`와 `Trigger`의 존재는 이 시스템의 강력함을 보여주는 부분이다. 간단한 설정만으로도 입력 값을 변형하거나 입력 조건을 복잡하게 만들 수 있어, 코드 레벨에서 처리해야 할 많은 부분을 에셋 레벨에서 해결할 수 있다.

-----

### 요약

언리얼 엔진의 Enhanced Input System은 입력을 `Input Action(IA)`과 `Input Mapping Context(IMC)`로 분리하여 관리하는 유연한 시스템이다. `PlayerController`에서 `IA`와 `IMC` 에셋을 만들고 C++ 코드를 통해 `BeginPlay` 시점에 `AddMappingContext` 함수로 IMC를 활성화하면, 정의된 키 매핑에 따라 입력을 처리할 수 있다. 이 방식은 입력 로직을 명확하게 분리하고, 게임 상황에 따라 동적으로 입력 체계를 변경할 수 있게 해준다.
