---
title: Shopee Code League 2021競賽心得
author: Yen-Ting, Su
date: '2021-03-25'
slug: shopee-competition-2021
categories:
  - Programming
tags:
  - Python
description: "蝦皮在2021年3月舉辦一場為期三週的數據競賽，開放給台灣及東南亞各國在學生與在職人士參與。競賽共分成三個項目：數據分析、數據科學及演算法相關問題，每週末各比一個項目。這次我組隊參加其中兩個項目：數據分析及數據科學競賽，這篇文章將和大家分享我此次的參賽心得。"
---

## 一、前言

蝦皮在2021年3月舉辦一場為期三週的數據競賽，開放給台灣及東南亞各國在學生與在職人士參與。競賽共分成三個項目：數據分析、數據科學及演算法相關問題，每週末各比一個項目。這次我組隊參加其中兩個項目：數據分析及數據科學競賽，這篇文章將和大家分享我此次的參賽心得。

競賽成果部份，數據分析競賽項目(Multi-Channel Contacts)在規定的三小時內，我們組別雖然沒有做出來，但事後有做出來並提交到Kaggle上，並拿到最高分，代表資料整理後的結果有符合官方要求。

在數據科學競賽項目(Address Elements Extraction)的Public Leaderboard是208名，準確度分數為0.49206。最後在Private Leaderboard的成果是第206名，準確度分數為0.50177，第一名是0.70151。總參賽隊伍數為1,034隊，Top排名比率為20.12%。

* [Shopee Code League 2021競賽官方網站](https://careers.shopee.sg/codeleague/)
![](https://i.imgur.com/NkPjaWk.png)

## 二、參賽目的

一直以來大家都說要多參加數據競賽，如果有得到名次的話，找工作會有加分。因為前一份工作較為忙碌，所以我也沒有花時間去打比賽。而最近在轉職期間，剛好看到蝦皮在宣傳這個競賽，想說試試看水溫，看看自己的能力在哪。順便藉由實務題目，在實作過程中來吸收新知。雖然這次參賽結果沒有很好的表現，但從收穫方面來看，明顯有感受到自己的成長。

這次和我一同參賽的隊友有3位，其中2位是我碩士班的學弟：冠廷和銘翔，一位目前在金融業當數據分析師，另一位剛畢業當完兵正要找工作。最後一位是我學弟在其他數據分析競賽認識的冠豪，目前在遊戲公司當數據分析師。我們隊伍4個人剛好都是財金背景出身，但對於資料科學領域都很有興趣(我們常笑說自己生涯路走歪了)。

## 三、數據分析競賽 - Multi-Channel Contacts

數據分析競賽是在星期六下午舉辦，競賽時間只有3個小時。參賽隊伍需要在這3個小時內，透過程式整理數據，符合官方要求的結果。簡單來說，就是看你資料整理的能力和正確性。

詳細競賽資訊可以參考Kaggle的競賽頁面：

* [Kaggle競賽頁面 - Shopee Code League - Address Elements Extraction](https://www.kaggle.com/c/scl-2021-da)

### 1. 題目說明

這次競賽是要整理蝦皮客服的資料，由於每位使用者可能透過不同的管道，例如電話、Email、線上客服等，來尋求客服的幫助。每當客戶以不同的管道聯繫，便會形成一個新的服務事件(ticket)，這有可能導致同一個客戶的問題，卻被拆分成好幾個ticket。為能夠了解客戶對於問題的真正聯絡次數，需要透過資料整理，於是便有此次的題目。

接下來說明競賽的資料內容與要求，下圖為擷取自蝦皮提供競賽的說明簡報：

![](https://i.imgur.com/uApmztO.png)

每筆資料有以下資訊：
* Id：蝦皮客服ticket的編號
* Email：使用者的信箱資訊
* Phone：使用者的電話資訊
* Order ID：使用者在蝦皮交易的訂單ID
* Contacts：使用者在此ticket聯絡的次數

上圖中共有4筆tickets，只要tickets之間的Email、Phone、OrderID，這三個欄位只要有其中一個相同，即視為是同一個使用者。所以上圖中4筆tickets，皆被視為是同一使用者。在辨識出來後，需要整理成蝦皮需要的格式，將每筆ticket的Id以'-'相連，最後用','隔出總聯絡次數，即4筆tickets的Contacts相加值(以圖中為例: 5+2+4+3=14)。

蝦皮提供的資料為Json格式，經過Python Pandas讀取後，資料筆數共計有500,000筆。

### 2. 競賽成果

由於我們並未在競賽規定的三小時內繳交結果，所以此處我用自己另外一個帳號提交。提交結果的得分為0.95326，而第一名的分數也是0.95326，所以代表我程式碼整理的結果是沒有問題的。

![](https://i.imgur.com/qKWIlPA.png)

### 3. 我的解法

這個題目的難點是要進行大量比對及歸類，我後來是參考別人實作出來的建議，以深度優先搜尋演算法(Depth-First Search; DFS)來解題。

我主要參考這篇文章：[How to implement depth-first search in Python](https://www.educative.io/edpresso/how-to-implement-depth-first-search-in-python)，透過Python的遞迴函式(Recursion)實作DFS寫法。我將程式碼稍微改寫，以符合我的需求。

針對蝦皮的問題，主要的解法步驟大致如下：
1. 對Email、Phone及Order ID這3個欄位，各自利用Group by先找出相同使用者的tickets，並儲存為Python的Set型別，再整合成Dict型別。
2. 以DFS演算法掃過Dicts，整理出相同使用者的tickets。
3. 依據使用者的tickets，計算每個使用者的總聯絡次數。
4. 整理出官方要求提供的結果格式。

我撰寫的程式碼已放在Github上，可以[[點選這裡]](https://github.com/SuYenTing/Shopee-Code-League-2021/blob/main/shopee_multi_channel_contacts.ipynb)觀看，整理的速度算非常快。

### 4. 更好的解法

我自己也好奇別人的寫法，Kaggle上面有蠻多人分享的，最讓我驚艷的應該是這篇：

[Shopee Code League - Multi-Channel Contacts: Multi-Channel Contacts - 0.953 & 15 sec](https://www.kaggle.com/yeogaa/multi-channel-contacts-0-953-15-sec)

這篇沒用到DFS演算法，只是單純利用Python的Dict型別來整理資料。但為何可以這麼快?我跟隊友討論時，找到一篇文章：

[Stack Overflow - Python dictionary vs list, which is faster?
](https://stackoverflow.com/questions/38927794/python-dictionary-vs-list-which-is-faster/38927968)

擷取自該篇文章回答內容：

> In Python, the average time complexity of a dictionary key lookup is O(1), since they are implemented as hash tables. The time complexity of lookup in a list is O(n) on average.

簡單來說，由於Python在儲存Dict型別的Keys是以Hash table方式儲存，所以搜尋Keys，不用一個一個找，只要丟到Hash Function內馬上就可以找到對應位置，所以速度會比list型別一個一個找還要快。

這篇文章有記載Python每個型別在不同的操作時，所需要的時間複雜度資訊：[Wiki Python - TimeComplexity](https://wiki.python.org/moin/TimeComplexity)，以後若有加快程式執行速度時，可以來參考這篇文章。

## 四、數據科學競賽 - Address Elements Extraction

本屆蝦皮數據科學競賽的的題目是「Address Elements Extraction」，競賽期間為一週。

詳細資訊可參考Kaggle競賽頁面：

* [Kaggle競賽頁面 - Shopee Code League - Address Elements Extraction](https://www.kaggle.com/c/scl-2021-ds/code)

### 1. 題目說明

競賽內容是從一堆印尼文的地址中，找出興趣點(Point of Interest; POI)和街道(street)。

以下是蝦皮在Kaggle上提供的資料範例：

![](https://i.imgur.com/Dc6eYwh.png)

我們要從原始地址(raw_address)欄位中，建立出預測模型，找出興趣點(POI)和街道(street)的資訊。這些地址是蝦皮客戶填寫的，而興趣點和街道則是官方標註。透過這個預測模型，可以增進地址辨識，提高貨物運輸的效率。

訓練集資料共有300,000筆，測試集資料共有50,000筆，其中30%做為Public Leaderboard，70%做為Private Leaderboard。

準確度計算方式為預測對的樣本數除上總樣本數，預測對的定義是興趣點(POI)和街道(street)的預測值和實際值皆要相同，若只預測其中一個對不算分，算是蠻嚴苛的。

這個題目的難點在於，每個人填寫地址時可能會用到很多種簡寫方式，或者是不小心寫錯字。所以同一個興趣點或街道，可能會有不同的寫法。例如說我想寄信到總統府，我可以寫「臺北市中正區重慶南路1段122號」、「台北重慶南路1段122號」、也可以直接寫個「總統府」三個字，只要郵差看得懂，信就可以寄達。當然，郵差在看最後兩個的地址寫法時，就要多花時間去處理這封郵件，這就是模型希望能夠改善的部分。

從上述的題目內容，可以將這個題目歸類為NLP問題，坦白說這不是我的強項。我習慣做的是用很多特徵去預測迴歸或分類問題。在NLP問題部份，我自己是有去看過一些文章，了解近期的發展脈絡，但沒有實作過的經驗。

### 2. 競賽成果

在競賽成果部份，我們組別在Public Leaderboard的成果是208名，準確度分數為0.49206。最後在Private Leaderboard的成果是第206名，準確度分數為0.50177，總參賽隊伍數為1,034隊，Top排名比率為20.12%。
![](https://i.imgur.com/50QRACx.png)

### 3. 資料探索式分析

在競賽題目與資料公布後，首先要做的是對資料做探索式分析，了解資料的長相，並且思考該如何處理，並應該用什麼模型來解決問題。

我對競賽資料進行探索式分析的程式碼，已放在Github上，可以[[點選這裡]](https://github.com/SuYenTing/Shopee-Code-League-2021/blob/main/shopee_address_elements_extraction(EDA).ipynb)觀看。

我大概觀察出以下成果：

* 預測的POI在資料內平均出現次數:3.21次。
* 預測的street在資料內平均出現次數:3.15次。
* POI和raw_address文字不一致之樣本數佔訓練集資料總樣本數比率:15.38 %
* street和raw_address文字不一致之樣本數佔訓練集資料總樣本數比率:5.80 %
* POI樣本空缺比數佔訓練集資料總樣本數比率:59.50 %
* street樣本空缺比數佔訓練集資料總樣本數比率:23.38 %
* 整理出錯誤寫法與正確寫法的對照表

從上述資訊可以看出，如果想用「規則式」的方式來比對，例如說只要地址中出現興趣點或街道名稱就取出來，是有困難的。因為每個POI或street平均出現的次數約只有3次，而且還有不一致的狀況。顯然這個問題還是必須透過機器/深度學習模型來進行學習與預測。

### 4. 我的預測模型

我花約一天的時間，去評估NLP領域中哪個模型適合處理這個議題。後來我選擇以Tensorflow官網的[Neural machine translation with attention](https://www.tensorflow.org/tutorials/text/nmt_with_attention)範例進行修改。這個範例是將西班牙文翻譯為英文，是以GRU和attention架構的seq2seq模型。

我會選擇這個模型主要是因為它和這次競賽的題目很類似，我們可以利用seq2seq模型，給一串地址文字進去模型，然後直接得到輸出文字結果。且這個模型架構在實作上改成符合我們的需求，以我目前的技術能力不會有困難。

在斷詞做tokenize部分，這邊有自己加入一些想法。傳統在做斷詞時會把標點符號拿掉(例如英文中常見的「,」)，但此處我們沒有做這樣的處理。因為我們認為「,」在地址中是具有意義的，因為它可能是區分不同類別(省份、地標、路、街道或號碼等)，所以「,」應該要留下來。

蝦皮官網要求提交的預測格式為一個欄位，興趣點和街道以斜線進行分隔。在一開始實作時，我是建立兩個模型，分別以原始地址預測興趣點和街道，想說讓機器專心做一件事就好，最後再把兩個模型預測的結果合併起來。

概念圖如下：

![](https://i.imgur.com/7eKZGSc.png)

以上述分開模型預測再結合的架構進行預測，在Public Leaderboard準確度是0.43646。

但後來發現，其實直接預測興趣點和街道(POI/street)更好，概念圖如下所示：

![](https://i.imgur.com/shEzgJr.png)

這樣的架構在Public Leaderboard準確度是0.47946，比分開預測的模型還要多約4%。看起來模型學習能力比想像中的還厲害，可以直接同時預測興趣點和街道。而且這樣的好處是就不用花時間去訓練兩個模型，只要專心訓練一個模型即可。

這次競賽我覺得可惜的部分，因為時間關係及對程式碼熟悉程度，沒有把Transformer機制加入進去，相較於Attetion，Transformer能夠將詞在地址中的位置資訊也考慮進去，效果應該會更好。

本次競賽模型的訓練程式碼，我已放在Github上，有興趣的話可以[[點選這裡]](https://github.com/SuYenTing/Shopee-Code-League-2021/blob/main/shopee_address_elements_extraction.ipynb)觀看。

## 五、心得

以前學投資學時，老師說要學好股票投資，就用真實的錢去買股票，這樣你才會在意，才會把投資學好。我想資料科學也是這樣，想要把資料科學學好，就去打Kaggle，因為看到Public Leaderboard，有比較才會讓人更進步！

這次很感謝隊友們一起參與，能夠藉由題目討論中，提出不同的面向與觀點，互相學習與成長。此外參與Kaggle的好處，也可以看看其他人是怎麼想這個問題。這次蝦皮競賽的隊伍除了台灣外，也包含到東南亞國家，算是一個國際型賽事。能夠和其他國家的人一起競爭，看看自己的程度在哪，釐清自己的現況與不足之處，並能努力去學習並補足。
