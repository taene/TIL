## 트러블슈팅 보고서: UI 모드 전환 후 입력 없음 문제
### 프로젝트: 인벤토리 시스템 개발
### 발생일: 2025-08-17

#### 1. 문제 현상 (Symptom)
- 'I' 키를 눌러 인벤토리 UI를 여는 것까지는 정상적으로 작동했다.
- 그러나 인벤토리가 열린 상태에서 'I' 키를 다시 눌러도 아무런 반응이 없었다. 즉, 인벤토리를 닫을 수 없는 문제가 발생했다.

#### 2. 원인 분석 (Root Cause Analysis)
- 이 문제는 언리얼 엔진의 입력 모드(Input Mode) 시스템의 동작 방식을 정확히 이해하지 못했기 때문에 발생했다.
- 주요 원인: ToggleInventory() 함수에서 인벤토리를 열 때 PlayerController->SetInputMode(FInputModeUIOnly())를 호출했다. 이 UIOnly 모드는 이름 그대로 모든 키보드 및 마우스 입력을 오직 UI 위젯에게만 전달하도록 설계되었다. 그 결과, 'I' 키 입력은 더 이상 AShooterCharacter의 Input Component에 도달하지 못했다. 캐릭터가 입력을 받지 못하니, ToggleInventory() 함수가 다시 호출될 방법이 없었던 것이다.
- 이것은 버그가 아니라, 입력의 흐름을 명확하게 분리하는 엔진의 의도된 동작이다.

#### 3. 해결 과정 (Resolution)
- 문제를 해결하기 위해 "UI가 활성화된 동안에는 UI가 직접 입력을 처리해야 한다"는 원칙에 따라 접근했다.
- UInventoryWidget C++ 클래스에 OnKeyDown 가상 함수를 오버라이드(override) 하도록 선언했다. OnKeyDown은 해당 위젯이 포커스를 가지고 있을 때 키보드 입력이 발생하면 자동으로 호출되는 함수다.
- UInventoryWidget::OnKeyDown 함수의 내용을 구현했다.
  - InKeyEvent.GetKey()를 통해 눌린 키가 'I' 키(EKeys::I) 인지 확인했다.
  - 'I' 키가 맞다면, GetOwningPlayerPawn()을 통해 이 위젯을 소유한 플레이어 캐릭터(AShooterCharacter)의 포인터를 얻어왔다.
  - 얻어온 캐릭터 포인터를 통해 PlayerCharacter->ToggleInventory() 함수를 직접 호출하여 UI를 닫도록 명령했다.
  - 마지막으로 FReply::Handled()를 반환하여, 이 'I' 키 입력은 여기서 완전히 처리되었으며 다른 곳으로 전파할 필요가 없다고 시스템에 알렸다.

```cpp
// Character.cpp
void AShooterCharacter::ToggleInventory()
{
	check(InventoryWidgetClass);

	AShootPlayerController* PlayerController = Cast<AShootPlayerController>(GetController());
	check(PlayerController)
	{
		if (InventoryWidget->IsInViewport())
		{
			InventoryWidget->RemoveFromParent();
			PlayerController->SetInputMode(FInputModeGameOnly());
			PlayerController->SetShowMouseCursor(false);
		}
		else
		{
			InventoryWidget->AddToViewport();
			PlayerController->SetInputMode(FInputModeUIOnly().SetWidgetToFocus(InventoryWidget->TakeWidget()));
			PlayerController->SetShowMouseCursor(true);
			InventoryWidget->SetKeyboardFocus();
			InventoryWidget->RefreshInventory(InventoryComp);
		}
	}
}

// InventoryWidget.h
public:
  virtual FReply NativeOnKeyDown(const FGeometry& InGeometry, const FKeyEvent& InKeyEvent) override;

// .cpp
FReply UInventoryWidget::NativeOnKeyDown(const FGeometry& InGeometry, const FKeyEvent& InKeyEvent)
{
    if (InKeyEvent.GetKey() == EKeys::I)
    {
        AShooterCharacter* PlayerCharacter = Cast<AShooterCharacter>(GetOwningPlayerPawn());
        if (PlayerCharacter)
        {
            PlayerCharacter->ToggleInventory();
            return FReply::Handled();
        }
    }
    return FReply::Unhandled();
}
```

- 이 과정을 통해, 입력의 제어권이 UI로 넘어갔을 때 UI 스스로가 '닫기' 로직을 트리거할 수 있게 되어 문제가 해결되었다.

#### 4. 교훈 및 예방 조치 (Lessons Learned & Prevention)
- FInputModeUIOnly와 FInputModeGameOnly를 전환하는 기능을 구현할 때는, 각 모드에서 어떤 객체가 입력을 처리할 것인지 명확하게 설계해야 한다.
- GameOnly 모드: 플레이어 폰(캐릭터) 또는 컨트롤러가 입력을 처리한다.
- UIOnly 모드: 포커스를 가진 UI 위젯이 입력을 처리한다.
- UI를 여는 기능은 캐릭터가 담당하더라도, 그 UI를 닫는 기능(특히 동일한 키를 사용하는 경우)은 UI 위젯 스스로가 담당하도록 OnKeyDown과 같은 함수를 오버라이드하는 것이 표준적인 해결 방법이다.
