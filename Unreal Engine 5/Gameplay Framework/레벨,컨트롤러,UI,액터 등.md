## 1. 언리얼 엔진 게임 프레임워크
- **GameMode**: 현재 게임이 어떤 규칙으로 돌아갈지 정의함
- **PlayerController**: 플레이어의 입력을 받아 처리함
- **Pawn/Character**: 플레이어 혹은 AI가 조종할 수 있는 오브젝트
- **UI**: 게임 화면에 표시되는 인터페이스
- [언리얼 게임플레이 프레임워크 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/gameplay-framework-in-unreal-engine)
> 위 요소들은 **Level** 안에서 함께 동작하고, **Actor**들이 실질적인 오브젝트로 존재하며, **UI**를 통해 플레이어와 정보를 주고 받는다.

<br/>

## 2. 레벨(Level)
### 🔍 메인 레벨과 서브 레벨
- 게임이 진행되는 맵, 장면(Scene), 무대(Stage)
- 하나의 레벨을 Persistent Level로 두고, Sub-Level을 추가로 불러와서 사용할 수 있다.
  - 지형, 조명, 인테리어 등 특정 **기능적 구분**을 위해 레벨을 여러 개로 나눈다.
<details>
  <summary>🖼️[Window - Levels]</summary>
  <img src="https://github.com/user-attachments/assets/857db64f-801d-4b35-b13a-8952e114a6d9" alt="Window - Levels">
</details>

<details>
  <summary>🖼️[레벨별 다른 색]</summary>
  <img src="https://github.com/user-attachments/assets/d128f64d-2cc6-4e99-bc9e-faa7014c19d1" alt="레벨별 다른 색">
</details>

<br/>

### 🔍 Level Streaming
- Level Streaming 기능을 통해 **필요한 시점**에만 레벨 데이터(지형, 액터 등)을 불러오고 unload해 **메모리 사용량과 성능**을 효율적으로 관리한다.
- 대규모 오픈월드 게임에서 중요한 기술 요소 중 하나
- [레벨 스트리밍 개요 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/level-streaming-overview-in-unreal-engine)
<details>
  <summary>🖼️[레벨 창-서브레벨 우클릭-스트리밍 방법 변경]</summary>
  <img src="https://github.com/user-attachments/assets/24d38c80-4b76-4e6a-8381-7d73e6d3e634" alt="레벨 창-서브레벨 우클릭-스트리밍 방법 변경">
</details>

<br/>

### 🔍 레벨 전환(Load/Travel)
- 게임 진행 상황에 따라 새로운 레벨로 전환이 필요할 때, 블루프린트/C++ 함수를 통해 다른 레벨을 로드한다.
- 여러 레벨을 이동하는 구조
<details>
  <summary>🖼️[BeginPlay() -> 3.0 Delay -> Open Level(by Name) ThirdPersonMap]</summary>
  <img src="https://github.com/user-attachments/assets/b7ddd144-ee7d-41f2-8a2b-496ed21f43fa" alt="레벨 전환 실습">
</details>

<br/>

## 3. 컨트롤러(Controller)
### 🔍 플레이어 컨트롤러(PlayerController)
- 플레이어가 입력한 키보드/마우스/패드 신호를 받아 게임 내 행동으로 연결한다.
- 플레이어 뷰(화면 카메라), UI 조작에도 중요한 역할을 수행한다.
- 하나의 플레이어는 일반적으로 하나의 PlayerController를 가진다.
- [플레이어 컨트롤러 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/player-controllers-in-unreal-engine)

<br/>

### 🔍 컨트롤러와 폰(Pawn)의 관계
- 컨트롤러는 폰을 소유(possess)해 직접 제어할 수 있다.
  - 플레이어 컨트롤러가 캐릭터(Characters)를 소유하면, 키보드 입력이 그 캐릭터의 움직임으로 이어진다.
- 소유 상태가 바뀌면, 다른 폰을 조종하거나 새로운 캐릭터로 교체할 수도 있다.

<br/>

### 🔍 AI 컨트롤러(AIController)
- NPC나 적 캐릭터의 인공지능 로직을 담는다.
- AI 전용 블루프린트/C++ 클래스를 만들어, Behavior Tree 등을 통해 의사결정 과정을 구현한다.
- [Behavior Tree 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/behavior-tree-in-unreal-engine---overview)
![Image](https://github.com/user-attachments/assets/06b5b25d-b067-44e9-8114-452d64d04d67)


<br/>

## 4. UI(User Interface) 시스템
### 🔍 UMG(Unreal Motion Graphics)
- UMG는 언리얼 엔진에서 UI를 만드는 전용 툴이자 프레임워크
- 블루프린트로 UI 요소를 시각적으로 설계하고, 버튼, 텍스트, 슬라이더, 이미지 등을 배치할 수 있다.
- 드래그&드롭 기능이 직관적이기 때문에 UI 레이아웃을 쉽게 구성할 수 있다.
- [UMG UI 디자이너 가이드 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/umg-ui-designer-quick-start-guide-in-unreal-engine)

<br/>

## 5. Actor와 게임 세계 구성
### 🔍 액터의 기본 구조와 생명주기(Lifecycle)
- 액터는 언리얼의 월드에 배치되는 모든 오브젝트의 부모 클래스이다.
  - [액터 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/actors-in-unreal-engine)
- **초기화->시작(Spawned)->Tick(프레임마다 업데이트)->Destroy** 등의 흐름을 가진다.
  - [액터 라이프사이클 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/unreal-engine-actor-lifecycle)
- 위치, 회전, 스케일(Transform)을 기본적으로 지니고, 다양한 컴포넌트를 붙일 수 있다.

<details>
  <summary>🖼️Actor Lifecycle</summary>
  <img src="https://github.com/user-attachments/assets/61e968e2-3258-47f4-b36a-db518516d377" alt="Actor Lifecycle">
</details>

<br/>

### 🔍 Pawn, Character, Component
- Actor > Pawn > Character
- **Pawn**: **Actor를 상속받아** "조종 가능한" 특징을 더한 클래스
- **Character**: **Pawn을 상속받아** "걷기, 달리기, 점프" 등의 이동 로직이 포함된 클래스
- **Component**: Actor가 자기 자신에 서브 오브젝트로 붙일 수 있는 특수한 타입의 오브젝트
- [컴포넌트 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/components-in-unreal-engine)

<br/>

### 🔍 이벤트(Event)와 상호작용
- Actor는 블루프린트 이벤트(BeginPlay, Tick, Overlap 등)를 통해 **상호작용** 로직을 구현한다.
- ex) 캐릭터가 문(Actor)에 가까이 가면 자동으로 열리는 이벤트 처리 등
- [이벤트 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/events-in-unreal-engine)

<br/>
