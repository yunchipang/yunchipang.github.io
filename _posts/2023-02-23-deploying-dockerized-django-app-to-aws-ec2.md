---
layout: post
title: "使用Docker部署Django REST API到AWS EC2"
permalink: /deploying-dockerized-django-app-to-aws-ec2.html
date: 2023-02-23
author: yunchipang
tags: [python, django, docker, aws]
---

最近埋頭苦寫的side project是用[Django REST Framework](https://www.django-rest-framework.org/)做出一個API，使用Docker containerize它，並利用Docker Image部署到AWS EC2上，讓使用者可以用Public IP Adress存取及使用。途中又踩了不少坑，在這篇文章記錄一下解法。github repo 傳送門：[drf-choreo-library](https://github.com/yunchipang/drf-choreo-library/)

<br/>

### 1. creare an API with Django REST Framework

本篇的主題會放在如何dockerize an app以及部署到雲端的部分，如何建立一個django app可以去看官方文檔的[quickstart](https://www.django-rest-framework.org/tutorial/quickstart/)以及[tutorial](https://www.django-rest-framework.org/tutorial/1-serialization/)，我皆從兩者受益良多，推薦推薦。可以先發想自己想做的API架構跟model，如何使用framework的部分跟著官方教學一步一步舉一反三地做就可以了。

<br/>

### 2. build a container with [Docker](https://www.docker.com/)

在部署到EC2之前，必須先建立container image，並將其push到dockerhub上面，方便之後從aws的虛擬機存取。如果尚未安裝[Docker](https://docs.docker.com/engine/install/)，可以去官方網站看一下自己的機器該如何安裝！安裝好之後可以用`docker info`或是`docker --version`檢查一下。

1. create a `Dockerfile`

    ```Dockerfile
    FROM python:3

    ENV PYTHONUNBUFFERED 1
    RUN mkdir /app
    WORKDIR /app
    COPY . /app/
    RUN pip install -r requirements.txt
    CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
    ```
    `Dockerfile`通常會長這樣，可以根據自己的需求去微調。

2. 建立docker image

    在根目錄執行以下命令以建立docker image，要注意最後還有一個`.`，代表在當前目錄建立image。
    ```bash
    $ docker build -t my-django-app .
    ```

3. 把建立好的docker image在container裡面跑起來！
   
    ```bash
    $ docker run -p 8000:8000 my-django-app
    ```
    這一步完成之後，到0.0.0.0:8000就可以看到app已經跑起來了！

4. 將當前的docker image掛上一個版本號

    ```bash
    $ docker tag my-django-app:latest my-dockerhub-username/my-django-app:1.0
    ```

5. 把docker image推到docker hub

    ```bash
    $ docker push my-dockerhub-username/my-django-app:1.0
    ```
    如果還沒有註冊或是登陸[dockerhub](https://hub.docker.com/)的可以先處理一下！已經有帳號的，在terminal裏`docker login`輸入帳號密碼就會顯示Login succeeded。

<br/>

### 3. deploy to AWS EC2 instance

這邊假設大家都會建立EC2 instance！步驟就跟平常一樣，只是在選機器的時候要記得選Amazon ECS-Optimized Amazon Linux 2。

1. 用SSH連上你的EC2 instance

    ```bash
    $ chmod 400 your-key-pair.pem
    $ ssh -i "your-key-pair.pem" ec2-user@ec2-18-144-47-2.us-west-1.compute.amazonaws.com
    ```
    將key pair下載下來放在當前根目錄，確認無法從外部存取，然後帶上key用ssh連上虛擬機器。`@`後面要放的是你的機器的Public DNS，在EC2 dashboard找到connect按鍵按下去，裡面會有說明如何用SSH Client連接instance。

2. 在EC2 instance安裝docker

    安裝並啟動docker。最後一步是把機器default的ec2-user加入docker group，這樣之後執行docker commands的時候就不需要sudo。
    ```bash
    $ sudo yum update -y
    $ sudo amazon-linux-extras install docker
    $ sudo service docker start
    $ sudo usermod -a -G docker ec2-user
    ```
    這些完成了之後`docker info`就可以看到docker相關的資訊。

3. 把docker image從dockerhub拉下來

    在ec2機器上，把之前已經推到dockerhub上面的image pull下來。如果前面有tag過版本號的記得要帶上。
    ```bash
    $ docker login
    $ docker pull my-dockerhub-username/my-django-app:1.0
    ```
    用`docker images`就可以看見當前有的images還有他們的container ID。如果有出現錯誤，可以用`docker ps`或是`docker ps -a`檢查。

4. run docker

    ```bash
    docker run -d -p 8000:8000 my-dockerhub-username/my-django-app
    ```
    沒有問題的話，跑完這個指令就可以在EC2 instance的Public DNS看到app跑起來啦！

<br/>

### 4. page redirect

前面1~3步結束，這個project基本上已經完成，重新指向只是一個nice to have。情況是：如果我不想要每次使用者都要從那一長串ec2的網址造訪我的api，那可以做一個簡單的redirect解決這個問題。首先，你需要擁有一個domain，可以是自己買的頂級網域或是像我一樣直接使用github pages的path。以下的步驟是如何在`https://xxxxx.github.io/` 的網域下新增頁面來重新指向的做法。

在github pages repository根目錄建立一個新的html檔案，檔名即是要使用的path的名稱。例如我如果想要使用者造訪`https://yunchipang.github.io/choreolibrary` 的時候被帶到ec2 instance, 我的檔名就叫做`choreolibrary.html`。貼入以下的code，可以在顯示文字的同時直接重新導向到meta裡面帶的網址，也就是要前往的ec2 instance地址（代換成自己的即可）。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Redirecting...</title>
    <meta http-equiv="refresh" content="0; url=http://ec2-54-183-149-150.us-west-1.compute.amazonaws.com:8000/">
  </head>
  <body>
    <p>Please wait while you are redirected to the EC2 instance...</p>
  </body>
</html>
```

commit之後到定義好的path，就會看到頁面顯示 Please wait while you are redirected to the EC2 instance... 的同時，網站很迅速地被轉到API頁面了。

<br/>

【感謝其他網路大神指點】
- [chatgpt](https://chat.openai.com/) == 我的神
- [Deploying Django Applications to AWS EC2 with Docker](https://stackabuse.com/deploying-django-applications-to-aws-ec2-with-docker/)

<br/>

【備註】一開始我跟的教學是用`docker-compose`去build container，但是因為部署到EC2的時候各種出錯，docker就是一直run不起來，所以後來我走標準路線改成直接用`docker` build，就沒有問題了！

【心得感想】光跟著教學影片無腦做project是學不到東西而且不會進步的，不要覺得啊我就跟著影片花一個半小時把這個project完整刻出來這就是我的東西了。沒有自己設計思考project架構、遇到報錯並解決報錯、練習舉一反三的話，我只是一台抄代碼機器罷了。不要急，要有扎扎實實砸時間下去的過程才會有深刻的學習，你寫出來的每一行程式你都要知道為什麼。通常主流的framework都會有官方文件，發現雖然慢慢讀官方文檔、嘗試理解並自己寫出來的方法比較慢，但是對於深刻理解更有幫助。跟影片做雖然好像呼啦啦repo就完整刻出來了，但是大部分時候都不知道自己在幹嘛。

<br/>