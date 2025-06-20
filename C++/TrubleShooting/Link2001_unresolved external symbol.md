## 학습 키워드

  - `static 멤버 변수`
  - `선언과 정의 (Declaration and Definition)`
  - `inline`
  - `링커 에러 (Linker Error)`
  - `unresolved external symbol (LNK2001)`
  - `ODR (One Definition Rule)`
  - `불완전한 형식 (Incomplete Type)`
  - `싱글톤 패턴 (Singleton Pattern)`
  - `(언리얼 엔진) 모듈 의존성`

<br/>

## 학습 내용

#### 1. 선언과 정의의 분리
- C++에서 코드는 '선언(Declaration)'과 '정의(Definition)'로 나뉜다.
- **선언**은 컴파일러에게 "이런 게 있을 거야"라고 **알려주는 약속**이며, **정의**는 "이게 바로 그것의 실체야"라고 **메모리를 할당하고 구현하는 과정**이다.

<br/>

#### 2. `static` 멤버 변수의 정의
- `static` 멤버 변수는 클래스 내부에 선언되지만, 실제 메모리 할당을 위한 정의는 클래스 외부(`.cpp` 파일)에 단 한 번만 이루어져야 한다.
- 이는 프로그램 전체에서 단 하나의 실체만 존재해야 한다는 ODR(One Definition Rule)을 따르기 때문이다.
- 이 정의가 없으면 링커가 변수의 실체를 찾지 못해 `unresolved external symbol` 에러를 발생시킨다.

<br/>

#### 3. `inline` 변수
```cpp
// MyClass.h (C++17 이상)
class MyClass
{
public:
    // 선언과 동시에 정의 (inline 키워드 덕분)
    inline static int myStaticVar = 0;
};

// .cpp 파일에 별도의 정의가 필요 없음!
```
- C++17의 `inline static`은 선언과 동시에 정의가 가능하게 해 편리하다.
- ⚠️ 예외: 클래스가 자기 자신을 가리키는 `static` 포인터 멤버를 가질 때(싱글톤 패턴 등), 클래스 정의가 완료되지 않은 '불완전한 형식' 상태에서 멤버를 정의하려 하면 컴파일 에러가 발생하고, 이 때는 전통적인 분리 방식을 사용해야 한다.

<br/>

#### 4. `unresolved external symbol` 에러
- '약속(선언)은 있는데 실체(정의)를 찾을 수 없다'는 의미다.
#### <원인>
  1. 단순 정의 누락: 가장 흔한 원인, 함수나 static 변수를 헤더(.h)에 선언만 하고, 소스 파일(.cpp)에서 구현이나 초기화를 하지 않았다.
    - 함수: .cpp 파일에 함수의 본문 {}을 구현한다.
    - static 멤버 변수: .cpp 파일에 `타입 클래스명::변수명 = 초기값;` 형태로 정의 코드를 추가한다.
  2. 템플릿 정의 문제: 템플릿 클래스나 템플릿 함수의 정의(구현)를 .cpp 파일에 작성했다.
    - 템플릿은 컴파일 타임에 코드가 생성되어야 하므로, 템플릿을 사용하는 모든 곳에서 구현 코드를 볼 수 있도록 헤더 파일에 정의까지 함께 작성해야 한다.
  3. 함수 시그니처 불일치/부모의 순수가상함수 미구현: 선언부와 정의부의 함수 시그니처가 다르다. 시그니처란 함수 이름, 매개변수의 타입/개수/순서, const 여부 등을 모두 포함한다. 링커는 이들을 모두 다른 함수로 취급한다.
  4. 라이브러리 설정 누락: 외부 라이브러리나 다른 프로젝트의 기능을 사용할 때 발생, 링커에게 해당 라이브러리 파일(.lib)의 위치나 이름을 알려주지 않았다.
    - Visual Studio: [프로젝트 속성] → [링커] → [입력] → [추가 종속성]에 필요한 .lib 파일을 추가한다. 그래도 못 찾는다면 [링커] → [일반] → [추가 라이브러리 디렉터리]에 .lib 파일이 있는 폴더 경로를 추가한다.
  5. 언리얼 모듈 설정 누락: 언리얼 엔진에서 매우 흔한 원인. 다른 모듈(예: Slate, AIModule, GameplayTasks 등)의 기능을 사용하면서, 내 모듈의 Build.cs 파일에 해당 모듈을 추가하지 않았다.
    - 프로젝트의 Source/모듈명/모듈명.Build.cs 파일을 연다. PublicDependencyModuleNames 또는 PrivateDependencyModuleNames에 사용하려는 기능이 포함된 모듈의 이름을 문자열로 추가한다. (예: "Slate", "SlateCore")

<br/>

### 구현 과정

1.  **`static` 멤버 변수의 올바른 사용**

      - `static` 변수는 클래스 외부에서 반드시 정의 및 초기화가 필요하다.

    ```cpp
    // MyObject.h
    class MyObject
    {
    public:
        // 1. 선언 (Declaration)
        static int objectCount;
    };

    // MyObject.cpp
    // 2. 정의 (Definition) - 이 부분이 없으면 링크 에러 발생
    int MyObject::objectCount = 0;
    ```

2.  **`inline static`의 예외 상황 (Incomplete Type)**

      - 자기 자신을 가리키는 `static` 포인터는 클래스 내부에서 `inline`으로 정의할 수 없다.

    ```cpp
    // Character.h - 싱글톤 인스턴스 포인터
    class Character
    {
    private:
        // 'inline static Character* instance = nullptr;' 처럼
        // 내부에서 바로 정의하면 '불완전한 형식' 문제로 컴파일 에러가 남.
        
        // 올바른 방법: 선언만 남긴다.
        static Character* instance; 
    };

    // Character.cpp
    // 외부에서 정의한다.
    Character* Character::instance = nullptr;
    ```

<br/>

### LINK2001 에러 발생 시 체크리스트
1. 에러 메시지 정확히 읽기: 어떤 **심볼 이름**을 못 찾겠다고 하는지 확인한다. 길고 복잡하게 깨진 이름이라면 함수일 가능성이 높다.
2. static: static 변수의 선언과 정의가 되어있는지 확인한다.
3. template: 선언과 정의가 .h 파일에 한번에 되어있는지 확인한다.
4. 정의(구현) 코드 확인: 그 이름에 해당하는 함수나 변수의 정의가 프로젝트 내에 정말 존재하는지 검색(Ctrl+F 또는 Ctrl+Shift+F)해 본다.
5. 오타 및 시그니처 확인: 선언과 정의의 이름, 파라미터, const 등이 100% 일치하는지 눈을 크게 뜨고 비교한다.
6. 외부 라이브러리인가?/언리얼 엔진인가?: 외부 라이브러리 함수라면 프로젝트 링커 설정에 .lib 파일이 제대로 추가되었는지 확인하고, 언리얼 엔진이라면 가장 먼저 Build.cs 파일의 모듈 의존성부터 의심한다.

## 느낀점

  - 컴파일 에러는 주로 문법 문제인 반면, 링커 에러는 코드 조각들을 합치는 과정에서 발생하는 연결의 문제라는 것을 명확히 이해했다.
  - `unresolved external symbol` 에러는 처음엔 당황스럽지만, 결국 '선언과 정의가 일치하는가?', '필요한 부품(라이브러리, 모듈)을 알려주었는가?' 라는 두 가지 질문으로 귀결된다.

