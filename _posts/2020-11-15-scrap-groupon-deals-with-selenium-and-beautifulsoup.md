---
layout: post
title: 使用selenium爬取groupon評價：定位按鈕避開popup視窗、自動翻頁爬取同個url上的所有資料
permalink: /scrap-groupon-deals-with-selenium-and-beautifulsoup.html
date: 2020-11-15
author: yunchipang
> tags: [python, BeautifulSoup, selenium]
---

這份code是自發性作業，自從上次教授教過用request跟BeautilfulSoup爬取跟解析groupon網站上某deal的comments之後，我就一直想試試看用selenium自動翻頁以爬取同個url上所有的comments（教授說這堂課不教翻頁，因為"it's beyond our current training"... 但我想要更多training啊那我自己寫好了xd）於是自己研究了解法，是很初階的用法但我已經感動不已。

先前情提要一下教授教的解法。因為在沒辦法控制瀏覽器自動翻頁的情況下，我們必須

```
from bs4 import BeautifulSoup
import requests
import nltk
import string
import pandas as pd
```

最後也要特別感謝我主管m跟qa team的w，分別在教授上課之前教過我BeautifulSoup還有在教授不教翻頁的情況下給我提點selenium，你們是我的知識寶庫：）