### ⚠️ while문에서의 getline()
#### 📢 문제 직면 후 사고 흐름
- 처음 getline()을 쓸 때는 문제가 없음
- 이후 getline()이 사용될 때 문제가 발생, 이전의 std::cin >> 입력을 받은 것 때문이라 추정됨
- 그래서 getline() 전에 std::cin.ignore(1)를 사용했지만 해결이 되지 않음(1인 이유는 \n 개행문자가 남는것이므로 1개의 문자를 넘긴다는 뜻)
- 찾아보니, 입력 버퍼에 남아있는 개행 문자(\n)가 원인이 맞았고, getline()을 받기 전에 버퍼에 남아있는 불필요한 문자를 제거해야 한다는 내 생각도 맞았음
- 그럼 어떻게 HOW 해결하는가?

#### 🔍 정확한 원인
- 원인: 입력 버퍼에 남아있는 개행 문자
1. `std::cin >> 변수;`의 동작 방식: 사용자가 `123`을 입력하고 enter키를 누르면, >> 연산자는 `123`만 가져가고 입력 버퍼에 엔터 키에 해당하는 개행 문자(\n)를 남긴다.
2. `std::getline(std::cin, 변수)`의 동작 방식: getline()은 입력 버퍼에서 개행 문자를 만날 때까지 모든 문자를 읽어온다.
  - 따라서, std::cin >> 다음에 std::getline()을 바로 사용하면, getline()은 버퍼에 남아있던 \n을 즉시 발견하고 "읽을 내용이 없다"고 판단하여 빈 문자열을 name 변수에 저장하고 실행을 종료해버린다.

#### ✨ 해결책
- 해결책: getline() 호출 전 입력 버퍼 비우기
  - std::getline()으로 이름을 입력받기 전에, 버퍼에 남아있는 불필요한 문자(특히 \n)를 제거해야 한다. 이때 std::cin.ignore() 함수를 사용하면 된다.
  - `std::cin.ignore()`: 입력 버퍼에서 문자를 무시하고 버리는 함수
  - `std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');`: 입력 버퍼에서 최대 문자 수만큼 문자를 무시하거나, 개행 문자(\n)를 만나면 거기까지 무시하고 종료하라는 의미.(이전에 남아있던 쓰레기 값들을 전부 깔끔하게 비워주는 역할)

```cpp
#include <iostream>
#include <string>
#include <limits> // std::numeric_limits를 위해 필요

void CreateCharacter()
{
    std::string name;
    int classChoice;

    std::cout << "용사님의 직업을 선택해주세요 (1: 전사, 2: 마법사) : ";
    std::cin >> classChoice; // 사용자가 '1' 입력 후 엔터 -> 버퍼에 '\n'이 남음

    // *** 해결책: 여기서 버퍼를 비워준다 ***
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');

    while (true)
    {
        std::cout << "용사님의 이름을 입력해주세요 : " << std::flush;
        std::getline(std::cin, name); // 버퍼가 비워졌으므로 정상적으로 입력을 기다림

        if (name.empty())
        {
            std::cout << "이름이 비어 있습니다. 다시 입력해주세요.\n";
            continue;
        }

        // 이름에 공백만 있는 경우도 처리 (선택 사항)
        if (name.find_first_not_of(" \t") == std::string::npos)
        {
            std::cout << "이름이 공백으로만 이루어져 있습니다. 다시 입력해주세요.\n";
            continue;
        }
        
        break;
    }

    std::cout << "반갑습니다, " << name << " 용사님!" << std::endl;
}

int main()
{
    CreateCharacter();
    return 0;
}
```
