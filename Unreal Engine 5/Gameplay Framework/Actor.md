## 1. Actor란?
- 언리얼 엔진에서 Actor는 Level(Map)내에 존재하는 모든 오브젝트의 기반 클래스
  - 건물, 캐릭터, 라이트, 카메라 등 눈에 보이는 오브젝트뿐 아니라 Trigger, Volume처럼 눈에 보이지 않는 것도 모두 액터의 일종이다.
  - 액터마다 고유한 컴포넌트를 갖고 있고, 위치-회전-스케일 같은 Transform 정보를 갖는다.
- Actor: 게임 세계에서 뭔가 역할을 하는 모든 존재

<br/>

## 2. 언리얼 엔진에서 자주 쓰이는 액터 종류
### 🔍 Empty Actor(기본 액터)
- 아무 컴포넌트가 없는 빈 액터
- 게임 로직 또는 특정 위치나 트리거 용도로만 필요할 때 유용하다.
- 원하는 컴포넌트를 직접 추가해 커스텀해서 사용하기도 한다.
<p>$\color{#808080}ex)\ A\ Empty\ Actor와\ B\ Empty\ Actor의\ 위치\ 사이를\ 배회하는\ AI\ 움직임을\ 구현$</p>
<details>
  <summary>🖼️Empty Actor</summary>
  <img src="https://github.com/user-attachments/assets/5bf39cdc-c98d-4af7-9220-b050c1d0fa71">
</details>

<br/>

### 🔍 Static Mesh Actor
- **고정된 메시(3D 모델)를 표시**하기 위한 액터
- 스태틱 메시 컴포넌트에 할당된 Model과 Material을 통해 **시각적 표현**을 한다.
<p>$\color{#808080}ex)\ 건물,\ 나무,\ 바위,\ 가구\ 등\ 움직임이\ 크게\ 필요없는\ 오브젝트$</p>
<details>
  <summary>🖼️Static Mesh Actor</summary>
  <img src="https://github.com/user-attachments/assets/32b32d52-7050-4cc6-8746-7142a994c172">
</details>

<br/>

### 🔍 Light Actor
- 씬에 **조명을 제공**하는 액터
- 씬 분위기를 조절할 때 꼭 필요한 액터, 섬세한 설정(색상, 밝기, 그림자 방식 등)이 가능하다.
  - Sky Light(레벨 전체 광원), Directional Light(태양광 느낌), Point Light(점광원), Spot Light(원뿔), Rect Light(직사각형) 등 다양하다.
- [환경 라이팅 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/lighting-the-environment-in-unreal-engine)
<details>
  <summary>Light 용어들</summary>
  <ul>
    <li><b>Specular</b>: 어떠한 물체 표면에 맺히는 라이트의 강조된(하이라이트) 부분을 말한다.</li>
    <li><b>Intensity(강도)</b>: 빛의 세기, 높을수록 밝아진다.</li>
    <dd>- Directional Light는 빛의 조도를 나타내는 럭스(Lux)단위를 사용하며, 1럭스는 1 제곱미터 당 1루멘이 비출 때의 조도를 의미한다.</dd>
    <dd>- Point, Spot, Rect Light는 빛의 광도를 나타내는 칸델라(cd)단위를 사용하며, 1칸델라는 1스테라디안 당 나오는 빛의 세기를 의미한다.</dd>
    <li><b>Attenuation Radius(감쇠 반경)</b>: 빛이 점점 감쇠하는 반경을 말한다. 빛은 감쇠 반경의 끝까지 나아가지만 광원에서 멀어질수록 빛이 세기가 급격히 감소한다(거리의 제곱에 반비례)</li>
    <li><b>Direct Light(직접광)</b>: 어디에도 부딪치지 않은 광원에서 바로 뿜어져 사물에 부딪히는 빛 </li>
    <li><b>Indirect Light(간접광)</b>: 직접광에 부딪친 빛이 반사된 뒤 사물에 부딪히는 빛</li>
    <li><b><i>Static</i></b>: 게임 내에서 라이트를 변경하지 않는 것을 의미한다. 라이팅을 미리 구울 수 있어(베이크) 렌더링 시 가장 빠른 속도를 자랑한다. 다만 게임 진행 시 해당 라이트를 이동시킬 수 없기 때문에 배경 등 좁은 영역으로만 사용할 수 있다.</li>
    <li><b><i>Stationary</i></b>: 스태틱 지오메트리에서의 반사광과 그림자가 라이트매스로 굽고(베이크) 다른 모든 것들은 실시간으로 렌더링한다. 게임 진행 중에 라이트의 색상과 강도 등을 변경할 수 있지만 이동과 관련된 것은 불가능하다.</li>
    <li><b><i>Moveable</i></b>: 완전한 동적 라이트로 다이내믹 섀도우(실시간 그림자)를 구현할 수 있다. 라이트를 들고 다니거나 하는 등의 모든 행동이 가능하지만 렌더링 속도가 느리다는 단점이 있다.</li>
  </ul>
</details>
<details>
  <summary>🖼️Light Actor</summary>
  <img src="https://github.com/user-attachments/assets/9472d4da-9acb-4716-9a10-6a5d78e10519">
  <img src="https://github.com/user-attachments/assets/927c5ff1-8b60-4f50-9bff-01e090af0331">
</details>

<br/>

### 🔍 Pawn & Character
- Pawn은 플레이어나 AI가 점유(조종)할 수 있는 액터
- 기본적으로 '이동'과 같은 인터랙션의 주체가 된다.
  - Character는 Pawn을 상속받아 움직임(워킹, 점프 등)이 기본적으로 포함된 액터이다.
- 플레이어가 직접 조작하는 주인공 캐릭터 혹은 AI 적 캐릭터를 구현할 때 주로 사용한다.
<details>
  <summary>🖼️Pawn&Character</summary>
  <img src="https://github.com/user-attachments/assets/210285dc-fe6d-4c8c-b6e2-7d60ad6fb6a1">
</details>

<br/>

### 🔍 Camera Actor
- 게임 내 **시점을 결정**하는 액터
- CutScene, Cinematic 또는 특정 연출 장면에서 필요한 카메라 위치를 설정할 때 유용하다.
  - Pawn 또는 Character에 카메라를 붙일 수도 있지만, 별도의 독립된 카메라 액터를 배치해 원하는 시점을 연출하기도 한다.
<details>
  <summary>🖼️Camera Actor</summary>
  <img src="https://github.com/user-attachments/assets/7f332436-b231-4889-a5cd-e4aadaadeea7">
</details>

<br/>

### 🔍 Volume Actor
- PostProcessVolume, LightmassImportanceVolume, TriggerVolume 등 눈에 보이지 않지만 **게임 환경이나 이벤트를 제어**하는 데 사용되는 액터 모음
<p>$\color{#808080}ex)\ PostProcessVolume을\ 이용해\ 특정\ 구역의\ 색감을\ 바꾸거나,\ TriggerVolume을\ 통해\ 캐릭터가\ 해당\ 볼륨\ 안에\ 들어오면\ 이벤트를\ 발생시킬\ 수\ 있습니다.$</p>
<details>
  <summary>🖼️Volume Actor</summary>
  <img src="https://github.com/user-attachments/assets/10aa2fd9-97d5-45c6-9012-722b6bbae8b6">
</details>

<br/>

## 3. 액터 배치 방법
### 🔍 콘텐츠 브라우저에서 드래그&드롭
- 콘텐츠 브라우저(Content Drawer) 혹은 액터 배치 패널에서 원하는 액터 에셋(스태틱 메시, 블루프린트 등)을 선택 → 뷰포트로 드래그 & 드롭 →  아웃라이너(Outliner)에 새롭게 추가됨

<br/>

### 🔍 기존 배치된 액터 복사&붙여넣기
- Ctrl+C, Ctrl+V
- Alt+드래그

<br/>

### 🔍 뷰포트에서 이동·회전·스케일 조정
- 액터를 선택하면 Transform Gizmo(화살표, 회전 핸들, 스케일 박스) 가 나타납니다.
- W, E, R 키를 사용해 이동, 회전, 스케일 모드로 전환 가능 (기본 단축키)
- Shift+드래그 등을 활용해 정밀하게 배치하거나, Snap 기능으로 그리드 단위로 정렬할 수도 있습니다.
![Image](https://github.com/user-attachments/assets/60cf4385-b987-4992-a01d-2617573659b1)
<br/>

### 🔍 블루프린트나 C++ 코드로 액터 스폰
- 레벨 에디터에서 직접 배치하지 않고, 게임 로직을 통해 런타임 시점에 생성할 수도 있습니다.
- 블루프린트 Spawn Actor from Class 노드나, C++의 SpawnActor<>() 함수를 이용합니다. 무작위 위치에서 적이 스폰되거나, 플레이어 상태에 따라 특정 아이템이 생성되는 등의 상황에 사용합니다.
<br/>
