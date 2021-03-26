---
title: 以Docker架設frp(fast reverse proxy)流程
author: Yen-Ting, Su
date: '2021-03-26'
slug: docker-frp
categories:
  - DevOps
tags:
  - Docker
  - frp
description: "最近開始要進行Tibame大數據班的專案，中心有提供一台電腦讓我們組別使用，但這台電腦卻只有虛擬IP，如果我在這台電腦上架一個資料庫或網站，沒辦法對外服務。本篇文章提供一個解決方法：在雲端開設免費的主機取得公網IP，並在雲端主機上架設反向代理伺服，實行內網穿透，讓虛擬IP主機能夠對外服務。"
---

## 一、前言

最近開始要進行Tibame大數據班的專案，中心有提供一台電腦讓我們組別使用，但這台電腦卻只有虛擬IP，如果我在這台電腦上架一個資料庫或網站，沒辦法對外服務。當然，我們可以直接把資料庫或網站架在雲端，即可直接對外服務，但是以雲端免費提供的資源，對於我們的需求是遠遠不足的，如果想要用更好的設備，勢必是要付錢。

後來我在網路上看到一篇巴哈姆特的文章，找到一個解法：

[【心得】frp內網穿透架設心得(文長,難)](https://forum.gamer.com.tw/Co.php?bsn=18673&sn=834074)

這篇文章是在說明如何用雲端機器的公網IP，透過frp(fast reverse proxy)做內網穿透，讓外面的人可以連線到自架的Minecraft遊戲。

我看一下這個方法可以解決我們目前的問題，我們只要租一台免費的雲端，拿到公網IP，在上面架一個frp server，然後在中心的電腦架一個frp client，就可以解決我們的需求。使用者只要連線到frp server，即可透過frp使用到frp client上的服務。

雖然標題寫文長、很難，但我看一下內容提到的技術點我自己都有概念，而且有找到frp的Github，裡面的說明文件還蠻詳細：

* [Github: fatedier/frp](https://github.com/fatedier/frp)

另外也有找到docker相關的Image，這兩個Image都有蠻多人在使用的：

* [docker hub: snowdreamtech/frps](https://hub.docker.com/r/snowdreamtech/frps)
* [docker hub: snowdreamtech/frpc](https://hub.docker.com/r/snowdreamtech/frpc)

![](https://i.imgur.com/IJgF3aR.png)

雖然不是官方套件，但下載量蠻大的，可信度應該很高。

另外也有看到dokcer-compose的寫法：[docker hub: cooooper/frp](https://hub.docker.com/r/cooooper/frp/)

綜合上述資訊，以docker compose來建立frp server和frp client是非常可行的。

## 二、frp介紹

frp全名為fast reverse proxy，依[Github: fatedier/frp](https://github.com/fatedier/frp)的介紹:

> frp is a fast reverse proxy to help you expose a local server behind a NAT or firewall to the Internet. As of now, it supports TCP and UDP, as well as HTTP and HTTPS protocols, where requests can be forwarded to internal services by domain name.

frp本質上即是一個反向代理(reverse proxy)的工具，反向代理和正向代理常被容易搞混，也不好理解，這邊我整理一下網路可以參考的資源：

* [終於有人把正向代理和反向代理解釋的明明白白了](https://kknews.cc/zh-tw/tech/k66p2gb.html)
* [WEB的正向代理與反向代理](https://jackterrylau.pixnet.net/blog/post/229520831-web%E7%9A%84%E6%AD%A3%E5%90%91%E4%BB%A3%E7%90%86%E8%88%87%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)

這兩個網頁都有舉出很好的例子來幫助我們理解正向代理和反向代理的差異。

## 三、實作環境說明

我的實作環境共有兩台主機：

* frp server: AWS雲端主機(有固定IP)，作業系統為Ubuntu20.04
* frp client: Win10主機裝[WSL2](https://docs.microsoft.com/zh-tw/windows/wsl/install-win10)，作業系統為Ubuntu18.04

兩台電腦皆裝有Docker環境。

## 四、frp server架設方式

frp server即為具有公網IP的主機，我們在雲端主機(Ubuntu20.04)環境執行以下指令：

```bash=
# 切換至home目錄
cd ~

# 建立frps資料夾
mkdir -p frps/frps

# 建立frps.ini檔案
cd ~/frps/frps
vi frps.ini

# 在frps.ini設定檔內輸入以下內容
[common]
bind_port = 7000
authentication_method = token
token = [請自行設定token]
# 輸入完後儲存離開

# 切換至frps
cd ~/frps
# 建立docker-compose檔案
vi docker-compose.yml

# 在docker-compose.yml檔案內輸入以下內容:
version: '3'
services:
  frps:
    image: snowdreamtech/frps
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./frps/frps.ini:/etc/frp/frps.ini
# 輸入完後儲存離開
```

經過上述程式碼執行後，在home目錄底下的frps資料夾架構為：
```bash=
frps
├── docker-compose.yml
└── frps
    └── frps.ini
```

接下來啟動dokcer-compose，即可建立frp server：
```bash=
# 切換路徑
cd ~/frps

# 執行docker-compose
docker-compose up -d
```

啟動完後，記得要在雲端主機管理頁面將Port號7000的防火牆打開。

## 五、frp client架設方式

frp client即為我們架設服務在上面，但沒有公網IP的主機。

接下來設定要內網穿透的Win10+WSL2主機(Ubuntu18.04)，在該環境執行以下指令：

```bash=
# 切換至home目錄
cd ~

# 建立frpc資料夾
mkdir -p frpc/frpc

# 建立frpc.ini檔案
cd ~/frpc/frpc
vi frpc.ini

# 在frpc.ini設定檔內輸入以下內容:
# [common]一定要輸入
# 此處我們額外設定MySQL及MongoDB兩個服務做為示範
# 若有其他服務可按需求往下增加
[common]
server_addr = [frp server的IP]
server_port = 7000
authentication_method = token
token = [請輸入frp server設定的token]

[MySQL]
type = tcp
local_ip = 127.0.0.1
local_port = 3306
remote_port = 3306

[MongoDB]
type = tcp
local_ip = 127.0.0.1
local_port = 27017
remote_port = 27017
# 輸入完後儲存離開

# 切換至frpc
cd ~/frpc
# 建立docker-compose檔案
vi docker-compose.yml

# 在docker-compose.yml檔案內輸入以下內容:
version: '3'
services:
  frpc:
    image: snowdreamtech/frpc
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./frpc/frpc.ini:/etc/frp/frpc.ini
# 輸入完後儲存離開
```

經過上述程式碼執行後，在home目錄底下的frpc資料夾架構為：
```bash=
frpc
├── docker-compose.yml
└── frpc
    └── frpc.ini
```

接下來啟動dokcer-compose，即可建立frp client：
```bash=
# 切換路徑
cd ~/frpc

# 執行docker-compose
docker-compose up -d
```

啟動完後，記得要在「雲端主機」(即frp server端)管理頁面將服務對應Port號的防火牆打開(本範例是MySQL 3306和MongoDB 27017)。記得**不是**frp client端這台主機的防火牆。

frp server和frp client設定完後，我們即可以透過雲端主機的公網IP，來使用內網服務。

## 六、結語

本篇文章介紹frp這個軟體，能以反向代理的方式突破內網，讓內網的服務能夠對外開放。並透過Docker來建立frp server和frp client，讓部署能夠更加方便。這篇文章寫到最後，突然想到之前用的[ngrok](https://ngrok.com/)，其實也是反向代理的概念。而frp看起來是一個非常好的替代品，免費而且又可以開很多Port，ngrok僅剩的優勢應該是他有https吧?

