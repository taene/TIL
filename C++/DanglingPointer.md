## Dangling Pointer
- **댕글링 포인터**: 이미 해제되었거나 더이상 유효하지 않은 메모리 위치를 가리키는 포인터
- dangling: 매달려있는, 불안정한(허공에 매달려 있는 것처럼 위험한 상태의 포인터)

<br/>

### ⚠️ 심각한 문제 발생 예시
1. 프로그램 비정상 종료(Crash): 운영체제가 보호된 메모리 영역에 접근하려는 시도를 감지하고 프로그램을 강제 종료시킴.(Segmentation Fault)
2. 데이터 손상(Data Corruption): Dangling Pointer가 가리키는 메모리 위치에 다른 데이터가 이미 할당되었을 수 있기 때문에, 이를 통해 값을 쓰려고 하면 엉뚱한 데이터를 덮어쓰게 돼 프로그램 전체의 데이터가 망가질 수 있음
3. 예측 불가능한 동작(Undefined Behavior): 컴파일러나 실행 환경에 따라 동작이 달라질 수 있음(디버깅이 어려워짐)
4. 보안 취약점: 악의적인 사용자가 Dangling pointer를 이용해 프로그램의 제어 흐름을 바꾸거나 민감한 정보에 접근할 수 있음

<br/>

### 🔍 발생 원인
#### 1. 메모리 해제 후 접근(Deallocation)
```cpp
#include <iostream>

int main()
{
    int* ptr = new int(10); // 동적 메모리 할당
    std::cout << "ptr이 가리키는 값: " << *ptr << std::endl; // 정상: 10

    delete ptr; // 메모리 해제
    // 이제 ptr은 댕글링 포인터가 된다!

    // std::cout << "해제 후 ptr이 가리키는 값: " << *ptr << std::endl; // 매우 위험! 댕글링 포인터 역참조 (Undefined Behavior)
    // *ptr = 20; // 더 위험! 이미 해제된 메모리에 값을 쓰려고 시도

    // 좋은 습관: 해제 후에는 nullptr로 초기화
    ptr = nullptr;

    if (ptr != nullptr) {
        std::cout << "nullptr이 아니라면 ptr이 가리키는 값: " << *ptr << std::endl;
    } else {
        std::cout << "ptr은 nullptr입니다." << std::endl;
    }

    return 0;
}
```
- delete로 동적 할당 메모리를 해제한 뒤, 해당 메모리를 가리키던 포인터를 다시 사용하려고 할 때 발생함

#### 2. 함수 반환 값(Returning Local Variable's Address)
```cpp
#include <iostream>

int* createInteger()
{
    int localVar = 100;
    // return &localVar; // 매우 위험! 지역 변수의 주소를 반환하면 안 된다.
                       // 함수가 종료되면 localVar은 사라지고, 반환된 포인터는 댕글링 포인터가 된다.
    int* heapVar = new int(100); // 동적 할당된 메모리는 함수가 끝나도 유지된다.
    return heapVar; // 이 경우는 괜찮지만, 호출한 쪽에서 delete를 책임져야 한다.
}

int main()
{
    int* danglingPtr = createInteger(); // 만약 createInteger가 지역 변수 주소를 반환했다면, 여기서 danglingPtr은 댕글링 포인터.

    // std::cout << "댕글링 포인터가 가리키는 값: " << *danglingPtr << std::endl; // 위험!
    
    // createInteger가 동적 할당된 메모리의 주소를 반환했다면 아래처럼 사용하고 해제해야 한다.
    if (danglingPtr != nullptr) {
        std::cout << "동적 할당된 변수의 값: " << *danglingPtr << std::endl;
        delete danglingPtr; // 메모리 해제
        danglingPtr = nullptr;
    }
    
    return 0;
}
```
- 함수 내부에 선언된 지역 변수의 주소를 반환할 때, 함수가 종료되면 지역 변수는 스택에서 사라지는데 이때 반환된 포인터는 이미 사라진 메모리 위치를 가리키게 됨

#### 3. 포인터의 유효 범위(Scope)
```cpp
#include <iostream>

int main() {
    int* p1 = new int(50);
    int* p2 = p1; // p1과 p2는 같은 메모리를 가리킨다.

    std::cout << "p1: " << *p1 << ", p2: " << *p2 << std::endl;

    delete p1; // p1이 가리키는 메모리 해제
    p1 = nullptr;

    // 이제 p2는 댕글링 포인터가 된다!
    // std::cout << "해제 후 p2: " << *p2 << std::endl; // 위험!

    return 0;
}
```
- 포인터 변수 자체가 유효 범위를 벗어나는 경우와는 다른 개념임
- 여러 포인터가 같은 메모리를 가리키고 있을 때 하나의 포인터로 메모리를 해제하면 다른 포인터들이 댕글링 포인터가 될 수 있음

<br/>

### 댕글링 포인터 회피와 관리 방법
#### 1. 해제 후 nullptr 할당
```cpp
int* ptr = new int(10);
// ... ptr 사용 ...
delete ptr;
ptr = nullptr; // 이제 ptr은 댕글링 포인터가 아니다!
```

#### 2. 스마트 포인터 활용 ✨👍
- 스마트 포인터: 객체가 소멸될 때 자신이 가리키는 메모리를 자동으로 해제해주는 기능을 가짐(RAII 패턴)
- std::unique_ptr: 특정 객체에 대한 유일한 소유권을 가짐
  - unique_ptr이 소멸되면 가리키던 객체도 자동으로 삭제됨
  - 복사 불가능, 이동만 가능 => 소유권 관리 명확함
- std::shared_ptr: 참조 카운팅(reference counting) 방식을 사용해 여러 shared_ptr이 하나의 객체를 공유할 수 있게 함
  - 객체를 가리키는 shared_ptr의 수가 0이 되면 객체가 자동으로 삭제됨
  - 순환 참조(circular dependency) 문제를 조심해야 함
- std::weak_ptr: shared_ptr과 함께 사용되며, shared_ptr이 관리하는 객체를 가리키지만 소유권을 가지지 않음(참조 카운트에 영향 x)
  - shared_ptr의 순환 참조 문제를 해결하는데 도움을 줌
  - 객체의 유효성을 안전하게 확인할 때 사용됨(lock() 함수를 통해 유효한 shared_ptr를 얻을 수 있음)
- Unreal Engine Smart Pointer: <tt>TSharedPtr</tt>, <tt>TUniquePtr</tt>, <tt>TWeakPtr</tt>

#### 3. 소유권(Ownership) 명확히 하기
- 포인터가 가리키는 메모리에 대한 소유권을 누가 갖는지 명확히 설계하는 것이 중요함
- 항상 "누가 이 메모리를 해제할 책임이 있는가?"를 고민할 것
- 이 문제 또한 스마트 포인터를 활용하면 해결가능

#### 4. RAII(Resource Acquisition Is Initialization) 패턴 활용
- "자원 획득은 초기화다" 라는 뜻
- 객체가 생성될 때 자원을 획득하고 객체가 소멸될 때 자원을 해제하는 디자인 패턴(대표적 예시: 스마트 포인터)

<br/>

### 언리얼 엔진에서의 Dangling Pointer
- 자체 **Garbage Collection**이 있음
  - UObject에 대해서는 delete를 직접 호출할 필요없이, 더이상 참조되지 않는 UObject들을 메모리에서 해제해줌
- ⚠️ UObject가 아닌 일반 C++ 객체나 동적 할당된 데이터에 대해서는 여전히 C++ 메모리 관리 규칙을 따라야 함!
- <tt>TSharedPtr</tt>, <tt>TUniquePtr</tt>, <tt>TWeakPtr</tt>: UObject가 GC에 의해 해제될 가능성이 있을 때, TWeakPtr를 사용하면 해당 객체가 아직 유효한지 확인하고 (IsValid()), 접근할 수 있음

<br/>
