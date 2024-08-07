---
title: 如何添加图片
author: lixinghui
date: 2023-8-1 12:00:00 +0800
categories: [Blogging, Demo]
tags: [example]
---

如何添加一张图片：

参考：

```
https://yunchipang.github.io/how-to-insert-images-in-posts.html
```

- `_posts` 是存放「你寫出來的文章」的地方，尚未被轉換成html檔案
- `_site` 是最終產生網頁的地方，也就是最後你在瀏覽器會看見的東西
- blog的根文件夹为blog所在的文件夹

## step1: 建立assets資料夾

首先在根目錄手動建立一個新的資料夾，可以叫做`assets`，之後如果有任何非文字的檔案或文件需要插入在文章裡，都可以存放在這個資料夾。而為了管理方便，我還會在`assets`裡面多新增ㄧ個subfolder叫做`images`，專門存放圖片。如果使用terminal請follow以下指令，當然你也可以使用finder直接新增資料夾（比較直覺）。

## step2: 插入圖片路徑

打開你正在編輯的文章md檔，準備插入先前存放在`assets/images/`資料夾裡面的圖片路徑。雖然在MarkDown語法裡面插入圖片，最基本標準的寫法如下，

```
![my screenshot](/screenshot.jpg)
```

但因為我們今天是要讓電腦知道我們想插入的圖片放在這個目錄的哪裡，所以必須明確指出位置。

```
![my screenshot](/assets/images/screenshot.jpg)
```

路徑照著你自己建立的資料夾寫就對了，但請記得`assest`資料夾一定要建立在「根目錄」不然電腦找不到。我前幾次傻傻的把圖片放到內建的`_site`資料夾裡面的`assets`，結果圖片根本跑不出來，因為檔案放錯地方！要記得`_site`是最終網頁被瀏覽器吃到的樣子，原始的檔案不可以放在那裡。

## step3: 編輯圖片大小及位置

如果想要簡單編輯自己插入的圖片大小及位置，也是可以直接在markdown檔案裡調整的。如果想要把圖片調整到預設的一半大小，直接在剛剛寫的檔案路徑後面指定：

```
![my screenshot](/assets/images/screenshot.jpg){:height="50%" width="50%"}
```

如此一來圖片就會順利顯示為一開始的一半大小，當然這條指令的圖片長寬都是可以自己調整的，存檔後就可以打開`$ jekyll serve`的指令在本地瀏覽器預覽囉！覺得調整的滿意就跟之前一樣下`$ git`系列指令推上遠端就大功告成了。


> 这样在本地markdown中无法查看该图片.
{: .prompt-tip }


> `![图片名](图片地址)`
> 
> 例如`![A](/assets/img/example/pic2.png)`

![丢掉幻想](/assets/img/example/pic2.png)

