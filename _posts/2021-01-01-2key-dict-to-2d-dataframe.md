---
layout: post
title: 將雙鍵單值的python dictionary轉為二維dataframe
permalink: /2key-dict-to-2d-dataframe.html
date: 2021-01-01
author: yunchipang
tags: python
---
這次的任務是接到BD部門的需求，外部廠商想要知道2020年跟敝公司合作的的成果如何，所以需要我們去系統撈資料出來並作金額加總。剛接到任務的時候我心想：「把整年賺的錢全部加起來還不簡單？」，但是因為需要做雙層分組（分月份＆分產品類別），我著實掙扎了一下子才解出來。最終解法是用python含兩個key的dictionary轉二維dataframe寫出來的，在這裡做個紀錄！

<br>

### 1. data cleaning

```python
import pandas as pd
import datetime

df = pd.read_csv('data.csv')
```
首先當然是import需要的packages，`pandas`是必備就不多說，`datetime`則是因為需要用月份分組，必須從「時間」資料裡面取「月份」出來使用而額外import的。


```python
df.dropna(axis=0, how='all', inplace=True)
df = df.loc[(df['status']!='rejected')&
            (df['twd_conversion_sale_amount']!=0)&
            (df['twd_conversion_payout']!=0)]
```
然後也依據任務需求當下做了些小清理。首先因為資料從系統下載下來，最末都會有一個全空的row很討厭，就用`dropna()`把不需要的資料刪除；再來因為`'status'`如果是`'rejected'`、`'twd_conversion_sale_amount'`和`'twd_conversion_payout'`為0的話都不需列入計算，於是也用`df.loc()` filter掉。

```python
df['date_time_dt'] = pd.to_datetime(df['date_time'])
df['month'] = df['date_time_dt'].dt.month
```
因為從csv讀進來的`'date_time'`欄位資料型態是object，於是多建一個column叫做`'date_time_dt'`並把格式轉為datetime，再用`dt.month`把月份資訊單獨取出來並建為`'month'`這個欄位稍後分組時使用。

<br>

### 2. get results

```python
gb = df.groupby(['month', 'payout_group'])

for k, data in gb:
    print(k)
```
因為需要按照月分跟產品類別分組，`df.groupby()`就直接帶入`'month'`跟`'payout_group'`兩個欄位。然後試著把每組的key先print出來看看。假設此份檔案只含1-3月份資料、並且分為`groupA`, `groupB`及`groupC`的話，出來的樣子將會如下：

```python
(1, 'groupA')
(1, 'groupB')
(1, 'groupC')
(2, 'groupA')
(2, 'groupB')
(2, 'groupC')
(3, 'groupA')
(3, 'groupB')
(3, 'groupC')
```

接下來就可以開始計算每組加總的金額。開始迴圈執行前，先定義`d`為一個空字典，再將兩個key用tuple的方式在for loop裡面指定，然後將每次迴圈得到的結果update到`d`這個字典。

```python
d = {}
for (k1, k2), data in gb:
    d.update({(k1, k2): data['twd_conversion_sale_amount'].sum()})
```

```python
d
```
將產出的d印出來看的話會類似像下面這樣，100, 200, 300那些數值即是每組`'twd_conversion_sale_amount'`的加總。

```python
{(1, 'groupA'): 100, 
 (1, 'groupB'): 200, 
 (1, 'groupC'): 300, 
 (2, 'groupA'): 400, 
 (2, 'groupB'): 500, 
 (2, 'groupC'): 600, 
 (3, 'groupA'): 700, 
 (3, 'groupB'): 800, 
 (3, 'groupC'): 900}
```
重頭戲、也是這次學到最神奇的指令來了，因為產生的dictionary是雙鍵單值 `{(key1, key2): value}` 的型態，但我希望最終可以用二維dataframe的方式呈現，所以需要轉換。

```python
gmvsdf = pd.DataFrame(gmvs.values(), index=pd.MultiIndex.from_tuples(gmvs.keys())).unstack(1)
```

最終呈現的表格如下圖。如此便可以成功地把最清楚完整的表格交給提出需求的同事，也可以節省自己非常多時間！（光想像用excel fitler到天荒地老就發抖...）

![final dataframe](/assets/images/2020-01-01-final-dataframe.png)

<br>

📚心得：

我當初最開始學python的時候不知為何沒有學到dictionary（都在笨笨用list亂拼），直到後來開始實習、必須天天跟資料相處才意識到字典的好用xd 我現在對字典的評價就是：不知道怎麼解的時候就用dict，總之就是哪裡都能放、什麼型態都能轉換的神奇東西！

<br>

參考資料：

[Dictionary with 2 keys and 1 value to pandas Dataframe](https://stackoverflow.com/questions/27218233/dictionary-with-2-keys-and-1-value-to-pandas-dataframe)

<br>