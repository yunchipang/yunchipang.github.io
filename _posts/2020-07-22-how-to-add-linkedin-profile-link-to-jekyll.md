---
layout: post
title: 如何在Jekyll網頁底下加入Linkedin個人主頁連結
permalink: /how-to-add-linkedin-profile-link-to-jekyll.html
date: 2020-07-22
author: yunchipang
tags: jekyll github-pages linkedin
---
大家剛使用Jekyll建立網頁之後，第一件事就是設定`_config.yml`檔案了對吧？（詳請請見：[如何使用Jekyll建立個人GitHub Pages網頁](https://yunchipang.github.io/setting-up-a-ghpages-site-with-jekyll.html)），在`username.github.io`打開自己的網頁之後，捲捲捲到最下面，可以看到當初在`_config.yml`設定的`title`, `email`, `description`等等資訊都被完好地呈現出來了。

但是可以連到自己其他社群媒體或網頁的內建連結只有`twitter`和`github`，如果我想要連到Linkedin怎麼辦？跟著這篇文章簡單修改`_config.yml`就可以輕鬆連結啦！

<br/>

## step1: 找到自己的linkedin username
到自己的Linkedin主頁，可以看到紅色框框裡的網址，把`username`複製下來。

![linked profile screenshot](assets/images/2020-07-22-linkedin-profile.png)

以我的主頁為例，我的username就是除了`https://www.linkedin.com/in/`以外剩下的部分，也就是`yunchipang`。但如果你的`username`不是很好看（Linkedin幫你自動產生，可能還帶有一串讀不懂的id亂碼），建議可以先到右上角藍色框框的"Edit public profile & URL"去修改一下自己的`username`，最好改得跟自己名字一樣，整齊美觀也方便別人閱讀。

<br/>

## step2: 把username貼到_config.yml檔案裡
在資料夾根目錄打開你的`_config.yml`檔案，找到social links的部分，建立`linkedin_username`，然後把你剛剛複製好的`username`貼上去。

	twitter_username: yunchipang
	github_username:  yunchipang
	linkedin_username: yunchipang  # add your linkedin

<br/>

## step3: 存檔並在local server預覽
將你剛剛編輯好的`_config.yml`存好，並使用本地瀏覽器預覽成果。

	$ jekyll serve

成功的話，你就可以在每一頁的footer link那邊看到剛剛多加入的Linkedin標誌和你自己的主頁連結了，點點看有沒有順利連到Linkedin！

![footer links screenshot](/assets/images/2020-07-22-footer-links.png)

<br/>

## step4: git指令
如果在本地瀏覽器成功預覽到的話，最後一步一樣就是推上github啦！記得好習慣先`$ git status`檢查一下有沒有成功改到`_config.yml`檔案。

	$ git status
	$ git add _config.yml
	$ git commit "add linkedin profile to footer"
	$ git push

