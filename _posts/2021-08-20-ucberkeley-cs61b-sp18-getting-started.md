---
layout: post
title: "[cs61b] 建立作業github repo並clone官方starter code＋註冊gradescope"
permalink: /ucberkeley-cs61b-sp18-getting-started.html
date: 2021-08-20
author: yunchipang
tags: [cs61b, java, git]
---

開始認真寫uc berkeley的cs61b之前必須先建置好java環境以及交作業的github repo，因為踩了一些小坑也受到網路上其他資源的幫助，決定把自己的路程也打包記錄下來。看了網路上很多心得跟筆記之後決定跟spring18的版本，因為autograder對公眾開放，就算不是正式enroll的ucb學生也可以使用系統交作業並得到分數。以下大概講解建立作業repo的步驟以及註冊gradescope流程。

- course website: [https://sp18.datastructur.es/](https://sp18.datastructur.es/)
- gitbook: [https://joshhug.gitbooks.io/hug61b/content/](https://joshhug.gitbooks.io/hug61b/content/)

<br>

### **1. create a github repo**
**step1. 在github建立新的repo並clone到local**

第一步先去github建立一個新的repository。假設我建立好github上面的作業repo名稱為`cs61b-sp18`，回到本地我想要建立cs61b作業資料夾的位置，把空的repo clone下來。command line可能會給 warning: You appear to have cloned an empty repository 但不用理他，資料夾有成功建立就好。

```shell
$ git clone https://<token>@github.com/yunchipang/cs61b-sp18.git
```

<br>

**step2. remote add官方[skeleton repo](https://github.com/Berkeley-CS61B/skeleton-sp18)**

先cd進我的本地作業資料夾，再remote add官方skeleton repo（內含所有homework & projects的starter code）

```shell
$ cd cs61b-sp18
$ git remote add skeleton http://<token>@github.com/Berkely-CS61B/skeleton-sp18.git
```

這時候使用`git remote`應該可以見到自己的repo跟skeleton

```shell
$ git remote
origin
skeleton
```

<br>

**step3. git pull**

把skeleton裡面的strater code全部pull到本地資料夾，就可以看到所有內容。

```shell
$ git pull skeleton master
```

```shell
$ ls
ec1		hw4		lab1		lab13		lab2setup	library-sp18	proj1gold
hw1		hw5		lab10		lab14		lab3		proj0		proj2
hw2		hw6		lab11		lab15		lab4		proj1a		proj3
hw3		hw7		lab12		lab2		lab9		proj1b
```



<br>

### **2. 註冊 [gradescope](https://www.gradescope.com/)**
到[gradescope](https://www.gradescope.com/)網站註冊一個新帳號，course ID填入`MNXYKX`，通過email驗證並得到課程access之後會看到以下畫面。交作業的時候再authorize gradescope存取你的github作業repo就可以了！

![cs61b-sp18-gradescope](/assets/images/cs61b-sp18-gradescope.png)

<br>

參考資料：
- [CS61B学习记录：利用Github管理课程代码](https://zhuanlan.zhihu.com/p/52776972)
- [CS61B autograder排坑篇](https://zhuanlan.zhihu.com/p/115229260)