## Class UWorld::GetWorld()

- `UWorld`는 언리얼 엔진에서 모든 액터(Actor)가 존재하는 공간, 즉 **레벨(Level)을 표현하는 클래스**이다.
- `GetWorld()`는 현재 액터가 속해 있는 `UWorld` 객체에 대한 포인터를 반환하는 **함수**이다.

-----

### UWorld 클래스

- `UWorld`는 **게임의 월드를 담는 최상위 컨테이너**이다. 여기에는 현재 스트리밍된 레벨, 월드에 존재하는 모든 액터, 플레이어 컨트롤러, 게임 모드 등 월드와 관련된 모든 정보가 포함된다.
- `UWorld`: 게임의 레벨, 액터, 타이머, 네트워크 상태 등 전체적인 월드 상태를 관리하는 핵심 객체!

주요 역할은 다음과 같다.

  * **액터 관리**: 월드에 존재하는 모든 액터를 추적하고 관리한다.
  * **물리 시뮬레이션**: 월드 내의 물리 상호작용을 처리한다.
  * **시간 관리**: 게임의 시간, 딜레이, 타이머 등을 관리한다.
  * **스트리밍**: 레벨 스트리밍을 통해 거대한 월드를 효율적으로 관리한다.
  * **렌더링**: 월드에 있는 객체들을 화면에 그리는 렌더링 루프를 관리한다.

-----

### GetWorld() 함수

- 대부분의 언리얼 오브젝트(`UObject`)는 `GetWorld()` 멤버 함수를 가지고 있다. 이 함수는 해당 오브젝트가 속한 `UWorld`의 인스턴스를 반환한다. 예를 들어, 액터, 액터 컴포넌트, 플레이어 컨트롤러 등은 모두 자신의 월드를 알기 위해 이 함수를 호출할 수 있다.

- `GetWorld()`는 월드에 액터를 스폰(spawn)하거나, 타이머를 설정하거나, 라인 트레이스(line trace)를 수행하는 등 월드와 상호작용이 필요한 거의 모든 작업에 필수적이다.

-----

### 사용 예제

`GetWorld()`를 사용하는 가장 흔한 경우는 다음과 같다.

- 타이머(Timer) 관리 → GetWorld()->GetTimerManager()
- 월드에서 액터 스폰(SpawnActor) → GetWorld()->SpawnActor()
- 레벨 전환(Open Level) → GetWorld()->ServerTravel()
- 물리적 충돌 등에서 시간 관련 연산 → GetWorld()->GetDeltaSeconds()

#### 1. 액터 스폰하기

새로운 액터를 월드에 생성할 때 사용한다.

```cpp
// MyActor.cpp

#include "MyProjectile.h" // 스폰할 액터의 헤더 파일

void AMyActor::SpawnProjectile()
{
    // GetWorld()를 호출하여 현재 월드를 가져온다.
    UWorld* World = GetWorld();
    if (World)
    {
        // 스폰할 위치와 회전값 설정
        FVector SpawnLocation = GetActorLocation() + GetActorForwardVector() * 100.0f;
        FRotator SpawnRotation = GetActorRotation();

        // 월드에 AMyProjectile 액터를 스폰한다.
        World->SpawnActor<AMyProjectile>(SpawnLocation, SpawnRotation);
    }
}
```

#### 2. 타이머 사용하기

특정 시간 후에 함수를 호출하는 타이머를 설정할 때 사용한다.

```cpp
// MyPlayerController.cpp

void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    FTimerHandle TimerHandle;
    // 5초 후에 MyDelayedFunction 함수를 한 번 호출하는 타이머 설정
    GetWorld()->GetTimerManager().SetTimer(TimerHandle, this, &AMyPlayerController::MyDelayedFunction, 5.0f, false);
}

void AMyPlayerController::MyDelayedFunction()
{
    // 5초 후에 실행될 내용
}
```

#### 3. 라인 트레이스 (레이캐스트)

월드에 보이지 않는 선을 쏴서 충돌하는 액터를 감지할 때 사용한다.

```cpp
// MyCharacter.cpp

void AMyCharacter::CheckForInteractables()
{
    FVector Start = GetActorLocation();
    FVector End = Start + GetActorForwardVector() * 200.0f; // 전방 200cm
    FHitResult HitResult;
    FCollisionQueryParams Params;
    Params.AddIgnoredActor(this); // 자기 자신은 충돌 검사에서 제외

    // 월드에 라인 트레이스를 수행한다.
    if (GetWorld()->LineTraceSingleByChannel(HitResult, Start, End, ECC_Visibility, Params))
    {
        // 무언가에 맞았다면 HitResult에 정보가 담긴다.
        AActor* HitActor = HitResult.GetActor();
        if (HitActor)
        {
            // 맞은 액터에 대한 처리
        }
    }
}
```

`GetWorld()`는 월드에 대한 거의 모든 접근에 필요한 **관문**과 같은 역할을 한다. 따라서 언리얼 C++ 프로그래밍에서 매우 중요하고 빈번하게 사용되는 함수이다.

-----

**요약**

| 구분 | 설명 |
| :--- | :--- |
| **`UWorld`** | 레벨, 액터 등 게임의 모든 것을 담는 공간이자 클래스. |
| **`GetWorld()`** | 현재 오브젝트가 속한 `UWorld`의 인스턴스를 가져오는 함수. |
| **주요 용도** | 액터 스폰, 타이머 설정, 라인 트레이스 등 월드와 상호작용하는 작업. |
