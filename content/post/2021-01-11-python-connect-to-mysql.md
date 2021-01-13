---
title: Python連線MySQL方法說明
author: Yen-Ting, Su
date: '2021-01-11'
slug: python-connect-to-mysql
categories:
  - Programming
tags:
  - Python
  - MySQL
description: "本篇文章是紀錄如何用Python程式碼來連線MySQL資料庫，將以mysql-connector-python套件來進行實作。"
---

本篇文章是紀錄如何用Python程式碼來連線MySQL資料庫。

## Step1. 安裝mysql-connector-python套件

在專案虛擬環境的終端機上，執行pip安裝語句：

```python=
pip install mysql-connector-python
```

![](https://i.imgur.com/7Ry2TqZ.png)

mysql-connector-python套件為MySQL官方針對Python出的，只需要pip裝完後即可連線至MySQL Server，不需要在電腦上安裝其他東西。

* [GitHub: mysql-connector-python套件](https://github.com/mysql/mysql-connector-python)。


## Step2. Python程式碼查詢MySQL範例

連線至MySQL資料庫程式碼參考自[w3Schools頁面](https://www.w3schools.com/python/python_mysql_select.asp)。

此處以連線到本地端的MySQL資料庫為例，Python的程式碼為：

```python=
# 載入套件
import mysql.connector

# 建立MySQL連線
conn = mysql.connector.connect(
    host='localhost',           # 連線主機名稱
    user='user',                # 登入帳號
    password='@MySQLPassword',  # 登入密碼
)
cursor = conn.cursor()

# 使用資料庫
cursor.execute("USE world;")

# 查詢資料
cursor.execute("SELECT * FROM city")

# 將查詢資料傳至Python
result = cursor.fetchall()

# 印出查詢結果
for x in result:
    print(x)

# 關閉資料庫連線
conn.close()
```

* **說明**

1. 資料庫與密碼記得要改成自己資料庫的設定。

2. cursor.execute()內接的是MySQL程式碼，除了本範例提供的select語句外，其他指令如insert、update、delete等，也可直接在Python內輸入MySQL語句使用。

3. 在Python內操作MySQL的程式碼可參考此篇文章：[[Python 使用 MySQL Connector 操作 MySQL/MariaDB 資料庫教學與範例]](https://officeguide.cc/python-mysql-mariadb-database-connector-tutorial-examples/)

## 補充：其他資料庫連線套件

* Python連線Oracle可參考：[Oracle官方文件說明](https://cx-oracle.readthedocs.io/en/latest/user_guide/installation.html)

* Python連線MS-SQL可參考：[pymssql套件說明](https://pymssql.readthedocs.io/en/stable/pymssql_examples.html)

