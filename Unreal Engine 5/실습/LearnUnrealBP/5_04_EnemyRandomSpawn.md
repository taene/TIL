## 학습 키워드
- Random float in Range Node
- SpawnActor Class Node
- Clamp

<br/>

## 학습 내용
### 배운 개념 요약
- Random float in Range Node
  - Min ~ Max 사이의 랜덤 값을 반환함
- SpawnActor Class Node
  - Class: Spawn할 액터 클래스 설정
  - Spawn Transform: spawn할 transform
  - Collision Handling Override
    - Default
    - Always Spawn, Ignore Collisions
    - Try to Adjust Location, But Always Spawn: 부딪힌 물체가 있으면 주변을 탐색해서 소환함
    - Try to Adjust Location, Don't Spawn if Still Colliding
    - Do not Spawn
- Clamp: 최솟값과 최댓값으로 정의된 특정 범위로 결과값을 제한한다.

<br/>

### 구현 과정
![image](https://github.com/user-attachments/assets/3b03e4b2-af0b-4caf-85d8-a058742fca4a)

![image](https://github.com/user-attachments/assets/dc520385-80b8-4aa8-bb89-ea2f2d0971b4)
- Stage(int) 변수를 통해, 0~20초 사이에는 0, 20~40초 사이에는 1이 되는 등, 20초마다 Stage가 1씩 증가함

![image](https://github.com/user-attachments/assets/2107ce04-f508-49d8-8a79-1a0035d54485)
- 원래라면 Stage*0.5 값이 랜덤값에서 계속 빼져서 0이하가 될 수 있어서 엄청 어려워질 수 있는데, 이를 막기위해 Clamp로 범위를 제한함

![image](https://github.com/user-attachments/assets/f193c6c1-e5b4-4f29-b4d1-a420950969e5)
![image](https://github.com/user-attachments/assets/574c8f44-a245-46d7-888f-206beb71c65e)


<br/>

## 느낀점
- 

<br/>
