---
layout: post
title: "[cs50x] 使用flask+postgresql建置web-app並架設在heroku"
permalink: /deploy-flask-app-with-postgresql-database-on-heroku.html
date: 2021-08-18
author: yunchipang
tags: [python, flask, postgresql, heroku]
---
cs50x final project在我苦思將近一個月、到處亂撞技術的厚牆之後，決定從簡單開始，使用教授課內教的flask搭配postgresql做為資料庫做一個web-app並架在heroku上使用。

<br>

### **idea**

在離開台北之前去做了最後一次光療，突然發現每次做完指甲，美甲師總是會問我「你生日幾月？」，問完幾月之後會問「你生日幾號？」，他需要在厚厚的數十本的顧客資料夾中翻找出我的資料，雖然用生日做排列查找已經是不錯的方法，但如果可以有前端互動跟後端資料庫應該更方便吧（我心想）所以牙一咬決定自己從頭開始刻，順便當作final project交出去一舉兩得xd

<br>

### **Flask + PostgreSQL**

- github repo: **[trexchichi-manicure](https://github.com/yunchipang/trexchichi-manicure)**

資料夾裡所有會用到的東西架構大概如下：

```
.
├── Procfile
├── app.py
├── models.py
├── manage.py
├── flask_session
├── migrations
├── manicure.db
├── requirements.txt
├── static
└── templates
    ├── index.html
    ├── layout.html
    ├── login.html
    ├── register.html
    ├── success.html
    ├── topup.html
    ├── transaction.html
    └── user.html
```

啟用database的時候記得跑完下面三條指令：

```
python3 manage.py db init
python3 manage.py db migrate
python3 manage.py db upgrade
```

<br>

### **errors**

**1. local**

```
ModuleNotFoundError: No module named 'flask._compat'
```

```
ImportError: cannot import name 'MigrateCommand' from 'flask_migrate'
```

嘗試在local run `python3 app.py`的時候被一些module error阻擋（如上），查了一下發現是版本問題，我的case把`flask`降到`1.1.4`、把`flask_migrate`降到`2.5.3`就ok了。



**2. heroku**

1. 檔案打架

推到heroku之後造訪網頁，毫無意外地收到`Application Error`，於是乖乖的用`heroku logs --tail`開始慢慢除錯。我得到的全部都是`H10 "App crashed"`的錯誤，網路上大部分的資訊說這是因為`Procfile`裡面有bug，所以我跟著把`Procfile`改來改去但還是一樣錯誤。直到最後我細細地想過一遍我資料夾裡面每個檔案的用途，發現我有`Pipfile`也有`requirements.txt`但基本上這兩個檔案的作用是一樣的，都是declare目前使用的各個套件的版本。所以最後我把`Pipfile`刪掉只留`requirements.txt`就deploy成功了。


2. 連接不到Heroku Postgres

```
sqlalchemy.exc.NoSuchModuleError: Can't load plugin: sqlalchemy.dialects:postgres
```
查了一下原因，原來這是database URI錯誤的問題。Heroku Postgres直接給的環境變數`DATABASE_URI`開頭是`postgres://`但是開頭是`postgresql://`的URI才能成功連接。於是這邊的做法會是寫一個if statement把URI的開頭換掉。

```python
basedir = os.path.abspath(os.path.dirname(__file__))
# either run on heroku env or local
DATABASE_URI = os.environ.get('DATABASE_URL') or 'sqlite:///'+ os.path.join(basedir, 'myDB.db')
if DATABASE_URI.startswith("postgres://"):
    DATABASE_URI = DATABASE_URI.replace("postgres://", "postgresql://")
app.config['SQLALCHEMY_DATABASE_URI'] = DATABASE_URI
```

<br>

📚心得：

算是在14天隔離生活中努力生出來的小專案，也因此可以把奮鬥了大半年的cs50結束了。遇到解不開的error還是會很挫折，但是慢慢可以告訴自己後退一點、休息一下，腦子清楚的時候再去好好地問google，有方法的：）

<br>

參考資源：

- [Python Flask Web API [Heroku]: It runs locally but shows Application Error when deployed](https://stackoverflow.com/questions/43593542/python-flask-web-api-heroku-it-runs-locally-but-shows-application-error-when)
- [Create a Web App and Deploy it to the Cloud in 20 minutes with Python](https://arctype.com/blog/postgres-heroku/)
- [Developping a Flask Web App with a PostreSQL Database - Making all the Possible Errors](https://blog.theodo.com/2017/03/developping-a-flask-web-app-with-a-postresql-database-making-all-the-possible-errors/)
