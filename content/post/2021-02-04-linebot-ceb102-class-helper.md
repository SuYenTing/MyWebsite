---
title: LineBot-課程小幫手
author: Yen-Ting, Su
date: '2021-02-04'
slug: linebot-ceb102-class-helper
categories:
  - Programming
tags:
  - Docker
  - MySQL
  - Python
  - Ngrok
  - LineBot
description: "這個是我自己獨力製作的小專案，主要是透過Docker部署一個Line機器人，這個Line機器人可以主動推播訊息提醒學生上課時間，使用者也可以與機器人進行互動，輸入關鍵字查詢教學相關表單。"
---

## 一、 前言

我在Tibame中壢上AI/Big Data資料分析師養成班，最近有學習到如何應用Docker架設Line機器人，以下是我實作的兩個作業：

* [Docker作業一筆記](https://suyenting.github.io/post/docker-hw1/)：說明如何以docker-compose部署MongoDB與Jupyter，並在Jupyter內以Python程式碼對MongoDB進行「增刪改查」。
* [Docker作業四筆記](https://suyenting.github.io/post/docker-hw4/)：說明如何以docker-compose部署Line機器人，使用到MySQL、Python(Flask)和Ngrok三個容器。這個Line機器人可以記錄使用者傳輸的訊息，以及儲存使用者上傳的照片檔案。

因為我對Line機器人還蠻有興趣的，且透過作業已掌握到基本的作法，所以想說來開發一些應用功能，解決一些生活實際遇到問題，順便鍛鍊自己的實作能力。

本次實作的的程式碼有放到[GitHub](https://github.com/SuYenTing/linebot-ceb102)上，但因為有一些是比較私人的資訊(例如資料庫帳密等)，這部份會拿掉，但我會留下輸入的格式提供參考。

## 二、 要解決的問題

目前在Tibame中壢上AI/Big Data資料分析師養成班，有遇到4個比較麻煩的事情：

>1. 因為中心請業界的講師授課，所以課表的時間不是固定的，有時候課程會調來調去，需要定期查看課程表來確認哪時候要上課。
>2. 課程時段分為早、中、晚，每個時段上課前需要打開手機的APP打卡，代表有出席課程，但同學們很常會忘記。
>3. 每天上課都會輪流安排一位同學寫教學日誌，可是同學們很容易忘記現在要輪到誰。
>4. 中心有很多線上表單(例如課表、課程滿意度調查、教學日誌等)，但每次找表單時都要花時間搜尋，有沒有辦法能夠更快的取得表單。

為解決上述問題，我決定來製作一個Line機器人，能夠有以下功能：

>1. 在每晚查看明天是否有課程，若有課程則推播訊息到群組，提醒同學明日要來上課。
>2. 每個時段上課前10分鐘，推播訊息到群組提醒同學記得要打卡，和提醒今日要負責撰寫教學日誌的同學。
>3. 使用者可向Line機器人詢問，取得需要的線上表單連結。

接下來我將依序說明，我是如何建立Line機器人來解決以上問題。

## 三、 資料清洗與整理

為能夠讓Line機器人解決問題，此處我設計兩張資料表，第一張表示課程資料表，讓程式能夠知道什麼時候要上課，進而做對應的訊息處理。第二張是Line機器人要讀的對話關鍵字，只要Line機器人讀到線上表單的關鍵字時，可以立即回應訊息給使用者，讓使用者能夠拿到線上表單的連結。

### 1. 課程資料表

為能夠讓程式定時推播課程資訊，首要的任務是要先把課表進行資料清洗，讓程式能夠方便讀取並進行對應處理。

中心的課表放在Google試算表，以連結分享給學員。課表內容大概像下面這樣：

![](https://i.imgur.com/WZAemp2.png)

可以看到若這個課表不符合資料正規化的要求，若直接給程式讀取做處理會有問題，勢必要先進行清洗整理。我透過Python將這個資料整理成像以下的資料表：

| 日期 | 期間 | 課程名稱 |
| -------- | -------- | -------- |
| 2021-02-01 | 上午 | Data Mining - 陳老師 |
| 2021-02-01 | 下午 | Data Mining - 陳老師 |
| 2021-02-06 | 下午 | PyETL - 施老師 |
| 2021-02-06 | 夜間 | PyETL - 施老師 |
| 2021-02-08 | 夜間 | 夜間課輔 - 蔡老師 |

整理成這個資料表後，程式在處理上就會變得很方便。

註：此處本來我是想要採用爬蟲自動下載Google試算表檔案擷取資料，這樣可以定時排程進行更新。Google有提供和Python的連接方式([參考文檔連結](https://developers.google.com/sheets/api/quickstart/python))，但因為表單的權限不是我的，所以沒辦法這樣做，這邊需要再研究一下。

除了課程表整理完後，還要加入該堂課負責撰寫教學日誌的同學，中心的規則是每日就一位同學負責撰寫，不管當日有幾堂課。此處依據學員名單，透過迴圈的方式輪流安排到各課程上。但比較麻煩的是，課表上有些課程並不需要撰寫教學日誌，例如「專題輔導」或「夜間課輔」等，所以迴圈時還需要考慮這些例外情況。最後整理的資料表如下：

| 日期 | 期間 | 課程名稱 | 教學日誌負責人 |
| -------- | -------- | -------- | -------- | 
| 2021-02-01 | 上午 | Data Mining - 陳老師 | A同學 |
| 2021-02-01 | 下午 | Data Mining - 陳老師 | A同學|
| 2021-02-06 | 下午 | PyETL - 施老師 | B同學 |
| 2021-02-06 | 夜間 | PyETL - 施老師 | B同學 |
| 2021-02-08 | 夜間 | 夜間課輔 - 蔡老師 | NULL |

完整的課程資料表整理程式碼可參考[GitHub程式碼](https://github.com/SuYenTing/linebot-ceb102/blob/main/app/curriculumToSQL.py)。

### 2. Line關鍵字對應訊息

為能夠讓Line機器人與使用者進行互動，此處設計關鍵字對應訊息表，當Line機器人比對到關鍵字時，能夠馬上回答對應訊息。(希望有朝一日能開發出AI聊天機器人!)

此處的關鍵字包含有：課程資訊表、教學日誌、筆記、課程滿意度調查表、出缺勤統計表及教室預約登記表，但由於每個關鍵字可能會有「簡稱」的狀況，所以此處我也加入使用者可能輸入的簡稱。為減少要輸入字詞，此處我採用json的格式來建立關鍵字及對應訊息資料：

```json=
{
	"課程資訊表": {
		"keyword": ["課程資訊表", "課程表", "課表"],
		"msg": "您要的課程資訊表連結來囉! \n https://docs.google.com/..."
	},

	"教學日誌": {
		"keyword": ["教學日誌", "教學日記"],
		"msg": "您要的教學日誌連結來囉! \n https://docs.google.com/..."
	},

	"筆記": {
		"keyword": ["筆記"],
		"msg": "您要的筆記連結來囉! \n https://drive.google.com/..."
	},

	"課程滿意度調查表": {
		"keyword": ["課程滿意度調查表", "課程滿意度", "滿意度"],
		"msg": "您要的課程滿意度調查表連結來囉! \n https://docs.google.com/..."
	},

	"出缺勤統計表": {
		"keyword": ["出缺勤統計表", "出缺勤"],
		"msg": "您要的出缺勤統計表連結來囉! \n https://docs.google.com/..."
	},

	"教室預約登記表": {
		"keyword": ["教室預約登記表", "教室預約", "教室", "預約"],
		"msg": "您要的教室預約登記表連結來囉! \n https://docs.google.com/..."
	}
}
```

接下來透過Python的pandas套件，可將此json檔案轉換成資料表，以下列出重要的程式碼，完整的程式碼可參考[GitHub程式碼](https://github.com/SuYenTing/linebot-ceb102/blob/main/app/keywordToSQL.py)。

``` python= 
# read linebot keyword table
keywordTable = pd.read_json('linebot_keyword.json', orient='index')
keywordTable = keywordTable.explode('keyword')
keywordTable['method'] = keywordTable.index
keywordTable = keywordTable[['method', 'keyword', 'msg']]
```

上面的程式碼執行完後，資料表格式如下：

| 表單名稱 | 關鍵字 | 對應訊息 |
| -------- | -------- | -------- |
| 課程資訊表 | 課程資訊表 | 您要的課程資訊表連結來囉! https://docs.google.com/... |
| 課程資訊表 | 課程表 | 您要的課程資訊表連結來囉! https://docs.google.com/... |
| 課程資訊表 | 課程表 | 您要的課程資訊表連結來囉! https://docs.google.com/... |
| 教學日誌 | 教學日誌 | 您要的教學日誌連結來囉! https://docs.google.com/... |
| 教學日誌 | 教學日記 | 您要的教學日誌連結來囉! https://docs.google.com/... |

在使用者輸入訊息後，後台可以從訊息中比對有沒有關鍵字在這個表上，如果有則輸出對應訊息給使用者。

## 四、 Line機器人邏輯處理

Line機器人部分，主要是用Python的Flask搭建，並搭配Line的[line-bot-sdk-python](https://github.com/line/line-bot-sdk-python)套件撰寫。本章節分別說明「推播提醒訊息」與「關鍵字回應」重要的程式碼部分。完整的程式碼可參考[GitHub程式碼](https://github.com/SuYenTing/linebot-ceb102/blob/main/app/main.py)。

### 1. 推播提醒訊息

為能夠定時推播提醒訊息，此處透過[flask_apscheduler](https://pypi.org/project/Flask-APScheduler/)套件來進行排程管理。

關於flask_apscheduler這個套件，我主要是參考[python3+flask 開發web(九)——flask_apscheduler定時任務框架](https://www.itread01.com/content/1545089771.html)文章，從裡面的程式碼提取進行改寫。

為達成前述所說的Line機器人功能：

>1. 在每晚查看明天是否有課程，若有課程則推播訊息到群組，提醒同學明日要來上課。
>2. 每個時段上課前10分鐘，推播訊息到群組提醒同學記得要打卡，和提醒今日要負責撰寫教學日誌的同學。

在Config這個類別下加入排程，我共加入了4個排程。
```python=
# 排程任務
class Config(object):
    JOBS = [{'id':'morning', 'func':'__main__:RemindClass', 'args':('上午',), 'trigger':'cron', 'hour':8, 'minute':50},
            {'id':'afternoon', 'func':'__main__:RemindClass', 'args':('下午',), 'trigger':'cron', 'hour':13, 'minute':20},
            {'id':'night', 'func':'__main__:RemindClass', 'args':('夜間',), 'trigger':'cron', 'hour':18, 'minute':20},
            {'id':'tmrClass', 'func':'__main__:RemindTmrClass', 'trigger':'cron', 'hour':21, 'minute':30}]
    SCHEDULER_TIMEZONE = 'Asia/Taipei'
```

Jobs為一個list包多個任務，每個任務以字典方式儲存，裡面的參數名稱為：
* id：任務名稱
* func：排程執行的函數
* args：輸入函數的引數值
* trigger：有分為cron和interval，此處用cron
    * cron：依確切時間執行排程任務(例如12:00執行)
    * interval：定時執行排程任務(例如每5分鐘執行一次)
* hour: 排程執行的時
* mintue: 排程執行的分

此處我的排程任務設定為：
1. 每天08:50、13:20及18:20執行RemindClass函數，RemindClass函數會至資料庫查詢當日課程資訊，判斷當日上午/下午/夜間是否有課，若有課程則推播訊息提醒學員打卡和教學日誌負責人。
2. 每晚21:30執行RemindTmrClass函數，判斷隔日是否有課程，若有課程則推播訊息提醒學員。

另外由於我的程式是放在AWS的雲端機器上執行，會面臨到時區不一致的問題，導致排程和我們預期的時間不一致。此處搜尋網路上的解法([參考此頁面](https://www.jianshu.com/p/b1881db7bc85))，主要是在Config類別下加入時區設定`SCHEDULER_TIMEZONE = 'Asia/Taipei'`。

另外在最後啟用Flask時，將`scheduler=APScheduler()`這行程式碼改為`scheduler = APScheduler(BackgroundScheduler(timezone="Asia/Taipei"))`，即可讓排程依據台灣的時區來執行。

```python=
# 開始運作Flask
if __name__ == "__main__":
    scheduler = APScheduler(BackgroundScheduler(timezone="Asia/Taipei"))  # 例項化APScheduler
    scheduler.init_app(app)  # 把任務列表放進flask
    scheduler.start()        # 啟動任務列表
    app.run(host='0.0.0.0')
```

程式運作後，Line機器人就會定時在群組內推播訊息，例如這是在早上8:50推播的訊息：

![](https://i.imgur.com/eqmTgeV.png)


### 2. 關鍵字回應

在Line機器人關鍵字回應部分，主要是對應到前文說的Line機器人功能：

>3. 使用者可向Line機器人詢問，取得需要的線上表單連結。

我的設計是使用者需要在訊息中包含「小幫手」這3個字，這樣我的Line機器人才會做回應。

如果使用者輸入「小幫手 給我課表」，則程式讀取到「小幫手」這三個字後，又偵測到「課表」這個關鍵字，就會提出回應訊息。例如：

![](https://i.imgur.com/oHlD0Xh.png)

這樣可以避免如果使用者只是單純聊天提到關鍵字時，結果Line機器人跑出來做回應會很奇怪。

但如果使用者輸入「小幫手」，後面接的關鍵字沒有在我們的清單內，則此時要提醒使用者有哪些關鍵字可以輸入使用。例如：

![](https://i.imgur.com/J97HoXj.png)

在程式內，主要是在接收文字訊息時進行處理，當發現使用者訊息有「小幫手」3個字存在時，即執行`FindKeyWordInText`函數(此為我自行定義的函數)。

此函數會至資料庫內取得目前的關鍵字對應訊息，並進行比對，若有比對到則輸出對應訊息，若沒有則輸出提醒訊息。

此函數輸出要回復的訊息`replyMsg`後，透過`line_bot_api`回傳訊息給使用者。

```python=
# linebot處理文字訊息
@handler.add(MessageEvent, message=TextMessage)
def handle_message(event):

    # linebot關鍵字回傳訊息
    if '小幫手' in event.message.text:
        replyMsg = FindKeyWordInText(text=event.message.text, userId=event.source.user_id)
        line_bot_api.reply_message(event.reply_token, TextSendMessage(text=replyMsg))
```

## 五、 Line機器人架構與部署

在撰寫好程式碼後，接下來就要開始進行部署。

CEB102課程小幫手Line機器人架構圖如下：

![](https://i.imgur.com/evTNSNG.png)

因為希望服務能夠常駐，所以我在AWS上開設一台虛擬主機和資料庫，虛擬主機是用Ubuntu 18.04版本，資料庫為MySQL 8.0。

此處使用Docker Compose來協助部署line機器人服務，共部署Python和Ngrok兩個容器。

Python用來做Flask的服務，即機器人的核心程式。

Ngrok主要是協助在沒有實體IP的情況下，透過Ngrok可以產出網址，讓外網來連入服務。雖然雲端本身已有實體IP，但此處用Ngrok的好處是他的網址有https加密，這樣才能符合Line的Webhook要求。

Docker Compose的yaml檔案有上傳至[Github](https://github.com/SuYenTing/linebot-ceb102/blob/main/docker-compose.yml)，另外裡面有用到的dockerfile也有上傳到[Github](https://github.com/SuYenTing/linebot-ceb102/blob/main/dockerfile/dockerfile-flask)。

## 六、結語

這個小專案靠自己獨力完成，從0到1花我大概整整兩天時間，寫這個文檔又花了一個早上。

資料清洗部分就花了半天，因為以前是寫R語言，現在要強迫自己用Python的pandas套件來處理，光是google查寫法加上理解就花很多時間，但透過這次實作對panda套件用法更加熟悉。

另外花時間的部分是在想排程的架構要怎麼做，有沒有需要寫到Linux的crontab，後來google發現`flask_apscheduler`套件似乎可行，仔細研究一下寫法並做測試沒問題後，就開始用`flask_apscheduler`套件來撰寫排程。

目前還有一個問題需要解決：ngrok的連線會有8小時的時間限制，超過時間會自動斷掉，導致機器人無法常駐提供服務。

網路上有提供將Line機器人部署到Heroku的方法([參考文章](https://medium.com/@mengchiang000/%E4%BD%88%E7%BD%B2%E8%81%8A%E5%A4%A9%E6%A9%9F%E5%99%A8%E4%BA%BA-linebot-%E5%88%B0heroku-65cbe34c8324))，之後會來嘗試將服務改成部署到Heroku上面。

