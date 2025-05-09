## 학습 키워드
- BehaviorTree(BT)
- Task: Composites[Selector/Sequence/Parallel/Task]/Decorator/Condition/Action
- Blackboard
- Navigation

<br/>

## 학습 내용
### 배운 개념 요약
![image](https://github.com/user-attachments/assets/ac0cd038-c89d-41fd-84ac-5da9a515cc85)
- [BehaviorTree(행동트리)](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/behavior-trees-in-unreal-engine): AI 캐릭터 구현 방법 중 하나
  - 같은 상태를 재작성하지 않고도 다른 목적이나 상황에 따라 상태를 재사용할 수 있도록 함
  - 작동 원리: 자료구조 Stack
  - Root: BT 시작점, 행동 결정의 밑 바탕
  - Composite: 여러 개의 자식Task로 구성된 Task
    - Selector: 조건을 검사해 상황에 맞거나 맞지 않는 Task를 구분해 실행시키는 역할(조건에 따라 **하나를 선택**)
    - Sequence: 여러 개의 Task를 **순차적으로 실행**하는 역할
    - Parallel: 여러 개의 Task를 묶어 **한번에 실행**하는 역할
  - Task: **실제 행동**(ex_캐릭터의 이동 등)
  - Decorator: Task 실행 여부에 **조건**을 추가하거나, 특정 기능을 추가하는 역할
  - Service: 일정 시간마다 실행되며 **주로 데이터를 갱신**하는 데 사용

![image](https://github.com/user-attachments/assets/7fcb64bf-8a0f-405f-8256-a360495e3b56)
- Unreal BT 대략적인 구조

<br/>

### 구현 과정(AI가 Player Character와 가깝거나 아닌 경우의 BT)
#### 1. BT 생성
- Content Drawer > Artificial Intelligence > Behavior Tree 생성(BT_)
- Root - Selector - Move To 노드 생성
- 특정 위치 데이터를 BT가 알기 위해서는 해당 데이터를 저장하고 읽을 수 있는 **블랙보드 애셋**이 필요함

#### 2. Blackboard Data 생성
- Content Drawer > Artificial Intelligence > Blackboard 생성(BB_)
- 블랙보드 생성 후 BT에 블랙보드 설정
- BB에서 New Key를 눌러 목표 위치를 저장하고 읽는 벡터 데이터 TargetPosition을 설정
- MoveTo에 TargetPosition(BB Key) 할당

#### 3. BT Decorator 생성
- 플레이어가 AI 캐릭터에 근접하면, TargetPosition을 플레이어 캐릭터의 위치로 설정함
- New Decorator > BTDecorator_BlueprintBase > BTD 생성(BTD_IsNearPlayer)

![image](https://github.com/user-attachments/assets/b6ac1771-ed6d-448f-b9ec-6246927a882f)
- Perform Condition Check: 값이 True면 해당 작업을 실행, False면 실행하지 않는 함수(오버라이드)
- Get Actor Location | Make Array | Sphere Overlap Actors: 캐릭터 주변을 구형으로 탐색하여, 범위 안의 모든 폰을 반환
- Get Player Pawn | Find: 반환된 폰 중 플레이어 캐릭터를 검색하여, 플레이어 캐릭터가 있으면 데코레이터의 Return Value를 True, 아니면 False를 반환
- Get BB | Set Value as Vector | String to Name | Get Player Pawn | Get Actor Location: 플레이어 탐색 성공 시 BB의 TargetPosition을 검색하고 해당 값을 설정

#### 4. BT Service 생성
- 플레이어가 AI 캐릭터에 근접하지 않을 경우, 무작위 위치로 설정함
- New Service > BTService_BlueprintBase > BTS 생성(BTS_FindingRandom)
- Receive Activation: 서비스 활성화 시 호출되는 함수(이벤트)

#### 5. Navigation 설정
- 범위 안에 플레이어가 탐색되지 않으면 BT는 TargetPosition을 무작위 위치로 선정하고 그곳으로 이동하게 설정했음
- 이 '적절한 무작위 위치 선정'을 위해 내비게이션을 이용할 것임
- Navigation: AI 캐릭터에게 최적의 동선을 제공하거나, 장애물을 피하도록 경로를 탐색하는 역할
- Level > Nav Mesh Bounds Bolume Actor 추가(Navigation 시스템이 움직일 수 있는 영역)
  - Actor를 선택하고 p를 누르면 영역이 가시화됨
  - Transform Location(0,0,0) / Scale or Brush Settings로 영역 설정

#### 6. Finding Random Event 할당
![image](https://github.com/user-attachments/assets/0a1a40b4-f2da-4b01-901c-e42d29abe1a7)
- 갈 수 있는 영역을 검색하고 갈 수 있다면 블랙보드의 TargetPosition 값을 해당 위치로 할당해줌
- GetRandomLocationInNavigableRadius: 특정 위치에서부터 반경을 잡아 그 안에 랜덤으로 포지션을 정해줌(Navigation 시스템 안에서만 정상 작동)

![image](https://github.com/user-attachments/assets/abc60e41-fa82-425f-8f3c-9f1756f54abe)
- TargetPosition에 도달 시 다음에 이동할 새로운 TargetPosition을 지정해야 함
- 프레임 마다 실행되는 Tick 이벤트를 추가하고 TargetPosition에 도달하게 되면 새로운 TargetPosition을 재검색 하도록 함

<br/>

![image](https://github.com/user-attachments/assets/66a54a1b-e6af-4276-b06d-17eb54216cc4)
- 최종 BT

<br/>
