## 학습 키워드
- Streaming Level/Persistent Level
- [레벨 관리 공식문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/managing-multiple-levels-in-unreal-engine)
- Window SDK 

<br/>

## 학습 내용
### 배운 개념 요약
- Persistent Level: 영구적인 레벨, 마스터 맵, 항상 로드되는 맵
- Streaming Level: 다른 맵을 마스터 맵으로 불러올 수 있는데, 이를 Streaming한다고 하고, 불리는 맵을 Streaming Level이라 함
- GameMode/Widget/Character
  - 중요한 게임 로직은 GameMode에
  - UI 표시 기능은 Widget에
  - 캐릭터의 동작 로직은 Character에

<br/>

### 구현 과정
![image](https://github.com/user-attachments/assets/64060783-0668-4a7a-bcbd-cd79d32ee751)
- Default Map: 땅, 라이트, 하늘이 포함된 처음 시작했을때 UI를 띄워줄 맵(Persistent)
- NewMap: 각종 Actor 장애물과 AI가 배치된 레벨(Streaming)
- List of maps to include in a package build(패키지 된 빌드에 포함시킬 맵 목록)에 Streaming Level이 될 NewMap을 배열에 추가함
- Project Settings > Maps > Default Map으로 설정

#### Game Build 하는법
- Windows SDK Download(in this computer)
- UE에서 Platforms > Refresh platform status(플랫폼 상태 새로고침)
- Platforms > Windows > Project Package(제품화)
  - Shipping: 실제 제품으로 배출
  - Development: 개발 환경으로 배출
  - DebugGame: 디버깅 용도로 배출
  - 프로젝트 패키징 전에 Project Settings > Description(설명)에서 프로젝트 이름, 버전, 회사 이름 등 설정하기
- Project Icon 바꾸는법
  - Project Settings > Platforms > Windows > Game Icon 바꾸기(.ico 파일)

<br/>
