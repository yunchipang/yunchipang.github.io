---
layout: post
title: 使用nltk對groupon餐廳評價做文字預處理&情感分析
permalink: /sentiment-analysis-with-groupon-deals.html
date: 2020-11-15
author: yunchipang
tags: [python, NLP]
---
這份code其實是我這學期 MKTG4080 - Digital Marketing 課程裡內教的內容，因為覺得很好玩所以稍微優化了一下放上來做個紀錄。教授提供了一份`groupon.xlsx`的檔案，裡面包涵四家餐廳在groupon網站上得到的comments，我們的任務是利用nltk包做sentences跟words的tokenize並用教授提供的定義算出這幾家餐廳各自的satisfactory ratio。

<br>

# import packages & read data
這次用來做語言處理的還是我們的老朋友nltk，string拿來remove stopwords，pandas拿來讀取教授事先提供的excel檔案。

```
import nltk
import string
import pandas as pd

df = pd.read_excel('groupon.xlsx', sheet_name='A)
```

# define function
```
def clean_text(x):
    sentences = nltk.sent_tokenize(x)
    words = [nltk.tokenize.word_tokenize(sentence.lower()) for sentence in sentences]
    words = sum(words, [])
    words = [word for word in words if word not in string.punctuation]
    words = [word for word in words if word not in nltk.corpus.stopwords.words('english')]
    lemmatizer = nltk.stem.wordnet.WordNetLemmatizer()
    l_words = [lemmatizer.lemmatize(i) for i in words]
    return l_words
```

# clean text
```
df['clean_text'] = df['comment'].apply(clean_text)
```

<br>

# sentiment analysis
教授也事先提供了兩包詞組，分別是positive跟negative，目標是利用手上清理好的text去跟裡面的字作配對，進一步算出這些comments是正面還是負面的評論。因為這兩包詞組是用換行分開的，讀進來之後要`strip('\n')`把換行符號去掉。

```
f1 = open('positive.txt','r').readlines()
positive = [i.strip('\n') for i in f1]
f2 = open('negative.txt','r').readlines()
negative = [i.strip('\n') for i in f2]
```

定義sentiment這個function，如果該詞存在於positive詞包裡面，就在他p的分數上+1，反之如果該詞存在於negative的詞包裡面，在他的n分數上+1，最後比較p跟n誰比較大，如果平手就return中性。

```
def f_senti(x):
    p = 0
    n = 0
    for i in x:
        if i in positive:
            p += 1
        elif i in negative:
            n += 1
    if p > n:
        return 'positive'
    elif p == n:
        return 'neutral'
    else:
        return 'negative'
```

製造`p_or_n` 這個column，顯示該comment屬於正面還是負面評論。

```
df['p_or_n'] = df['clean_text'].apply(f_senti)
```

把df按照不同deal ID拆成四個df（先用`df['Deal ID'].unique()`就可以知道有幾家餐廳、他們的ID分別是多少）

```
df1 = df[df['Deal_ID'] == 4828]
df2 = df[df['Deal_ID'] == 15174]
df3 = df[df['Deal_ID'] == 15178]
df4 = df[df['Deal_ID'] == 19691]
```

最後利用positive的數量除以總數define一個計算satisfactory ratio的function，大功告成！

```
def satisfaction(x):
    return len(x[x['p_or_n'] == 'positive'])/len(x)
    
print(satisfaction(df1), satisfaction(df2), satisfaction(df3), satisfaction(df4))
# 0.9 0.9 0.95 0.95
```



