---
layout: post
title: 如何在jekyll網站手動插入google analytics追蹤碼
permalink: /adding-google-analytics-to-jekyll.html
date: 2020-07-25
author: yunchipang
tags: [jekyll, javascript]
---
在建立好自己的jekyll網站之後，對掌控網站流量有興趣的人可能會想要自己安裝google analytics追蹤碼，好時時觀察自己網站哪篇文章最受歡迎？有誰來看？主要受眾來自哪裡？這篇文章將講解如何一步一步手動在自己的jekyll網站裡面安裝GA追蹤碼。

<br/>

## Google Analytics (GA) 是什麼？
先簡單介紹一下google analytics，這是google開發並提供給大眾免費使用的一款網路流量追蹤工具，只要建立自己的帳號、拿到追蹤碼(tracking code)放到自己的網站裡，就可以從GA後台實時觀測自己的網站表現，包括使用者來源、行為、從哪個網頁進來、在每個網頁停留多久等等，對調整自己的經營策略頗有幫助。

那在著手插入google analytics設定碼之前，首先請確認你已經在自己的根目錄擁有這些從gem複製過來的資料夾，這邊存放著主題設定檔，對你至關重要。

- `_includes`
- `_layouts`
- `_sass`
- `assets`

如果尚未複製這些資料夾，請follow[這篇文章](https://yunchipang.github.io/overriding-default-theme-in-jekyll.html) step1&2的指示，完成後再回來這篇文章。那我們趕快開始安裝GA追蹤碼吧！

<br/>

## step1: 建立GA帳號及資源
要申請GA追蹤碼，首先你當然必須有一個google analytics帳號。到[Google Analytics](https://analytics.google.com)申請一個帳號，並為這個帳號申請一個資源(property)。完成基本的設定之後，GA會給你的一個此資源專屬的tracking-ID，先把這頁放著，等一下我們再回來複製。

![google analytcis tracking ID page](/assets/images/2020-07-25-ga-tracking-id.png)

<br/>

## step2: 建立 _includes/analytics.html 文件
第一步請在你網站根目錄的`_includes`資料夾裡建立一個叫做`analytics.html`的新文件，並且貼入以下程式碼。

{% raw %}
```html
<script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');
  
    ga('create', '{{ site.google_analytics }}', 'auto');
    ga('send', 'pageview');
  
</script>
```
{% endraw %}

<br/>

## step3: 在 _config.yml 貼入專屬tracking-ID
打開位於根目錄的`_config.yml`檔案，貼入剛剛我們在google analytics拿到的專屬追蹤ID。

```yml
google_analytics: UA-XXXXXXXXX-1 # your own tracking ID
```

<br/>

## step4: 編輯 _includes/head.html 檔案
最後一部請打開位於`_includes`資料夾中的`head.html`檔案，把以下程式碼貼在`</head>`之前。這組程式碼的目的是讓google analytics可以成功追蹤每一個網頁。

{% raw %}
```liquid
{% if site.google_analytics and jekyll.environment == 'production' %}
{% include analytics.html %}
{% endif %}
```
{% endraw %}

<br/>

參考資料：

- [How to add Google Analytics to a Jekyll website](https://desiredpersona.com/google-analytics-jekyll/)
- [Add Google Analytics to a Jekyll Website](https://curtisvermeeren.github.io/2016/11/18/Jekyll-Google-Analytics.html)

<br/>
