---
title: Linux(Ubuntu 18.04)建立R及GPU環境相關設定流程
author: Yen-Ting, Su
date: '2020-09-22'
slug: linux-ubuntu-configure
categories:
  - Programming
tags:
  - Linux
  - Ubuntu
  - R
  - GPU
description: "老師近期用研究經費添購一台工作站，讓機器學習模型程式在訓練時能夠更加快速。這台工作站安裝Linux Ubuntu 18.04作業系統，在設定環境過程中難免還是採到坑，這篇內容是我建立相關環境的相關指令說明，紀錄起來以後有需要時可以查詢。"
---

老師近期用研究經費添購一台工作站，讓機器學習模型程式在訓練時能夠更加快速。這台工作站安裝Linux Ubuntu 18.04作業系統，在設定環境過程中難免還是採到坑，這篇內容是我建立相關環境的相關指令說明，紀錄起來以後有需要時可以查詢。

### 1. Putty安裝方式

[下載頁面連結位置](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

![](https://i.imgur.com/tWZ743O.png)

下載後點選putty.exe檔案即可進入連線畫面

![](https://i.imgur.com/6PIgJsA.png)

### 2. 切換終端機命令列為英文

將終端機命列列系統訊息由中文切為英文，因為這樣在處理Bug時會有比較多的資源可以參考。

直接在圖形化介面進行修正，搜尋Settings -> Region & Language

修改 Language 和 Formats選項，之後重新登入即可轉為英文環境


### 3. SSH套件相關設定

開機時自動啟動SSH

```linux
sudo systemctl enable ssh
```

為確保資安，將SSH的Port由22改為[新的Port號碼]
主要是修改sshd_config文件，將裡面的Port由22改為[新的Port號碼]

```linux
sudo vi /etc/ssh/sshd_config
```

相關改法可看[參考頁面](https://coder.tw/?p=5020)

防火牆記得設定允許Port [新的Port號碼] 連入

```linux
sudo ufw allow [新的Port號碼]
```

另外為了資安設定，可以針對SHH Port加入IP白名單
寫法可參考[頁面連結](https://noob.tw/ufw/)
如果有需要設定IP範圍，會用到網路遮罩寫法，對應的寫法可參考[Wiki頁面說明](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#IPv4_CIDR_blocks)

### 4. 安裝R

[指令參考頁面](https://www.digitalocean.com/community/tutorials/how-to-install-r-on-ubuntu-18-04) 

```linux
sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran40/'
sudo apt update
sudo apt install r-base
```

安裝好後在命令列直接輸入`R`即可進入R環境

```linux
R
```

### 5. 安裝RStudio Server

[指令參考頁面](https://rstudio.com/products/rstudio/download-server/debian-ubuntu/)

```linux
sudo apt-get install gdebi-core
wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.3.1073-amd64.deb
sudo gdebi rstudio-server-1.3.1073-amd64.deb
```

防火牆設定允許Port 8787 連入

```linux
sudo ufw allow 8787
```

設定開機時自動啟動RStudio Server

```linux
sudo systemctl enable rstudio-server
```

### 6. Rstudio Server維護相關指令

請參考[頁面連結](https://support.rstudio.com/hc/en-us/articles/200532327-Managing-the-Server)


### 7. 建立使用者

```linux
sudo adduser [使用者名稱]
```

若要改變密碼，先以SSH登入該使用者，之後輸入`passwd`，依照系統提示進行密碼修改

```linux
passwd
```

### 8. 安裝Linux套件

依據R套件需求，安裝Linux相關套件

```linux
sudo apt install libssl-dev
sudo apt install libxml2-dev
sudo apt install libcurl4-openssl-dev
```

### 9. 重開機指令

```linux
sudo systemctl reboot
```

### 10. 安裝NVIDIA顯卡驅動、CUDA、CuDNN(配合Tensorflow需求)

由於Tensorflow套件會隨時更新其硬體需求條件，所以必須要先至官網確認其硬體環境要求。[點此連結](https://www.tensorflow.org/install/gpu?hl=zh-tw)

![](https://i.imgur.com/gz94JPA.png)

官網有提供Ubuntu18.04版本的安裝指令，但實測發現裝CUDA10.1時會出問題。
經過查詢發現NVIDIA驅動必須要裝450版本才不會出問題(官網的程式碼是裝430版本)。

目前Tensorflow在Github的[安裝指引](https://github.com/tensorflow/docs/blob/master/site/en/install/gpu.md)上已更新NVIDIA驅動要裝450版本，但官網沒有。

首先安裝NVIDIA顯示卡驅動450版本，參考此頁面(https://medium.com/@jackfrisht/install-nvidia-driver-via-ppa-in-ubuntu-18-04-fc9a8c4658b9)

```linux
# 新增PPA套件庫
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install ubuntu-drivers-common

# 查看可安裝的版本
ubuntu-drivers devices

# 安裝NVIDA驅動450版本
sudo apt install nvidia-driver-450 

# 重新啟動
sudo reboot
```

輸入顯卡查詢指令判斷是否有正常運作，經查詢後確定GPU有成功抓到。

```linux
nvidia-smi
```

成功畫面如下:

![](https://i.imgur.com/9ErTXHe.png)


接下來安裝Cuda10.1及CuDnn7.6.5

```linux
# Install development and runtime libraries (~4GB)
sudo apt-get install --no-install-recommends \
    cuda-10-1 \
    libcudnn7=7.6.5.32-1+cuda10.1  \
    libcudnn7-dev=7.6.5.32-1+cuda10.1
    
# 重新開機
sudo reboot
```

安裝好後，需要在環境變數中加入CUDA路徑:

```linux
# 打開.bashrc檔案
sudo nano ~/.bashrc

# 將下列兩行貼到.bashrc文件最後面並儲存然後退出
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/64:$LD_LIBRARY_PATH

# 更新.bashrc檔案
source ~/.bashrc

# 查詢CUDA版本
nvcc -V
```

成功畫面如下:
![](https://i.imgur.com/y5pwyVB.png)

接下來再安裝Tensor RT:

```linux
sudo apt-get install -y --no-install-recommends libnvinfer6=6.0.1-1+cuda10.1 \
    libnvinfer-dev=6.0.1-1+cuda10.1 \
    libnvinfer-plugin6=6.0.1-1+cuda10.1
```

執行後若沒錯誤即安裝完成。


-----
此區域尚未設定完成，需要確認該電腦是否有裝SLI橋接器才能進行。
```linux
若有兩張顯卡要進行SLI串接，則執行以下指令:
sudo nvidia-xconfig
sudo nvidia-xconfig --multigpu=on
sudo nvidia-xconfig --sli=On
```
-----

#### 補充說明: 顯示卡安裝過程可能發生gcc版本不一致，解決方法如下:

首先查詢gcc版本指令

```linux
gcc --version
```

CUDA10.1對應的gcc版本為7，若版本不一致則安裝gcc7版，並調整gcc7版優先權

[參考頁面](https://archerfmy.github.io/2017/04/12/How-to-switch-your-gcc-g-version-in-ubuntu/)

```linux
# 安裝gcc-7
sudo apt-get install gcc-7 g++-7

# 修改優先權限(此處系統有gcc9和gcc7版本 設定將gcc7版本優先權高於gcc9版本)
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 50
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 50
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 40
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 40

# 確認是否為gcc7版
gcc --version
```

之後重新執行CUDA安裝即可。


### 11. 安裝Anaconda

至[Anaconda官網](https://www.anaconda.com/products/individual)尋找下載連結

安裝流程說明[參考頁面](https://medium.com/@maniac.tw/%E5%AE%89%E8%A3%9D-anaconda-tensorflow-2bbc6e220fa4)

```linux
# 下載檔案
wget https://repo.anaconda.com/archive/Anaconda3-2020.07-Linux-x86_64.sh

# 執行安裝
bash Anaconda3-2020.07-Linux-x86_64.sh

# 依照系統提示安裝並加入環境變數(PATH)

# 更新.bashrc檔案
source ~/.bashrc
```

### 12. 安裝Python虛擬環境測試Tensorflow

```linux
# 建立虛擬環境
conda create --name "r-reticulate" python=3.6

# 進入虛擬環境
conda activate "r-reticulate"

# 安裝相關套件
# 首先升級pip
pip install --upgrade pip

# 安裝tensorflow
pip install tensorflow

# 安裝keras
pip install keras

# 虛擬環境內執行Python
python

# 在Python測試tensorflow2.0是否能正常運行
import tensorflow as tf
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
print('GPU', tf.test.is_gpu_available())
a = tf.constant(2.0)
b = tf.constant(4.0)
print(a + b)
```


### 13. 安裝R Keras

首先打開R Studio Server

```R
# 安裝套件
install.packages("keras")
```

安裝好後，執行範例程式碼，[點選此連結](https://github.com/rstudio/keras/blob/master/vignettes/examples/mnist_mlp.R)到RStudio的keras套件之Github頁面複製程式碼在R中執行。

成功執行的畫面
![](https://i.imgur.com/nkm8s8G.png)

















