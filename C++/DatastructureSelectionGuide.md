![image](https://github.com/user-attachments/assets/2032915f-6b60-424f-8f1c-21cc79daa184)

- "Order is Important?" → No
    - 문자의 출현 빈도를 세는 것이 목적이므로 순서는 중요하지 않음
    - 결과값이 정렬될 필요가 없음
- "Look-up Keys?" → Yes
    - 각 문자(key)에 대한 빈도수(value) 조회가 필요
    - 특정 문자의 빈도를 빠르게 업데이트하고 접근해야 함
- "Allow Duplicates?" → No
    - 각 문자는 하나의 빈도수만 가짐
    - 같은 문자에 대해 여러 개의 카운트 값을 저장할 필요 없음
- "Separate Key/Value?" → Yes
    - 문자(key)와 그 빈도수(value)를 분리해서 저장해야 함
    - 키-값 쌍으로 데이터를 관리해야 함

