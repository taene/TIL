## 트러블슈팅 보고서: UMG 위젯 포인터 nullptr 문제
### 프로젝트: 인벤토리 시스템 개발
### 발생일: 2025-08-15

#### 1. 문제 현상 (Symptom)
- UInventoryWidget C++ 클래스에서 UPROPERTY(meta=(BindWidget)) 매크로를 사용하여 TObjectPtr<UWrapBox> WrapBox 변수를 선언했다.
- UMG 에디터의 WBP_Inventory 위젯 블루프린트에도 WrapBox라는 이름의 Wrap Box 컴포넌트를 추가했음에도 불구하고, RefreshInventory() 함수 호출 시 WrapBox 포인터가 nullptr이어서 에디터가 크래시되는 현상이 발생했다.

#### 2. 원인 분석 (Root Cause Analysis)
- meta=(BindWidget) 지정자는 C++ 코드의 변수와 UMG 디자이너의 위젯을 이름을 기반으로 자동으로 연결해주는 강력한 기능이다.
- 이 기능이 정상적으로 작동하려면 몇 가지 전제 조건이 충족되어야 한다.
- 디버깅 결과, 가장 핵심적인 전제 조건이 누락되었음을 확인했다.

- 주요 원인: InventoryWidget을 CreateWidget 하지 않았었다. CreateWidget<>()으로 위젯 인스턴스를 만들어줘야지만 C++ 클래스의 변수와 WBP의 변수가 바인드된다는 것을 알았다.

- 해당 현상의 원인 가능성:
  - 위젯 블루프린트의 부모 클래스(Parent Class)를 만든 C++ 클래스로 지정하지 않음
  - UMG 디자이너의 위젯 **변수 이름(Variable Name)**과 C++ 변수명의 불일치 (오타 등)
  - UMG 위젯의 'Is Variable?' 체크박스 비활성화
  - C++ 변수 타입(UWrapBox)과 UMG 위젯 타입의 불일치

#### 3. 해결 과정 (Resolution)
```cpp
// Character::BeginPlay()
	InventoryWidget = CreateWidget<UInventoryWidget>(GetWorld(), InventoryWidgetClass);
```

#### 4. 교훈 및 예방 조치 (Lessons Learned & Prevention)
< C++ 기반의 UMG 작업을 할 때 BindWidget이 작동하지 않을 경우 체크리스트 >
1. 리부모(Reparenting): 위젯 블루프린트의 부모가 C++ 클래스인지
2. 변수 이름: 디자이너의 'Is Variable?'이 체크되고, 'Variable Name'이 C++ 변수명과 일치하는지
3. 타입 일치: C++ 변수 타입과 UMG 위젯 타입이 일치하는지
4. 위젯 인스턴스: CreateWidget로 해당 위젯의 인스턴스를 만들고 있는지
