## 학습 키워드

  - 게임모드 (GameMode)
  - 폰 (Pawn), 캐릭터 (Character)
  - 캐릭터 무브먼트 컴포넌트 (Character Movement Component)
  - 스켈레탈 메시 (Skeletal Mesh)
  - 스프링 암 (Spring Arm), 카메라 컴포넌트
  - DefaultPawnClass

<br/>

## 학습 내용

### 배운 개념 요약

  - **게임모드 (GameMode)**

      - 게임의 전반적인 규칙과 흐름을 관리하는 컨트롤 타워 역할을 하는 클래스다.
      - 어떤 캐릭터(Pawn)를 스폰할지, 어떤 플레이어 컨트롤러를 사용할지, 승패 조건 등 게임의 핵심 로직을 담당한다.
      - `GameMode`는 멀티플레이 기능이 일부 포함된 버전이고, `GameModeBase`는 더 단순화된 버전이다.

  - **폰(Pawn) vs 캐릭터(Character)**

      - **Pawn**: 플레이어나 AI가 조종할 수 있는 가장 기본적인 클래스다. 이동 로직이 기본적으로 포함되어 있지 않아, 비행기나 드론처럼 특수한 이동 방식을 구현할 때 자유도가 높다.
      - **Character**: Pawn을 상속받은 클래스로, `CharacterMovementComponent`가 기본적으로 포함되어 있다. 걷기, 달리기, 점프 등 이족 보행 캐릭터에 필요한 대부분의 기능이 이미 구현되어 있어 사람 형태의 캐릭터를 만들기에 최적화되어 있다.

  - **Character의 기본 컴포넌트**

      - `CapsuleComponent`: 캐릭터의 충돌 범위를 정의하는 루트 컴포넌트.
      - `SkeletalMeshComponent`: 뼈대(Skeleton)가 있는 3D 모델과 애니메이션을 적용하는 컴포넌트.
      - `CharacterMovementComponent`: 이동, 점프, 중력 등 실제 움직임 로직을 담당하는 핵심 컴포넌트.

  - **3인칭 카메라 구현**

      - 일반적으로 `SpringArmComponent`와 `CameraComponent`를 함께 사용해 구현한다.
      - `SpringArmComponent`는 카메라와 캐릭터 사이의 거리를 유지하고, 벽 같은 장애물에 카메라가 파고드는 것을 막아주는 역할을 한다.
      - `CameraComponent`는 실제 플레이어가 보게 될 시점을 제공하는 카메라다.

<br/>

### 구현 과정

1.  **게임모드 생성 및 적용**

    1.  `AGameMode`를 상속받는 C++ 클래스 `ASpartaGameMode`를 생성한다.
    2.  에디터에서 C++ 클래스를 기반으로 `BP_SpartaGameMode` 블루프린트 클래스를 만든다. (C++로 구조를 잡고, 세부 설정은 블루프린트에서 하기 위함)
    3.  `Project Settings > Maps & Modes`에서 `Default GameMode`를 `BP_SpartaGameMode`로 설정하여 프로젝트 전역에 적용한다.

2.  **캐릭터 생성 및 설정**

    1.  `ACharacter`를 상속받는 C++ 클래스 `ASpartaCharacter`를 생성한다.
    2.  이를 기반으로 `BP_SpartaCharacter` 블루프린트 클래스를 만든다.
    3.  `BP_SpartaCharacter`의 `Mesh` 컴포넌트를 선택하고, Details 패널에서 `Skeletal Mesh` 에셋(`SKM_Manny` 등)을 할당한다.
    4.  메시가 캐릭터의 전방(X축)을 보도록 회전 값을 조정하고(Yaw: -90), 캡슐 컴포넌트 바닥에 발이 오도록 위치를 조정한다.

3.  **3인칭 카메라 추가 (C++)**

    1.  `SpartaCharacter.h`에 `USpringArmComponent`와 `UCameraComponent` 포인터 변수를 `UPROPERTY`와 함께 선언한다.
    2.  `SpartaCharacter.cpp` 생성자에서 `CreateDefaultSubobject`를 사용해 두 컴포넌트를 생성한다.
    3.  `SpringArm`을 루트 컴포넌트에 부착하고, `Camera`는 `SpringArm`에 부착한다.
    4.  컨트롤러의 회전(마우스 움직임)이 카메라에 반영되도록 `SpringArm`의 `bUsePawnControlRotation`을 `true`로 설정하고, `Camera`의 동일 옵션은 `false`로 설정한다.

    ```cpp
    // ASpartaCharacter.cpp 생성자
    ASpartaCharacter::ASpartaCharacter()
    {
        // 스프링 암 생성 및 루트에 부착
        SpringArmComp = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
        SpringArmComp->SetupAttachment(RootComponent);
        SpringArmComp->TargetArmLength = 300.0f;
        SpringArmComp->bUsePawnControlRotation = true; // 컨트롤러 회전 값을 사용

        // 카메라 생성 및 스프링 암에 부착
        CameraComp = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
        CameraComp->SetupAttachment(SpringArmComp, USpringArmComponent::SocketName);
        CameraComp->bUsePawnControlRotation = false; // 카메라는 스프링 암을 따라가므로 이 옵션은 끔
    }
    ```

4.  **기본 캐릭터 스폰 설정**

    1.  `ASpartaGameMode.cpp` 생성자에서 `DefaultPawnClass`를 `ASpartaCharacter::StaticClass()`로 설정한다.
    2.  더 좋은 방법은 에디터에서 `BP_SpartaGameMode`를 열고 `Default Pawn Class`를 `BP_SpartaCharacter` 블루프린트로 직접 지정하는 것이다.

<br/>

## 느낀점

  - `GameMode`, `Character`, `PlayerController` 등 언리얼 게임 프레임워크가 제공하는 기본 클래스들의 역할과 관계를 이해하는 것이 정말 중요하다. 이 구조를 따라가기만 해도 캐릭터 구현의 뼈대가 쉽게 잡힌다.
  - `Pawn`으로 직접 이동 로직을 구현하는 것과 `Character` 클래스를 사용하는 것의 차이를 명확히 알게 되었다. 대부분의 경우 `Character` 클래스가 제공하는 기능을 활용하는 것이 훨씬 효율적이다.
  - C++로 클래스의 기본 구조와 핵심 기능을 정의하고, 블루프린트에서는 에셋을 할당하거나 프로퍼티 값을 미세 조정하는 하이브리드 방식의 강력함을 체감했다. 코드 수정 없이 에디터에서 빠르게 값을 바꿔보며 테스트할 수 있다는 점이 큰 장점이다.

-----

### 요약

언리얼 엔진에서 캐릭터를 구현하려면 `GameMode`로 게임 규칙을 설정하고, `ACharacter`를 상속받아 플레이어 캐릭터 클래스를 만든다. `Character` 클래스는 이동, 충돌, 모델 표시를 위한 컴포넌트들을 기본으로 제공한다. 3인칭 카메라는 `SpringArmComponent`와 `CameraComponent`를 C++로 추가하여 구현하며, 최종적으로 `GameMode`의 `DefaultPawnClass`에 생성한 캐릭터를 지정하면 게임 시작 시 자동으로 스폰된다.
