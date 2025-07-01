## 학습 키워드

  - `Transform`
  - `FTransform`
  - `Tick`
  - `DeltaTime`
  - `BeginPlay`
  - 프레임 독립성 (Frame-Rate Independence)
  - `AddActorLocalRotation`


## 학습 내용

### 배운 개념 요약

  - **Transform**: 액터의 월드 내 **위치Location**, **회전Rotation**, **크기Scale**를 나타내는 세 가지 속성의 집합이다. `FTransform` 구조체는 이 세 가지 요소를 하나로 묶어 관리한다.
  - **좌표계**: 절대적인 기준인 **월드 좌표계 World Space**와 부모를 기준으로 하는 상대적인 **로컬 좌표계 Local Space**가 있다.
  - **Tick 함수**: 매 프레임마다 특정 로직을 실행시키기 위해 호출되는 함수다. 사용하려면 생성자에서 `PrimaryActorTick.bCanEverTick`을 `true`로 설정해야 한다.
  - **DeltaTime**: `Tick` 함수에 인자로 전달되며, 바로 이전 프레임부터 현재 프레임까지 걸린 시간을 초 단위로 나타낸다.
  - **프레임 독립적인 움직임**: 게임의 프레임 속도(FPS)와 상관없이 일관된 속도를 유지하는 움직임을 의미한다. 이동 거리나 회전 각도에 `DeltaTime`을 곱해주면 구현할 수 있다. 예를 들어 초당 100 유닛을 움직이려면, 매 프레임 `100.0f * DeltaTime` 만큼 이동시키면 된다.

### 구현 과정

`Tick`과 `DeltaTime`을 이용해 액터를 1초에 `90`도씩 Y축(Yaw) 기준으로 회전시키는 과정이다.

1.  **Tick 활성화 및 변수 초기화**: `AItem` 액터의 생성자에서 `Tick` 함수를 활성화하고, 회전 속도(`RotationSpeed`) 변수를 초기화한다.

    ```cpp
    // Item.h
    UPROPERTY()
    float RotationSpeed;

    // Item.cpp
    AItem::AItem()
    {
        // Tick 함수를 켠다.
        PrimaryActorTick.bCanEverTick = true;
        // 기본 회전 속도 (1초에 90도 회전)
        RotationSpeed = 90.0f;
    }
    ```

2.  **Tick 함수 로직 구현**: `Tick` 함수 내에서 `DeltaTime`을 이용해 프레임에 독립적인 회전 값을 계산하고, `AddActorLocalRotation` 함수로 현재 회전 값에 더해준다.

    ```cpp
    // Item.cpp
    void AItem::Tick(float DeltaTime)
    {
        Super::Tick(DeltaTime);

        // 초당 RotationSpeed만큼, 한 프레임당 (RotationSpeed * DeltaTime)만큼 회전
        AddActorLocalRotation(FRotator(0.0f, RotationSpeed * DeltaTime, 0.0f));
    }
    ```

      - `AddActorLocalRotation`은 액터의 로컬 축을 기준으로 회전시킨다.
      - 월드 축 기준으로 회전시키려면 `AddActorWorldRotation`을 사용하면 된다.


## 느낀점

  - `DeltaTime`의 중요성을 명확히 알게 됐다. FPS가 다른 환경에서도 모든 유저에게 동일한 게임 경험을 제공하려면, 시간에 기반한 로직 작성이 필수적이라는 것을 깨달았다.
  - `Tick`은 매 프레임 호출되므로 성능에 민감할 수밖에 없다. 불필요한 경우 반드시 `bCanEverTick = false;`로 비활성화하여 최적화를 챙기는 습관을 들여야겠다.
  - `BeginPlay()`는 초기 상태 설정에, `Tick()`은 지속적인 상태 변화에 사용하는 역할 분담이 확실해서 코드를 체계적으로 관리하기 좋다고 느꼈다.

-----

### 요약

| 개념 | 설명 |
| :--- | :--- |
| **Transform** | 액터의 위치, 회전, 크기를 나타내는 속성. |
| **Tick()** | 매 프레임 호출되는 함수로, 지속적인 업데이트 로직에 사용. |
| **DeltaTime** | 이전 프레임과 현재 프레임 사이의 시간 간격(초). |
| **프레임 독립성** | `DeltaTime`을 곱하여 FPS에 관계없이 일관된 속도를 보장하는 것. |
