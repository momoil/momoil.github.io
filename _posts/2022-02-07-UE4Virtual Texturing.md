---
layout: post
title: "UE4_虚拟纹理挖沟渠制作方法"
date: 2022-02-07
tags: [UE4]
comments: true
author: momo
---

UE4 UE4_虚拟纹理（Virtual Texturing Methods）挖沟渠制作方法

<!-- more -->
# 开始 #


# 项目设置 #
1. 在主菜单中，选择 编辑（Edit） 菜单并选择 项目设置（Project Settings）。在 引擎（Engine） > 渲染（Rendering） > 虚拟纹理（Virtual Textures） 类目下，将 启用虚拟纹理支持（Enable Virtual texture support） 设为true。
2. 重启 项目。
![Image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/1.webp)

# 创建运行时虚拟纹理资产 #
1. 在 内容浏览器 中，利用右键点击快捷菜单或 新增（Add New） 按钮从 材质和纹理（Materials & Textures） 类目创建 运行时虚拟纹理 资产。
2. 为 运行时虚拟纹理 资产命名为 RTV_Texture。

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/2.webp)

# 创建运行时虚拟纹理材质 #
1. 新建笔刷材质 Mat_Draw，材质中添加节点Runtime virtual Texture output（运行时虚拟纹理输出）；
2. 为Runtime virtual Texture output添加一个笔刷材质；
3. 添加地形材质Mat_Landscape，打开材质细节面板，搜索D3D11曲面细分模式为PN三角形，无裂纹置换为Ture，最大置换为2048，添加节点Runtime Virtual Texture Sample 连接到世界场景位移，同时正常设置地形的材质(根据你需要的地形，进行设置地形材质即可，无特殊需求)
4. Runtime Virtual Texture Sample 的细节面板中，虚拟纹理设置为RTV_Texture；
5. 只对z轴进行偏移，可以乘一个（0,0,1）的三维向量；

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/3.png)

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/4.png)

# 运行时虚拟纹理体积 #
1. 在放置Actor中搜索到 *运行时虚拟纹理体积*，放入场景，然后在场景中选中 *运行时虚拟纹理体积*  然后点击蓝图，将选项转换为蓝图类，资产命名为BP_VirtualTexture；
2. 新建蓝图BP_Spline，添加Spline组件；
3. 我们要将虚拟纹理绘制在面片上，所以我们在该蓝图中运行时生成静态网格体组件，网格体为Plane，关闭碰撞，投射阴影等，组件材质为画笔材质，同时组件细节面板中的虚拟纹理设置为RTV_Texture；

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/5.png)

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/6.png)

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/7.png)

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-7-RVT/8.png)

# 地形设置 #
地形的细节面板中，正/负ZBounds延展都设置为1000；曲面细分组件画面大小设置为最小，关闭使用默认衰减，


# 使用说明 #
1. 地形材质设置为 Mat_Landscape ，用 BP_VirtualTexture 框住需要挖沟渠的范围，
2. 在范围内放置BP_Spline ，拖动样条线进行摆放路径，
3. 使用方式：
> 在样条线的细节面板中的 Scale 为沟渠宽度；
> 
> 在样条线的细节面板中的 Hight 为沟渠高度；
> 
> 调整BP_Spline蓝图中的TimeLine时间轴长度，可以设置多长时间绘制完成；
> 
> 修改材质Mat_Draw中的内容和图片，可以修改绘制的笔刷形状
> 
> 材质Mat_Landscape中的三维向量(1,1,1)可以修改为贴图，是沟渠挖开之后的材质样式；(备注：你也可以删除之后自己写自动化地貌材质)
> 
备注：Test文件暂未上传网络；

