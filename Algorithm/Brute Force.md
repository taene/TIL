- 무식하게 풀기(Brute Force): 컴퓨터의 빠른 계산 능력을 이용해 가능한 경우의 수를 일일이 나열하면서 답을 찾는 방법
- 완전 탐색(exhaustive search): 가능한 방법을 전부 만들어 보는 알고리즘들

# 재귀 호출과 완전 탐색
- 작업의 범위를 작게 쪼개면, 각 쪼개진 조각들의 형태가 유사해지는 작업들을 많이 볼 수 있다.
- 재귀 함수(recursive function): 자신이 수행할 작업을 유사한 형태의 여러 조각으로 쪼갠 뒤 그 중 한 조각을 수행하고, 나머지를 자기 자신을 호출해 실행하는 함수
- 모든 재귀 함수는 ‘더이상 쪼개지지 않는’ 최소한의 작업에 도달했을 때 답을 곧장 반환하는 조건문을 포함해야 한다. 이때 쪼개지지 않는 가장 작은 작업들을 재귀 호출의 **기저 사례**(base case)라고 한다.
- 기저 사례를 선택할 때는 존재하는 모든 입력이 항상 기저 사례의 답을 이용해 계산될 수 있도록 해야 한다.

## 예제: 중첩 반복문 대체하기
- 0~n개의 원소 중 4개를 고르는 모든 경우를 출력하는 코드 (조합, nC4)
```cpp
for(int i=0; i<n; ++i)
	for(int j=i+1; j<n; ++j)
		for(int k=j+1; k<n; ++k)
			for(int l=k+1; l<n; ++l)
				cout << i << " " << j << " " << k << " " << l << endl;
```
- Flow
	- 위 코드 조각이 하는 작업을 4개의 조각으로 나눔 -> 각 조각에서 하나의 원소를 고르는 것!
	- 원소를 고른 뒤, 남은 원소들을 고르는 작업을 자기 자신을 호출해 떠넘기는 재귀 함수를 작성해보자
	- 이때 남은 원소들을 고르는 작업을, 다음과 같은 입력들의 집합으로 정의할 수 있다.
		- 원소들의 총 개수
		- 더 골라야 할 원소들의 개수
		- 지금까지 고른 원소들의 번호
```cpp
// n: 원소들의 총 개수
// picked: 지금까지 고른 원소들의 번호
// toPick: 더 골라야 할 원소들의 개수

// 앞으로 toPick개의 원소를 고르는 모든 방법을 출력
void pick(int n, vector<int>& picked, int toPick)
{
	// 기저 사례: 더 고를 원소가 없을 때 고른 원소들을 출력한다
	if(toPick==0) { printPicked(picked); return; }
	// 고를 수 있는 가장 작은 번호를 계산한다
	int smallest = picked.empty() ? 0 : picked.back() + 1;
	// 이 단계에서 원소 하나를 고른다.
	for(int next = smallest; next < n; ++next)
	{
		picked.push_back(next);
		pick(n, picked, toPick - 1);
		picked.pop_back();
	}
}
```

## 예제: 보글 게임
- 게임 목적: 상하좌우/대각선으로 인접한 칸들의 글자들을 이어서 단어를 찾아내는 것
	- 각 글자들은 대각선으로도 이어질 수 있으며, 한 글자가 두 번 이상 사용될 수도 있다.
- hasWord(y, x, word) = 보글 게임판의 (y, x)애서 시작하는 단어 word의 존재 여부를 반환한다.
### 기저 사례의 선택
- 더 이상의 탐색 없이 간단히 답을 낼 수 있는 경우들을 기저 사례로 선택한다.
	1. 위치 (y, x)에 있는 글자가 원하는 단어의 첫 글자가 아닌 경우 항상 실패
	2. (1번에 해당하지 않는 경우) 원하는 단어가 한 글자인 경우 항상 성공
- 입력이 잘못되거나 범위에서 벗어난 경우도 기저 사례로 택해서 맨 처음에 처리하기!

### 구현
```cpp
const int dx[8] = {-1, -1, -1, 1, 1, 1, 0, 0};
const int dy[8] = {-1, 0, 1, -1, 0, 1, -1, 1};

bool hasWord(int y, int x, const string& word)
{
	if(!intRange(y,x)) return false; // 기저사례1: 시작 위치가 범위 밖이면 무조건 실패
	if(board[y][x] != word[0]) return false; // 2: 첫 글자가 일치하지 않으면 실패
	if(word.size() == 1) return true; // 3: 단어 길이가 1이면 성공
	
	for(int direction=0; direction<8; ++direction)
	{
		int nextY = y + dy[direction];
		int nextX = x + dx[direction];
		
		// 다음 칸이 범위 안에 있는지, 첫 글자는 일치하는지 확인할 필요가 없으므로 바로 처리
		if(hasWord(nextY, nextX, word.substr(1)))
		{
			return true;
		}
	}
	return false;
}
```

### 완전 탐색 레시피
- 어떤 문제를 완전 탐색으로 해결하기 위해 필요한 과정
	1. 완전 탐색은 존재하는 모든 답을 하나씩 검사하므로, 걸리는 시간은 가능한 답의 수에 정확히 비례한다. 최대 크기의 입력을 가정했을 때 답의 개수를 계산하고 이들을 모두 제한 시간 안에 생성할 수 있을지를 가늠한다. 만약 시간 안에 계산할 수 없다면, 다른 설계 패러다임을 적용해야 한다.
	2. 가능한 모든 답의 후보를 만드는 과정을 여러 개의 선택으로 나눈다. 각 선택은 답의 후보를 만드는 과정의 한 조각이 된다.
	3. 그중 하나의 조각을 선택해 답의 일부를 만들고, 나머지 답을 재귀 호출을 통해 완성한다.
	4. 조각이 하나밖에 남지 않은 경우, 혹은 하나도 남지 않은 경우에는 답을 생성했으므로, 이것을 기저 사례로 선택해 처리한다.

---

- 문제: N과 N개의 자연수가 주어진다. 여기서 몇개의 숫자를 골라 합을 mod 11을 했을 때 나오는 가장 큰 수를 구하라.
- 입력:
10    
24 35 38 40 49 59 60 67 83 98

```cpp
// 첫번째 풀이
#include <iostream>
#include <vector>

using namespace std;

int n;
vector<int> v;

int ret(int idx, int sum)
{
    if (idx == n)
    {
        return sum % 11;
    }

    return max(ret(idx + 1, sum + v[idx]), ret(idx + 1, sum));
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        int temp;
        cin >> temp;
        v.push_back(temp);
    }

    cout << ret(0, 0) << '\n';

    return 0;
}
```

```cpp
// 다른 풀이
#include <iostream>
#include <vector>

using namespace std;

int n, cnt, ret, temp;
vector<int> v;

void go(int idx, int sum)
{
    if (idx == n)
    {
        ret = max(ret, sum % 11);
        cnt++;
        return;
    }
    go(idx + 1, sum + v[idx]);
    go(idx + 1, sum);
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> temp;
        v.push_back(temp);
    }

    go(0,0);
    cout << ret << '\n';
    cout << cnt << '\n';

    return 0;
}
```
