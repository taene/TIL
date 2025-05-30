## QuickSort
### 설명
- 퀵 정렬(Quick Sort): **분할(Partition)과 재귀를 사용**하여 최종적으로 정렬된 리스트를 얻는 정렬 방법
- 분할(Partition): **피벗(Pivot)이라는 값을 기준**으로 왼쪽과 오른쪽으로 분할을 수행한다.
- 시간복잡도: **O(N log N)**, 공간복잡도: O(log N)
- 최악의 경우(정렬이 된 리스트를 정렬할 때), O(N^2)의 시간복잡도를 가지며 매우 비효율적으로 동작한다.
- 최선의 경우 삽입 정렬이 O(N)으로 매우 빠른 속도를 보이지만, 평균적인 경우 퀵 정렬이 가장 효율적이다.
- **Unstable Sor**t: 동일한 값이 있을 경우, 정렬 후 그 값들끼리의 순서가 유지되지 않는 정렬

<br/>

### Code
```cpp
#include <iostream>
#include <vector>

using namespace std;

vector<int> arr;

void Quick_sort(int left, int right);
int Partition(int left, int right);

void Swap(int a, int b)
{
    int temp;
    temp = arr[a];
    arr[a] = arr[b];
    arr[b] = temp;
}

void Asc(int size)
{
    // 선택 정렬
    int min;
    for (int i = 0; i < size - 1; i++)
    {
        min = i;
        for (int j = i + 1; j < size; j++)
        {
            if (arr[min] > arr[j])
            {
                min = j;
            }
        }

        Swap(i, min);
    }
}

void Desc(int size)
{
    // 퀵 정렬
    Quick_sort(0, size - 1);
}

void Quick_sort(int left, int right)
{
    if (left >= right) return;

    int pivot = Partition(left, right);

    Quick_sort(left, pivot - 1);
    Quick_sort(pivot + 1, right);
}

int Partition(int left, int right)
{
    int pivot = arr[right];
    int pivot_index = right;

    right--;

    while (true)
    {
        // 내림차순
        while (arr[left] > pivot) left++;
        while (arr[right] < pivot) right--;

        if (left >= right) break;
        else
        {
            Swap(left, right);

            left++;
        }
    }

    Swap(left, pivot_index);

    return left;
}

int main()
{
    int temp;

    cout << "입력할 숫자의 수를 입력하시오: ";
    int size;
    cin >> size;

    cout << "숫자 입력: ";
    for (int i = 0; i < size; i++)
    {
        cin >> temp;
        arr.push_back(temp);
    }
    
    cout << "배열 정렬 선택 => 1: 오름차순, 2: 내림차순 \n";
    cin >> temp;

    if (temp == 1) Asc(size);
    else Desc(size);

    cout << "정렬한 배열: ";
    for (int i = 0; i < size; i++)
    {
        cout << arr[i] << " ";
    }

    return 0;
}
```

<br/>

### Quick_sort()
1. 재귀 형태로, 무한 루프의 빠지지 않기 위한 조건으로 left >= right 일 경우 return 한다.
2. Partition 함수로 pivot의 인덱스를 구하고, pivot을 기준으로 왼쪽 원소들과 오른쪽 원소들로 분할하여, 분할된 왼쪽 범위와 오른쪽 범위를 다시 재귀 호출한다.
3. 여기서 분할을 할 때 pivot은 제외한다. (pivot은 이미 정렬이 된 상태이기 때문)
4. 오름차순 기준으로 pivot 기준 왼쪽은 pivot보다 작은 값들로 이루어져 있고, pivot보다 오른쪽은 pivot보다 큰 값들로 이루어져 있기 때문에, 이후 분할된 배열이 어떻게 정렬이 되든 **pivot의 위치는 이미 확정이 된 상태**이다. (위의 코드는 내림차순으로, pivot 기분 왼쪽은 큰 값, 오른쪽은 작은 값들로 이뤄져있음)

<br/>

### Partition()
1. pivot으로 가장 오른쪽의 값을 선택했고, pivot 값과 pivot_index 값을 각각 변수로 초기화한다.
2. 가장 오른쪽 값을 pivot으로 선택했기 때문에 이 값을 비교 범위에서 제외하기 위해 right--한다.
3. left는 가장 왼쪽에서 시작해서 pivot보다 큰 값이 나올 때까지 오른쪽으로 이동한다.
4. right는 가장 오른쪽에서 시작해서 pivot보다 작은 값이 나올 때까지 왼쪽으로 이동한다.
5. 오른쪽으로 이동하는 left가 왼쪽으로 이동하는 right보다 크거나 같아지면 while문을 종료하고, 그렇지 않으면 left 위치한 값과 right 위치한 값을 교환하고 left를 1 증가시킨 후에 다시 위 작업을 반복한다.
6. left가 right 보다 크거나 같은 곳 즉, **pivot의 값이 위치할 인덱스 값을 구했기 때문에** pivot 값과 해당 위치의 값을 교환해준 후 pivot index(left)를 return 한다.

<br/>
