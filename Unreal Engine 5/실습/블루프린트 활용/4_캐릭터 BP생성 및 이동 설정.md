## 캐릭터 블루프린트 생성
![image](https://github.com/user-attachments/assets/15be5afb-0e71-4cf4-9092-8f1a20077607)
- Content > ThirdPerson > Blueprints > BP_ThirdPersonCharacter 열기
- 복제하여 MyCharacter로 이름 변경

<br/>

## Input Setting 확인
![image](https://github.com/user-attachments/assets/56d11825-21f2-414e-b346-aeb0e31df457)
![image](https://github.com/user-attachments/assets/2689415c-b464-4cd5-892d-88fa63addbd5)
- InputAction 3개: IA_Jump(digital-bool), IA_Look(Vector2D), IA_Move(Vector2D)
- InputMappingContext (Check only PC)
  - IA_Jump
    - Space Bar
  - IA_Move
    - W : <tt>Swizzle</tt>
    - S : <tt>Swizzle</tt> , <tt>Negate</tt>
    - A : <tt>Negate</tt>
    - D
    - Up/Down/Right/Left
  - IA_Look
    - Mouse XY 2D-Axis : <tt>Negate</tt>

<br/>

## 캐릭터 블루프린트 연결
![image](https://github.com/user-attachments/assets/ab2b93b9-f2ea-4ed3-8957-0aae8ecfe4da)
- World Settings > Select GameMode > Default Pawn Class > MyCharacter로 변경

![image](https://github.com/user-attachments/assets/d626ea81-226c-4e86-8ee4-6a1b44253fa1)
- 잘 적용된 모습이다.

<br/>
