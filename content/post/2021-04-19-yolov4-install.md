---
title: YOLOv4手把手安裝流程
author: Yen-Ting, Su
date: '2021-04-19'
slug: yolov4-install
categories:
  - Deep Learning
tags:
  - YOLO
description: "最近在課堂上學到YOLOv3的安裝與使用方法，由於老師上課時講得比較快，且我想自己實作YOLOv4模型(國產模型，要多多支持!)，所以自己整理一份從安裝到實作的筆記，方便之後可以回顧。本篇文章主要說明如何在Windows10上安裝GPU版本的YOLOv4模型。"
---

最近在課堂上學到YOLOv3的安裝與使用方法，由於老師上課時講得比較快，且我想自己實作YOLOv4模型(國產模型，要多多支持!)，所以自己整理一份從安裝到實作的筆記，方便之後可以回顧，並和大家分享。因為文章較長，我分成兩篇文章，第一篇文章是如何在Windows上安裝YOLOv4套件，第二篇則是進行YOLOv4的實作。

本篇文章主要說明如何在Windows上安裝YOLOv4套件，安裝流程主要參考YOLOv4套件作者Alexey Bochkovskiy在Github：[AlexeyAB/darknet](https://github.com/AlexeyAB/darknet) 的說明文件。

另一篇文章：[YOLOv4手把手實作應用](https://suyenting.github.io/post/yolov4-hands-on/)，主要說明如何用YOLOv4套件進行實作，訓練一個能夠辨識筆電、鍵盤及滑鼠三個類別的模型。

由於套件可能還會持續更新，安裝或使用方法可能會有改變，本文僅提供參考。詳細內容可直接閱讀套件作者的Github說明文件。

## 一、 安裝環境

本篇文章是在Win10作業系統環境下，安裝YOLOv4模型GPU版本，以下為硬體及軟體版本資訊：

* 硬體(桌電)
    * CPU：Intel i7-8700
    * 記憶體：16 GB
    * GPU：NVIDIA RTX 2070Ti
* 軟體
    * 作業系統：Windows 10
    * CUDA：11.1
    * cuDNN：8.0.4
    * OpenCV：4.5.1
    * Cmake：3.20.1
    * Visual Studio 2017

## 二、 安裝前置作業

由於套件持續更新，此處的版本僅提供參考，建議直接到套件作者的Github: [AlexeyAB/darknet](https://github.com/AlexeyAB/darknet)再確認整個安裝流程。


* 安裝[Visual Studio 2017](https://visualstudio.microsoft.com/zh-hant/)(記得要安裝英文語言套件)

![](https://i.imgur.com/PBeebNU.png)

![](https://i.imgur.com/ovHE9O1.png)


* 安裝[CUDA](https://developer.nvidia.com/cuda-toolkit-archive)(版本>= 10.2)(建議先安裝Visual Studio 2017，再安裝CUDA)
* 安裝[cuDNN](https://developer.nvidia.com/rdp/cudnn-archive)(版本>=8.0.2)(記得配合CUDA版本)
* 安裝[OpenCV](https://opencv.org/releases/)(版本>=2.4)

下載Windows版本的OpenCV解壓縮後，將opencv資料夾放到電腦中某一個位置(我自己是放在C:\Users\User路徑下)，並將此路徑新增到**環境變數的Path內**：

```
# [openv資料夾放置的目錄]\opencv\build\x64\vc15\bin
C:\Users\User\opencv\build\x64\vc15\bin
```

以及設定**環境變數值**：

```
# [openv資料夾放置的目錄]\opencv\build
OpenCV_DIR = C:\Users\User\opencv\build
```

![](https://i.imgur.com/XoM2hUi.png)

* 安裝[Cmake](https://cmake.org/download/)(版本>=3.18)

## 三、 安裝流程

套件作者在Github說明文件中，推薦在Windows環境使用`vcpkg`來進行編譯，但我個人試作時並沒有成功。後來改採另一個方法，以Cmake來編譯。

Windows版本不用像Linux要特別去設定Makefile以開啟GPU版本，Cmake會自行判斷環境變數偵測電腦是否有裝GPU及CUDA，若有安裝則會自動裝YOLOv4模型的GPU版本。

### 1. 下載Github專案

開啟命令提示字元:
```bash=
# 切換到想要放置的目錄位置(可自行修改)
cd C:\Users\User

# 以下取自Github說明文件:
git clone https://github.com/AlexeyAB/darknet.git
```

### 2. Cmake建立編譯文件

打開Cmake-gui：

![](https://i.imgur.com/ybLS4Q2.png)

按下圖的畫面輸入相關設定，路徑設定請選擇git clone下載的資料夾：

![](https://i.imgur.com/j59Cl7x.png)

接下來Cmake會開始檢查環境，若沒有問題，即可按下`Generate`：

![](https://i.imgur.com/V7zG2Ea.png)

系統提示畫面最後兩行如下，若相同則代表沒問題：

![](https://i.imgur.com/3hEnxT5.png)


### 3. 開始編譯

打開darknet目錄中的`Darknet.sln`檔案(以VS2017開啟)：

![](https://i.imgur.com/jCO15f6.png)

在右側選擇`ALL_BUILD`，點選右鍵選擇建置：

![](https://i.imgur.com/cupNXpj.png)

建置完後畫面如下：

![](https://i.imgur.com/NGI7IuS.png)

接下來在右側選擇`INSTALL`，點選右鍵選擇建置：

![](https://i.imgur.com/LKql4Dg.png)

建置完後畫面如下：

![](https://i.imgur.com/G3YyHf3.png)

上述步驟做完後，在darknet目錄中，可以看到`darknet.exe`，這個即為YOLOv4模型的主程式，待會會以此程式來進行訓練及預測：

![](https://i.imgur.com/Ee8br2R.png)


### 4. 範例測試

為測試程式是否能夠正常運作，此處我們執行以下指令：

```bash=
# 切換到darknet資料夾
cd C:\Users\User\darknet

# 複製darknet.exe檔案到子目錄
cp darknet.exe .\build\darknet\x64\

# 切換到執行程式的資料夾
cd C:\Users\User\darknet\build\darknet\x64
```

下載YOLOv4權重，請[點選這裡下載](https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v3_optimal/yolov4.weights)。下載後將`yolov4.weights`放置到`C:\Users\User\darknet\build\darknet\x64`目錄內。

接著繼續執行指令，以訓練好的YOLOv4模型來預測照片：

```bash=
darknet detector test cfg/coco.data cfg/yolov4.cfg yolov4.weights data/dog.jpg
```

如果有跑出圖片，代表YOLOv4套件安裝成功！

![](https://i.imgur.com/BnsNnVW.jpg)


