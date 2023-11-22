---
title: Python-Tkinter教程
author: lixinghui
date: 2023-6-14 12:00:00 +0800
categories: [Note, Tkinter]
tags: [学习资料]
---




## 控件类型

下表列出了 Tkinter 中常用的 15 个控件：

| 控件类型    | 控件名称         | 控件作用                                                     |
| ----------- | ---------------- | ------------------------------------------------------------ |
| Button      | 按钮             | 点击按钮时触发/执行一些事件（函数）                          |
| Canvas      | 画布             | 提供绘制图，比如直线、矩形、多边形等                         |
| Checkbutton | 复选框           | 多项选择按钮，用于在程序中提供多项选择框                     |
| Entry       | 文本框输入框     | 用于接收单行文本输入                                         |
| Frame       | 框架（容器）控件 | 定义一个窗体（根窗口也是一个窗体），用于承载其他控件，即作为其他控件的容器 |
| Lable       | 标签控件         | 用于显示单行文本或者图片                                     |
| LableFrame  | 容器控件         | 一个简单的容器控件，常用于复杂的窗口布局。                   |
| Listbox     | 列表框控件       | 以列表的形式显示文本                                         |
| Menu        | 菜单控件         | 菜单组件（下拉菜单和弹出菜单）                               |
| Menubutton  | 菜单按钮控件     | 用于显示菜单项                                               |
| Message     | 信息控件         | 用于显示多行不可编辑的文本，与 Label控件类似，增加了自动分行的功能 |
| messageBox  | 消息框控件       | 定义与用户交互的消息对话框                                   |
| OptionMenu  | 选项菜单         | 下拉菜单                                                     |
| PanedWindow | 窗口布局管理组件 | 为组件提供一个框架，允许用户自己划分窗口空间                 |
| Radiobutton | 单选框           | 单项选择按钮，只允许从多个选项中选择一项                     |
| Scale       | 进度条控件       | 定义一个线性“滑块”用来控制范围，可以设定起始值和结束值，并显示当前位置的精确值 |
| Spinbox     | 高级输入框       | Entry 控件的升级版，可以通过该组件的上、下箭头选择不同的值   |
| Scrollbar   | 滚动条           | 默认垂直方向，鼠标拖动改变数值，可以和 Text、Listbox、Canvas等控件配合使用 |
| Text        | 多行文本框       | 接收或输出多行文本内容                                       |
| Toplevel    | 子窗口           | 在创建一个独立于主窗口之外的子窗口，位于主窗口的上一层，可作为其他控件的容器 |


在后续内容中，我们会陆续对上表中涉及的控件进行介绍。当然，除了上述控件外，还有一些高级控件，比如 PanedWindow、messagebox、LableFrame、Spinbox，在后续章节也会讲解。

---

## 控件基本属性

从上表来看，每个控件都有着各自不同的功能，即使有些控件功能相似，但它们的适用场景也不同。

在 Tkinter 中不同的控件受到各自参数的约束（即参数），所有控件既有相同属性，也有各自独有的属性。本节内容，先对这些控件的共用属性做简单介绍，如下表所示：



| 属性名称    | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| anchor      | 定义控件或者文字信息在窗口内的位置                           |
| bg          | bg 是 background 的缩写，用来定义控件的背景颜色，参数值可以颜色的十六进制数，或者颜色英文单词 |
| bitmap      | 定义显示在控件内的位图文件                                   |
| borderwidth | 定于控件的边框宽度，单位是像素                               |
| command     | 该参数用于执行事件函数，比如单击按钮时执行特定的动作，可将执行用户自定义的函数 |
| cursor      | 当鼠标指针移动到控件上时，定义鼠标指针的类型，字符换格式，参数值有 crosshair（十字光标）watch（待加载圆圈）plus（加号）arrow（箭头）等 |
| font        | 若控件支持设置标题文字，就可以使用此属性来定义，它是一个数组格式的参数 (字体,大小，字体样式) |
| fg          | fg 是 foreground 的缩写，用来定义控件的前景色，也就是字体的颜色 |
| height      | 该参数值用来设置控件的高度，文本控件以字符的数目为高度（px），其他控件则以像素为单位 |
| image       | 定义显示在控件内的图片文件                                   |
| justify     | 定义多行文字的排列方式，此属性可以是 LEFT/CENTER/RIGHT       |
| padx/pady   | 定义控件内的文字或者图片与控件边框之间的水平/垂直距离        |
| relief      | 定义控件的边框样式，参数值为FLAT（平的）/RAISED（凸起的）/SUNKEN（凹陷的）/GROOVE（沟槽桩边缘）/RIDGE（脊状边缘） |
| text        | 定义控件的标题文字                                           |
| state       | 控制控件是否处于可用状态，参数值默认为 NORMAL/DISABLED，默认为 NORMAL（正常的） |
| width       | 用于设置控件的宽度，使用方法与 height 相同                   |

注意：对于上述属性先做大体的了解即可，因为后续章节会对这些控件做更为详细的介绍。

---



-   导入python库

```python
import tkinter as tk
```
## 主窗口控件

1、创建主窗口

```python
root_window =tk.Tk()
```

2、开启主循环，让窗口一直显示

```
root_window.mainloop()
```

3、设置窗口名称

```
root_window.title('hello world')
```

4、设置窗口大小:宽x高,注,此处不能为 "\*",必须使用 "x"

```
root_window.geometry('450x300')
```

4.1、设置窗口显示在屏幕中间

```python
# 设置窗口大小变量
width = 300
height = 300
# 窗口居中，获取屏幕尺寸以计算布局参数，使窗口居屏幕中央
screenwidth = root_window.winfo_screenwidth()
screenheight = root_window.winfo_screenheight()
size_geo = '%dx%d+%d+%d' % (width, height, (screenwidth-width)/2, (screenheight-height)/2)
root_window.geometry(size_geo)
```

5、更改左上角窗口的的icon图标

```
root_window.iconbitmap('aa.ico')
```

6、设置主窗口的背景颜色

颜色值可以是英文单词，或者颜色值的16进制数,除此之外还可以使用Tk内置的颜色常量

```
root_window["background"] = "#C9C9C9"
```

7、当用户点击关闭窗口时，窗口不会关闭，而是触发回调函数。


```python
# 定义回调函数，当用户点击窗口x退出时，执行用户自定义的函数
def QueryWindow():
    print("Query Window")
    root_window.quit()

# window.protocol("协议名",回调函数)
root_window.protocol('WM_DELETE_WINDOW', QueryWindow)
```



## Label标签控件

1、显示文字

```python
# 添加文本内,设置字体的前景色和背景色，和字体类型、大小
text=tk.Label(root_window,text="C语言中文网，欢迎您",bg="yellow",fg="red",font=('Times', 20, 'bold italic'))
# 将文本内容放置在主窗口内
text.pack()
```

2、显示图片

```python
from PIL import Image, ImageTk

# 加载PNG图片
image = Image.open("image.png")

# 将图片转换为Tkinter可用的格式
photo = ImageTk.PhotoImage(image)

# 创建Label控件，并显示图片
label = tk.Label(window, image=photo)
label.pack()
```

## Button按键控件

1、设置按键及回调函数，`command`参数代表的是回调函数

```python
# 添加按钮，以及按钮的文本，并通过command 参数设置关闭窗口的功能
button=tk.Button(root_window,text="关闭",command=root_window.quit)
# 将按钮放置在主窗口内
button.pack()
```

2、获取按键文本

```
text=button['text']
```

3、设置按键属性

```
button.config(text="关闭",bg="white")
```

4、设置按键的长宽

```
button=tk.Button(root_window,text="打开",command=ButtonCallback,bg="green",width=20, height=5)
```

## 动态变量

在界面编程的过程中，有时我们需要“动态跟踪”一些变量值的变化，从而保证值的变换及时的反映到显示界面上，但是 Python 内置的数据类型是无法这一目的的，因此使用了 Tcl 内置的对象，我们把这些方法创建的数据类型称为“动态类型”

类型有： StringVar() 、BooleanVar()、DoubleVar()、IntVar()等等

```py
# -*- coding:utf-8 -*-

import tkinter as tk
import time

root = tk.Tk()
# root.title("C语言中文网")
# root.iconbitmap('C:/Users/Administrator/Desktop/C语言中文网logo.ico')
root.geometry('450x150+100+100')
root.resizable(0,0)
root.title('我们的时钟')

# 获取时间的函数
def gettime():
    # 获取当前时间
    dstr.set(time.strftime("%H:%M:%S"))
    # 每隔 1s 调用一次 gettime()函数来获取时间
    root.after(1000, gettime)
# 生成动态字符串
dstr = tk.StringVar()
# 利用 textvariable 来实现文本变化
lb = tk.Label(root,textvariable=dstr,fg='green',font=("微软雅黑",85))
lb.pack()
# 调用生成时间的函数
gettime()
# 显示窗口
root.mainloop()
```

## Entry单行文本控件

Entry 控件还提供了一些常用的方法，如下所示：

| 方法                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| delete()            | 根据索引值删除输入框内的值                                   |
| get()               | 获取输入框内的是                                             |
| set()               | 设置输入框内的值                                             |
| insert()            | 在指定的位置插入字符串                                       |
| index()             | 返回指定的索引值                                             |
| select_clear()      | 取消选中状态                                                 |
| select_adujst()     | 确保输入框中选中的范围包含 index 参数所指定的字符，选中指定索引和光标所在位置之前的字符 |
| select_from (index) | 设置一个新的选中范围，通过索引值 index 来设置                |
| select_present()    | 返回输入框是否有处于选中状态的文本，如果有则返回 true，否则返回 false。 |
| select_to()         | 选中指定索引与光标之间的所有值                               |
| select_range()      | 选中指定索引与光标之间的所有值，参数值为 start,end，要求 start 必须小于 end。 |


注意：在 Entry 控件中，我们可以通过以下方式来指定字符的所在位置：

-   数字索引：表示从 0 开始的索引数字；
-   "ANCHOE"：在存在字符的情况下，它对应第一个被选中的字符；
-   "END"：对应已存在文本中的最后一个位置；
-   "insert(index,'字符')：将字符插入到 index 指定的索引位置。



指定文本框内容以何种样式的字符显示，比如密码可以将值设为 show="*"



1、获取控件内容

```
entry1=tk.Entry(root,width=40)
en_in=entry1.get()
```

2、设置控件内容

```
entry1=tk.Entry(root,width=40)
entry1.insert(0,"请输入你想输入的:")
```

3、删除控件内容

```
entry1.delete(0,"end") #开始，结束
```



## Text文本框控件

Text 中的方法有几十个之多，这里不进行一一列举，主要对常用的方法进行介绍，如下表所示：

| 方法                            | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| bbox(index)                     | 返回指定索引的字符的边界框，返回值是一个 4 元组，格式为(x,y,width,height) |
| edit_modified()                 | 该方法用于查询和设置 modified 标志（该标标志用于追踪 Text 组件的内容是否发生变化） |
| edit_redo()                     | “恢复”上一次的“撤销”操作，如果设置 undo 选项为 False，则该方法无效。 |
| edit_separator()                | 插入一个“分隔符”到存放操作记录的栈中，用于表示已经完成一次完整的操作，如果设置 undo 选项为 False，则该方法无效。 |
| get(index1, index2)             | 返回特定位置的字符，或者一个范围内的文字。                   |
| image_cget(index, option)       | 返回 index 参数指定的嵌入 image 对象的 option 选项的值，如果给定的位置没有嵌入 image 对象，则抛出 TclError 异常 |
| image_create()                  | 在 index 参数指定的位置嵌入一个 image 对象，该 image 对象必须是 Tkinter 的 PhotoImage 或 BitmapImage 实例。 |
| insert(index, text)             | 在 index 参数指定的位置插入字符串，第一个参数也可以设置为 INSERT，表示在光标处插入，END 表示在末尾处插入。 |
| delete(startindex [, endindex]) | 删除特定位置的字符，或者一个范围内的文字。                   |
| see(index)                      | 如果指定索引位置的文字是可见的，则返回 True，否则返回 False。 |

```python
# 创建一个文本控件
# width 一行可见的字符数；height 显示的行数
text = Text(win, width=50, height=30, undo=True, autoseparators=False)
# 适用 pack(fill=X) 可以设置文本域的填充模式。比如 X表示沿水平方向填充，Y表示沿垂直方向填充，BOTH表示沿水平、垂直方向填充
text.pack()
```

1、插入文本

```
# 添加一些文本内容
for i in range(100):
    text.insert(tk.END, f"Line {i+1}\n")
```

2、删除文本

```
def clear_text():
    text.delete("1.0",tk.END) #第一行第一列开始，一直删除到末尾
```

这里的插入，获取，删除，索引内容都是"行.列"，例如第一行第三列为"1.3"

3、查看最新一行的输出

```
text.insert('end', 'New line of text\n')
text_box.see('end')
```

The `insert_text()` function inserts a new line of text at the end of the text widget using the `insert()` method with the `'end'` index, and then calls the `see()` method with the `'end'` index to scroll the view to the end of the text.

## Listbox控件

下面对列表框控件（Listbox）的常用方法做简单的介绍：

| 方法                            | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| activate(index)                 | 将给定索引号对应的选项激活，即文本下方画一条下划线           |
| bbox(index)                     | 返回给定索引号对应的选项的边框，返回值是一个以像素为单位的 4 元祖表示边框：(xoffset, yoffset, width, height)， xoffset 和 yoffset 表示距离左上角的偏移位置 |
| curselection()                  | 返回一个元组，包含被选中的选项序号（从 0 开始）              |
| delete(first, last=None)        | 删除参数 first 到 last 范围内（包含 first 和 last）的所有选项 |
| get(first, last=None)           | 返回一个元组，包含参数 first 到 last 范围内（包含 first 和 last）的所有选项的文本 |
| index(index)                    | 返回与 index 参数相应选项的序号                              |
| itemcget(index, option)         | 获得 index 参数指定的项目对应的选项（由 option 参数指定）    |
| itemconfig(index, **options)    | 设置 index 参数指定的项目对应的选项（由可变参数 **option 指定） |
| nearest(y)                      | 返回与给定参数 y 在垂直坐标上最接近的项目的序号              |
| selection_set(first, last=None) | 设置参数 first 到 last 范围内（包含 first 和 last）选项为选中状态，使用 selection_includes(序号) 可以判断选项是否被选中。 |
| size()                          | 返回 Listbox 组件中选项的数量                                |
| xview(*args)                    | 该方法用于在水平方向上滚动 Listbox 组件的内容，一般通过绑定 Scollbar 组件的 command 选项来实现。 如果第一个参数是 "moveto"，则第二个参数表示滚动到指定的位置：0.0 表示最左端，1.0 表示最右端；如果第一个参数是 "scroll"，则第二个参数表示滚动的数量，第三个参数表示滚动的单位（可以是 "units" 或 "pages"），例如：xview("scroll", 2, "pages")表示向右滚动二行。 |
| yview(*args)                    | 该方法用于在垂直方向上滚动 Listbox 组件的内容，一般通过绑定 Scollbar 组件的 command 选项来实现 |


除了共有属性之外，列表框控件也有一些其他属性，如下表所示：

| 属性              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| listvariable      | 1. 指向一个 StringVar 类型的变量，该变量存放 Listbox 中所有的项目 2. 在 StringVar 类型的变量中，用空格分隔每个项目，例如 var.set("c c++ java python") |
| selectbackground  | 1. 指定当某个项目被选中的时候背景颜色，默认值由系统指定      |
| selectborderwidth | 1. 指定当某个项目被选中的时候边框的宽度 2. 默认是由 selectbackground 指定的颜色填充，没有边框 3. 如果设置了此选项，Listbox 的每一项会相应变大，被选中项为 "raised" 样式 |
| selectforeground  | 1. 指定当某个项目被选中的时候文本颜色，默认值由系统指定      |
| selectmode        | 1. 决定选择的模式，tk 提供了四种不同的选择模式，分别是："single"（单选）、"browse"（也是单选，但拖动鼠标或通过方向键可以直接改变选项）、"multiple"（多选）和 "extended"（也是多选，但需要同时按住 Shift 键或 Ctrl 键或拖拽鼠标实现），默认是 "browse" |
| setgrid           | 指定一个布尔类型的值，决定是否启用网格控制，默认值是 False   |
| takefocus         | 指定该组件是否接受输入焦点（用户可以通过 tab 键将焦点转移上来），默认值是 True |
| xscrollcommand    | 为 Listbox 组件添加一条水平滚动条，将此选项与 Scrollbar 组件相关联即可 |
| yscrollcommand    | 为 Listbox 组件添加一条垂直滚动条，将此选项与 Scrollbar 组件相关联即可 |

1、添加删除元素

```python
# 创建Listbox，通过 listvariable来传递变量
lb = tk.Listbox(window,selectmode = "single")
# 新建一个序列，然后将值循环添加到Listbox控件中
items = ["C", "Java", "Python", "C#", "Golang", "Runby"]
for i in items:
    lb.insert('end', i)  # 从最后一个位置开始加入值
lb.insert(0, '编程学习')  # 在第一个位置插入一段字符串
# lb.delete(4)  # 删除第2个位置处的索引
lb.pack()
```

2、选择元素

```
lb.select_set(0)
```

3、获取选择元素

```
val = lb.get(lb.curselection())
```

## Combobox下拉菜单控件

需要注意的是 Combobox 并不包含在 tkinter 模块中，而是包含在`tkinter.ttk`子模块中，需要导入ttk包

```
from tkinter import ttk
```

示例代码：

```python
# 创建下拉菜单
cbox = ttk.Combobox(win)
# 使用 grid() 来控制控件的位置
cbox.grid(row = 1, sticky="NW")
# 设置下拉菜单中的值
cbox['value'] = ('C','C#','Go','Python','Java')

#通过 current() 设置下拉菜单选项的默认值
cbox.current(3)

# 编写回调函数，绑定执行事件,向文本插入选中文本
def func(event):
    text.insert('insert',cbox.get()+"\n")
# 绑定下拉菜单事件
cbox.bind("<<ComboboxSelected>>",func)

select=cbox.get()
```

## Radiobutton单选框控件

示例代码：

```py
site = [('美团外卖',1),
        ('饿了么外卖',2),
        ('美团闪购',3),
        ('艾奇外卖',4)]
# IntVar() 用于处理整数类型的变量
v = tk.IntVar()
# 重构后的写法，也非常简单易懂
for name, num in site:
    radio_button = tk.Radiobutton(window,text = name, variable = v,value =num)
    radio_button.pack(anchor ='w')

def get_variable():
    print(f"v={v.get()}")

button=tk.Button(text="get value",command=get_variable)
button.pack()
```



## Checkbutton复选框控件

示例代码：

```
# 新建整型变量
CheckVar1 = tk.IntVar()
CheckVar2 = tk.IntVar()
CheckVar3 = tk.IntVar()
# 设置三个复选框控件，使用variable参数来接收变量
check1 = Checkbutton(win, text="Python",font=('微软雅黑', 15,'bold'),variable = CheckVar1,onvalue=1,offvalue=0)
check2 = Checkbutton(win, text="C语言",font=('微软雅黑', 15,'bold'),variable = CheckVar2,onvalue=1,offvalue=0)
check3 = Checkbutton(win, text="Java",font=('微软雅黑', 15,'bold'),variable = CheckVar3,onvalue=1,offvalue=0)
# 选择第一个为默认选项
# check1.select ()
check1.pack (side = LEFT)
check2.pack (side = LEFT)
check3.pack (side = LEFT)
# 定义执行函数
def study():
    # 没有选择任何项目的情况下
    if (CheckVar1.get() == 0 and CheckVar2.get() == 0 and CheckVar3.get() == 0):
        s = '您还没选择任语言'
    else:
        s1 = "Python" if CheckVar1.get() == 1 else ""
        s2 = "C语言" if CheckVar2.get() == 1 else ""
        s3 = "Java" if CheckVar3.get() == 1 else ""
        s = "您选择了%s %s %s" % (s1, s2, s3)
     #设置标签lb2的字体
    lb2.config(text=s)
btn = Button(win,text="选好了",bg='#BEBEBE',command=study)
btn.pack(side = LEFT)
# 该标签，用来显示选择的文本
lb2 = Label(win,text='',bg ='#9BCD9B',font=('微软雅黑', 11,'bold'),width = 5,height=2)
lb2.pack(side = BOTTOM, fill = X)
```

## Scale滑块控件

示例代码：

```python
# 创建一个文本标签
label = tk.Label(window, bg='#9FB6CD',width=18, text='')
label.grid(row =2)
# 创建执行函数
def select_price(value):
    label.config(text='您购买的数量是 ' + value)
# 创建 Scale控件
scale = tk.Scale(window,
             label='选择您要购买的数量',
             from_=1,
             to= 100,
             orient=tk.HORIZONTAL,   # 设置Scale控件平方向显示
             length=400,
             tickinterval=9,       # 设置刻度滑动条的间隔
             command=select_price)  # 调用执行函数，是数值显示在 Label控件中
scale.grid(row =1)
```

## Scrollbar滚动条控件

Scrollbar 控件可以与 Listbox、Text、Canvas 以及 Entry 等控件一起使用

示例代码：

```python
# 创建一个滚动条控件，默认为垂直方向
sbar1= tk.Scrollbar(root)
# 将滚动条放置在右侧，并设置当窗口大小改变时滚动条会沿着垂直方向延展
sbar1.pack(side=RIGHT, fill=Y)
# 创建水平滚动条，默认为水平方向,当拖动窗口时会沿着X轴方向填充
sbar2 = Scrollbar (root, orient=HORIZONTAL)
sbar2.pack(side=BOTTOM, fill=X)
# 创建列表框控件,并添加两个滚动条（分别在垂直和水平方向），使用 set() 进行设置
mylist = tk.Listbox(root,xscrollcommand = sbar2.set,yscrollcommand = sbar1.set)
for i in range(30):
    mylist.insert(END,'第'+ str(i+1)+'次:'+'C语言中文网，网址为：c.biancheng.net'+ '\n' )
# 当窗口改变大小时会在X与Y方向填满窗口
mylist.pack(side=LEFT,fill = BOTH)
# 使用 command 关联控件的 yview、xview方法
sbar1.config(command =mylist.yview)
sbar2.config(command = mylist.xview)
```

## 文件选择对话框

文件对话框在 GUI 程序中经常的使用到，比如上传文档需要从本地选择一个文件，包括文件的打开和保存功能都需要一个文件对话框来实现。Tkinter 提供文件对话框被封装在`tkinter.filedailog`模块中，该模块提供了有关文件对话框的常用函数，经常使用的有以下几个：



| 方法                | 说明                                         |
| ------------------- | -------------------------------------------- |
| Open()              | 打开个某个文件                               |
| SaveAs()            | 打开一个保存文件的对话框                     |
| askopenfilename()   | 打开某个文件，并以包函文件名的路径作为返回值 |
| askopenfilenames()  | 同时打开多个文件，并以元组形式返回多个文件名 |
| askopenfile()       | 打开文件，并返回文件流对象                   |
| askopenfiles()      | 打开多个文件，并以列表形式返回多个文件流对象 |
| asksaveasfilename() | 选择以什么文件名保存文件，并返回文件名       |
| asksaveasfile()     | 选择以什么类型保存文件，并返回文件流对象     |
| askdirectory        | 选择目录，并返回目录名                       |

```py
# 定义一个处理文件的相关函数
def askfile():
    # 从本地选择一个文件，并返回文件的目录
    filename = tkinter.filedialog.askopenfilename()
    if filename != '':
         lb.config(text= filename)
    else:
         lb.config(text='您没有选择任何文件')
```

## 消息对话框

消息对话框主要起到信息提示、警告、说明、询问等作用，通常配合“事件函数”一起使用，比如执行某个操作出现了错误，然后弹出错误消息提示框。通过使用消息对话框可以提升用户的交互体验，也使得 GUI 程序更加人性化。消息对话框主要包含了以下常用方法：



| 方法                                     | 说明                         |
| ---------------------------------------- | ---------------------------- |
| askokcancel(title=None, message=None)    | 打开一个“确定／取消”的对话框 |
| askquestion(title=None, message=None)    | 打开一个“是／否”的对话框。   |
| askretrycancel(title=None, message=None) | 打开一个“重试／取消”的对话框 |
| askyesno(title=None, message=None)       | 打开一个“是／否”的对话框     |
| showerror(title=None, message=None)      | 打开一个错误提示对话框       |
| showinfo(title=None, message=None)       | 打开一个信息提示对话框       |
| showwarning(title=None, message=None)    | 打开一个警告提示对话框       |


上述方法拥有相同的选项参数，如下表所示：



| 属性    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| default | 1. 设置默认的按钮（也就是按下回车响应的那个按钮） 2. 默认是第一个按钮（像“确定”，“是”或“重试”） 3. 可以设置的值根据对话框函数的不同，可以选择 CANCEL，IGNORE，OK，NO，RETRY 或 YES |
| icon    | 1. 指定对话框显示的图标 2. 可以指定的值有：ERROR，INFO，QUESTION 或 WARNING 3. 注意：不能指定自己的图标 |
| parent  | 1. 如果不指定该选项，那么对话框默认显示在根窗口上 2. 如果想要将对话框显示在子窗口上，那么可以设置 parent= 子窗口对象 |


上述方法的返回值一般会是一个布尔值，或者是“YES”，“NO”，“OK”等，这些方法使用较为简单，此处不进行逐一列举，看个简单的示例即可：

```

import tkinter.messagebox
result=tkinter.messagebox.askokcancel ("提示"," 你确定要关闭窗口吗? ")
# 返回布尔值参数
print(result)
```

## 布局方式

Tkinter 提供了三种常用的布局管理器，分别是 pack()、grid() 以及 place()，如下表所示：

| 方法    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| pack()  | 按照控件的添加顺序其进行排列，遗憾的是此方法灵活性较差       |
| grid()  | 以行和列（网格）形式对控件进行排列，此种方法使用起来较为灵活 |
| place() | 可以指定组件大小以及摆放位置，三个方法中最为灵活的布局方法   |


上述三种方法适用于 Tkinter 中的所有控件，在讲解前面内容时，对其中一些方法已经做了相关的介绍，比如 pack() 和 grid()。在本节会对上述三个方法的应用场景做更为详细的介绍。

### pack()

pack() 是一种较为简单的布局方法，在不使用任何参数的情况下，它会将控件以添加时的先后顺序，自上而下，一行一行的进行排列，并且默认居中显示。pack() 方法的常用参数如下所示：

| 属性        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| anchor      | 组件在窗口中的对齐方式，有 9 个方位参数值，比如"n"/"w"/"s"/"e"/"ne"，以及 "center" 等（这里的 e w s n分别代表，东西南北） |
| expand      | 是否可扩展窗口，参数值为 True（扩展）或者 False（不扩展），默认为 False，若设置为 True，则控件的位置始终位于窗口的中央位置 |
| fill        | 参数值为 X/Y/BOTH/NONE，表示允许控件在水平/垂直/同时在两个方向上进行拉伸，比如当 fill = X 时，控件会占满水平方向上的所有剩余的空间。 |
| ipadx,ipady | 需要与 fill 参数值共同使用，表示组件与内容和组件边框的距离（内边距），比如文本内容和组件边框的距离，单位为像素(p)，或者厘米(c)、英寸(i) |
| padx,pady   | 用于控制组件之间的上下、左右的距离（外边距），单位为像素(p)，或者厘米(c)、英寸(i) |
| side        | 组件放置在窗口的哪个位置上，参数值 'top','bottom','left','right'。注意，单词小写时需要使用字符串格式，若为大写单词则不必使用字符串格式 |

示例代码：

```python

lb_red = Label(win,text="红色",bg="Red",fg='#ffffff',relief=GROOVE)
# 默认以top方式放置
lb_red.pack()
lb_blue = Label(win,text="蓝色",bg="blue",fg='#ffffff',relief=GROOVE)
# 沿着水平方向填充，使用 pady 控制蓝色标签与其他标签的上下距离为 5 个像素
lb_blue.pack(fill=X,pady='5px')
lb_green = Label(win,text="绿色",bg="green",fg='#ffffff',relief=RAISED)
# 将 黄色标签所在区域都填充为黄色，当使用 fill 参数时，必须设置 expand = 1，否则不能生效
lb_green.pack(side=LEFT,expand=1,fill = BOTH)
```

### grid()

grid() 函数是一种基于网格式的布局管理方法，相当于把窗口看成了一张由行和列组成的表格。当使用该 grid 函数进行布局的时，表格内的每个单元格都可以放置一个控件。，从而实现对界面的布局管理。

注意：这里的所说的“表格”是虚拟出来，目的是便于大家理解，其实窗体并不会因为使用了 gird() 函数，而增加一个表格。

grid() 函数的常用参数如下所示：



| 属性        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| column      | 控件位于表格中的第几列，窗体最左边的为起始列，默认为第 0 列  |
| columnsapn  | 控件实例所跨的列数，默认为 1 列，通过该参数可以合并一行中多个领近单元格。 |
| ipadx,ipady | 用于控制内边距，在单元格内部，左右、上下方向上填充指定大小的空间。 |
| padx,pady   | 用于控制外边距，在单元格外部，左右、上下方向上填充指定大小的空间。 |
| row         | 控件位于表格中的第几行，窗体最上面为起始行，默认为第 0 行    |
| rowspan     | 控件实例所跨的行数，默认为 1 行，通过该参数可以合并一列中多个领近单元格。 |
| sticky      | 该属性用来设置控件位于单元格那个方位上，参数值和 anchor 相同，若不设置该参数则控件在单元格内居中 |


**grid() 方法相比 pack() 方法来说要更加灵活，以网格的方式对组件进行布局管理，让整个布局显得非常简洁、优雅。如果说非要从三个布局管理器中选择一个使用的话，那么我推荐大家使用 grid() 方法。**

>   这里有一点需要大家要特别注意，在一个程序中不能同时使用 pack() 和 grid() 方法，这两个方法只能二选一，否则程序会运行错误。


下面看一组有关 grid() 函数的简单的示例：

```
#在窗口内创建按钮，以表格的形式依次排列
for i in range (10):
    for j in range (10):
        Button (win, text=" (" + str(i) + ","+ str(j)+ ")",bg='#D1EEEE') .grid(row=i,column=j)
# 在第5行第11列添加一个Label标签
Label(win,text="C语言中文网",fg='blue',font=('楷体',12,'bold')).grid(row =4,column=11)
```

### place()

与前两种布局方法相比，采用 place() 方法进行布局管理要更加精细化，通过 place() 布局管理器可以直接指定控件在窗体内的绝对位置，或者相对于其他控件定位的相对位置。

下面对 place 布局管理器的常用属性做相关介绍：



| 属性                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| anchor              | 定义控件在窗体内的方位，参数值N/NE/E/SE/S/SW/W/NW 或 CENTER，默认值是 NW |
| bordermode          | 定义控件的坐标是否要考虑边界的宽度，参数值为 OUTSIDE（排除边界） 或 INSIDE（包含边界），默认值 INSIDE。 |
| x、y                | 定义控件在根窗体中水平和垂直方向上的起始绝对位置             |
| relx、rely          | 1. 定义控件相对于根窗口（或其他控件）在水平和垂直方向上的相对位置（即位移比例），取值范围再 0.0~1.0 之间 2. 可设置 in_ 参数项，相对于某个其他控件的位置 |
| height、width       | 控件自身的高度和宽度（单位为像素）                           |
| relheight、relwidth | 控件高度和宽度相对于根窗体高度和宽度的比例，取值也在 0.0~1.0 之间 |


通过上述描述我们知道，`relx`和`rely`参数指定的是控件相对于父组件的位置，而`relwidth`和`relheight`参数则是指定控件相对于父组件的尺寸大小。

```py
# -*- coding: utf-8 -*-
from tkinter import *
#主窗口
win = Tk()
win.title("C语言中文网")
# win.iconbitmap('C:/Users/Administrator/Desktop/C语言中文网logo.ico')

#创建一个frame窗体对象，用来包裹标签
frame = Frame (win, relief=SUNKEN, borderwidth=2, width=450, height=250)
# 在水平、垂直方向上填充窗体
frame. pack (side=TOP, fill=BOTH, expand=1)

# 创建 "位置1"
Label1 = Label ( frame, text="位置1",bg='blue',fg='white')
# 使用 place,设置第一个标签位于距离窗体左上角的位置（40,40）和其大小（width，height）
# 注意这里（x,y）位置坐标指的是标签左上角的位置（以NW左上角进行绝对定位，默认为NW）
Label1.place (x=40,y=40, width=60, height=30)

# 设置标签2
Label2 = Label (frame, text="位置2",bg='purple',fg='white')
# 以右上角进行绝对值定位，anchor=NE，第二个标签的位置在距离窗体左上角的(180，80)
Label2.place(x=180,y=80, anchor=NE, width=60, height=30)

# 设置标签3
Label3 = Label (frame, text="位置3",bg='green',fg='white')
# 设置水平起始位置相对于窗体水平距离的0.6倍，垂直的绝对距离为80，大小为60，30
Label3.place(relx=0.6,y=80, width=60, height=30)

# 设置标签4
Label4 = Label (frame, text="位置4",bg='gray',fg='white')
# 设置水平起始位置相对于窗体水平距离的0.01倍，垂直的绝对距离为80，并设置高度为窗体高度比例的0.5倍，宽度为80
Label4.place(relx=0.01,y=80,relheight=0.4,width=80)
#开始事件循环
win. mainloop()
```

>   注意：在一个父组件中 place()方法可以与 grid() 方法混合使用，要与 pack() 进行区别。


参考：

http://c.biancheng.net/tkinter/

