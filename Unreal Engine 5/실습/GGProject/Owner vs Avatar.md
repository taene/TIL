## 학습 키워드 <br>

- Gameplay Ability System (GAS)
- ActorInfo, Owning Actor
- Avatar Actor
- Multi Box Trace
- Actors to Ignore

<br/>

## 학습 내용

### 배운 개념 요약

게임플레이 어빌리티 시스템(GAS) 환경에서 트레이스(Trace)를 사용할 때 자기 자신을 무시하기 위해서는 어빌리티의 `Owning Actor`가 아닌 `Avatar Actor`를 `Actors to Ignore` 배열에 추가해야 한다.

  * **Owning Actor**: 어빌리티의 논리적인 소유자를 의미한다. 라이라(Lyra)와 같은 아키텍처에서는 어빌리티 시스템 컴포넌트(ASC)를 `Player State`에 두므로, `Owning Actor`는 보통 `Player State`를 반환한다. `Player State`는 월드에 물리적인 형태가 없는 논리적인 액터이다.
  * **Avatar Actor**: 어빌리티 시스템이 실제로 빙의하고 있는, 월드 상의 물리적인 실체를 의미한다. 일반적으로 플레이어가 조종하는 `Character` 또는 `Pawn`이 해당한다.

물리적인 충돌을 감지하는 트레이스 함수는 월드에 존재하는 액터를 대상으로 하므로, 물리적인 실체인 `Avatar Actor`를 무시하도록 설정해야 의도한 대로 동작한다.

<br/>

### 구현 과정

**문제 상황:** `Get Owning Actor from Actor Info` 노드의 반환 값을 `Actors to Ignore` 인자로 사용하여 박스 트레이스를 실행했을 때, 트레이스를 발사한 자기 자신의 캐릭터가 `Out Hits`에 포함되는 문제가 발생했다. `Print String`으로 확인 결과 `Get Owning Actor`는 `Player State`를 반환하고 있었고, `Hit Actor`는 플레이어 캐릭터 블루프린트였다.

**원인 분석:** 트레이스 함수에 무시하도록 전달한 액터는 `Player State`이지만, 이는 물리적 콜리전이 없는 논리적 액터이다. 따라서 트레이스는 `Player State`를 무시하는 데는 성공했지만, 정작 월드에 존재하는 물리적 실체인 플레이어 `Character`는 무시하지 못하고 감지한 것이다.

**해결 방법:** `Get Owning Actor from Actor Info` 노드를 **`Get Avatar Actor from Actor Info`** 노드로 교체했다. 이 노드는 어빌리티를 사용하는 물리적 실체, 즉 플레이어 `Character`를 직접 반환한다. 이 `Avatar Actor`를 `Actors to Ignore` 배열에 넘겨주자, 트레이스가 자기 자신을 정확히 무시하고 원하는 대상만 감지하게 되었다.

1.  기존 `Multi Box Trace For Objects` 노드의 `Actors to Ignore` 핀에 연결된 `Get Owning Actor from Actor Info` 노드를 제거한다.
2.  `Get Avatar Actor from Actor Info` 노드를 생성하고, `Actor Info`를 연결한다.
3.  `Get Avatar Actor from Actor Info`의 반환 값을 `Make Array` 노드를 통해 배열로 만든 후, `Actors to Ignore` 핀에 연결한다.

이 과정을 통해 트레이스 로직이 의도한 대로 자기 자신을 제외하고 충돌을 감지하도록 수정할 수 있었다.

<br/>

### 요약

| 개념 | 설명 | 사용 사례 |
| :--- | :--- | :--- |
| **Owning Actor** | 어빌리티의 논리적 소유자 (주로 `PlayerState`) | 어빌리티 소유권 확인, 데이터 접근 |
| **Avatar Actor** | 어빌리티의 물리적 실체 (주로 `Character`) | 트레이스 무시, 월드에서의 물리적 상호작용 |
| **문제 원인** | 트레이스에서 무시해야 할 물리적 대상(`Avatar`) 대신 논리적 소유자(`Owner`)를 지정함. |
| **해결** | `Get Avatar Actor from Actor Info`를 사용하여 `Character`를 직접 무시 목록에 추가함. |
