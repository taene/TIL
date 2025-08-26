## 학습 키워드

- 네트워크 롤(NetRole)
- 로컬 롤(Local Role)
- 리모트 롤(Remote Role)
- Authority
- Autonomous Proxy
- Simulated Proxy
- RPC(Remote Procedure Call)

<br/>

## 학습 내용

### 배운 개념 요약

- 언리얼 엔진의 멀티플레이어 환경에서 액터를 구분하는 핵심은'**제어의 주체**'가 누구인지를 파악하는 것이다. 서버에 존재하는 액터는 모두 로컬 롤로 `ROLE_Authority`를 가지기 때문에 이것만으로는 구분이 불가능하다. 따라서 **리모트 롤**(Remote Role)을 함께 확인하여 제어 관계를 명확히 정의한다.

  * **서버에 접속한 플레이어 캐릭터**:

      * **로컬 롤**: `ROLE_Authority` (서버가 최종 권위를 가짐)
      * **리모트 롤**: `ROLE_AutonomousProxy` (원격 클라이언트가 제어권을 위임받았음을 의미)
      * 서버는 이 조합을 통해 해당 액터가 '플레이어에 의해 제어되는 액터'임을 인지한다.

  * **서버에서 직접 스폰한 AI 캐릭터**:

      * **로컬 롤**: `ROLE_Authority` (서버가 최종 권위를 가짐)
      * **리모트 롤**: `ROLE_SimulatedProxy` (원격 클라이언트는 서버의 정보를 받아 시뮬레이션만 할 뿐, 제어권이 없음을 의미)
      * 서버는 이 조합을 통해 해당 액터가 '서버 자신에 의해 제어되는 액터'임을 인지한다.

- 이러한 명확한 구분은 **클라이언트 RPC**를 구현하는 데 필수적이다. 클라이언트 RPC는 서버가 특정 '소유자(Owner)' 클라이언트에게만 명령을 내리는 기능인데, 리모트 롤이 `AutonomousProxy`인 액터만이 명확한 소유자 관계를 가지므로 RPC를 수신할 수 있다.

<br/>

### 구현 과정

플레이어 캐릭터의 체력이 낮아졌을 때, 해당 플레이어의 클라이언트에만 경고 UI를 띄우는 클라이언트 RPC 구현 과정이다.

1.  **`.h` (헤더 파일)에 클라이언트 RPC 함수를 선언한다.**
    `UFUNCTION` 매크로에 `Client` 키워드를 추가하여 이 함수가 클라이언트에서 실행될 RPC임을 명시한다.

    ```cpp
    // PlayerCharacter.h

    #pragma once

    #include "CoreMinimal.h"
    #include "GameFramework/Character.h"
    #include "PlayerCharacter.generated.h"

    UCLASS()
    class MYPROJECT_API APlayerCharacter : public ACharacter
    {
        GENERATED_BODY()

    public:
        // ... (생성자 및 다른 함수들)

        // 서버에서 호출되어 소유자 클라이언트에서 실행될 RPC 함수
        UFUNCTION(Client, Reliable)
        void Client_ShowLowHealthWarning();
    };
    ```

2.  **`.cpp` (소스 파일)에 서버 로직과 클라이언트 실행 로직을 구현한다.**
    서버의 권한을 가진 액터(`GetLocalRole() == ROLE_Authority`)에서 클라이언트 RPC를 호출한다. `_Implementation` 접미사가 붙은 함수에 클라이언트에서 실제 실행될 내용을 작성한다.

    ```cpp
    // PlayerCharacter.cpp

    #include "PlayerCharacter.h"

    void APlayerCharacter::TakeDamage(/*...*/>)
    {
        // ... 데미지 처리 로직 ...

        // 이 로직은 서버에서만 실행되어야 한다.
        if (GetLocalRole() == ROLE_Authority)
        {
            if (Health < LowHealthThreshold)
            {
                // 서버가 소유자 클라이언트에게 RPC를 호출한다.
                // 엔진은 이 캐릭터의 리모트 롤이 AutonomousProxy임을 알고
                // 해당 클라이언트로 이 호출을 전달한다.
                Client_ShowLowHealthWarning();
            }
        }
    }

    // 클라이언트 RPC의 실제 구현부. 소유자 클라이언트에서만 실행된다.
    void APlayerCharacter::Client_ShowLowHealthWarning_Implementation()
    {
        // 여기에 경고 UI를 화면에 표시하는 코드를 구현한다.
        // 이 코드가 AI 캐릭터에 있었다면, 호출될 클라이언트가 없으므로 무시된다.
        ShowWarningUI();
    }
    ```

<br/>

## 느낀점

- 처음에는 서버에 있는 액터는 모두 `Authority`를 가지는데, 어떻게 플레이어와 AI를 구분해서 플레이어에게만 RPC를 보낼 수 있는지 의문이었다. 로컬 롤만으로는 제어의 주체를 파악할 수 없다는 한계를 깨달았다.

- 핵심은 **리모트 롤**에 있었다. 리모트 롤이 액터와 원격 연결 간의 '제어 관계'를 정의해주는 crucial 한 정보라는 것을 알게 되었다. `Authority`와 `AutonomousProxy`의 조합을 통해 비로소 '주인 있는 액터'를 식별할 수 있고, 이것이 클라이언트 RPC가 정확한 대상으로 전달될 수 있는 근간이 된다는 사실을 이해했다.

- 단순히 역할을 외우는 것을 넘어, 이 두 가지 롤의 조합이 왜 필요하고 어떻게 상호작용하는지 이해하니 네트워크 동작 원리가 훨씬 명확하게 보이기 시작한다. 이 기본 원리가 라이라의 게임플레이 어빌리티 시스템 같은 복잡한 구조의 기반이 된다고 생각하니, 기초의 중요성을 다시 한번 실감하게 된다.
