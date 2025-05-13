# 🎮 Mini Project

## 1. 사용 에셋
- Modular Dungeon Collection
- Third Person

<br/>

## 2. 구현 기능
- 캐릭터 이동 + 회전
- `E` 키를 누르면 문이 열리는 상호작용 구현

![image](https://github.com/user-attachments/assets/af5eaa07-493e-48be-a2a7-d65bf7037806)
- BP_Character에서 LineTrace 구현과 Hit Actor이 Door 태그를 가진지 확인하고 BP_Door의 OpenDoor Event 실행

![image](https://github.com/user-attachments/assets/4c078690-fe61-4aba-9bd5-3398cadec4a4)
- BP_DungeonDoor에서 LeftDoor Static Mesh와 RightDoor Static Mesh 각각의 Rotation 값을 받음(GetOwner(StaticMesh) > Get Relative Rotation)
- TimeLine과 Lerp Rotator를 활용해 왼쪽 문과 오른쪽 문의 각 Rotation 값을 부드럽게 회전함
  - Set Relative Rotation으로 각 Mesh의 회전값을 설정

<br/>

## 3. 결과물
![OpenDoor](https://github.com/user-attachments/assets/8e8dd2bb-6438-43be-a87e-a74cc87d8ba2)
![image](https://github.com/user-attachments/assets/04560e7e-61cd-4871-aae1-280719489c92)

<br/>

## 4. 소감
- 상호작용의 E키를 누르고 Line Trace 활성화는 Character BP에, 문이 회전하는 것은 Door BP에 구현해서 그 사이를 Custom Event로 연결하는 법을 배웠다
  - 아예 BP_Interaction라는 상호작용 관련된 BP를 따로 만들어 구현하는 것이 좋다
- 하나의 Door Actor에 왼쪽 문과 오른쪽 문의 움직임을 따로 구현하는 것을 배웠다

<br/>
