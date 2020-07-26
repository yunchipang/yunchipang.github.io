---
layout: post
title: 如何在markdown檔案中撰寫含有liquid的代碼塊
permalink: /writing-liquid-in-code-blocks-in-markdown.html
date: 2020-07-26
author: yunchipang
tags: [jekyll, liquid]
---
我們之前已經討論過如何在你用Jekyll寫的網站裡面自己插入並且系統性地管理[標籤(tags)](https://yunchipang.github.io/adding-tags-and-related-posts.html)，基本上我們在html和css裡面寫liquid是沒有問題的，但如果你已在嘗試寫技術部落格或是個人學習筆記，你可能也會發現在markdown檔案裡的代碼塊(code block)插入liquid語法是有困難的，他可能會自己陷進循環，所以跑出來的網頁總是跟我們要的不一樣。

這篇文章要來解釋如何在使用markdown寫文章的時候在code block裡面插入liquid語法，讓他不會不受控地亂跑亂跑，靜靜地以程式碼呈現。

<br/>

## 如何使用 raw & endraw

在md輸入含liquid的code block方法很簡單，就是在code block的前後分別插入 `{ % raw % }{ % endraw % }`，程式碼就不會在code block裡自己硬跑。舉個例子，假如今天我想放在code block裡面的程式碼如下：

{% raw %}
```md
{% for tag in site.tags %}
<a href="#{{ tag[0] | slugify }}" class="post-tag">{{ tag[0] }}</a>
{% endfor %}
```
{% endraw %}

那麼其實我寫在markdown裡面的語法是這樣的：

![screenshot](/assets/images/2020-07-26-raw-endraw.png)

因為jekyll真的不讓我在`{ % raw % }{ % endraw % }`外面再寫一層`{ % raw % }{ % endraw % }`，所以只好用螢幕截圖給各位示意，別見怪。另外請注意：`{` 和 `%` 之間並沒有空白鍵，這邊文字區塊是為了讓程式碼可以正常出現而多加了個空白，寫代碼的時候請按照螢幕截圖來寫～

<br/>

