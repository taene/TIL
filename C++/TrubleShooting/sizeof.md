## 함수의 인자로 넘겨받은 배열의 크기(sizeof)

```cpp
int main()
{
    int arr[] = {1,2,3,4,5};
    int size = sizeof(arr) / sizeof(int);  // 5
}
```
- 위와 같이 배열이 주어지면 sizeof를 사용해 배열의 크기와 원소의 크기를 통해 배열의 길이를 알 수 있다.

<br/>

```cpp
float avg(int* arr)
{
    int size = sizeof(arr) / sizeof(int);  // Error
    return static_cast<float>(sum(arr)) / size;
}
```
- ⚠️ 하지만, 함수의 인자로 배열을 넘겨받아서 위와 같이 계산하면 구할 수 없다.
- 그 이유는 인자로 arr을 받을 때 **배열이 아닌 포인터로 받기 때문**이다. 즉 배열의 이름(주소)을 받는다. 그래서 **분자가 배열의 크기가 아닌 포인터의 크기가 된다**.
- **정적 배열은 그 원소들이 미리 정해지므로, sizeof 연산자를 통해 컴파일 시간에 배열의 크기를 알 수 있다**. (정확히는 sizeof는 인자가 정적 배열일 경우 내부적으로 배열의 크기를 계산하도록 되어있다고 안다)
- 그런데 함수에서 배열을 받으려면 첫 원소의 주소만 받는 것이 효율적이므로 당연히 포인터로 받을 수밖에 없다.
- 그리고 **포인터가 가리키는 배열은 정적 배열과 달리 크기를 동적으로 할당받을 수 있기 때문에 그 크기를 미리 알 수가 없다**. 때문에 문자열과 같이 널문자로 배열의 끝을 표시해주지 않는 이상 배열의 크기를 알 수 없다.
- ✅ 따라서 함수의 인자로 배열을 전달한다면 size 또한 함께 인자로 전달하거나, 문자열과 같이 배열의 끝을 알리는 값을 넣어 검사해주어야 한다.

<br/>
