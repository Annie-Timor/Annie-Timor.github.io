---
title: 独立EXE文件导出
author: lixinghui
date: 2023-3-25 12:00:00 +0800
categories: [Labwindows]
tags: [Labwindows]
---


参考链接：

[Labwindows打包制作Setup安装程序的步骤_cvi软件打包发布_小坏坏_的博客-CSDN博客](https://blog.csdn.net/zhou8400/article/details/108103054)



在我使用的`Libwindows/CVI`和Matlab的`APP Designer`在导出到没有软件环境包的电脑时，都需要包含自身的运行环境到没有软件的电脑中去。也就是需要给目标电脑首先安装运行环境。

在`Libwindows`中制作setup的安装包步骤为：
1）点击`Build->Distributions->Manage Distributions`

2）默认是没有的，需要点击`New`来新建一个，设置应用的名字和安装的类型。

3）完成之后会默认进入Edit Install界面，有六个大的板块需要确认，分别是`General,Files,Shortcuts,Drivers&Components,Registry Keys,Advanced`。

4.1）**General**：

​	可以设置应用名称，输出路径，版本号，`Install Dialog Options`选项栏可以选择语言、readme文件、license、welcome的背景图片和标题等等。

4.2）**File**：

​	上本部分是工程开发文件，下半部分是设置安装的文件和目录，将需要安装的文件通过`add file`按钮添加下来就可以了。

4.3）**Shortcuts**：

​	该选项是用户安装的时候在哪些地方生成快捷方式，默认只在开始菜单创建快捷方式，可以点击右键添加新的快捷方式。桌面的设置为`Target File=[Program Files]\Main\main.exe`，就是链接文件为可执行文件就可以了，`Destination Directory=[Desktop]`

4.4）**Drivers&Components**：

​	这一项就是驱动和组件了，应用程序依赖于哪些东西就需要包含哪些依赖库，把需要的库勾选上就可以了。

4.5）**Registry Keys**：

​	这个为注册表信息？一般不做修改

4.6）**Advanced**：

​	这个注意修改最小支持的安装版本，我的labwindows2017可设置WIN7,WIN8,WIN10等版本。还可以设置安装文件概要。

5）生成安装文件

​	在上面的步骤都完成之后，点击`Build->Distributions->Build Main`，就会开始生成安装文件。生成完成之后，安装文件会在`cvidistkit.Main->Volume`这个文件夹内。

生成了安装文件夹之后，可以拷贝这个文件夹给需要的电脑去安装程序就好了。



在其他电脑已经安装了依赖环境之后，下次生成可执行文件就不需要这么麻烦了，直接把工程下的`.exe`文件和`.uir`文件拷贝到目标电脑就可以了。在有依赖环境之后，可以直接运行。





​	
