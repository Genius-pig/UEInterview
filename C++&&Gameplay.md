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







