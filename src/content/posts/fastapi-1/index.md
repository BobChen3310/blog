---
title: FastAPI 學習筆記 Part 1 - Hello FastAPI
published: 2025-10-23
description: ''
image: ''
tags: [Web, Python]
category: 'Web'
lang: 'zh-TW'
draft: false
---

## 前言

最近要期中考了，有一堂課要我們使用 ESP32 製作一個專題，還要求必須預想一個情境，我想到的就只有宿舍房間門禁系統可以做

我們學校宿舍只有在大門有設置門禁系統，但房間還是使用鑰匙解鎖，屬實是有一些不方便，萬一鑰匙沒帶室友把房間鎖住了還得去管理室借鑰匙

我就想到，出門上課或離開宿舍一定會帶學生證，那房間的門鎖是不是也能改用學生證解鎖的方式呢，當然也是有弊端，如系統建置的成本、若只是要去裝水

洗衣服之類的還得帶上學生證，但這就不在討論範圍了。因為 ESP32 要跟遠端進行資料同步及驗證的部分，所以需要寫個後端來做這些事，在查詢的過程中
發現了 FastAPI，但我完全沒學過，只能在這兩週內惡補了🥴

## 準備工作

1. 創個工作資料夾

```bash
mkdir -p smartlock-project
```

2. 建立 Python 虛擬環境

```bash
cd smartlock-project/
python -m venv .venv
```

3. 啟用虛擬環境

```bash
# Linux/MacOS
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

4. 安裝套件

```python
pip install fastapi[standard] 
```

## 開始實作

之後創建一個 `app.py`，然後進到該文件

```python
from fastapi import FastAPI

app = FastAPI()     # 建立 FastAPI instance

@app.get("/")       # 路徑及操作
async def root():
    return {"message": "Hello World"}
```

然後回到終端，輸入 `fastapi dev app.py` 就可以讓 API 跑起來了

在瀏覽器輸入 [http://localhost:8000](http://localhost:8000) 就會得到這個結果 `{"message": "Hello World"}`

[http://localhost:8000/docs](http://localhost:8000/docs) 則是 FastAPI 產生的 Swagger 文檔，可以在這裡進行交互操作

## 結語

以上就是 Part 1 的說明，我也還在學習中，對很多東西都還不是很了解，我們下次見!