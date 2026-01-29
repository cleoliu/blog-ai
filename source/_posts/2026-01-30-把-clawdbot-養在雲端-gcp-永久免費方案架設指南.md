---
title: GCP永久免費部署Clawdbot AI助手：完整教學
date: 2026-01-30 01:40:46
subtitle: 打造24小時不間斷AI助理，零成本運行
categories:
  - DevOps
tags:
  - gcp
  - ai
  - deployment
  - node.js
  - tutorial
cover_index: https://images.unsplash.com/photo-1499750310107-5fef28a66643?w=450&h=450&fit=crop&crop=center
---

# 在 Google Cloud Platform (GCP) 永久免費方案上部署 Clawdbot AI 助手

本部署指南將詳細說明如何在 Google Cloud Platform (GCP) 的永久免費方案 (Always Free Tier) 上部署 Clawdbot AI 助手。透過此指南，您將能夠建立一個 24 小時不間斷運行的 Clawdbot 實例，而無需額外支付伺服器費用。

## 1. 部署概述

### 部署目標
本指南的目標是協助使用者在 Google Cloud Platform (GCP) 上，利用其永久免費方案提供的資源，部署並運行一個 Clawdbot AI 助手。部署完成後，Clawdbot 將能夠全天候運行，並透過您選擇的通訊平台（如 Telegram、Discord 等）提供服務。

### 部署架構
*   **雲端平台：** Google Cloud Platform (GCP)
*   **計算資源：** Compute Engine VM 執行個體 (e2-micro 機器類型)
*   **作業系統：** Ubuntu 22.04 LTS 或 Debian 12
*   **應用程式：** Clawdbot (基於 Node.js)
*   **程序管理：** PM2 (用於確保 Clawdbot 持續運行並在系統重啟後自動啟動)
*   **儲存：** 30 GB 標準永久磁碟 (Standard persistent disk)

此架構利用 GCP 的免費資源限制，提供一個穩定且經濟高效的 Clawdbot 託管環境。

## 2. 環境準備

在開始部署之前，請確保您已完成以下準備工作：

### 系統需求
*   **Google 帳戶：** 一個有效的 Google 帳戶，用於登入 GCP。
*   **信用卡：** 一張有效的信用卡，用於 GCP 帳戶驗證。GCP 免費方案不會自動扣款，但驗證是必須的。

### 依賴項目 (將在 VM 上安裝)
*   **Node.js：** 建議使用 v20 或 v22 版本，Clawdbot 運行環境。
*   **npm：** Node.js 的套件管理器。
*   **nvm：** Node Version Manager，用於方便地管理 Node.js 版本。
*   **git：** 版本控制工具，可能用於某些 Clawdbot 相關的安裝或更新。
*   **curl：** 命令行工具，用於下載檔案。
*   **build-essential：** 編譯 C/C++ 程式所需的工具套件。
*   **PM2：** Node.js 應用程序的生產級程序管理器。

### 配置要求 (GCP 免費方案限制)
為了符合 GCP 永久免費方案的資格，請務必遵循以下配置要求：

*   **區域 (Region)：** 必須選擇以下三個區域之一：
    *   `us-west1` (奧勒岡)
    *   `us-central1` (愛荷華)
    *   `us-east1` (南卡羅來納)
    *   **推薦：** `us-west1`
*   **機器類型 (Machine Type)：** 必須選擇 `e2-micro` (2 vCPU, 1 GB RAM)。
*   **開機磁碟 (Boot Disk)：**
    *   **作業系統：** 建議選擇 `Ubuntu 22.04 LTS` 或 `Debian 12`。
    *   **磁碟大小：** 必須設定為 `30 GB` (標準永久磁碟 Standard persistent disk)。
*   **網路流量：** 每月享有 200 GB 的免費出口流量 (Egress)，對於文字型聊天機器人而言通常足夠。

## 3. 部署步驟

### 步驟 A：建立 GCP 專案與 VM 執行個體

1.  **登入 GCP Console：**
    *   開啟您的瀏覽器，前往 [Google Cloud Console](https://console.cloud.google.com/) 並使用您的 Google 帳戶登入。

2.  **建立新專案：**
    *   在左上角的導覽選單中，點擊當前專案名稱旁的下拉箭頭。
    *   選擇「新增專案 (New Project)」。
    *   輸入專案名稱，例如 `my-clawdbot-project`，然後點擊「建立 (Create)」。

3.  **前往 Compute Engine：**
    *   在左側導覽選單中，依序找到並點擊「Compute Engine」->「VM 執行個體 (VM Instances)」。
    *   如果這是您首次使用 Compute Engine，可能需要等待幾秒鐘來初始化服務。

4.  **建立執行個體 (Create Instance)：**
    *   點擊頁面頂部的「建立執行個體 (Create Instance)」按鈕。
    *   **配置 VM 執行個體以符合免費方案要求 (非常重要)：**
        *   **名稱 (Name)：** 輸入一個易於識別的名稱，例如 `clawdbot-instance`。
        *   **區域 (Region)：** 選擇 **`us-west1`**、`us-central1` 或 `us-east1` 之一。
        *   **地區 (Zone)：** 選擇您所選區域下的任何一個地區，例如 `us-west1-b`。
        *   **機器類型 (Machine Type)：** 點擊「變更 (Change)」，在「系列 (Series)」中選擇 `E2`，然後在「機器類型 (Machine type)」中選擇 **`e2-micro`**。
        *   **開機磁碟 (Boot Disk)：**
            *   點擊「變更 (Change)」按鈕。
            *   在「作業系統 (Operating system)」中，選擇 **`Ubuntu`** 並選擇版本 **`Ubuntu 22.04 LTS`** (或 `Debian 12`)。
            *   將「大小 (GB)」設定為 **`30`** GB。
            *   「開機磁碟類型 (Boot disk type)」選擇「標準永久磁碟 (Standard persistent disk)」。
            *   點擊「選取 (Select)」。
        *   **防火牆 (Firewall)：** 勾選「允許 HTTP 流量 (Allow HTTP traffic)」和「允許 HTTPS 流量 (Allow HTTPS traffic)」。
    *   確認所有設定無誤後，點擊頁面底部的「建立 (Create)」按鈕。
    *   等待幾秒鐘，直到 VM 執行個體狀態顯示為綠色勾勾 (Running)。

### 步驟 B：連線至 VM 執行個體 (SSH)

1.  在 Compute Engine 的「VM 執行個體」列表中，找到您剛建立的執行個體。
2.  在該執行個體列的右側，點擊「SSH」按鈕。
3.  一個新的瀏覽器視窗將彈出，並自動建立與 VM 執行個體的 SSH 連線。這表示您已成功進入您的雲端主機。

### 步驟 C：安裝系統依賴與 Node.js 環境

在 SSH 終端機中，依序執行以下命令：

1.  **更新系統並安裝必要套件：**

    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install -y curl git build-essential
    ```

2.  **安裝 Node.js (使用 nvm 管理)：**
    我們建議使用 Node Version Manager (nvm) 來安裝和管理 Node.js，這會讓環境更整潔。

    ```bash
    # 下載並執行 nvm 安裝腳本
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

    # 讓環境變數生效 (或關閉 SSH 視窗後重新開啟)
    source ~/.bashrc

    # 安裝 Node.js v22 (或您偏好的版本，如 v20)
    nvm install 22

    # 使用 Node.js v22
    nvm use 22

    # 驗證 Node.js 和 npm 版本
    node -v
    npm -v
    ```

### 步驟 D：部署 Clawdbot 應用程式

1.  **全域安裝 Clawdbot：**

    ```bash
    npm install -g clawdbot
    ```

2.  **啟動 Clawdbot 並進行初始設定：**
    首次啟動 Clawdbot 會引導您完成設定精靈 (Wizard)。

    ```bash
    clawdbot
    ```

    *   當提示 `Choose mode:` 時，選擇 `Local`。
    *   Clawdbot 將會詢問您要連結的平台（例如 Telegram、Discord 等），請依照指示輸入相應的 Token 或 API 金鑰。
    *   完成設定後，Clawdbot 將會啟動並開始運行。

### 步驟 E：配置 PM2 進行程序管理與自動啟動

為了確保 Clawdbot 即使在您關閉 SSH 視窗後也能持續運行，並在 VM 執行個體重啟後自動啟動，我們將使用 PM2。

1.  **全域安裝 PM2：**

    ```bash
    npm install -g pm2
    ```

2.  **使用 PM2 啟動 Clawdbot：**
    這將在背景啟動 Clawdbot，並由 PM2 監控其運行。

    ```bash
    pm2 start clawdbot --name "my-bot" -- start
    ```

    *   `--name "my-bot"` 給您的 Clawdbot 進程一個易於識別的名稱。
    *   `-- start` 是傳遞給 `clawdbot` 命令的參數，確保它以正常模式啟動。

3.  **設定 PM2 儲存當前進程列表：**
    這會將當前由 PM2 管理的進程列表儲存起來，以便在系統重啟後恢復。

    ```bash
    pm2 save
    ```

4.  **設定 PM2 開機自動啟動：**
    這將生成一個系統服務腳本，確保 PM2 及其管理的應用程序在 VM 執行個體啟動時自動啟動。

    ```bash
    pm2 startup
    ```

    *   執行此命令後，PM2 會輸出一個需要您複製並執行的命令（通常以 `sudo env PATH=$PATH:/usr/bin ...` 開頭）。請將該命令複製並貼上到 SSH 終端機中執行。

## 4. 驗證測試

部署完成後，請進行以下測試以確保 Clawdbot 正常運行：

1.  **檢查 Clawdbot 運行狀態：**
    在 SSH 終端機中，執行以下命令查看 PM2 管理的進程狀態：

    ```bash
    pm2 status
    ```
    您應該會看到 `my-bot` (或您設定的名稱) 狀態為 `online`。

2.  **測試 Clawdbot 功能：**
    *   開啟您在 Clawdbot 設定時連結的通訊平台（例如 Telegram 或 Discord）。
    *   向您的 Clawdbot 傳送訊息，驗證它是否能正常回應。

3.  **驗證自動啟動功能 (可選但推薦)：**
    *   在 GCP Console 中，前往您的 VM 執行個體列表。
    *   選擇您的 `clawdbot-instance`，點擊「停止 (Stop)」然後再點擊「啟動 (Start)」，或直接點擊「重設 (Reset)」來重啟 VM。
    *   等待 VM 重新啟動完成後，再次透過 SSH 連線到 VM。
    *   執行 `pm2 status`，確認 `my-bot` 進程是否自動啟動並處於 `online` 狀態。

## 5. 常見問題排解

*   **VM 執行個體建立失敗或不符合免費方案：**
    *   **問題：** 建立 VM 時提示收費或無法選擇 `e2-micro`。
    *   **解決方案：** 仔細檢查「區域 (Region)」、「機器類型 (Machine Type)」和「開機磁碟大小 (Boot Disk Size)」是否嚴格按照第 2 節「配置要求」中的說明設定。任何不符合免費方案的選項都會導致額外收費。

*   **SSH 連線問題：**
    *   **問題：** 無法透過瀏覽器 SSH 連線到 VM。
    *   **解決方案：**
        *   檢查 VM 狀態是否為「運行中 (Running)」。
        *   確保在建立 VM 時勾選了「允許 HTTP/HTTPS 流量」，雖然這不是 SSH 的直接要求，但良好的網路配置有助於排除問題。
        *   嘗試等待幾分鐘後重試。

*   **Clawdbot 未啟動或停止運行：**
    *   **問題：** `pm2 status` 顯示 `my-bot` 狀態為 `stopped` 或 `errored`。
    *   **解決方案：**
        *   **查看日誌：** 使用 `pm2 logs my-bot` 查看 Clawdbot 的運行日誌，尋找錯誤訊息。
        *   **檢查 Node.js 環境：** 確保 Node.js 和 npm 已正確安裝且版本符合要求 (`node -v`, `npm -v`)。
        *   **重新執行設定：** 嘗試手動啟動 `clawdbot` 並重新完成設定精靈，確認所有 Token 和 API 金鑰都正確無誤。
        *   **重啟應用：** `pm2 restart my-bot`。

*   **Clawdbot 在關閉 SSH 視窗後停止運行：**
    *   **問題：** 關閉 SSH 連線後，Bot 停止回應。
    *   **解決方案：** 這表示 PM2 未正確配置。請確保您已執行了「步驟 E」中的所有命令，特別是 `pm2 start ...`、`pm2 save` 和 `pm2 startup` (並執行其提示的命令)。

*   **記憶體不足 (Out Of Memory - OOM)：**
    *   **問題：** Clawdbot 運行一段時間後崩潰，日誌顯示記憶體不足錯誤。
    *   **解決方案：** `e2-micro` 機器類型只有 1GB RAM，對於複雜或資源密集型的 Clawdbot 插件（例如使用 Puppeteer 進行瀏覽器自動化）可能不足。
        *   嘗試禁用不必要的 Clawdbot 插件。
        *   Clawdbot 主要設計用於文字對話和 API 串接，避免在其上運行過於耗費資源的任務。
        *   如果記憶體問題持續存在，您可能需要考慮升級機器類型（這將不再符合永久免費方案）。

## 6. 維護和監控

為了確保 Clawdbot 的穩定運行和安全性，建議進行以下維護和監控：

*   **系統更新：**
    *   定期登入 VM 執行個體，更新作業系統和已安裝的套件。
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

*   **Clawdbot 和 Node.js 更新：**
    *   **Node.js：** 如果有新的穩定版 Node.js 發布，您可以使用 nvm 進行更新：
        ```bash
        nvm install <新版本號> # 例如 nvm install 22
        nvm use <新版本號>
        pm2 restart my-bot
        ```
    *   **Clawdbot：** 更新 Clawdbot 到最新版本：
        ```bash
        npm update -g clawdbot
        pm2 restart my-bot
        ```

*   **監控運行狀態：**
    *   **PM2 監控：** 使用 `pm2 status` 檢查 Clawdbot 的運行狀態。
    *   **日誌查看：** 使用 `pm2 logs my-bot` 實時查看 Clawdbot 的輸出日誌，以便及時發現問題。
    *   **GCP 監控：** 在 GCP Console 中，您可以查看 VM 執行個體的基礎監控指標，如 CPU 使用率和網路流量，以了解資源消耗情況。

*   **備份：**
    *   定期備份 Clawdbot 的配置文件（如果它們儲存在 VM 上且您進行了大量自定義），以防不測。

*   **資源限制意識：**
    *   始終記住 `e2-micro` 的資源限制。避免在免費層級的 VM 上運行超出其能力的應用或服務。

恭喜您！您的 Clawdbot AI 助手現在已在 Google Cloud Platform 上穩定運行。祝您和您的 AI 夥伴交流愉快！🤖✨