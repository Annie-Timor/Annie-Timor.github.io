---
title: Makefile笔记
author: lixinghui
date: 2023-2-9 12:00:00 +0800
categories: [Note, Makefile]
tags: [学习资料]
---

## 示例

``` makefile 
.PHONY: clean

######################################
# platform
######################################
PLATPORM = linux
# PLATPORM = windows
######################################

# binaries
######################################
CC = gcc

######################################
# target
######################################

OUT_FILE = Project

ifeq ($(PLATPORM), linux)
TARGET = $(addsuffix .out, $(OUT_FILE))
else
TARGET = $(addsuffix .exe, $(OUT_FILE))
endif
######################################

# building variables
######################################
# debug build?
DEBUG = 1
# optimization
OPT = -Og

######################################
# Build path
######################################
BUILD_DIR = build

######################################
# C sources
######################################
C_SOURCES =  \
main.c \
add/add_fun.c \
sub/sub_fun.c \
cout/cout_msg.c  

######################################
# C include
# the compiling request add -I
######################################
C_INCLUDES =  \
-I add \
-I sub \
-I cout

######################################
# C object
######################################
C_OBJS=$(addprefix $(BUILD_DIR)/, $(notdir $(C_SOURCES:.c=.o)))

vpath %.c $(dir $(C_SOURCES))

ifeq ($(DEBUG), 1)
CFLAGS += -g 
endif


$(TARGET) : $(C_OBJS)
$(CC) $^ -o $@

$(BUILD_DIR)/%.o : %.c 
$(CC) -c $(CFLAGS) -MMD $(C_INCLUDES) -o $@ $<

clean:
ifeq ($(PLATPORM), linux)
rm -rf $(C_OBJS) $(TARGET)
else
del /Q /S $(BUILD_DIR)\*.o $(TARGET)
endif

#######################################
# dependencies
#######################################
-include $(wildcard $(BUILD_DIR)/*.d)
```
## 多文件Makefile示例

Makefile在windows下运行很多奇奇怪怪的问题，难搞，linux就没有。

### step0

在windows下和在linux下的clean命令不一样，注意在windows下面是使用`/`,`$(OBJ_DIR)\*.o`而在linux下是`$(OBJ_DIR)/*.o`，否则会报错“无效开关”，windows如下所示：

```makefile
.PHONY:clean

clean:
	del /Q /S $(OBJ_DIR)\*.o $(BIN)
```



### step1

假设有main.c\add_fun.c\add_fun.h\sub_fun.c\sub_fun.h\cout_msg.c\cout_msg.h，文件都在同一个文件夹内,so easy。

```makefile
obj=main.o add_fun.o sub_fun.o cout_msg.o

a.out:$(obj)
	gcc $(obj) -o $@

%.o:%.c
	gcc -c $< -o $@

.PHONY:clean

clean:
	rm -rf $(obj) a.out
```

### step2

使用变量定义一下，其中`\`为换行符

```makefile

SRC = \
main.c \
add_fun.c \
sub_fun.c \
cout_msg.c 

INC = -I ./

OBJ=$(SRC:.c=.o)

BIN=a.out

$(BIN):$(OBJ)
	gcc $(OBJ) -o $(BIN)

%.o:%.c
	gcc -c $< -o $@

.PHONY:clean cout

clean:
	rm -rf $(OBJ) $(BIN)

cout:
	@echo "SRC = $(SRC)"
	@echo "INC = $(INC)"
	@echo "OBJ = $(OBJ)"

```

### step3

文件分文件夹进行定义，编译产生的`.o`文件放入一个文件夹内，切记添加`vpath`，因为在`$(OBJ_DIR)/%.o:%.c`这一句里面，需要`build/add_fun.c`，其中`build`为`$(OBJ_DIR)`，那么`%`的值为`add_fun`，依赖也就为`add_fun.c`，但是查找当前文件夹并没有一个叫`add_fun.c`的文件，该文件在`add/add_fun.c`，这时编译就报错`make: *** No rule to make target 'build/add_fun.o', needed by 'a.out'.  Stop.`，我们通过`vpath`来添加，使用规则为：

```
vpath PATTERN DIRECTORIES
```

为符合模式“PATTERN”的文件指定搜索目录“DIRECTORIES”。多个目录使用空格或者
冒号（：）分开。

```
vpath PATTERN
```

清除之前为符合模式“PATTERN”的文件设置的搜索路径。

```makefile
vpath
```

清除所有已被设置的文件搜索路径。

那么更新之后的Makefile文件如下所示：

```makefile

SRC = \
main.c \
add/add_fun.c \
sub/sub_fun.c \
cout/cout_msg.c 

INC = \
-I add \
-I sub \
-I cout

OBJ_DIR = build

OBJ=$(addprefix $(OBJ_DIR)/, $(notdir $(SRC:.c=.o)))
#OBJ=$(SRC:.c=.o)

BIN=a.out

# add src file to compiling dir
# no this compiling error
vpath %.c $(dir $(SRC))

$(BIN):$(OBJ)
	gcc $(OBJ) -o $(BIN)

$(OBJ_DIR)/%.o:%.c
	gcc -c $(INC) $< -o $@

.PHONY:clean cout

clean:
	rm -rf $(OBJ) $(BIN)

cout:
	@echo "SRC = $(SRC)"
	@echo "INC = $(INC)"
	@echo "OBJ = $(OBJ)"

```

### step4

调整结构，添加部分注释，添加依赖。

当我们只改变`.h`文件的时候，存在无法检测到改变，从而不编译，这是我们就需要添加依赖，让它检测到`.h`文件的变化，从而重新编译包含该文件的源文件。

> 1、添加了 -include *.d 指令; 
>
> 2.、gcc 编译指令中，添加了 -MMD 参数;

```makefile
.PHONY: clean cout

######################################
# binaries
######################################
CC = gcc

######################################
# target
######################################
TARGET = Project.out

######################################
# building variables
######################################
# debug build?
DEBUG = 1
# optimization
OPT = -Og

######################################
# Build path
######################################
BUILD_DIR = build

######################################
# C sources
######################################
C_SOURCES =  \
main.c \
add/add_fun.c \
sub/sub_fun.c \
cout/cout_msg.c  

######################################
# C include
# the compiling request add -I
######################################
C_INCLUDES =  \
-I add \
-I sub \
-I cout

######################################
# C object
######################################
C_OBJS=$(addprefix $(BUILD_DIR)/, $(notdir $(C_SOURCES:.c=.o)))

vpath %.c $(dir $(C_SOURCES))

ifeq ($(DEBUG), 1)
CFLAGS += -g 
endif


$(TARGET) : $(C_OBJS)
	$(CC) $^ -o $@

$(BUILD_DIR)/%.o : %.c 
	$(CC) -c $(CFLAGS) -MMD $(C_INCLUDES) -o $@ $<

clean:
	rm -rf $(C_OBJS) $(TARGET)

cout:
	@echo "C_OBJS = $(C_OBJS)"
	@echo "CFLAGS = $(CFLAGS)"

#######################################
# dependencies
#######################################
-include $(wildcard $(BUILD_DIR)/*.d)
```



添加dll动态链接库



## 1、简介

把一个`.c`文件编译成可执行文件，一般需要预处理、编译、汇编、链接四个步骤，每个步骤会分别调用预处理器、编译器、汇编器、链接器来完成。在linux中我们使用gcc来编译文件：

```sh
gcc helloworld.c -o a.out
gcc -o a.out helloworld.c
```

gcc 会分别调用预处理器、编译器、汇编器和链接器来自动完成程序编译的整个过程，不需要用户一个命令一个命令分别输入了。

进行预处理，使用 - E 参数来完成：

```sh
gcc -E helloworld.c
```

汇编处理，使用 - S 参数来完成：

```sh
gcc -S -o helloworld.S helloworld.c
```

进行编译，不链接，使用 - c 参数来完成：

```sh
gcc -c -o helloworld.o helloworld.c
```

只编译，不链接的时候，如果该文件依赖其他文件，在`gcc -c -o xx.o xx.c`的时候`xx.c`后面也不能跟其他c文件，这里不能多文件编译生成o文件。

## 2、规则

规则是 Makefile的基本组成单元。一个规则通常由目标、目标依赖和命令三部分构成：

```
目标：目标依赖
    命令
a.out: hello.c
    gcc -o a.out hello.c
```

一个规则中的三个部分并不是都必须要有的，但是一定要有一个目标。

一般会选择第一个作为默认目标，同时也可以有多目标，多规则目标等等。

伪目标并不是一个真正的文件，可以看做是一个标签。伪目标一般没有依赖关系，也不会生成对应的目标文件，可以无条件执行，纯粹是为了执行某一个命令，如 clean 执行清理工作。

```makefile
.PHONY: clean
...
clean:
    rm -f a.out hello.o
```

## 3、命令

命令一般由 shell命令（echo、ls）和编译器的一些工具（gcc、ld、ar、objcopy 等）组成，使用 tab 键缩进。对于规则中的每一个命令，make 会开一个进程执行，每条命令执行完，make 会监测每个命令的返回码。正确才会执行下一条命令，出错则退出编译流程。

```
Makefile:3: *** missing separator.  Stop.
```

报错这个一般是有命令不是以tab来缩进的。

在命令前加一个@可以不打印当前执行的命令。

## 4、变量

例如有一个工程为add.c、add.h、sub.c、sub.h、main.c文件构成，使用变量编写Makefile文件为：

```makefile
.PHONY:clean

CC=gcc
BIN=a.out
OBJS=main.o add.o sub.o

$(BIN):$(OBJS)
	@echo "start compiling"
	$(CC) -o $(BIN) $(OBJS)
	@echo "compiling done"

main.o:main.c add.c sub.c
	@$(CC) -c -o main.o main.c

add.o:add.c
	@$(CC) -c -o add.o add.c

sub.o:sub.c
	@$(CC) -c -o sub.o sub.c


clean:
	rm -f $(BIN) $(OBJS)
```

### 变量赋值

Makefile中的变量赋值有多种形式，比如：

*   条件赋值：`?=`
*   追加赋值：`+=`
*   立即变量：`:=`
*   延迟变量：`=`

### 自动变量

*   `$@`：目标
*   `$^`：所有目标依赖
*   `$<`：目标依赖列表中的第一个依赖
*   `$?`：所有目标依赖中被修改过的文件

*   `$%`：当规则的目标是一个静态库文件时，$% 代表静态库的一个成员名
*   `$+`：类似 $^，但是保留了依赖文件中重复出现的文件
*   `$*`：在模式匹配和静态模式规则中，代表目标模式中 % 的部分。比如 hello.c，当匹配模式为 %.c 时，$* 表示 hello
*   `$(@D)`：表示目标文件的目录部分
*   `$(@F)`：表示目标文件的文件名部分
*   `$(*D)`：在模式匹配中，表示目标模式中 % 的目录部分
*   `$(*F)`：在模式匹配中，表示目标模式中 % 的文件名部分
*   `-:`：告诉 make 在编译时忽略所有的错误
*   `@: `：告诉 make 在执行命令前不要显示命令
*   `$:`：

使用自动变量之后，Makefile为：

```makefile
.PHONY:clean

CC=gcc
BIN=a.out
OBJS=main.o add.o sub.o


$(BIN):$(OBJS)
	@echo "start compiling"
	$(CC) -o $(BIN) $(OBJS)
	@echo "compiling done"

main.o:main.c 
	@$(CC) -c -o $@ $^

add.o:add.c
	@$(CC) -c -o $@ $^

sub.o:sub.c
	@$(CC) -c -o $@ $^


clean:
	rm -f $(BIN) $(OBJS)

```

### 变量替换

使用匹配符 % 匹配变量，使用 % 保留变量值中的指定字符串，然后其他部分使用指定字符串代替。

```
.PHONY: all
SRC := main.c sub.c
OBJ := $(SRC:%.c=%.o)
all:
    @echo "SRC = $(SRC)"
    @echo "OBJ = $(OBJ)"
```

### 环境变量

除了用户自定义的一些变量，make 在解析 Makefile中还会引入一些系统环境变量，如编译参数 CFLAGS、SHELL、MAKE 等。若用户在命令行中传递跟系统环境变量同名的变量，系统环境变量也会被传递的同名变量覆盖。

```
# make HOSTNAME=zhaixue.cc
```

#### 

- 在一个Makefile文件在可以通过`make -C subdir`，去子文件夹中执行Makefile。
- 在递归执行的时候，可以通过`export`将变量从主Makefile传递到子Makefile。
- 可以通过`override`定义变量防止命令行中传递进来的值改变Makefile中的变量值。

### 条件判断

> `ifeq`：判断两个参数是够相等
>
> `ifneq`：判断参数是否不相等
>
> `ifdef`：判断一个变量是否已经定义
>
> `ifndef`：判断一个变量是否没有定义

条件判断语句由三个关键字组成：ifeq、else、endif。ifeq 后面的比较语句使用小括号抱起来，ifeq 和小括号之间要用空格隔开，小括号里的两个参数用逗号隔开。

## 5、函数

扩展通配符函数：wildcard

```
SRC  = $(wildcard *.c)
HEAD = $(wildcard *.h)
all:
    @echo "SRC = $(SRC)"
    @echo "HEAD = $(HEAD)"
```

### 用户自定义函数

对于用户自定义函数，在 Makefile 中要使用 call 函数间接调用，各个参数之间使用空格隔开：

```
PHONY: all
define func
    @echo "pram1 = $(0)"
    @echo "pram2 = $(1)"
endef
all:
    $(call func, hello zhaixue.cc)
```

### subst 函数

`subst` 函数用来实现字符串的替换，将字符串 text 中的 old 替换为 new

```
$(subst old,new,text)
```

### patsubst 函数

`patsubst `函数主要用来模式替换：使用通配符 % 代表一个单词中的若干字符，在 PATTERN 和 REPLACEMENT 如果都包含这个通配符，表示两者表示的是相同的若干个字符，并执行替换操作。

```
$(patsubst PATTERN, REPLACEMENT, TEXT)
```

### strip 函数

`strip` 函数是一个去空格函数：一个字符串通常有多个单词，单词之间使用一个或多个空格进行分割，strip 函数用来将多个连续的空字符合并成一个，并去掉字符串开头、末尾的空字符。

### findstring 函数

`findstring` 函数用来查找一个字符串，如果找到，则返回字符串 FIND，否则，返回空。使用格式如下：

```
$(findstring FIND, IN)
```

### filter 函数

`filter` 函数用来过滤掉一个指定的字符串，只留下符合 PATTERN 格式的单词，使用格式如下：

```
$(filter PATTERN…,TEXT)
```

### filter-out 函数

`filter-out` 函数是一个反过滤函数，功能和 `filter` 函数恰恰相反：该函数会过滤掉所有符合 PATTERN 模式的单词，保留所有不符合此模式的单词。

### sort 函数：单词排序

`sort` 函数对字符串 LIST 中的单词以首字母为准进行排序，并删除重复的单词。使用格式如下：

```
$(sort LIST)
```

### word 函数：取单词

`word` 函数的作用是从一个字符串 TEXT 中，按照指定的数目 N 取单词，如果 N 的值大于字符串中单词的个数，返回空；如果 N 为 0，则出错。使用格式如下：

```
 $(word N,TEXT)
```

### wordlist 函数：取字串

`wordlist` 函数用来从一个字符串 TEXT 中取出从 N 到 M 之间的一个单词串，N 和 M 都是从 1 开始的一个数字，函数的返回值是字符串 TEXT 中从 N 到 M 的一个单词串。当 N 比字符串 TEXT 中的单词个数大时，函数返回空。

```
$(wordlist N, M, TEXT)
```

### words 函数：统计单词数目

`words` 函数用来统计一个字符串 TEXT 中单词的个数，使用格式如下：

```
$(words TEXT)
```

### firstword 函数：取首个单词

`firstword` 函数用来取一个字符串中的首个单词，`firstword` 函数其实就相当于 $(word 1,TEXT)，使用格式如下：

```
$(firstword NAMES…)
$(word 1,TEXT)
```

### dir 函数：取文件路径

`dir` 函数用来从一个路径名中截取目录的部分。使用格式如下：

```
$(dir NAMES…)
```

### notdir 函数：取文件名

`notdir` 函数和 `dir` 函数实现完全相反的功能：从一个文件路径名中去文件名，而不是目录。`notdir` 函数的使用方法和 `dir` 函数相同。

```
$(notdir NAMES…)
```

### suffix 函数：取文件名后缀

`suffix` 函数从一系列文件名序列中，取出各个文件名的后缀。

```
$(suffix NAMES…)
```

### basename 函数：取文件名前缀

`basename` 函数从一系列文件名序列中，取出各个文件名的前缀部分，如果一个文件名中包括多个点号，`basename` 函数返回最后一个点号之前的文件名部分；如果一个文件名没有前缀，函数返回空字符串。

```
$(basename NAMES…)
```

### addsuffix 函数：给文件名加后缀

`addsuffix` 函数的作用是：给文件列表中的每个文件名添加后缀 SUFFIX

```
$(addsuffix SUFFIX, NAMES…)
```

### addprefix 函数：给文件名加前缀

`addprefix` 函数的作用是：给文件列表中的每个文件名添加一个前缀 PREFIX

```
$(addprefix PREFIX, NAMES…)
```

### join 函数：单词连接

`join` 函数的作用是：将字符串 LIST1 和字符串 LIST2 的各个单词依次连接，合并为新的单词构成的字符串

```
$(join LIST1,LIST2)
```

### wildcard 函数：列出所有符号匹配模式的文件

`wildcard` 函数的作用是：列出当前目录下所有符合 PATTREN 模式的文件名，其中 PATTREN 可以使用 [shell](https://www.zhaixue.cc/shell/shell-intro.html) 能识别的通配符：？、* 等。

```
$(wildcard PATTERN)
```

### foreach 函数：循环或遍历操作

`foreach` 函数的工作过程是：把 LIST 中使用空格分割的单词依次取出并赋值给变量 VAR，然后执行 TEXT 表达式。重复这个过程，直到遍历完 LIST 中的最后一个单词。函数的返回值是 TEXT 多次计算的结果。

```
$(foreach VAR,LIST,TEXT)
```

###  if 函数：上下文中实现条件判断

`if` 函数提供了在一个函数上下文中实现条件判断的功能，类似于 `ifeq` 关键字，`if` 函数的第一个参数 CONDITION 表示条件判断，展开后如果非空，则条件为真，执行 THEN-PART 部分；否则，如果有 ELSE-PART 部分，则执行 ELSE-PART 部分。`if` 函数的使用格式如下：

```
$(if CONDITION,THEN-PART)
$(if CONDITION,THEN-PART[,ELSE-PART])
```

### call 函数：用户自定义的函数

我们给函数传递的参数，在函数内部可以使用 $(0)、$(1)… 直接使用。`call` 函数是唯一一个可以用来创建新的参数化的函数。`call` 函数不仅可以用来调用一个用户自定义函数并传参，还可以向一个表达式传参：

```
$(function arguments)
${function arguments}
```

### origin 函数：变量是从哪里来的

顾名思义，`origin` 函数的作用就是告诉你，你所关注的一个变量是从哪里来的。函数的使用格式为：

```
$(origin <variable>)
```

如果变量没有定义，origin 函数的返回值为：undefined，不同的返回值代表变量的类型不同。常见的返回值如下;

*   default：变量是一个默认的定义，比如 CC 这个变量
*   file：这个变量被定义在 [Makefile](https://www.zhaixue.cc/makefile/makefile-intro.html) 中
*   command line：这个变量是被命令行定义的
*   override：这个变量是被 override 指示符重新定义过的
*   automatic：一个命令运行中的自动化变量

### shell函数：shell命令

`shell`函数的参数是 `shell`命令，它和反引号 \`\`具有相同的功能。`shell`命令的运行结果即为 `shell`函数的返回值。

```
$(shell pwd)
```

### error 函数：提示信息并终止 make 的继续执行

当遇到 error 函数时，就会给用户一个错误提示信息，并终止 make 的继续执行

```
$(error TEXT…)
```

### warning 函数：提示信息

warning 函数跟 error 函数类似，也会给用户提示信息，唯一的区别是：warning 函数不会终止 make 的运行，make 会继续运行下去。

```
$(warning TEXT…)
```



## 通配符

在 Makefile 中可以使用的通配符有：* 、? 、 […]。通配符的使用方法和含义和在 shell中一样，除此之外Makefile 还有经常使用的几个自动变量也可以看做特殊通配符：

*   $@：所有目标文件
*   $^：目标依赖的所有文件
*   $<：第一个依赖文件
*   $?：所有更新过的依赖文件

在 Makefile中，通配符主要用在两个场合：

- 用在规则的目标和依赖中

- 用在规则的命令中