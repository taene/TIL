## 어빌리티 시스템 및 관련 구현 개요
이 문서는 제공된 소스를 바탕으로 어빌리티 시스템의 주요 개념, 구현 방법, 그리고 중요한 설계 원칙을 상세히 설명합니다. 특히 언리얼 엔진의 게임플레이 어빌리티 시스템을 라이라(Lyra) 프로젝트에서 어떻게 활용하고 확장하는지에 초점을 맞춥니다.

### 1. 어빌리티 시스템의 핵심 개념 및 구조
> 라이라 프로젝트에서 어빌리티 시스템은 단순히 게임플레이 어빌리티 시스템을 사용하는 것을 넘어, **내부적으로 한 번 더 래핑(wrapping)하여 사용 편의성을 높이고 있습니다.**

- 기본 플러그인 및 모듈:
  - 어빌리티 시스템을 사용하려면 GameplayAbilities 플러그인이 활성화되어야 합니다.
  - 내부적으로 GameplayAbilities와 GameplayTasks 두 가지 모듈이 등록되어야 하는데, 이 둘은 "**매우 의존적**(dependent)"이며 "**하나의 세트**"로 간주됩니다. GameplayAbilities 내에서 GameplayTasks 기능이 많이 사용되기 때문입니다.
- 핵심 컴포넌트:
  - **어빌리티 시스템 컴포넌트 (AbilitySystemComponent):** 모든 능력을 부여하고 제거하며 관리하는 "관리자" 역할을 합니다. 이 컴포넌트가 있어야만 능력을 허용할 수 있습니다. 라이라에서는 원본을 그대로 사용하지 않고 한 번 더 래핑하여 사용하고 있으며, 이는 **"확실히 편하다"**고 언급됩니다.
  - **게임플레이 어빌리티 (GameplayAbility):** 실제 능력 자체를 정의하는 클래스입니다. 라이라에서는 이를 상속받아 HKGameplayAbility를 만듭니다.

<br/>

### 2. 어빌리티 시스템 컴포넌트의 부착 위치 결정 (가장 중요한 설계 결정)
> 어빌리티 시스템 컴포넌트를 어디에 부착할 것인지는 매우 중요한 설계 결정입니다. 모든 캐릭터(플레이어, 몬스터 등)는 어빌리티 시스템 컴포넌트를 "**무조건 가지고 있어야 합니다**."   
> 그 이유는 게임 이펙트를 통해 데미지를 가하거나 어트리뷰트 값을 수정하는 등의 기능이 어빌리티 시스템 내에서 관리되기 때문입니다. 몬스터에 이 컴포넌트가 없으면 데미지를 가할 수 없습니다.

- 고려 사항:
GameMode: 게임당 하나만 존재하므로 적합하지 않습니다.
Character: 싱글 게임이고 플레이어가 한 번 결정되면 바뀌지 않는다는 보장이 있다면 캐릭터에 부착해도 문제가 없습니다. 그러나 캐릭터가 변경될 경우, 부여된 스킬 관리가 복잡해질 수 있습니다.
Controller: 몬스터 등 플레이어가 아닌 엔티티는 컨트롤러를 사용하지 않으므로 적합하지 않습니다.
- 라이라의 선택: **PlayerState** (라이라는 "**데디케이티드 게임 기반**"이므로 어빌리티 시스템 컴포넌트를 PlayerState에 부착합니다.)
  - 이유: 네트워크 게임에서 컨트롤러는 서버에만 존재하며, 각 클라이언트에게 직접 원본을 넘겨줄 수 없습니다.
  - PlayerState는 PlayerController에 있는 정보를 기록하고 복사되어 다른 클라이언트에 넘겨줄 수 있는 "복사본" 개념입니다.
  - HP, MP 등 모든 플레이어의 상태 정보가 어빌리티 시스템에서 관리되기 때문에, 모든 플레이어가 서로의 어빌리티 시스템 정보를 알아야 합니다. PlayerState는 이러한 "**상호 인지**"를 가능하게 합니다.
  - GameMode에 있는 게임 정보를 GameState가 기록하여 넘겨주는 것과 유사한 개념입니다.

<br/>

### 3. 어빌리티 시스템 컴포넌트 초기화 과정
> PlayerState에 어빌리티 시스템 컴포넌트를 생성하고 초기화하는 과정은 다음과 같습니다.

- 생성: PlayerState의 생성자 단계에서 CreateDefaultSubobject를 이용해 어빌리티 시스템 컴포넌트를 생성합니다.
- 초기 설정 (InitAbilityActorInfo):
  - 어빌리티 시스템은 생성 후 "누가 소유자이고 아바타가 무엇인지" 반드시 결정해 줘야 합니다. 이 설정은 InitAbilityActorInfo 함수를 통해 이루어집니다.
  - 오너 (Owner): 어빌리티 시스템 컴포넌트 자체를 소유하는 객체입니다. PlayerState에 부착했으므로 초기에는 PlayerState (즉, this)가 오너가 됩니다.
  - 아바타 (Avatar): 실제로 능력을 사용하는 액터입니다. PlayerState가 직접 능력을 사용하는 것이 아니므로, 초기 PostInitializeComponents 단계에서는 아직 폰이 Possess되지 않은 상태이므로 nullptr로 임시 설정됩니다. 이는 나중에 폰이 완전히 준비된 시점에 재설정됩니다.
  - 캐릭터에 부착할 경우: 오너와 아바타 모두 캐릭터(this, this)로 설정될 수 있습니다.
- 최종 초기화: HeroComponent 활용
  - 폰이 완전히 Possess되고 모든 데이터가 준비된 시점(데이터 이니셜라이즈 단계)은 HeroComponent의 HandleChangeInitState 함수에서 보장됩니다.
  - 이 시점에서 PhoneExtensionComponent를 통해 InitializeAbilitySystem 함수가 실행됩니다.
    - 역할: 어빌리티 시스템을 PhoneExtensionComponent에 캐싱합니다. (향후 사용 편의성을 위함)
  - InitAbilityActorInfo를 통해 어빌리티 시스템의 아바타 액터 정보를 재설정합니다. 이 시점에서는 폰이 Possess되어 있으므로 PlayerState가 오너가 되고, PhoneExtensionComponent의 오너인 **폰(실제 캐릭터)**이 아바타로 설정됩니다.
  - UninitializeAbilitySystem은 어빌리티 시스템 컴포넌트가 존재할 경우 단순히 nullptr로 설정하여 제거하는 간단한 로직입니다.

<br/>

### 4. 어빌리티 셋 (AbilitySet) 및 능력 부여 방식
- 어빌리티 셋의 역할:
  - 라이라에서는 능력을 **"카테고리로 구분"**하기 위해 AbilitySet을 사용합니다. 전사 스킬, 궁수 스킬 등을 묶어서 관리할 수 있습니다.
  - AbilitySet은 PrimaryDataAsset으로 구성되며, FHKAbilitySet 배열을 내부에 포함합니다.
  - FHKAbilitySet은 어빌리티(GameplayAbility)와 인풋 태그(Input Tag)를 매칭시켜 놓은 것이 핵심입니다. 이는 InputConfig가 액션과 인풋 태그를 매칭시키는 것과 유사합니다.
  - 이를 통해 어빌리티 시스템은 태그를 이용하여 원하는 어빌리티를 쉽게 가져올 수 있습니다. (예: "어택" 태그를 통해 공격 어빌리티를 찾음)
  - 내부적으로 어빌리티 레벨을 관리할 수 있으나, 라이라에서는 현재 사용되지 않습니다.
- 어빌리티 셋 부착 위치: PawnData
  - AbilitySet은 PawnData에서 관리됩니다. PawnData는 사용할 폰의 종합적인 정보를 담고 있으며, 어떤 능력을 사용할 것인지까지 결정합니다.
- 어빌리티 부여 시점: PlayerState::SetPawnData
  - PawnData에 있는 AbilitySet의 능력들은 PlayerState에서 SetPawnData 함수가 호출되는 시점에 부여됩니다.
  - PlayerState가 PawnData를 캐싱하는 핵심적인 이유 중 하나가 바로 이 어빌리티 시스템 때문입니다. SetPawnData 시점에 PawnData를 캐싱함과 동시에 AbilitySet에 있는 모든 능력을 가져와 어빌리티 시스템에 부여합니다.
- 능력 부여 로직: GiveToAbilitySystem 함수
  - 라이라에서는 AbilitySystemComponent의 GiveAbility 함수를 직접 호출하는 대신, AbilitySet 내부에 구현된 GiveToAbilitySystem 함수를 사용합니다. 이는 **"편리하게 사용하기 위함"**입니다.
  - 이 함수는 AbilitySet 내의 GrantedGameplayAbilities 배열을 순회하며 각 어빌리티를 어빌리티 시스템에 부여합니다.
    - CDO (Class Default Object) 형태 활용:GiveToAbilitySystem은 어빌리티를 CDO 형태로 가져옵니다. 이는 UObject 상속 객체의 생성(가비지 컬렉션 대상)을 피하고 성능 최적화를 위함입니다.
- 어빌리티는 크게 세 가지 인스턴스 모드를 가집니다.
  - Non-Instanced (CDO): 아예 생성하지 않고 CDO 형태로만 유지. 가장 가볍지만 값 수정이 어려움.
  - Instanced Per-Actor: 액터별로 한 번씩 생성. 라이라에서 주로 사용되는 방식.
  - Instanced Per-Execution: 스킬 사용 시마다 생성. 가장 무거움.
  - 라이라와 같은 FPS 게임에서는 데미지 값이 캐릭터마다 크게 변할 일이 없으므로 CDO 형태를 사용하여 한 번 더 최적화합니다.
    - FGameplayAbilitySpec (SPC) 사용:CDO 형태로 가져온 어빌리티를 **FGameplayAbilitySpec (SPC)**으로 묶어서 사용합니다.
    - SPC는 어빌리티 자체 외에 "제공자(Source Object)" 및 **"다이내믹 어빌리티 태그(Dynamic Ability Tags)"**와 같은 추가 정보를 설정할 수 있게 합니다. 이를 통해 특정 태그와 어빌리티를 매핑할 수 있습니다.
  - 결국 AbilitySystemComponent의 GiveAbility 함수는 이 SPC 객체만을 인자로 받습니다.

<br/>

### 5. 어빌리티 핸들(Handle)을 통한 관리
- 핸들의 역할: 어빌리티 시스템에서 능력을 부여하고 나면 "**핸들**"을 반환받습니다. 언리얼 엔진에서는 이 핸들 방식을 굉장히 많이 사용합니다.
- 핸들은 int32 값으로, 어빌리티 시스템 내부의 ActiveAbilities 배열의 인덱스와 맵핑됩니다. 이는 "**완전한 최적화**"를 위한 것으로, 어빌리티 탐색 속도를 매우 빠르게 합니다.
- 이점:
  - 빠른 탐색: 많은 어빌리티가 한 프레임당 사용될 수 있으므로, 배열 인덱스(핸들 값)를 통해 즉시 접근하여 성능상 유리합니다.
  - 쉬운 제거: 특정 능력을 제거해야 할 때, 핸들 값만 있으면 해당 인덱스를 찾아 쉽게 제거할 수 있습니다.
- GrantedAbilityHandles:AbilitySet 또한 내부적으로 GrantedAbilityHandles라는 배열을 가지고, 부여된 어빌리티의 핸들 값을 저장합니다.
- 이는 나중에 어빌리티 셋이 적용된 능력을 일괄적으로 제거해야 할 경우를 대비하기 위함입니다. AbilitySystemComponent의 능력 제거 함수는 핸들 값을 필요로 합니다.

<br/>

### 결론
라이라의 어빌리티 시스템은 언리얼 엔진의 기본 게임플레이 어빌리티 시스템을 기반으로 하지만, PlayerState에 컴포넌트를 부착하고, PawnData를 통해 AbilitySet을 관리하며, 최적화를 위해 CDO 및 핸들 기반의 배열 탐색 방식을 적극적으로 활용하는 등 다층적인 래핑과 최적화를 통해 효율적이고 유연하게 확장된 형태로 구현되어 있습니다. 특히 HeroComponent를 통해 폰의 완전한 준비 시점에 어빌리티 시스템의 아바타 정보를 확정하는 방식은 라이라의 컴포넌트 기반 설계 철학을 잘 보여줍니다.
