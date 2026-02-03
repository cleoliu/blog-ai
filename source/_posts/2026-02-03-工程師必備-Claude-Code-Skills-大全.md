---
title: 工程師必備 Claude Code Skills 大全：30+ 實用技能即裝即用
date: 2026-02-03 12:50:00
subtitle: 複製貼上就能用的 Claude Code 技能包，涵蓋開發、測試、文件、DevOps 全流程
categories:
  - AI/ML
tags:
  - claude
  - claude-code
  - skills
  - productivity
  - developer-tools
cover_index: https://images.unsplash.com/photo-1461749280684-dccba630e2f6?w=450&h=450&fit=crop&crop=center
---

> 🛠️ 上一篇講了怎麼建立技能，這篇直接給你 30+ 個現成的，複製貼上就能用。

這是一份工程師日常開發會用到的 Claude Code Skills 清單，按使用場景分類。每個技能都經過實戰驗證，即裝即用。

---

# 快速設置

## 1. 建立技能目錄

```bash
mkdir -p ~/.claude/commands
```

## 2. 安裝方式

每個技能就是一個 `.md` 檔案。例如要安裝 `/review` 技能：

```bash
# 建立檔案
cat > ~/.claude/commands/review.md << 'EOF'
（貼上技能內容）
EOF
```

或者直接用編輯器：

```bash
code ~/.claude/commands/review.md
```

## 3. 一鍵安裝全部

想要一次裝完？把這篇文章的技能全部下載：

```bash
# Clone 技能包（範例）
git clone https://github.com/example/claude-skills ~/.claude/commands-backup
cp ~/.claude/commands-backup/*.md ~/.claude/commands/
```

---

# 🔍 程式碼審查類

## /review - 標準 Code Review

```markdown
請以資深工程師的角度 review 以下程式碼：

$ARGUMENTS

## Review 重點

1. **安全性**：SQL injection、XSS、敏感資料暴露
2. **效能**：N+1 查詢、不必要的迴圈、記憶體洩漏
3. **可讀性**：命名、註解、結構
4. **錯誤處理**：異常捕獲、邊界條件
5. **最佳實踐**：設計模式、框架慣例

## 輸出格式

評分：X/10
### 🚨 嚴重問題
### ⚠️ 建議改進  
### ✅ 做得好的地方
### 📝 修改後的程式碼
```

## /security - 安全性審查

```markdown
請進行安全性專項審查：

$ARGUMENTS

## 檢查項目

1. **注入攻擊**：SQL、NoSQL、Command、LDAP
2. **認證授權**：身份驗證漏洞、權限繞過
3. **敏感資料**：硬編碼密鑰、日誌洩漏、不當傳輸
4. **輸入驗證**：XSS、路徑遍歷、不安全反序列化
5. **依賴風險**：已知漏洞的套件

## 輸出

按 OWASP Top 10 分類問題，標註嚴重等級（Critical/High/Medium/Low），並給出修復建議。
```

## /perf - 效能審查

```markdown
請進行效能專項審查：

$ARGUMENTS

## 分析重點

1. **時間複雜度**：演算法效率
2. **空間複雜度**：記憶體使用
3. **I/O 效能**：資料庫查詢、網路請求、檔案操作
4. **並發處理**：race condition、死鎖
5. **快取策略**：可快取的計算或查詢

## 輸出

列出效能瓶頸，估算影響程度，提供優化方案和預期改善幅度。
```

---

# 🐛 Debug 與問題排查

## /debug - 錯誤診斷

```markdown
請分析這個錯誤並提供解決方案：

$ARGUMENTS

## 分析流程

1. **錯誤解讀**：這個錯誤訊息代表什麼？
2. **常見原因**：列出 3-5 個可能的原因
3. **排查步驟**：如何確定是哪個原因？
4. **解決方案**：針對每個原因的修復方式
5. **預防措施**：如何避免再次發生？

如果資訊不足，請列出需要的額外資訊（堆疊追蹤、環境、重現步驟等）。
```

## /explain-error - 錯誤訊息翻譯

```markdown
請用白話解釋這個錯誤：

$ARGUMENTS

## 要求

1. 一句話說明這個錯誤是什麼
2. 用生活化比喻解釋為什麼會發生
3. 最直接的解決方式（複製貼上能用）
4. 如果解決方式不唯一，列出其他選項

語氣輕鬆，對新手友善。
```

## /trace - 問題追蹤

```markdown
幫我追蹤這個問題的根本原因：

$ARGUMENTS

## 使用 5 Whys 分析法

1. **現象**：發生了什麼？
2. **Why 1**：為什麼會發生這個現象？
3. **Why 2**：為什麼會導致上述原因？
4. **Why 3**：繼續深入...
5. **Why 4**：...
6. **Why 5**：根本原因

## 最終輸出

- 根本原因
- 短期修復（止血）
- 長期解決方案（治本）
```

---

# ✍️ 程式碼生成

## /impl - 實作功能

```markdown
請實作以下功能：

$ARGUMENTS

## 要求

1. 使用目前專案的語言和框架
2. 遵循專案現有的程式碼風格
3. 包含必要的錯誤處理
4. 加上簡潔的註解
5. 考慮邊界條件

## 輸出

1. 完整可執行的程式碼
2. 使用範例
3. 需要的額外依賴（如有）
```

## /test - 生成測試

```markdown
請為以下程式碼生成測試：

$ARGUMENTS

## 測試類型

1. **單元測試**：每個函式的基本功能
2. **邊界測試**：空值、極值、異常輸入
3. **錯誤情境**：預期應該失敗的案例

## 要求

- 使用專案現有的測試框架
- 測試名稱要描述性強
- 使用 AAA 模式（Arrange-Act-Assert）
- 覆蓋率目標 > 80%

如果不確定用什麼框架，預設使用 Jest（JS）或 pytest（Python）。
```

## /mock - 生成 Mock 資料

```markdown
請根據這個結構生成 mock 資料：

$ARGUMENTS

## 要求

1. 生成 5-10 筆合理的假資料
2. 資料要有變化性（不是全部相同）
3. 符合欄位的語意（email 欄位要是 email 格式）
4. 考慮關聯性（如果有 foreign key）

## 輸出格式

JSON 陣列，可直接用於 seed 或測試。
```

## /api - 設計 API

```markdown
請設計以下功能的 REST API：

$ARGUMENTS

## 輸出內容

1. **Endpoints 清單**
   - Method + Path
   - 說明
   
2. **Request/Response 範例**
   - Headers
   - Body（JSON Schema）
   - 狀態碼

3. **錯誤處理**
   - 錯誤碼定義
   - 錯誤回應格式

4. **OpenAPI Spec**（YAML 格式）

遵循 RESTful 最佳實踐。
```

---

# 📝 文件與溝通

## /doc - 生成文件

```markdown
請為這段程式碼生成文件：

$ARGUMENTS

## 輸出內容

1. **概述**：這段程式碼做什麼？
2. **參數說明**：每個參數的型別和用途
3. **回傳值**：型別和可能的值
4. **使用範例**：2-3 個使用情境
5. **注意事項**：陷阱、限制、相依性

格式：使用該語言標準的文件格式（JSDoc/docstring/XML doc）
```

## /readme - 生成 README

```markdown
請為這個專案生成 README：

$ARGUMENTS

## 結構

1. **專案名稱 + 一句話描述**
2. **Features**：主要功能列表
3. **Quick Start**：最快上手的步驟
4. **Installation**：安裝指南
5. **Usage**：使用範例
6. **Configuration**：設定選項
7. **API Reference**（如適用）
8. **Contributing**
9. **License**

風格要簡潔專業，多用 code block 和表格。
```

## /commit - Commit Message

```markdown
請根據這些變更生成 commit message：

$ARGUMENTS

## 格式：Conventional Commits

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types: feat|fix|docs|style|refactor|test|chore|perf|ci

## 要求

- 標題 < 50 字元
- 使用祈使句（Add 而非 Added）
- 如果變更複雜，加上 body 說明 why
```

## /pr - PR 描述

```markdown
請生成 Pull Request 描述：

$ARGUMENTS

## 格式

### 📋 概述
（這個 PR 解決什麼問題？做了什麼？）

### 🔄 主要變更
- 

### 🧪 測試方式
1. 

### 📸 截圖
（UI 變更請附圖）

### ✅ Checklist
- [ ] Self-review 完成
- [ ] 測試通過
- [ ] 文件已更新

### 🔗 相關 Issue
Closes #
```

## /release - Release Notes

```markdown
請根據這些 commits 生成 release notes：

$ARGUMENTS

## 格式

# v X.Y.Z (YYYY-MM-DD)

## ✨ New Features
- 

## 🐛 Bug Fixes
- 

## 🔧 Improvements
- 

## 💥 Breaking Changes
- 

## 📦 Dependencies
- 

---

語氣專業但友善，讓用戶知道升級會得到什麼。
```

---

# 🔄 重構與優化

## /refactor - 重構建議

```markdown
請分析這段程式碼並提供重構建議：

$ARGUMENTS

## 分析維度

1. **DRY**：消除重複
2. **SRP**：單一職責
3. **命名**：清晰度
4. **複雜度**：圈複雜度、巢狀深度
5. **設計模式**：適用的模式

## 輸出

1. 問題清單（按嚴重程度排序）
2. 重構策略
3. 重構後的程式碼
4. Before/After 對比
```

## /clean - 清理程式碼

```markdown
請清理這段程式碼：

$ARGUMENTS

## 清理項目

1. 移除未使用的變數、import、函式
2. 統一命名風格
3. 整理 import 順序
4. 補上遺漏的型別註解
5. 格式化（縮排、空行、括號）

## 輸出

清理後的完整程式碼，不需解釋。
```

## /types - 加上型別

```markdown
請為這段 JavaScript 加上 TypeScript 型別：

$ARGUMENTS

## 要求

1. 推斷合理的型別，避免過多 `any`
2. 使用 interface 而非 type（除非需要 union）
3. 參數和回傳值都要有型別
4. 考慮 null/undefined 的情況

## 輸出

完整的 TypeScript 程式碼 + 需要的 type 定義。
```

## /modernize - 現代化語法

```markdown
請將這段程式碼升級為現代語法：

$ARGUMENTS

## 升級內容

JavaScript/TypeScript：
- var → const/let
- function → arrow function（適當時）
- callback → Promise/async-await
- 使用解構、展開運算子
- Optional chaining、nullish coalescing

Python：
- f-strings
- Type hints
- Walrus operator（適當時）
- match-case（3.10+）

## 輸出

升級後的程式碼，標註改動的地方。
```

---

# 🗄️ 資料庫

## /sql - 生成 SQL

```markdown
請根據需求生成 SQL：

$ARGUMENTS

## 要求

1. 使用標準 SQL（或指定的方言）
2. 加上適當的索引建議
3. 考慮效能（避免 SELECT *）
4. 處理 NULL 情況

## 輸出

```sql
-- 說明這個 query 做什麼
SELECT ...
```

加上執行計畫分析建議（如適用）。
```

## /explain-sql - 解釋 SQL

```markdown
請解釋這個 SQL query：

$ARGUMENTS

## 分析

1. **白話解釋**：這個 query 做什麼？
2. **執行順序**：FROM → WHERE → GROUP BY → SELECT...
3. **效能評估**：
   - 有用到索引嗎？
   - 會全表掃描嗎？
   - 資料量大時會怎樣？
4. **優化建議**（如有）
```

## /migration - 生成 Migration

```markdown
請生成資料庫 migration：

$ARGUMENTS

## 框架

（自動偵測專案用的 ORM，或預設 Prisma/Sequelize/Knex）

## 輸出

1. Up migration（執行變更）
2. Down migration（回滾）
3. 資料遷移腳本（如需要搬動現有資料）
4. 注意事項（鎖表時間、索引建立策略）
```

---

# 🏗️ 架構與設計

## /design - 系統設計

```markdown
請設計以下系統：

$ARGUMENTS

## 設計文件

### 1. 需求分析
- 功能需求
- 非功能需求（效能、可用性、規模）

### 2. 高層架構
- 系統元件圖
- 資料流

### 3. 技術選型
- 語言/框架
- 資料庫
- 快取
- 訊息佇列（如需要）

### 4. API 設計
- 主要 endpoints

### 5. 資料模型
- ERD 或 Schema

### 6. 部署架構
- 基礎設施需求

### 7. 待決事項
- 需要進一步討論的問題
```

## /diagram - 生成圖表

```markdown
請生成以下圖表：

$ARGUMENTS

## 輸出格式

使用 Mermaid 語法：

```mermaid
（圖表內容）
```

支援類型：
- flowchart（流程圖）
- sequenceDiagram（時序圖）
- classDiagram（類別圖）
- erDiagram（ER 圖）
- stateDiagram（狀態圖）
```

---

# ⚙️ DevOps

## /dockerfile - 生成 Dockerfile

```markdown
請為這個專案生成 Dockerfile：

$ARGUMENTS

## 要求

1. Multi-stage build（減少映像大小）
2. 非 root 使用者
3. 適當的 .dockerignore
4. 健康檢查
5. 環境變數配置

## 輸出

1. Dockerfile
2. .dockerignore
3. docker-compose.yml（開發用）
```

## /ci - 生成 CI 配置

```markdown
請生成 CI/CD 配置：

$ARGUMENTS

## 平台

（自動偵測 .github、.gitlab-ci、Jenkinsfile，或指定）

## Pipeline 階段

1. Install dependencies
2. Lint
3. Test
4. Build
5. Deploy（to staging/production）

## 輸出

完整的配置檔，包含：
- 快取策略
- 並行執行
- 條件執行（main branch only 等）
```

## /k8s - Kubernetes 配置

```markdown
請生成 Kubernetes 部署配置：

$ARGUMENTS

## 輸出

1. Deployment
2. Service
3. ConfigMap
4. Secret（結構，不含真實值）
5. Ingress（如需要）
6. HPA（自動擴展）

使用 YAML，加上詳細註解。
```

---

# 🌐 前端專用

## /component - React 元件

```markdown
請建立 React 元件：

$ARGUMENTS

## 要求

1. 使用 TypeScript
2. 使用 hooks
3. Props 要有型別定義
4. 包含基本樣式
5. 處理 loading/error 狀態

## 輸出

1. 元件檔案
2. 型別定義
3. 使用範例
4. 簡單測試
```

## /hook - 自訂 Hook

```markdown
請建立 React 自訂 Hook：

$ARGUMENTS

## 要求

1. 命名以 use 開頭
2. 完整的 TypeScript 型別
3. 處理 cleanup
4. 考慮重複渲染的效能影響

## 輸出

1. Hook 實作
2. 使用範例
3. 測試案例
```

## /a11y - 無障礙審查

```markdown
請審查這段 HTML/JSX 的無障礙性：

$ARGUMENTS

## 檢查項目

1. **語意化**：使用正確的 HTML 標籤
2. **鍵盤操作**：可以用 Tab 導航嗎？
3. **螢幕閱讀器**：有適當的 aria 標籤嗎？
4. **對比度**：顏色對比是否足夠？
5. **表單**：label 和 input 有關聯嗎？

## 輸出

問題清單 + 修復後的程式碼，符合 WCAG 2.1 AA 標準。
```

---

# 📚 學習與解釋

## /explain - 概念解釋

```markdown
請用繁體中文解釋：

$ARGUMENTS

## 格式

1. **一句話總結**
2. **生活化比喻**
3. **實際應用場景**（2-3 個）
4. **程式碼範例**（如適用）
5. **常見誤區**
6. **延伸學習**

語氣輕鬆，對新手友善。
```

## /compare - 技術比較

```markdown
請比較以下技術：

$ARGUMENTS

## 比較表

| 面向 | A | B |
|------|---|---|
| 學習曲線 | | |
| 效能 | | |
| 生態系 | | |
| 適用場景 | | |
| 優點 | | |
| 缺點 | | |

## 結論

什麼情況選 A？什麼情況選 B？
```

## /how - 怎麼做

```markdown
請告訴我怎麼做：

$ARGUMENTS

## 格式

### 前置條件
（需要先準備什麼）

### 步驟
1. 
2. 
3. 

### 驗證
（怎麼確認成功了）

### 常見問題
（可能遇到的坑和解法）

要具體，能複製貼上直接執行。
```

---

# 🔧 工具類

## /regex - 正規表達式

```markdown
請幫我寫正規表達式：

$ARGUMENTS

## 輸出

1. **正規表達式**：`/.../`
2. **解釋**：每個部分的意思
3. **測試案例**：
   - ✅ 應該 match 的
   - ❌ 不應該 match 的
4. **程式碼範例**（JavaScript/Python）
```

## /json - JSON 處理

```markdown
請處理這段 JSON：

$ARGUMENTS

## 支援操作

- 格式化（美化）
- 壓縮（單行）
- 驗證
- 轉換為其他格式（YAML、TypeScript interface）
- 提取特定欄位
- 合併多個 JSON

指定你要做什麼，我會處理。
```

## /convert - 格式轉換

```markdown
請轉換以下內容：

$ARGUMENTS

## 支援轉換

- JSON ↔ YAML
- JSON → TypeScript interface
- CSV → JSON
- Markdown → HTML
- SQL → ORM model
- 其他（請指定）
```

---

# 使用技巧

## 組合使用

```bash
# 先 review 再生成測試
/review src/utils.ts
/test src/utils.ts

# 先解釋再實作
/explain WebSocket
/impl 一個簡單的 WebSocket chat
```

## 客製化

每個技能都可以根據你的需求調整。例如：

- 加上團隊的 code style guide
- 指定使用的框架版本
- 加上公司特定的規範

## 分享技能

```bash
# 把技能加入版本控制
cd ~/.claude
git init
git add commands/
git commit -m "Add custom skills"
git remote add origin git@github.com:yourname/claude-skills.git
git push -u origin main
```

---

# 結語

這 30+ 個技能涵蓋了工程師日常開發的大部分場景。建議先挑 5-10 個最常用的開始，用順了再逐步擴充。

記住：好的技能是迭代出來的。用了幾次發現不順手，就改。這才是真正屬於你的工具。

Happy coding! 🚀

---

*延伸閱讀：[Claude Code 自訂技能完全指南](/2026/02/02/Claude-Skill-完全指南/) — 從零開始建立技能的基礎教學*

*最後更新：2026-02-03*
