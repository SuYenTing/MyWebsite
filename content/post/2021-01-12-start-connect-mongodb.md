---
title: 快速啟動及連線MongoDB方法
author: Yen-Ting, Su
date: '2021-01-12'
slug: start-connect-mongodb
categories:
  - Programming
tags:
  - MongoDB
  - Batch
description: "最近開始學MongoDB，因為每次執行MongoDB都要在Windows的命令提示字元打一些程式碼，所以想說寫個簡單的batch檔案及設定環境變數來解決這個麻煩的事情。"
---

補習班最近開始上MongoDB課程，因為每次上課時都要打一些程式碼來執行MongoDB，所以想說寫個簡單的batch檔案及環境變數設定來解決這個麻煩的事情。

# 1. 啟用MongoDB資料庫的Server端

打開文字檔，輸入以下字串：

```bash=
cd C:\Users\User\mongodb-win32-x86_64-windows-4.4.2\bin
mongod --dbpath  C:\Users\User\mongodb_data

# 格式說明
# cd [mongod.exe資料夾路徑]
# mongod --dbpath [MongoDB資料庫位置]
```
![](https://i.imgur.com/Cx7ty87.png)


接下來另存新檔，檔名為`"start_mongodb.bat"`，加上雙引號及bat附檔名代表強制存為批次檔。(檔名也可以取自己喜歡的)

![](https://i.imgur.com/3CA44uf.png)


執行此批次檔即可自動啟動MongoDB資料庫的Server。
![](https://i.imgur.com/nFoUqCc.png)


# 2. 連線MongoDB資料庫

在資料夾的**本機**點右鍵選**內容**。

![](https://i.imgur.com/S2Gfxo3.png)

選**進階系統設定** -> **環境變數** -> **Path** -> **編輯**。

![](https://i.imgur.com/9Gp06L5.png)

在編輯環境變數內，點選**新增**，輸入**mongo.exe資料夾路徑**。

![](https://i.imgur.com/F37CEzn.png)

輸入完之後按確認。

在執行MongoDB Server的情況下，打開**命令提示字元**，輸入`mongo`即可連入MongoDB。

![](https://i.imgur.com/EPhxJmW.png)


# 3. 姚大哥提供更快的做法

補習班有位姚大哥在我分享這篇文章後，提供更好的做法給我參考，受益良多。

以下為姚大哥提供的batch的程式碼，直接將啟動MongoDB Server和連線整合在一起，我有稍微改寫一下程式碼。將下面的程式碼貼入文字檔中，另存為bat副檔名即可使用。

```bash=
@echo off 

set mongo_path="C:\Users\User\mongodb-win32-x86_64-windows-4.4.2\bin"

set mongo_db="C:\Users\User\mongodb_data"

start cmd /k "cd %mongo_path% && mongod --dbpath %mongo_db%"

timeout /t 3

start cmd /k "cd %mongo_path% && mongo"
```

程式碼說明：

```bash=
:: 「::」在表示這行為註解，或者可輸入rem
:: @echo off 表示關閉回顯，開頭加入此行後，執行的命令就不會呈現在命令提示字元上
@echo off 

:: 設定mongodb的程式路徑(此行依據自己的路徑自行修改)
set mongo_path="C:\Users\User\mongodb-win32-x86_64-windows-4.4.2\bin"

:: 設定mongodb的資料庫位置(此行依據自己的路徑自行修改)
set mongo_db="C:\Users\User\mongodb_data"

:: start cmd 表示啟動命令提示字元，後面的 /k 表示執行後面的指令後不關閉視窗
:: cd 為更換路徑
:: %mongo_path% 為變數，表示mongodb的程式路徑
:: && 表示接續執行命令
:: mongod 為建立mongodb server
:: --dbpath 為mongodb的資料庫位置參數
:: %mongo_path% 為變數，表示mongodb的資料庫位置
:: 綜合上述，此行程式碼即為打開命令提示字元，更換至mongodb的程式路徑，建立mongodb server
start cmd /k "cd %mongo_path% && mongod --dbpath %mongo_db%"

:: 暫停3秒，此處暫停是因為要等待server建立好後才執行連線
timeout /t 3

:: 執行連線mongodb，先切換至mongodb的程式路徑後，執行mongo指令
start cmd /k "cd %mongo_path% && mongo"
```







