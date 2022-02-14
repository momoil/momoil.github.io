---
layout: post
title: "UE4_视频显示到屏幕进行调整"
date: 2022-02-14
tags: [UE4]
comments: true
author: momo
---

UE4 UE4_视频显示到屏幕进行调整

<!-- more -->
# 开始 #

# 第一步： #

- 打开UE4,新建文件夹"视频投射到屏幕"，在项目文件的目录中新建一个文件夹，自己随意取一个名字；
- 将准备好的视频放入项目文件中，然后拖入UE4"视频投射到屏幕"文件夹；

# 第二步： #

- 新建媒体播放器，和媒体纹理；
- 新建后期材质，命名为M_Video；
- 打开材质M_Video，修改材质域为后期处理材质；

# 第三步： #

- 把媒体纹理拖入后期材质；
- 找到SceneTexture节点，修改场景纹理ID为后期处理输入0；
- 使用Lerp节点把两个材质融合，提升Lerp节点的Alpha变量公开为参数Alpha；

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-14/2.png)

# 第四步： #

- 新建Actor，命名为"BP_VideoDisplayTo"
- 打开"BP_投射工具"，添加后期处理组件(PostProcess)；
- 打开构造脚本Construction Scip，为蓝图添加后期材质，节点如图所示：

![image](https://raw.githubusercontent.com/momoil/momoil.github.io/master/images/2022-2-14/1.png)

# 第五步： #

- 在场景拖入后期处理盒子，把材质放入后期处理盒子；
- 在Sequence轨道中添加 媒体轨道 ；
- 在媒体轨道中右键属性，添加之前新建的媒体纹理；

# 使用说明： #

- 修改Alpha(0-1)参数的值,可以控制视频的透明度；