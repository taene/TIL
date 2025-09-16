## 학습 키워드 

  * Gameplay Ability System (GAS)
  * ExecutionCalculation
  * AttributeSet
  * Meta Attribute (메타 어트리뷰트)
  * Direct Modification (직접 수정)
  * Software Architecture
  * Separation of Concerns (SoC, 역할 분리 원칙)

<br/>

## 학습 내용

### 배운 개념 요약 

`ExecutionCalculation`에서 계산된 데미지를 `AttributeSet`에 적용하는 두 가지 아키텍처 패턴에 대해 학습했다. 각 패턴은 장단점이 명확하여, 프로젝트의 방향성과 확장성을 고려한 신중한 선택이 필요하다.

1.  **직접 수정 (Direct Modification) 방식**

      * `ExecutionCalculation`이 최종 계산 결과를 실제 어트리뷰트(예: `Shield`)에 직접 수정하는 `Modifier`를 출력하는 방식이다.
      * 데이터 흐름이 `ExecCalc` → `실제 어트리뷰트`로 단순하고 직관적이다.

2.  **메타 어트리뷰트 (Meta Attribute) 방식**

      * `ExecutionCalculation`이 계산 결과를 `Damage`, `ShieldDamage`와 같은 임시 어트리뷰트(메타 어트리뷰트)에 `Override`로 출력하는 방식이다.
      * 이후 `AttributeSet`의 `PostGameplayEffectExecute` 함수가 이 메타 어트리뷰트 값을 읽어 실제 어트리뷰트에 반영하고, 메타 어트리뷰트는 0으로 초기화한다.
      * 데이터 흐름이 `ExecCalc` → `메타 어트리뷰트` → `PostGameplayEffectExecute` → `실제 어트리뷰트`로 한 단계를 더 거친다.

<br/>

### 구현 과정 

두 방식의 장단점을 비교 분석하여 최종적으로 어떤 아키텍처가 더 우수한지 결론을 내렸다.

| 평가 항목 | 직접 수정 | 메타 어트리뷰트 | 최종 선택 |
| :--- | :--- | :--- | :--- |
| **단순성** | 코드가 간결하고 데이터 흐름이 짧다. | 한 단계 더 거쳐 상대적으로 복잡하다. | 직접 수정 |
| **확장성** | 기능 추가 시 `ExecCalc`가 비대해진다. | `AttributeSet` 내에서 독립적인 기능 확장이 용이하다. | **메타 어트리뷰트** |
| **역할 분리** | `ExecCalc`가 계산과 적용 방식을 모두 결정한다. | `ExecCalc`는 '계산', `AttributeSet`은 '적용'으로 역할이 명확히 분리된다. | **메타 어트리뷰트** |
| **일관성** | `LyraHealthSet`, `GGResourceSet`의 패턴과 다르다. | 프로젝트의 다른 `AttributeSet`들과 구조적 일관성을 유지한다. | **메타 어트리뷰트** |

#### 코드 예시

  * **직접 수정 방식의 `ExecCalc` 출력부**

    ```cpp
    // ExecCalc가 Shield 어트리뷰트를 직접 수정하도록 Additive Modifier를 출력한다.
    OutExecutionOutput.AddOutputModifier(
        FGameplayModifierEvaluatedData(UGGShieldSet::GetShieldAttribute(), EGameplayModOp::Additive, -DamageToShield));
    ```

  * **메타 어트리뷰트 방식의 구현부**

    ```cpp
    // 1. ExecCalc는 ShieldDamage 라는 메타 어트리뷰트에 Override로 출력한다.
    OutExecutionOutput.AddOutputModifier(
        FGameplayModifierEvaluatedData(UGGShieldSet::GetShieldDamageAttribute(), EGameplayModOp::Override, DamageToShield));

    // 2. ShieldSet의 PostGameplayEffectExecute가 값을 받아 처리한다.
    void UGGShieldSet::PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data)
    {
        if (Data.EvaluatedData.Attribute == GetShieldDamageAttribute())
        {
            const float DamageDone = GetShieldDamage();
            SetShield(GetShield() - DamageDone); // 실제 Shield 값에 반영
            SetShieldDamage(0.0f);               // 메타 어트리뷰트 초기화
        }
    }
    ```

<br/>

## 느낀점 

- 단순히 기능을 구현하는 것을 넘어, 어떤 설계가 장기적으로 프로젝트에 유리한지 깊게 고민하는 계기가 되었다. 처음에는 코드 라인이 적고 흐름이 단순한 '직접 수정' 방식이 더 효율적이라고 생각했다. 하지만 "쉴드 관통", "데미지 반사" 등 미래에 추가될 수 있는 기능들을 고려해보니, `ExecCalc`에 모든 로직을 집중시키는 것이 얼마나 위험한 설계인지 깨달았다.

- '메타 어트리뷰트' 방식은 당장의 복잡성은 조금 증가하지만, `ExecCalc`와 `AttributeSet`의 역할을 명확히 분리하여 각 모듈이 자신의 책임에만 집중할 수 있는 환경을 만들어준다. 이는 향후 기능 추가 시 `ExecCalc`의 거대한 코드를 수정할 필요 없이, `ShieldSet` 내부에서 독립적으로 로직을 확장할 수 있게 해주는 매우 큰 장점이다.

- 특히 `LyraHealthSet`이나 내가 만든 `GGResourceSet`이 이미 이 패턴을 사용하고 있다는 점에서, 프로젝트 전체의 구조적 일관성을 지키는 것이 얼마나 중요한지도 다시 한번 생각하게 되었다. 일관성 있는 아키텍처는 미래의 내가, 또는 새로운 팀원이 코드를 이해하고 적응하는 데 드는 비용을 크게 줄여줄 것이다.

- 결론적으로, 좋은 개발자는 단순히 동작하는 코드를 짜는 것을 넘어, 미래를 예측하고 변화에 유연하게 대처할 수 있는 견고한 구조를 설계해야 한다는 것을 배웠다. 메타 어트리뷰트 패턴은 그러한 설계 철학이 담긴 훌륭한 예시이다.
