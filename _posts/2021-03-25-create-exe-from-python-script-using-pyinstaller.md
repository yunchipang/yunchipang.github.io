---
layout: post
title: 使用PyInstaller將Python打包成執行檔(executable)
permalink: /create-exe-from-python-script-using-pyinstaller.html
date: 2021-03-25
author: yunchipang
tags: [python]
---

這篇文章敘述的專案和[上篇](https://yunchipang.github.io/run-selenium-on-gcp-and-update-to-google-sheets.html)是同一份，只是基於有的時候不一定所有request都值得花那個金錢成本和時間成本架到GCP或是AWS，當時主管建議的做法有以下三種：

1. 將原始python script分享給提出request的同事，讓他們需要的時候自己使用command line執行。
	* 優點：我們這邊寫完script就可以結束。
	* 缺點：同事電腦需要建置運算環境，如果專案比較複雜，需要安裝的模組跟套件就更多。就算幫一人完成安裝，之後無論是同事換電腦或是專案轉給別人負責，都需要重新幫忙建置。另外，使用command line指令也是潛在的技術門檻。
2. 將python script打包成執行檔(executable)並將檔案交給