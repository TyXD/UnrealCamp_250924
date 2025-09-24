---

# 🔩 [내일배움캠프 언리얼엔진] 인터페이스로 설계하는 유연한 아이템 시스템

> ## **학습 키워드**
>
> *   `Interface`: 클래스가 반드시 구현해야 할 함수 목록을 정의하는 '계약서'.
> *   `Inheritance (상속)` vs `Interface (구현)`: 기능의 '상속'과 함수의 '구현 약속'의 차이.
> *   `Polymorphism (다형성)`: 다양한 아이템 타입을 동일한 인터페이스로 제어하는 개념.
> *   `Pure Virtual Function`: 인터페이스에서 자식 클래스가 함수를 반드시 재정의하도록 강제하는 문법.
> *   `Item Hierarchy`: 아이템들의 공통 부모 클래스와 개별 아이템 클래스로 구성된 계층 구조.

<br>

---

## **1. 인터페이스(Interface) 이해하기: 유연한 설계의 첫걸음**

---

이번 챕터에서는 캐릭터와 상호작용할 다양한 아이템(코인, 지뢰, 회복 아이템 등) 시스템을 구축합니다. 

이 아이템들은 각기 다른 효과를 가지지만, 

'플레이어와 상호작용한다'는 공통된 특징이 있습니다. 

이렇게 여러 클래스가 공통된 행동 방식을 가져야 할 때, 

**인터페이스(Interface)**를 사용하면 매우 유연하고 확장 가능한 시스템을 설계할 수 있습니다.

### 1️⃣ **인터페이스란?**

**인터페이스**는 특정 클래스가 **"이러이러한 함수들을 반드시 가지고 있어야 한다"**고 

명시하는 일종의 **'설계 계약서'**입니다. 

함수의 이름과 파라미터 같은 '틀'만 정의하고, 실제 내용은 각 클래스가 자유롭게 채워 넣도록 합니다.

> **상속(Inheritance)과의 차이점**
> *   **상속**: 부모 클래스의 **구현된 기능**을 그대로 물려받습니다. (is-a 관계, 예: '강아지'는 '동물'이다)
>   
> *   **인터페이스**: 특정 **기능을 구현하겠다**는 약속만 합니다. (has-a 관계, 예: '강아지'는 '짖는' 기능을 가진다)

인터페이스를 사용하면, 서로 다른 아이템이라도 "플레이어와 닿았을 때"라는 

공통된 상황에서 `OnItemOverlap`이라는 동일한 함수를 호출하여 처리할 수 있습니다. 

이는 코드의 일관성을 높이고 유지보수를 매우 쉽게 만듭니다.

### 2️⃣ **`ItemInterface` 정의하기**

모든 아이템이 가져야 할 공통 함수들을 인터페이스로 정의해 보겠습니다.

*   **`MyItemInterface.h` (헤더 파일)**
    ```cpp
    #pragma once
    
    #include "CoreMinimal.h"
    #include "UObject/Interface.h"
    #include "MyItemInterface.generated.h"
    
    // 언리얼 리플렉션 시스템을 위한 UInterface
    UINTERFACE(MinimalAPI)
    class UMyItemInterface : public UInterface
    {
    	GENERATED_BODY()
    };
    
    // 실제 C++ 코드에서 사용할 함수들을 정의하는 IInterface
    class MYPROJECT_API IMyItemInterface
    {
    	GENERATED_BODY()
    
    public:
    	// 아이템과 다른 액터가 겹쳤을 때 호출될 함수
    	virtual void OnItemOverlap(AActor* OverlapActor) = 0;
    
    	// 아이템을 활성화(사용)할 때 호출될 함수
    	virtual void ActivateItem(AActor* Activator) = 0;
    
    	// 아이템의 타입을 반환할 함수
    	virtual FName GetItemType() const = 0;
    };
    ```
> #### 💡 **`= 0` (순수 가상 함수)**
>
> 함수 선언 끝에 `= 0`을 붙이면 **순수 가상 함수(Pure Virtual Function)**가 됩니다.
>
> 이는 "구현부가 없으니, 상속받는 클래스는 이 함수를 **반드시** 직접 구현해야 한다"는 강제성을 부여합니다.

<br>

---

## **2. 아이템 계층 구조 설계 및 구현**

---

인터페이스를 정의했다면, 이제 이를 구현할 아이템 클래스들의 계층 구조를 설계합니다.

### 1️⃣ **`ABaseItem`: 모든 아이템의 공통 부모**

모든 아이템이 공통적으로 가질 속성과 기능을 담을 부모 클래스 `ABaseItem`을 생성합니다. 

이 클래스는 `AActor`를 상속받고, 우리가 만든 `IMyItemInterface`를 구현합니다.

*   **`BaseItem.h` (헤더 파일)**
    ```cpp
    #pragma once
    
    #include "CoreMinimal.h"
    #include "GameFramework/Actor.h"
    #include "MyItemInterface.h" // 우리가 만든 인터페이스 포함
    #include "BaseItem.generated.h"
    
    UCLASS()
    class MYPROJECT_API ABaseItem : public AActor, public IMyItemInterface
    {
    	GENERATED_BODY()
    
    public:	
    	ABaseItem();
    
    protected:
    	// 아이템의 타입을 에디터에서 설정할 수 있도록 UPROPERTY로 노출
    	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item")
    	FName ItemType;
    
    	// 인터페이스에서 요구한 함수들을 override하여 구현 선언
    	virtual void OnItemOverlap(AActor* OverlapActor) override;
    	virtual void ActivateItem(AActor* Activator) override;
    	virtual FName GetItemType() const override;
    
    	// 모든 아이템이 공통으로 사용할 소멸 함수
    	virtual void DestroyItem();
    };
    ```
*   **`BaseItem.cpp` (소스 파일)**
    ```cpp
    #include "BaseItem.h"
    
    ABaseItem::ABaseItem()
    {
    	PrimaryActorTick.bCanEverTick = false;
    }
    
    // 자식 클래스에서 구체적인 내용을 채울 수 있도록 기본 구현은 비워둡니다.
    void ABaseItem::OnItemOverlap(AActor* OverlapActor) {}
    void ABaseItem::ActivateItem(AActor* Activator) {}
    
    FName ABaseItem::GetItemType() const
    {
    	return ItemType;
    }
    
    void ABaseItem::DestroyItem()
    {
    	Destroy(); // 액터를 월드에서 제거
    }
    ```

### 2️⃣ **`ACoinItem`, `AHealingItem`, `AMineItem`: 개별 아이템 구현**

이제 `ABaseItem`을 상속받아 각기 다른 특성을 가진 아이템들을 만듭니다.

*   **코인 아이템 (`CoinItem.h`, `CoinItem.cpp`)**
    > 점수(`PointValue`)라는 고유한 속성을 가지며, 활성화(`ActivateItem`)되면 자신을 파괴합니다.
    >
    > `SmallCoin`과 `BigCoin`은 이 클래스를 상속받아 점수와 타입만 다르게 설정합니다.

    ```cpp
    // BigCoinItem.cpp
    #include "BigCoinItem.h"
    
    ABigCoinItem::ABigCoinItem()
    {
        // 부모(ACoinItem)의 PointValue 변수 값을 설정
    	PointValue = 50;
        // 부모(ABaseItem)의 ItemType 변수 값을 설정
    	ItemType = "BigCoin";
    }
    
    void ABigCoinItem::ActivateItem(AActor* Activator)
    {
        // 아이템이 활성화되면 점수를 추가하는 로직 (나중에 구현) 후,
    	DestroyItem(); // 자신을 파괴
    }
    ```

*   **힐링 아이템 (`HealingItem.h`, `HealingItem.cpp`)**
    > 회복량(`HealAmount`)이라는 고유 속성을 가집니다.
    >
    > 활성화되면 플레이어의 체력을 회복시키고 파괴됩니다.

    ```cpp
    // HealingItem.cpp
    #include "HealingItem.h"
    
    AHealingItem::AHealingItem()
    {
    	HealAmount = 20.0f;
    	ItemType = "Healing";
    }
    
    void AHealingItem::ActivateItem(AActor* Activator)
    {
    	// 플레이어의 체력을 회복시키는 로직 (나중에 구현) 후,
    	DestroyItem();
    }
    ```

*   **지뢰 아이템 (`MineItem.h`, `MineItem.cpp`)**
    > 폭발 딜레이, 범위, 데미지 등 복잡한 속성을 가집니다.
    >
    > 활성화 로직 또한 즉시 파괴가 아닌, 타이머를 이용한 지연 폭발로 구현될 것입니다.

    ```cpp
    // MineItem.cpp
    #include "MineItem.h"
    
    AMineItem::AMineItem()
    {
    	ExplosionDelay = 5.0f;
    	ExplosionRadius = 300.0f;
    	ExplosionDamage = 40.0f;
    	ItemType = "Mine";
    }
    
    void AMineItem::ActivateItem(AActor* Activator)
    {
        // 타이머를 설정하여 일정 시간 뒤 폭발하는 로직 (나중에 구현)
    }
    ```

이처럼 인터페이스 기반으로 계층 구조를 설계하면, **'코인'**, **'힐링 아이템'**, **'지뢰'**처럼

완전히 다른 아이템이라도 `ActivateItem`이라는 **동일한 함수 호출**로 각자의 고유한 효과를 발동시킬 수 있습니다. 

이는 향후 새로운 아이템을 추가하거나 기존 아이템의 로직을 변경할 때 매우 큰 유연성을 제공합니다.

---

#언리얼엔진 #인터페이스 #아이템시스템 #내일배움캠프
