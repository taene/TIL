## 학습 키워드

  - 리플렉션(Reflection)
  - UCLASS
  - UPROPERTY
  - UFUNCTION
  - C++/블루프린트 연동

<br/>

## 학습 내용

### 배운 개념 요약

  - **리플렉션 (Reflection) 시스템**

      - 언리얼 엔진이 C++ 클래스, 변수, 함수 등의 정보를 메타데이터로 저장하고, 이를 에디터나 블루프린트에서 활용할 수 있게 하는 핵심 기술이다.
      - C++ 코드를 "반사(Reflect)"하여 에디터와 블루프린트에서 직접 값을 수정하거나 함수를 호출할 수 있게 해준다.
      - 이를 통해 프로그래머가 C++로 만든 핵심 로직을 다른 팀원들이 에디터에서 직관적으로 조정하며 협업 효율을 극대화할 수 있다.

  - **C++과 블루프린트의 관계**

      - C++은 성능이 중요한 핵심 로직 구현에, 블루프린트는 빠른 프로토타이핑이나 시각적인 로직 작성에 사용된다.
      - 리플렉션 시스템은 이 둘을 잇는 다리 역할을 하며, 하이브리드 워크플로우를 가능하게 한다.

  - **주요 리플렉션 매크로**

      - `UCLASS()`: C++ 클래스를 언리얼 리플렉션 시스템에 등록한다.
          - `Blueprintable`: 해당 클래스를 블루프린트에서 상속할 수 있게 한다.
          - `BlueprintType`: 블루프린트에서 변수 타입으로 사용할 수 있게 한다.
      - `UPROPERTY()`: 멤버 변수를 리플렉션 시스템에 등록한다.
          - 첫번째 인자: 에디터 디테일 패널에서의 수정 가능 여부를 결정한다. (수정이 가능해지면 블루프린트 상에서 볼 수 있게 된다.)
            - `VisibleAnywhere`: 수정 불가능
            - `EditAnywhere`: 클래스 디폴트와 인스턴스 모두 수정 가능
            - `EditDefaultsOnly`: 클래스 디폴트만 수정 가능
            - `EditInstanceOnly`: 인스턴스만 수정 가능
          - 두번째 인자: 블루프린트에서 변수 값을 읽거나 쓸 수 있는지 결정한다.
            - `BlueprintReadOnly`: 읽기만 가능(Get)
            - `BlueprintReadWrite`: 읽고 쓰기가 가능(Get, Set)
          - 세번째 인자: 디테일 패널에서 변수를 그룹화하여 정리한다.
            - `Category`: `Category="Item|Components"`는 에디터의 카테고리에서 Item 상위 카테고리 안에 Components 하위 카테고리에 보이게 한다는 뜻이다.
      - `UFUNCTION()`: 멤버 함수를 리플렉션 시스템에 등록한다.
          - `BlueprintCallable`: 블루프린트에서 실행 핀이 있는 노드로 호출할 수 있게 한다.
          - `BlueprintPure`: 실행 핀 없이 값만 반환하는 순수(Pure) 노드로 만든다. (Getter 함수에 주로 사용)
          - `BlueprintImplementableEvent`: C++에서는 함수를 선언만 하고, 실제 구현은 블루프린트의 이벤트 그래프에서 하도록 한다.

<br/>

### 구현 과정

1.  **클래스 리플렉션 등록 (`UCLASS`)**

      - 헤더 파일 마지막에 `.generated.h` 파일을 포함하고, 클래스 선언부 위에 `UCLASS()` 매크로, 클래스 내부에 `GENERATED_BODY()` 매크로를 추가한다.

    ```cpp
    // Item.h
    #pragma once

    #include "CoreMinimal.h"
    #include "GameFramework/Actor.h"
    #include "Item.generated.h" // 항상 마지막에 위치해야 함

    UCLASS(Blueprintable, BlueprintType) // 블루프린트 상속 및 변수 타입으로 사용 가능
    class SPARTAPROJECT_API AItem : public AActor
    {
        GENERATED_BODY()
        // ...
    };
    ```

2.  **변수 리플렉션 등록 (`UPROPERTY`)**

      - 멤버 변수 위에 `UPROPERTY()` 매크로와 원하는 지정자를 추가하여 에디터 및 블루프린트 노출 방식을 설정한다.

    ```cpp
    // Item.h
    protected:
        // 에디터와 블루프린트에서 수정 가능한 Static Mesh 컴포넌트
        UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item|Components")
        UStaticMeshComponent* StaticMeshComp;

        // 클래스 블루프린트의 기본값에서만 수정 가능한 회전 속도 변수
        UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Item Properties")
        float RotationSpeed;
    ```

      - 위와 같이 설정하면 블루프린트 클래스의 '클래스 디폴트' 창에서 `RotationSpeed` 값을 직접 수정할 수 있다.

3.  **함수 리플렉션 등록 (`UFUNCTION`)**

      - 멤버 함수 선언부 위에 `UFUNCTION()` 매크로와 지정자를 추가하여 블루프린트에서의 사용 방식을 정의한다.

    ```cpp
    // Item.h
    public:
        // 블루프린트에서 호출 가능한 일반 함수
        UFUNCTION(BlueprintCallable, Category = "Item|Actions")
        void ResetActorPosition();

        // 블루프린트에서 값만 가져오는 순수 함수
        UFUNCTION(BlueprintPure, Category = "Item|Properties")
        float GetRotationSpeed() const;

        // C++에서 호출하고, 구현은 블루프린트에서 하는 이벤트
        UFUNCTION(BlueprintImplementableEvent, Category = "Item|Event")
        void OnItemPickedUp();
    ```

      - C++에서 `OnItemPickedUp()` 함수를 호출하면, 블루프린트에서 해당 이벤트에 연결된 노드들이 실행된다.

<br/>

## 느낀점

  - 언리얼의 리플렉션 시스템은 C++의 성능과 블루프린트의 편의성을 동시에 잡을 수 있게 해주는 강력한 기능임을 다시 한번 깨달았다.
  - 단순히 코드를 노출하는 것을 넘어, `UPROPERTY`와 `UFUNCTION`의 다양한 지정자를 통해 데이터의 접근 수준과 함수의 동작 방식을 세밀하게 제어할 수 있다는 점이 인상 깊었다.
  - 이를 통해 C++ 프로그래머는 견고한 백엔드 시스템을 구축하고, 기획자나 아티스트는 블루프린트와 에디터에서 안전하고 직관적으로 그 기능을 활용할 수 있는 협업 환경이 만들어진다. 앞으로는 각 지정자의 역할을 명확히 이해하고 목적에 맞게 사용하여 효율적인 개발 파이프라인을 설계해야겠다.

-----

### 요약

언리얼 엔진의 리플렉션 시스템은 `UCLASS`, `UPROPERTY`, `UFUNCTION` 매크로를 사용해 C++ 코드를 엔진과 블루프린트에 노출시키는 기술이다. 이를 통해 C++의 성능과 블루프린트의 빠른 개발 속도라는 장점을 모두 취하는 효율적인 하이브리드 개발이 가능하다. 각 매크로의 지정자를 잘 활용하면 노출 범위와 권한을 세밀하게 제어할 수 있어 협업 효율을 극대화할 수 있다.
