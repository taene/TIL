## Smart Pointer
- 스마트 포인터: 객체의 생명 주기를 자동으로 관리해주는 클래스 템플릿
- 핵심 원리: RAII 패턴("자원 획득은 초기화다" 라는 뜻)
- 객체가 생성될 때 자원(메모리 등)을 얻고, 객체가 범위를 벗어나 소멸될 때 자동으로 자원을 해제하도록 함

### 스마트 포인터 사용 이유
1. 메모리 누수(Memory Leak): new로 할당된 메모리를 delete 하는 것을 잊어서 발생하는 문제를 방지함
2. 댕글링 포인터(Dangling Pointer): 이미 해제된 메모리를 가리키는 포인터를 사용하는 위험을 줄여줌
3. 복잡한 자원 관리 단순화: 수동으로 new와 delete를 관리하는 것보다 코드가 간결해지고 안정성이 높아짐(예외 발생 시에도 자원 해제를 보장해줌)

<br/>

## C++ 스마트 포인터 종류 및 사용법
### 🔍 std::unique_ptr (독점적 소유권)
- 하나의 스마트 포인터만이 특정 객체를 독점적으로 소유함(unique_ptr이 소멸되거나 다른 객체를 가리키도록 재설정되면, 이전에 가리키던 객체는 자동으로 메모리에서 해제됨)

#### 특징
  - 복사가 불가능하고, 이동만 가능함
  - 소유권이 이전되면 기존 unique_ptr은 더 이상 해당 객체를 소유하지 않음(nullptr 상태가 됨)
  - 가장 가볍고 성능 오버헤드가 적은 스마트 포인터
  - 배열도 관리할 수 있음(std::unique_ptr<T[]>)

#### 생성
  - std::make_unique를 사용해 생성하는 것을 권장함(예외 안전성 보장, 간결해지는 코드)
```cpp
#include <iostream>
#include <memory> // 스마트 포인터 헤더
#include <vector>

class MyClass {
public:
    MyClass(int val) : value(val) {
        std::cout << "MyClass(" << value << ") 생성됨" << std::endl;
    }
    ~MyClass() {
        std::cout << "MyClass(" << value << ") 소멸됨" << std::endl;
    }
    void PrintValue() const {
        std::cout << "Value: " << value << std::endl;
    }
private:
    int value;
};

// 팩토리 함수 예시
std::unique_ptr<MyClass> CreateMyClass(int val) {
    return std::make_unique<MyClass>(val); // 소유권이 반환됨
}

int main() {
    // 1. std::make_unique 사용 (권장)
    std::unique_ptr<MyClass> u_ptr1 = std::make_unique<MyClass>(10);
    u_ptr1->PrintValue(); // 일반 포인터처럼 -> 연산자 사용 가능

    // 2. new 사용 (C++11)
    // std::unique_ptr<MyClass> u_ptr2(new MyClass(20));
    // u_ptr2->PrintValue();

    // 3. 소유권 이전 (move)
    std::unique_ptr<MyClass> u_ptr3 = std::move(u_ptr1); // u_ptr1은 이제 nullptr
    if (u_ptr1) {
        std::cout << "u_ptr1은 유효함" << std::endl;
    } else {
        std::cout << "u_ptr1은 nullptr임 (소유권 이전됨)" << std::endl;
    }
    u_ptr3->PrintValue();

    // 4. 함수에서 unique_ptr 반환
    std::unique_ptr<MyClass> u_ptr4 = CreateMyClass(30);
    u_ptr4->PrintValue();

    // 5. unique_ptr 재설정
    u_ptr3.reset(new MyClass(40)); // 기존 u_ptr3이 가리키던 객체는 소멸, 새 객체 가리킴
    u_ptr3->PrintValue();

    u_ptr3.reset(); // u_ptr3이 가리키던 객체 소멸, u_ptr3은 nullptr
    if (!u_ptr3) {
        std::cout << "u_ptr3은 reset 후 nullptr임" << std::endl;
    }

    // 6. 배열 관리
    std::unique_ptr<int[]> u_arr_ptr = std::make_unique<int[]>(5); // 크기가 5인 int 배열
    for (int i = 0; i < 5; ++i) {
        u_arr_ptr[i] = i * 10; // 배열처럼 접근
        std::cout << "u_arr_ptr[" << i << "] = " << u_arr_ptr[i] << std::endl;
    }
    // u_arr_ptr이 범위를 벗어나면 배열 메모리 자동 해제

    std::cout << "main 함수 종료 직전" << std::endl;
    // u_ptr1 (nullptr), u_ptr3 (nullptr), u_ptr4, u_arr_ptr 등이 범위를 벗어나면서
    // 관리하던 객체들이 자동으로 소멸됨
    return 0;
}
```

#### 주요 멤버 함수()
- <tt>get()</tt>: 관리하는 객체의 일반 포인터를 반환(⚠️ 이 포인터로 delete 하면 안됨)
- <tt>reset(p)</tt>: 현재 관리하는 객체의 메모리를 해제하고, 새로운 포인터 p가 가리키는 객체를 관리하도록 함(p가 없다면 nullptr을 관리)
- <tt>release()</tt>: 관리하는 객체에 대한 소유권을 포기하고 일반 포인터를 반환함(unique_ptr은 nullptr이 되고 반환된 포인터는 수동 delete 해줘야 함)
- <tt>operator*()</tt>, <tt>operator->()</tt>: 관리하는 객체에 접근

#### 사용 사례
- PImpl(Pointer to Implementation) Idiom: 클래스의 구현 세부 사항을 숨기고 컴파일 의존성을 줄일 때
- 팩토리 함수: 객체를 생성하고 그 소유권을 호출자에게 넘겨줄 때
- 단일 소유권이 명확한 경우: 특정 스코프 내에서만 사용되고 다른 곳과 공유될 필요 없는 동적 객체 관리

<br/>

### 🔍 std::shared_ptr (공유된 소유권, 참조 카운팅)
- 참조 카운팅(reference counting) 방식을 사용해 여러 스마트 포인터가 하나의 객체를 공유할 수 있게 함
- 객체를 가리키는 shared_ptr의 개수를 세는 **제어 블록**(Control Block)이 함께 관리됨

#### 특징
- 복사 가능(복사될 때마다 참조 카운트 증가, shared_ptr이 소멸될 때마다 참조 카운트 감소)
- 참조 카운트가 0이 되면(해당 객체를 가리키는 shared_ptr이 하나도 없게 되면) 객체는 자동으로 메모리에서 해제됨
- unique_ptr보다 약간의 성능 오버헤드가 있음(참조 카운트 관리 때문)
- **순환 참조(Circular Reference)** 문제를 주의해야 함(두 객체가 서로를 shared_ptr로 가리키면 참조 카운트가 **절대 0이 되지 않기 때문에 메모리 누수가 발생**할 수 있음, **이때 std::weak_ptr 사용**)

#### 생성
- std::make_shared를 사용해 생성하는 것을 권장함(객체와 제어 블록을 한 번의 할당으로 생성해 효율적이고 예외 안전성을 높임)

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Resource {
public:
    Resource(int id) : id_(id) { std::cout << "Resource " << id_ << " 생성됨" << std::endl; }
    ~Resource() { std::cout << "Resource " << id_ << " 소멸됨" << std::endl; }
    void Use() { std::cout << "Resource " << id_ << " 사용 중" << std::endl; }
private:
    int id_;
};

int main() {
    // 1. std::make_shared 사용 (권장)
    std::shared_ptr<Resource> s_ptr1 = std::make_shared<Resource>(1);
    std::cout << "s_ptr1 참조 카운트: " << s_ptr1.use_count() << std::endl; // 1

    s_ptr1->Use();

    {
        std::shared_ptr<Resource> s_ptr2 = s_ptr1; // 복사, 참조 카운트 증가
        std::cout << "s_ptr2 생성 후 s_ptr1 참조 카운트: " << s_ptr1.use_count() << std::endl; // 2
        std::cout << "s_ptr2 생성 후 s_ptr2 참조 카운트: " << s_ptr2.use_count() << std::endl; // 2
        s_ptr2->Use();
    } // s_ptr2가 범위를 벗어나면서 소멸, 참조 카운트 감소

    std::cout << "s_ptr2 소멸 후 s_ptr1 참조 카운트: " << s_ptr1.use_count() << std::endl; // 1

    // 2. 컨테이너에 저장
    std::vector<std::shared_ptr<Resource>> resources;
    resources.push_back(s_ptr1); // 복사, 참조 카운트 증가
    std::cout << "벡터에 추가 후 s_ptr1 참조 카운트: " << s_ptr1.use_count() << std::endl; // 2
    
    resources.push_back(std::make_shared<Resource>(2)); // 새 Resource 객체, 참조 카운트 1
    std::cout << "새 Resource 추가 후 벡터 마지막 요소 참조 카운트: " << resources.back().use_count() << std::endl; // 1

    // s_ptr1이 관리하는 Resource 객체는 s_ptr1과 resources[0] 두 곳에서 참조 중
    
    s_ptr1.reset(); // s_ptr1은 더 이상 Resource 1을 가리키지 않음. 참조 카운트 감소
    std::cout << "s_ptr1 reset 후 resources[0] 참조 카운트: " << resources[0].use_count() << std::endl; // 1

    std::cout << "main 함수 종료 직전" << std::endl;
    // resources 벡터가 소멸되면서 내부의 shared_ptr들이 소멸되고,
    // 각 Resource 객체의 참조 카운트가 0이 되면 자동으로 소멸됨
    return 0;
}
```

#### 주요 멤버 함수()
- <tt>get()</tt>: 관리하는 객체의 일반 포인터를 반환(⚠️ 이 포인터로 delete 하면 안됨)
- <tt>reset(p)</tt>: 현재 관리하는 객체의 메모리를 해제하고, 새로운 포인터 p가 가리키는 객체를 관리하도록 함(p가 없다면 nullptr을 관리)
- <tt>use_count()</tt>: 현재 객체를 공유하고 있는 shared_ptr의 개수(참조 카운트)를 반환함(디버깅 용도)
- <tt>operator*()</tt>, <tt>operator->()</tt>: 관리하는 객체에 접근

#### 사용 사례
- 공유 자원 관리: 여러 객체가 하나의 자원을 공유하고, 자원의 생명 주기가 그 객체들 중 누구 하나에 종속되지 않을 때
- 콜백 함수나 비동기 작업: 객체가 콜백 함수 실행 완료 or 비동기 작업 완료 시점까지 살아 있어야 할 때
- 자료 구조: 그래프나 트리 구조에서 노드들이 서로를 참조할 때(but, 순환 참조 주의)

<br/>

### 🔍 std::weak_ptr (shared_ptr로 관리되는 객체에 대한 비소유적(약한) 참조)
- shared_ptr이 관리하는 객체에 대한 비소유적(non-owning) 참조를 제공함
- 즉, weak_ptr은 객체의 생명 주기에 영향을 주지 않음(참조 카운트 증가 x)

#### 특징
- shared_ptr로부터 생성됨
- 직접적으로 객체에 접근할 수 없음(객체에 접근하려면 lock() 멤버 함수를 호출해 유효한 shared_ptr를 얻어야 함
- lock(): 만약 원본 객체가 이미 소멸됐다면, nullptr을 가리키는 shared_ptr을 반환함(댕글링 포인터 문제를 안전하게 회피함)
- 주요 용도: shared_ptr의 **순환 참조 문제 해결** 및 **객체의 존재 여부 확인**

#### 생성
```cpp
#include <iostream>
#include <memory>
#include <string>

struct Child; // 전방 선언

struct Parent {
    std::string name;
    // std::shared_ptr<Child> child; // 이렇게 하면 순환 참조 발생 가능성
    std::weak_ptr<Child> child;   // weak_ptr로 변경하여 순환 참조 방지

    Parent(const std::string& n) : name(n) {
        std::cout << "Parent " << name << " 생성됨" << std::endl;
    }
    ~Parent() {
        std::cout << "Parent " << name << " 소멸됨" << std::endl;
    }
};

struct Child {
    std::string name;
    std::shared_ptr<Parent> parent; // 부모는 강한 참조 유지

    Child(const std::string& n) : name(n) {
        std::cout << "Child " << name << " 생성됨" << std::endl;
    }
    ~Child() {
        std::cout << "Child " << name << " 소멸됨" << std::endl;
    }
};

int main() {
    std::shared_ptr<Parent> p = std::make_shared<Parent>("Dad");
    std::shared_ptr<Child> c = std::make_shared<Child>("Son");

    // 상호 참조 설정
    p->child = c; // Parent가 Child를 weak_ptr로 참조
    c->parent = p; // Child가 Parent를 shared_ptr로 참조

    std::cout << "Parent의 child 참조 카운트 (weak_ptr는 영향 없음): " << c.use_count() << std::endl; // c의 참조 카운트는 1 (main의 c)
    std::cout << "Child의 parent 참조 카운트: " << p.use_count() << std::endl; // p의 참조 카운트는 2 (main의 p, c->parent)

    // Parent에서 Child 사용 예시
    if (std::shared_ptr<Child> locked_child = p->child.lock()) { // weak_ptr로부터 shared_ptr 얻기
        std::cout << p->name << "의 자식은 " << locked_child->name << " (유효함)" << std::endl;
    } else {
        std::cout << p->name << "의 자식은 이미 소멸됨" << std::endl;
    }

    // c.reset(); // Child 객체를 먼저 소멸시켜보자 (main의 c 소멸)
    // 이제 Child 객체는 Parent의 child (weak_ptr)만 가리키므로, 실제로는 참조 카운트 0.
    // 하지만 Parent 객체는 아직 Child의 parent (shared_ptr)가 가리키고 있음.

    // 만약 Parent의 child가 shared_ptr였다면, p와 c가 서로를 가리켜서
    // main 함수가 끝나도 둘 다 소멸되지 않는 순환 참조가 발생했을 것!
    // weak_ptr를 사용함으로써 Parent가 Child의 생명 주기를 연장시키지 않는다.

    std::cout << "main 함수 종료 직전" << std::endl;
    // p와 c가 범위를 벗어나면, 참조 카운트에 따라 Parent와 Child 객체가 순서대로 잘 소멸됨.
    // (만약 Parent가 Child를 shared_ptr로 가졌고, Child도 Parent를 shared_ptr로 가졌다면 둘 다 소멸 안됨)
    return 0;
}
```

#### 주요 멤버 함수()
- <tt>lock()</tt>: 관리되는 객체에 대한 shared_ptr 반환(소멸되어 있으면 nullptr을 가리키는 shared_ptr 반환)
- <tt>expired()</tt>: 관리되는 객체가 이미 소멸되었는지 여부를 bool 값으로 반환(lock().get() == nullptr과 유사함)
- <tt>reset()</tt>: weak_ptr를 비우고 아무것도 가리키지 않도록 함

#### 사용 사례
- 순환 참조 해결: 위 예시처럼 부모-자식 관계 등에서 한쪽 또는 양쪽을 weak_ptr로 만들어 순환 고리를 끊을 때
- 캐시(Cache) 구현: 객체의 존재를 감시하지만, 객체의 생명 주기를 연장하고 싶지 않을 때(캐시에서 객체에 대한 weak_ptr를 저장하고, 필요시 lock()으로 shared_ptr를 얻어 사용함)
- 옵저버 패턴(Observer Pattern): Subject(주체 객체)가 Observer를 weak_ptr로 참조하여, Observer가 먼저 소멸돼도 Subject가 안전하게 동작하도록 할 때

<br/>
