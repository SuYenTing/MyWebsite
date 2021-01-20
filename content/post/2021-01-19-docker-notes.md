---
title: Docker課程補充筆記
author: Yen-Ting, Su
date: '2021-01-19'
slug: docker-notes
categories:
  - Programming
tags:
  - Docker
description: "這是我在tibame學習Docker課程的補充筆記，本堂課由周廷諺老師授課。不得不說老師上課講解的非常好，除了講解Docker的指令外，也講解DevOps及雲端的觀念，讓我們對於資訊業產品在運作的過程中有更清楚的認識。之前有曾經稍微玩過Docker，但對於Docker還是懵懵懂懂，今日上完課後感覺有種任督二脈被打通的感覺！"
---

這是我在tibame學習Docker課程的補充筆記，本堂課由周廷諺老師授課。不得不說老師上課講解的非常好，除了講解Docker的指令外，也講解DevOps及雲端的觀念，讓我們對於資訊業產品在運作的過程中有更清楚的認識。之前有曾經稍微玩過Docker，但對於Docker還是懵懵懂懂，今日上完課後感覺有種任督二脈被打通的感覺！

這堂課在環境建置部分，是在Windows10作業系統下，以VMware架設Linux CentOS7虛擬環境，並在虛擬環境上執行Docker部署。由於老師已經在他的講義中提供程式碼，所以這篇文章就沒有另外列出來。本篇文章主要是整理老師上課補充且講義未紀錄到的部份。


## 一、課前準備
* [周廷諺老師上課講義連結](https://docs.google.com/document/d/1L5I7TuhAvHuXqAuW7fhmpIpcWBNfdVL61kefiCwzTJU/edit)

* [Docker課程論壇-AI / Big Data 資料分析師 - 超實用作業區](https://2311.cxcxc.info/viewforum.php?f=20)

* [Docker Hub](https://hub.docker.com/)

* [VMware下載位置連結](https://www.vmware.com/tw/products/workstation-pro/workstation-pro-evaluation.html)
點選`Windows 適用的 Workstation 16 Pro`按鈕下載

* [CentOS7映像檔下載連結](http://centos.cs.nctu.edu.tw/7.9.2009/isos/x86_64/)
選擇`CentOS-7-x86_64-DVD-2009.iso`檔案

* 新聞補充：[DDoS勒索不成，駭客最後讓一家靠雲端的公司關門大吉：Code Spaces的血淋淋教訓！](https://www.ithome.com.tw/news/88797)

---
## 二、DevOps
![](https://i.imgur.com/8Ef8RFY.png)

圖片來源文章：[DevOps is a culture, not a role!](https://neonrocket.medium.com/devops-is-a-culture-not-a-role-be1bed149b0)

---
## 三、雲原生(Cloud-Native)
![](https://i.imgur.com/QZOZJnS.png)

圖片來源文章：[Cloud Native Applications — The Why, The What & The How.](https://medium.com/velotio-perspectives/cloud-native-applications-the-why-the-what-the-how-9b2d31897496)

---
## 四、CI/CD
[CI/CD：維基百科](https://zh.wikipedia.org/wiki/CI/CD)

* CI (Continuous Integration) 持續整合
* CD (Continuous Deployment) 持續部署

概念參考文章：[CI/CD (持續性整合 / 部署) - 因為懶，所以更要 CI/CD！概念講解！](https://blog.kennycoder.io/2020/04/07/CI-CD-%E6%8C%81%E7%BA%8C%E6%80%A7%E6%95%B4%E5%90%88-%E9%83%A8%E7%BD%B2-%E5%9B%A0%E7%82%BA%E6%87%B6%EF%BC%8C%E6%89%80%E4%BB%A5%E6%9B%B4%E8%A6%81CI-CD%EF%BC%81%E6%A6%82%E5%BF%B5%E8%AC%9B%E8%A7%A3%EF%BC%81/)

---
## 五、VMware Station網路介接模式

![](https://i.imgur.com/7Kerz29.png)
圖片來自VMware建立虛擬環境設定

* Bridge
手動設定IP，但是可讓VM的網路跟實體主機一樣

* NAT
透過DHCP自動取得IP，快速上網模式不需太多設定

* Host-only
只有自己的主機才能夠連上

對網路設定有興趣，可以參考TCP/IP相關文章：
[鳥哥的 Linux 私房菜-第二章、基礎網路概念](http://linux.vbird.org/linux_server/0110network_basic.php)

---
## 六、網路時間協定(NTP)

參考文章：

* [網路時間協定：維基百科](https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E6%99%82%E9%96%93%E5%8D%94%E5%AE%9A)

* [鳥哥的 Linux 私房菜-第十五章、時間伺服器： NTP 伺服器](http://linux.vbird.org/linux_server/0440ntp.php)

---
## 七、Linux init指令

以下指令在terminal上直接輸入即可執行：

* init 0: 停機
* init 1: 單使用者形式,只root進行維護
* init 2: 多使用者,不能使用net file system
* init 3: 完全多使用者
* init 5: 圖形化
* init 4: 安全模式
* init 6: 重啟

指令參考頁面：[linux啟動級別的含義(init 0-6)](https://www.itread01.com/p/175241.html)

---
## 八、虛擬環境複製

記得虛擬機器要先關機才進行複製

![](https://i.imgur.com/USlxEFs.png)

![](https://i.imgur.com/sxqS8Gp.png)

![](https://i.imgur.com/BIo2ujV.png)

![](https://i.imgur.com/sVnSLzk.png)

![](https://i.imgur.com/5MfsoJZ.png)

![](https://i.imgur.com/jnh8CXS.png)

![](https://i.imgur.com/qSAIdJM.png)

---
## 九、將虛擬機器放置在背景運行

![](https://i.imgur.com/L59myzm.png)


---
## 十、MobaXterm連線方式

![](https://i.imgur.com/0bRlIjt.png)

![](https://i.imgur.com/JhqpggB.png)


---
## 十一、讓sudo指令不用輸入密碼做法

在terminal上輸入指令：

```
sudo visudo
```

會打開vi文件：

107行註解掉，110行加入註解

![](https://i.imgur.com/1fZ0en6.png)

接下來輸入`:x`儲存vi文件並離開，這樣即設定完成

之後該名具有管理權限的使用者在Linux上輸入`sudo`指令時就不用再輸入密碼。

---
## 十二、AWS雲端服務介紹

AWS官方說明：

* [AWS Cloud9：用來撰寫、執行和偵錯程式碼的雲端 IDE](https://aws.amazon.com/tw/cloud9/)

* [AWS Lambda-無伺服器運算：執行程式碼更輕鬆，不必考慮伺服器選項。只需為使用的運算時間支付費用。](https://aws.amazon.com/tw/lambda/)

無伺服器運算應用範例：[無伺服器案例介紹：可口可樂的Serverless之旅](https://www.ithome.com.tw/news/112431)

---
## 十三、docker run的--link參數不要使用(官方建議)

參考來源：[Docker官方手冊-Legacy container links](https://docs.docker.com/network/links/)

![](https://i.imgur.com/6ECNIGn.png)

--link是Docker的舊功能，且有安全性的問題，未來可能會被棄用

可改用--network參數取代

