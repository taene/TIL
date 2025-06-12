## const
### const 변수
#### 1. const 비-멤버 변수
```cpp
const int num = 1; // 일반적인 표현
int const num = 1; // 위와 같은 의미
num = 2;           // Compile Error
```
- 위와 같이 선언하면 num은 변경할 수 없는 변수인 상수가 된다.
- 또한 함수의 반환형이나 매개변수가 const 변수형일 때도 같은 의미이다.

#### 2. const 멤버 변수
```cpp
class Foo
{	
    const int num; // 메모리 할당이 아님 
    
    Foo(void)		
    : num(1) // const int num = 1;	
    {}
}; 
class Bar
{
    const int num; 	
    
    Bar(void)	
    {		
    	num = 1; // Compile Error		
        // const int num;		
        // num = 1;	
    }
};
```
- const 변수는 반드시 선언 시 초기화를 해야 한다.
- class의 멤버 변수를 const로 선언 시에는 반드시 초기화 리스트를 사용해서 초기화를 해줘야 한다.

#### 3. const 포인터 변수
```cpp
int num = 1;
const int* ptr = &num;	// *ptr 상수화

*ptr = 2;	// Compile Error
num = 2;	// Pass
```
- 첫번째, const 위치가 맨 앞에 있으면서 포인터 변수가 가리키는 값에 대해서 상수화를 시키는 경우
  - 포인터 변수가 가리키는 num 자체가 상수화가 되는 것이 아니므로 num 값을 바꾸는것은 된다.

```cpp
int num1 = 1;
int num2 = 2;
int* const ptr = &num1;	// ptr을 상수화

ptr = &num2;	// Compile Error
```
- 두번째, const 위치가 type과 변수 이름 사이에 있으면서 포인터 변수 자체를 상수화 시키는 경우
  - 포인터 변수는 대상의 주소 값을 저장하는 변수인데, 위의 ptr은 자기 자신을 상수화 시키는 것이기 때문에 num2의 주소 값으로 변경하려고 하면 에러가 발생한다.

```cpp
// 위의 두 경우를 모두 사용하는 형태
int num = 1;
const int* const ptr = &num;
```

<br/>

### const 멤버 함수
```cpp
int GetString(void) const;	// Compile Error

class Foo
{
    int num = 1;
    
    int GetNum(void) const
    {
    	int a = 1;
        a++;	// 지역 변수는 가능하다
        
        num++;	// Compile Error
        return num;
    }
};
```
- const 멤버 함수는 class의 멤버 함수만 const로 상수화를 시킬 수 있고 비-멤버 함수는 const 함수로 선언이 불가능하다.
- const의 위치는 함수 선언문 맨 뒤이고, 해당 멤버 함수 내에서 모든 멤버 변수를 상수화시킨다는 의미이다. 따라서 위와 같이 멤버 변수인 num은 변경할 수 없고 지역 변수인 a는 변경할 수 있다.

<br/>

### const 멤버 함수에 관하여
```cpp
FORCEINLINE const FString& GetName() const { return Name; }
FORCEINLINE class UCard* GetCard() const { return Card; }
```
- 첫 번째 코드에서 GetName 함수는 const 멤버 함수이기 때문에 Name 멤버 변수에 대한 변경을 허용하지 않는데, 반환값이 const FString& 형식이어야만 이를 보장한다. 질문에 있는 대로, FString&로 반환할 경우 호출자가 반환된 참조를 통해 Name을 변경할 수 있기 때문에 컴파일러는 오류를 발생시킨다.
- 두 번째 코드에서 GetCard 함수는 포인터를 반환하는데, 여기서 중요한 것은 반환되는 UCard* 포인터 자체는 const 함수인 ‘GetCard’에서 변경할 수 없다. 그러나 이 포인터가 가리키는 객체, 즉 UCard 객체는 const가 아니기 때문에, 반환 후에 객체의 상태를 변경할 수 있는 가능성이 있다.
- 여기서 const 멤버 함수에서 포인터를 반환할 때, 객체의 상태 변경을 전혀 허용하지 않으려면 반환 타입을 const UCard*로 지정해야 한다. 이렇게 반환 타입에 const를 붙이면 포인터를 통해 원본 객체를 변경하는 것이 금지된다.

```cpp
FORCEINLINE const class UCard* GetCard() const { return Card; }
```
- 이 경우 GetCard의 반환값을 통해 UCard 객체의 변경이 불가능하다는 것을 보장할 수 있다.

<br/>
