---
title: YOLOv4手把手實作應用
author: Yen-Ting, Su
date: '2021-04-20'
slug: YOLOv4-hands-on
categories:
  - Deep Learning
tags:
  - YOLO
description: "本篇文章主要為說明如何進行YOLOv4的實作，我是使用YOLOv4-tiny模型，在預訓練好的模型權重為基礎下，讓模型能夠從影像中預測出3個分類：筆電、滑鼠和鍵盤。會選擇這3個分類是因為在生活中比較好取得，可以讓我們在模型訓練完後，以筆電的攝像頭即時偵測，看看模型能不能夠偵測到。"
---

本篇文章主要為說明如何進行YOLOv4的實作，我是使用YOLOv4-tiny模型，在預訓練好的模型權重為基礎下，讓模型能夠從影像中預測出3個分類：筆電、滑鼠和鍵盤。會選擇這3個分類是因為在生活中比較好取得，可以讓我們在模型訓練完後，以筆電的攝像頭即時偵測，看看模型能不能夠偵測到。

由於此處會有需多檔案在不同的資料夾間移來移去，建議先閱讀我寫的另一篇文章：[YOLOv4手把手安裝流程](https://suyenting.github.io/post/yolov4-install/)，了解我在Windows上安裝YOLOv4套件的流程與路徑位置，在閱讀本篇文章時會比較好理解。

## 一、 模型訓練與預測

實作部分，我們將預測筆電、滑鼠和鍵盤這3個物品，共有3個分類。由於我們的分類比較少，且為了加快模型學習速度，所以此處我們採用**YOLOv4模型的tiny版本**。

### 1. 複製檔案

在桌面建立一個名為`model`的資料夾，存放模行訓練所需的資料及相關設定。

將以下檔案**複製**到`model`資料夾內：

* C:\Users\User\Desktop\darknet\darknet.exe
* C:\Users\User\Desktop\darknet\build\darknet\x64\pthreadVC2.dll
* C:\Users\User\Desktop\darknet\cfg\yolov4-tiny-custom.cfg

另外，還需要下載YOLOv4-tiny模型預訓練的權重檔案：[yolov4-tiny.conv.29](https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v4_pre/yolov4-tiny.conv.29)

目前我們的資料夾共有4個檔案：

![](https://i.imgur.com/ut9e3JV.png)


### 2. 調整模型設定

以文字編輯器打開`yolov4-tiny-custom.cfg`檔案，對裡面的設定進行修改，以符合我們預測的需求。此處模型修改的方式，參考自套件作者在Github上提供的說明：

* change line batch to batch=64
    * 文件內的batch參數已設定為64，不用改

* change line subdivisions to subdivisions=16
    * 將subdivisions參數改為16

* change line max_batches to (classes*2000 but not less than number of * training images, but not less than number of training images and not less than 6000), f.e. max_batches=6000 if you train for 3 classes
    * 由於我們要預測3個分類，此處按作者建議設定max_batches為6,000

* change line steps to 80% and 90% of max_batches, f.e. steps=4800,5400
    * 取max_batches的80%和90%值做為steps的值，由於max_batches設定為6,000，故按作者建議的比率設定為4800,5400

* set network size width=416 height=416 or any value multiple of 32:
    * 修改width及height參數為416

* change line classes=80 to your number of objects in each of 3 [yolo]-layers
    * 此處我們預測3分類，所以將classes參數設定為3。因我們是使用YOLOv4-tiny，只要在文件中修改2個地方即可。若為YOLOv4原模型，需要修改3個地方。

* change [filters=255] to filters=(classes + 5)x3 in the 3 [convolutional] before each [yolo] layer, keep in mind that it only has to be the last [convolutional] before each of the [yolo] layers.
    * 由於我們預測3分類，所以修改filters參數值為(3+5)x3=24。filters參數修改的位置在每個[yolo]標籤前的[convolutional]標籤內，不要修改其他層的filters，總計要在文件中修改3個filters參數值。

![](https://i.imgur.com/VtqLZzS.png)

![](https://i.imgur.com/cCKxtji.png)

以上設定配置完後，儲存完即可關閉。


### 3. 下載訓練及驗證圖片集

由於我們是要做物件偵測，除了要有照片外，每張照片上面還必須要有物件所在的範圍資訊。幸好Google有提供[Open Images Dataset](https://storage.googleapis.com/openimages/web/index.html)照片素材庫，讓我們可以免去準備圖片的問題。

另外更方便的是，Github專案：[theAIGuysCode/OIDv4_ToolKit](https://github.com/theAIGuysCode/OIDv4_ToolKit)，已經寫好Python程式，可以來協助下載Open Images Dataset的圖片，並且轉為YOLOv4模型所需要的標籤格式。

首先我們先下載此Github專案，開啟命令提示字元:

```bash=
# 切換到想要放置的目錄位置(可自行修改)
cd C:\Users\User

# 以下取自Github說明文件:
git clone https://github.com/theAIGuysCode/OIDv4_ToolKit.git
```

安裝套件所需的套件，以系統管理員身分開啟命令提示字元：
```bash=
# 切換到套件資料夾路徑底下
cd C:\Users\User\OIDv4_ToolKit

# 安裝所需Python相依套件
pip3 install -r requirements.txt
```

接下來執行程式來下載我們所需要的圖片，本次我們要預測筆電、滑鼠和鍵盤，這3個圖片在Open Images Dataset中的分類名稱為Laptop、Computer mouse和Computer keyboard。

打開命令提示字元，執行`main.py程式`，將Open Images Dataset所需要的分類圖片下載下來：
```bash=
# 切換到套件資料夾路徑底下
cd C:\Users\User\OIDv4_ToolKit

# main.py程式碼代入參數格式為:
# python3 main.py downloader --classes [下載的分類名稱] --type_csv [資料類別] --limit [下載最大數量] --multiclasses[是否一起下載分類]
# 其中[下載的分類名稱]，可以放入多個分類名稱，以空白格做區分。
# 若分類名稱內原本有空白，則需以底線作取代

# 下載訓練集資料
python main.py downloader --classes Laptop Computer_mouse Computer_keyboard --type_csv train --limit 2000 --multiclasses 1

# 下載驗證集資料
python main.py downloader --classes Laptop Computer_mouse Computer_keyboard --type_csv validation --limit 200 --multiclasses 1
```

執行上述指令時，若程式有詢問時，一律輸入y即可：
![](https://i.imgur.com/dNqOhtr.png)

### 4. 整理YOLOv4模型讀取資料格式要求

檔案下載好後，可在`C:\Users\User\OIDv4_ToolKit\OID\Dataset`路徑下，看到下載的圖片及標籤(label)資料夾。但比較麻煩的是，這邊的標籤格式並不符合YOLOv4模型要求，但還好這個Github專案有寫一支`convert_annotations.py`程式可以協助做轉換。

打開`OIDv4_ToolKit`資料夾內的`classes.txt`檔案，輸入我們下載的照片分類名稱：

![](https://i.imgur.com/kuU93lS.png)

![](https://i.imgur.com/6fprF6h.png)

修改好後，在命令提示字元中，執行`convert_annotations.py`程式

```bash=
# 切換到套件資料夾路徑底下
cd C:\Users\User\OIDv4_ToolKit

# 執行產生YOLOv4模型所需標籤格式之程式
Python convert_annotations.py
```

執行完後，我們查看train和validation資料夾：

* C:\Users\User\OIDv4_ToolKit\OID\Dataset\train\Laptop_Computer mouse_Computer keyboard
* C:\Users\User\OIDv4_ToolKit\OID\Dataset\validation\Laptop_Computer mouse_Computer keyboard

可以看到每張照片都有對應一個txt檔案，此txt檔案即為讓YOLOv4模型知道這張照片裡面有什麼物件分類，以及物件分類在照片中的位置。

![](https://i.imgur.com/yjcRYKf.png)

txt檔案內儲存的資訊依序為類別編號、物件範圍中心的x值、物件範圍中心的y值、物件範圍寬度及物件範圍高度
![](https://i.imgur.com/3qI9c03.png)

YOLOv4模型主要是透過讀取存放照片目錄的txt檔案，來知道要訓練/驗證的照片放置在哪裡。所以此處我們還需要再執行一支程式，產生train.txt及valid.txt兩個照片目錄檔案，供YOLOv4模型來讀取。

我們在`C:\Users\User\OIDv4_ToolKit`目錄底下，新增這支名為`create_catalog.py`程式，這支程式是我自己寫的：

```python=
# 建立train和valid的照片目錄
import os

# 紀錄當前目錄
originDir = os.getcwd()

# 讀取使用者設定的分類名稱
classes = []
with open("classes.txt", "r") as myFile:
    for num, line in enumerate(myFile, 0):
        line = line.rstrip("\n")
        classes.append(line)
    myFile.close()

# 分類名稱資料夾
classesFolderName = '_'.join(classes)

# 迴圈train和validation資料夾
for folder in ['train', 'validation']:

    # 切換目錄
    os.chdir('./OID/Dataset/' + folder + '/' + classesFolderName)

    # 產生資料夾照片目錄
    image_files = []
    for filename in os.listdir(os.getcwd()):
        if filename.endswith('.jpg'):
            image_files.append('data/' + folder + '/' + filename)

    # 切換至原目錄
    os.chdir(originDir)

    # 寫出照片目錄檔案
    with open(folder + '.txt', 'w') as outfile:
        for image in image_files:
            outfile.write(image)
            outfile.write('\n')
        outfile.close()
```

在命令提示字元中，執行`create_catalog.py`程式：

```bash=
# 切換到套件資料夾路徑底下
cd C:\Users\User\OIDv4_ToolKit

# 執行產生YOLOv4模型所需標籤格式之程式
Python create_catalog.py
```

執行後，即可在`OIDv4_ToolKit`資料夾中，看到`train.txt`和`validation.txt`檔案，裡面存放每張要訓練/驗證照片的路徑。

![](https://i.imgur.com/Q52cofD.png)

接下來在我們要將`OIDv4_ToolKit`資料夾下載的照片資料，移到剛桌面上的`model`資料夾。

首先在`model`資料夾中，建立`data`和`backup`及`results`資料夾:

![](https://i.imgur.com/eIcaQwo.png)

在`model\data`資料夾下，將剛在`OIDv4_ToolKit`資料夾建立的`train.txt`和`validation.txt`檔案複製過去。

![](https://i.imgur.com/ETXsHLp.png)


接著將`C:\Users\User\OIDv4_ToolKit\OID\Dataset\train\Laptop_Computer mouse_Computer keyboard`存放訓練集照片的資料夾，移到`model\data`資料夾內，並將資料夾名稱由`Laptop_Computer mouse_Computer keyboard`改為`train`。

![](https://i.imgur.com/IEV5txC.png)

再來將`C:\Users\User\OIDv4_ToolKit\OID\Dataset\validation\Laptop_Computer mouse_Computer keyboard`存放驗證集照片的資料夾，移到`model\data`資料夾內，並將資料夾名稱由`Laptop_Computer mouse_Computer keyboard`改為`validation`。

![](https://i.imgur.com/3kjMpAN.png)

處理完後，目前`model\data`資料夾內長相為：

![](https://i.imgur.com/7tyFecB.png)


然後，在`model\data`資料夾中新增`obj.names`檔案，裡面要放上YOLOv4模型要預測的分類名稱。直接將`OIDv4_ToolKit/classes.txt`複製過來即可，記得順序要相同。

![](https://i.imgur.com/Q6J8XPx.png)

接著再新增`obj.data`檔案，此檔案是讓YOLOv4模型所需資料的相關路徑：

在`obj.data`檔案中輸入以下資訊：

```bash=
classes = 3
train  = data/train.txt
valid  = data/validation.txt
names = data/obj.names
backup = backup/
```

* classes: 即YOLOv4模型要預測分類的數量
* train: 訓練集資料照片的位置資訊檔案
* valid: 驗證集資料照片的位置資訊檔案
* names: 分類對應名稱
* backup: YOLOv4模型儲存的位置

![](https://i.imgur.com/8ndPLzX.png)

### 5. 開始訓練YOLOv4模型

首先確認`model`資料夾內容是否和我的一致：

![](https://i.imgur.com/UCZmVPW.png)

`model\data`資料夾內容：

![](https://i.imgur.com/qnKUn9s.png)

接下來在開啟命令提示字元，輸入執行指令：

```bash=
# 切換資料夾路徑
cd C:\Users\User\Desktop\model

# 開始訓練
# darknet detector train [模型資料路徑設定檔] [YOLOv4模型設定檔] [預訓練的權重檔案]
darknet detector train data/obj.data yolov4-tiny-custom.cfg yolov4-tiny.conv.29 -map
```

執行成功時，會跳出一張圖片呈現目前模型訓練的狀況。

另外可至工作管理員查看GPU是否有在運作，只要專屬GPU記憶體使用量有呈現在使用的狀況，即代表YOLOv4模型有使用到GPU：

![](https://i.imgur.com/Vo32S0o.png)


若訓練過程中發生`CUDA Error: out of memory`:

![](https://i.imgur.com/fyuAVB4.png)

或者`CUDA Error: an illegal memory access was encountered`：

![](https://i.imgur.com/3bvparT.png)

上述兩個狀況是代表GPU記憶體不足，可以修改yolov4-tiny-custom.cfg內的subdivisons參數值往上調整(不超過batch值)，避免GPU記憶體負擔太大。

![](https://i.imgur.com/pjpYqP4.png)

關於subdivisons參數，可以參考[這篇文章](https://github.com/pjreddie/darknet/issues/224)的回答：

![](https://i.imgur.com/2Diy6cy.png)

subdivisons主要是設定mini-batch的數量，所以subdivisons值設定愈大，對於GPU記憶體負擔就會愈小。

模型訓練完後，可以在`model/backup`內，看到自動儲存的模型：

![](https://i.imgur.com/4ULOC34.png)
* 備註：因為我在`yolov4-tiny-custom.cfg`設定檔的max_batches參數設定超過10,000次，所以此處只會每10,000次存一次模型。如果max_batches設定小於10,000次，則每1,000次會儲存一次model。

另外在`model`資料夾中，可以看到`chart_yolov4-tiny-custom.png`檔案，此即整個模型訓練過程的損失函數圖：

![](https://i.imgur.com/kmfvfiF.png)

總共跑12,000次疊代，整個模型訓練大概花一小時多左右的時間。


### 6. YOLOv4模型評估準則

YOLOv4模型評估標準涵蓋到兩種：
* IoU (intersect over union)
* mAP (mean average precision)

關於評估準則計算的方式，可以參考相關文章說明：
* [Laplace's Lab - 正確理解 YOLO 的辨識準確率](https://laplacetw.github.io/data-sci-yolo-accuracy/)
* [Tommy Huang - 深度學習系列: 什麼是AP/mAP?](https://chih-sheng-huang821.medium.com/%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92%E7%B3%BB%E5%88%97-%E4%BB%80%E9%BA%BC%E6%98%AFap-map-aaf089920848)

我們可以執行以下指令來得到模型預測的績效：

```bash=
# 切換資料夾路徑
cd C:\Users\User\Desktop\model

# darknet detector map [模型資料路徑設定檔] [YOLOv4模型設定檔] [訓練過程中最佳的權重檔案]
darknet detector map data/obj.data yolov4-tiny-custom.cfg backup/yolov4-tiny-custom_best.weights
```
![](https://i.imgur.com/0G5dPsj.png)

以我們訓練的模型績效來看，mAP@0.50有到82.38%，效果算蠻好的。

### 7. YOLOv4模型預測

在執行預測前，我們需要將`C:\Users\User\darknet\data\labels`這個資料夾，複製到我們的`model/data`資料夾底下：

![](https://i.imgur.com/Os8oG74.png)

這個labels資料夾裡面存放一些英文字母和數字的圖片，主要是待會我們在預測時，為了能夠在物件標註的範圍上，顯示物件名稱和confidence使用。

YOLOv4模型可以輸入以下指令針對一張照片進行預測：

```bash=
# 切換資料夾路徑
cd C:\Users\User\Desktop\model

# darknet detector test [模型資料路徑設定檔] [YOLOv4模型設定檔] [訓練過程中最佳的權重檔案] [要預測的照片路徑位置]
darknet detector test data/obj.data yolov4-tiny-custom.cfg backup/yolov4-tiny-custom_best.weights data/validation/0421a6e230527185.jpg
```

執行指令後會跳出預測照片即模型物件偵測到的範圍：

![](https://i.imgur.com/yFRrxO2.jpg)

並會在`model`資料夾下產生`predictions.jpg`圖片：

![](https://i.imgur.com/qJ3NwhA.png)

不過這行指令只能一次預測一張圖片，如果想要儲存每張圖片的偵測結果，需要額外再撰寫程式。

* 相關討論文章：[Run YoloV3 detections on thousands of images and save outputs?](https://github.com/pjreddie/darknet/issues/723)

此處我自己寫一支Python程式，透過迴圈的方式來批次執行`darknet detector test`指令預測照片，並存取下來。請在`model`資料夾下新增一支名為`predict.py`的Python程式：

```python=
# 載入套件
import sys
import os

# 接收參數-預測照片路徑txt檔案
# 例如:
# modelCfg = 'yolov4-tiny-custom.cfg'
# modelWeights = 'backup/yolov4-tiny-custom_best.weights'
# imageFilePath = 'data/validation.txt'
modelCfg = sys.argv[1]
modelWeights = sys.argv[2]
imageFilePath = sys.argv[3]

# 建立預測資料夾
if 'predictImage' not in os.listdir():
    os.makedirs('predictImage')

# 讀取照片路徑
imageFiles = []
with open(imageFilePath, 'r') as myFile:
    for num, line in enumerate(myFile, 0):
        line = line.rstrip("\n")
        imageFiles.append(line)
    myFile.close()

# 迴圈執行darknet預測執行
for imageFile in imageFiles:

    # 建立cmd指令
    commands = ' '.join(['darknet detector test data/obj.data', modelCfg, modelWeights, imageFile, '-dont_show'])

    # 執行cmd指令
    os.system(commands)

    # 將預測照片重新命名並移至預測資料夾下
    os.replace('predictions.jpg', 'predictImage/pred_' + imageFile.split('/')[-1])
```

接著打開命令提示字元，輸入：

```bash=
# 切換資料夾路徑
cd C:\Users\User\Desktop\model

# 開始訓練
# python predict.py [YOLOv4模型設定檔] [預訓練的權重檔案] [預測的照片路徑檔案]
python predict.py yolov4-tiny-custom.cfg backup/yolov4-tiny-custom_best.weights data/validation.txt
```

執行後，即會在`model`資料夾底下產生一個`predictImage`資料夾，裡面會存放YOLOv4模型預測的照片：

![](https://i.imgur.com/CwYfPnq.png)

不過這個方法會有點慢，因為每次使用`darknet detector test`指令預測一張照片時，都要重新載入權重一次。我自己找官方套件的說明，好像並沒有看到能夠一次預測全部照片的指令，此處未來看看有沒有更好的做法。


## 二、模型應用

此處我們以筆電的攝像頭做即時偵測，來測試看看我們自己訓練的模型表現。

### 1. 筆電環境

筆電的CPU規格為Intel i5-8265U，GPU規格為NVIDIA GeForce MX250，darknet是編譯有GPU的版本。我自己有實測如果是以CPU跑即時偵測，FPS大概在3左右，如果是以GPU跑的話，FPS可以到30左右，明顯可看出GPU在影像辨識的優勢。

在筆電上，我一樣在桌面上建立一個名為`model`的資料夾，資料夾內容如下，提供給大家參考：

![](https://i.imgur.com/nZ97i3O.png)
* `data`資料夾：資料夾配置如下說明
* `darknet.exe`：按文章前面介紹安裝YOLOv4的方法於筆電上編譯出來
* `pthreadVC2.dll`：由`C:\Users\User\Desktop\darknet\build\darknet\x64\pthreadVC2.dll`複製過來
* `yolov4-tiny-custom.cfg`：我們客製化的模型，我直接從桌電複製過來
* `yolov4-tiny-custom_best.weights`：在桌電上訓練好的權重檔案，複製過來


![](https://i.imgur.com/ZCmXO5l.png)

* `labels`資料夾是從`C:\Users\User\darknet\data\labels`複製出來的，裡面存放一些英文字母和數字的圖片，主要是待會我們在預測時，為了能夠在物件標註的範圍上，顯示物件名稱和confidence使用。

* `obj.data`檔案內容為：

```bash=
classes = 3
names = data/obj.names
```

* `obj.names`檔案內容為：

```bash=
Laptop
Computer mouse
Computer keyboard
```

### 2. 實測成果

因為我手邊只有滑鼠，所以我們測試自己訓練的模型，能不能辨識出滑鼠，測試結果確實有偵測到：

![](https://i.imgur.com/RhiRu58.jpg)

執行筆電攝像頭的做即時偵測的指令如下：

```bash=
# 切換資料夾路徑
cd C:\Users\User\Desktop\model

# 執行筆電攝像頭即時偵測
# darknet.exe detector demo [模型資料路徑設定檔] [YOLOv4模型設定檔] [訓練過程中最佳的權重檔案] -c 0
darknet.exe detector demo data/obj.data yolov4-tiny-custom.cfg yolov4-tiny-custom_best.weights -c 0
```





