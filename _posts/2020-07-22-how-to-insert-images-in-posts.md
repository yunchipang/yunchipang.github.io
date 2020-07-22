---
layout: post
title: 如何在使用Jekyll寫網頁的時候插入圖片
permalink: /how-to-insert-images-in-posts.html
date: 2020-07-22
author: yunchipang
tags: jekyll github-pages
---
終於在GitHub-Pages架好自己的Jekyll網站了嗎？下一步就是努力產製文章囉！但如果我今天想要在自己的學習筆記或是某教學文章中插入漂亮圖片或是螢幕截圖來幫助讀者理解該怎麼辦呢？

## Jekyll資料夾架構

首先要了解Jekyll網頁資料夾的架構。我們用前一篇教學文章[如何使用Jekyll建立個人GitHub Pages網頁](https://yunchipang.github.io/setting-up-a-ghpages-site-with-jekyll.html)的步驟在本機架設好一個專門寫這個網站的資料夾之後，假設我的根目錄就取名叫做`yunchipang.igthub.io`，在`yunchipang.igthub.io`裡面會自動出現`_posts`和`_site`兩個資料夾（還有其他文件這裡先不多提）

![finder screenshot](/assets/images/2020-07-22-finder-screenshot.png)

- `_posts` 是存放「你寫出來的文章」的地方，尚未被轉換成html檔案
- `_site` 是最終產生網頁的地方，也就是最後你在瀏覽器會看見的東西

所以如果我們想要在文章裡插入圖片，我們必須把圖片存進正確的位置、告訴資料夾這個圖片的路徑、還要寫出正確的MarkDown語法讓電腦讀懂。

<br/>

## step1: 建立assets資料夾
首先在根目錄手動建立一個新的資料夾，可以叫做`assets`，之後如果有任何非文字的檔案或文件需要插入在文章裡，都可以存放在這個資料夾。而為了管理方便，我還會在`assets`裡面多新增ㄧ個subfolder叫做`images`，專門存放圖片。如果使用terminal請follow以下指令，當然你也可以使用finder直接新增資料夾（比較直覺）。

	$ cd yunchipang.github.io # cd 到根目錄
	$ mkdir assets
	$ cd assets
	$ mkdir images
	
如果你還想插入別的非文字檔（例如pdf），請依此類推建立其他subfolder，之後同一種類的檔案都收在同個資料夾比較好管理。

<br/>

## step2: 插入圖片路徑
打開你正在編輯的文章md檔，準備插入先前存放在`assets/images/`資料夾裡面的圖片路徑。雖然在MarkDown語法裡面插入圖片，最基本標準的寫法如下，

	![my screenshot](/screenshot.jpg)
	
但因為我們今天是要讓電腦知道我們想插入的圖片放在這個目錄的哪裡，所以必須明確指出位置。

	![my screenshot](/assets/images/screenshot.jpg)

路徑照著你自己建立的資料夾寫就對了，但請記得`assest`資料夾一定要建立在「根目錄」不然電腦找不到。我前幾次傻傻的把圖片放到內建的`_site`資料夾裡面的`assets`，結果圖片根本跑不出來，因為檔案放錯地方！要記得`_site`是最終網頁被瀏覽器吃到的樣子，原始的檔案不可以放在那裡。

<br/>

## step3: 存檔並且用local server檢查
第三步驟就和平常建立文章一樣啦！按存檔之後在terminal下`$ jekyll serve`的指令，就可以在本機瀏覽器看看你的漂亮圖片有沒有跑出來啦！（溫馨提醒：`$ jekyll serve`要回到根目錄才可以下）

	$ jekyll serve

<br/>

### 參考資料
- [https://jekyllrb.com/docs/posts/](https://jekyllrb.com/docs/posts/)
- [I cannot get an image to display](https://talk.jekyllrb.com/t/i-cannot-get-an-image-to-display/850/2)
