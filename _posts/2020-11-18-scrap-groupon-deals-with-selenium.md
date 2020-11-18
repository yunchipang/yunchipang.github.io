---
layout: post
title: python爬groupon評價：selenium定位按鈕避開popup視窗、自動翻頁爬取同個url上的所有資料
permalink: /scrap-groupon-deals-with-selenium.html
date: 2020-11-18
author: yunchipang
tags: [python, selenium]
---

這份code是自發性作業，自從上次教授教過用`requests`跟`BeautilfulSoup`爬取跟解析groupon網站上某deal的comments之後，我就一直想試試看用selenium自動翻頁以爬取同個url上所有的comments，於是自己研究了解法，是很初階的用法但我已經感動不已。

<br>

先前情提要一下教授教的解法。因為在沒辦法控制瀏覽器自動翻頁的情況下，我們只能爬取該url頁面上直接顯示的所有comments（大約8-10筆），所以直接使用`requests.get()`加上用BeautifulSoup的`soup.find_all()`定位到comments的`class`就可以了。不過我第一次爬的時候`status_code`得到`403 forbidden`，後來在headers裡面加上`user-agent`重爬才成功。

```python
from bs4 import BeautifulSoup
import requests
```

```python
url = "https://www.groupon.com/deals/gg-fitbit-alta-fitness-wristband"
headers = {"user-agent": "XXXXX"}  # replace XXXXX with your own user-agent

r = requests.get(url, headers=headers)
```

```python
soup = BeautifulSoup(ps, "html.parser")
content = []
for i in soup.find_all("div", class_="tip-text"):
    content.append(i.text.strip())
```

檢查一下爬下來parse完的comments。

```python
content
```

```python
['Defective',
 'I’m seeing a pattern here in the reviews! I, like others, had less than a year and won’t keep the charge. I am very Disappointed in this purchase.',
 "It stopped working correctly within first few months.  First battery wouldn't last long, then die completely and have to be completely set up again, all previous tracking would be lost.  Couldn't return and can't use.",
 'I bought four of these for my family and ALL four did not work.  ALL four were separate purchases so they didn’t come in one shipment.  Mine doesn’t log all of my steps, two others doesn’t show anything one the display and on the app shows outrageous amount of steps taken so they don’t pair correctly.',
 'This doesn’t stay connected. Never will buy a used Fitbit. It sucks',
 'Love my Fitbit however bought the warranty and need to use it how do I?',
 'My 2nd one and it doesn’t work long. Battery doesn’t stay charged and won’t come on when on wrist.',
 'Very unhappy with this product. Does not perform as anticipated or as listed in the details. I purchased while home in anticipation of major surgery...thinking this could help monitor my post-surgery workout! No! only added to my frustration as I attempted to get this connected to my smart-devices. It does not provide readouts and will not stay connected to the FitBit app. Poor deal!!!']
```

<br>

但是如果我們真的想要爬取所有113頁的comments該怎麼辦呢？下定決心把一直想試試看的selenium搬出來用！import了可以放`user-agent`的`Options`、可以等按鈕出現再做事的`WebDriverWait`、拿來放條件的`expected_conditions`、還有尋找element的`By`。

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
```

```python
options = webdriver.ChromeOptions()
options.add_argument("--user-agent=XXXXX") # replace XXXXX with your own user-agent

driver = webdriver.Chrome(options=options)
driver.get(url)

time.sleep(0.5)
```

但途中遇到的一個問題是，有時候driver剛進入groupon他就會跳這個popup出來擋住我（如下圖），所以需要定位那個"no thanks"的按鈕並按掉他（我本來想用dismiss alert的方法解，但w跟我說因為這個popup本身不是用js做的而是html的div，所以必須真的去按按鈕）。解決popup之後就直接去按comment section下面的"see all reviews"。

![groupon popup screenshot](assets/images/2020-11-18-groupon-popup.png)

```python
# close popup
nothx = driver.find_element_by_id("nothx")
nothx.click()

# click see all reviews
see_all_reviews = driver.find_element_by_class_name("all-tips-link")
see_all_reviews.click()

time.sleep(0.5)
```

成功進到see all reviews的視窗之後，我們要做的重複動作是：內迴圈用driver定位評價的element並抓取該元素文本（用loop遍歷xpath） -> 將文本append到前面設定好的空list(comments) -> 內圈結束後要再將comments append到all_comments（包含全部comments的大list）-> 按 next page -> 再用外迴圈將以上動作重複113次（到113頁爬完為止），最後用`driver.close()`關閉整個瀏覽器。而因為all_comments原本會是list包list的形式（每頁的所有comments是一個list，113頁的所有comments又會是一個大list）也要記得把all_comments攤平。

```python
options = webdriver.ChromeOptions()
options.add_argument("--user-agent=XXXXX") # replace XXXXX with your own user-agent

driver = webdriver.Chrome(options=options)
driver.get(url)

time.sleep(0.5)

# close popup
nothx = driver.find_element_by_id('nothx')
nothx.click()

# see all reviews
see_all_reviews = driver.find_element_by_class_name("all-tips-link")
see_all_reviews.click()

time.sleep(0.5)

all_comments = []
for p in range1(113):
    comments = []
    for i in range(1, 11):
        try:
            e = driver.find_element_by_xpath('//*[@id="all-tips-modal"]/div[2]/div[4]/div['+str(i)+']/div[3]')
            comments.append(e.text)
        except:
            break
    all_comments.append(comments)
    
    next_page=WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, '//*[@id="all-tips-modal"]/div[2]/div[5]/div[2]')))
    next_page.click()
    time.sleep(0.5)

driver.close()

# flatten all_comments
all_comments_flat = sum(all_comments, [])
```

最後113頁裡總共獲得1127筆comment，我會嘗試在下篇文章用這些comments做groupon這兩項商品的情感分析（回答教授提供的問題集）。

<br>


📚 過程中解過的error跟遇到的困難：

1. 發現網頁load出"next page"的按鈕需要一點時間，所以後來用了`WebDriverWait`這個方法讓他等待至多10秒到next page的按鈕變成clickable。另外我本來習慣是用class_name去定位下一頁的按鈕，但偏偏這個按鈕有3個class name，一直噴錯跟我說這個element找不到，於是最後轉用xpath定位就可以了！
2. 一開始想要一口氣拿到所有的web element組成的list，再用另外一個迴圈把全部text都挑出來，但後來發現在xpath不唯一的情況下，必須在前面提到的「內迴圈」裡面就拿到text，因為每頁同個位置的comments的xpath是重複的，如果全部拿出來之後才解文本，拿到的text會變成最後一頁的所有comment重複113遍。
3. 我原本想要在driver跑完之後另寫迴圈提取web element的text，卻一直噴invalid session ID的error，後來才知道要從web element拿text的話 要在`driver.close()`之前做完！

<br>

📚 寫完這份code學會的事情：

1. 迴圈是個很棒的東西，邏輯要想清楚！
2. xpath超好用，但要找的element絕對位置跟相對位置都要檢查。


<br>

自己研究selenium真是困難又有趣，也要特別感謝主管m跟qa team的w，分別在教授上課之前教過我BeautifulSoup還有在教授不教翻頁的情況下給我提點selenium，整天回答笨問題還幫我除錯，你們真是我的知識寶庫xddd自己寫出上課沒有教的東西真的特別快樂～

<br>