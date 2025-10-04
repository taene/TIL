## 학습 키워드 <br>

- 언리얼 엔진
- 리플리케이션(Replication)
- 멀티캐스트(Multicast) RPC
- 데이터 동기화

<br/>

## 학습 내용 

### 배운 개념 요약

멀티플레이 환경에서 서버가 발사한 투사체의 발사(Muzzle) 및 충돌(Impact) 이펙트가 클라이언트에서 보이지 않는 문제를 해결했다.

이 문제의 원인은 `NetMulticast` RPC가 클라이언트에서 함수를 '호출'은 해주지만, 함수 실행에 필요한 데이터(이펙트 에셋, 사운드 에셋 등)까지 자동으로 넘겨주지는 않기 때문이다. 서버에만 초기화되어 있던 이펙트 데이터 구조체(`FGGProjectileParams`)가 클라이언트에는 동기화되지 않아, 클라이언트의 해당 데이터는 `nullptr` 상태였다. 따라서 RPC가 호출되어도 실제 재생할 에셋이 없어 아무 일도 일어나지 않았던 것이다.

해결책은 이펙트 데이터가 담긴 구조체 자체를 **리플리케이션(Replication)** 하여 서버의 데이터를 클라이언트와 동기화하는 것이다. `UPROPERTY(Replicated)` 지정자와 `GetLifetimeReplicatedProps` 함수를 통해 특정 프로퍼티를 네트워크를 통해 복제하도록 설정함으로써, 모든 클라이언트가 올바른 에셋 정보를 가지고 RPC를 실행하게 만들어 문제를 해결한다.

<br/>

### 구현 과정

#### 1. 헤더 파일 (`.h`) 수정

투사체 액터의 헤더 파일에서 데이터 구조체 프로퍼티에 `Replicated` 키워드를 추가하고, 리플리케이션 관련 함수를 오버라이드한다.

```cpp
// GGSkillProjectile.h

#pragma once

#include "CoreMinimal.h"
#include "Weapons/GGSkillActor.h"
#include "Net/UnrealNetwork.h" // 리플리케이션 헤더 추가
#include "GGSkillProjectile.generated.h"

// ...

UCLASS()
class GGCORERUNTIME_API AGGSkillProjectile : public AGGSkillActor
{
	GENERATED_BODY()

public:
	// ... (생성자 및 다른 함수 선언)

	// 리플리케이트될 프로퍼티를 등록하는 함수를 오버라이드 선언한다.
	virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;

protected:
	// ...

	// UPROPERTY에 Replicated 키워드를 추가하여 이 프로퍼티가 복제 대상임을 명시한다.
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Replicated, meta = (ShowOnlyInnerProperties))
	FGGProjectileParams ProjectileParams;

	// ...
};
```

<br/>

#### 2. 소스 파일 (`.cpp`) 수정

소스 파일에서는 생성자에서 액터의 리플리케이션을 활성화하고, `GetLifetimeReplicatedProps` 함수를 실제로 구현하여 어떤 프로퍼티를 복제할지 엔진에 알려준다.

```cpp
// GGSkillProjectile.cpp

#include "Weapons/GGSkillProjectile.h"
#include "Net/UnrealNetwork.h" // 헤더 추가
// ...

AGGSkillProjectile::AGGSkillProjectile()
{
	// 이 액터가 네트워크 상에서 복제되어야 함을 설정한다.
	bReplicates = true;
	
	// ...

	// ProjectileMovementComponent 역시 복제되도록 설정하여 움직임을 동기화한다.
	ProjectileMovement->SetIsReplicated(true);
}

// 리플리케이트할 프로퍼티를 등록하는 함수를 구현한다.
void AGGSkillProjectile::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	// DOREPLIFETIME 매크로를 사용하여 ProjectileParams 프로퍼티를 리플리케이션 목록에 등록한다.
	DOREPLIFETIME(AGGSkillProjectile, ProjectileParams);
}

// ... (나머지 코드)
```

<br/>

## 느낀점

`NetMulticast` RPC는 모든 클라이언트에게 "이 함수를 실행해\!" 라고 명령을 내리는 확성기와 같다는 것을 깨달았다. 하지만 그 명령을 수행하기 위해 필요한 준비물(데이터)은 각자 알아서 가지고 있어야 하는 것이었다. 서버가 `ProjectileParams`라는 준비물을 가지고 명령을 내렸지만, 클라이언트들은 빈손이었기 때문에 명령을 수행할 수 없었던 것이다.

이번 경험을 통해 **RPC는 '행위'를 동기화**하고, **리플리케이션은 '상태(데이터)'를 동기화**한다는 핵심적인 차이를 명확하게 이해하게 되었다. 멀티플레이어 환경에서는 어떤 액터의 특정 기능이 클라이언트에서 제대로 동작하지 않을 때, 단순히 RPC 호출 여부만 확인할 것이 아니라, 그 기능을 실행하는 데 필요한 모든 데이터가 클라이언트에 제대로 존재하는지(리플리케이트되었는지)를 먼저 확인하는 습관을 들여야겠다. 이것이 서버-클라이언트 모델의 기본이자 가장 중요한 원칙임을 다시 한번 상기한다.
