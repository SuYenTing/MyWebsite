---
title: 用Heroku來部署LineBot-課程小幫手
author: Yen-Ting, Su
date: '2021-02-06'
slug: linebot-ceb102-class-helper-heroku
categories:
  - Programming
tags:
  - Heroku
  - Python
  - LineBot
description: "在上一篇文章「[LineBot-課程小幫手](https://suyenting.github.io/post/linebot-ceb102-class-helper/)」提到，我們透過Ngrok產生一個有加密的https網址，讓Line來聯繫我們寫的Flask。可是Ngrok免費版有一個致命的缺點是8小時就會自動停止服務，若需要再用必須要換一個新的網址，對我們來說會非常麻煩。為解決上述問題，本篇文章將說明如何用Heroku來部署Line機器人服務。"
---

此篇文章用到的程式碼已有上傳至[Github](https://github.com/SuYenTing/linebot-ceb102-heroku)，可至Github觀看完整的程式碼及檔案架構。

## 一、前言

在上一篇文章「[LineBot-課程小幫手](https://suyenting.github.io/post/linebot-ceb102-class-helper/)」提到，我們透過Ngrok產生一個有加密的https網址，讓Line來聯繫我們寫的Flask。可是Ngrok免費版有一個致命的缺點是8小時就會自動停止服務，若需要再用必須要換一個新的網址，對我們來說會非常麻煩。

上面這個問題，可以透過Heroku來協助我們解決。Heroku可以直接上傳Python的Flask檔案，他會直接部署並啟用服務，且服務的網址是有加密的https，非常方便。缺點是若半小時內沒能使用服務，即會進入休眠，若有人來造訪才會被喚醒。但網路上有人提供「自動喚醒」的方法協助解決，所以這個問題可以被解決。

所以接下來這篇文章，將示範如何把LineBot課程小幫手放到Heroku上來提供服務。在一開始要先準備：

1. 申請Heroku帳號([Heroku首頁](https://www.heroku.com/))
2. 安裝[Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)：可以透過git指令管理放在Heroku的檔案

這篇文章為上篇文章[LineBot-課程小幫手](https://suyenting.github.io/post/linebot-ceb102-class-helper/)的延伸，如果想了解此機器人的功能和要解決的問題，可參考該篇文章。

## 二、建立部署檔案

在安裝好Heroku帳號和Heroku CLI後，首先建立相關的部署檔案設定，以下是我部署提交資料夾內的檔案：

* Procfile

此為讓Heroku知道部署時對應的服務與要執行的檔案，我的設定如下：

```
web: gunicorn main:app –log-file -
clock: python clock.py
```

web表示這是一個web的服務，gunicorn是一個可以部署Python的HTTP server，後面的main表示我們要執行的web程式是`main.py`。

clock表示這是一個定時排程器，`clock.py`即為要定時排程的檔案。Heroku官網推薦使用APScheduler套件，詳情可參考[Heroku官網說明-Scheduled Jobs with Custom Clock Processes in Python with APScheduler](https://devcenter.heroku.com/articles/clock-processes-python)。


Procfile其他詳細設定說明可參考[Heroku官網說明-The Procfile](https://devcenter.heroku.com/articles/procfile)。

* requirements.txt

部署Python服務時需要安裝的套件清單，檔案內容如下：

```
gunicorn
flask
line-bot-sdk
mysql-connector-python
APScheduler==3.0.0
requests
```

* runtime.txt
Python安裝的版本，此處要先查閱[Heroku可安裝版本的清單](https://devcenter.heroku.com/articles/python-support)(會隨時更新)，不然會報錯。此處安裝python-3.7.9版本，檔案寫法如下：

```
python-3.7.9
```

* main.py

此為Python Flask程式碼，主要用於Line機器人在接收使用者訊息後，判斷要回應的訊息。

* clock.py

此為Python程式碼，主要用於Line機器人定時推播訊息到群組，以及自動喚醒Heroku服務功能，裡面有撰寫APScheduler套件的程式碼。

此處特別說明一個機制：自動喚醒。由於Heroku在半小時內沒人來使用服務，即會自動休眠，且**休眠後排程也會跟著一起休眠，不會因為排程時間到而自動啟動排程**。所以為維持機器人的推播功能，一定要讓Heroku持續醒著提供服務。

參考這邊文章[LINE BOT SDK：Heroku 夜未眠（一）](https://ithelp.ithome.com.tw/articles/10218874)的作法，透過Python的requests套件，定時排程去requests我們的服務網址，這樣就可以讓Heroku以為我們的服務一直有人來使用。此處透過APScheduler套件來設定排程，關鍵的程式碼如下：

```python=
# 防止睡眠
def DoNotSleep():
    url = "https://linebotceb102.herokuapp.com/"
    r = requests.get(url)

# 防止自動休眠
sched.add_job(DoNotSleep, trigger='interval', id='doNotSleeps_job', minutes=20)
```

從上面的程式碼，可以看到我每20分鐘即會去request服務的網址。

* secretFile.json
此為LineBot的token權限與資料庫帳密，main.py和clock.py兩支程式會使用到此資料。

## 三、開始部署

首先在Heroku建立新專案：

![](https://i.imgur.com/9wHoI6R.png)

輸入專案名稱，這個名稱會是待會服務的網址名稱：

![](https://i.imgur.com/TXlvAJC.png)

建立好後，打開命令提示字元，首先在這台電腦登入Heroku，這個只要做一次即可：
```bash=
# 登入heroku
heroku login
```

接下來執行以下命令，將撰寫好的檔案部署到Heroku上面。

```bash=
# 切換到專案路徑
cd C:\Users\User\Desktop\herouku-linebot

# git指令-將此資料夾和Herouku連動(第一次執行才使用)
git init
heroku git:remote -a linebot-class-helper

# git指令(第一次及往後更新時使用)
git add .
git commit -am "make it better"
git push heroku master
```

以下是我執行上述指令的畫面印出的內容：

```bash=
C:\Users\User>cd C:\Users\User\Desktop\herouku-linebot

C:\Users\User\Desktop\herouku-linebot>git init
Initialized empty Git repository in C:/Users/User/Desktop/herouku-linebot/.git/

C:\Users\User\Desktop\herouku-linebot>heroku git:remote -a linebot-class-helper
set git remote heroku to https://git.heroku.com/linebot-class-helper.git

C:\Users\User\Desktop\herouku-linebot>git add .
warning: LF will be replaced by CRLF in main.py.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in secretFile.json.
The file will have its original line endings in your working directory

C:\Users\User\Desktop\herouku-linebot>git commit -am "make it better"
[master (root-commit) d241e91] make it better
 6 files changed, 238 insertions(+)
 create mode 100644 Procfile
 create mode 100644 clock.py
 create mode 100644 main.py
 create mode 100644 requirements.txt
 create mode 100644 runtime.txt
 create mode 100644 secretFile.json

C:\Users\User\Desktop\herouku-linebot>git push heroku master
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (8/8), 3.78 KiB | 968.00 KiB/s, done.
Total 8 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Building on the Heroku-20 stack
remote: -----> Python app detected
remote: -----> Installing python-3.7.9
remote: -----> Installing pip 20.1.1, setuptools 47.1.1 and wheel 0.34.2
remote: -----> Installing SQLite3
remote: -----> Installing requirements with pip
remote:        Collecting gunicorn
remote:          Downloading gunicorn-20.0.4-py2.py3-none-any.whl (77 kB)
remote:        Collecting flask
remote:          Downloading Flask-1.1.2-py2.py3-none-any.whl (94 kB)
remote:        Collecting line-bot-sdk
remote:          Downloading line_bot_sdk-1.18.0-py2.py3-none-any.whl (66 kB)
remote:        Collecting mysql-connector-python
remote:          Downloading mysql_connector_python-8.0.23-cp37-cp37m-manylinux1_x86_64.whl (18.0 MB)
remote:        Collecting APScheduler==3.0.0
remote:          Downloading APScheduler-3.0.0-py2.py3-none-any.whl (46 kB)
remote:        Collecting click>=5.1
remote:          Downloading click-7.1.2-py2.py3-none-any.whl (82 kB)
remote:        Collecting Jinja2>=2.10.1
remote:          Downloading Jinja2-2.11.3-py2.py3-none-any.whl (125 kB)
remote:        Collecting itsdangerous>=0.24
remote:          Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
remote:        Collecting Werkzeug>=0.15
remote:          Downloading Werkzeug-1.0.1-py2.py3-none-any.whl (298 kB)
remote:        Collecting requests>=2.0
remote:          Downloading requests-2.25.1-py2.py3-none-any.whl (61 kB)
remote:        Collecting future
remote:          Downloading future-0.18.2.tar.gz (829 kB)
remote:        Collecting protobuf>=3.0.0
remote:          Downloading protobuf-3.14.0-cp37-cp37m-manylinux1_x86_64.whl (1.0 MB)
remote:        Collecting pytz
remote:          Downloading pytz-2021.1-py2.py3-none-any.whl (510 kB)
remote:        Collecting six
remote:          Downloading six-1.15.0-py2.py3-none-any.whl (10 kB)
remote:        Collecting tzlocal
remote:          Downloading tzlocal-2.1-py2.py3-none-any.whl (16 kB)
remote:        Collecting MarkupSafe>=0.23
remote:          Downloading MarkupSafe-1.1.1-cp37-cp37m-manylinux2010_x86_64.whl (33 kB)
remote:        Collecting certifi>=2017.4.17
remote:          Downloading certifi-2020.12.5-py2.py3-none-any.whl (147 kB)
remote:        Collecting urllib3<1.27,>=1.21.1
remote:          Downloading urllib3-1.26.3-py2.py3-none-any.whl (137 kB)
remote:        Collecting chardet<5,>=3.0.2
remote:          Downloading chardet-4.0.0-py2.py3-none-any.whl (178 kB)
remote:        Collecting idna<3,>=2.5
remote:          Downloading idna-2.10-py2.py3-none-any.whl (58 kB)
remote:        Building wheels for collected packages: future
remote:          Building wheel for future (setup.py): started
remote:          Building wheel for future (setup.py): finished with status 'done'
remote:          Created wheel for future: filename=future-0.18.2-py3-none-any.whl size=491058 sha256=227198e59d98e19080bca5d76a2bba414d024daa76f24c501cdecb8540684c6f
remote:          Stored in directory: /tmp/pip-ephem-wheel-cache-134xi0gt/wheels/56/b0/fe/4410d17b32f1f0c3cf54cdfb2bc04d7b4b8f4ae377e2229ba0
remote:        Successfully built future
remote:        Installing collected packages: gunicorn, click, MarkupSafe, Jinja2, itsdangerous, Werkzeug, flask, certifi, urllib3, chardet, idna, requests, future, line-bot-sdk, six, protobuf, mysql-connector-python, pytz, tzlocal, APScheduler
remote:        Successfully installed APScheduler-3.0.0 Jinja2-2.11.3 MarkupSafe-1.1.1 Werkzeug-1.0.1 certifi-2020.12.5 chardet-4.0.0 click-7.1.2 flask-1.1.2 future-0.18.2 gunicorn-20.0.4 idna-2.10 itsdangerous-1.1.0 line-bot-sdk-1.18.0 mysql-connector-python-8.0.23 protobuf-3.14.0 pytz-2021.1 requests-2.25.1 six-1.15.0 tzlocal-2.1 urllib3-1.26.3
remote: -----> Discovering process types
remote:        Procfile declares types -> clock, web
remote:
remote: -----> Compressing...
remote:        Done: 73.1M
remote: -----> Launching...
remote:        Released v3
remote:        https://linebot-class-helper.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/linebot-class-helper.git
 * [new branch]      master -> master
```

若部署沒有問題，結果就會如上面，若有問題，就會印出錯誤訊息。

接下來到Heroku的Dashboard畫面中，點選Overview的頁籤，可看到我們剛在Procfile設定要啟用的功能已經列在上面。但clock後面卻顯示OFF。

![](https://i.imgur.com/9O4vYfU.png)

如果要啟用的話，可以到Resources頁籤，在clock點選修改按鈕，將clock功能打開，這樣即可開啟排程功能。

![](https://i.imgur.com/7LZp1OR.png)

## 四、設定時區

由於LineBot有進行排程，排程的時間設定是以台灣時區為準，但Heroku默認設定的時區會和台灣不一致，所以需要額外設定，這樣排程才會如我們所想的運作。

時區設定在Settings的頁籤底下，在Config Vars區塊點選Reveal Config Vars按鈕。

![](https://i.imgur.com/ipKwH1P.png)

接下來輸入參數名稱：`TZ`及參數值：`Asia/Taipei`，按下Add按鈕，這樣即可設定好時區。

![](https://i.imgur.com/cJbdFYE.png)


## 五、與Line連動

Line機器人服務部署完後，即可將服務網址貼到Line Develoer的webhook連結設定。

服務網址的格式為：`https://[專案名稱].herokuapp.com/`，可以發現Heroku提供的是加密網址，符合Line機器人Webhook的要求。

以我們的服務為例，網址即為[https://linebot-class-helper.herokuapp.com/](https://linebot-class-helper.herokuapp.com/)。

將服務網址貼到Line Developer的Webhook：

![](https://i.imgur.com/NNRfs83.png)

這樣就大功告成，可以開始使用Line機器人的服務！

## 六、參考文章

* [如何使用 Heroku 部屬一個 Web App 網頁應用程式](https://blog.techbridge.cc/2020/03/08/how-to-use-heroku-to-deploy-clear-mysql-db-web-app-tutorial/)
* [佈署聊天機器人(linebot)到heroku](https://medium.com/@mengchiang000/%E4%BD%88%E7%BD%B2%E8%81%8A%E5%A4%A9%E6%A9%9F%E5%99%A8%E4%BA%BA-linebot-%E5%88%B0heroku-65cbe34c8324)
* [Day04 部署到Heroku](https://ithelp.ithome.com.tw/articles/10239201)
* [Scheduled Jobs with Custom Clock Processes in Python with APScheduler](https://devcenter.heroku.com/articles/clock-processes-python)
* [第 13 天：LINE BOT SDK：Heroku 夜未眠（一）](https://ithelp.ithome.com.tw/articles/10218874)