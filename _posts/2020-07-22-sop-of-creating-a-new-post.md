---
layout: post
title: 在Jekyll創建的GitHub Pages寫一篇新文章的標準步驟
permalink: /sop-of-creating-a-new-post.html
date: 2020-07-22
author: yunchipang
tags: jekyll
---
為了防止自己老是忘記學會的事情，決定寫一篇如何製造新文章的步驟SOP，以後寫文章就靠這篇舊文章了，可絕對不要忘記步驟跟語法啊拜託！

## step1: markdown檔案建立
cd到存放此網站文件的工作目錄，在`_posts`的指定文件夾建立markdown檔案，命名為`YYYY-MM-DD-title-of-the-post.md`

```zsh
$ cd yunchipang.github.io/_posts
$ touch YYYY-MM-DD-title-of-the-post.md
```

用喜歡的文字編輯器打開這個markdown檔案，編輯最開頭的front matter部分。

```md
---
layout: post
title: 自訂的文章標題
permalink: /title-of-the-post.html
date: YYYY-MM-DD
author: yunchipang
---
```

接下就是自由地寫文章啦！多練習markdown語法就會愈寫愈熟練的。可以連續寫個好幾天，確定完成之後就存檔。

<br/>

## step2: 在本機檢查文章內容及排版
雖然我們在編輯文章的時候可以用文字編輯器一邊寫一邊預覽（我是用MacDown），但是最後文章呈現在網頁上還是會有差異，因此存完md檔之後建議先在本機用local server看一下預期畫面喔！使用以下指令，follow terminal給的server address在瀏覽器裡檢查，看完用`ctrl+c`離開就可以了。

```zsh
$ jekyll serve
```

另外，為了避免之後看到錯誤還要回本機改、改完還要重新`add`, `commit`, `push`一次（有夠麻煩），建議大家寫完一篇文章就要慢慢讀一遍認真檢查有沒有錯字（我超常打錯字...）

<br/>

## step3: git指令 (add, commit, push)
這個步驟是讓本機的git知道你已經新增了這篇文章，好習慣是下任何一個指令前都先用`$ git status`檢查一下現在的狀態。

```zsh
$ git status
$ git add _posts/YYYY-MM-DD-title-of-the-post.md
$ git commit -m "add YYYY-MM-DD-title-of-the-post.md"
```
	
最後一個步驟就是將你新增的文章推到remote的github repo啦！

```zsh
$ git push
```
	
如果有要推到指定的分支請使用：

```zsh
$ git push <remote name> <branch name>
```

如果出現local和remote的檔案不一致（可能有手癢在remote偷改東西）請先pull下來再push回去。

```zsh
$ git pull
$ git push
```

雖然這篇文章主要是寫給自己看的（我是個金魚腦）但如果讀者也受用我會很開心的！每個人在往github推東西的時候都會遇到各式各樣的困難，所以今天份要提醒自己的事是：程式寫不出來不要對自己生氣，所有事情都是熟能生巧，可以多查資料、多看stackoverflow，google是你最好的朋友：）

<br/>
