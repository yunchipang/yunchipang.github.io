---
layout: post
title: 為Jekyll文章加入tags & related posts
permalink: /adding-tags-and-related-posts.html
date: 2020-07-24
author: yunchipang
tags: jekyll
---
當部落格文章跟學習筆記都愈寫愈多，我們總是會希望可以建立目錄、文章分類或是標籤來管理跟尋找文章對吧？這時候`category`跟`tags`這兩個功能就派上用場了！

簡單解釋兩者有什麼不一樣的話，`category`是有層級概念的分類，可以一層包一層。例如我可以設定一個category叫做"Artifitial Intelligence"、下一層包的是"Machine Learning"、再下一層包的是"Deep Learning"，是階梯的概念。相對來說`tags`就比較散亂，一篇文可以有非常多個`tags`他們之間沒有誰高誰低，都可以同時存在，只是標示出這篇文章有提到的內容和哪幾個標籤有關。

因為我可能會比較常寫跨領域的內容，`category`會讓我常常不知道這篇文章應該要歸於哪個分類，所以暫時只決定用`tags`讓自己方便管理。用哪一個、或是兩者都用完全取決於個人習慣跟喜好，自己開心就好！

## step1: 建立tags頁面
在根目錄建立tags.md的新文件，使用以下這段程式碼：

{% raw %}
```md
---
layout: default
title: tags
permalink: /tags/
---
<div class="tags-expo">
  <div class="tags-expo-list">
    {% for tag in site.tags %}
    <a href="#{{ tag[0] | slugify }}" class="post-tag">{{ tag[0] }}</a>
    {% endfor %}
  </div>
  <br/>
  <div class="tags-expo-section">
    {% for tag in site.tags %}
    <h2 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h2>
    <ul class="tags-expo-posts">
      {% for post in tag[1] %}
        <a href="{{ site.baseurl }}{{ post.url }}">
      <li>
        {{ post.title }}
      </li>
      </a>
      {% endfor %}
    </ul>
    {% endfor %}
  </div>
</div>
```
{% endraw %}

- `layout: default`會讓這個頁面的按鈕顯示在頂端nav bar上
- `title: tags` 即是顯示出來按鈕上的文字
- `permalink: /tags/` 就是連結後綴

至於front matter下方的html代碼，完全可以根據個人的喜好跟閱讀習慣做修改跟調整，這只是我自己喜歡的樣子：）

<br/>

## step2: 在每篇文開頭加入此篇文的tags
第二步驟是在`_layouts/post.html`選定自己要加入tags的位置貼入以下程式碼。

請仔細閱讀`post.html`裡面的程式碼，因為這關係著你每一篇文章的排版。建議同步打開你目前的網頁，觀察哪一行是哪一行，這樣可以比較精準判斷你要把tags的小連結放在這篇文章的哪裡。例如我想要把tags的連結放在我每篇文章的標題下面、時間之前，我就找到`post.html`的`<header></header>`跟`<time></time>`，並把以下程式碼塞進兩者之間：

{% raw %}
```md
<!--to add tags-->
    {% if page.tags %}
      {% for tag in page.tags %}
      <a href="{{ site.baseurl }}{{ site.tag_page }}#{{ tag | slugify }}" class="post-tag">{{ tag }}</a>
      {% endfor %}
    {%- endif -%}
<!--to add tags-->
```
{% endraw %}

前後的`<!--to add tags-->`comment是我為了易於辨識每一段代碼各自的目的而加上去的，可以自行刪除。

<br/>

## step3: 列出與此文有關的其他文章（推薦閱讀）
基於我們通常希望這項功能出現在讀者閱讀完一篇文章的最下面，好讓他可以繼續點擊閱讀他有興趣的相關文章，我們要把程式碼塞在`post.html`的`content`後面（放之前仔細讀code會比較好找位置！）

{% raw %}
```md
<!--to add related posts-->
  {% assign firstTag = page.tags | first %}
  {% assign relatedCount = 0 %}

  <div class="related-posts">
    <h3>related posts:</h3>
    {% assign firstTag = page.tags | first %}
    {% assign relatedCount = 0 %}
    {% for related in site.tags[firstTag] %}

    {% unless page.permalink == related.permalink %}
      {% assign relatedCount = relatedCount | plus: 1 %}
      <a href="{{ site.baseurl }}{{related.permalink}}">{{ related.title }}</a><br>
      {% endunless %}
        
      {% if relatedCount == 3 %}
        {% break %}
      {% endif %}
    {% endfor %}
  </div>
<!--to add related posts-->
```
{% endraw %}
    
{% raw %}`{% if relatedCount == 3 %}{% break %}{% endif %}`{% endraw %}會把推薦閱讀的數量限制在3以內，這樣就不用擔心文章太多的時候相關文章會顯示一拖拉庫東西。那當然如果你希望文末可以一口氣顯示更多推薦文章，自行增大這個數字即可。

跟完這三步驟，順利的話你就可以在每篇文章的起頭看到`tags`的連結們、文章結束處也會有根據這篇文章`tags`所產生的推薦閱讀清單了：）

<br/>

#### 參考資料
主要是參考這三篇前輩寫的內容自己拼拼湊湊、試來試去，最終搞出的解法。每個人的情況可以能不一樣，不要緊張，冷靜下來多看幾篇文章、多試幾次就成功了！Liquid語言我原本完全不認識，剛開始用前輩的幾組程式碼胡亂拼湊，結果去terminal `$ jekyll serve`的時候狂噴error，回到`_layouts/post.html`認真讀碼，其實很簡單一下就看懂了，很像html前後要關起來的概念xd

- [為 GitHub 上的 Jekyll 添加 Tags](https://nk910216.github.io/2017/08/11/UsingTagsForJekyll/)
- [使用 Jekyll + Github 來建立一個部落格（二）](https://mmiooimm.github.io/2018/09/12/2018-09-12-add-tag-to-jekyll/)
- [在jekyll blog的文章內加入新tag](https://ithelp.ithome.com.tw/articles/10210700)

<br/>
