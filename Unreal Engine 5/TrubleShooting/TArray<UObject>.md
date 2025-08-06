# TArray<UObject>는 왜 동작하지 않을까? - UObject와 GC의 원리 이해
> 언리얼 C++의 철칙: UObject는 반드시 포인터로 다루기

<br/>

## 학습 키워드
- UObject
- 가비지 컬렉션 (Garbage Collection, GC)
- TArray
- TObjectPtr
- GetComponents
- 포인터 기반 관리 (Pointer-based Management)

## 학습 내용
### 배운 개념 요약
- 언리얼 엔진에서 `UObject`를 상속받는 모든 클래스(예: `AActor`, `UActorComponent`)는 값(value)으로 다룰 수 없으며, 반드시 **포인터**로 관리해야 한다.
- `TArray<UStatComponent>`와 같이 `UObject` 파생 객체를 값으로 저장하려는 시도는 컴파일 오류나 런타임 크래시를 유발한다.
- 그 이유는 언리얼 엔진의 **가비지 컬렉션(GC)** 시스템이 `UObject` 객체의 생명 주기를 포인터를 통해 추적하고 관리하기 때문이다. 값으로 복사된 객체는 GC의 추적 범위를 벗어나 메모리 문제를 일으킨다.
- 따라서 `UObject`의 배열은 `TArray<TObjectPtr<UStatComponent>>` 또는 `TArray<UStatComponent*>`와 같이 포인터를 저장하는 형태로 선언해야 한다. `TObjectPtr`은 dangling 포인터 문제 등을 방지해주는 더 안전하고 현대적인 방식이다.

<br/>

### 구현 과정
잘못된 코드와 올바른 코드의 비교를 통해 올바른 `UObject` 관리 방식을 익혔다.

1.  **잘못된 선언 (컴파일 오류 발생)**
    `UObject` 파생 클래스를 값으로 저장하려는 시도이다.

    ```cpp
    // UStatComponent 객체 자체를 배열에 담으려고 해서 오류 발생
    TArray<UStatComponent> StatComponents;
    ```

2.  **올바른 선언**
    `UStatComponent`에 대한 포인터를 담는 배열로 선언한다.

    ```cpp
    // UStatComponent에 대한 포인터를 담는 TArray
    TArray<TObjectPtr<UStatComponent>> StatComponents;
    ```

3.  **배열 데이터 채우기 및 순회**
    선언된 포인터 배열에 실제 컴포넌트들의 주소를 채워 넣고 사용하는 방법이다.

    * **특정 액터에서 컴포넌트 가져오기**
        액터의 `GetComponents` 함수를 사용하는 것이 가장 일반적이다.

        ```cpp
        // 이 액터에 부착된 모든 UStatComponent의 포인터를 배열에 채운다.
        TArray<TObjectPtr<UStatComponent>> FoundStatComponents;
        this->GetComponents<UStatComponent>(FoundStatComponents);
        
        // 배열을 순회하며 각 컴포넌트에 접근
        for (const TObjectPtr<UStatComponent>& StatComp : FoundStatComponents)
        {
            // 포인터가 유효한지(nullptr이 아닌지) 항상 확인한다.
            if (StatComp)
            {
                // 포인터를 통해 멤버 함수나 변수에 접근한다.
                StatComp->InitializeStats(); 
            }
        }
        ```

<br/>

## 느낀점
- 표준 C++에서는 객체를 값으로 컨테이너에 저장하는 것이 일반적이지만, 언리얼 엔진의 메모리 관리 시스템(GC) 때문에 `UObject`는 전혀 다른 규칙을 따른다는 것을 명확히 알게 되었다. 이 차이점을 이해하는 것이 언리얼 C++의 첫걸음이라 느껴진다.
- `TArray<UStatComponent>` 라는 실수는 단순히 문법의 문제가 아니라, 언리얼 엔진의 핵심 철학인 GC 시스템을 이해하지 못해 발생한 개념적인 문제였다.
- 왜 `UObject`를 포인터로 다뤄야 하는지에 대한 근본적인 이유를 알게 되니, 앞으로 비슷한 실수를 하지 않을 자신감이 생겼다. 또한 모든 포인터 변수는 사용 전에 `if (Pointer)`로 유효성을 검사하는 습관이 매우 중요하다는 것을 다시 한번 깨달았다.
- 라이라 프로젝트에서 `GetAllActorsOfClass` 같은 전역 검색 함수의 사용을 지양하고 서브시스템 등을 통해 객체를 관리하는 이유도 성능 문제와 직결된다는 점을 알게 되었다. 당장의 기능 구현뿐만 아니라, 프로젝트의 규모가 커졌을 때의 확장성과 성능까지 고려하는 시야를 길러야겠다.
