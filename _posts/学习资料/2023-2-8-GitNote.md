---
title: Git Note
author: lixinghui
date: 2023-2-8 12:00:00 +0800
categories: [Note, Git]
tags: [学习资料]
---



## 初始化git

### 1、设置用户名和email

```sh
git config --global user.name "your name"
git config --global user.email "xxx@qq.com"
```

### 2、初始化git

```sh
git init
```

## 添加文件和注释

### 3、添加文件

```sh
git add .
git add xx.txt
```

`add`后面跟需要添加的文件，有多个文件使用空格空开

### 4、添加注释

```sh
git commit -m "input your commit"
```

`-m`后面是本次提交的说明

## 查看状态和记录

### 5、查看仓库状态

```sh
git status
```

如果文件内容修改，使用`git status`会显示文件已修改，

### 6、查看修改后的文件和上一版本有什么区别

```sh
git diff xx.txt
```

`xx.txt`是需要查看改变的文件

**修改完成之后再次添加到仓库使用`git add`命令，添加之后必须也要添加注释`git commit -m`，对于仓库的状态可以经常性的使用`git status`命令来查看**

### 7、查看历史记录

```sh
git log
```

也可以查看简短版本的：

```sh
git log --pretty=oneline
```

### 8、查看命令记录

```sh
git reflog
```

查看命令记录可以看到每次的commit操作和代码版本的回退等等。

## 回退软件版本

### 9、回退到上个版本的代码

```sh
git reset --hard HEAD^
```

可以使用`HEAD^`回退到上个版本，也可以使用`HEAD^^`回退到前两个版本，或者使用`HEAD~10`回退到10个版本之前，通过`HEAD`回退之后，`git log`就无法查看当前版本之后的版本了，也可以使用commit的哈希值进行回退，这个可以回退到任意版本。

### 10、回退到后面版本的代码

```sh
git reset --hard 00ef7f1
```

当回退到以前的版本之后，再想回到现在的版本就无法使用`HEAD`了，可以用过`git reflog`来查看每次更新的记录，根据版本的哈希值进行回退，这里的`00ef7f1`为需要回退版本的哈希值。

## 工作区、暂存区、仓库

### 11、暂存区概念

如果本地文件修改之后，使用`git status`查看仓库的状态，会显示文件被修改，如果新增文件，则会显示文件未被跟踪，这里是把本地和master（默认仓库名称）进行比较。在把修改的文件`git add`之后，再查看状态，会显示`Changes to be committed:`已暂存可以提交的文件，这里的文件是存放在暂存区的（stage），在使用`git commit -m ""`之后，暂存区里面的文件会提交到仓库里面，这时暂存区就没有文件了。如果`git add`之后，并没有`git commit -m ""` 到仓库，这时候修改本地文件，但是没有`git add`到暂存区，这时选择`git commit -m`到仓库是不会包含第二次修改的文件的，只有暂存区的文件会被同步到仓库。

### 删除暂存区文件

```sh
git rm --cached <file>
```

当我们使用` git add `命令添加工作区的文件到暂存区时，我们想要对这个暂存区里面的文件执行删除操作时，我们则使用 `git rm --cached <file> `来修改暂存区目录（不修改工作区目录）， `git rm --cached <file> `相当于`git add`的一个逆过程。如果是文件夹记得添加`-r`

### 手动删除本地文件

当我们手动删除了本地的文件，再进行`git status`的时候会报很多红色的`delete`，这个时候他是和暂存区的文件比较，发现我们少了这些文件，如果是误删就恢复到commit的版本去就可以了，如果是需要删除的文件，就使用`git rm --cached <file>`来删除暂存区的文件就可以了。

### 12、撤销工作区（本地文件）的修改

```sh
git checkout -- readme.txt
git restore readme.txt
```

以上两个命令都可以，第二个应该是新版本的git出现的，同时兼容低版本的命令。当在本地修改文件之后，发现需要撤销本次修改，可以使用`git check -- `或者`git restore`来撤销本地的操作，一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；另一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

### 13、撤销暂存区的修改

```sh
git reset HEAD readme.txt
git restore --staged readme.txt
```

以上两个命令都可以，把暂存区内的文件回退到工作区，也可以认为是删除暂存区的文件，如果`git add`之后继续对本地文件进行了修改，再使用`git restore --staged readme.txt`回退版本，会把暂存区的文件删除，本地文件不变。

### 14、删除文件

```sh
git rm zz.txt
git commit -m "delete zz.txt"
```

当需要从仓库内删除某个文件的时候，先在本地删除，这时本地和仓库的文件就不一样了，如果需要删除仓库内的文件，使用`git rm zz.txt`，再`commit`，如果需要恢复误删的文件，可以从仓库内同步到工作区可以使用`git checkout -- zz.txt`或者`git restore zz.txt`.

## 远程仓库

### 15、添加远程仓库

```sh
git remote add other git@github.com:Annie-Timor/git_learn.git
git remote add origin git@github.com:Annie-Timor/git_learn.git
```

`other`或者`origin`这个名称可以自己更改的

### 16、删除远程连接仓库

```sh
git remote remove origin
git remote remove other
```

### 17、查看远程库信息

```sh
git remote -v
```

### 18、推送到远程仓库

```sh
git push -u origin master
```

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

仓库内容不为空，推送为：

```sh
git push origin master
```

### 19、克隆远程仓库

```sh
git clone git@github.com:Annie-Timor/git_learn.git
```

在这里远程地址可以使用git，也可以使用https，如`https://github.com/Annie-Timor/git_learn.git`，但是相对于https,使用git的速度会更快一些。

### 20、拉取远程仓库

```sh
git pull <远程主机名> <远程分支名>:<本地分支名>
git pull origin master:dev
```

`git pull`用于从远程获取代码并合并到本地，如果是和当前分支合并`:`后的分支名称可以不要.

## 分支管理

### 21、创建分支

```sh
git branch dev
git checkout -b dev
git switch -c dev
```

创建dev分支，后面两个是创建分支并进入

### 22、切换分支

```sh
git checkout dev
git switch dev
```

切换到`dev`分支上

### 23、合并分支

```sh
git merge dev
git merge --no-ff -m "merge with no-ff" dev
```

合并`dev`分支到当前分支来，使用参数`--no-ff`开启普通合并模式，普通合并模式会产生commit记录（默认合并是快速模式，不会有合并记录）

### 24、删除分支

```sh
git branch -d dev
```

删除`dev`分支

### 25、查看分支信息

```sh
git branch
```

当前所在的分支前面会有个*号

### 26、查看分支合并图

```sh
git log --graph
```

### 27、保存工作现场

```sh
git stash
```

把工作区保存到当前分支

### 28、恢复工作现场

```sh
git stash apply
git stash drop
or
git stash pop
```

### 29、查看`stash`列表

```sh
git stash list
```

### 30、复制特定提交到当前分支

```sh
git cherry-pick 4c805e2
```

后面这个`4c805e2`是提交的哈希值，每次`commit`都会有这个。

### 31、强制删除未合并的分支

```sh
git branch -D feature
```

`feature`为需要删除的分支名称，参数一定为`-D`,`-d`无法删除未合并的分支。

## 标签管理

### 32、创建标签

```sh
git tag <name>
git tag v1.0
git tag v0.9 f52c633
git tag -a v0.1 -m "version 0.1 released" 1094adb
```

创建标签默认是在当前最新提交的`commit`上的，也可以指定一个`commit`的哈希值来打标签。还可以创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字

### 33、查看所有标签

```sh
git tag
```

- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

## 文件重命名

### 34、重命名文件

```sh
$ git mv -v oldfile newfile
# $ git mv -f oldfile newfile
# 已经追踪，无需进行 add -u
$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    oldfile -> newfile

# 重命名之后正常 commit push 就可以了
$ git commit -m "rename oldfile to newfile"
$ git push
```

> `-v`：显示信息，`-f`：强制移动，`-k`：跳过移动出错的文件
{: .prompt-tip }

### 35、重命名文件夹

```sh
$ git mv -v oldfolder newfolder
# $ git mv -v oldfolder/ newfolder/

# 已经追踪，无需进行 add -u
$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    oldfolder/... -> newfolder/...

# 重命名之后正常 commit push 就可以了
$ git commit -m "rename oldfolder to newfolder"
$ git push

```

`/` 不会产生影响。**若 newfolder 文件夹原本已经存在，则会将 oldfolder 移入 newfolder。**

## 大文件报错

当文件中有超过git上传上限的文件，正常操作首先`git add`，然后`git commit`，在这两步都不会报错，在`git push`的时候，就会报文件过大的错误，在这个时候，删除本地文件，再进行`git commit`，再`push`依然报错，产生这个问题的原因为`git push`不仅仅`push`当前版本，还要`push`所有本地的版本，解决办法为：

1、查看历史版本

```sh
git log
```

2、回退到之前的版本

```sh
git reset --hard 00ef7f1
```

`00ef7f1`为需要回退到那个版本的哈希值

3、确保删除大文件之后再`add`

```sh
git add 
```

4、`git commit -m "xxx"`，`git push origin master`

## Gitee操作

### 更改仓库信息

仓库的所有信息都可以在仓库的设置中修改，进入网页版的gitee，选择要修改信息的仓库，点击管理，按照自己需要修改就好了。修改仓库名称或者链接地址之后本地连接这个仓库的信息也需要修改一下，如下：

```sh
git remote -v
git remote remove origin
git remote add origin <新的远程仓库地址>
```

首先查看本地链接的远程仓库有哪一些，删除原来链接的仓库，再重新进行链接。

### 删除仓库

删除仓库也在仓库的设置里面，有一个删除仓库的选项。

### 删除文件

删除已经提交到远程仓库内的文件，这里只能通过命令操作，如下：

```sh
git pull origin master
ls
git rm -r --cached .\element
git commit -m "删除element文件夹"
git push origin master
```

其中`git rm`命令的格式如下：

```sh
# 删除文件
git rm <file>
# 强制删除，已放入暂存区的文件
git rm -f <file>
# 从暂存区删除，但保留到本地
git rm --cached <file>
# 删除文件夹，递归删除
git rm -r ./element
```

### 删除标签

```sh
# 删除本地标签
git tag -d <name>
# 删除远程标签
git push --delete origin <name>
# 删除与分支同名的标签
git push origin :refs/tags/<name>
```



## .gitignore详解

### git忽略优先级（由高到低）

>从命令行中读取可用的忽略规则
>
>当前目录定义的规则
>
>父级目录定义的规则，依次递推
>
>$GIT_DIR/info/exclude 文件中定义的规则
>
>core.excludesfile中定义的全局规则

### git忽略规则匹配语法

> 空格不匹配任意文件，可作为分隔符，可用反斜杠转义
> 开头的文件标识注释，可以使用反斜杠进行转义
> ! 开头的模式标识否定，该文件将会再次被包含，如果排除了该文件的父级目录，则使用 ! 也不会再次被包含。可以使用反斜杠进行转义
> / 结束的模式只匹配文件夹以及在该文件夹路径下的内容，但是不匹配该文件
> / 开始的模式匹配项目跟目录
> 如果一个模式不包含斜杠，则它匹配相对于当前 .gitignore 文件路径的内容，如果该模式不在 .gitignore 文件中，则相对于项目根目录
> ** 匹配多级目录，可在开始，中间，结束
> ? 通用匹配单个字符
> \* 通用匹配零个或多个字符
> [] 通用匹配单个字符列表

### 常用匹配示例

> bin/: 忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件
> /bin: 忽略根目录下的bin文件
> /\*.c: 忽略 cat.c，不忽略 build/cat.c
> debug/*.obj: 忽略 debug/io.obj，不忽略 debug/common/io.obj 和 tools/debug/io.obj
> \*\*/foo: 忽略/foo, a/foo, a/b/foo等
> a/**/b: 忽略a/b, a/x/b, a/x/y/b等
> !/bin/run.sh: 不忽略 bin 目录下的 run.sh 文件
> *.log: 忽略所有 .log 文件
> config.php: 忽略当前路径的 config.php 文件

### .gitignore规则不生效

.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。

解决方法就是先把本地缓存删除（改变成未track状态），然后再提交:

```sh
git rm -r --cached .
git add .
git commit -m "updata .gitignore"
```

你想添加一个文件到Git，但发现添加不了，原因是这个文件被.gitignore忽略了：

```sh
$ git add App.class
The following paths are ignored by one of your .gitignore files:
App.class
Use -f if you really want to add them.
```

如果你确实想添加该文件，可以用-f强制添加到Git：

```sh
git add -f App.class
```

或者你发现，可能是.gitignore写得有问题，需要找出来到底哪个规则写错了，可以用git check-ignore命令检查：

```sh
$ git check-ignore -v App.class
.gitignore:3:*.class    App.class
```

.gitignore文件参考格式

```shell
*.o
*.d
*.exe

```

