---
title: Redis資料庫安裝筆記
author: Yen-Ting, Su
date: '2021-02-09'
slug: install-redis-db
categories:
  - Programming
tags:
  - Redis
  - Database
description: "本篇文章說明如何在VMware虛擬機(主體為Windows作業系統)的CentOS7作業系統上，安裝Redis資料庫。並可以在Windows作業系統上以Python連入。此處提供兩種安裝方法，第一個是直接在虛擬機安裝，第二個是透過Docker來安裝。"
---

* [Redis官網](https://redis.io/)

![](https://i.imgur.com/IyqcjHC.png)

* [DB-Engines Ranking - 資料庫排名](https://db-engines.com/en/ranking)

![](https://i.imgur.com/WfDjbde.png)

# 安裝環境說明
* 虛擬機器軟體：VMware Workstation 16 Pro
* 主體作業系統：Windows 10
* 客體作業系統：CentOS7

Redis官網只提供在Linux作業系統上安裝的方式，故此篇文章是在CentOS7作業系統上進行安裝。

# 一、在CentOS7安裝Python3

CentOS7本身自帶Python2程式，需要另行安裝Python3

我參考的安裝說明: [【Python】在 CentOS 7 上安裝 Python3](https://kirin.idv.tw/python-install-python3-in-centos7/)

```bash=
# 安裝python3.6版本
sudo yum install python36

# 升級pip套件 
pip3 install --upgrade pip
# 提醒:記得是pip3 若為pip會裝成python2的套件

# 輸入Python3後即會進到Python環境
python3
# 提醒:一定要輸入python3 若輸入python會進到python2.7環境

# 若要離開Python環境回到terminal環境則輸入 `quit()`
```

# 二、安裝與連線Redis資料庫

* [redis官網提供的安裝指令](https://redis.io/download)

除了在Linux作業系統本機上直接安裝Redis資料庫外，也可利用Docker來安裝

接下來將分別說明**本機直接安裝**和**Docker安裝**

可自行挑選任一安裝方式


## 1. 在Linux本機直接安裝Redis資料庫

* 此處參考的安裝說明: [【Redis】CentOS7下安装Redis服务](https://segmentfault.com/a/1190000023092469)


### Step1. 安裝gcc9

在本機安裝需編譯Redis資料庫，故需要安裝gcc，此處安裝gcc第9版

```bash=
# 首先在CentOS7上安裝gcc
sudo yum -y install centos-release-scl
sudo yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

# scl命令啟用僅在當前shell有效 退出shell後重進會回復成原本設定(即沒有gcc)
scl enable devtoolset-9 bash

# 查看gcc版本
gcc --version
```

### Step2. 安裝Redis
```bash=
# 取得Redis安裝壓縮檔
wget https://download.redis.io/releases/redis-6.0.10.tar.gz

# 解壓縮
tar xzf redis-6.0.10.tar.gz

# 切換當redis安裝資料夾路徑
cd redis-6.0.10

# 執行編譯
make
```

安裝成功畫面如下圖所示：
![](https://i.imgur.com/E4xznhD.png)

### Step3. 執行Redis server

接下來執行redis server
```bash=
# 執行redis server
src/redis-server
```

成功執行redis server畫面如下：
![](https://i.imgur.com/jPf6EnY.png)

執行後畫面會被server綁定，此時需要開新的terminal進行連線

若要終止server運作，可直接按`ctrl+c`


### Step4. 連線Redis server

```bash=
# 要記得先切換到redis-6.0.10資料夾底下
cd redis-6.0.10

# 執行連線
src/redis-cli
```

### Step5. 測試Redis指令

```bash=
# 依據官網提供測試程式碼 在Redis環境內執行
set foo bar
get foo
```

執行結果如下：
![](https://i.imgur.com/ONSlKan.png)

### Step6. 在Python內執行Redis

我參考的說明：[Python redis 使用介绍](https://www.runoob.com/w3cnote/python-redis-intro.html)

若要在CentOS7上執行Python3，直接在terminal上輸入`python3`，注意這邊不能輸入`python`，若輸入`python`會直接進入到`python2`環境

```bash=
# 首先安裝python的redis套件
sudo pip3 install redis

# 執行Python3
python3
```

進到python3環境後，輸入以下指令進行測試：
```python=
# 載入套件
import redis

# 連入redis資料庫
r = redis.StrictRedis(host='localhost', port=6379, db=0)

# redis指令
r.set('foo', 'bar')
r.get('foo')
```

執行成果：
![](https://i.imgur.com/kObBXRi.png)

### 補充: Windows主機直接連線Redis

在虛擬主機上啟動Redis Server後，也可以從Windows的Python連入虛擬機內的Redis資料庫進行操作，此處需要先打開CentOS7防火牆的Port。

在CentOS7執行以下指令：
```bash=
# 防火牆永久開啟6379的Port  6379為Redis資料庫服務的端口
sudo firewall-cmd --zone=public --permanent --add-port=6379/tcp
```

設定好防火牆後，重新啟動Redis伺服器，此處需要多加`--protected-mode`引數，要關閉保護模式才能從Windows連入：
```bash=
src/redis-server --protected-mode no
```

此外，還需要設定VMware的Port Forwarding，首先點選Edit->Virtual Network Editor：

![](https://i.imgur.com/Mytm89O.png)

啟用管理員權限，這樣才能修改設定：
![](https://i.imgur.com/jsrJW2L.png)

選擇VMnet8，然後點選NAT Settings：
![](https://i.imgur.com/0d7ZjjC.png)

點選Add：
![](https://i.imgur.com/VGxTDgy.png)

輸入相關設定，紅框內的IP需依自行的虛擬主機調整：
![](https://i.imgur.com/LucZRil.png)

設定好後一路按ok結束。

接下來打開Windows的PyCharm，安裝Redis套件(`pip install redis`)，接下來輸入以下程式碼即可順利連接。

```python=
import redis

# 連入redis資料庫
r = redis.StrictRedis(host='192.168.37.129', port=6379, db=0)
# 此處需要自行調整host引數值，host為虛擬機的IP，可在linux輸入ifconfig查詢

# redis指令
r.set('foo', 'bar')
r.get('foo')
```

## 2. 用Docker安裝Redis資料庫

* [Docker Hub - redis官方image說明](https://hub.docker.com/_/redis)

* 我參考的安裝說明: [[Redis] 使用 Docker 安裝 Redis](https://marcus116.blogspot.com/2019/02/how-to-run-redis-in-docker.html)

### Step1. 安裝CentOS的Docker套件

此處參考周廷諺老師的Docker講義

```bash=
# 安裝yum管理工具套件
sudo yum install -y yum-utils

# 透過yum管理工具導入docker的repo來源
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 安裝docker主程式及所需套件
sudo yum install docker-ce docker-ce-cli containerd.io -y

# 啟動docker
sudo systemctl start docker

# 自動常駐/啟動docker
sudo systemctl enable docker

# 檢查docker是否啟用並常駐
sudo systemctl status docker

# 將當前使用者加入docker群組
sudo usermod -aG docker $USER

# 登出目前使用者
exit

# 重新登入後即可使用docker
```

### Step2. 啟動Docker Redis容器
```bash=
# 建立Redis資料夾
mkdir redis

# 切換至Redis
cd redis

# 搜尋Redis image
docker search redis

# 下載Redis image
docker pull redis

# 創建並啟動容器 
docker run \
--name redis-lab \
-p 6379:6379 \
-d \
-v $(pwd)/data:/data \
--restart unless-stopped \
redis

# docker run參數說明
# --name: 容器名稱
# -p: 主機port:容器port
# -d: 背景執行
# -v: 主機資料夾:容器資料夾
# --restart: unless-stopped表示除非被手動停止 否則系統重新啟動時該容器即會自動重啟
# 關於--restart引數可參考: https://docs.docker.com/config/containers/start-containers-automatically/
```

### Step3. 在Python內執行redis

進到python3環境後，輸入以下指令進行測試：

```python=
# 載入套件
import redis

# 連入redis資料庫
r = redis.StrictRedis(host='localhost', port=6379, db=0)

# redis指令
r.set('foo', 'bar')
r.get('foo')
```

### 補充: Windows主機直接連線Redis

在虛擬主機上部署好Docker的Redis Server後，除了在CentOS7的terminal上直接執行外，也可直接在Windows上的Python執行。

Docker部署時會略過防火牆限制，所以不用自行設定防火牆。

此外，還需要設定VMware的Port Forwarding，首先點選Edit->Virtual Network Editor：

![](https://i.imgur.com/Mytm89O.png)

啟用管理員權限，這樣才能修改設定：
![](https://i.imgur.com/jsrJW2L.png)

選擇VMnet8，然後點選NAT Settings：
![](https://i.imgur.com/0d7ZjjC.png)

點選Add：
![](https://i.imgur.com/VGxTDgy.png)

輸入相關設定，紅框內的IP需依自行的虛擬主機調整：
![](https://i.imgur.com/LucZRil.png)

設定好後一路按ok結束。

接下來打開Windows的PyCharm，安裝Redis套件(`pip install redis`)，接下來輸入以下程式碼即可順利連接。

打開PyCharm，安裝redis套件(`pip install redis`)，，接下來輸入以下程式碼即可順利連接。

```python=
import redis

# 連入redis資料庫
r = redis.StrictRedis(host='192.168.37.129', port=6379, db=0)
# 此處需要自行調整host引數值，host為虛擬機的IP，可在linux輸入ifconfig查詢

# redis指令
r.set('foo', 'bar')
r.get('foo')
```



