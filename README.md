# Hexo 部落格

這是使用 [Hexo](https://hexo.io/) 建立的靜態部落格專案。

## 專案簡介

Hexo 是一個快速、簡潔且強大的部落格框架。您可以使用 Markdown 撰寫文章，Hexo 會快速產生靜態網站供您部署。

## 安裝與設定

### 環境要求

- Node.js (建議 18.0 以上版本)
- Git

### 安裝依賴

```bash
npm install
```

## 使用方法

### 啟動本地伺服器

```bash
npx hexo server
```

或

```bash
npm run server
npx hexo server --port 4000
```

訪問 [http://localhost:4000](http://localhost:4000) 即可預覽您的部落格。

### 新增文章

```bash
npx hexo new "文章標題"
```

### 清理快取

```bash
npx hexo clean
```

### 產生靜態檔案

```bash
npx hexo generate
```

### 部署

```bash
npx hexo deploy
```

## 專案結構

```
.
├── _config.yml          # 網站組態檔
├── package.json         # 應用程式資訊
├── scaffolds/           # 鷹架
├── source/              # 資源資料夾
│   ├── _drafts/         # 草稿
│   └── _posts/          # 文章
└── themes/              # 主題
```

## 設定

主要設定檔為 `_config.yml`，您可以在此檔案中修改網站標題、作者、語言等設定。

## 主題

目前使用 [Phantom](https://github.com/klugjo/hexo-theme-phantom) 主題，這是一個現代化、響應式的美觀主題。您可以在 [Hexo Themes](https://hexo.io/themes/) 找到更多主題。

### 主題特色

- 響應式設計，適合各種螢幕尺寸
- 支援封面圖片展示
- 現代化的卡片式佈局
- 內建社群媒體連結支援
- 支援 Disqus 留言系統

## 留言系統設定

本部落格使用 [Disqus](https://disqus.com) 作為留言系統。以下是完整的設定步驟：

### 1. Disqus 帳戶設定

1. **註冊 Disqus 帳戶**：
   - 前往 [disqus.com](https://disqus.com) 註冊帳戶
   - 登入後點選右上角個人頭像 → "Admin"

2. **建立網站**：
   - 在 Admin 頁面點選 "Create Site"
   - 設定網站名稱（例如：Cleo's Blog）
   - 設定 **Shortname**（這是最重要的設定，例如：`cleoliu`）
   - 在 "Website URL" 填入：`https://cleoliu.github.io/blog-ai`

3. **記錄 Shortname**：
   - 在 Settings → General 中可以查看你的 shortname
   - 這個 shortname 將用於主題配置

### 2. 主題配置

在 `themes/phantom/_config.yml` 中設定：

```yaml
comments:
  # Disqus comments
  disqus_shortname: your-shortname-here  # 替換為你的實際 shortname
```

### 3. 啟用文章留言

在你想要啟用留言的文章 frontmatter 中添加：

```yaml
---
title: 你的文章標題
date: 2026-02-02 20:02:31
comments: true  # 啟用留言功能
---
```

### 4. 部署與測試

1. **重新生成網站**：
   ```bash
   npx hexo clean && npx hexo generate
   ```

2. **推送到 GitHub Pages**：
   ```bash
   git add .
   git commit -m "Enable Disqus comments"
   git push origin main
   ```

3. **確認設定**：
   - 等待 GitHub Actions 部署完成
   - 訪問線上版本的文章頁面
   - 確認 Disqus 留言區正常載入

### 5. 常見問題排除

- **留言區顯示「We were unable to load Disqus」**：
  - 檢查 Disqus Admin 中的網站 URL 是否正確
  - 確認 shortname 拼寫正確
  - 確認文章中有設定 `comments: true`

- **留言區沒有顯示**：
  - 確認主題配置中的 `disqus_shortname` 已設定
  - 檢查瀏覽器是否阻擋了第三方腳本
  - 在本地測試時，Disqus 可能無法在 localhost 上正常運作

- **域名不匹配**：
  - 在 Disqus Admin → Settings → General 中
  - 確認 "Website URL" 設定為正確的部署網址

## 部署

### GitHub Pages 自動部署

本專案已設定 GitHub Actions 自動部署到 GitHub Pages。

#### 設定步驟：

1. **推送程式碼到 GitHub**：

   ```bash
   git add .
   git commit -m "Update GitHub Actions workflow"
   git push origin main
   ```

2. **啟用 GitHub Pages**：

   - 前往 GitHub 儲存庫設定頁面
   - 找到 "Pages" 區段
   - 在 "Source" 中選擇 "GitHub Actions"
   - **重要**：確保在 "Actions" > "General" 中，"Workflow permissions" 設定為 "Read and write permissions"

3. **自動部署**：
   - 每次推送到 `main` 分支時會自動觸發部署
   - 部署完成後可在 `https://cleoliu.github.io/Hexo` 查看
   - 可以在 "Actions" 標籤中查看部署狀態

#### 手動部署（可選）：

如果您偏好手動部署，可以安裝部署套件：

```bash
npm install hexo-deployer-git --save
```

然後在 `_config.yml` 中設定：

```yaml
deploy:
  type: git
  repo: https://github.com/cleoliu/Hexo.git
  branch: gh-pages
```

執行部署：

```bash
npx hexo deploy
```

### 其他部署方式

也支援其他部署方式：

- **Netlify**：連接 GitHub 儲存庫自動部署
- **Vercel**：匯入 GitHub 專案一鍵部署
- **自訂伺服器**：上傳 `public/` 資料夾內容

## 更多資源

- [Hexo 官方文檔](https://hexo.io/docs/)
- [Hexo 主題](https://hexo.io/themes/)
- [Hexo 外掛](https://hexo.io/plugins/)

## 授權

MIT License
