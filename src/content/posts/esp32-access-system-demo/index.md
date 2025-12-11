---
title: ESP32 智慧門禁系統實作：整合 RFID、Telegram 遠端解鎖與即時監控平台
published: 2025-10-23
description: ''
image: ''
tags: [IoT, Python, JavaScript]
lang: 'zh-TW'
draft: false
---

## 前言

目前我住的宿舍各房間的房門還是使用傳統的鑰匙鎖，如果忘記帶鑰匙有可能被室友鎖在外面，剛好這學期的某堂課要製作物聯網相關的專題，這就變成我要做的題目了。

此專題為基於 ESP32 的門禁系統，整合後端 API、前端管理介面、PostgreSQL 資料庫、MQTT 通訊與 Telegram Bot 遠端控制機制。後端與機器人程式均使用 uv 進行套件管理，ESP32 採用 MicroPython 實作 RFID 門禁邏輯及接收 MQTT 訂閱內容。

## 功能特色

- 使用 ESP32 搭配 RC522、LCD1602、蜂鳴器、電磁閥與 LED 完成門禁控制
- 支援卡片感應、門禁事件上報、警示提示
- 後端採 FastAPI，使用 uv 進行依賴管理與執行
- 使用 PostgreSQL 儲存學生資料、設備紀錄與刷卡事件
- 刷卡驗證採 HTTP Request，遠端解鎖透過 MQTT
- 前端採 React，提供設備管理、學生管理、紀錄查詢等功能
- 提供 Telegram Bot 遠端解鎖能力

## 專案結構

```
esp32-access-system/
├── backend/                    # 後端 API
│   ├── app/
│   │   ├── routers/            # 路由
│   │   ├── auth.py             # API 安全設定
│   │   ├── database.py         # 資料庫建立程式
│   │   ├── email_utils.py      # 驗證信發送程式
│   │   └── models.py           # ORM 模型
│   ├── pyproject.toml          # 專案配置與依賴
│   ├── main.py                 # 入口程式
│   └── .env.example            # 環境變數範例
├── bot/                        # Telegram Bot
│   ├── bot.py                  # 主程式
│   ├── pyproject.toml          # 專案配置與依賴
│   └── .env.example            # 環境變數範例
├── frontend/                   # React 前端
├── esp32/                      # ESP32 MicroPython 程式
│   ├── modules/
│   ├── esp32_door.py
│   ├── mfrc522.py
│   └── lcd1602.py
└── README.md                   # 說明文件
```

## 快速開始

### 1. **環境需求**
* Python 3.12+
* Node.js 20+
* PostgreSQL 17+

### 2. **電路連接**

![](./images/circuit.png)

LCD1602 I2C
- VCC `ESP32 5V`
- GND `ESP32 GND`
- SDA `PIN32`
- SCL `PIN27`

RC522
- SDA `PIN0`
- SCK `PIN18`
- MOSI `PIN23`
- MISO `PIN19`
- GND `ESP32 GND`
- RST `PIN4`
- 3.3V `ESP32 3.3V`

無緣蜂鳴器
- 正極 `PIN14`
- 負極 `ESP32 GND`

紅色 LED
- 正極 `PIN16`
- 負極 `220 歐姆電阻 -> ESP32 GND`

綠色 LED
- 正極 `PIN13`
- 負極 `220 歐姆電阻 -> ESP32 GND`

電磁閥 (沒有分正負極)
- A 端 `繼電器 NO 腳`
- B 端 `5V 外接電源負極`

繼電器
- DC+ `ESP32 5V`
- DC- `ESP32 GND`
- IN `PIN17`
- COM `5V 外接電源正極`
- NO `電磁閥 A 端`

### 3. **後端設定**

1. **進入專案目錄**
    ```bash
    cd esp32-access-system/backend
    ```

2. **安裝依賴**
    ```bash
    uv sync
    ```

3. **設定環境變數**

    複製 `.env.example` 為 `.env` 並填入以下資訊：
    ```bash
    cp .env.example .env
    ```

    ```
    # Database Setting
    DATABASE_URL=postgresql+asyncpg://db_admin:db_passwd@localhost:5432/db_name

    # JWT Setting
    SECRET_KEY=your-secret-key
    ALGORITHM=HS256

    BOT_API_SECRET=your-bot-api-secret

    # MQTT Setting
    MQTT_BROKER=your-broker-ip
    MQTT_PORT=your-broker-port
    MQTT_USERNAME=username
    MQTT_PASSWORD=password

    # Email Setting
    MAIL_USERNAME=your-email
    MAIL_PASSWORD=your-passwd
    MAIL_FROM=your-email
    MAIL_FROM_NAME=your-email-sender
    MAIL_SERVER=your-smtp-server-ip
    MAIL_PORT=your-smtp-server-port
    ```

4. **啟動後端**
    ```bash
    # 啟動後端
    uv run main.py
    
    # 建立管理員帳號
    uv run create_admin.py
    ```
    之後就可透過 Nginx 進行反向代理部署了。

### 4. **前端設定**

1. **進入專案目錄**
    ```bash
    cd esp32-access-system/frontend
    ```

2. **安裝依賴**
    ```bash
    pnpm install
    ```

3. **API 連線設定**

    修改 `src/api/client.js` 中的 `baseURL` 指向正確的後端位址。
    ```js
    import axios from 'axios';

    const apiClient = axios.create({
        baseURL: '將這裡改成你後端 API 的位址',
        headers: {
            'Content-Type': 'application/json',
        },
    });
    ```

4. **編譯打包**
    ```bash
    pnpm build
    ```
    編譯後的檔案位於 `dist/` 目錄，可透過 Nginx 進行反向代理部署。

### 5. 機器人設定

1. **進入專案目錄**
    ```bash
    cd esp32-access-system/bot
    ```

2. **安裝依賴**
    ```bash
    uv sync
    ```

3. **設定環境變數**

    複製 `.env.example` 為 `.env` 並填入以下資訊：
    ```bash
    cp .env.example .env
    ```

    ```
    TELEGRAM_BOT_TOKEN=your-bot-token
    BACKEND_API_URL=you-backend-api-url
    BOT_API_SECRET=your-bot-api-secret
    ```

4. **啟動 Bot**
    ```bash
    uv run bot.py
    ```

### 6. ESP32 程式

將 `esp32-access-system/esp32` 資料夾中的所有檔案上傳到 ESP32，安裝 `umqtt.simple` 套件，修改 `esp32_door.py` 中的 `CONFIG`

```python
CONFIG = {
    "WIFI_SSID": "your-wifi-ssid",
    "WIFI_PASS": "your-wifi-password",
    "API_URL": "your-api-url", 
    "DEVICE_TOKEN": "your-device-token",
    "DEVICE_ID": "your-device-id",
    "MQTT_BROKER": "your-mqtt-broker-ip",
    "MQTT_USER": "your-mqtt-broker-username",
    "MQTT_PASS": "your-mqtt-broker-password",
    "MQTT_PORT": "your-mqtt-broker-port"            # Port Number 記得把雙引號刪除
}
```

之後在 ESP32 執行 `esp32_door.py` 即可。

## 實作影片

<iframe width="85%" height="400" src="https://www.youtube.com/embed/EdsFFSmrGfo" title="ESP32 智慧門禁系統 Demo" frameborder="0" allowfullscreen></iframe>

## 參考連結

 - [專案儲存庫](https://github.com/BobChen3310/esp32-access-system)
 - [FastAPI 官方文檔](https://fastapi.tiangolo.com/)
 - [React 官方文檔](https://react.dev/learn/)
 - [python-telegram-bot 官方文檔](https://docs.python-telegram-bot.org/en/stable/)