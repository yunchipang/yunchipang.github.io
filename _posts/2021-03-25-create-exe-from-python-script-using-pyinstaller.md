---
layout: post
title: 使用PyInstaller將Python檔案打包成執行檔(executable)
permalink: /create-exe-from-python-script-using-pyinstaller.html
date: 2021-03-25
author: yunchipang
tags: [python]
---

這篇文章敘述的專案和[上篇](https://yunchipang.github.io/run-selenium-on-gcp-and-update-to-google-sheets.html)是同一份，只是基於有的時候不一定所有request都值得花那個金錢成本和時間成本架到GCP或是AWS，當時主管建議的做法有以下三種：

<br>

1. 將原始python script分享給提出request的同事，讓他們需要的時候自己使用command line執行。
	* 優點：寫完script就可以結束。
	* 缺點：同事電腦需要建置運算環境，如果專案比較複雜，需要安裝的模組跟套件就更多。就算幫一人完成安裝，之後無論是同事換電腦或是專案轉給別人負責，都需要重新幫忙建置。另外，使用command line指令也是潛在的技術門檻。

2. 將python script架到雲端運算環境並排程自動執行。
	*  優點：自動化程度最高，完成設定之後可以全部排程處理無須手動，偶爾到後台檢查ㄧ下沒有噴錯 即可。負責人員可以完全不用碰到程式碼。
	*  缺點：額外架設時間＆額外花費。

3. 將python script打包成執行檔(executable)並將檔案交給提出request的同事。
	* 優點：可以在任何環境中運行，不需要額外安裝程式語言、套件或模組，就算加上selenium內容也無需另外帶上`chromedriver.exe`，可以一個執行檔解決。
	* 缺點：每次使用還是必須手動點擊。

<br>

最後長官建議使用方法三，即是把python script打包成executable的寫法。畢竟這個方法比較折衷，小案子以不用額外花錢為最高指導原則，方便性中等沒有關係。然後因為在使用`PyInstaller`打包過程中踩了些坑，寫文章記錄一下，也方便自己之後如果還有收到相似的需求可以回頭複習。

<br>

# **python script**

跟一般寫碼的過程比較不同的是，因為這支程式有包含呼叫`chromedriver.exe`來控制瀏覽器執行`selenium`爬蟲的部分，所以在路徑方面會比較複雜一點點。像平常在local運行的話，通常就是告訴程式去哪裡找`chromedriver.exe`，路徑即為`chromedriver.exe`在本機裡面的位置。例如下面這樣：

```python
driver = webdriver.Chrome(executable_path='/path/to/chromedriver.exe')
```

但執行檔運行的原理比較不一樣，在我們雙擊exe檔案之後，電腦會在系統臨時目錄下建立一個臨時資料夾，然後在那裡執行exe。因此如果我在`.py`裡面寫死`chromedriver.exe`的位置，打包成exe後程式便會找不到`chromedriver.exe`的位置。我後來找到的解決方法是在`.py`裡面多寫一個`resource_path()`的function，以應變執行方式，無論是直接執行python script還是打包成exe，程式都會產生相應路徑，並且順利找到`chromedriver.exe`所在的位置。

```python
def resource_path(relative_path):
    # run exe
    try: 
        base_path = sys._MEIPASS
    # run locally
    except Exception: 
        base_path = os.path.dirname(__file__)
    
    joined_path = os.path.join(base_path, relative_path)
    return joined_path
```

這個function的用途是，無論現在我們是直接執行`.py`還是打包成exe後double click執行，程式都可以透過本身執行路徑找到`chromedriver`的相對位置。邏輯是先試試看找`sys._MEIPASS`（exe在運行時生成的依賴文件的路徑），如果出現錯誤，就代表現在程式不是透過exe運行（而是直接執行`.py`），那就拿`os.path.dirname(__file__)`用相對路徑找到`chromedriver`。這個function其實可以用於所有需要尋找絕對路徑的依賴文件，只是專案裡我只需用到`chromedriver`，所以只拿他舉例，其實`relative_path`裡面塞任何東西都可以。我嘗試用兩種方法print`base_path`跟`joined_path`出來的結果如下：

```
# by python3
base_path:  /Users/username/Documents/myscript
joined_path:  /Users/username/Documents/myscript/./driver/chromedriver

# by exe
base_path:  /var/folders/06/rbhkng8d46s9q9bq4dsj__j00000gn/T/_MEIRwV0EA
joined_path:  /var/folders/06/rbhkng8d46s9q9bq4dsj__j00000gn/T/_MEIRwV0EA/./driver/chromedriver
```

而關於`sys._MEIPASS`比較詳細的解釋，可以參考以下文章（我也是第一次遇見）。

- [Python中 sys._MEIPASS 是什麼](https://www.twblogs.net/a/5d034603bd9eee487be9807d)
- [如何使用Pyinstaller - cx_Freeze可執行文件sys.MEIPASS/sys.executable](http://hk.uwenku.com/question/p-ayjbpfbr-bp.html)

<br>

# **[PyInstaller](pyinstaller.readthedocs.io)使用**

首先從專案資料夾架構開始介紹。開始打包時，專案資料夾裡面應該只有原始的`.py`還有`driver`資料夾跟裡面的`chromedriver`。架構如下：

```
├── myscript.py
└── driver
    └── chromedriver
```

最基本的打包指令為：

```shell
$ pyinstaller myscript.py
```

打包完成後用`tree`指令在終端機裡印出的directory架構將會如下。`build`, `dist`和`myscript.spec`都是`pyinstaller`自動產出的，最終可以單獨運行的exe執行檔便會存放在`dist`目錄裡面。另外，因為此份專案需要包含`chromedriver`的使用，我參考了網路上的做法後，在專案資料夾底下多開了一個目錄取名為`driver`，裡面就只存放`chromedriver.exe`這個執行檔。

```
$ tree /path/to/project
├── build
│   └── myscript
├── myscript.py
├── myscript.spec
├── dist
│   └── myscript
└── driver
    └── chromedriver
```

<br>

自動產生的`.spec`檔的概念就是一切關於這個exe你想要specify的內容都可以寫在裡面，如果有其他想指定的內容或想import的library，可以在打包時加`--hidden-import`在`pyinstaller`的指令後面，或是打包完成之後再直接進去修改`.spec`檔。我自己後來比較喜歡直接修改`.spec`因爲不用拘泥於固定的指令語法，直接輸入即可。需要多注意的是，因為使用到的`chromedriver`不是一個`python`模組，我們必須add `chromedriver.exe` as a binary file（而不是用`import`的方式）在`.spec`的`binaries`項目裡面加入`chromedriver.exe`的路徑即可完成這個步驟。最後我的`.spec`裡面的`a`部分是長這樣的。主要需要特別註明的部分就是`binaries`而已。另外因為我第一次打包時error msg有提醒我我需要`cmath`這個module，所以在`hiddenimports`的部分有特別加上。

```
a = Analysis(['myscript.py'],
             pathex=[],
             binaries=[('./driver/chromedriver', './selenium/webdriver')],
             datas=[],
             hiddenimports=['cmath'],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)
```

打包完成之後，我把`dist`裡面的`myscript.exe`壓縮傳給發request的同事，在他的電腦也可以運行無阻後便確認成功。`pyinstaller`其實很好用，網路上資訊夠多doc也寫得很詳盡，除了一併打包`chromedriver`那段卡了有點久，其他部分都很好上手：）

<br>

感謝各前輩及網路上大神指點：

- [Make Python & Selenium program executable (.exe) (How to include webdriver in .exe)](https://www.zacoding.com/en/post/python-selenium-to-exe/)
- [How to include chromedriver with pyinstaller?](https://stackoverflow.com/questions/39563851/how-to-include-chromedriver-with-pyinstaller)
- [將你的python code 打包 - pyinstaller](https://peaceful0907.medium.com/%E5%B0%87%E4%BD%A0%E7%9A%84python-code-%E6%89%93%E5%8C%85-pyinstaller-6777d0e06f58)
- [How to use PyInstaller to create Python executables](https://www.infoworld.com/article/3543792/how-to-use-pyinstaller-to-create-python-executables.html)

<br>

【心得時間】

在二月到三月這段水深火熱地準備GRE的時間中，竟然還可以質量如此高地寫出這份自己覺得學到很多的小案子（大哭）覺得自己摸索中學到最多的是如何寫出flexible的code。因為之前沒機會接很多request，頂多就是在自己local端單次執行，也比較常用jupyter notebook而非寫完整的python script，所以比較沒機會透過設想各種不同的運行環境進而去寫可以應付各種狀況的code，這份真的既好玩又摸到很多新知識，拜託大家多多給我發request吧（跪下）

<br>