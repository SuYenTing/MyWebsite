---
title: 專題作業-職缺小幫手
author: Yen-Ting, Su
date: '2021-02-28'
slug: dash-job-analysis-helper
categories:
  - Programming
tags:
  - Python
  - Dash
description: "這是我獨力製作的一個小專題，專題內容是一個簡單的職缺分析網站。使用者可在此網站上輸入想要搜尋的職缺關鍵字後，系統會至104人力銀行搜尋相關職缺，並將職缺分析結果呈現給使用者觀看。"
---


## 一、專題簡介

這是我獨力製作的一個小專題，專題內容是一個簡單的職缺分析網站。使用者可在此網站上輸入想要搜尋的職缺關鍵字後，系統會至104人力銀行搜尋相關職缺，並將職缺分析結果呈現給使用者觀看。

* 專案相關的程式碼可參考我的[Github](https://github.com/SuYenTing/job_analysis_helper)
* 網站畫面
    * 使用者輸入想要搜尋的職缺及爬取的搜尋頁數後，網站會去執行爬蟲程式，並在畫面上呈現目前爬蟲程式運作的狀況：
![](https://i.imgur.com/QZoA3hk.png)
    * 使用者在查詢職缺後，可以觀看該職缺的分析結果，此處主要呈現一些資料探索式分析的結果：
![](https://i.imgur.com/SbbvCDi.png)
![](https://i.imgur.com/MmeD06n.png)

網站主要是透過Python的Dash套件製作，資料庫為MySQL，另外我有包成docker-compose，能夠直接快速部署。

我會選擇用Dash的原因是它和我之前學的RStudio邏輯很像，可以快速部署網頁內容與視覺化呈現。

> 這篇文章有對Dash與Rstudio兩個工具進行比較。
> [Python Dash vs. R Shiny – Which To Choose in 2021 and Beyond](https://appsilon.com/dash-vs-shiny/)

---

## 二、Dash簡介

> Dash 是建構於 Plotly.js、React.js 與 Flask 之上的 Python 網頁應用程式框架，能夠將常見的使用者介面元件包含像是下拉式選單、滑桿或圖形>與 資料分析應用快速地連結起來，讓以 Python 為主的資料科學團隊不需要 JavaScript 也可以建立出具備高度互動性的圖表與儀表板。
> (文字來源：[數據交點-互動式圖表及 Python](https://www.datainpoint.com/data-science-in-action/19-communicating-with-dash.html#%E9%97%9C%E6%96%BC-plotly-%E8%88%87-dash))

我自己在學習Dash主要是參考[Plotly-Dash官方教學手冊](https://dash.plotly.com/)，裡面共有6個章節可進行學習，把裡面的範例都看過一遍就可以開始實作Dash：
![](https://i.imgur.com/U36TXdg.png)

此處以Dash官方教學手冊的[Dash App Layout With Figure and Slider](https://dash.plotly.com/basic-callbacks)範例，來簡單說明一下Dash運作的方式，程式碼執行後會產生互動式圖表：

```python=
# 載入Dash相關套件
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

# 載入Plotly繪圖套件
import plotly.express as px

# 載入pandas資料整理套件
import pandas as pd

# 讀取plotly提供的範例csv檔案
# 此檔案包含各國家在不同年代的人口數、預期壽命與人均GDP等資訊，為一個Panel Data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminderDataFiveYear.csv')

# 要載入的css套件
external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

# 建立一個Dash的app
app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

# Dash-前台：設定Dash的版面(app.layout)
app.layout = html.Div([

    # 互動式圖表放置處，此處有命名id為graph-with-slider
    # 這個id很重要，要讓後台辨識用的
    dcc.Graph(id='graph-with-slider'),
    
    # 設定一個slider，讓使用者能夠拖曳年份
    # 後台依據此slider目前選擇的年份繪製對應的圖表資訊
    dcc.Slider(
        # slider的id，後台辨識用的
        id='year-slider',        
        # 使用者能夠拖曳slider最小值
        min=df['year'].min(),
        # 使用者能夠拖曳slider最大值
        max=df['year'].max(),
        # slider的默認值
        value=df['year'].min(),
        # slider的選項名稱
        marks={str(year): str(year) for year in df['year'].unique()},
        step=None
    )
])


# Dash-後台
# callback函數：
# 輸入(Input)為前台畫面id='year-slider'的值，即使用者現在選取Slider的年份
# 輸出(Output)為圖表放置到前台畫面id='graph-with-slider'的位置
# 當Input有更動時，此函數即會執行，並將運作完的結果返回到Output
# 在這個範例中，當使用者調整Slider的年份，後台察覺前台有變化，
# 即會執行update_figure函數，依選取年份繪製出的圖形後，
# 輸出到前台畫面的圖片放置處
@app.callback(
    Output('graph-with-slider', 'figure'),
    Input('year-slider', 'value'))
def update_figure(selected_year):

    # 選取前台使用者選取的年份
    filtered_df = df[df.year == selected_year]

    # 繪製圖表，此處繪製散佈圖，X軸為人均GDP，Y軸為預期壽命
    fig = px.scatter(filtered_df, x="gdpPercap", y="lifeExp",
                     size="pop", color="continent", hover_name="country",
                     log_x=True, size_max=55)

    # 降低圖片動畫速度
    fig.update_layout(transition_duration=500)

    # 輸出圖表，圖表放置在callback函數指定的Output id位置
    return fig

# 執行主程式
if __name__ == '__main__':
    app.run_server(debug=True)
```

從上述可知，在製作Dash時，想要給使用者看到的畫面內容放置在`app.layout`內，需要互動的部分，則透過`callback function`來達成。

---

## 三、專題遇到的問題與解決方法

實作過程當然也不是那麼的順利，這邊我將說明我在這個專案遇到問題的部分，以及我如何處理。

### 問題1. 決定網站版型

由於Dash套件本身沒有提供Bootstrap，如果要有響應式設計(RWD)，需要額外安裝`dash-bootstrap-components`套件，[套件的官網](https://dash-bootstrap-components.opensource.faculty.ai/)有詳細的說明。

網站的版型我參考[官網的Example](https://dash-bootstrap-components.opensource.faculty.ai/examples/)，選擇[Simple sidebar](https://dash-bootstrap-components.opensource.faculty.ai/examples/simple-sidebar/)來模板來進行專題開發。

### 問題2. 文字雲圖片輸出到Dash畫面做法

依據wordcloud套件的[範例程式碼](https://amueller.github.io/word_cloud/auto_examples/simple.html#sphx-glr-auto-examples-simple-py)，在Jupyter上執行文字雲的程式碼，可以直接輸出圖片。但如果在Dash中要將文字雲圖片透過callbacks傳回到前台，需要在進行額外的處理。

此處我參考stackoverflow的問題解法([how to show wordcloud image on dash web application](https://stackoverflow.com/questions/58907867/how-to-show-wordcloud-image-on-dash-web-application))，透過Base64編碼和解碼圖片，可順利執行出來。

### 問題3. 點選「查詢」按鈕後才動作

Dash中的callback只要接受到input的改變，就會馬上進函數運算後改變output。如果想要讓使用者按下「查詢」按鈕後才執行函數，可以參考[Dash官網的範例: Button Basic Example](https://dash.plotly.com/dash-html-components/button)。

在[Basic Dash Callbacks](https://dash.plotly.com/basic-callbacks)教學頁中的Dash App With State章節也有提及，主要是在callback引數中利用`State()`函數來避免被馬上執行，而按鈕的按鈕次數(n_clicks)做為`Input()`，當使用者按下按鈕，按鈕次數一改變時，即會執行函數。

### 問題4. 下拉式選單動態選項

在職缺分析頁面，有提供一個下拉式的選單，這個選單的選項是要依據資料庫目前有存放的查詢資訊來呈現：

![](https://i.imgur.com/2Qqs8qF.png)

此問題的處理方法我用簡單的程式碼來作範例，我依官網的[Dropdown範例](https://dash.plotly.com/dash-core-components/dropdown)進行修改，主要是透過後台的callback函數來更新前台的下拉式選單選項：

```python=
import random
import dash
import dash_html_components as html
import dash_core_components as dcc

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

# 前台畫面設定
app.layout = html.Div([
    dcc.Dropdown(id='demo-dropdown'),
    html.Button(id='submitButton', children='點我改變選項')
])


@app.callback(dash.dependencies.Output('demo-dropdown', 'options'),
              dash.dependencies.Output('demo-dropdown', 'value'),
              dash.dependencies.Input('submitButton', 'n_clicks'))
def update_dropdown(value):

    # 產生5個隨機值
    randomList = random.sample(range(1, 100), 5)

    # 建立隨機5個選項
    options = [{'label': '選項' + str(i), 'value': i} for i in randomList]

    # 設定默認值
    value = min(randomList)

    # 返回dropdown選項與默認值
    return options, value


if __name__ == '__main__':
    app.run_server(debug=True)
```

### 問題5. Loading圖示功能

有時候後台計算的時間會比較長，為讓使用者知道後台正在計算而不是網站當掉，此時就需要有Loading的圖示來提醒使用者。

Dash在Loading的部分有提供[dcc.Loading()](https://dash.plotly.com/dash-core-components/loading)函數，而在Dash Bootstrap套件中，則有[dbc.Spinner()](https://dash-bootstrap-components.opensource.faculty.ai/docs/components/spinner/)函數。

### 問題6. 輸出爬蟲程式運作資訊

由於爬蟲執行的時間會較長，為讓使用者能夠知道目前程式的執行進度，需要將目前程式執行的狀況輸出到前端畫面。

處理方法是每當使用者按下查詢按鈕時，會建立一個依當下timestamp檔名的log檔案，接下來爬蟲程式執行時，會將執行的資訊儲存至該log檔案。

而在前台部分，主要透過[dcc.Interval()](https://dash.plotly.com/dash-core-components/interval)函數，來定時讀取log檔案內的資訊，並將結果返回至前端。


---

## 四、結語

這個專案大概做三個禮拜的時間，前期主要是撰寫爬蟲程式碼及探索式資料分析，後期則是學習如何撰寫Dash，提供使用者UI介面進行操作。

目前的專案我發現還有以下問題需要在改善：

1. 使用者在執行職缺分析時，網站反應的速度很慢。

    這是因為每次執行時，會先依據使用者給的條件到資料庫撈資料出來(花時間)，然後還要在後台計算後(更花時間)，才會把結果呈現到前端。

    目前我想到的改進方法，或許可以採用Redis來處理。在使用者爬蟲完後，先行計算職缺分析結果，並放到記憶體內儲存，這樣使用者在查詢時，可以快速取得資料。

2. 多位使用者同時執行職缺爬蟲時，畫面呈現的log紀錄還是會互相干擾。

    這部分目前我還在思考要如何處理，可能要先仔細去研究Python的logging套件細節。

