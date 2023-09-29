# C++和客户端

### 1. 使用C++创建一个继承自AActor的枪支类，在这个类中，创建两个组件:骨骼模型组件，粒子组件。他们分别作为枪支外形和火舌
- 请分别写出这两个组件的声明和创建过程。
- 满足声明的宏标记，要求骨骼模型组件可在蓝图中被默认定义，粒子组件任何时候可以被定义。
- 实现中，骨骼模型作为root组件节点，粒子组件挂载到根骨骼模型上。

关于EditDefaultOnly和EditAnywhere的区别，请看[视频](https://www.youtube.com/watch?v=arCwPQEDsmI)。一句话概括就是EditAnyWhere既可以在编辑器里更改，也可以在被实例化的时候更改（比如你把你定义的Actor拖拽到World上运行）。请看[UEInterviewDemo]()里的Contents/Weapons/ShortGun。

```c++
	UPROPERTY(EditDefaultsOnly)
	TObjectPtr<USkeletalMeshComponent> WeaponSkeletalMesh;
	
	UPROPERTY(EditAnywhere)
	TObjectPtr<UNiagaraComponent> FireEffect;
```

```c++
	WeaponSkeletalMesh = CreateDefaultSubobject<USkeletalMeshComponent>("Weapon");

	SetRootComponent(WeaponSkeletalMesh);

	FireEffect = CreateDefaultSubobject<UNiagaraComponent>("FireEffect");
	FireEffect->SetupAttachment(RootComponent);
```

### 2. 请简要阐述UFUNCTION()中
- BlueprintCallable,BlueprintPure的区别是什么？
- BlueprintImplementableEvent有什么用？
- BlueprintNativeEvent有什么用？

[参考连接](https://www.tomlooman.com/unreal-engine-ufunction-specifiers/)

#### BlueprintCallable,BlueprintPure的区别是什么？

区别是BlueprintCallable有执行线，BlueprintPure没有。

[![pPqSHM9.png](https://z1.ax1x.com/2023/09/29/pPqSHM9.png)](https://imgse.com/i/pPqSHM9)

```c++
	UFUNCTION(BlueprintPure)
	int GetWeaponSize();

	UFUNCTION(BlueprintCallable)
	void DestroyWeapon();
```
#### BlueprintImplementableEvent有什么用？

BlueprintImplementableEvent只能在蓝图实现，常常与BlueprintCallable相配合使用。

#### BlueprintNativeEvent有什么用？

与BlueprintImplementableEvent有区别的是，BlueprintNativeEvent可以在C++中写Native方法，C++ Native为后缀一般以_Implementation，并可以在蓝图中重写。

详细参考见[Aura](https://github.com/Genius-pig/Aura)中的Source/Aura/Public/Iteraction/CombatInterface.h。

### 3. 请简要阐述AnimBP的动画状态机和Montage
  
- 哪一个更适合用在做近战攻击?
- 如果实现一个上下肢分离的换子弹动画(边跑步，边换弹)，应该怎么做？
- 怎么创建根骨骼动画节点，以方便在代码里调用？
- 简单阐述，Montage动画播完如何获取响应事件？

#### 他们分别适用在什么情景？ 

状态机更适用在动画过渡，比如从idle到跑步，Montage适合触发事件，Montage可以被分到不同的Slot里。Montage可以和GAS配合使用。Montage只有在AnimBP里有这个Slot和PlayMontage这两个同时存在才会播放动画。

[![pPq9l1e.md.png](https://z1.ax1x.com/2023/09/29/pPq9l1e.md.png)](https://imgse.com/i/pPq9l1e)

[![pPq967q.md.png](https://z1.ax1x.com/2023/09/29/pPq967q.md.png)](https://imgse.com/i/pPq967q)

[![pPq92NV.png](https://z1.ax1x.com/2023/09/29/pPq92NV.png)](https://imgse.com/i/pPq92NV)


#### 如果实现一个上下肢分离的换子弹动画(边跑步，边换弹)，应该怎么做？

可以用Layered blend per bone

[![pPqCpDA.png](https://z1.ax1x.com/2023/09/29/pPqCpDA.png)](https://imgse.com/i/pPqCpDA)

#### 怎么创建根骨骼动画节点，以方便在代码里调用？

[![pPqCcKH.png](https://z1.ax1x.com/2023/09/29/pPqCcKH.png)](https://imgse.com/i/pPqCcKH)

在骨骼处添加socket，c++用GetSocketLocation。

```c++
GetMesh()->GetSocketLocation(LeftHandSocketName);
```

#### 简单阐述，Montage动画播完如何获取响应事件？

[![pPqCjI0.png](https://z1.ax1x.com/2023/09/29/pPqCjI0.png)](https://imgse.com/i/pPqCjI0)

如果使用GAS的话，可以使用PlayMontageAndWait。

### 4. 请创建一个DataTable的结构，用于管理枪支系统的参数

- 创建继承自虚幻提供的FTableRowBase的结构，结构名FGunData。
- 该结构中有，枪支ID，枪支名字，子弹数目，icon图标和引用的Edit生成类。
- 请写出基于枪支ID遍历的函数。

```c++
#pragma once

#include "CoreMinimal.h"
#include "Engine/DataAsset.h"
#include "Engine/DataTable.h"
#include "WeaponInfo.generated.h"

USTRUCT(BlueprintType)
struct FGunData : public FTableRowBase
{
	GENERATED_BODY();

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	int32 WeaponID;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FName WeaponName;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	int32 BulletNumber;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	TObjectPtr<const UTexture2D> Icon = nullptr;
};

/**
 * 
 */
UCLASS()
class UEINTERVIEWDEMO_API UWeaponInfo : public UDataAsset
{
	GENERATED_BODY()
public:
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "WeaponInformation")
	TArray<FGunData> WeaponInfo;

	FGunData FindInfoForID(int32 ID);
};
```

```
#include "Data/WeaponInfo.h"

FGunData UWeaponInfo::FindInfoForID(int32 ID)
{
	for (int i = 0; i < WeaponInfo.Num(); i++)
	{
		if (WeaponInfo[i].WeaponID == ID)
		{
			return WeaponInfo[i];
		}
	}

	return FGunData();
}
```

[![pPqPIT1.png](https://z1.ax1x.com/2023/09/29/pPqPIT1.png)](https://imgse.com/i/pPqPIT1)

### 5. 请简述AI的BehaviorTree以下节点的功能
- Service，Task和Decorator的功能。
- 如果敌人寻路到玩家跟前，执行攻击，应该选择上述哪个节点进行功能实现？

#### Service，Task和Decorator的功能。

Service和Decorator都依附于Composite和Task。Decorator最经典的就是Blackboard，根据Blackboard的值来决定是否执行分支。Service最经典的就是Run EQS，对周边环境进行查询。

Decorator：条件节点，是否执行分支。
Service：非条件节点，执行服务，并以自己的频率。

Task：任务节点，可以包含多个Decorator和Service。可以理解成树中的叶子节点。
Composite：分支节点，可以包含多个Decorator和Service。可以理解为树的分支节点。

详情请见我的[知乎文章](https://zhuanlan.zhihu.com/p/643179445)

#### 如果敌人寻路到玩家跟前，执行攻击，应该选择上述哪个节点进行功能实现？

首先实现一个Service叫FindNearPlayers（啥名都可以）在Composite，如果找到了在Task上实现攻击。

[![pPqiJB9.png](https://z1.ax1x.com/2023/09/29/pPqiJB9.png)](https://imgse.com/i/pPqiJB9)

具体AI实现可看Aura的Content\Blueprints\AI\BehaviorTree

### 6. 什么是虚幻引擎蓝图？请简要解释其作用和用途。

图形化编程，可以实现虚幻大部分功能。

### 7. 请解释虚幻引擎蓝图中的事件（Event）图和脚本图之间的区别。

Event有点像函数，它是执行一个网络的开端。脚本就是每个蓝图节点。

[![pPqiw9K.png](https://z1.ax1x.com/2023/09/29/pPqiw9K.png)](https://imgse.com/i/pPqiw9K)

### 8. 如何创建一个新的虚幻引擎蓝图类？

写个c++基类，然后继承这个c++类去创建一个蓝图。

### 9. 你能解释蓝图的变量是什么吗？它们有哪些不同的数据类型？

蓝图变量相当于C++ class里的成员变量，蓝图本地变量相当于C++ 函数内变量。可以有任何虚幻支持的数据类型。

### 10. 如何在虚幻引擎蓝图中创建自定义函数？

[![pPqif9f.png](https://z1.ax1x.com/2023/09/29/pPqif9f.png)](https://imgse.com/i/pPqif9f)

点那个加号。

### 11. 什么是蓝图接口？请提供一个使用蓝图接口的示例。
























