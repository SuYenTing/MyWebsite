---
title: VirtualBox:實體主機以SSH連線至虛擬主機
author: Yen-Ting, Su
date: '2020-12-24'
slug: VirtualBox
categories:
  - Programming
tags:
  - VirtualBox
description: "最近在學習如何架設CentOS虛擬主機在Windows上，因為在虛擬主機內用terminal輸入指令還是比較麻煩，所以想透過SSH方式，直接在Windows主機上進行虛擬主機的操作。"
---

最近在學習如何架設CentOS虛擬主機在Windows上，因為在虛擬主機內用terminal輸入指令還是比較麻煩，所以想透過SSH方式，直接在Windows主機上進行虛擬主機的操作。

# 作業環境：
虛擬機器軟體：Oracle VM VirtualBox-6.1.12
主體作業系統：Windows10
客體作業系統: Linux CentOS7

# Step1：尋找主體(Windows)的IP

在Windows中打開命令提示字元，輸入`ipconfig`，尋找**乙太網路卡 VirtualBox Host-Only Network**區段，紀錄區段內的IP位置。

![](https://i.imgur.com/cKgcadl.png)

以我的電腦為例，主體的IP為192.168.56.1。

# Step2：尋找客體(CentOS7)的IP

打開虛擬機，在terminal中輸入`ifconfig`，尋找IP。

![](https://i.imgur.com/v5wsNlW.png)

以我的為例，客體的IP為10.0.2.15。

另外由於是以SSH連線，所以CentOS這邊需要架立好**SSH服務**，以及**開通SSH的port防火牆**。

# Step3：虛擬主機軟體設定

在確立好主體與客體的IP後，打開VirtualBox，選擇設定。

![](https://i.imgur.com/FaOGI9D.png)

接下來選網路，點選連接埠轉送。 
![](https://i.imgur.com/A0APdoq.png)

新增轉送規則，依序輸入對應資訊，輸入好後按確認即設定完成。
![](https://i.imgur.com/0OprYcm.png)

# Step4：以putty進行連線測試

Step3設定好後，即可開始測試連線，我主要是透過Putty軟體來進行連線。連線時記得還是要開啟虛擬主機。

在Windows上打開Putty，設定如下：

![](https://i.imgur.com/TGrRlGt.png)

輸入完後按下Open即可連入CentOS，接下來輸入登入帳密資訊即可進行操作：

![](https://i.imgur.com/BJzgFnr.png)

# 結語

以上就是我自己在VirtualBox上設定SSH連線的過程，提供給大家參考。

我參考的相關網站資訊：

1. [windows 上 利用ssh連至本機virtualbox 上的ubuntu](https://medium.com/@aaa3air/windows-%E4%B8%8A-%E5%88%A9%E7%94%A8ssh%E9%80%A3%E8%87%B3%E6%9C%AC%E6%A9%9Fvirtualbox-%E4%B8%8A%E7%9A%84ubuntu-81a443a5fcf7)

2. [VirtualBox 實體主機連線至虛擬主機網路設定教學](https://blog.gtwang.org/virtualization/virtualbox-network-connection-between-host-and-guest-tutorial/)

