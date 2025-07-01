## 학습 키워드

  - Actor Lifecycle
  - UE\_LOG
  - BeginPlay
  - EndPlay
  - Constructor
  - PostInitializeComponents
  - Tick
  - Destroyed

## 학습 내용

### 배운 개념 요약

  - **언리얼 엔진 로그 (UE\_LOG)**

      - `UE_LOG` 매크로를 사용해 에디터의 'Output Log' 창에 디버그 메시지를 출력할 수 있다.
      - 매크로는 `로그 카테고리`, `로그 수준(Verbosity)`, `출력 메시지` 세 부분으로 구성된다.
      - `LogTemp`는 임시 카테고리이며, `DECLARE_LOG_CATEGORY_EXTERN`과 `DEFINE_LOG_CATEGORY`를 사용해 프로젝트 전용 카테고리를 만들어 로그를 체계적으로 관리할 수 있다.

  - **Actor 라이프 사이클**

      - Actor는 생성(Spawn), 초기화, 실행, 파괴(Destroy)의 생명 주기를 가지며, 각 단계에서 특정 함수가 자동으로 호출된다. 이 흐름을 이해하는 것은 컴포넌트 초기화, 리소스 관리, 성능 최적화에 필수적이다.

#### > **주요 라이프 사이클 함수 호출 순서 및 역할** <

##### 1.  **`생성자 (Constructor)`**
- 호출 시점: C++ 클래스 객체가 **메모리에 생성**될 때 딱 한 번 호출된다.
- 아직 액터가 월드 (World)에 완전히 등록되지 않은 상태이므로, **다른 액터나 월드 관련 기능**을 안전하게 호출하기 어렵다.
- 주로 `CreateDefaultSubobject` 등을 사용해 **컴포넌트 생성** 및 **초기 변수 세팅**에 활용한다.
##### 2.  **`PostInitializeComponents()`**
- 호출 시점: 액터의 모든 **컴포넌트**가 생성·초기화된 뒤 자동 호출된다.
- 각 컴포넌트가 이미 준비된 상태이므로, **컴포넌트 간 상호작용**(예: 서로 다른 컴포넌트 참조 설정)이 필요한 코드를 배치하기 좋다.
- 보통 생성자에서는 단순한 ‘생성/할당’만 하고, PostInitializeComponents()에서 컴포넌트들 사이의 의존 관계를 설정한다.
##### 3.  **`BeginPlay()`**
- 호출 시점: **Play In Editor** (PIE)나 **런타임**에서 게임이 시작될 때, 혹은 이미 실행 중인 게임에 `SpawnActor` 등으로 새 액터가 생성될 때 **한 번** 호출된다.
- 이 시점에는 이미 **월드와 다른 액터**들이 준비된 상태이므로, 자유롭게 상호작용이 가능하다.
- AI, 게임 모드, 플레이어 컨트롤러 등 **다른 시스템과 연동**도 주로 BeginPlay에서 초기화한다.
- 타이머, Delegate(Event) 바인딩 등을 시작하기에도 적합하다.
##### 4.  **`Tick(float DeltaTime)`**
- 호출 시점: 매 프레임마다 호출된다. (액터의 `PrimaryActorTick.bCanEverTick = true;` 설정 필요)
- **실시간 업데이트**가 필요한 로직 (캐릭터 이동, 물리 연산, 카메라 추적 등)을 처리한다.
- 이벤트 (Event) 기반으로 전환할 수 있는 부분은 `Tick`을 사용하지 않는 편이 **성능**에 유리하다.
##### 5.  **`Destroyed()`**
- 호출 시점: `Destroy()` 함수를 **직접 호출**하여 액터를 제거할 때 **직전에** 호출된다. 다만 **레벨 전환**이나 **게임 종료** 시에는 종종 건너뛰어지기도 하므로 (상황마다 엔진 동작이 달라짐), 절대적으로 보장되는 것은 아니다.
- `Destroyed()`가 불린 뒤에는 최종적으로 `EndPlay()`도 함께 호출된다.
- **수동으로 액터를 제거**할 때, 마지막 정리 코드를 넣을 수 있는 곳이다.
    - **But**, 게임 종료나 레벨 언로드 시에는 호출되지 않을 수 있으므로, **모든 중요한 정리**를 `Destroyed()`에만 의존하면 놓치는 케이스가 생길 수 있다.
- 정리할 자원 예시
    - **수동 할당한 메모리**: `new` 또는 동적 할당한 오브젝트가 있다면 여기서 `delete`하거나 해제한다.
    - **스폰된 자식 액터**: 이 액터가 생성한 다른 액터나 컴포넌트 중, **자동으로 해제되지 않는** 것이 있다면 제거 처리한다.
    - **Delegate / Event 바인딩**: 게임 전역적 또는 외부 클래스에 바인딩해둔 델리게이트가 있다면 해제한다.
    - **사운드/파티클 등**: 필요 시 이 액터가 재생 중인 사운드나 파티클을 수동으로 정리한다.
##### 6.  **`EndPlay(const EEndPlayReason::Type EndPlayReason)`**
- 호출 시점: 액터가 더 이상 월드에서 활동하지 않게 될 때 호출된다. (파괴, 레벨 전환, 게임 종료 등)
- `EEndPlayReason::Type`으로 어떤 이유로 EndPlay가 호출되었는지(파괴, 레벨 언로드, 게임 종료 등)를 구분한다.
- **게임 종료나 레벨 언로드** 같은 상황에서도 `EndPlay()`는 상대적으로 호출 보장이 높지만, Destroyed()는 건너뛸 수 있다.
    - 따라서 **중요한 정리 로직 (자원 해제, Timer 해제, 상태 저장 등)은 `EndPlay()`에 넣는 것**이 보다 안전하다.
- 정리할 자원 예시
    - **타이머**: `GetWorldTimerManager().ClearTimer(…)` 와 같이 타이머를 정리한다.
    - **동적 할당 리소스**: 여전히 해제되지 않은 동적 메모리가 남아 있다면 여기서 정리한다.
    - **데이터 저장**: 게임 진행 상황 (점수, 인벤토리 등)을 파일/DB에 저장하거나, 상위 시스템에 콜백을 보내는 로직도 EndPlay에서 처리 가능하다.

### 구현 과정

1.  **로그 카테고리 설정**

      - 헤더(.h) 파일에 `DECLARE_LOG_CATEGORY_EXTERN`를 사용해 새로운 로그 카테고리를 선언한다.
      - 소스(.cpp) 파일에 `DEFINE_LOG_CATEGORY`를 사용해 선언된 카테고리를 정의(구현)한다.

2.  **라이프 사이클 함수 오버라이드 및 로그 추가**

      - 액터의 헤더(.h) 파일에서 `BeginPlay`, `EndPlay` 등 오버라이드할 라이프 사이클 함수들을 선언한다.
      - 소스(.cpp) 파일에서 각 함수를 구현한다.
      - 구현부 첫 줄에는 항상 부모 클래스의 함수를 호출(`Super::FunctionName()`)한다.
      - `UE_LOG` 매크로와 `GetName()` 함수를 사용하여 어떤 액터의 어떤 라이프 사이클 함수가 호출되었는지 로그를 출력한다.

    ```cpp
    // Item.h
    DECLARE_LOG_CATEGORY_EXTERN(LogMyActor, Warning, All);

    virtual void PostInitializeComponents() override;
    virtual void BeginPlay() override;
    virtual void Destroyed() override;
    virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;

    // Item.cpp
    DEFINE_LOG_CATEGORY(LogMyActor);

    void AItem::PostInitializeComponents()
    {
        Super::PostInitializeComponents();
        UE_LOG(LogMyActor, Warning, TEXT("%s PostInitializeComponents"), *GetName());
    }

    void AItem::BeginPlay()
    {
        Super::BeginPlay();
        UE_LOG(LogMyActor, Warning, TEXT("%s BeginPlay"), *GetName());
    }

    // Destroyed, EndPlay 등 다른 함수도 유사하게 구현
    ```

3.  **호출 순서 확인**

      - 에디터 상단 메뉴의 `Window > Output Log`를 통해 로그 창을 연다.
      - 플레이 인 에디터(PIE)를 시작하면 생성자부터 `BeginPlay`까지의 로그가 순서대로 출력된다.
      - 플레이 도중 `Outliner`에서 액터를 직접 삭제하면 `Destroyed()`와 `EndPlay()`가 순서대로 호출되는 것을 확인할 수 있다.
      - PIE를 중지하면 `Destroyed()`는 호출되지 않고 `EndPlay()`만 호출된다.

## 느낀점

  - **초기화 로직의 분리**: 이전에는 모든 초기화 코드를 생성자나 `BeginPlay`에 넣으려고 했다. 하지만 `CreateDefaultSubobject`는 생성자에서, 컴포넌트 간의 연결은 `PostInitializeComponents`에서, 다른 액터와의 상호작용은 `BeginPlay`에서 처리해야 한다는 것을 명확히 알게 되었다. 각 함수의 호출 시점과 목적을 이해하고 코드를 분리하는 것이 안정적인 프로그램을 만드는 핵심이다.
  - **`EndPlay`의 중요성**: `Destroyed`는 수동으로 액터를 파괴할 때만 호출될 수 있어 신뢰성이 떨어진다. 반면 `EndPlay`는 레벨 전환이나 게임 종료 시에도 호출이 보장되므로, 동적으로 할당한 메모리나 타이머 해제 등 중요한 정리 작업은 `EndPlay`에 구현하는 것이 훨씬 안전하다는 것을 깨달았다.
  - **로그를 통한 흐름 파악**: 단순히 변수 값을 확인하는 용도를 넘어, `UE_LOG`를 라이프 사이클 함수에 적용하니 엔진 내부의 동작 흐름이 눈에 보여 매우 유용했다. 복잡한 시스템을 분석할 때 로그를 찍어보는 습관이 디버깅 효율을 크게 높여줄 것 같다.

-----

### 요약

| 함수명 | 호출 시점 | 주요 역할 | 주의사항 |
| :--- | :--- | :--- | :--- |
| **생성자** | C++ 객체 메모리 생성 시 | 컴포넌트 생성 (`CreateDefaultSubobject`), 기본 변수 초기화 | 월드 접근 불가 |
| **`PostInitializeComponents`** | 모든 컴포넌트 생성 완료 후 | 컴포넌트 간 참조 설정 등 의존성 처리 | - |
| **`BeginPlay`** | 게임 시작 또는 액터 스폰 시 | 다른 액터와 상호작용, 타이머/이벤트 바인딩 시작 | - |
| **`Tick`** | 매 프레임 | 실시간 로직 처리 | 성능 저하 유발 가능, 필요할 때만 활성화 |
| **`Destroyed`** | `Destroy()` 명시적 호출 시 | 수동 파괴 시 필요한 마지막 정리 작업 | 호출 보장 안 됨 (레벨 전환, 게임 종료 시) |
| **`EndPlay`** | 액터가 월드에서 제거될 때 | 타이머 해제, 데이터 저장 등 **안전한** 리소스 정리 | `Destroyed`보다 신뢰성 높음 |
