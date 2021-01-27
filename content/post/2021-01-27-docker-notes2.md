---
title: Docker課程補充筆記2
author: Yen-Ting, Su
date: '2021-01-27'
slug: docker-notes2
categories:
  - Programming
tags:
  - Docker
description: "這是我在tibame學習Docker課程的第二份補充筆記，主要是將老師在上課提到的名詞，蒐集網路上的相關資訊進行閱讀並消化整理出來。希望透過這份筆記，未來在工作時能夠知道別人在說什麼資訊語言，並能讓自己以資訊語言進行溝通。"
---

這是我在tibame學習Docker課程的第二份補充筆記，主要是將老師在上課提到的名詞，蒐集網路上的相關資訊進行閱讀並消化整理出來。希望透過這份筆記，未來在工作時能夠知道別人在說什麼資訊語言，並能讓自己以資訊語言進行溝通。

[第一份Docker課程補充筆記可點此](https://suyenting.github.io/post/docker-notes/)

本篇筆記都是從網路上擷取下來的資訊進行彙整，擷取來源皆有註記，若想要更清楚的資訊可以直接到原網站觀看。

## 一、作業系統概念-恐龍本

![](https://i.imgur.com/6rKGs7h.png)

Operating System Concepts Tenth Edition. Avi Silberschatz · Peter Baer Galvin · Greg Gagne. John Wiley & Sons, Inc. ISBN 978-1-118-06333-0

* [Operating System Concepts 第8版原文PDF檔案](http://www.uobabylon.edu.iq/download/M.S%202013-2014/Operating_System_Concepts,_8th_Edition%5BA4%5D.pdf)

* 此部落格主有提供各章節整理中文筆記
    * [[系列文目錄] 作業系統 Operating System Concepts](https://mropengate.blogspot.com/2017/09/operating-system-concepts.html)

* 書籍購買資訊：目前中文出到第9版，英文已出到第10版
    * [Operating System Concepts, 10/e](https://www.tenlong.com.tw/products/9781119586166)
    * [作業系統概念 9/e](https://www.books.com.tw/products/0010650554?sloc=main)


## 二、負載平衡(Load Balance)

>負載平衡（Load balancing）是一種電腦技術，用來在多個電腦（電腦叢集）、網路連接、CPU、磁碟驅動器或其他資源中分配負載，以達到最佳化資源使用、最大化吞吐率、最小化回應時間、同時避免過載的目的。 
>
>使用帶有負載平衡的多個伺服器組件，取代單一的組件，可以通過冗餘提高可靠性。負載平衡服務通常是由專用軟體和硬體來完成。 
>
>主要作用是將大量作業合理地分攤到多個操作單元上進行執行，用於解決網際網路架構中的高並行和高可用的問題。

(引用擷取自[維基百科: 負載均衡](https://zh.wikipedia.org/zh-tw/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1))

* [Load balance 負載平衡設計](https://blog.niclin.tw/2018/11/18/load-balance-%E8%B2%A0%E8%BC%89%E5%B9%B3%E8%A1%A1%E8%A8%AD%E8%A8%88/)

![](https://i.imgur.com/vjhfAIT.png)
(圖片來源：[大架構的概念與程式設計－－（三）Load Balance](https://akuma1.pixnet.net/blog/post/168326978-%E5%A4%A7%E6%9E%B6%E6%A7%8B%E7%9A%84%E6%A6%82%E5%BF%B5%E8%88%87%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88%EF%BC%8D%EF%BC%8D%EF%BC%88%E4%B8%89%EF%BC%89load-balance))

## 三、三層式架構(3-tier architecture)

* 第一層: 使用者介面層(User Show Layer / UI / Presentation layer)
    * Web
    * 例如Flask
* 第二層: 商業邏輯層(Business Logic Layer)
    * AP(Application)
    * 例如Python
* 第三層: 資料存取層(Data Access Layer)
    * DB(Database)
    * 例如MySQL
    * 資料庫為公司最核心命脈，所以在最後一層

![](https://i.imgur.com/1VZGjyd.png)
(圖片來源：[IBM Knowledge Center - Three-tier architectures](https://www.ibm.com/support/knowledgecenter/zh-tw/SSEQTP_8.5.5/com.ibm.websphere.base.iseries.doc/ae/covr_3-tier.html))

可參考資料:
* [三層結構與 Asp.Net MVC 的簡介](https://shunnien.github.io/2017/07/29/3-tier-and-mvc-introduction/)
* [MVC 三層架構 是什麼? 我只知道三層肉](https://medium.com/@steph.c/%E4%B8%89%E5%B1%A4%E6%9E%B6%E6%A7%8B%E6%98%AF%E4%BB%80%E9%BA%BC-%E6%88%91%E5%8F%AA%E7%9F%A5%E9%81%93%E4%B8%89%E5%B1%A4%E8%82%89-efe542c38aaf)

## 四、設計模式(design pattern)


>在軟體工程中，設計模式（design pattern）是對軟體設計中普遍存在（反覆出現）的各種問題，所提出的解決方案。這個術語是由埃里希·伽瑪（Erich Gamma）等人在1990年代從建築設計領域引入到計算機科學的。
>
>設計模式並不直接用來完成程式碼的編寫，而是描述在各種不同情況下，要怎麼解決問題的一種方案。物件導向設計模式通常以類別或物件來描述其中的關係和相互作用，但不涉及用來完成應用程式的特定類別或物件。設計模式能使不穩定依賴於相對穩定、具體依賴於相對抽象，避免會引起麻煩的緊耦合，以增強軟體設計面對並適應變化的能力。
>
>並非所有的軟體模式都是設計模式，設計模式特指軟體「設計」層次上的問題。還有其他非設計模式的模式，如架構模式。同時，演算法不能算是一種設計模式，因為演算法主要是用來解決計算上的問題，而非設計上的問題。

(引用擷取自[維基百科: 設計模式](https://zh.wikipedia.org/wiki/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F_(%E8%AE%A1%E7%AE%97%E6%9C%BA)))

* 老師推薦書籍: [大話設計模式](https://www.tenlong.com.tw/products/9789866761799)

![](https://i.imgur.com/GM3uHrX.png)

## 五、單元測試(unit test)

* [維基百科: 單元測試](https://zh.wikipedia.org/wiki/%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95)
* [一次搞懂單元測試、整合測試、端對端測試之間的差異](https://blog.miniasp.com/post/2019/02/18/Unit-testing-Integration-testing-e2e-testing)
    * 單元測試 (Unit testing)：以程式碼的最小單位進行測試，保護程式邏輯不會在系統維護的過程中遭到破壞，也進一步確保維護中的程式碼品質。
    * 整合測試 (Integration testing)：整合多方資源進行測試，確保模組與模組之間的互動行為正確無誤，也讓不同模組在各自開發維護的過程中不會因為功能調整而遭到破壞。
    * 端對端測試 (E2E testing)：所謂的「端對端」(E2E) 是指從使用者的角度出發(一端)，對真實系統(另一端)進行測試。
(以上說明擷取自：[一次搞懂單元測試、整合測試、端對端測試之間的差異](https://blog.miniasp.com/post/2019/02/18/Unit-testing-Integration-testing-e2e-testing))

## 六、部署策略

* [[好文分享]應用部署的六種策略](https://blog.marsen.me/2018/01/07/2018/six_strategies_for_application_deployment/)
    * 重建部署(Recreate)：版本A下線後版本B上線
    * 滾動部署（滾動更新或者增量釋出）(Ramped, rolling-update or incremental)：版本B緩慢更新並替代版本A
    * 藍綠部署(Blue/Green)：版本B並行與版本A釋出，然後流量切換到版本B
    * 金絲雀部署(Canary)：版本B向一部分使用者釋出，然後完全放開
    * A/B部署(A/B testing)：版本B只向特定條件的使用者釋出
    * 影子部署(Shadow)：版本B接受真實的流量請求，但是不產生響應
    (以上說明擷取自：[[好文分享]應用部署的六種策略](https://blog.marsen.me/2018/01/07/2018/six_strategies_for_application_deployment/))

## 七、Python連接AWS套件(Boto3套件)

* [適用於 Python 的 AWS 開發套件 (Boto3)](https://aws.amazon.com/tw/sdk-for-python/)

> 透過適用於 Python 的 AWS 開發套件 (boto3) 可快速開始使用 AWS。Boto3 可以輕鬆將 Python 應用程式、程式庫或指令碼與 AWS 服務進行整合，包括 Amazon S3、Amazon EC2 和 Amazon DynamoDB 等。

* [Boto3套件官方說明手冊](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html)

## 八、基礎架構程式碼(IaC)

* [趨勢科技-甚麼是基礎架構即代碼（Infrastructure as Code）？](https://www.trendmicro.com/zh_hk/what-is/cloud-security/infrastructure-as-code.html)
* [AWS 雲端開發套件](https://aws.amazon.com/tw/cdk/)
* [AWS CloudFormation](https://aws.amazon.com/tw/cloudformation/)
* AWS IaC說明宣傳影片：透過代碼即可控制系統架構
{%youtube Omppm_YUG2g %}
* 傳統Data Center影片：管理機器、調整系統架構時需要到機房進行操作
{%youtube PVubvo6H8Oc %}

## 九、三種雲服務模式

* [一文讀懂 IaaS、PaaS、SaaS 三種雲服務模式](https://kknews.cc/zh-tw/tech/vl8g8my.html)
    * IaaS(Infrastructure-as-a-Service)：基礎設施即服務
    * PaaS(Platform-as-a-Service)：平台即服務
    * Saas(Software-as-a-Service)：軟體即服務

* [IaaS & PaaS & SaaS 三兄弟](https://medium.com/@stfk1105/iaas-paas-saas-%E4%B8%89%E5%85%84%E5%BC%9F-c745dfa0cfd4)：此篇文章有良好的舉例可供參考

![](https://i.imgur.com/yPfCsAr.png)
(圖片來源：[alibabacloud：What is the difference between IaaS, PaaS and SaaS?](https://www.alibabacloud.com/tc/knowledge/difference-between-iaas-paas-saas))

## 十、HTTPS

* [維基百科: 超級文字傳輸安全協定](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)
* [一文搞懂 HTTP 和 HTTPS 是什麼？兩者有什麼差別](https://tw.alphacamp.co/blog/http-https-difference)


>SSL 的全名是 Secure Sockets Layer，即安全通訊端層，簡而言之，這是一種標準的技術，用於保持網際網路連線安全以及防止在兩個系統之間發送的所有敏感資料被罪犯讀取及修改任何傳輸的資訊，包括潛在的個人詳細資料。兩個系統可以是伺服器與用戶端 (例如購物網站與瀏覽器)，或者伺服器至伺服器 (例如，含有個人身份資訊或含有薪資資訊的應用程式)。 
>這樣做是為了確保使用者與網站、或兩個系統之間傳輸的任何資料保持無法被讀取的狀態。此技術可使用加密演算法以混淆輸送中的資料，防止駭客在資料透過連線發送時讀取資料。此資訊可能是任何敏感或個人資訊，包括信用卡號與其他財務資訊、姓名與地址。 
>
>TSL (Transport Layer Security，傳輸層安全性) 是更新、更安全的 SSL 版本。我們仍將安全性憑證稱為 SSL，因為這是較常用的詞彙，不過當您透過DigiCert購買 SSL 時，您所購買的其實是最新的 TLS 憑證及 ECC、RSA 或 DSA 的加密選項。
>
>HTTPS (Hyper Text Transfer Protocol Secure，超級文字傳輸協議安全) 會在網站受到 SSL 憑證保護時在網址中出現。該憑證的詳細資料包括發行機構與網站擁有人的企業名稱，可以透過按一下瀏覽器列上的鎖定標記進行檢視。

(引用出處：[什麼是 SSL、TLS 以及 HTTPS？](https://www.websecurity.digicert.com/zh/tw/security-topics/what-is-ssl-tls-https))

![](https://i.imgur.com/xkZ2ISM.png)
(圖片來源：[帝濤發展及策劃 - HTTPS 加密連線SSL 證書](https://www.rodpm.com/webinar/https-%E5%8A%A0%E5%AF%86%E9%80%A3%E7%B7%9Assl-%E8%AD%89%E6%9B%B8/))

## 十一、網域名domain

> 網域名稱（英語：Domain Name，簡稱：Domain），簡稱域名、網域，是由一串用點分隔的字元組成的網際網路上某一台電腦或電腦組的名稱，用於在資料傳輸時標識電腦的電子方位。域名可以說是一個IP位址的代稱，目的是為了便於記憶後者。例如，wikipedia.org是一個域名，。人們可以直接存取wikipedia.org來代替IP位址，然後域名系統（DNS）就會將它轉化成便於機器辨識的IP位址。這樣，人們只需要記憶wikipedia.org這一串帶有特殊含義的字元，而不需要記憶沒有含義的數字

(引用擷取自：[維基百科：域名](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D))

![](https://i.imgur.com/rJC4f8b.png)

(圖片來源：[鵠崙設計 - 什麼是 URL 網址 IP ？ 網域 Domain 中文 意思是什麼？](https://www.design-hu.com/web-news/domain.html))

## 十二、Kubernetes(K8S)

![](https://i.imgur.com/AeYDD13.png)

* [Kubernetes官方網站](https://kubernetes.io/)
* [維基百科：Kubernetes](https://zh.wikipedia.org/wiki/Kubernetes)
* [Kubernetes 30天學習筆記](https://ithelp.ithome.com.tw/articles/10192401)

> Kubernetes 是一個協助我們自動化部署、擴張以及管理容器應用程式(containerized applications)的系統。相較於需要手動部署每個容器化應用程式(containers)到每台機器上，Kubernetes 可以幫我們做到以下幾件事情：
>
>* 同時部署多個 containers 到一台機器上，甚至多台機器。
>* 管理各個 container 的狀態。如果提供某個服務的 container 不小心 crash 了，Kubernetes 會偵測到並重啟這個 container，確保持續提供服務
>* 將一台機器上所有的 containers 轉移到另外一台機器上。
>* 提供機器高度擴張性。Kubernetes cluster 可以從一台機器，延展到多台機器共同運行。

(引用擷取自：[Kubernetes 30天學習筆記](https://ithelp.ithome.com.tw/articles/10192401))

## 十三、Docker三劍客

* [Docker三剑客：Compose、Machine和Swarm](https://blog.csdn.net/wfs1994/article/details/80601027)

| Docker Compose | Docker Machine | Docker Swarm |
| -------- | -------- | -------- |
|單主機多容器|跨多主機裝Docker|多主機多容器|
| ![](https://i.imgur.com/iG8RLg3.png =x300)| ![](https://i.imgur.com/mqMh5WY.png =x300)| ![](https://i.imgur.com/PIPNh7f.png =x300)|

* Docker Compose架構圖
![](https://i.imgur.com/0HzVWjG.png)
(圖片來源: [Containerized Python Development – Part 2](https://www.docker.com/blog/containerized-python-development-part-2/))

> 容器就像積木一樣，透過各種軟體積木組裝成系統架構，一旦使用多個容器時，就面臨到協同調度（Orchestration）的問題，例如部署一個Web服務容器和DB服務容器，同時需要將它們連接起來，且必須先啟動資料庫再執行Web伺服器，而Docker Compose便是針對單機上控制多容器所發展的管理工具。

(引用擷取自：[網管人 - 部署Docker Compose　實例示範定義檔撰寫](https://www.netadmin.com.tw/netadmin/zh-tw/technology/5324EFB207BF41A19C923CB0C0A8E448))

* Docker Machine架構圖
![](https://i.imgur.com/Ppefgub.png)
(圖片來源: [Docker Machine to control Docker Hosts on Google Cloud](https://rominirani.com/docker-machine-to-control-docker-hosts-on-google-cloud-3a48b46809dc))

> Docker Machine的用途就是在各種虛擬化平台（VMware vSphere、Microsoft Hyper-V、Oracle VirtualBox等）上，迅速建立已內建Docker Engine的虛擬機，只需「docker-machine create」一行指令並修改其參數，就能適用所有虛擬化平台和雲端服務，無須學習各種雲端平台和虛擬平台API，節省建立虛擬機和安裝Docker Engine的時間。

(引用擷取自：[網管人 - 探索Docker Machine　建立虛擬機下達管理指令](https://www.netadmin.com.tw/netadmin/zh-tw/technology/108CA5A2475F4AFD9064F8ACF5E9103B))

* Docker Swarm架構圖
![](https://i.imgur.com/ROTLvx8.png)
(圖片來源: [Create Cluster using docker swarm](https://medium.com/tech-tajawal/create-cluster-using-docker-swarm-94d7b2a10c43))

>簡單來說，當企業和組織的IT管理人員要使用容器技術時，只要透過Docker容器管理技術即可使用。然而，隨著企業和組織使用容器的數量逐漸變多時，便需要有一個能夠管理及調度眾多容器的平台，而Docker Swarm就是Docker公司所推出原生的容器調度管理平台。

(引用擷取自：[網管人 - Docker Swarm容器管理　規畫部署一次傾囊相授](https://www.netadmin.com.tw/netadmin/zh-tw/feature/167CDFB3615E42229B5C7053DC452755))


