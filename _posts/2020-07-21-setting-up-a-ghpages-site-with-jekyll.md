---
layout: post
title: 如何使用Jekyll建立個人GitHub Pages網頁
permalink: /setting-up-a-ghpages-site-with-jekyll.html
date: 2020-07-21
author: yunchipang
tags: jekyll github github-pages
---

前情提要：為了架設個人網站在GitHub Pages，我大概看遍所有網路上可找到的資料、並且前前後後刪除了約莫10次我的`username.github.io` repo，因為一旦follow某一個網路上的教學，做一做就會感覺出錯（或是做不下去），但我又不會修改或是把東西救回來，只好一直打掉重練。身為這樣一個尚在努力學習的程式新手，希望把我最後一次終於成功用Jekyll架出GitHub Pages的經驗寫出來，讓其他也還在程式之路上掙扎的新手們可以有個參考。

<br/>

### 什麼是Jekyll?
[Jekyll](https://jekyllrb.com/)是基於Ruby語法寫成的一款「靜態網頁產生器」(Static Site Generator)，相似用途的架站工具還有[Hugo](https://gohugo.io/), [Hexo](https://hexo.io/), [Pelican](https://blog.getpelican.com/)等等。

### 為何選擇Jekyll?
因為python是我主要使用的語言，當初原本想要選擇[Pelican](https://blog.getpelican.com/)做為架github pages的靜態網頁產生器，但因為網路上資訊不夠多，而且途中頻頻碰壁（怎麼樣就是架不起來），我決定改為使用網路教學內容較豐富、而且github pages支援的Jekyll。那廢話不多說我們趕快開始架站吧！

<br/>

## step1: 於GitHub建立新的repo
到自己的GitHub頁面，按下右上角的"+"符號，按下"New Repository"建立一個新的repo。因為我們這個repo是要拿來管理GitHub Pages的，所以名稱一定要取為`username.github.io`(username就是自己GitHub的使用者名稱，請自行代換）。

![screenshot](/assets/images/2020-07-21-create-a-new-repo.png)

<br/>

## step2: 安裝GitHub Pages Gem
GitHub Pages Gem是用來安裝在本機中用來啟用Jekyll的套件。打開terminal，用以下指令安裝，安裝完畢可以看一下版本訊息。

	$ gem install github-pages
	$ github-pages --version

如果沒有辦法安裝的話，可能是因為尚未安裝Ruby（前面有提到Jekyll是用Ruby跑的）所以請先安裝Ruby，如果你是Mac系統可以透過homebrew安裝。

    $ brew install ruby
    $ ruby --version

<br/>

## step3: 在本地建立一個新的Jekyll專案
轉換工作目錄到你要存放此專案的資料夾，建立一個新的jekyll site。這個資料夾可以自由取名，例如`blog`, `myblog`, `myawesomeblog`等等都可以，我為了方便管理乾脆取跟github repo一樣的名字`username.github.io`（跟前面說的一樣，username請代換成自己的）建立好工作目錄後，轉換位置到此工作目錄裡面。

	$ jekyll new username.github.io
	$ cd username.github.io


<br/>

## step4: config.yml設定
新的Jekyll專案建立之後，第一個要設定的檔案就是`config.yml`。`config.yml`這個檔案有點像是此jekyll專案的基礎設定，網站的`title`（網站標題）, `description`（網站描述）, `baseurl`（網站主網址：如果網站要建在`https://username.github.io/blog`的話，這裡要打`/blog`）,`url`（網站網址：即是`https://username.github.io`）還有連接到其他社群（例如twitter, github等等）的連結都要在這裡設定。

我自己設定的範例如下。

	title: Yunchi Pang
	email: yunchipang@gmail.com
	description: final year student at CUHK | business analytics & quantitative marketing
	baseurl: ""
	url: "https://yunchipang.github.io"
	twitter_username: yunchipang
	github_username:  yunchipang

其實也不知道是不是真的這樣用，讓我再玩一下jekyll如果有新發現再回來更新！

<br/>

## step5: 在本地預覽網站
`config.yml`存好檔之後，在本地預覽網站，看是不是自己想要（或想像中）的樣子。

	$ jekyll serve

上面的指令跑完之後，用瀏覽器打開[localhost:4000](https://localhost:4000)就可以看到你的網站現在的外觀了。做到這邊我們只剩最後一步，推到github上啦！

<br/>

## step6: 推上GitHub
先確認你現在的工作目錄是你之前創好的`username.github.io`資料夾。寫下面指令的時候`username` 一樣都要記得帶換成你自己設定的。

	$ git init
	$ git add .
	$ git commit -m "initial commit"
	$ git remote add origin https://github.com/username/username.github.io
	$ git push origin master

這時候你再到自己的GitHub看一下`username.github.io`的repo有沒有順利更新（東西有沒有順利從本地推上去），如果有出現跟你本地資料夾裡面一樣的東西的話，恭喜你push成功了！接著就可以到自己的網址`https://username.github.io`檢查漂亮網站有沒有出現囉～

希望上面的資料對正在自己架網站的你有幫助，但我畢竟不是個工程師，如果此篇文章內容有誤，煩請務必告知在下，必定立刻向您請教並且修改（害別人的專案爛掉我承受不起啊）萬分感謝！

<br/>

#### 我的參考資料（跪謝前輩大老們幫助）
-  [Setting up a GitHub Pages site with Jekyll](https://docs.github.com/en/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll) （官方guide非常有參考價值但某些路直直走下去還是死胡同，請多方參考解決方案）
- [使用 Jekyll 建立自己的 Github Page Blog](https://nk910216.github.io/2017/02/05/HowToSetupBlog/)
- [GitHub Pages + Jekyll 快速把 Markdown 文檔變成部落格](https://fokayx.com/2015/05/14/GitHubPages-Jekyll.html)

