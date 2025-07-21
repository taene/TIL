## 학습 키워드

  - UMG 애니메이션 (UMG Animation)
  - 키프레임 (Keyframe)
  - 위젯 컴포넌트 (WidgetComponent)
  - 위젯 스페이스 (Widget Space)
  - C++에서 블루프린트 함수 호출

<br/>


## 학습 내용

### 배운 개념 요약

  - **UMG 애니메이션 시스템**

      - 언리얼 엔진의 UI 시스템(UMG)은 위젯 블루프린트 내에 강력한 애니메이션 편집 기능을 내장하고 있다.
      - **타임라인**(Timeline)**에 **키프레임**(Keyframe)을 추가하는 방식으로 UI 요소의 속성(위치, 크기, 투명도, 색상 등)을 시간에 따라 변화시켜 동적인 UI 연출을 만들 수 있다.
      - 예를 들어, `Render Opacity` (불투명도) 값을 0초에 0, 1초에 1로 설정하면 1초에 걸쳐 서서히 나타나는 페이드 인(Fade-in) 효과를 구현할 수 있다.

  - **C++에서 블루프린트 함수 호출**

      - C++ 코드에서 위젯 블루프린트 내에 정의된 함수나 이벤트를 호출해야 하는 경우가 있다. (예: 게임 오버 시점에서 UI 애니메이션 재생)
      - 이 경우, 위젯 인스턴스에서 `FindFunctionByName()`으로 함수의 `UFunction` 포인터를 찾은 뒤, `ProcessEvent()`를 통해 해당 함수를 실행할 수 있다. 이는 C++와 블루프린트가 양방향으로 소통할 수 있게 해주는 강력한 기능이다.

  - **위젯 컴포넌트 (WidgetComponent)**

      - UMG로 만든 2D UI 위젯을 3D 월드 공간에 표시할 수 있게 해주는 액터 컴포넌트다.
      - NPC의 머리 위에 뜨는 체력 바나 이름표, 월드에 고정된 상호작용 프롬프트 등을 구현하는 데 사용된다.
      - **위젯 스페이스 (Widget Space)** 설정을 통해 3D 공간에서의 렌더링 방식을 결정할 수 있다.
          - **`World`**: 위젯이 3D 공간상의 평면처럼 존재한다. 카메라 각도에 따라 원근감이 적용되고 다른 오브젝트에 가려질 수 있다.
          - **`Screen`**: 위젯이 항상 카메라를 바라보며 화면에 평평하게 렌더링된다. 위치는 3D 월드 좌표를 따라가지만, 항상 정면으로 보여 가독성이 중요한 체력 바 같은 UI에 적합하다.

<br/>


### 구현 과정

#### 1. 게임 오버 UI 애니메이션 제작

1.  **위젯 준비**: `WBP_MainMenu` 위젯을 열고, "Game Over!"와 최종 점수를 표시할 `TextBlock` 위젯 두 개를 추가한다. 이 텍스트들이 평소에는 보이지 않도록 `Details` 패널에서 `Visibility`를 `Hidden`으로 설정한다.
2.  **애니메이션 생성**: 디자이너 좌측 하단의 `Animations` 패널에서 `+ Animation` 버튼을 눌러 `Anim_TimeOver`라는 새 애니메이션을 만든다.
3.  **키프레임 설정**:
      - `GameOverText` 위젯을 타임라인에 트랙으로 추가한다.
      - `Render Opacity` 속성을 추가하고, 타임라인을 움직여가며 0초에 0, 2초에 1, 4초에 1 등으로 키프레임을 찍어 페이드 인 효과를 만든다.
4.  **블루프린트 함수로 래핑**: `Graph` 탭에서 `PlayGameOverAnim`이라는 새 함수를 만든다. 이 함수는 숨겨져 있던 `GameOverText`를 `Visible`로 바꾸고, `Play Animation` 노드를 이용해 `Anim_TimeOver` 애니메이션을 재생시킨다. 애니메이션 재생이 끝나면 최종 점수 텍스트도 `Visible`로 변경한다.

#### 2. C++에서 게임 오버 애니메이션 호출

1.  **`PlayerController` 로직 수정**: `ASpartaPlayerController::ShowMainMenu` 함수에 `bIsRestart`가 `true`(게임 오버 상황)일 때 애니메이션을 재생하는 로직을 추가한다.

2.  **블루프린트 함수 호출**: `MainMenuWidgetInstance->FindFunctionByName(TEXT("PlayGameOverAnim"))`으로 위젯에 만든 함수의 포인터를 찾는다. 함수를 찾았다면, `MainMenuWidgetInstance->ProcessEvent(PlayAnimFunc, nullptr)` 코드로 해당 애니메이션 재생 함수를 실행한다.

    ```cpp
    // SpartaPlayerController.cpp의 ShowMainMenu 함수 내부
    if (bIsRestart)
    {
        // 블루프린트에서 만든 'PlayGameOverAnim' 함수를 이름으로 찾는다.
        UFunction* PlayAnimFunc = MainMenuWidgetInstance->FindFunctionByName(TEXT("PlayGameOverAnim"));
        if (PlayAnimFunc)
        {
            // 찾은 함수를 실행한다.
            MainMenuWidgetInstance->ProcessEvent(PlayAnimFunc, nullptr);
        }
        // ... 최종 점수 텍스트 업데이트 로직 ...
    }
    ```

#### 3. 캐릭터 머리 위 체력바 (3D 위젯) 구현

1.  **체력바 위젯 생성**: `WBP_HP`라는 이름의 새 위젯 블루프린트를 만들고, 내부에 `TextBlock` 하나만 배치하여 체력을 표시할 준비를 한다.
2.  **`WidgetComponent` 추가**: `ASpartaCharacter.h`에 `UWidgetComponent* OverheadWidget` 멤버 변수를 선언한다.
3.  **컴포넌트 설정 (C++)**: `ASpartaCharacter` 생성자에서 `CreateDefaultSubobject`로 `WidgetComponent`를 생성하고, 캐릭터의 `Mesh`에 부착한다. `SetWidgetSpace(EWidgetSpace::Screen)`으로 설정하여 항상 화면을 바라보도록 한다.
4.  **컴포넌트 설정 (블루프린트)**: `BP_SpartaCharacter` 블루프린트 에디터에서 `OverheadWidget` 컴포넌트를 선택하고, `Widget Class` 속성에 `WBP_HP`를 할당한다. 뷰포트에서 위젯의 위치를 머리 위로 적절히 조정한다.
5.  **실시간 업데이트 함수 구현**: `ASpartaCharacter`에 `UpdateOverheadHP` 함수를 만든다. 이 함수는 `WidgetComponent`에서 실제 `UUserWidget` 객체를 얻어온 뒤, 그 안의 `TextBlock`를 이름으로 찾아 `SetText`를 호출하여 현재 체력 값으로 갱신한다.
6.  **업데이트 호출**: `BeginPlay`, `TakeDamage`, `AddHealth` 등 체력이 변경될 수 있는 모든 시점에서 `UpdateOverheadHP` 함수를 호출하여 체력바가 실시간으로 반영되도록 한다.

#### 4. 캐릭터 사망과 게임 오버 연동

1.  **`OnDeath` 로직 수정**: 캐릭터의 체력이 0이 되어 `ASpartaCharacter::OnDeath` 함수가 호출되면, `GameState`의 `OnGameOver` 함수를 호출하도록 연결한다.
2.  **`OnGameOver` 로직 수정**: `ASpartaGameState::OnGameOver` 함수에서는 `PlayerController`의 `SetPause(true)`를 호출하여 게임을 일시정지시키고, `ShowMainMenu(true)`를 호출하여 위에서 만든 애니메이션이 포함된 게임 오버 화면을 띄운다.

<br/>


## 느낀점

  - UMG의 애니메이션 기능은 단순히 UI를 표시하는 것을 넘어, 사용자에게 동적인 피드백을 주며 게임의 몰입감을 한층 높여주는 중요한 요소임을 깨달았다. 키프레임 기반의 직관적인 인터페이스 덕분에 코드 없이도 복잡한 연출을 쉽게 만들 수 있었다.
  - `WidgetComponent`는 UI와 3D 월드의 경계를 허무는 매우 창의적인 기능이다. 화면에 고정된 HUD뿐만 아니라, 월드 공간 자체에 정보를 띄울 수 있게 되면서 게임 디자인의 폭이 훨씬 넓어졌다. 특히 `Screen Space` 옵션은 3D 위치를 가지면서도 2D처럼 항상 정면을 바라보게 해줘 가독성이 중요한 체력바 등에 최적화된 기능이라고 생각한다.
  - 이번 강의를 통해 C++와 블루프린트의 유기적인 연동이 더욱 깊어졌다. C++에서 블루프린트 함수를 `FindFunctionByName`과 `ProcessEvent`로 호출하는 방식은, 성능이 중요한 핵심 로직은 C++로 짜고 시각적인 연출이나 간단한 로직은 블루프린트로 구현하는 하이브리드 개발의 정수를 보여준다.

-----

### 요약

- UMG의 **애니메이션** 기능을 사용하면 위젯의 투명도, 위치 등을 **키프레임**으로 조절하여 동적인 UI 연출을 만들 수 있다.
- 이렇게 블루프린트에서 만든 애니메이션은 C++의 `FindFunctionByName`과 `ProcessEvent`를 통해 호출할 수 있다.
- 3D 월드에 UI를 표시하려면 `WidgetComponent`를 사용하며, 캐릭터 머리 위 체력바처럼 항상 카메라를 바라봐야 하는 UI는 `Space` 설정을 `Screen`으로 지정한다.
- 캐릭터가 사망하면(`OnDeath`) `GameState`의 `OnGameOver`를 호출하고, 여기서 다시 `PlayerController`를 통해 게임을 일시정지시키고 애니메이션이 포함된 게임 오버 메뉴를 표시하는 흐름으로 전체 게임 루프를 완성한다.
