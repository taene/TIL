## 학습 키워드
- Actor > Pawn > Character
- Possess
- CharacterMovement Component
- Controller
- Material
- Component

<br/>

## 학습 내용
### 배운 개념 요약
- Actor를 상속받는 Pawn Class의 특이점: Possess
  - Possess(빙의): Controller Actor에서 입력을 전달받을 수 있는 기능
  - Controller Actor가 Pawn에 Possess하면 입력을 전달할 수 있는 상태가 됨
- CharacterMovement Component: Character Owner에 대한 움직임 로직을 처리함(걷기, 뛰기, 날기, 헤엄치기 등 지원함, 현재 속도 및 가속도의 영향을 받음, 가속도는 지금까지 축적된 입력 벡터에 따라 매 프레임 업데이트 됨)
- Controller Actor: Player Controller / AI Controller
  - Player Controller: 키보드와 마우스로부터 입력을 받아 Pawn으로 전달함
  - AI Controller: 정해진 규칙에 따라 AI가 동작하면서 그것에 대한 처리 결과를 Pawn에 전달함(Pawn은 전달받으면 그것에 대한 입력 처리를 할 수 있게 됨)
- Component 종류
  - Scene Component: Transform(위치, 회전, 스케일)
  - Static Mesh Component: Scene Component의 Child(Transform을 갖고 있음) + Static Mesh(변형되지 않는 3D 모델)
    - Material: 컴포넌트는 아니지만 3D 혹은 2D 물체에 적용되는 재질
      - Metallic: 금속 표면 조정
      - Specular: 비금속 표면의 반사도 조정
      - Roughness: 0 유광 ~ 1 무광
  - Skeletal Mesh Component: 애니메이션이 있고, 뼈가 있는 3D에셋인 스켈레탈 메시 에셋을 표현하며, 애니메이션을 넣을 수 있는 컴포넌트
  - Light Component(Directional/Rect/Sky/Spot/Point)
  - SpringArm Component & Camera Component: 촬영하여 화면 또는 기록으로 나타내는 기능을 하는 컴포넌트
- BP Node
  - Add: 더한다 / Get: 가져온다 / Set: 설정한다
  - (Component) Activate: 활성화한다 / Deactivate: 비활성화한다 / Is Active: 활성화유무 체크 후 return
  - Is Valid(노드): 현재 컴포넌트가 정상인지 return true-false / ? Is Valid(함수): 정상이면 Is Valid 실행, 비정상이면 Is Not Valid 실행

<br/>
