---
title: GitLab備份及復原流程筆記
author: Yen-Ting, Su
date: '2022-07-10'
slug: gitlab-backup-and-restore
categories:
  - gitlab
tags:
  - gitlab
description: "之前在工作上被交辦要架設私有的GitLab Server供團隊使用，方便大家做好程式碼版控以及團隊合作開發。在GitLab Server成功架起並且順利運作一段期間後，近期剛好遇到上面要求要擬一個災難復原演練計畫。於是想說藉由這次機會，把GitLab Server的備份和還原流程跑過一遍，避免在未來真的遇到意外事件時，發生無法挽回的狀況。"
---

## 一、前言

之前在工作上被交辦要架設私有的GitLab Server供團隊使用，方便大家做好程式碼版控以及團隊合作開發。在GitLab Server成功架起並且順利運作一段期間後，近期剛好遇到上面要求要擬一個災難復原演練計畫。於是想說藉由這次機會，把GitLab Server的備份和還原流程跑過一遍，避免在未來真的遇到意外事件時，發生無法挽回的狀況。

此處災難復原演練計畫模擬的場景是，如果GitLab Server硬碟遭受意外被破壞，且無法順利復原時，我們對應的措施是重新建立新的GitLab Server主機，並透過每日固定在異地主機上的完整備份檔案，來進行還原。

由於軟體更迭速度很快，網路上搜尋的資源大部分皆為舊的指令。所以這篇文章是我直接參考[GitLab官方文件](https://docs.gitlab.com/ee/raketasks/backup_restore.html#back-up-and-restore-gitlab)，成功執行備份和還原的筆記。所以，如果你在看這篇文章時，離我撰寫的時間已經有一段時間，建議還是要去看[GitLab官方文件](https://docs.gitlab.com/ee/raketasks/backup_restore.html#back-up-and-restore-gitlab)會最正確，也可避免遇到奇怪的bug問題。

接下來本篇筆記將在第二節說明，如何進行每日自動完整備份GitLab的流程。在第三節說明，如何在新主機上安裝GitLab並執行還原。

## 二、備份流程

### Step1. 查看主機GitLab版本

首先查看目前主機的GitLab版本，因備份復原的主機GitLab版本需與原主機架設版本相同。

```bash
# 查看GitLab版本
sudo gitlab-rake gitlab:env:info
```

執行指令後需要等待一些時間，由執行後結果來看，目前主機GitLab版本為`14.5.2-ee`：

```bash
GitLab information
Version:        14.5.2-ee
Revision:       4511944420f
Directory:      /opt/gitlab/embedded/service/gitlab-rails
DB Adapter:     PostgreSQL
DB Version:     12.7
URL:            http://gitlab.example.com
HTTP Clone URL: http://gitlab.example.com/some-group/some-project.git
SSH Clone URL:  git@gitlab.example.com:some-group/some-project.git
Elasticsearch:  no
Geo:            no
Using LDAP:     no
Using Omniauth: yes
Omniauth Providers:
```

### Step2. 執行GitLab備份

確認好版本後，接下來執行GitLab備份指令，此備份指令依官方文件所述，僅適用於12.2以後的版本。

```bash
# 執行GitLab備份
sudo gitlab-backup create
```

在備份執行成功後，切換至`/var/opt/gitlab/backups`該目錄底下，可查看到備份檔案(副檔名為.tar)，此處我的備份檔案的名稱為: `1657127793_2022_07_07_14.5.2-ee_gitlab_backup.tar`。

若備份時遇到以下問題:

```bash
2022-07-05 15:32:25 +0800 -- Dumping database ...
rake aborted!
Errno::EACCES: Permission denied @ dir_s_mkdir - /var/opt/gitlab/backups
/opt/gitlab/embedded/service/gitlab-rails/lib/backup/database.rb:28:in `dump'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/backup.rake:136:in `block (4 levels) in <top (required)>'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/backup.rake:12:in `block (3 levels) in <top (required)>'
/opt/gitlab/embedded/bin/bundle:23:in `load'
/opt/gitlab/embedded/bin/bundle:23:in `<main>'
Tasks: TOP => gitlab:backup:db:create
(See full trace by running task with --trace)
```

則將`/var/opt/gitlab/backups`資料夾權限開給`git`使用者：

```bash
chown -R git:git /var/opt/gitlab/backups
```

* 解法參考來源: [stackoverflow: gitlab dosent restore backup](https://stackoverflow.com/questions/57769551/gitlab-dosent-restore-backup)


### Step3. 將備份檔案傳至異地主機

利用SSH指令將備份檔案傳至異地主機，此處已與異地主機先建立好SSH金鑰，不用輸入密碼即可連線，方便待會可以設定排程，讓主機自動執行備份。另外，需要注意的是，要記得用`sudo`來建立SSH金鑰，因為待會執行定期排程指令時會用`root`身份來執行。

* SSH金鑰建立方式，可參考此篇文章: [SSH 公開金鑰認證：不用打密碼登入 Linux 設定教學，安全又方便](https://blog.gtwang.org/linux/linux-ssh-public-key-authentication/)

接下來執行scp指令傳到異地主機資料夾：

```bash=
sudo scp /var/opt/gitlab/backups/[備份檔案名稱] [使用者]@[主機IP]:[備份資料夾路徑]/[備份檔案名稱]
```


### Step4. 建立crontab排程做定期備份

上述流程執行皆沒問題後，接下來撰寫排程要執行的檔案`gitlab_backup.sh`，以下程式碼是我自己撰寫，可依需求自行調整：

```bash
# 執行GitLab備份
gitlab-backup create

# 取得GitLab backups資料夾內最新的備份檔案名稱
filename=`ls -Art /var/opt/gitlab/backups/ | tail -n 1`

# 將備份檔案傳至異地主機
scp /var/opt/gitlab/backups/$filename [使用者]@[主機IP]:[備份資料夾路徑]/$filename

# 將GitLab帳密及相關設定傳至異地主機
scp /etc/gitlab/gitlab-secrets.json [使用者]@[主機IP]:[備份資料夾路徑]/gitlab-secrets.json
scp /etc/gitlab/gitlab.rb [使用者]@[主機IP]:[備份資料夾路徑]/gitlab.rb

# 若GitLab backups資料夾內檔案超過兩個以上 則刪除1天以前的備份檔案 以節省硬碟空間
declare -i fileNums=`ls /var/opt/gitlab/backups | wc -l`
declare -i reqNums=2
if ((fileNums >= reqNums)); then
    find /var/opt/gitlab/backups -type f -mtime +1 -name '*.tar' -delete
fi
```

執行檔案撰寫好後，在crontab上建立執行定期備份排程。

```bash
# 編輯排程(記得用sudo 以root權限執行)
sudo crontab -e

# 於crontab上新增排程
# 每天凌晨02:00執行
0 2 * * * /home/pi/gitlab_backup.sh
```

## 三、還原流程

### Step1. 前置作業
* 建立還原主機(此處以Ubuntu20.04版本為例)。
* 將備份檔案由異地主機複製到還原主機：
    * 1657127793_2022_07_07_14.5.2-ee_gitlab_backup.tar
    * gitlab-secrets.json
    * gitlab.rb

### Stpe2. 安裝GitLab

在還原主機上安裝GitLab時，注意要與原主機的GitLab的版本相同。([Ubuntu安裝GitLab官方安裝說明](https://about.gitlab.com/install/#ubuntu))。先前版本下載位置可至[GitLab官方安裝檔倉庫](https://packages.gitlab.com/gitlab/)尋找，本範例Gitlab下載安裝檔名稱為`gitlab-ee_14.5.2-ee.0_amd64.deb`。

```bash
# 安裝相關Linux套件
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
sudo apt-get install -y postfix

# 設定GitLab repository(由GitLab官方安裝檔倉庫取得連結)
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

# 執行安裝指令
sudo apt-get install gitlab-ee=14.5.2-ee.0

# 開始執行GitLab
sudo gitlab-ctl reconfigure
```

### Step3. 執行還原指令

在還原主機安裝好GitLab後，開始執行還原：

```bash
# 搬移GitLab備份檔案至 /var/opt/gitlab/backups/ 資料夾底下
sudo mv 1657127793_2022_07_07_14.5.2-ee_gitlab_backup.tar /var/opt/gitlab/backups/

# 更改檔案權限改git使用
sudo chown git:git /var/opt/gitlab/backups/1657127793_2022_07_07_14.5.2-ee_gitlab_backup.tar

# 關閉GitLab資料
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq

# 確認GitLab狀況
sudo gitlab-ctl status

# 執行Gitlab還原指令
sudo gitlab-backup restore BACKUP=1657127793_2022_07_07_14.5.2-ee

# 將備份的 gitlab-secrets.json 和 gitlab.rb 檔案移至還原主機Gitlab目錄下
sudo mv gitlab-secrets.json /etc/gitlab/gitlab-secrets.json
sudo mv gitlab.rb /etc/gitlab/gitlab.rb

# 重新啟動GitLab
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
sudo gitlab-rake gitlab:check SANITIZE=true
```

執行上述還原指令後，即可連至在還原主機建立的GitLab上，以原本主機的帳密登入，並且可以看到還原後的專案內容。

若還原測試完成後，想要移除還原機上的GitLab，則執行以下指令即可刪除GitLab：

```bash
# 關閉GitLab服務
sudo gitlab-ctl stop

# 移除GitLab
sudo apt-get remove gitlab-ee 
```

## 四、參考資料

* [GitLab官方文件: Back up and restore GitLab](https://docs.gitlab.com/ee/raketasks/backup_restore.html#back-up-and-restore-gitlab)
* [GitLab官方文件: Back up GitLab](https://docs.gitlab.com/ee/raketasks/backup_gitlab.html)
* [GitLab官方文件: Restore GitLab](https://docs.gitlab.com/ee/raketasks/restore_gitlab.html)


