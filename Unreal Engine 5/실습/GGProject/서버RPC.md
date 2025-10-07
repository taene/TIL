## 학습 키워드 <br>

  * 언리얼 엔진 (Unreal Engine)
  * 변수 리플리케이션 (Variable Replication)
  * 서버 권한 모델 (Server Authoritative Model)
  * 서버 RPC (Server Remote Procedure Call)
  * RepNotify (OnRep)
  * 게임플레이 어빌리티 시스템 (Gameplay Ability System, GAS)

<br>

## 학습 내용

### 배운 개념 요약 <br>

언리얼 엔진의 변수 리플리케이션은 **서버에서 클라이언트로 흐르는 단방향 통신**을 기본 원칙으로 한다. 이는 모든 게임 로직의 최종 권한을 서버가 갖는 '서버 권한 모델'에 기반하며, 클라이언트의 메모리 조작을 통한 치트 행위를 원천적으로 방지하고 모든 플레이어의 게임 상태를 일관되게 동기화하기 위함이다.

클라이언트가 서버의 상태를 변경해야 할 경우, 변수를 직접 수정하는 것이 아니라 서버 RPC (Remote Procedure Call)를 통해 서버에 '요청'을 보내야 한다. 서버는 이 요청을 받아 유효성을 검증한 뒤, 자신의 권위 있는(Authoritative) 버전의 변수 값을 변경한다. 이 변경된 값은 리플리케이션 시스템을 통해 다시 모든 클라이언트로 전파된다.

이때, 클라이언트 측에서는 `ReplicatedUsing` 키워드로 지정된 **OnRep (RepNotify)** 함수가 자동으로 호출된다. 이 함수는 서버로부터 새로운 값을 수신했을 때 UI를 갱신하거나 이펙트를 재생하는 등, 클라이언트에서만 필요한 시각적/청각적 피드백을 처리하는 데 사용한다.

라이라(Lyra)와 같은 AAA급 프로젝트에서는 이러한 통신 흐름을 게임플레이 어빌리티 시스템(GAS)을 통해 더욱 체계적으로 관리한다. 개별 RPC 대신 어빌리티 활성화를 요청하고, 서버의 어빌리티 시스템이 유효성 검사부터 어트리뷰트(리플리케이트되는 변수) 변경까지의 모든 과정을 데이터 기반으로 처리함으로써 확장성과 유지보수성을 극대화한다.

<br>

### 구현 과정 <br>

클라이언트의 요청으로 서버의 리플리케이트 변수를 변경하고, 이를 다시 클라이언트에서 피드백받는 과정은 다음과 같이 구현한다.

1.  **변수 선언 (.h)**

      * `UPROPERTY`에 `ReplicatedUsing` 키워드를 사용하여 변수를 선언하고, 값이 복제될 때 호출될 OnRep 함수를 지정한다.
      * 클라이언트가 호출할 `UFUNCTION`에 `Server`, `Reliable`, `WithValidation` 키워드를 붙여 서버 RPC 함수를 선언한다.

    <!-- end list -->

    ```cpp
    // MyCharacter.h

    UPROPERTY(ReplicatedUsing = OnRep_CurrentHealth)
    float CurrentHealth;

    UFUNCTION()
    void OnRep_CurrentHealth();

    UFUNCTION(Server, Reliable, WithValidation)
    void Server_RequestHealthChange(float Delta);
    ```

2.  **리플리케이션 등록 (.cpp)**

      * `GetLifetimeReplicatedProps` 함수를 오버라이드하여 `DOREPLIFETIME` 매크로로 해당 변수를 리플리케이션 목록에 등록한다.

    <!-- end list -->

    ```cpp
    // MyCharacter.cpp

    #include "Net/UnrealNetwork.h"

    void AMyCharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
    {
        Super::GetLifetimeReplicatedProps(OutLifetimeProps);
        DOREPLIFETIME(AMyCharacter, CurrentHealth);
    }
    ```

3.  **서버 RPC 구현 (.cpp)**

      * `_Validate` 함수를 구현하여 서버에 도달한 요청의 유효성을 검사한다. 여기서 `false`를 반환하면 `_Implementation` 함수는 실행되지 않는다.
      * `_Implementation` 함수에 서버에서 실행될 실제 로직을 구현한다. 서버는 여기서 변수 값을 직접 변경하며, 이 변경은 자동으로 모든 클라이언트에 전파된다.

    <!-- end list -->

    ```cpp
    // MyCharacter.cpp

    bool AMyCharacter::Server_RequestHealthChange_Validate(float Delta)
    {
        // 비정상적인 값 요청을 여기서 차단할 수 있다.
        return true;
    }

    void AMyCharacter::Server_RequestHealthChange_Implementation(float Delta)
    {
        // 서버만이 변수를 수정할 권한을 가진다.
        if (GetLocalRole() == ROLE_Authority)
        {
            CurrentHealth = FMath::Clamp(CurrentHealth + Delta, 0.0f, 100.0f);
        }
    }
    ```

4.  **OnRep 함수 구현 (.cpp)**

      * `OnRep_` 함수 내부에 클라이언트에서만 실행되어야 할 로직(UI 업데이트, 사운드 재생 등)을 구현한다. 이 함수는 서버에서 변수 값이 변경되어 클라이언트로 성공적으로 복제되었을 때만 호출된다.

    <!-- end list -->

    ```cpp
    // MyCharacter.cpp

    void AMyCharacter::OnRep_CurrentHealth()
    {
        // 이 코드는 서버에서는 실행되지 않는다.
        // 체력 바 UI를 갱신하는 등의 시각적 피드백을 처리한다.
    }
    ```

<br>

## 느낀점 <br>

'변수 리플리케이션은 서버에서 클라이언트로만 흐른다'는 단순한 규칙이 멀티플레이어 게임의 공정성과 안정성을 지키는 핵심적인 설계 철학임을 깨달았다. 클라이언트의 요청(RPC), 서버의 처리 및 전파(Replication), 그리고 클라이언트의 피드백(OnRep)으로 이어지는 명확한 역할 분리는 서버 권한 모델을 어떻게 코드로 구현하는지 직관적으로 보여준다. 과거에는 클라이언트의 변수가 왜 서버에 반영되지 않는지 의문을 가졌지만, 이제는 이것이 치트 방지와 동기화를 위한 필수적인 구조임을 이해하게 되었다.

특히 라이라 프로젝트가 이를 GAS로 추상화하여 관리하는 방식은 인상 깊다. 단순히 기능 구현을 넘어, 수많은 스킬과 아이템, 상태 변화가 상호작용하는 복잡한 시스템을 어떻게 체계적이고 데이터 중심으로 설계해야 하는지에 대한 깊은 통찰을 제공한다. 이는 단순한 코딩 기술을 넘어 AAA급 게임 개발의 아키텍처를 고민하는 시야를 넓혀주는 중요한 학습 경험이다.
