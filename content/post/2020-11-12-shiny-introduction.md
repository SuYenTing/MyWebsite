---
title: Shiny基本介紹
author: Yen-Ting, Su
date: '2020-11-12'
slug: Shiny
categories:
  - Programming
tags:
  - Shiny
description: "此文章是我在中山財管為學弟妹上課的講義，該課程共有3小時的時間，參與人數約15人左右。文章內主要示範如何建立一個可以繪製美股技術分析圖形的Shiny程式。"
---

# 一、參考資料
1. 電子書：[Mastering Shiny](https://mastering-shiny.org/)  (Hadley Wickham著作)
2. 網頁介紹：[R 講題分享 – 利用 R 和 Shiny 製作網頁應用 (作者：Taiwan R User Group)](https://programmermagazine.github.io/201309/htm/article6.html)
3. Shiny作品集：[Shiny Gallery](https://shiny.rstudio.com/gallery/)
4. [Rstudio官網教學](https://shiny.rstudio.com/tutorial/written-tutorial/lesson1/)
5. R Shiny cheat sheet： [[網站頁面]](https://rstudio.com/resources/cheatsheets/) [[直接下載]](https://github.com/rstudio/cheatsheets/raw/master/shiny.pdf)

---

# 二、Shiny簡介

## 1. Shiny範例介紹
* Shiny簡單架構圖(出處:[R 講題分享 – 利用 R 和 Shiny 製作網頁應用 (作者：Taiwan R User Group)](https://programmermagazine.github.io/201309/htm/article6.html))
![](https://i.imgur.com/af62aRv.png)

以R Shiny最基本範例來做介紹，在R程式中輸入:

```r=
library(shiny)
runExample("01_hello")
```

執行後會跳出Shiny範例：

![](https://i.imgur.com/SQ44tgM.png)

以下為範例檔案的程式碼：

```r=
library(shiny)

# Define UI for app that draws a histogram ----
ui <- fluidPage(

  # App title ----
  titlePanel("Hello Shiny!"),

  # Sidebar layout with input and output definitions ----
  sidebarLayout(

    # Sidebar panel for inputs ----
    sidebarPanel(

      # Input: Slider for the number of bins ----
      sliderInput(inputId = "bins",
                  label = "Number of bins:",
                  min = 1,
                  max = 50,
                  value = 30)

    ),

    # Main panel for displaying outputs ----
    mainPanel(

      # Output: Histogram ----
      plotOutput(outputId = "distPlot")

    )
  )
)

# Define server logic required to draw a histogram ----
server <- function(input, output) {

  # Histogram of the Old Faithful Geyser Data ----
  # with requested number of bins
  # This expression that generates a histogram is wrapped in a call
  # to renderPlot to indicate that:
  #
  # 1. It is "reactive" and therefore should be automatically
  #    re-executed when inputs (input$bins) change
  # 2. Its output type is a plot
  output$distPlot <- renderPlot({

    x    <- faithful$waiting
    bins <- seq(min(x), max(x), length.out = input$bins + 1)

    hist(x, breaks = bins, col = "#75AADB", border = "white",
         xlab = "Waiting time to next eruption (in mins)",
         main = "Histogram of waiting times")

    })

}

# Create Shiny app ----
shinyApp(ui = ui, server = server)
```

* faithful資料集說明
```r=
?faithful
```

Waiting time between eruptions and the duration of the eruption for the Old Faithful geyser in Yellowstone National Park, Wyoming, USA.

A data frame with 272 observations on 2 variables.

| 欄位名稱 | 資料類型 | 說明 |
| :--------: | :--------: | :--------: |
| eruptions(噴發時間長度) | numeric | Eruption time in mins |
| waiting(下一次噴發等待時間) | numeric | Waiting time to next eruption (in mins) |


![](https://i.imgur.com/HBeXfhZ.png)
圖片來源: [https://en.wikipedia.org/wiki/Old_Faithful](https://en.wikipedia.org/wiki/Old_Faithful)

## 2. 小試身手
請繪製噴發時間長度(eruptions)的直方圖在Shiny上。

## 3. 呈現資料表
R shiny有提供呈現資料表的函數`renderDataTable`，但呈現表格的方式會不太好看，所以通常會使用`DT`套件，此套件說明可以參考此[頁面](https://rstudio.github.io/DT/shiny.html)。

接下來在剛剛的範例集中，呈現faithful的資料表給使用者觀看。

```r=
library(shiny)
library(DT)

# UI
ui <- fluidPage(
  
  HTML('<center>'),
  h3("Faithful資料表-DT版"),
  HTML('</center>'),
  dataTableOutput("DT_table"),
  
  hr(),
  
  HTML('<center>'),
  h3("Faithful資料表-Shiny版"),
  HTML('</center>'),
  tableOutput("shiny_table")
)

# Server
server <- function(input, output) {

  # DT版
  output$DT_table <- renderDataTable({
    datatable(faithful)
  })
  
  # Shiny版
  output$shiny_table <- renderTable({
    faithful
  })
}

# Create Shiny app
shinyApp(ui = ui, server = server)
```

---

# 三、Shiny版面介紹

Shiny版面說明可參考Hadley Wickham的[Mastering Shiny](https://mastering-shiny.org/basic-ui.html#layout)書籍。

以下介紹我自己常用的版型：

```r=
library(shiny)
library(shinythemes)

# UI
ui <- navbarPage("Shiny範例", id = "tabs",
                 
                 # 主題設定
                 theme = shinytheme("flatly"),
                 
                 #################### 主題1 ####################
                 tabPanel("主題1", value = "主題1",
                          
                          tabsetPanel(type = "tabs",
                                      
                                      tabPanel("頁籤1", value = "頁籤1",
                                               
                                               h3("這裡是頁籤1頁面")
                                      ),
                                      
                                      tabPanel("頁籤2", value = "頁籤2",
                                               h3("這裡是頁籤2頁面")
                                      )
                          )
                 ),
                 
                 #################### 主題2 ####################
                 tabPanel("主題2", value = "主題2",
                          
                          h3("這裡是主題2頁面"),
                          
                          sidebarLayout(
                            
                            sidebarPanel(
                              
                              h3("這裡是sidebarPanel")
                            ),
                            
                            mainPanel(
                              
                              h3("這裡是mainPanel")
                            )
                          )
                 ),
                 
                 
                 #################### 加入CSS修改字體 ####################
                 tags$head(tags$style(HTML("@import url('//fonts.googleapis.com/css?family=Noto+Sans+TC');
                                   body, h2, h3, center {font-family: 'Noto Sans TC', sans-serif;}")))
)

# Server
server <- function(input, output) {
  
}

# Create Shiny app
shinyApp(ui = ui, server = server)
```

* 補充: Shiny的主題設定可到此[頁面](https://rstudio.github.io/shinythemes/)來挑選。

---

# 四、Shiny Widgets介紹

Shiny範例中有滑桿(Slider)可以選擇直方圖要畫的bin時外，Shiny也有提供其他常見的小工具，可以參考[Shiny Widgets Gallery](https://shiny.rstudio.com/gallery/widget-gallery.html)。

接下我們以Shiny範例來實作不同的Shiny Widgets功能。

## 1. 以選擇框(Select box)來調整bins

此範例是呈現如何透過`selectInput`來讓使用者挑選bins。

```r=
library(shiny)

# UI
ui <- fluidPage(
  
  # 選擇框
  selectInput("bins", 
              label = h3("Please select bins:"), 
              choices = list("15" = 15, 
                             "20" = 20, 
                             "30" = 30), 
              selected = 15),
  
  # 呈現直方圖
  plotOutput(outputId = "distPlot")
)

# Server
server <- function(input, output) {
  
  output$distPlot <- renderPlot({
    
    x    <- faithful$waiting
    bins <- seq(min(x), max(x), length.out = as.numeric(input$bins) + 1)
    
    hist(x, breaks = bins, col = "#75AADB", border = "white",
         xlab = "Waiting time to next eruption (in mins)",
         main = "Histogram of waiting times")
    
  })
}

# Create Shiny app
shinyApp(ui = ui, server = server)
```

## 2. 從後台控制選項內容

從上面的範例可以看到我們是在前端設定好可以調整的選項，但有時候我們希望選項能夠依據資料狀況自動呈現，例如說根據資料樣本數來決定bins可以選擇的大小。此時可以在server端透過`updateSelectInput`函數來更新ui的選擇項目。

```r=
library(shiny)

# UI
ui <- fluidPage(
  
  # 選擇框
  selectInput("bins", 
              label = h3("Please select bins:"), 
              choices = NULL, 
              selected = NULL),
  
  # 呈現直方圖
  plotOutput(outputId = "distPlot")
)

# Server
server <- function(input, output, session) {
  
  # 更新選項內容
  binsOption <- seq(5, 30, 5)
  binsOption <- split(unname(binsOption), binsOption)
  updateSelectInput(session, "bins", 
                    choices = binsOption,
                    selected = binsOption[[1]])
  
  output$distPlot <- renderPlot({
    
    x    <- faithful$waiting
    bins <- seq(min(x), max(x), length.out = as.numeric(input$bins) + 1)
    
    hist(x, breaks = bins, col = "#75AADB", border = "white",
         xlab = "Waiting time to next eruption (in mins)",
         main = "Histogram of waiting times")
    
  })
}

# Create Shiny app
shinyApp(ui = ui, server = server)
```

## 3. 以輸入值形式(Text input)來調整bins

此範例是呈現如何透過`textInput`讓使用者自行輸入bins。

```r=
library(shiny)

# UI
ui <- fluidPage(
  
  # 輸入對話框
  textInput("bins", label = h3("請輸入bins:"), value = "5"),
  
  # 呈現直方圖
  plotOutput(outputId = "distPlot")
)

# Server
server <- function(input, output, session) {
  
  output$distPlot <- renderPlot({
    
    x    <- faithful$waiting
    bins <- seq(min(x), max(x), length.out = as.numeric(input$bins) + 1)
    
    hist(x, breaks = bins, col = "#75AADB", border = "white",
         xlab = "Waiting time to next eruption (in mins)",
         main = "Histogram of waiting times")
    
  })
}

# Create Shiny app
shinyApp(ui = ui, server = server)
```

---

# 五、觸發事件寫法

在**以輸入值形式(Text input)來調整bins**範例中，我們透過`textInput`讓使用者自行輸入bins來決定繪製直方圖，但可能會發生兩個問題：

1. 使用者輸入不合適的值導致圖形無法繪製
2. 使用者還沒輸入完馬上跑出圖形

為解決上述問題，我們可以設定一個按鈕，當使用者輸入完想要呈現的值後，按下按鈕圖才會繪製。此處需要加入`actionButton`函數做按鈕及[`observeEvent`](https://shiny.rstudio.com/reference/shiny/1.0.1/observeEvent.html)搭配[`isolate`](https://shiny.rstudio.com/articles/isolation.html)做觸發事件。

```r=
library(shiny)

# UI
ui <- fluidPage(
  
  # 輸入對話框
  textInput("bins", label = h3("請輸入bins:"), value = "5"),
  
  # 建立繪製按鈕
  actionButton("action", label = "繪製"),
  
  # 呈現直方圖
  plotOutput(outputId = "distPlot")
)

# Server
server <- function(input, output, session) {
  
  observeEvent(input$action, {
    
    output$distPlot <- renderPlot({
      
      binNums <- as.numeric(isolate(input$bins))
      x    <- faithful$waiting
      bins <- seq(min(x), max(x), length.out = binNums + 1)
      
      hist(x, breaks = bins, col = "#75AADB", border = "white",
           xlab = "Waiting time to next eruption (in mins)",
           main = "Histogram of waiting times")
      
    })
  })
}

# Create Shiny app
shinyApp(ui = ui, server = server)
```

若使用者輸入錯誤的值，則提示輸入錯誤之寫法：

```r=
library(shiny)

# UI
ui <- fluidPage(
  
  # 輸入對話框
  textInput("bins", label = h3("請輸入bins(需介於5至30之間):"), value = "5"),
  
  # 建立繪製按鈕
  actionButton("action", label = "繪製"),
  
  # 輸出警告
  span(textOutput(outputId = "warningText"), style = "color:red; font-size:20px"),
  
  # 呈現直方圖
  plotOutput(outputId = "distPlot")
)

# Server
server <- function(input, output, session) {
  
  observeEvent(input$action, {
    
    # 讀取使用者輸入的值
    binNums <- as.numeric(isolate(input$bins))
    binNums <- ifelse(is.na(binNums), 0, binNums)
    
    # 判斷是否在合理範圍
    if((binNums >= 5) & (binNums <= 30)){
      
      # 位在合理範圍內清空警告提醒並繪圖 
      output$warningText <- renderText({NULL})
      output$distPlot <- renderPlot({
        
        x    <- faithful$waiting
        bins <- seq(min(x), max(x), length.out = binNums + 1)
        
        hist(x, breaks = bins, col = "#75AADB", border = "white",
             xlab = "Waiting time to next eruption (in mins)",
             main = "Histogram of waiting times")
      })
      
    }else{
      
      # 沒有在合理範圍內則輸出警告提醒及清空圖形
      output$warningText <- renderText({"請輸入正確的值域範圍唷!"})
      output$distPlot <- renderPlot({NULL})
    }
  })
}

# Create Shiny app
shinyApp(ui = ui, server = server)
```

---

# 六、上傳Shiny

做好的Shiny如果想讓別人能夠觀看使用(即對外服務)，可以上傳到R Studio的Shinyapps空間上或者自己架設R Shiny Server。

## 1. R Shinyapps介紹

可參考下列頁面資訊：
1. [Shinyapps.io官網](https://www.shinyapps.io/)
2. [shinyapps.io教學](https://docs.rstudio.com/shinyapps.io/index.html)

在Shinyapps.io官網登入後，即可看到官網說明上傳步驟：

![](https://i.imgur.com/fWZLq3l.png)

上述步驟接在R程式執行，步驟1及步驟2只要執行1次即可，要特別注意步驟2的token和secret資訊不能輕易給別人，否則他就可以操控你的帳戶。

步驟3是上傳Shiny的語法，將想要上傳的Shiny程式碼放在一個資料夾內，資料夾名稱即為專案名稱。Shiny程式碼的檔名請命名為**app.r**，一定要命名為這個，這樣才知道哪個是Shiny要執行的檔案。另外程式碼一定要是**utf8編碼**，否則上傳會出錯。

執行後即可在看到專案，點選網址即可看到剛製作的Shiny作品，其他人也可以透過此網址來觀賞你的Shiny作品。

![](https://i.imgur.com/NrYgbC2.png)

![](https://i.imgur.com/0bk4QcE.png)

如果想要刪除專案，先點選紅框左邊的盒子圖案(封存)，再按下右邊的垃圾桶圖案，即可刪除專案。

![](https://i.imgur.com/QNLymEA.png)


## 2. R Shiny Server

由於Shinyapps.io對於部分套件並不支援，所以如果想要使用其他套件的話，可以自己架設R Shiny Server來對外服務。

R Shiny Server架設只能夠在linux作業系統上，安裝流程可參考[Shiny官網說明](https://rstudio.com/products/shiny/download-server/)。

安裝完後記得要開啟防火牆3838的port(Shiny Server默認)，才能讓提供服務。

Shiny程式放在`/srv/shiny-server`資料夾路徑底下即可對外服務。

---

# 七、實作-股票技術分析圖形繪製網站

目標: 使用者輸入股票代碼後，呈現股票技術分析圖形給使用者觀看。

```r=
library(shiny)
library(lubridate)
library(DT)
library(shinythemes)
library(tidyquant)
library(tidyverse)

# UI
ui <- navbarPage("Shiny", id = "tabs", 
                 
                 theme = shinytheme("flatly"),
                 
                 tabPanel("繪製技術分析圖形", value = "繪製技術分析圖形",
                          
                          sidebarLayout(
                            
                            # 側邊頁面
                            sidebarPanel(
                              
                              # 輸入股票代碼
                              textInput("stockCode", label = h5("請輸入Yahoo Finance股票代碼:"), 
                                        value = "", placeholder = ""),
                              
                              # 查詢日期範圍
                              dateRangeInput("dates", label = h5("請選擇繪製的日期範圍"),
                                             start = Sys.Date()-days(90),
                                             end = Sys.Date(),
                                             min = Sys.Date()-years(5),
                                             max = Sys.Date(),
                                             separator = "至"),
                              
                              # 查詢按鈕
                              actionButton("action", label = "查詢"),
                              
                              # 輸出警告
                              br(),
                              span(textOutput(outputId = "warningText"), style = "color:red"),
                              
                              # 說明資料來源
                              br(),
                              tags$div(
                                "本網站數據來源: ",
                                tags$a(href = "https://finance.yahoo.com/", "Yahoo Finance")
                              ),
                            ),
                            
                            # 主頁面
                            mainPanel(
                              
                              HTML("<center>"),
                              h3("技術分析圖形"),
                              HTML("</center>"),
                              plotOutput("stockPlot"),
                              hr(),
                              HTML("<center>"),
                              h3("股價資訊表"),
                              HTML("</center>"),
                              dataTableOutput("stockData")
                            )
                          )
                 ),
                 
                 # 字體設定:使用CSS語法
                 tags$head(
                   tags$style(HTML("@import url('//fonts.googleapis.com/css?family=Noto+Sans+TC');
                                   body, h2, h3, h4, h5, center {font-family: 'Noto Sans TC', sans-serif;}")))
)

# Server
server <- function(input, output, session) {
  
  # 啟動查詢
  observeEvent(input$action, {
    
    # 讀取使用者設定資訊
    stockCode <- isolate(input$stockCode)
    print(stockCode)
    stockCode <- ifelse(stockCode == "", "NA", stockCode)  # 防呆機制
    fromDate <- isolate(input$dates[1])
    toDate <- isolate(input$dates[2])
    
    # 下載股票資料
    stockPriceData <- tq_get(stockCode, 
                             get = "stock.price", 
                             from = fromDate, 
                             to = toDate)
    
    # 判斷是否有下載到資料
    if(is.na(stockPriceData)){
      
      # 若未載到資料則提示警告訊息給使用者
      output$warningText <- renderText({"請輸入正確的Yahoo Finance股票代碼唷!"})
      
    }else{
      
      # 清空警告訊息
      output$warningText <- renderText({NULL})
      
      # 繪製技術分析圖形
      output$stockPlot <- renderPlot({
        
        stockPriceData <- stockPriceData %>%
          mutate(date = ymd(date),
                 chg = ifelse(close > open, "up", "dn"),  # 判斷當日上漲/下跌
                 flat_bar = (open == close),              # 判斷當日開盤價是否等於收盤價(繪圖需額外繪製)
                 num = c(1:n()))                          # 樣本點編碼
        
        plotCode <- unique(stockPriceData$symbol)
        plotStartDate <- min(stockPriceData$date)
        plotEndDate <- max(stockPriceData$date)
        
        p <- ggplot(stockPriceData, aes(x = num)) +                                   # 建立畫布
          geom_linerange(aes(ymin = low, ymax = high)) +                              # 繪製上影線和下影線
          geom_rect(aes(xmin = num-1/3 , xmax =num+1/3, ymin = pmin(open, close),     # 繪製K棒實體部分
                        ymax = pmax(open, close), fill = chg)) +
          scale_fill_manual(values = c("dn" = "forestgreen", "up" = "firebrick")) +   # 改變K棒顏色，跌為綠色，漲為紅色
          scale_color_manual(values=c("blue", "orange", "green")) +                   # 改變移動平均線顏色(此處參考奇摩股市網的顏色)
          theme_bw() +                                                                # 將畫布底圖顏色改為白色
          labs(title = paste0(plotCode),                                              # 加入標題及改變X和Y軸名稱
               subtitle = paste0("繪製期間: ", plotStartDate, " 至 ", plotEndDate),
               x="", y="") +
          theme(text = element_text(size = 16),
                plot.title = element_text(hjust = 0.5),                               # 將標題置中
                plot.subtitle = element_text(hjust = 0.5),
                plot.margin = margin(0.5, 1.3, 0, 0, "cm"),
                legend.position="bottom",                                             # 把legend移到上面
                legend.title=element_blank()) +                                       # 將legend的title改掉
          guides(fill = FALSE)                                                        # 將fill方式繪圖的legend拿掉
        
        
        # 繪製垂直線位置 表示比對範圍
        dataRowNum <- nrow(stockPriceData)
        p <- p +
          scale_x_continuous(breaks = seq(dataRowNum, 1,-ceiling(dataRowNum/5)),             # 將x軸的標籤名稱更改為日期
                             labels = stockPriceData$date[seq(dataRowNum, 1,-ceiling(dataRowNum/5))], expand = c(0, 0))
        
        # 若有開盤價=收盤價之交易日資料，因geom_rect無法繪製，要用geom_segment補繪
        if(any(stockPriceData$flat_bar)){
          p <- p +
            geom_segment(data = stockPriceData[stockPriceData$flat_bar,],
                         aes(x = num-1/3, y = close, yend = close, xend = num+1/3))
        }
        
        return(p)
      })
      
      # 輸出表格
      output$stockData <- DT::renderDataTable({
        datatable(stockPriceData %>% arrange(desc(date)))
      })
    }
  })
}

# Run the application 
shinyApp(ui = ui, server = server)
```

* 補充：plotly套件有股票技術分析圖形範例程式碼[candlestick](https://plotly.com/r/candlestick-charts/)，可替換程式碼內的ggplot2寫法。


---

# 八、建立Shiny帳密登入機制

我們可透過[shinymanager套件](https://github.com/datastorm-open/shinymanager)來為Shiny建立帳密登入機制，但需注意此套件無法在Shinyapps.io上使用。

以下程式碼摘自套件Github範例:

```r=
# 建立帳密資訊
credentials <- data.frame(
  user = c("shiny", "shinymanager"), # mandatory
  password = c("azerty", "12345"),   # mandatory
  start = c("2019-04-15"),           # optinal (all others)
  expire = c(NA, "2019-12-31"),
  admin = c(FALSE, TRUE),
  comment = "Simple and secure authentification mechanism 
  for single ‘Shiny’ applications.",
  stringsAsFactors = FALSE
)

library(shiny)
library(shinymanager)

# UI
ui <- fluidPage(
  tags$h2("My secure application"),
  verbatimTextOutput("auth_output")
)

# Wrap your UI with secure_app
ui <- secure_app(ui)

# Server
server <- function(input, output, session) {
  
  # call the server part
  # check_credentials returns a function to authenticate users
  res_auth <- secure_server(
    check_credentials = check_credentials(credentials)
  )
  
  output$auth_output <- renderPrint({
    reactiveValuesToList(res_auth)
  })
  
  # your classic server logic
  
}

shinyApp(ui, server)
```

---

# 補充

## 補充範例1

```r=
library(shiny)

# Define UI for app that draws a histogram ----
ui <- fluidPage(
  
  # App title ----
  titlePanel("Hello Shiny!"),
  
  # Sidebar layout with input and output definitions ----
  sidebarLayout(
    
    # Sidebar panel for inputs ----
    sidebarPanel(
      
      # Input: Slider for the number of bins ----
      sliderInput(inputId = "bins",
                  label = "Number of bins:",
                  min = 1,
                  max = 50,
                  value = 30)
    ),
    
    # Main panel for displaying outputs ----
    mainPanel(
      
      # Output: Histogram ----
      plotOutput(outputId = "waiting_distPlot"),
      plotOutput(outputId = "eruptions_distPlot")
    )
  )
)

# Define server logic required to draw a histogram ----
server <- function(input, output) {
  
  # Histogram of the Old Faithful Geyser Data ----
  # with requested number of bins
  # This expression that generates a histogram is wrapped in a call
  # to renderPlot to indicate that:
  #
  # 1. It is "reactive" and therefore should be automatically
  #    re-executed when inputs (input$bins) change
  # 2. Its output type is a plot
  
  output$waiting_distPlot <- renderPlot({
    
    x    <- faithful$waiting
    bins <- seq(min(x), max(x), length.out = input$bins + 1)
    
    hist(x, breaks = bins, col = "#75AADB", border = "white",
         xlab = "Waiting time to next eruption (in mins)",
         main = "Histogram of waiting times")
    
  })
  
  output$eruptions_distPlot <- renderPlot({
    
    x    <- faithful$eruptions
    bins <- seq(min(x), max(x), length.out = input$bins + 1)
    
    hist(x, breaks = bins, col = "#75AADB", border = "white",
         xlab = "Waiting time to next eruption (in mins)",
         main = "Histogram of waiting times")
    
  })
  
}

# Create Shiny app ----
shinyApp(ui = ui, server = server)

```

## 補充範例2

```r=
library(shiny)
library(lubridate)
library(DT)
library(shinythemes)
library(plotly)
library(tidyquant)
library(tidyverse)
library(shinymanager)


credentials <- data.frame(
  user = c("nsysu", "shiny", "shinymanager"), # mandatory
  password = c("nsysu", "azerty", "12345"),   # mandatory
  start = c("2019-04-15"),           # optinal (all others)
  expire = c(NA, NA, "2020-12-31"),
  admin = c(FALSE, FALSE, TRUE),
  comment = "Simple and secure authentification mechanism 
  for single ‘Shiny’ applications.",
  stringsAsFactors = FALSE
)


# UI
ui <- navbarPage("Shiny", id = "tabs", 
                 
                 theme = shinytheme("flatly"),
                 
                 tabPanel("繪製技術分析圖形", value = "繪製技術分析圖形",
                          
                          sidebarLayout(
                            
                            # 側邊頁面
                            sidebarPanel(
                              
                              # 輸入股票代碼
                              textInput("stockCode", label = h5("請輸入Yahoo Finance股票代碼:"), 
                                        value = "", placeholder = ""),
                              
                              # 查詢日期範圍
                              dateRangeInput("dates", label = h5("請選擇繪製的日期範圍"),
                                             start = Sys.Date()-days(90),
                                             end = Sys.Date(),
                                             min = Sys.Date()-years(5),
                                             max = Sys.Date(),
                                             separator = "至"),
                              
                              # 查詢按鈕
                              actionButton("action", label = "查詢"),
                              
                              # 輸出警告
                              br(),
                              span(textOutput(outputId = "warningText"), style = "color:red"),
                              
                              # 說明資料來源
                              br(),
                              tags$div(
                                "本網站數據來源: ",
                                tags$a(href = "https://finance.yahoo.com/", "Yahoo Finance")
                              ),
                            ),
                            
                            # 主頁面
                            mainPanel(
                              
                              HTML("<center>"),
                              h3("技術分析圖形"),
                              HTML("</center>"),
                              plotlyOutput("stockPlot"),
                              hr(),
                              HTML("<center>"),
                              h3("股價資訊表"),
                              HTML("</center>"),
                              dataTableOutput("stockData")
                            )
                          )
                 ),
                 
                 # 字體設定:使用CSS語法
                 tags$head(
                   tags$style(HTML("@import url('//fonts.googleapis.com/css?family=Noto+Sans+TC');
                                   body, h2, h3, h4, h5, center {font-family: 'Noto Sans TC', sans-serif;}")))
)

ui <- secure_app(ui, theme = shinytheme("flatly"))

# Server
server <- function(input, output, session) {
  
  # call the server part
  # check_credentials returns a function to authenticate users
  res_auth <- secure_server(
    check_credentials = check_credentials(credentials)
  )
  
  output$auth_output <- renderPrint({
    reactiveValuesToList(res_auth)
  })
  
  # 啟動查詢
  observeEvent(input$action, {
    
    # 讀取使用者設定資訊
    stockCode <- isolate(input$stockCode)
    print(stockCode)
    stockCode <- ifelse(stockCode == "", "NA", stockCode)  # 防呆機制
    fromDate <- isolate(input$dates[1])
    toDate <- isolate(input$dates[2])
    
    # 下載股票資料
    stockPriceData <- tq_get(stockCode, 
                             get = "stock.price", 
                             from = fromDate, 
                             to = toDate)
    
    # 判斷是否有下載到資料
    if(is.na(stockPriceData)){
      
      # 若未載到資料則提示警告訊息給使用者
      output$warningText <- renderText({"請輸入正確的Yahoo Finance股票代碼唷!"})
      
    }else{
      
      # 清空警告訊息
      output$warningText <- renderText({NULL})
      
      output$stockPlot <- renderPlotly({
        fig <- stockPriceData %>% plot_ly(x = ~date, type="candlestick",
                              open = ~open, close = ~close,
                              high = ~high, low = ~low) 
        fig <- fig %>% layout(title = "Basic Candlestick Chart")
      })
      
      
      # # 繪製技術分析圖形
      # output$stockPlot <- renderPlot({
      #   
      #   stockPriceData <- stockPriceData %>%
      #     mutate(date = ymd(date),
      #            chg = ifelse(close > open, "up", "dn"),  # 判斷當日上漲/下跌
      #            flat_bar = (open == close),              # 判斷當日開盤價是否等於收盤價(繪圖需額外繪製)
      #            num = c(1:n()))                          # 樣本點編碼
      #   
      #   plotCode <- unique(stockPriceData$symbol)
      #   plotStartDate <- min(stockPriceData$date)
      #   plotEndDate <- max(stockPriceData$date)
      #   
      #   p <- ggplot(stockPriceData, aes(x = num)) +                                   # 建立畫布
      #     geom_linerange(aes(ymin = low, ymax = high)) +                              # 繪製上影線和下影線
      #     geom_rect(aes(xmin = num-1/3 , xmax =num+1/3, ymin = pmin(open, close),     # 繪製K棒實體部分
      #                   ymax = pmax(open, close), fill = chg)) +
      #     scale_fill_manual(values = c("dn" = "forestgreen", "up" = "firebrick")) +   # 改變K棒顏色，跌為綠色，漲為紅色
      #     scale_color_manual(values=c("blue", "orange", "green")) +                   # 改變移動平均線顏色(此處參考奇摩股市網的顏色)
      #     theme_bw() +                                                                # 將畫布底圖顏色改為白色
      #     labs(title = paste0(plotCode),                                              # 加入標題及改變X和Y軸名稱
      #          subtitle = paste0("繪製期間: ", plotStartDate, " 至 ", plotEndDate),
      #          x="", y="") +
      #     theme(text = element_text(size = 16),
      #           plot.title = element_text(hjust = 0.5),                               # 將標題置中
      #           plot.subtitle = element_text(hjust = 0.5),
      #           plot.margin = margin(0.5, 1.3, 0, 0, "cm"),
      #           legend.position="bottom",                                             # 把legend移到上面
      #           legend.title=element_blank()) +                                       # 將legend的title改掉
      #     guides(fill = FALSE)                                                        # 將fill方式繪圖的legend拿掉
      #   
      #   
      #   # 繪製垂直線位置 表示比對範圍
      #   dataRowNum <- nrow(stockPriceData)
      #   p <- p +
      #     scale_x_continuous(breaks = seq(dataRowNum, 1,-ceiling(dataRowNum/5)),             # 將x軸的標籤名稱更改為日期
      #                        labels = stockPriceData$date[seq(dataRowNum, 1,-ceiling(dataRowNum/5))], expand = c(0, 0))
      #   
      #   # 若有開盤價=收盤價之交易日資料，因geom_rect無法繪製，要用geom_segment補繪
      #   if(any(stockPriceData$flat_bar)){
      #     p <- p +
      #       geom_segment(data = stockPriceData[stockPriceData$flat_bar,],
      #                    aes(x = num-1/3, y = close, yend = close, xend = num+1/3))
      #   }
      #   
      #   return(p)
      # })
      
      # 輸出表格
      output$stockData <- DT::renderDataTable({
        datatable(stockPriceData %>% arrange(desc(date)))
      })
    }
  })
}

# Run the application 
shinyApp(ui = ui, server = server)
```







