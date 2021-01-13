---
title: 將Python程式放在Linux背景執行
author: Yen-Ting, Su
date: '2021-01-13'
slug: run-python-in-linux-background
categories:
  - Programming
tags:
  - Python
  - Linux
description: "本篇文章說明如何在Linux作業系統的Shell背景執行Python程式，而且在使用者離線或登出時不會中斷。"
---

最近在寫Python的爬蟲程式碼，為了測試在Linux上是否能夠正常運作，所以會開啟Shell在上面執行Python跑程式碼。但有時為了測試爬蟲的穩定性，我們會讓這支程式運作一段期間，這時候可能會需要即使我離線，程式也能夠在Linux一直運作的需求。

本篇文章將從最基本的指令開始說明，依序拓展到能夠滿足這個需求的指令。

若覺得本篇文章太長，這邊放上最後寫好的指令提供給大家參考：

```bash=
nohup python -u myPythonCode.py &> myPythonCode.log &
```

## 1. Linux執行Python程式碼

在Shell上執行時Python程式碼時，輸入指令如下：

```bash=
python materialInfo.py
```

materialInfo.py這支程式是用來爬取公開資訊觀測站重大訊息的，在程式內我有將程式當前運作狀況印出來，執行後畫面如下所示：

![](https://i.imgur.com/fo5OGZt.png)

## 2. Python程式碼放入Linux背景執行

這個Shell在執行Python程式時，就不能做其他事情，若想做其他事情可以直接開新的Shell來操作。但若只想要在當前的Shell做其他事情是可以的，只要將Python放在背景執行即可，我們只要在原本的指令後面加上`&`，即可將程式放到背景執行。執行指令如下：

```bash=
python materialInfo.py &
```

輸入指令後，會發現跳出一個數字，以我的畫面為例是1773，這個數字代表的是**PID**，代表我們已將程式放入背景執行，但這時發現程式依舊會印訊息出來。

![](https://i.imgur.com/w1jwekr.png)


我們可以將Python程式碼內的`print()`註解掉，這樣就不會印東西出來。可是我們還是想了解程式運作的狀況，不然會不知道程式到底有沒有如我們所想地運作。這時候我們可以利用Linux的導入功能，將程式產出的訊息寫入到log檔案內，不用印出來，指令改寫如下：

```bash=
python -u materialInfo.py &> materialInfo.log &
```

此處的Python指令會多加一個參數`-u`，這個參數主要是讓Python能夠立即將訊息放到log檔案內，這樣我們可以方便地去查閱log內容，判斷程式當前運作狀況。執行指令後畫面如下：

![](https://i.imgur.com/Tn0i6fC.png)

執行後確實沒有印出訊息來，若想要觀察程式運作狀況，除了利用`ps`指令外，也可以直接下指令`cat materialInfo.log`印出log檔案內容。

若想要終止程式，可以透過`kill`指定PID將程序刪除。

## 3. 讓Python程式碼在離線或登出後還能在Linux背景執行

但目前還有一個問題是，儘管將程式碼放在背景執行，只要使用者關掉Shell或登出，這個背景執行就會強制被中斷。為了解決這個問題，可以在指令添加`nohup`，其名稱為`no hang up`的縮寫，意即使用者離線或登出後，程式依然能夠持續執行。我們將指令改寫如下：

```bash=
nohup python -u materialInfo.py &> materialInfo.log &
```

執行指令後，離線或登出再重新進來，我們下指令`ps aux`，即可查到程式確實依然還在運作。或者也可透過`cat materialInfo.log`讀取log檔案觀察程式狀況。

若想要終止程式，一樣透過`kill`指令刪除PID終止程式。

以上就是本篇文章的說明內容，我們主要是透過`nohup`與`&`兩個指令結合，就可以讓程式一直在linux背景常駐執行。

謝謝您的閱讀！

## 4. 參考資料

* [Linux 的 nohup 指令使用教學與範例，登出不中斷程式執行](https://blog.gtwang.org/linux/linux-nohup-command-tutorial/)
* [把命令放到背景執行](http://linuxdiary.blogspot.com/2007/10/blog-post_30.html)
* [解決python nohup linux 後臺執行輸出的問題](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/356864/)


