---
title: 软件安装、新建工程、运行
author: lixinghui
date: 2023-3-25 12:00:00 +0800
categories: [Labwindows]
tags: [Labwindows]
---


## 一、软件安装包下载
软件在百度网盘的我的网盘》软件安装包》Labwindows2017中，把压缩包的三个部分解压出来。

## 二、软件的安装

软件安装参考以下链接：

[NI LabWindows/CVI 2017破解版详细安装教程(附破解补丁)_编程开发_软件教程_脚本之家 (jb51.net)](https://www.jb51.net/softjc/555541.html)

## 三、新建工程

点击`New`下面的`Project`,在这里默认建立了一个`Untitled`的工作区和工程，点击`File->Save Untitled as ... `，把工程另存为想要的工程名，默认勾选了`Auto Save Workspace`工作区就已经默认保存了。

再新建界面文件，点击`File->New->User Interface`，保存界面文件，默认生成的是`Untitled.uir`，点击`File->Save Untitled.uir as ...` ,设置自己喜欢的名称，保存。

在用户界面上放置控件，对控件设置用户需要的参数。

## 四、生成代码

点击`Code->Generate->All Code`，这时会弹出一个CVI Message 说要插入代码必须指定文件，想要现在指定吗，选Yes，对于创建新工程还是加入到当前工程，选择加入到当前工程内。这时需要注意，`Select QuitUserInterface Callbacks`这个选项没有手动选上，这是选择退出用户界面的回调函数，在界面中需要设置一个回调函数用来关闭界面，设置回调函数为`QUIT_CALLBACK`，再次生成的时候选上这个回调函数。

如果多次生成代码到同一个文件，会提示文件已存在，是否覆盖，这里如果已经做好了备份就点击`yes`，没有备份就自己手动备份一下。

## 五、编译和执行

编译：点击`Build->Build`，也可以按快捷键`Ctrl+M`。

执行：点击`Run->Debug main.exe`，也可以按`Shift+F5`。

也可以直接点击绿色的三角，对工程进行调试。

点击右上角的×关闭应用，和一般的应用程序一样。

## 六、设置应用的大小和名称

应用的大小可以通过直接拖动界面文件内的画布大小来修改，如放大或者缩小，也可以双击面板空白处，设置宽和高的具体值，也可以设置面板相对于左上的偏移值，设置Top和Left的值。

可以勾选`Other Attributes`中的`Scale Contents On Resize`,在调整了界面大小的时候，界面内容也会进行尺度变换，当然也可以最小的宽和高在尺度变换的时候。

默认的界面是`Untitled Panel`（未命名的面板），可以通过点击面板本身修改`Title`该项参数来修改面板的名称。

