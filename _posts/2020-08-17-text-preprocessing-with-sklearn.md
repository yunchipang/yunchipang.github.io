---
layout: post
title: 如何使用scikit-learn做NLP資料清理及文字預處理
permalink: /text-preprocessing-with-sklearn.html
date: 2020-08-17
author: yunchipang
tags: [python, sklearn, NLP]
---
這個project本身是在參加[shopee code league](https://careers.shopee.sg/codeleague/)時其中的一個挑戰，歸類為Sentitment Analysis（情感分析），材料是蝦皮網站上各種消費者留下的review，我們的任務是做資料清理、vectorization、並建模預測每條review會給出的rating(1~5)。不過因為我還是自學新手，對模型的掌握還不是很好，就不寫出來殘害世人。這篇文章只會介紹資料清理跟文字預處理的部分，code如果有寫錯或是可以改進的地方也歡迎跟我說（眼冒愛心）

<br/>

## import libraries
```python
import nltk
import pandas as pd
import re
import string

stopwords = nltk.corpus.stopwords.words('english')
ps = nltk.PorterStemmer()
```
* `stopwords`: 指定為`nltk`包裡面預設好的英文stopwords（當然如果這個project是指在處理其他語言，可以更改`'english'`為其他語言，目前支援的語言可以詢問google大神）
*   `ps`: 指定`nltk`包裡面的`PorterStemmer`作為文字預處理的詞幹提取工具

<br/>

## import data
```python
train = pd.read_csv('train.csv')
test = pd.read_csv('test.csv')
```

<br/>

## data preprocessing
```python
# calculate the percentage of punctuation
def count_punct(text):
    count = sum([1 for char in text if char in string.punctuation])
    return round(count/(len(text) - text.count(" ")), 3)*100
    
# tokenization, stemming, and removing stopwords all-in-one function
def clean_text(text):
    text = "".join([word.lower() for word in text if word not in string.punctuation])
    tokens = re.split('\W+', text)
    text = [ps.stem(word) for word in tokens if word not in stopwords]
    return text
```
這個步驟共完成兩項主要的文字預處理。首先，因為我想要提取一個review裡的punctuation比例作為模型的其中一項feature，這邊需要define一個function叫做`count_punct()`，用於計算單個review裡面標點符號的比例。再來，`clean_text()`這個步驟就完成了其他主要的文字預處理，包括tokenization, stemming（視個人需求也可以代換成lemmatizing）以及去除stopwords。

```python
train['review_len'] = train['review'].apply(lambda x: len(x) - x.count(" "))
train['punct%'] = train['review'].apply(lambda x: count_punct(x))

test['review_len'] = test['review'].apply(lambda x: len(x) - x.count(" "))
test['punct%'] = test['review'].apply(lambda x: count_punct(x))
```
define完function之後當然就是apply到各個dataframe裡面的review column啦！我在這個步驟還有多設定一個feature叫做`review_len`，就是計算單個review裡面有多少字，也可能是一個強大的預測factor。而這裡`.apply()`推薦使用lambda，寫起來簡單、方便、快速、好理解（好處真多xd）

<br/>

## apply CountVectorizer
```python
from sklearn.feature_extraction.text import CountVectorizer

count_vect = CountVectorizer(analyzer=clean_text)
X_counts = count_vect.fit_transform(train['review'])
X_counts_test = count_vect.fit_transform(test['review'])
```
預處理的最後一步當然就是對我們想處理的review文字做vectorization啦！我在這個project裡使用的是`CountVectorizer`，相對於`TfidfVectorizer`跑起來負擔小一點。這個步驟需要跑比較久，大家要對電腦有耐心，不要隨意拍打餵食或是refresh，小心功虧一簣。完成後可以print出來檢查一下。

```python
print(X_counts)
```
可能的output會長這樣：

```
<30000x21677 sparse matrix of type '<class 'numpy.int64'>'
	with 272574 stored elements in Compressed Sparse Row format>
```

因為vectorize過後的`X_counts`是一個"sparse matrix"（一個大部分value都為零的超大矩陣），所以我們在合併dataframe之前要把它用pandas轉為`DataFrame`，再將轉為`DataFrame`的`X_counts`和前面處理過的`review_len`及`punct%`兩個feature合併。

```python
X_features = pd.concat([train[['review_len', 'punct%']].reset_index(drop=True), pd.DataFrame(X_counts.toarray())], axis=1)
X_features_test = pd.concat([test[['review_len', 'punct%']].reset_index(drop=True), pd.DataFrame(X_counts_test.toarray())], axis=1)
```
最後`X_features`跟`X_features_test`就是拿去建模型的材料（分別為train跟test）。比賽中我最後用的模型是`RandomForestClassifier`，在[kaggle](https://www.kaggle.com/yunchipang/competitions)拿到前25%的成績。雖然在很多人眼裡可能不怎樣，但這是第一個我獨自清理+寫出來的NLP模型，內心感動不已啊（掩面）之後如果變強了會再回來試試看修改進步的！