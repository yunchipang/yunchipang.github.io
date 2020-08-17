---
layout: post
title: 顧客會不會打開email？行銷資料合併、清理、分析與建模
permalink: /marketing-analytics-email-open-rate.html
date: 2020-08-17
author: yunchipang
tags: [marketing, python, sklearn, data-analytics]
---
這個project是來自[shopee code league](https://careers.shopee.sg/codeleague/)這個比賽的最終[challenge](https://www.kaggle.com/c/student-shopee-code-league-marketing-analytics)：被歸類於marketing analytics的binary classification。題目是利用官方提供的三個數據集去合併、清理及建立模型，根據有用的線索去預測顧客會不會打開蝦皮寄出的行銷email。

我覺得行銷分析最有趣但也最麻煩的地方在於，分析師必須先瞭解在這個情況下，有哪些因素可能會影響消費者做決定，這跟其他技術更硬、更偏向工程師寫的模型很不一樣，行銷分析的程式碼都不會很難，但分析師必須了解很多消費者行為及商業知識，並根據情況選取或製造feature。

<br/>

# import libraries
```python
import pandas as pd
import numpy as np
import datetime as dt
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
```
* 因為dataset裡面有不好分析的時間，要利用`datetime`包把它解開，或製造新的feature（例如星期幾）
* `matplotlib`和`seaborn`都是前期拿來觀察data分佈和表現的visualization工具，用不到就可以不用import

<br/>

# import data
```python
train = pd.read_csv('train.csv')
test = pd.read_csv('test.csv')
users = pd.read_csv('users.csv')
```
`users.csv`是蝦皮方除了train和test之外給的獨立data，裡面主要是一些使用者資料，包含3個意義不明的attribute（比賽方沒有給解釋，只能硬用xd）、使用者年齡、以及使用者的email帳號後綴（gmail, yahoo, hotmail等等）。我在資料預處理的階段就會把users裡面用的到的資料提取並合併到train跟test。

<br/>

# data cleaning
### 1) merge with user info
```python
train = pd.merge(train, users, on='user_id', how='left')
test = pd.merge(test, users, on='user_id', how='left')
```
資料預處理的第一步就是把`users`dataset裡面的東西base on `user_id`合併到train跟test set裡。

### 2) get_dummies
```python
# remove time from grass_date
train['grass_date'] = [str.replace(" 00:00:00+08:00", "") for str in train['grass_date']]
test['grass_date'] = [str.replace(" 00:00:00+08:00", "") for str in test['grass_date']]

# convert grass_date from string to datetime
train['grass_date'] = train['grass_date'].apply(lambda x: dt.datetime.strptime(x, '%Y-%m-%d'))
test['grass_date'] = test['grass_date'].apply(lambda x: dt.datetime.strptime(x, '%Y-%m-%d'))

# create a new column 'weekday'
train['grass_weekday'] = train['grass_date'].apply(lambda x: dt.datetime.strftime(x, '%w'))
test['grass_weekday'] = test['grass_date'].apply(lambda x: dt.datetime.strftime(x, '%w'))

train['grass_weekday'] = train['grass_weekday'].astype('category')
test['grass_weekday'] = test['grass_weekday'].astype('category')

# grass_weekday
weekday_dum_train = pd.get_dummies(train['grass_weekday'], prefix='weekday_')
weekday_dum_test = pd.get_dummies(test['grass_weekday'], prefix='weekday_')
```
再來是把categorical variables轉換成dummies。首先處理`grass_date`，這項應該有更好的轉換方法，但我在時間壓力之下先用了比較笨但比較直接的`.replace()`把`grass_date`的除了年日月的部分（時、分、秒等等）去除了、用`dt.datetime.strptime()`把原本是string的時間改成datetime形式、再用`dt.datetime.strftime()`從時間多增加一項稱作`grass_weekday`的feature，即可知道郵件寄出的日子是星期幾。最後把`grass_weekday`利用`pd.get_dummies()`轉換為dummy variables。

```python
# country_code
country_dum_train = pd.get_dummies(train['country_code'], prefix='country_')
country_dum_test = pd.get_dummies(test['country_code'], prefix='country_')

# attr_3
attr_3_dum_train = pd.get_dummies(train['attr_3'], prefix='attr_3_')
attr_3_dum_test = pd.get_dummies(test['attr_3'], prefix='attr_3_')

# domain
domain_dum_train = pd.get_dummies(train['domain'], prefix='is_')
domain_dum_test = pd.get_dummies(test['domain'], prefix='is_')
```
接著處理剩下比較簡單的`country_code`, `attr_3`和`domain`。就依序把三個feature利用`pd.get_dummies()` 轉換成dummy variables就可以了，記得train&test都要處理。

```python
# concat them into train & test
train = pd.concat([train, country_dum_train, weekday_dum_train, attr_3_dum_train, domain_dum_train], axis=1)
test = pd.concat([test, country_dum_test, weekday_dum_test, attr_3_dum_test, domain_dum_test], axis=1)
```
最後記得把剛剛用`get_dummies()`製造出來的dataframes都橫的concat到train&test。

### 3) fill in NaNs
```python
# replace Never open/login/checkout with max value
train['last_open_day'] = train['last_open_day'].replace('Never open', 900)
train['last_login_day'] = train['last_login_day'].replace('Never login', 1000)
train['last_checkout_day'] = train['last_checkout_day'].replace('Never checkout', 1000)

test['last_open_day'] = test['last_open_day'].replace('Never open', 900)
test['last_login_day'] = test['last_login_day'].replace('Never login', 1000)
test['last_checkout_day'] = test['last_checkout_day'].replace('Never checkout', 1000)
```
第三步是填補缺失值。首先在 `last_open_day`, `last_chekcout_day`, `last_login_day`三項中有出現'never open', 'never checkout'及'never login'的值，這些雖然不是缺失值，但他們的存在也會為我們建模帶來不便，所以這個步驟中我選擇用單項feature中最大的值（或再大一點）來填充這些'never'。

```python
# attr_1 & attr_2
train['attr_1'] = train['attr_1'].fillna(0)
train['attr_2'] = train['attr_2'].fillna(0)

test['attr_1'] = test['attr_1'].fillna(0)
test['attr_2'] = test['attr_2'].fillna(0)
```
因為`attr_1`及`attr_2`是boolean，觀察了一下整個dataset發現空白的值其實就是0（所有存在的值都是1），所以我將兩個attribute的NaN都填入0。

```python
# age
train['age'] = train['age'].fillna(train['age'].median())
test['age'] = test['age'].fillna(test['age'].median())
```
觀察了一下使用者年齡分佈，我選擇用median去填充（當然你可以選擇其他方法，可以試試看哪個做出來的模型比較高分）。

### 4) convert dtypes
```python
# country_code
train['country_code'] = train['country_code'].astype('category')
test['country_code'] = test['country_code'].astype('category')
```
資料清理的最後一步即是轉換dtypes。為了建模的乾淨方便，辨識清楚每項variable的性質、並將其轉換為最適合的dtype。

```python
# last open/login/checkout day
train['last_open_day'] = train['last_open_day'].astype('int64')
train['last_login_day'] = train['last_login_day'].astype('int64')
train['last_checkout_day'] = train['last_checkout_day'].astype('int64')

test['last_open_day'] = test['last_open_day'].astype('int64')
test['last_login_day'] = test['last_login_day'].astype('int64')
test['last_checkout_day'] = test['last_checkout_day'].astype('int64')
```

```python
# attr_1 & attr_2
train['attr_1'] = train['attr_1'].astype('int64')
test['attr_1'] = test['attr_1'].astype('int64')

train['attr_2'] = train['attr_2'].astype('int64')
test['attr_2'] = test['attr_2'].astype('int64')
```
```python
# age
train['age'] = train['age'].astype('int64')
test['age'] = test['age'].astype('int64')
```

<br/>

# data preparation
```python
train.columns
```
開始建立模型前，我們的好習慣是要先確定要拿來使用的features有哪些，將它們挑出來並製造一個獨立的、嶄新的dataframe，這樣到時候回來看code也會比較乾淨清楚。這裡使用`train.columns`先大概看一下我們手上所有的features有哪些。

```python
# prepare X and y
X_features = ['subject_line_length', 'last_open_day', 'last_login_day', 'last_checkout_day',
              'open_count_last_10_days', 'open_count_last_30_days', 'open_count_last_60_days',
              'login_count_last_10_days', 'login_count_last_30_days', 'login_count_last_60_days',
              'checkout_count_last_10_days', 'checkout_count_last_30_days', 'checkout_count_last_60_days',
              'attr_1', 'attr_2', 'age', 'attr_3__0.0', 'attr_3__1.0','attr_3__2.0', 'attr_3__3.0', 'attr_3__4.0',
              'country__1', 'country__2', 'country__3', 'country__4', 'country__5','country__6', 'country__7', 
              'weekday__0', 'weekday__1', 'weekday__2', 'weekday__3', 'weekday__4', 'weekday__5', 'weekday__6',
              'is__@163.com', 'is__@gmail.com', 'is__@hotmail.com', 'is__@icloud.com','is__@live.com', 
              'is__@outlook.com', 'is__@qq.com','is__@rocketmail.com', 'is__@yahoo.com', 'is__@ymail.com', 'is__other']

X_train = train[X_features]
y_train = train['open_flag']

X_test = test[X_features]
```
這個步驟中我選出我要使用的features名單，將他們建立成為`X_features`這個list，這樣便可以直接使用這個variable指定變數，建造`X_train`和`X_test`這兩個dataframe，也不要忘記把y項挑出來。

<br/>

# model building
```python
from sklearn.model_selection import KFold, cross_val_score
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(n_estimators=200, n_jobs=-1, random_state=42)
rf_model = rf.fit(X_train, y_train)
```
終於來到最後一步的建模，比賽中我在多方比較之下選用的明顯比較高分的隨機森林classifier。

```python
k_fold = KFold(n_splits=5)
rf_cv_scores = cross_val_score(rf, X_train, y_train, cv=k_fold, scoring='accuracy', n_jobs=-1)
print('rf cross validation scores:', rf_cv_scores)
rf_cv_scores_avg = np.mean(rf_cv_scores)
print('rf cross validation scores mean:', rf_cv_scores_avg)
```

使用cross-validation看分數，而最後模型的結果如下：
* rf cross validation scores: [0.86701115 0.87476203 0.89706282 0.89461518 0.89406405]
* rf cross validation scores mean: 0.8855030468323515

```python
y_pred_rf = rf_model.predict(X_test)
rf_df = pd.DataFrame({'row_id': test['row_id'], 'open_flag': y_pred_rf})
rf_df.to_csv('randomforest.csv', index=0)
```
最後記得要使用訓練好的模型預測test set，並把結果作成指定模式的dataframe、輸出成csv並且上傳到kaggle比賽頁面才算完成。

<br/>

後記：

為期兩個多月的shopee code league終於結束了，雖然過程中一直有孤軍奮戰的感覺，但感謝我的隊友們還是時不時會理我一下（咦）。感謝自己雖然比較晚起步、也不是工科出身，但一直很喜歡玩資料跟寫模型，雖然很難很辛苦很累但也一直莫名地很努力、很不願放棄。雖然比賽中神經網路的部分我真的做不太到，但跟商業比較有關的資料分析跟自然語言處理其實我都打從心底蠻喜歡的！希望之後還有機會找到神隊友一起玩資料打比賽<3