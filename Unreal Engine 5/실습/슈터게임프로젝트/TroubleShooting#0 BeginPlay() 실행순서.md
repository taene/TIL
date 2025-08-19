## 트러블슈팅 보고서: BeginPlay()의 실행순서에 따른 Delegate 꼬임 문제
### 프로젝트: 인벤토리 시스템 개발
### 발생일: 2025-08-13

#### 1. 문제 현상 (Symptom)
- UInventoryComponent가 자신의 초기화가 끝났음을 알리기 위해 BeginPlay에서 델리게이트를 **방송**(Broadcast)하고, AItem 액터는 BeginPlay에서 해당 델리게이트를 **감시**(Bind)하도록 설계했다.
- 하지만 실제 테스트 결과, AItem 액터가 UInventoryComponent의 델리게이트 신호를 받지 못하는 현상이 발생했다.

#### 2. 원인 분석 (Root Cause Analysis)
- 이 문제의 핵심 원인은 언리얼 엔진의 BeginPlay 실행 순서가 액터마다 보장되지 않는다는 특성에 있다.
- 디버깅 결과, 다음과 같은 '**경합 조건**(Race Condition)'이 발생하고 있었다.
1. UInventoryComponent의 BeginPlay()가 먼저 실행됨: 컴포넌트가 자신의 초기화를 모두 마친 뒤, "준비 완료!" 라는 OnInventoryUpdated 델리게이트 신호를 방송(Broadcast)했다.
2. 하지만 이 시점에는 아직 아무도 이 방송을 듣고 있지 않았다. AItem 액터는 아직 자신의 BeginPlay를 실행하기 전이라, 델리게이트에 감시 함수를 등록하지 않은 상태였다.
3. AItem의 BeginPlay()가 뒤늦게 실행됨: AItem은 이제서야 InventoryComponent의 OnInventoryUpdated 델리게이트에 자신의 함수를 **감시**(Bind)하도록 등록했다.
4. 결과: 방송은 이미 끝나버렸고, 감시자는 뒤늦게 도착했다. 결국 AItem은 자신이 기다리던 신호를 영원히 받지 못하게 되었다.

- 이는 마치 "오전 9시에 기차역에서 만나자"고 약속했는데, 한 명이 8시 50분에 도착해서 "나 왔어!" 라고 외치고 그냥 떠나버린 뒤, 다른 한 명이 9시에 도착해서 상대를 기다리는 것과 같은 상황이다.

#### 3. 해결 과정 (Resolution)
- 이 문제를 해결하기 위해, 신호를 보내는 쪽(UInventoryComponent)에 '**자신의 현재 상태를 알려주는 기능**'을 추가하고, 신호를 받는 쪽(AItem)은 '상태를 먼저 확인하고 행동하는' 지능적인 로직을 구현했다.
1. UInventoryComponent 수정:
   - bool bIsInitialized 멤버 변수를 추가하여, BeginPlay가 완료되면 true로 설정하도록 했다.
   - 외부에서 이 상태를 안전하게 확인할 수 있는 IsInitialized() getter 함수를 추가했다.
2. AItem의 BeginPlay 로직 수정:
   - 무조건 델리게이트를 감시(Bind)하는 대신, 다음과 같은 '방어적 프로그래밍(Defensive Programming)' 로직을 적용했다.
     1. 먼저 InventoryComponent의 IsInitialized() 함수를 호출하여 "혹시 방송이 이미 끝났나?" 라고 상태를 확인한다.
     2. 만약 true (이미 초기화 완료)라면, 기다릴 필요 없이 즉시 필요한 로직(아이템 추가 등)을 실행한다.
     3. 만약 false (아직 초기화 전)라면, 그때 델리게이트에 감시 함수를 등록하고 신호를 기다린다.

- 이 설계를 통해, BeginPlay가 어떤 순서로 실행되더라도 AItem은 InventoryComponent의 상태를 놓치지 않고 100% 안정적으로 자신의 로직을 실행할 수 있게 되었다.

#### 4. 교훈 및 예방 조치 (Lessons Learned & Prevention)
- 델리게이트는 매우 강력한 도구지만, 비동기적으로 호출되는 BeginPlay와 같은 함수 내에서 사용할 때는 항상 실행 순서 문제를 염두에 두어야 한다.
   - 핵심 원칙: "신호를 기다리기 전에, 신호가 이미 지나갔는지 먼저 확인하라."
   - 예방 조치: 델리게이트를 방송하는 클래스는 자신의 상태를 외부에 알려줄 수 있는 IsReady(), IsInitialized() 같은 상태 변수와 getter 함수를 함께 제공하는 것이 좋다. 이를 통해 델리게이트 구독자는 경합 조건에 빠지지 않고 안정적으로 이벤트를 처리할 수 있다. 이는 라이라(Lyra)와 같은 대규모 프로젝트에서 시스템 간의 안정적인 상호작용을 보장하는 핵심적인 설계 패턴이다.
