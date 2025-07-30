## 학습 키워드
- 데이터 중심 설계 (Data-Driven Design)
- UDataAsset
- 소켓 (Socket)
- 의존성 관리 (Dependency Management)
- 열거형 클래스 (Enum Class)
- 병렬 개발 (Parallel Development)
- 무기 부착물 시스템 (Weapon Attachment System)

<br/>

## 학습 내용
### 배운 개념 요약
- **데이터와 로직의 분리**: 게임 시스템을 확장 가능하고 유지보수하기 쉽게 만드는 핵심 원칙. `UDataAsset`을 사용하여 게임의 설정값(데이터)을 C++ 코드(로직)와 분리하는 방법을 배웠다. 이는 라이라(Lyra) 프로젝트의 핵심 아키텍처 철학이다.

- **UDataAsset (설계도)**: 순수한 데이터 컨테이너. 월드에 존재하거나 스스로 동작하지 않으며, 무기의 스탯, 외형, 부착 소켓 이름 등 각종 '정보'를 담아두는 '설계도' 역할을 한다.

- **Actor Component (조립공)**: 실제 '일'을 하는 로직. `UDataAsset`이라는 설계도를 읽어서, 게임 월드에 실제로 메시를 생성하고 소켓에 부착하는 등의 능동적인 작업을 수행한다. 데이터 애셋이 마법처럼 동작하는 것이 아니라, 이 로직 컴포넌트가 있기 때문에 가능한 것이었다.

- **Socket**: 스켈레탈 메시의 뼈에 부착하는 가상의 좌표. 아이템, 이펙트 등을 정확한 위치에 부착하기 위해 사용된다.

- **Enum Class**: `EWeaponType`, `EAttachmentSlot`처럼 정해진 목록을 정의하는 C++ 타입. `UENUM(BlueprintType)` 매크로와 함께 사용하면 블루프린트에서도 쓸 수 있으며, 타입 안정성과 가독성을 크게 향상시킨다.

- **의존성 관리**: 프로젝트가 커질수록 중요해지는 개념. `enum` 같은 공용 타입을 별도의 헤더 파일(`WeaponTypes.h`)로 분리해야 하는 이유를 이해했다. 특정 클래스에 `enum`을 종속시키면, 단순히 `enum` 정보만 필요한 다른 클래스에서 불필요한 헤더를 `#include`하게 되고, 이는 컴파일 시간 증가와 순환 참조 문제로 이어진다. 타입을 독립시켜 의존성 트리의 최하단에 두는 것이 좋은 설계이다.

<br/>

### 구현 과정
1.  **공용 타입 정의 (`WeaponTypes.h`)**:
    - `AActor`나 `UObject`를 상속받지 않는 순수 C++ 헤더 파일을 생성한다.
    - 여기에 `UENUM(BlueprintType)` 매크로를 사용하여 `EWeaponType`, `EAttachmentSlot` 같은 `enum class`들을 정의한다.
    - 이 파일은 프로젝트의 다른 모든 부분에서 자유롭게 `#include` 할 수 있는 '공용 자산'이 된다.

2.  **데이터 설계도 정의 (`UWeaponDataAsset.h`, `UWeaponAttachmentDataAsset.h`)**:
    - `UDataAsset`을 상속받는 클래스를 생성한다.
    - 무기 데이터를 담을 `UWeaponDataAsset`에는 무기 스탯, 외형 정보, 그리고 `#include "WeaponTypes.h"`를 통해 `EWeaponType` 변수 등을 추가한다.
    - 부착물 데이터를 담을 `UWeaponAttachmentDataAsset`에는 부착물 메시, 부착될 소켓 이름, 스탯 변경치 등의 정보를 추가한다.

3.  **로직 구현체 정의 (`UWeaponAttachmentComponent.h/.cpp`)**:
    - `UActorComponent`를 상속받는 '조립공' 컴포넌트를 생성한다.
    - 이 컴포넌트는 `UWeaponAttachmentDataAsset`을 입력으로 받는 함수(예: `EquipAttachment`)를 가진다.
    - 함수 내부에서는 입력받은 데이터 애셋의 정보를 읽어 `NewObject`로 `UStaticMeshComponent`를 생성하고, `AttachToComponent`를 호출하여 무기 메시의 지정된 소켓에 실제로 부착하는 로직을 구현한다.

4.  **최종 조립 (`AWeapon`)**:
    - 무기 액터 `AWeapon`은 `UWeaponDataAsset` 변수를 가져서 자신의 기본 정체성을 정의한다.
    - `AWeapon`은 `UWeaponAttachmentComponent`를 자신의 컴포넌트로 소유하여, 부착물 관련 모든 로직을 이 컴포넌트에 위임한다.

<br/>

---
## 느낀점
- 처음에는 `UDataAsset`만 만들면 어떻게 알아서 기능이 구현되는지 의아했지만, **데이터(설계도)와 로직(조립공)의 역할이 명확히 분리**되어 있다는 것을 깨달았다. 데이터 애셋은 단지 정보 묶음일 뿐이고, 그 정보를 해석해서 실제 동작을 수행하는 액터나 컴포넌트가 반드시 필요하다는 사실을 이해한 것이 가장 큰 수확이다.

- `enum`을 별도 파일로 분리하는 것이 처음엔 번거롭다고 생각했지만, **의존성 관리와 컴파일 시간 최적화**라는 장기적인 관점에서 왜 그것이 필수적인지 알게 되었다. 좋은 아키텍처는 당장의 편의성보다 미래의 확장성과 유지보수를 먼저 고려해야 한다는 것을 배웠다.

- 이 데이터 중심 설계 방식을 통해 개발자 간의 명확한 '계약'이 만들어진다는 점이 인상 깊었다. 데이터 애셋의 구조를 먼저 함께 정의하면, 각자 맡은 부분을 독립적으로 개발한 뒤 나중에 합칠 수 있으므로 진정한 의미의 병렬 개발이 가능해진다.
