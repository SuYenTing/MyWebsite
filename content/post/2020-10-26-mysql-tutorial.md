---
title: MySQL基本介紹
author: Yen-Ting, Su
date: '2020-10-26'
slug: mysql-tutorial
categories:
  - Programming
tags:
  - MySQL
description: "此篇文章為MySQL基本介紹，這是我在中山財管為學弟妹上課的講義，該課程共有3小時的時間，參與人數約25人左右。"
---

![](class.jpg)
圖片說明：上課照片


* MySQL安裝簡報：[下載位置](https://github.com/SuYenTing/Quantitative_investment_material_in_R/blob/master/MySQL/20201022_MySQL%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8.pdf)。

## 一、MySQL介紹

* [ORACLE官網定義](https://www.oracle.com/tw/database/what-is-database.html)

> 資料庫是一個對結構化資訊或資料的組織性收集，通常以電子方式儲存在電腦系統。資料庫通常由資料庫管理系統(DBMS)控制。資料和DBMS，以及和它們相關的應用程式，統稱為資料庫系統，通常又簡稱資料庫。
目前運行中最常見的資料庫型態通常是在一系列的表格中進行行列間建模，使得處理和資料查詢更為有效。如此一來，資料就可以很容易取得、管理、修改、更新、控制和編組。大部分的資料庫使用結構化查詢語言來編寫或查詢資料。 

* MySQL資料庫最新排名：目前第二 (Ref: [DB-Engines Ranking](https://db-engines.com/en/ranking))

![](https://i.imgur.com/moYUUZq.png)

* 基本上使用完全免費，沒有任何限制

* 推薦課程： 
    * MIS512 資料庫系統 黃三益 (資管碩)
    * MIS205 資料庫管理 魏春旺 (資管系)

* 推薦書籍：[MySQL新手入門超級手冊-第二版(適用MySQL 8.x與MariaDB 10.x)](https://www.books.com.tw/products/0010792087?sloc=main)

------

## 二、建立資料表

此章節我們將學習如何匯入資料至資料庫內。此處我們將以調整後股價及籌碼面資料做範例。

### Step1. 下載TEJ特殊轉檔資料

我們先至TEJ資料庫特殊轉檔下載待會要做範例的數據集：

* 股票範圍：上(下)市普通股
* 日期區間：為2020/01/01至2020/10/23

* 調整後股價(日頻)特殊轉檔(輸出檔案命名為：adj_stock_price_data.txt)
![](https://i.imgur.com/xKJOirm.png)

* 籌碼面資料特殊轉檔(輸出檔案命名為：institutional_data.txt)
![](https://i.imgur.com/7av0Wvw.png)

由於TEJ資料庫是用Big5編碼儲存資料，但我們建議資料庫還是以UTF8儲存會比較好，所以此處先對下載好的檔案做編碼轉換。編碼轉換的方式很簡單，只要直接打開資料後，在另存新檔時選取要儲存的編碼即可。

![](https://i.imgur.com/Tzgih8J.png)

### Step2. 建立資料庫與資料表

整理好資料後，接下打開MySQL Workbench開始建立資料庫與資料表。

* 建立資料庫流程：

![](https://i.imgur.com/q07kKvT.png)

* 建立資料表流程：

    * 調整後股價資料表相關設定
    ![](https://i.imgur.com/6iUwYwD.png)

    * 三大法人籌碼面資料表相關設定
    ![](https://i.imgur.com/azDoevq.png)

若想要更深入了解欄位變數型態，可參考此頁面[SQL Data Types for MySQL, SQL Server, and MS Access](https://www.w3schools.com/sql/sql_datatypes.asp)

### Step3. 匯入數據

匯入資料程式碼如下所示。需要特別注意的是，要匯入的檔案路徑**不能有中文字串**，且路徑需為**左斜線**。

```sql=
--- 匯入調整後股價資料表 ---
load data local infile 'C:/Users/adm/Desktop/adj_stock_price_data.txt'
into table stock_market.adj_stock_price_data LINES TERMINATED BY '\r\n';

--- 匯入調整後股價資料表 ---
load data local infile 'C:/Users/adm/Desktop/institutional_data.txt'
into table stock_market.institutional_data LINES TERMINATED BY '\r\n';

--- 格式說明 ---
load data local infile '[檔案存放路徑]' 
into table [資料表名] LINES TERMINATED BY '\r\n';
```

若執行上述語句時，會發現MySQL報3948錯誤代碼：

![](https://i.imgur.com/WH4MTDq.png)

這個原因是MySQL在8.0版本有提升安全性要求，對`load data local`指令有限制。在5.7版本原本是個很方便好用的指令，結果8.0版本要用時被限制，而且還不知道要怎麼解除，這個問題一直被討論，有興趣可參考此[頁面](https://bugs.mysql.com/bug.php?id=91891)。當然最後官方團隊有提供解決方法。

總共需要做兩個處理，第一個在MySQL登入畫面時，進行連線設定：

![](https://i.imgur.com/MTKBaYr.png)

接下來在Connection -> Advanced -> Others 的框中，新增`OPT_LOCAL_INFILE=1` 參數設定，新增好後按Close然後連線至資料庫。

![](https://i.imgur.com/STRPncI.png)

第二個處理為在查詢區中輸入以下程式碼：

```sql=
--- 開啟local_infile功能 ---
SET GLOBAL local_infile=1;

--- 查詢local_infile參數值 ---
SHOW VARIABLES LIKE "local_infile";
```

![](https://i.imgur.com/SR25uZC.png)

若local_infile的值為ON，代表設定成功。

經過上述兩個步驟，就可以開啟load data local的功能，可以順利匯入資料，且只要設定這一次就好，之後就不用再重跑。

我們重新執行以下指令：

```sql=
--- 匯入調整後股價資料表 ---
load data local infile 'C:/Users/adm/Desktop/adj_stock_price_data.txt'
into table stock_market.adj_stock_price_data LINES TERMINATED BY '\r\n';

--- 匯入調整後股價資料表 ---
load data local infile 'C:/Users/adm/Desktop/institutional_data.txt'
into table stock_market.adj_stock_price_data LINES TERMINATED BY '\r\n';

--- 查詢匯入資料狀況 ---
SELECT * FROM stock_market.adj_stock_price_data;
SELECT * FROM stock_market.institutional_data;
```

查詢資料後，會發現欄位的名稱也被一併匯入資料庫：

![](https://i.imgur.com/IAyKZ6N.png)

所以要把欄位名稱刪除，執行以下指令：

```sql=
--- 解除安全限制 才能使用delete from語句 ---
SET SQL_SAFE_UPDATES=0;

--- 刪除日期為0之資料 ---
delete from stock_market.adj_stock_price_data where date=0;
delete from stock_market.institutional_data where date=0;

--- 查詢資料表 ---
SELECT * FROM stock_market.adj_stock_price_data;
SELECT * FROM stock_market.institutional_data;
```

查詢後可發現欄位名稱已被刪除：

![](https://i.imgur.com/CXHxwKV.png)

另外還有需要要注意的一點，資料內容會有空白的狀況。在某筆資料列按右鍵選擇Copy Row後，貼在指令區會發現股票代碼及名稱有空白，這個空白必需要消除避免R程式在併表時發生錯誤。

![](https://i.imgur.com/Pa9afSU.png)

我們針對股票代碼及股票名稱消除空白，消除的指令為：

```sql=
--- 刪除調整後股價表股票代碼及股票名稱空白格 ---
UPDATE stock_market.adj_stock_price_data SET code = TRIM(code) WHERE code LIKE '% %';
UPDATE stock_market.adj_stock_price_data SET name = TRIM(name) WHERE name LIKE '% %';

--- 刪除三大法人籌碼資料股票代碼及股票名稱空白格 ---
UPDATE stock_market.institutional_data SET code = TRIM(code) WHERE code LIKE '% %';
UPDATE stock_market.institutional_data SET name = TRIM(name) WHERE name LIKE '% %';
```

執行完以上指令後，重新再做一次Copy Row，可發現目標欄位空白皆已被消除。

![](https://i.imgur.com/yjMUnK9.png)

以上就是MySQL資料庫匯入數據的做法。

------

## 三、MySQL常用程式碼介紹

資料庫有資料後，我們就可以開始介紹MySQL資料整理的程式碼。

### 查詢： Select

```sql=
--- 查詢資料表 ---
select * 
from stock_market.adj_stock_price_data;

--- 查詢資料表指定欄位 ---
select code, name, date, close 
from stock_market.adj_stock_price_data;

--- 查詢資料表指定欄位不重複值 ---
select distinct code 
from stock_market.adj_stock_price_data;

select distinct code, name 
from stock_market.adj_stock_price_data;
```

### 排序： Order by

```sql=
--- 由小到大排序 ---
select * 
from stock_market.adj_stock_price_data 
order by close;

select * 
from stock_market.adj_stock_price_data 
order by close asc;

--- 由大到小排序 ---
select * 
from stock_market.adj_stock_price_data 
order by close desc;

--- 先排日期由小到大再排收盤價由大到小 ---
select * 
from stock_market.adj_stock_price_data 
order by date, close desc;
```

### 篩選條件： Where

```sql=
--- 篩選條件 ---
select * 
from stock_market.adj_stock_price_data 
where date > 20201001;

select * 
from stock_market.adj_stock_price_data 
where date = 20201005;

select * 
from stock_market.adj_stock_price_data 
where date <= 20201005;

--- 篩選條件or寫法 ---
select * 
from stock_market.adj_stock_price_data 
where date = 20201005 or date = 20201012 
order by code, date;

--- 篩選條件in寫法(等同or) ---
select * 
from stock_market.adj_stock_price_data 
where date in (20201005, 20201012) 
order by code, date;

--- 篩選條件and寫法 ---
select * 
from stock_market.adj_stock_price_data 
where name in ('台積電', '大立光', '中華電') and date = 20201005 
order by code;

--- 篩選條件and和or同時用寫法 ---
select * 
from stock_market.adj_stock_price_data 
where name in ('台積電', '大立光', '中華電') and (date = 20201005 or date = 20201012) 
order by code, date desc;
```

### 限制設定：Limit

```sql=
--- 選出20201019前10名之股票 ---
select * from stock_market.adj_stock_price_data 
where date = 20201019 
order by close desc 
limit 10;

--- 選出整個資料表最新10個交易日日期 ---
select distinct date 
from stock_market.adj_stock_price_data 
order by date desc limit 10;
```

### 數學計算

```sql=
--- 計算每檔股票一張的價錢 ---
select code, name, date, close*1000 as price 
from stock_market.adj_stock_price_data 
where date = 20201019;

--- 計算每檔股票振幅大小 ---
select *, (high-low)/close as price_range 
from stock_market.adj_stock_price_data 
where date = 20201019 
order by price_range desc;
```

### 小試身手1

1. 請以股票代碼搜尋2303, 2344, 2603三檔股票在2020/10/19的資訊，並且按收盤價由大到小排序。
2. 請找出2020/10單日成交量最高前十名的股票代碼、名稱、日期、成交量(資料表只要這3個欄位)。
3. 請算出台積電在2020/10期間每個交易日的均價(開高低收平均)，並按時間由最新到最舊排序

### 函數運算

```sql=
--- 函數運算:字串位置取法範例 ---
select *, 
    left(date, 4) as dYear, 
    substring(date, 5, 2) as dMonth, 
    right(date, 2) as dDay 
from stock_market.adj_stock_price_data;

--- 函數運算範例  ---
select code, name, date, close, 
    round(close, 2) as round_close, 
    floor(close) as floor_close, 
    ceiling(close) as ceiling_close, 
    log(close) as log_close, 
    round(log(close), 2) as rlog_close
from stock_market.adj_stock_price_data;
```

### 流程控制: If與Case when

```sql=
--- if收盤價大於500視為高股價 ---
select *, if(close > 500, '高股價', '低股價') as class 
from stock_market.adj_stock_price_data 
where date = 20201019;

--- case when對收盤價進行分群---
select *, 
    case 
	when close < 50 then 'A'
	when close < 100 then 'B'
	when close < 500 then 'C'
	else 'D'
    end as class
from stock_market.adj_stock_price_data
where date = 20201019;
```

### 群組函式: Group by與Having

```sql=
--- 計算台股每日股票檔數、總成量、均量及個股最大量、最小量 ---
select date, 
    count(code) as stock_nums,
    sum(trade_volume) as total_volume, 
    avg(trade_volume) as avg_volume,
    max(trade_volume) as max_volume,
    min(trade_volume) as min_volume
from stock_market.adj_stock_price_data 
group by date 
order by date desc;

--- 計算台股2020/09月平均收盤價 ---
select code, name, avg(close) as avg_close
from stock_market.adj_stock_price_data 
where date >= 20200901 and date <= 20200930
group by code, name 
order by code;

--- 計算台股2020/09月平均收盤價且月平均收盤價小於10元 ---
select code, name, avg(close) as avg_close
from stock_market.adj_stock_price_data 
where date >= 20200901 and date <= 20200930
group by code, name 
having avg_close < 10
order by code;
```

### 子查詢

```sql=
--- 子查詢表: 找出高股價分類股票 ---
select * from (
    select *, if(close > 500, '高股價', '低股價') as class 
    from stock_market.adj_stock_price_data 
    where date = 20201019) as t 
where class = '高股價' order by close desc;

--- 子查詢條件: 找出台泥最新10日股價資訊 ---
select * 
from stock_market.adj_stock_price_data 
where code = 1101 
and date in (
    select date from (
        select distinct date 
        from stock_market.adj_stock_price_data 
        order by date desc limit 10) as t) 
order by date;
```

### 併表

![](https://i.imgur.com/IvkvCB4.png)

併表程式碼可參考此[網頁說明](https://www.w3schools.com/sql/sql_ref_join.asp)，基本上我們最常用的就是Left Join。

```sql=
-- Inner join --
select * from stock_market.adj_stock_price_data as a
inner join stock_market.institutional_data as b
on a.code = b.code and a.date = b.date;

-- Inner join指定欄位 --
select a.code, a.name, a.date, a.close, b.total_net_buy 
from stock_market.adj_stock_price_data as a
inner join stock_market.institutional_data as b
on a.code = b.code and a.date = b.date;

-- Left join指定欄位 --
select a.code, a.name, a.date, a.close, b.total_net_buy 
from stock_market.adj_stock_price_data as a
left join stock_market.institutional_data as b
on a.code = b.code and a.date = b.date;
```

### 小試身手2

* 請找出在2020/10月中，日均量>5000張且外資累積總買賣超>10,000張之股票代碼。

### 匯入資料

除了前面以`load data local infile`匯入txt檔案至MySQL資料表為，我們也可以`inset into`指令來加入資料至資料表。

```sql=
--- 新增一筆資料至調整後股價資料表 ---
INSERT INTO stock_market.adj_stock_price_data 
(code, name, date, open, high, low, close, trade_volume, market_value)
VALUES 
('9999', '中山大學', 20201026, 0, 0, 0, 0, 0, 0);

--- 新增多筆資料至調整後股價資料表 ---
INSERT INTO stock_market.adj_stock_price_data 
(code, name, date, open, high, low, close, trade_volume, market_value)
VALUES 
('9999', '中山大學', 20201025, 0, 0, 0, 0, 0, 0),
('9999', '中山大學', 20201024, 0, 0, 0, 0, 0, 0),
('9999', '中山大學', 20201023, 0, 0, 0, 0, 0, 0);
```

### 清空資料表

在MySQL中，如果想把資料全部清除但保留資料欄位，可以使用`truncate`指令。

輸入程式碼：
```sql=
truncate stock_market.adj_stock_price_data;
```

也可以直接從MySQL Workbench直接操作：

![](https://i.imgur.com/eK2X2AO.png)

執行完之後，調整後股價表內的資料就會消失不見，但資料欄位還是會留著：

![](https://i.imgur.com/K1KNEtX.png)


### 查詢資料表欄位資訊

如果想要查詢資料表目前的欄位資訊，可以輸入：
```sql=
show full columns from stock_market.adj_stock_price_data;
```

### 查詢資料表建表程式碼

若想要查詢目前資料表建立的程式碼，可以輸入：

```sql=
show create table stock_market.adj_stock_price_data;
```

### 刪除資料表

若要刪掉整個資料表，則輸入：
```sql=
drop table stock_market.adj_stock_price_data;
```

或者也可透過MySQL Workbench直接操作：

![](https://i.imgur.com/qO2Jzku.png)

刪除後就看不到該資料表：

![](https://i.imgur.com/ysXHMVd.png)

如果你懂`drop table`語法，那看這張圖你就會笑：
![](https://i.imgur.com/pwxzxoz.jpg)

駭客攻擊資料庫手法：[SQL Injection](https://zh.wikipedia.org/wiki/SQL%E6%B3%A8%E5%85%A5)

------

## 四、MySQL程式對接(R、Python)

### R程式與MySQL對接程式範例

R與MySQL對接的套件為[RMySQL](https://github.com/r-dbi/RMySQL)。

以下是在R中連線至資料庫下載資料的範例：

```r=
# 讀取R套件
library(RMySQL)

# 建立連線
con <- dbConnect(dbDriver("MySQL"), 
                 host = "localhost",   # 資料庫IP位置
                 user = "root",        # 帳號
                 password = "mysql",   # 密碼
                 port = 3306)          # Port

# 進行編碼轉換
query <- "set names big5;"
dbSendQuery(con, query)

# 下載資料
query <- "select * from stock_market.adj_stock_price_data;"
res <- dbSendQuery(con, query)
data <- dbFetch(res, n = -1)

# 清除Fetch結果
dbClearResult(res)

# 結束連線
dbDisconnect(con)
```

### Python程式與MySQL對接程式範例

在Python程式中，利用pandas和pymysql即可自資料庫中下載資料。

```python=
# 載入套件
import pandas as pd
import pymysql

# 連線資料庫設定
def connect_mysql():  
    global connect, cursor
    connect = pymysql.connect(host = "localhost",
                              user = "root", 
                              password = "mysql",
                              port = 3306)
    cursor = connect.cursor()

# 下載資料
connect_mysql()   
stock = pd.read_sql("SELECT * FROM stock_market.adj_stock_price_data;", 
                    con = connect)

# 觀看資料結果
print(stock.head(10))

# 以下為直接用pymysql套件寫法
# 載入套件
import pymysql
# 建立連線
con = pymysql.connect(host="localhost", 
                      port=3306, 
                      user="root",
                      passwd="mysql", 
                      charset="utf8")
# 下載資料
query = "select * from stock_market.adj_stock_price_data;"
cursor = con.cursor()
cursor.execute(query)
data = cursor.fetchall()
print(data[1])
```

### 小試身手

請試著在R或Python中寫程式碼，傳入學號、姓名、留言及時間至指定的MySQL資料表。
Hint: 
* 利用SQL的`inesrt into`指令
* R的目前日期時間寫法： 
```r=
Sys.time()
```

* Python目前日期時間寫法：
```python=
from datetime import datetime
datetime.now()
```

------

## 五、資料庫索引

資料庫索引的優點是可以增加條件篩選(where)的速度，但因為要建立索引，所以必須要額外使用硬碟空間，且在新增/修改/刪除資料時，也必須要動態維護索引導致速度會變慢。

我們通常都會以常被where篩選的欄位來建立索引，以調整後股價資料表為例，常會使用股票代碼及日期來做篩選，因此針對這兩個欄位就會建立索引，以增加條件篩選的速度。

以下為索引相關的程式碼指令：

```sql=
--- 建立索引 ---
CREATE INDEX [索引名稱] ON [表名]([欄位名稱])

--- 查詢索引 ---
SHOW INDEX FROM [表名];

--- 刪除索引 ---
DROP INDEX [索引名稱] ON [表名]; 
```

接下來實際測試建立索引前後的查詢速度，此處以董監事持股狀況表為例，該資料表期間為2005/01至2020/09，共有10,168,107筆資料。假設我們想要搜尋台積電2330在201912的董監事持股資料，程式寫法為：

```sql=
SELECT * FROM test.director_share_holding where code = 2330 and date = 201912;
```

此查詢語句執行後速度為：10.703秒

![](https://i.imgur.com/uC6khIh.png)

我們為此表的code及date欄位添加索引並重新執行篩選查詢語句：

```sql=
CREATE INDEX code ON test.director_share_holding (code);
CREATE INDEX date ON test.director_share_holding (date);
SELECT * FROM test.director_share_holding where code = 2330 and date = 201912;
```

![](https://i.imgur.com/7OM8u4T.png)

加入索引後執行查詢的速度為：0.078秒

由上述範例中，在加入索引後查詢的速度由10.703秒變為0.078秒，速度有明顯的增加。

------

## 六、建立資料庫服務

若架設好資料庫，想要讓其他人可以連線一起使用，則需要做以下設定：

### Step1. 設定使用者登入帳密及權限

為資訊安全，此處除了設定帳密外，也會鎖定學校IP。

![](https://i.imgur.com/Qxjhnpy.png)

設定好帳號後，還需要給該帳號資料庫權限。此處我們只開放該使用者只能查詢stock_market資料庫。

![](https://i.imgur.com/gfYhp5w.png)

![](https://i.imgur.com/Bfdjo3w.png)

### Step2. 打開防火牆

為讓其他使用者能夠連入，還需要開啟伺服器端的防火牆。MySQL服務的默認Port為3306，需將此Port打開。通常會將服務的Port做修改(透過my.ini)，不要用默認的Port避免被輕易的攻擊，但此處只是範例所以還是用原本默認的Port。

防火牆設定部分，主要是去控制台找Windows Defender防火牆 -> 選擇進階設定 -> 點選輸入規則 -> 新增規則。

![](https://i.imgur.com/JDvtluI.png)

### Step3. 使用者連線設定

在客戶端連線部分，使用者在Workbench上點選新增連線按鈕，將連線資訊填入，包含資料庫IP、帳號、密碼及Port，設定好後即可連入使用。

![](https://i.imgur.com/v7uElxS.png)

---

## 七、課堂補充程式碼：

```sql=
--- 建立調整後股價資料表 ---
CREATE TABLE `stock_market`.`adj_stock_price_data` (
  `code` VARCHAR(10) NULL COMMENT '股票代碼',
  `name` VARCHAR(10) NULL COMMENT '股票名稱',
  `date` INT(8) NULL COMMENT '日期',
  `open` FLOAT NULL COMMENT '開盤價',
  `high` FLOAT NULL COMMENT '最高價',
  `low` FLOAT NULL COMMENT '最低價',
  `close` FLOAT NULL COMMENT '收盤價',
  `trade_volume` FLOAT NULL COMMENT '成交量',
  `market_value` FLOAT NULL COMMENT '市值')
COMMENT = '調整後股價資料表';

--- 建立籌碼面資料表 ---
CREATE TABLE `stock_market`.`institutional_data` (
  `code` varchar(10) DEFAULT NULL COMMENT '公司代碼',
  `name` varchar(10) DEFAULT NULL COMMENT '簡稱',
  `date` int DEFAULT NULL COMMENT '年月日',
  `foreign_net_buy` float DEFAULT NULL COMMENT '外資買賣超',
  `trust_net_buy` float DEFAULT NULL COMMENT '投信買賣超',
  `dealer_net_buy` float DEFAULT NULL COMMENT '自營商買賣超',
  `total_net_buy` float DEFAULT NULL COMMENT '合計買賣超'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='籌碼面資料';
```

---

## 八、刪除MySQL程式

若要刪除MySQL程式，可以使用MySQL Installer，卸載會比較方便且乾淨。

![](https://i.imgur.com/wQm7w4h.png)

![](https://i.imgur.com/J1TzGHv.png)






