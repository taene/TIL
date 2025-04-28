# 액터의 구조
- 월드에 속한 콘텐츠의 기본 단위를 액터라고 한다.
- 액터는 트랜스폼을 가지며, 월드로부터 틱과 시간 서비스를 제공받는다.
- 액터는 논리적 개념이며 컴포넌트를 감싸는 포장 박스에 불과하다.
- 실질적인 구현은 컴포넌트가 진행하고 액터는 다수의 컴포넌트를 소유하는 관리자 역할이다.
- 다수의 컴포넌트를 대표하는 컴포넌트를 루트 컴포넌트(Root Component)라고 한다.
- 액터는 루트 컴포넌트를 가져야 하며, 루트 컴포넌트의 트랜스폼은 액터의 트랜스폼을 의미한다.
 

<br/>

# C++ 액터에서 컴포넌트의 생성
- 컴포넌트는 언리얼 오브젝트이므로 UPROPERTY를 설정하고 TObjectPtr로 포인터를 선언한다.
  - 언리얼5 버전부터 헤더에 언리얼 오브젝트를 선언할 때 일반 포인터에서 TObjectPtr로 변경
- 컴포넌트의 등록
  - CDO(Class Default Object)에서 생성한 컴포넌트 ( 즉, 생성자에서 생성한 컴포넌트 )는 자동으로 월드에 등록된다.
  - NewObject로 생성한 컴포넌트는 반드시 등록 절차를 거쳐야 한다. (ex. RegisterComponent)
  - 등록된 컴포넌트는 월드의 기능을 사용할 수 있으며, 물리와 렌더링 처리에 합류한다.
- 컴포넌트의 확장 설계
  - 에디터 편집 및 블루프린트의 승계를 위한 설정
  - UPROPERTY에 지정자(Specifier)를 설정할 수 있다.
- 컴포넌트 지정자
  - Visible / Edit: 객체 타입 / 값 타입으로 사용
  - Anywhere / DefaultsOnly / InstanceOnly: 에디터에서 편집 가능 영역 (Anywhere을 주로 사용, ex.객체타입 - VisibleAnywhere(컴포넌트는 객체타입), 값타입 - EditAnywhere)
  - BluprintReadOnly / BlueprintReadWrite: 블루프린트로 확장시 읽기 혹은 읽기쓰기 권한을 부여
  - Category: 에디터 편집 영역(Detail)에서의 카테고리 지정
 
<br/>
