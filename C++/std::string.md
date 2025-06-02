## string 라이브러리의 getline()
- getline(cin, 문자열을 저장할 string객체, 종결 문자);
- 최대 문자 수를 입력하지 않아도 된다.
- 종결 문자를 만날 때 까지 모든 문자열을 입력 받아 하나의 string 객체에 저장한다.

​<br/>

## istream 라이브러리의 cin.getline()>
- cin.getline(변수 주소, 최대 입력 가능 문자수, 종결 문자);
- 문자 배열이며 마지막 글자가 ‘\0’(terminator)인 c-string을 입력 받는데 사용한다.
- n-1개의 문자 개수만큼 읽어와 str에 저장한다. (n번째 문자는 NULL(‘\0’)로 바꾼다.)
- 종결 문자를 별도로 지정해주지 않으면 엔터(‘\n’)로 인식한다.

​<br/>

## string 클래스 멤버함수
### string의 특정 원소 접근
| 함수 | 설명 |
| :-: | --- |
| str.at(index) | index 위치의 문자 반환. 유효한 범위인지 체크 O |
| str[index] | index 위치의 문자 반환. 유효한 범위인지 체크 X. 따라서 at 함수보다 접근이 빠름 |
| str.front() | 문자열의 가장 앞 문자 반환 |
| str.back() | 문자열의 가장 뒤 문자 반환 |

​<br/>

### string의 크기
| 함수 | 설명 |
| :-: | --- |
| str.length() | 문자열 길이 반환 |
| str.size() | 문자열 길이 반환 (length와 동일) |
| str.capacity() | 문자열이 사용중인 메모리 크기 반환 |
| str.resize(n) | string을 n의 크기로 만듦. 기존의 문자열 길이보다 n이 작다면 남은 부분은 삭제하고, n이 크다면 빈공간으로 채움 |
| str.resize(n, 'a') | n이 string의 길이보다 더 크다면, 빈 공간을 'a'로 채움 |
| str.shrink_to_fit() | string의 capacity가 실제 사용하는 메모리보다 큰 경우 낭비되는 메모리가 없도록 메모리를 줄여줌 |
| str.reserve(n) | size = n만큼의 메모리를 미리 할당해줌 |
| str.empty() | str이 빈 문자열인지 확인 |

​<br/>

### string에 삽입, 추가, 삭제
| 함수 | 설명 |
| :-: | --- |
| str.append(str2) | str 뒤에 str2 문자열을 이어 붙여줌 ('+' 와 같은 역할) |
| str.append(str2, n, m) | str 뒤에 'str2의 n index부터 m개의 문자'를 이어 붙여줌 |
| str.append(n, 'a')	 | str 뒤에 n개의 'a'를 이어 붙여줌 |
| str.insert(n, str2) | n번째 index 앞에 str2 문자열을 삽입함 |
| str.replace(n, k, str2) | n번째 index부터 k개의 문자를 str2로 대체함 |
| str.clear() | 저장된 문자열을 모두 지움 |
| str.erase(n, m) | n번째 index부터 m개의 문자를 지움 |
| str.erase(n, m) (iterator) | n~m index의 문자열을 지움 (n과 m은 iterator) |
| str.erase() | clear와 같은 동작 |
| str.push_back(c) | str의 맨 뒤에 c 문자를 붙여줌 |
| str.pop_back() | str의 맨 뒤의 문자를 제거 |
| str.assign(str2) | str에 str2 문자열을 할당. (변수 정의와 동일) |

​<br/>

### 기타 유용한 string 멤버 함수
| 함수 | 설명 |
| :-: | --- |
| str.find("abcd") | "abcd"가 str에 포함되어있는지를 확인. 찾으면 해당 부분의 첫번째 index를 반환 |
| str.find("abcd", n) | n번째 index부터 "abcd"를 find |
| str.substr() | str 전체를 반환 |
| str.substr(n) | str의 n번째 index부터 끝까지의 문자를 부분문자열로 반환 |
| str.substr(n, k) | str의 n번째 index부터 k개의 문자를 부분문자열로 반환 |
| str.compare(str2) | str과 str2가 같은지를 비교. 같다면 0, str<str2 인 경우 음수, str>str2 인 경우 양수를 반환 |
| swap(str1, str2) | str1과 str2를 바꿔줌. reference를 교환하는 방식 |
| isdigit(c) | c 문자가 숫자이면 true, 아니면 false를 반환  |
| isalpha(c) | c 문자가 영어이면 true, 아니면 false를 반환 |
| toupper(c) | c 문자를 대문자로 변환 |
| tolower(c) | c 문자를 소문자로 변환 |

​<br/>

## string의 int 변환
- **stoi** = string to int
- **stof** = string to float
- **stol** = string to long
- **stod** = string to double

​<br/>

## string split()
```cpp
vector<string> split(const string& input, string delimiter)
{
	vector<string> result;
	auto start = 0;
	auto end = input.find(delimiter);
	while (end != string::npos)
	{
		result.push_back(input.substr(start, end - start));
		start = end + delimiter.size();
		end = input.find(delimiter, start);
	}
	result.push_back(input.substr(start));
	return result;
}
```

​<br/>
