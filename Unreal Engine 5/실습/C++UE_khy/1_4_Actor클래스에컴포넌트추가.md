## 학습 키워드

  - Actor 클래스 구조
  - 컴포넌트 (Component)
  - 루트 컴포넌트 (Root Component)
  - `USceneComponent`
  - `UStaticMeshComponent`
  - `CreateDefaultSubobject`
  - `SetRootComponent`
  - `SetupAttachment`
  - `ConstructorHelpers::FObjectFinder`

## 학습 내용

### 배운 개념 요약

  - **Actor와 컴포넌트**: Actor는 월드에 배치될 수 있는 기본 단위이며, 그 자체로는 아무런 기능이 없다.  여기에 '컴포넌트'라는 부품을 붙여 시각적 형태, 소리, 충돌 등 다양한 기능을 구현한다.
  - **컴포넌트 계층 구조**: 모든 Actor는 반드시 하나의 루트 컴포넌트(Root Component)를 가져야 한다. 루트 컴포넌트는 해당 Actor의 월드 내 위치, 회전, 크기(Transform)를 결정하며, 다른 모든 컴포넌트들은 이 루트에 계층적으로 연결(Attach)된다.
  - **주요 컴포넌트**:
      - `USceneComponent`: 위치, 회전, 크기 정보만 가진 채 눈에 보이지 않는 컴포넌트. 다른 컴포넌트들을 계층적으로 구성하기 위한 기준점 역할을 한다.
      - `UStaticMeshComponent`: 애니메이션이 없는 3D 모델(Static Mesh)을 화면에 렌더링하는 역할을 한다.
  - **C++에서의 에셋 할당**: 생성자 내에서 `ConstructorHelpers::FObjectFinder`를 사용하면, 콘텐츠 브라우저의 에셋 경로를 이용해 Static Mesh나 Material 같은 리소스를 직접 로드하고 컴포넌트에 할당할 수 있다.

### 구현 과정

1.  **컴포넌트 변수 선언**: Actor의 헤더 파일(.h)에 `USceneComponent`와 `UStaticMeshComponent` 포인터 변수를 선언한다.

    ```cpp
    // Item.h

    UPROPERTY()
    TObjectPtr<USceneComponent> SceneRoot;

    UPROPERTY()
    TObjectPtr<UStaticMeshComponent> StaticMeshComp;
    ```

2.  **컴포넌트 생성 및 초기화**: Actor의 생성자(.cpp)에서 컴포넌트들을 생성하고 계층 구조를 설정한다.

    ```cpp
    // Item.cpp

    AItem::AItem()
    {
        // 1. SceneComponent를 생성하고 RootComponent로 설정
        SceneRoot = CreateDefaultSubobject<USceneComponent>(TEXT("SceneRoot"));
        SetRootComponent(SceneRoot);

        // 2. StaticMeshComponent를 생성하고 RootComponent에 Attach
        StaticMeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("StaticMesh"));
        StaticMeshComp->SetupAttachment(SceneRoot);
    }
    ```

3.  **에셋 로드 및 할당**: `ConstructorHelpers::FObjectFinder`를 사용하여 Static Mesh와 Material 에셋을 로드하고, `SetStaticMesh`, `SetMaterial` 함수로 컴포넌트에 적용한다.

    ```cpp
    // Item.cpp (생성자 내부)

    // Static Mesh 로드
    static ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset(TEXT("/Game/Resources/Props/SM_Chair.SM_Chair"));
    if (MeshAsset.Succeeded())
    {
        StaticMeshComp->SetStaticMesh(MeshAsset.Object);
    }

    // Material 로드
    static ConstructorHelpers::FObjectFinder<UMaterial> MaterialAsset(TEXT("/Game/Resources/Materials/M_Metal_Gold.M_Metal_Gold"));
    if (MaterialAsset.Succeeded())
    {
        StaticMeshComp->SetMaterial(0, MaterialAsset.Object);
    }
    ```

## 느낀점

  - Actor와 Component의 관계는 언리얼 엔진 객체지향 설계의 핵심이다. Actor라는 뼈대에 Component라는 장기를 조립해 하나의 완전한 개체를 만드는 방식은 매우 직관적이고 확장성이 좋다.
  - C++ 코드 레벨에서 컴포넌트를 생성하고, 계층 구조를 설정하고, 에셋까지 할당하는 전체 과정을 직접 다뤄봄으로써 Actor가 어떻게 구성되는지 근본적인 원리를 이해할 수 있었다.
  - `ConstructorHelpers::FObjectFinder`는 경로를 통해 에셋을 직접 참조하는 방식으로, 특정 에셋이 반드시 필요한 경우 유용하다. 하지만 경로가 바뀌거나 에셋이 삭제되면 문제가 생길 수 있으므로, 유연한 관리가 필요할 땐 `UPROPERTY`를 사용해 에디터에서 에셋을 할당하는 방식이 더 효율적일 것 같다.

-----

### 요약

| 개념 | 설명 |
| :--- | :--- |
| **Actor와 Component** | Actor는 컨테이너, Component는 기능 부품이다.  |
| **Component 계층** | 모든 Actor는 Transform을 제공하는 `RootComponent`를 필수로 가지며, 다른 Component들은 여기에 붙는다.|
| **C++ 구현 흐름** | `변수 선언(.h)` -\> `CreateDefaultSubobject`로 생성(.cpp) -\> `SetRootComponent`/`SetupAttachment`로 조립(.cpp) 순으로 진행된다. |
| **에셋 할당** | 생성자 내에서 `ConstructorHelpers::FObjectFinder`를 사용해 C++ 코드로 직접 에셋을 로드하고 설정할 수 있다.|
