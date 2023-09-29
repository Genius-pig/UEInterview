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

区别是BlueprintCallable有执行线，BlueprintPure没有。

[![pPqSHM9.png](https://z1.ax1x.com/2023/09/29/pPqSHM9.png)](https://imgse.com/i/pPqSHM9)

```c++
	UFUNCTION(BlueprintPure)
	int GetWeaponSize();

	UFUNCTION(BlueprintCallable)
	void DestroyWeapon();
```
BlueprintImplementableEvent只能在蓝图实现。与BlueprintImplementableEvent有区别的是，BlueprintNativeEvent可以在C++中写Native方法，C++ Native为后缀一般以_Implementation，并可以在蓝图中重写。

详细参考见[Aura](https://github.com/Genius-pig/Aura)中的Source/Aura/Public/Iteraction/CombatInterface.h。






