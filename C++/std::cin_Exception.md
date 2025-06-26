## 학습 키워드

  - `std::cin`
  - `cin.fail()`
  - `cin.clear()`
  - `cin.ignore()`
  - C++ 입력 스트림
  - 입력 버퍼
  - 예외 처리

<br/>

## 학습 내용

### 배운 개념 요약

  - `std::cin`은 내부적으로 스트림의 상태를 나타내는 '상태 플래그'를 가지며, 잘못된 입력 시 `failbit`가 설정되어 더 이상 입력을 받지 않는 '좀비' 상태가 된다.
  - `cin.fail()`: `failbit`나 `badbit`가 설정되었는지 확인하여 입력 실패 여부를 `true`/`false`로 반환한다.
  - `cin.clear()`: `failbit` 등 오류 상태 플래그를 `goodbit`으로 초기화하여, `cin`이 다시 입력을 받을 수 있는 상태로 만든다.
  - `cin.ignore()`: 입력 버퍼에 남아있는 잘못된 문자들을 제거한다. `cin.clear()`만으로는 버퍼가 비워지지 않으므로, 이 함수를 통해 잘못된 입력을 비워주지 않으면 무한 루프에 빠질 수 있다. 보통 `cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n')` 형태로 사용하여 한 줄 전체를 비운다.
  - 이 세 함수를 조합하면 사용자가 예상치 못한 값을 입력하더라도 프로그램이 비정상 종료되거나 무한 루프에 빠지는 것을 막고, 견고한 입력 로직을 만들 수 있다.

### 구현 과정

  - 아래는 `cin.fail()`, `cin.clear()`, `cin.ignore()`를 모두 사용하여 사용자가 올바른 정수 값을 입력할 때까지 입력을 반복해서 요청하는 코드다.

```cpp
#include <iostream>
#include <limits> // std::numeric_limits를 사용하기 위해 필요

int main()
{
    int number;

    while (true)
    {
        std::cout << "숫자를 입력하세요: ";
        std::cin >> number;

        // 1. 입력 실패 감지
        if (std::cin.fail())
        {
            std::cout << "오류: 숫자가 아닌 값이 입력되었습니다." << std::endl;
            
            // 2. cin의 오류 상태 플래그 초기화
            std::cin.clear();
            
            // 3. 입력 버퍼에 남아있는 잘못된 입력(개행 문자 포함)을 모두 비움
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
            
            continue; // 다시 입력을 받기 위해 루프의 시작으로 이동
        }
        
        // 추가 검사: "123abc"와 같이 숫자 뒤에 문자가 오는 경우를 처리
        // cin.peek()은 버퍼에서 문자를 제거하지 않고 다음 문자를 확인
        if (std::cin.peek() != '\n' && std::cin.peek() != EOF)
        {
            std::cout << "오류: 숫자 뒤에 다른 문자가 있습니다." << std::endl;
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
            continue;
        }

        break; // 올바른 입력을 받았으므로 루프 종료
    }

    std::cout << "정상적으로 입력된 숫자: " << number << std::endl;

    return 0;
}
```

<br/>

## 느낀점

  - 단순히 `std::cin >> variable;`만 사용했을 때 왜 문자를 입력하면 프로그램이 꼬이는지 근본적인 이유를 알게 되었다. `cin`이 단순한 함수가 아니라 내부 상태를 관리하는 객체라는 점이 흥미로웠다.
  - `fail()`로 오류를 감지하고, `clear()`로 상태를 되돌린 뒤, `ignore()`로 쓰레기 값을 버리는 3단계 프로세스는 반드시 기억해야 할 중요한 패턴이다.
  - 특히 `cin.ignore()`를 사용하지 않으면 무한 루프에 빠지기 쉽다는 것을 실습을 통해 체감했다. 안정적인 프로그램을 만들기 위해서는 이런 예외 상황 처리가 필수적이라는 것을 다시 한번 깨달았다. `cin.peek()`을 이용해 입력 버퍼를 추가로 검사하는 기법도 유용하게 사용할 수 있을 것 같다.

### 요약

| 함수 | 기능 | 목적 |
|---|---|---|
| `cin.fail()` | 입력 스트림의 오류 상태 확인 | 잘못된 입력 감지 |
| `cin.clear()` | 오류 상태 플래그 초기화 | `cin` 재활성화 |
| `cin.ignore()` | 입력 버퍼의 문자 제거 | 잘못된 입력 값 폐기 및 무한 루프 방지 |
