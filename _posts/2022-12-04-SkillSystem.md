---
layout: post
title: "UEC++ 技能连击系统"
date: 2022-12-04
tags: [UE4]
comments: true
author: momo
---

UEC++ 技能连击系统

<!-- more -->

# 开始 #

Actor组件部分源码如下：

> SkillSystemComponent.h
>
> ```c++
> // Fill out your copyright notice in the Description page of Project Settings.
> 
> #pragma once
> 
> #include "CoreMinimal.h"
> #include "Components/ActorComponent.h"
> #include "Engine/DataTable.h"
> #include "GameFramework/Character.h"
> #include "SkillSystemComponent.generated.h"
> 
> //按键枚举类型
> UENUM(BlueprintType)
> enum class EInputType : uint8
> {
> 	None,
> 	MBL,
> 	MBR
> };
> 
> // Struch结构体
> USTRUCT(BlueprintType)
> struct FSkill : public FTableRowBase
> {
> 	GENERATED_USTRUCT_BODY()
> public:
> 	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "Skill")
> 		TMap<EInputType, FName> SkillNextcombo;
> 	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "Skill")
> 		float Delay;
> 	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "Skill")
> 		UAnimMontage* AnimMontage;
> };
> 
> UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
> class SKILLSYSTEM_API USkillSystemComponent : public UActorComponent
> {
> 	GENERATED_BODY()
> 
> public:	
> 	// Sets default values for this component's properties
> 	USkillSystemComponent();
> 
> protected:
> 	// Called when the game starts
> 	virtual void BeginPlay() override;
> 
> public:	
> 	// Called every frame
> 	virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
> 
> 	//debug
> 	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Skill")
> 		bool Debug = false;
> 	void PrintToScreen(FString ScreenMessage);
> 
> 	// Skill
> 	bool CanEnterNextCombo = false;
> 	bool IsConCombo = false;
> 	FSkill  CurrentSkill;
> 	EInputType  InputBuffer;
> 	// 初始连击技能
> 	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Skill")
> 		TMap<EInputType, FName> SkillList;
> 	//技能数据表
> 	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Skill")
> 		UDataTable* SkillDataTable;
> 
> 	// Function
> 	void PlaySkill(FName SkillName);
> 	UFUNCTION(BlueprintCallable, Category = "Skill")
> 		void SkillInput(EInputType InputType);
> 	UFUNCTION(BlueprintCallable, Category = "Skill")
> 		void ComboEnd();
> };
> ```
>
> 

SkillSystemComponent.cpp

```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "SkillSystemComponent.h"

// Sets default values for this component's properties
USkillSystemComponent::USkillSystemComponent()
{
	// Set this component to be initialized when the game starts, and to be ticked every frame.  You can turn these features
	// off to improve performance if you don't need them.
	PrimaryComponentTick.bCanEverTick = false;

	// ...
}


// Called when the game starts
void USkillSystemComponent::BeginPlay()
{
	Super::BeginPlay();

	// ...
	
}


// Called every frame
void USkillSystemComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	// ...
}

void USkillSystemComponent::PrintToScreen(FString ScreenMessage)
{
	if (Debug)
	{
		UE_LOG(LogTemp, Log, TEXT("%s"), *ScreenMessage);
		GEngine->AddOnScreenDebugMessage(-1, 1.f, FColor::Green, ScreenMessage);
	}
}

void USkillSystemComponent::PlaySkill(FName SkillName)
{
	PrintToScreen(L"Plugins/SkillSystem/Source/SkillSystem/Private/SkillSystemComponent.cpp  PlaySkill()");
	TArray<UAnimMontage*> SkillMontages;
	const FString ContextString = GetFullName();
	FSkill* SkillMontage = SkillDataTable->FindRow<FSkill>(SkillName, ContextString);
	if (IsValid(SkillMontage->AnimMontage))
	{
		// SkillMontage不为空时
		ACharacter* Character = Cast<ACharacter>(GetOwner());
		Character->PlayAnimMontage(SkillMontage->AnimMontage, 1.0, "None");
		IsConCombo = true;
		CurrentSkill = *SkillMontage;
		InputBuffer = EInputType::None;
		PrintToScreen(L"Plugins/SkillSystem/Source/SkillSystem/Private/SkillSystemComponent.cpp Skill release");
	}
	else 
	{
		// SkillMontage为空时
		PrintToScreen(L"Plugins/SkillSystem/Source/SkillSystem/Private/SkillSystemComponent.cpp NO SkillMontage");
		//ComboEnd();
	}
}

void USkillSystemComponent::SkillInput(EInputType InputType)
{
	if (IsConCombo)
	{
		// 可以开始连击，则选择释放下一次攻击
		if (CanEnterNextCombo)
		{
			TMap<EInputType, FName> m_SkillNextcombo = CurrentSkill.SkillNextcombo;
			FName* MyFName = m_SkillNextcombo.Find(InputType);
			PlaySkill(*MyFName);
		}
		else
		{
			// 无法开始连击，则选择缓存下一次攻击的输入
			this->InputBuffer = InputType;
		}
	}
	else
	{
		//第一次攻击
		if (SkillList.Num() != 0)
		{
			FName* MyFName = SkillList.Find(InputType);
			PlaySkill(*MyFName);
		}
	}
}

void USkillSystemComponent::ComboEnd()
{
	// 结束连击，重置所有参数
	PrintToScreen(L"ComboEnd()");
	CurrentSkill.AnimMontage = nullptr;
	CurrentSkill.SkillNextcombo.Reset();
	CanEnterNextCombo = false;
	IsConCombo = false;
	InputBuffer = EInputType::None;
}
```

动画通知代码如下：

技能开始：

```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "SkillSystemComponent.h"
#include "GameFramework/Character.h"
#include "Animation/AnimNotifies/AnimNotify.h"
#include "SkillStart.generated.h"

/**
 * 
 */
UCLASS()
class SKILLSYSTEM_API USkillStart : public UAnimNotify
{
	GENERATED_BODY()

	//UFUNCTION()
	virtual void Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation) override;
};

```

```c++
#include "SkillStart.h"

void USkillStart::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::Notify(MeshComp, Animation);
	// 执行的逻辑….
	AActor *MyOwner = MeshComp->GetOwner();
	USkillSystemComponent* SkillSystemComponent = Cast<USkillSystemComponent>(MyOwner->GetComponentByClass(USkillSystemComponent::StaticClass()));
	if (IsValid(SkillSystemComponent))
	{
		SkillSystemComponent->PrintToScreen(L"MySkilDemo/Plugins/SkillSystem/Source/SkillSystem/Private/SkillStart.cpp  SkillStart");
		SkillSystemComponent->CanEnterNextCombo = false;
	}
}
```



可进行下一个技能（只取cpp部分代码，其余内容和上面动画通知一样）

```c++
void USkillNext::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::Notify(MeshComp, Animation);
	// 执行的逻辑….
	AActor* MyOwner = MeshComp->GetOwner();
	USkillSystemComponent* SkillSystemComponent = Cast<USkillSystemComponent>(MyOwner->GetComponentByClass(USkillSystemComponent::StaticClass()));
	if (IsValid(SkillSystemComponent))
	{
		SkillSystemComponent->PrintToScreen(L"MySkilDemo/Plugins/SkillSystem/Source/SkillSystem/Private/SkillNext.cpp  SkillNext");
		SkillSystemComponent->CanEnterNextCombo = true;
	}
}

```

动画结束

```c++
void USkillEnd::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::Notify(MeshComp, Animation);
	// 执行的逻辑….
	AActor* MyOwner = MeshComp->GetOwner();
	USkillSystemComponent* SkillSystemComponent = Cast<USkillSystemComponent>(MyOwner->GetComponentByClass(USkillSystemComponent::StaticClass()));
	if (IsValid(SkillSystemComponent))
	{
		SkillSystemComponent->PrintToScreen(L"MySkilDemo/Plugins/SkillSystem/Source/SkillSystem/Private/SkillEnd.cpp  SkillEnd");
		float time = SkillSystemComponent->CurrentSkill.Delay;
		FPlatformProcess::Sleep(time);
		SkillSystemComponent->ComboEnd();
	}
}

```
