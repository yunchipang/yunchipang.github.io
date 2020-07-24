---
layout: post
title: 如何在jekyll預設主題下自己修改 html & css 樣式
permalink: /overriding-default-theme-in-jekyll.html
date: 2020-07-24
author: yunchipang
tags: [jekyll, html, css]
---
用jekyll和github-pages建好屬於自己的部落格之後，你是不是跟我一樣常常看著網頁覺得某個地方可以改得再漂亮一點？但是用的是jekyll預設的主題，其實整體來說蠻不錯的，只想改點小地方，該怎麼辦呢？這篇文章會簡單解釋gem-based theme的概念，然後教大家怎麼覆寫預設css。

<br/>

## 什麼是 Gem-based theme？
你如果按照官網方法安裝jekyll並建立一個新的jekyll site的話，預設的主題叫做`minima`，並且在你的網頁資料夾根目錄，可以看到`Gemfile`和`Gemfile.lock`這兩個檔案，簡單來說這兩個檔案是負責追蹤Minima這個主題的更新，因為版面配置的檔案們不是直接存在你的根目錄、而是存在`gem`裡面，所以當官方增加了這個主題的某些功能，你可以直接使用而不需手刻或手動更新。

## 什麼是「覆寫」(override)?

>”Jekyll will look first to your site’s content before looking to the theme’s defaults for any requested file in the following folders”

那這些檔案不存在我們自己的根目錄，如果想手動更改某些設定怎麼辦？這些預設檔案本身無法被更改（其實可以，但到網頁的時候跑不出來），所以如果我們想要自己更改一些設定跟版面配置，可以在根目錄複製一組minima主題的設定檔，並在副本更改，override（覆寫）預設值。

大概理解覆寫概念之後，我們就趕快來動手寫寫看吧！

<br/>

## step1: 找到你的主題設定檔(gem)
要覆寫(override) theme本身的default設定，我們需要做的第一件事是找到你現在所用gem-based主題的gem。該如何找到預設檔案呢？在terminal打入以下指令：

```zsh
$ bundle info --path <theme name>
```

例如jekyll預設是的主題是minima，就鍵入

```zsh
$ bundle info --path minima
```

terminal1便會給你theme的路徑，例如

```zsh
/usr/local/lib/ruby/gems/2.6.0/gems/minima-2.5.1
```

<br/>

## step2: 複製到根目錄
找到路徑之後，可以用finder進去一個一個點開看。在minima-2.5.1這個資料夾裡面會有四個預設資料夾：

- `_includes`
- `_layouts`
- `_sass`
- `assets`

無論我們有沒有全部都要自己改，先四個資料夾都先copy到自己的根目錄就對了！因為他們是互相連動的，如果少複製幾個東西，到時候雖然該改的都改了，可能在你的網頁上還是跑不出來。

<br/>

## step3: 編輯複製過來的檔案
把四個資料夾都複製到自己的根目錄之後，就可以瞄準想改的東西開始改了。先給一個觀念，在_layouts這個資料夾裡，寫的是每種layout型式的html檔案，也就是你的文件會在瀏覽器上呈現的樣子。預設有四：

- `default.html`
- `home.html`
- `page.html`
- `post.html`

拿我們要覆寫e的`post.html`來解說最簡單，他就是我們寫的每一篇文章會在網頁上呈現的格式。header tag裡面的是h1標題，再來是時間(time tag)、再來是文章作者(author)、最後是主要內容(content)等等。如果我們想要在其中的某個位置安插一個新的東西，那就要修改文件去覆蓋這個預設的post.html檔案了。

那我要示範改什麼呢？我覺得minima主題本身預設的code block背景色跟邊框實在太醜了，想自己改成淺灰色背景跟無邊框的效果。下圖左為更改前、圖右為更改後。

![before editing](/assets/images/2020-07-24-code-block-1.png){:height="50%" width="50%"}![after editing](/assets/images/2020-07-24-code-block-3.png){:height="50%" width="50%"}

想達成這個效果，有兩個檔案要動：
- `_sass/minima/_base.scss`
- `_sass/minima/_syntax-highlighting.scss`。

<br/>

#### step(1) 編輯 _base.scss
打開`_base.scss`，找到 `/** code formatting **/` 下面的 `pre, code { }` （可以使用ctrl+f 的find功能會比較快），可以看到default內容長這樣：

```scss
pre,
code {
  @include relative-font-size(0.9375);
  border: 1px solid $grey-color-light;
  border-radius: 3px;
  background-color: #eef;
}
```

預設的background-color是#eef，去網路上一查發現這就是我討厭的淺紫色，於是選定我想要的淺灰色(html whitesmoke #f5f5f5) 修改。再來因為我也不喜歡程式碼區塊有邊框，所以我也把`border: 1px solid $grey-color-light;`這行comment掉了。

備註：因為我害怕把預設內容弄不見，所以改code的時候習慣是把原本的comment掉（在css用`/** **/`把code包起來）而不是直接刪除，這樣之後如果想恢復原狀會比較方便，但這邊為了方便閱讀就把不需要的代碼先刪掉。所以最後我的code長這樣：

```scss
pre,
code {
  @include relative-font-size(0.9375);
  border-radius: 3px;
  background-color: #f5f5f5;
}
```

這裡改的是程式碼區塊文字的背景色，如果只有`_base.scss`改好就會長這樣，看得出來文字背景色還沒有變：

![code block 2](/assets/images/2020-07-24-code-block-2.png)

<br/>

#### step(2) 編輯 _syntax-highlighting.scss
打開`_syntax-highlighting.scss`，找到 `/** syntax highlighting styles **/`，這是在設定整個程式碼區塊（方形）的顏色。預設代碼如下：

```scss
.highlight {
  background: #fff;
  @extend %vertical-rhythm;

  .highlighter-rouge & {
    background: #eef;
  }
```

改成這樣：

```scss
.highlight {
  background: #fff;
  @extend %vertical-rhythm;

  .highlighter-rouge & {
    /**background: #eef;**/
    background: #f5f5f5;
  }
```

兩個檔案都改好之後，預覽應該就會長這樣，大功告成啦！

![code block 3](/assets/images/2020-07-24-code-block-3.png)

<br/>

筆記：其實這個問題我在網路找半天都沒找到解決方案，但靈機一動，把所有scss檔案都打開(也只有四個xd）然後用find功能找到所有跟backgorund-color有關的指令，試來試去就成功啦！如果遇到什麼困難，jekyll官方說明一定要認真讀，非常有用。

#### 參考資料
- [https://jekyllrb.com/docs/themes/](https://jekyllrb.com/docs/themes/)

<br/>