---
layout: post
title: WordPress網站換主機：使用All-in-One WP Migration外掛打包搬家到AWS Lightsail
permalink: /how-to-migrate-a-wordpress-site.html
date: 2021-07-01
author: yunchipang
tags: [wordpress, aws lightsail]
---

今年我的旅遊部落格 **[triztravel.com](https://triztravel.com)** 快滿兩歲了，當初大二剛開始碰HTML, CSS跟SEO，對虛擬主機、網域這些專有名詞都不太熟悉的時候，在網路上做了多方比較，租了Siteground的虛擬主機來用，但是...

1. 去年突然收到他們要結束服務的email通知，不知道前景如何
2. 除了第一年有優惠，之後續約的價錢都落在180 USD/year 左右，真的太貴...
3. 我的網站流量不大，主要是自己寫好玩的＆做個旅遊記錄而已

因此今年趁著被自動續約之前趕快把網站包一包搬家啦！這次選用 **[AWS Lightsail](https://aws.amazon.com/lightsail/)** 主要有幾個原因：

1. 便宜！最低階的方案一個月只要3.5USD，覺得對我來說已經很夠用
2. 一直想碰碰看AWS，單純覺得是個練技術的好機會

花了一點時間踩了一些坑之後，網站總算順利打包並搬家成功了！以下就稍微過一遍我把WordPress網站從Siteground搬到AWS Lightsail的步驟，也包括DNS的重新設定。

<br>

### **1. AWS Lightsail 建立 instance**

1. 前往 **[Lightsail](https://lightsail.aws.amazon.com/)** 首頁
2. 選擇 create instance
3. 選擇 instance image & WordPress
![select instance image & blueprint](/assets/images/2021-07-01-create-instance-1.png)
4. 選擇方案（依照個人需求）
![](/assets/images/2021-07-01-create-instance-2.png)
5. 輸入instance名稱：隨意取名就可以了，例如我的網站叫triztravel.com，那執行個體就取名叫triztravel-1
6. create instance

<br>

### **2. 登入新建立的WordPress部落格**

![login to new wordpress](/assets/images/2021-07-01-login-to-new-wordpress.png)

建立好instance之後，可以看到以上畫面，而`18.183.232.128`就是我的public IP，把這串數字貼到browser裡面就可以看到新建好的、乾淨的WordPress部落格。透過右下角的藍色"manage"圖示變可以登入Dashboard。這裡要注意的是，首次登入需要透過預設帳號密碼：

- username: user
- password: 到instance的終端機，使用 `cat bitnami_application_password`指令取得預設密碼

<br>

### **3. 舊WordPress網站打包**

處理完新主機的基本設定，現在要將舊網站一鍵打包搬到新主機。我用的搬家外掛是 [All-in-One WP Migration](https://tw.wordpress.org/plugins/all-in-one-wp-migration/)（雖然WordPress搬家外掛很多，但我只用過這個而且覺得不錯方便xd）。

1. 首先進入舊網站的`/wp-admin`後台
2. 點入All-in-One WP Migration > 匯出 > 檔案

![export old website](/assets/images/2021-07-01-wp-migration-export.png)

3. 整個網站會被下載到local端，檔案為`.wpress`格式

<br>

### **4. 將整包舊網站import到新主機**

要把整包舊網站上傳的話，新網站也需安裝All-in-one WP migration。目前免費版的All-in-one WP migration支援上傳40MB以下大小的檔案，如果你的網站超過40MB，有兩個解決辦法：

1. 使用付費版 All-in-one WP migration
2. 修改免費版 All-in-one WP migration 對於上傳檔案大小的限制

貧窮如我當然是走第二條路。我原本直接去改 All-in-one WP migration 的config但失敗，找了資料後發現使用這個做法的All-in-one WP migration必須是v6.7以下的版本。如果直接安裝不小心裝到太新的版本的話（像我裝到7.4）就需要downgrade。可以去下載v6.7的zip檔，到 `plugins` > `upload plugins` 直接用上傳檔案的方式安裝，最後他會詢問要不要覆蓋先前安裝的版本，選確認就可以了。

增加import檔案的max_size這個步驟，可以透過修改外掛的一行程式碼來達成。

1. 到左側sidebar的 外掛 > 外掛編輯器
2. 右側 選取需要編輯的外掛 > All-in-one WP Migration > constants.php
3. 找到檔案大小限制的程式碼（可能在284行）並修改


```php
define( 'AI1WM_MAX_FILE_SIZE', 2 << 28 );
```

上面為原本default的限制，如果我們想要將檔案大小升級成10GB的話，將其改成以下程式碼：

```php
define( 'AI1WM_MAX_FILE_SIZE', 536870912 * 20 );
```

到 All-in-one WP Migration > 匯入 即可以看到maximum upload file size: 10GB。最後一步只要將剛剛下載到本地的 `.wpress` 檔案上傳就大功告成。

![import to new website](/assets/images/2021-07-01-wp-migration-import.png)


<br>

### **5. DNS重新設定**

雖然instance剛啟動好的時候，可以直接透過public IP前往網站（例如我的public IP: [18.183.232.128](http://18.183.232.128/)），但最後一步還是要將買好的網域指向這個IP位址，訪客才能透過domain進入網站，而這個步驟須通過設定DNS完成。我原本是使用siteground主機帶的DNS服務，但網站搬到Lightsail，因此改用Lightsail自帶的DNS服務。

#### **5-1 新增DNS zone**

1. 進入Lightsail主控台 > Networking > Create DNS zone
2. 輸入網域（eg. triztravel.com）
3. Create DNS zone

#### **5-2 add record**

新增好DNS區域後，要告訴伺服器說，所有造訪`triztravel.com`網域的人，請把他們帶到`18.183.232.128`這個IP位址，步驟如下：

 1. 找到剛剛建立的DNS zone，進入manage頁面
 2. 按 `+ Add Record`
 3. 選擇`A record`（用IPv4導向domain）、subdomain留白、並在resolves填入主機的IP位址
 4. 按下左側綠勾勾確認
 5. 在DNS zone裡即可以看到被分配到的nameservers

![](/assets/images/2021-07-01-dns-setup-add-record.png)

#### **5-3 到網域供應商端重新設定DNS**

最後一個步驟是將網域 DNS 記錄的管理轉移至AWS Lightsail。

1. 找到Lighsail分配給我的nameservers並複製名稱
2. 到目前的網域供應商（eg. namecheap）修改nameservers

![namecheap advanced dns setup](/assets/images/2021-07-01-dns-setup.png)

雖然官方是說要等最多48hr才會生效，但實際上大概10分鐘就會重新指向。完成DNS重新設定後再造訪你的網域，就會得到跟原本一模一樣的部落格了。

<br>

參考資源：

- [Increase 512MB limit in All-in-One WP Migration](https://medium.com/@bernardagustinowidjanarko/increase-512mb-limit-in-all-in-one-wp-migration-9a3a9216dea)
- [All-in-One WP Migration Import Stuck Solved](https://webhostingadvices.com/all-in-one-wp-migration-import-stuck/)
- [Host a Wordpress Website on AWS using Amazon Lightsail (For Only $3.50 a month!)](https://youtu.be/49aOUHkvlgg)
- [改個 DNS 是要改多久？- Domain 管理的常見問題](https://medium.com/starbugs/%E9%80%A3-pm-%E4%B9%9F%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84-dns-%E5%B0%8F%E7%9F%A5%E8%AD%98-d00b43e4fe9a)
- [在 中建立 DNS 區域以管理網域的 DNS 記錄Amazon Lightsail](https://lightsail.aws.amazon.com/ls/docs/zh_tw/articles/lightsail-how-to-create-dns-entry)


<br>
