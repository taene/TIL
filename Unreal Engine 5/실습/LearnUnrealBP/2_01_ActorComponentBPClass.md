## 학습 키워드
- Actor
- Component
- Parent&Child Component
- Inheritance(상속)
- Class
- Instance

<br/>

## 학습 내용
### 배운 개념 요약
- Parent&Child Component: 부모자식 관계
  - Actor안의 DefaultSceneRoot Component는 Actor에 기본적으로 존재하는 Component
  - Actor에 컴포넌트를 추가하면 DefaultSceneRoot Component의 자식으로 추가됨
  - 이를 부모 자식 컴포넌트라고 하고, 자식 컴포넌트는 부모 컴포넌트의 상대위치를 따름
  - 절대위치로 바꾸고 싶다면, Child Component의 Transform에서 위치 타입을 World로 바꾸면 됨
- Inheritance: 상속, 부모의 특성과 기능을 물려받는 것(자식은 부모의 것에 자신의 것을 확장할 수 있음)
  - 부모클래스, 자식클래스 라고 함
  - 클래스 간에 가능함
- Class: 설계도
- Instance: 구현체(객체)
#### Actor Instance를 BP Class로 변환
- Actor를 스크립트 작동방식을 가질 수 있는 재사용 가능한 BP Class로 변환함
  - New Subclass(새 서브클래스): 현재 액터 인스턴스의 부모 클래스의 Child Class 중 하나를 선택해, 현재 액터의 부모 클래스로 바꿔 BP Class 생성
  - Cild Actors(자손 액터): 새로운 액터를 만들고 그 액터 밑에 자식 액터로 해당 액터를 넣음
  - Harvest Components(컴포넌트 수집): 컴포넌트들만 모아 새 액터를 만들고 그 액터안에 컴포넌트를 다시 넣음

<br/>

