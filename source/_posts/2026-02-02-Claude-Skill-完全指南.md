---
title: Claude Code 自訂技能 (Custom Slash Commands) 完全指南
date: 2026-02-02 21:40:00
subtitle: 讓你的 AI 助理學會專屬技能，提升工作效率
categories:
  - AI/ML
tags:
  - claude
  - ai-assistant
  - skills
  - productivity
  - automation
cover_index: https://images.unsplash.com/photo-1677442136019-21780ecad995?w=450&h=450&fit=crop&crop=center
comments: true
---

> 🎯 與其重複解釋同樣的事情，不如教會 AI 一次搞定。

Claude Code（前身 Claude CLI）支援自訂技能（Custom Slash Commands），讓你可以建立可重複使用的指令，大幅提升與 AI 協作的效率。這篇文章會帶你從零開始建立自己的技能系統。

---

# 什麼是 Claude Skill？

簡單說，Skill 就是**預先定義好的指令模板**。當你發現自己經常對 AI 說類似的話，例如：

- 「幫我 review 這段 code，注意安全性和效能」
- 「用繁體中文解釋這個概念，要淺顯易懂」
- 「幫我把這個錯誤訊息翻譯成解決方案」

這些都可以封裝成一個 Skill，之後只要打 `/review`、`/explain`、`/debug` 就能觸發。

---

# 技能存放位置

Claude Code 會從以下位置載入技能：

| 位置 | 說明 | 作用範圍 |
|------|------|----------|
| `.claude/commands/` | 專案目錄下 | 僅該專案 |
| `~/.claude/commands/` | 家目錄下 | 所有專案 |

建議把通用技能放在 `~/.claude/commands/`，專案特定的放在專案目錄下。

---

# 建立你的第一個技能

## 步驟 1：建立目錄

```bash
mkdir -p ~/.claude/commands
```

## 步驟 2：建立技能檔案

技能檔案是 Markdown 格式，檔名就是指令名稱。

例如建立 `/review` 指令：

```bash
touch ~/.claude/commands/review.md
```

## 步驟 3：編寫技能內容

```markdown
# Code Review 專家

請以資深工程師的角度 review 以下程式碼：

$ARGUMENTS

## Review 重點

1. **安全性**：有沒有潛在的安全漏洞？
2. **效能**：有沒有效能瓶頸或可優化的地方？
3. **可讀性**：命名是否清晰？結構是否合理？
4. **錯誤處理**：異常情況有沒有妥善處理？
5. **最佳實踐**：有沒有違反該語言/框架的慣例？

## 輸出格式

- 先總結整體評價（1-10 分）
- 再列出具體問題和建議
- 最後給出改進後的程式碼（如適用）
```

## 步驟 4：使用技能

在 Claude Code 中輸入：

```
/review 
```

然後貼上你的程式碼，或者直接：

```
/review src/utils/auth.ts
```

---

# 進階技巧

## 使用 $ARGUMENTS 變數

`$ARGUMENTS` 會被替換成你在指令後面輸入的內容。

例如技能內容：
```markdown
請將以下內容翻譯成 $ARGUMENTS：
```

使用方式：
```
/translate 日文
這是要翻譯的內容...
```

## 多檔案技能

你可以在技能中參考其他檔案：

```markdown
# 專案 Context

請參考以下檔案瞭解專案架構：
- README.md
- src/index.ts
- package.json

然後根據這個 context 來 $ARGUMENTS
```

## 組合技能

一個技能可以是另一個技能的延伸：

```markdown
# 深度 Review（包含測試建議）

首先執行標準 code review（同 /review），然後額外提供：

1. 單元測試建議
2. 整合測試場景
3. Edge case 列表

$ARGUMENTS
```

---

# 實用技能範例

## 1. /explain - 概念解釋器

```markdown
# 概念解釋專家

請用繁體中文解釋以下概念：

$ARGUMENTS

## 要求

1. 先用一句話總結
2. 再用生活化的比喻說明
3. 給出 2-3 個實際應用場景
4. 如果是程式概念，附上簡單的程式碼範例
5. 最後列出相關的進階主題（供延伸學習）

語氣要輕鬆易懂，避免過度專業術語。
```

## 2. /debug - 錯誤診斷

```markdown
# Debug 助手

請分析以下錯誤訊息並提供解決方案：

$ARGUMENTS

## 分析步驟

1. **錯誤類型**：這是什麼類型的錯誤？
2. **根本原因**：最可能的原因是什麼？
3. **解決方案**：具體的修復步驟
4. **預防措施**：如何避免未來再次發生？

如果錯誤訊息不夠完整，請列出需要的額外資訊。
```

## 3. /commit - Git Commit 訊息生成

```markdown
# Commit Message 生成器

請根據以下變更生成符合 Conventional Commits 規範的 commit message：

$ARGUMENTS

## 規範

格式：`<type>(<scope>): <description>`

Types:
- feat: 新功能
- fix: 修復 bug
- docs: 文件更新
- style: 格式調整（不影響程式邏輯）
- refactor: 重構
- test: 測試相關
- chore: 建置/工具相關

## 輸出

1. 主要 commit message（一行）
2. 詳細說明（如需要）
3. Breaking changes（如有）
```

## 4. /pr - Pull Request 描述生成

```markdown
# PR 描述生成器

請根據以下變更生成 Pull Request 描述：

$ARGUMENTS

## 輸出格式

### 📋 Summary
（一段話總結這個 PR 做了什麼）

### 🔄 Changes
（列出主要變更）

### 🧪 Testing
（說明如何測試這些變更）

### 📸 Screenshots
（如果是 UI 變更，提示需要截圖）

### ✅ Checklist
- [ ] 程式碼已自我 review
- [ ] 已新增必要的測試
- [ ] 文件已更新（如適用）
```

## 5. /refactor - 重構建議

```markdown
# 重構顧問

請分析以下程式碼並提供重構建議：

$ARGUMENTS

## 分析面向

1. **DRY 原則**：有沒有重複的程式碼？
2. **單一職責**：每個函式/類別是否只做一件事？
3. **命名**：變數和函式命名是否清晰？
4. **複雜度**：有沒有過於複雜的邏輯可以簡化？
5. **模式**：是否適合套用設計模式？

## 輸出

1. 問題清單（按優先順序）
2. 重構後的程式碼
3. 重構前後的對比說明
```

---

# 技能管理技巧

## 列出所有技能

```bash
ls ~/.claude/commands/
```

## 快速編輯技能

```bash
code ~/.claude/commands/review.md
# 或
vim ~/.claude/commands/review.md
```

## 分享技能

把 `~/.claude/commands/` 放進 dotfiles repo，就能在不同機器間同步。

## 版本控制

```bash
cd ~/.claude
git init
git add commands/
git commit -m "Add custom skills"
```

---

# 結語

自訂技能是提升 AI 協作效率的利器。花點時間把常用的工作流程封裝成技能，長期下來會省下大量時間。

我的建議：
1. **從小開始**：先建立 2-3 個最常用的技能
2. **持續迭代**：用了幾次後根據實際需求調整
3. **分享交流**：好用的技能可以分享給團隊

Happy prompting! 🚀

---

*最後更新：2026-02-02*

*相關資源：*
- [Claude Code 官方文件](https://docs.anthropic.com/claude-code)
- [Conventional Commits](https://www.conventionalcommits.org/)
