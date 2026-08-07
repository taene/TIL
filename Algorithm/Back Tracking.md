- 완전 탐색(Brute Force)에서 가지치기를 통해 최대한 불필요한 탐색을 피하는 알고리즘

---

- 문제: N과 N개의 자연수가 주어진다. 여기서 몇개의 숫자를 골라 합을 mod 11을 했을 때 나오는 가장 큰 수를 구하라.
- 입력:
10    
24 35 38 40 49 59 60 67 83 98

```cpp
#include <iostream>
#include <vector>

using namespace std;

int n, cnt, ret, temp;
vector<int> v;

void go(int idx, int sum)
{
    if (ret == 10) return; // < Back Tracking!! / mod 연산: 예를 들어 mod 11을 할 때, mod 연산의 결과값은 1 ~ 10으로 최대 결과값은 10이 된다. (mod n => 1 ~ n-1의 결과값을 가짐)
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
- 추가된 Back Tracking 한 줄이 없을 경우 => 출력은 ret::10, cnt::1024가 되지만, 추가 후의 출력은 ret::10, cnt::10이 된다.
