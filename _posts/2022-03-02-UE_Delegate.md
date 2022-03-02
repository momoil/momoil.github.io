---
layout: post
title: "UEC++ 代理"
date: 2022-03-02
tags: [UE4]
comments: true
author: momo
---

UEC++ UEC++ 代理

<!-- more -->

# 开始 #

# 单播代理 #

>     //单播代理 无参数
>     DECLARE_DELEGATE(FTestDelegateNoparam);
>     //单播代理 两个参数
>     DECLARE_DELEGATE_TwoParams(FTestDelegateTwoparams, float, const FString&);
>     //单播代理 一个返回值,两个参数
>     DECLARE_DELEGATE_RetVal_TwoParams(int32, FTestDelegateTwoparamsRetVal, float, const FString&);
>     
>     //定义了一个代理的成员变量
>     FTestDelegateTwoparamsRetVal DelegateTwoparamsRetVal;
>     
	
## 代理绑定函数 ##

>     //定义一个函数指针
>     typedef int32(*p)();
>     
>     // Called when the game starts or when spawned
>     void AMyActor::BeginPlay()
>     {
>     	Super::BeginPlay();
>     
>     	//绑定函数
>     	DelegateTwoparamsRetVal.BindUObject(this, &AMyActor::Fun);
>     
>     	//判断是否已经绑定
>     	if (DelegateTwoparamsRetVal.IsBound())
>     	{
>     		//执行函数
>     		DelegateTwoparamsRetVal.Execute(12, "Hello Delegate！");
>     	}
>     	/*
>     	*对于无参数的绑定可以这样执行，等于判定和执行的结合
>     	*DelegateTwoparamsRetVal.ExecuteIfBound();
>     	*/
>     
>     	/*
>     	*匿名函数
>     	* "="是以值传递的方式[=]
>     	* "&"是以引用的方式传递的[&a ,&b][this]
>     	* mutable,不加无法修改值，加了才可以修改
>     	* ->int32 表示返回值为int32
>     	*/
>     
>     	/*
>     	DelegateTwoparamsRetVal.BindLambda([this](float a, const FString& s) ->int32
>     		{
>     			return 1;
>     		});
>     	*/
>     
>     }

## 绑定函数的方式 ##

	//DelegateTwoparamsRetVal.BindLambda(匿名函数);									绑定匿名函数的代理
	//DelegateTwoparamsRetVal.BindRaw(类的指针,类名::方法名);							绑定原生C++类的代理
	//TSharedPtr<类名> A1 = MakeShareable(new 类名);									创建共享指针
	//DelegateTwoparamsRetVal.BindSP(A1.ToSharedRef(),&类名::方法名);					绑定一个共享指针
	//DelegateTwoparamsRetVal.BindStatic();											绑定一个静态函数
	//TSharedPtr<类名,ESPMode::ThreadSafo> A2 = MakeShareable(new 类名);				创建线程安全的指针
	//DelegateTwoparamsRetVal.BindThreadSafeSP(A2.ToSharedRef(),&类名::方法名);		绑定一个线程安全的共享指针
	//DelegateTwoparamsRetVal.BindUFunction(this, FName("Fun"));	//绑定一个UFunction的方法,通过函数名去绑定，用到了反射的东西，函数声明需要加UFunction宏
	//DelegateTwoparamsRetVal.BindUObject() //绑定一个继承UObject类型的函数
	//DelegateTwoparamsRetVal.Unbind() //解除绑定

# 多播代理 #

    //多播代理
    //注意多播代理上绑定的函数执行的顺序是不确定的
    //多播代理是没有返回值的
    DECLARE_MULTICAST_DELEGATE(FTestMutliDelegateNoparam);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FTestMutliDelegateTwoparams, float, const FString&);


	//定义了一个多播代理的成员变量
	FTestMutliDelegateTwoparams MutliDelegate;


	//多播代理的绑定
	MutliDelegate.AddUObject(this, &AMyActor::Fun1);
	//多播代理的执行
	MutliDelegate.Broadcast(5.2, "Hello Delegate！");

# 动态单播代理 #
    
    //动态单播代理
    //可以暴露给蓝图进行使用
    //参数需要一个名字，需要暴露给蓝图使用
	//DECLARE_DYNAMIC_DELEGATE_RetVal_OneParam(int32, FDyStandardDelegate, int32, a);
    
    // Declare dynamic standard delegate class
    DECLARE_DYNAMIC_DELEGATE(FDyStandardDelegate);
    
    // TestFunction for dynamic standard delegate
    UFUNCTION(BlueprintCallable)
    void TestFunc_DynamicStandardDelegate(const FDyStandardDelegate& MyDyStandardDelegate)
    {
    	UE_LOG(LogTemp, Error, TEXT("Dynamic standard delegate!"));
    }

# 绑定UFUNCTION标记的函数 #

- 只能绑定UFUNCTION标记的函数
    
>     // TestFunction for dynamic standard delegate bind ufunction
>     UFUNCTION()
>     	void TestFunc_DynamicStandardDelegate_BindUFunction()
>     {
>     	UE_LOG(LogTemp, Error, TEXT("Dynamic standard delegate bind ufuntion!"));
>     }
>     
>     const FDyStandardDelegate& MyDyStandardDelegateBindUFuntion
>     MyDyStandardDelegateBindUFuntion.BindUFunction(this, "TestFunc_DynamicStandardDelegate_BindUFunction");


# 动态多播代理 #

- 动态多播代理没有返回值

>     // 声明一个无参数、无返回值的代理
>     DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnTestOneDelegate);
>     
>     // 声明一个有一个参数、无返回值的代理
>     DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnTestTwoDelegate, bool, bTest);


----------


>     // Declare dynamic multicast delegate class
>     DECLARE_DYNAMIC_MULTICAST_DELEGATE(FDyMulticastDelegate);
>     DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FDyMulticastDelegateOneParam, FString, Str);
>     
>     // Declare dynamic standard delegate object
>     UPROPERTY(BlueprintAssignable)
>     	FDyMulticastDelegate MyDyMulticastDelegate;
>     
>     UPROPERTY(BlueprintAssignable)
>     	FDyMulticastDelegateOneParam MyDyMulticastDelegateOneParam;

## C++绑定 ##

>     // TestFunction for dynamic multicast delegate
>     UFUNCTION(BlueprintCallable)
>     void TestFunc_DynamicMulticastDelegate(const FDyStandardDelegate& MyDyStandardDelegate1, const FDyStandardDelegate& MyDyStandardDelegate2)
>     {
>     	UE_LOG(LogTemp, Error, TEXT("Dynamic multicast delegate!"));
>     	MyDyMulticastDelegate.Add(MyDyStandardDelegate1);
>     	MyDyMulticastDelegate.Add(MyDyStandardDelegate2);
>     }
>     //动态多播代理执行
>     MyDyMulticastDelegate.Broadcast();