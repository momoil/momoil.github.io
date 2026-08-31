---
layout: post
title: "pyautogui 常用工具操作"
date: 2026-08-31
tags: [Python]
comments: true
author: momo
---

PyAutoGUI 常用的屏幕、鼠标操作整理。

<!-- more -->

# 屏幕大小

```python
size = pyautogui.size()
print(size)
```

# 鼠标位置

```python
mouse_pos = pyautogui.position()
print(mouse_pos)
```

# 判断点是否在屏幕内

```python
print(pyautogui.onScreen(100,100))
```

# 鼠标移动

```python
size = pyautogui.size()
#把鼠标移动到(10,10)的位置，周期1秒
pyautogui.moveTo(10,10, duration=1)
#把鼠标移动到画面中央，周期0.5秒
pyautogui.moveTo(size.width/2, size.height/2, duration=0.5)
#鼠标相对移动，周期1秒
pyautogui.moveRel(100,None, duration=1)
```

- moveTo：绝对坐标
- moveRel：相对坐标
- `pyautogui.moveRel(x轴数值,y轴数值, <duration=1>时间)`

# 实时获取鼠标位置

```python
#上一次的位置
last_pos = pyautogui.position()
try:
    while True:
        #新位置
        new_pos = pyautogui.position()
        if last_pos != new_pos:
            print(new_pos)
            last_pos = new_pos
except KeyboardInterrupt:
    print("'\nExit.'")
```

# 鼠标按键

```python
pyautogui.mouseDown()   #按下鼠标按键（左键）

pyautogui.mouseUp()     #释放鼠标按键（左键）

pyautogui.click()       
#向计算机发送虚拟的鼠标点击(click()函数只是前面两个函数调用的方便封装)
#默认在当前光标位置，使用鼠标左键点击

pyautogui.click([x,y,button='left/right/middle'])  

#在(x,y)处点击鼠标左键、右键、中键
#但不推荐使用这种方法，下面这种方法效果更好
#pyautogui.moveTo(x,y,duration=t)
#pyautogui.click()

pyautogui.doubleClick() #双击鼠标左键

pyautogui.rightClick()  #单击鼠标右键

pyautogui.middleClick() #单击鼠标中键
```

# 根据图片定位点击

```python
#系统准备时间
time.sleep(2)
#取得帮助菜单位置
help_pos = pyautogui.locateOnScreen('C:\Users\baichuanjiang\PycharmProjects\test\help.png')
goto_pos = pyautogui.center(help_pos)
print(goto_pos)
#移动鼠标
#pyautogui.moveTo(goto_pos, duration=1)
#点击
#pyautogui.click()
#再移动鼠标
#pyautogui.moveRel(None, 650, duration=1)
#再点击
#pyautogui.click()
```
