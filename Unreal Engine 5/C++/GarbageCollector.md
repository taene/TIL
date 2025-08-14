## 학습 키워드

- 언리얼 엔진
- 메모리 관리
- 가비지 컬렉터(GC)
- UObject 아우터
- UPROPERTY
- 강한 참조, 약한 참조, 순환 참조
- TWeakObjectPtr

## 학습 내용

### 배운 개념 요약

- 언리얼 엔진의 자동 메모리 관리 시스템인 가비지 컬렉터(GC)와 그 핵심 원리를 학습했다. 
- 일반 C++의 `new`/`delete` 방식이 유발하는 메모리 누수, 이중 해제, 댕글링 포인터 등의 문제를 언리얼은 어떻게 해결하는지 이해했다.

  * **가비지 컬렉터 (GC)의 3대 원칙**

    1.  **`UObject` 상속**: GC는 `UObject`를 상속하는 클래스만 관리한다.
    2.  **`UPROPERTY()` 표기**: `UObject`에 대한 참조는 반드시 `UPROPERTY()` 매크로로 감싸 GC에게 "중요한 참조"임을 알려야 한다.
    3.  **`delete` 금지**: `UObject`는 절대 `delete`로 직접 파괴하면 안 된다. 대신 포인터에 `nullptr`를 대입하여 참조를 끊으면 GC가 알아서 수거한다.

  * **`UObject`와 Outer**

      * **`UObject`**: 단순한 부모 클래스가 아닌, 리플렉션, 직렬화, 에디터 통합 등 언리얼의 핵심 기능을 제공하는 기반이다. 라이라(Lyra)와 같은 프로젝트는 `UDataAsset`을 활용해 게임 로직을 데이터 중심으로 설계하는데, 이 모든 것이 `UObject` 덕분에 가능하다.
      * **아우터(Outer)**: 객체의 "소유자"를 지정하는 개념이다. **"주인이 사라지면 소유물도 함께 사라진다"** 는 원칙에 따라 메모리가 연쇄적으로 자동 정리된다. `NewObject<T>(Outer)` 호출 시 아우터를 명확히 지정하는 것이 중요하다.

  * **참조 유형과 순환 참조**

      * **강한 참조 (Hard Reference)**: 일반 `UObject*` 포인터. "소유"의 관계를 나타내며, 이 참조가 남아있는 한 객체는 GC되지 않는다.
      * **약한 참조 (Weak Reference)**: `TWeakObjectPtr<T>` 템플릿. "관찰"의 관계를 나타내며, GC 결정에 영향을 주지 않는다. 참조 대상이 사라지면 자동으로 `nullptr`처럼 동작하므로, 사용 전 `IsValid()` 체크가 필수다.
      * **순환 참조**: 두 객체가 서로를 강한 참조로 붙잡아 메모리 누수를 일으키는 주범이다. 부모-자식 관계에서 자식이 부모를 참조할 때 약한 참조를 사용해 이 순환 고리를 끊어야 한다.

<br/>

### 구현 과정

- 순환 참조 문제를 해결하는 과정을 코드로 확인했다.
- 두 클래스가 서로를 `UPROPERTY()`로 참조하면 메모리 누수가 발생한다.

**1. 문제의 코드 (순환 참조)**

```cpp
// Parent.h
UCLASS()
class UParent : public UObject
{
    GENERATED_BODY()

public:
    // Parent가 Child를 강하게 참조 (소유)
    UPROPERTY()
    TObjectPtr<class UChild> Child;
};

// Child.h
UCLASS()
class UChild : public UObject
{
    GENERATED_BODY()

public:
    // Child도 Parent를 강하게 참조 (문제 발생!)
    UPROPERTY()
    TObjectPtr<class UParent> Parent; 
};
```

이 구조에서는 `Parent`와 `Child` 객체에 대한 외부 참조가 모두 사라져도, 둘이 서로를 붙잡고 있기 때문에 GC가 메모리에서 해제하지 못한다.

**2. 해결책 (약한 참조 사용)**
부모-자식 관계에서, 소유권을 가지는 부모가 자식을 **강한 참조**로, 소유의 대상인 자식은 부모를 **약한 참조**로 관찰하게 하여 순환 고리를 끊는다.

```cpp
// Parent.h (변경 없음)
UCLASS()
class UParent : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY()
    TObjectPtr<class UChild> Child;
};

// Child.h (수정된 코드)
UCLASS()
class UChild : public UObject
{
    GENERATED_BODY()

public:
    // Child가 Parent를 약하게 참조 (관찰)
    UPROPERTY()
    TWeakObjectPtr<class UParent> Parent;

    void DoSomethingWithParent()
    {
        // 약한 참조는 항상 유효성 검사 후 사용한다.
        if (Parent.IsValid())
        {
            UParent* MyParent = Parent.Get();
            // ... 안전하게 Parent 객체 사용
        }
    }
};
```

`UChild`의 `Parent` 참조를 `TWeakObjectPtr`로 변경함으로써, `Parent` 객체가 파괴되면 `Child` 객체도 GC 대상이 될 수 있게 되었다. 이것이 언리얼에서 메모리를 안전하게 관리하는 핵심 패턴이다.

<br/>

## 느낀점

- 언리얼의 메모리 관리는 단순히 '어떻게'를 넘어 '왜'를 이해하는 것이 중요했다.
- `UPROPERTY`를 붙이는 것은 문법이 아니라 GC와의 약속이며, `Outer`는 객체지향 설계를 메모리 생명주기와 연결하는 중요한 철학이다.
- 특히 강한 참조와 약한 참조의 구분은 객체 간의 관계를 '소유'와 '관찰'로 명확히 설계하도록 강제한다.
- 이런 원칙들이 모여 라이라(Lyra)와 같은 복잡하고 데이터 중심적인 AAA급 프로젝트의 안정성을 지탱하는 기반이 된다는 것을 깨달았다.
- 이제는 포인터를 사용할 때마다 이 객체의 주인은 누구이고, 이 참조는 소유인가 관찰인가를 먼저 고민하는 습관을 들여야겠다.
