---
title: Cat-Raising：用 Next.js 打造科學養貓的營養計算工具
date: 2026-02-05 20:18:00
subtitle: 從零到一，開發一個基於 AAFCO 標準的貓咪營養分析 PWA
categories:
  - 開發筆記
tags:
  - nextjs
  - react
  - supabase
  - pwa
  - 寵物科技
cover_index: https://res.cloudinary.com/dsvl326mi/image/upload/c_fill,w_800,h_800,g_center/v1770293992/blog_covers/c2egznglswshssppfxxd.png
comments: true
---

> 🐱 每一位貓奴都該擁有的科學餵養工具——讓乾物質計算不再是噩夢。

身為多貓家庭的一員，你是不是也曾經對著貓糧包裝上的營養成分表發呆，試圖搞懂「這罐主食罐的蛋白質夠不夠高」、「這款乾糧的碳水會不會太多」？

Cat-Raising 就是為了解決這個痛點而生的。

---

# 專案緣起

養貓新手最常聽到的一句話就是：「要看乾物質比，不是看原始成分」。

問題是，手動計算乾物質比真的很麻煩：

```
乾物質含量 = 100% - 水分%
乾物質蛋白質 = 蛋白質% ÷ 乾物質含量 × 100%
```

每換一款貓糧都要拿計算機敲半天，還要對照 AAFCO 標準判斷合不合格。於是我決定做一個工具來自動化這件事。

# 技術棧選擇

```
┌─────────────────────────────────────────┐
│           Cat-Raising 架構              │
├─────────────────────────────────────────┤
│  Frontend: Next.js 14 + React 18        │
│  Styling: Tailwind CSS 4 + shadcn/ui    │
│  Backend: Supabase (PostgreSQL + Auth)  │
│  Deploy: Vercel (免費方案)               │
│  PWA: next-pwa + Service Worker         │
└─────────────────────────────────────────┘
```

## 為什麼選 Next.js 14？

1. **App Router** - 新版路由系統更直覺，檔案即路由
2. **Server Components** - 減少 client bundle 大小
3. **Built-in Optimizations** - 圖片優化、字體優化開箱即用
4. **Vercel 部署** - 一鍵部署，免費額度對小專案綽綽有餘

## 為什麼選 Supabase？

1. **PostgreSQL** - 正統關聯式資料庫，不是 NoSQL 的妥協
2. **Row Level Security** - 內建的資料隔離，不用自己寫權限邏輯
3. **Auth 系統** - Google OAuth 幾行程式碼就搞定
4. **免費額度** - 對個人專案來說非常夠用

# 核心功能實作

## 營養計算引擎

這是整個專案的核心，程式碼其實不複雜：

```typescript
export function calculateNutrition(input: FoodCalculationInput): CalculationResult {
  // 基礎計算：乾物質含量
  const dryMatterContent = 100 - input.moisture_percent

  // 乾物質基準營養成分
  const dmProtein = (input.protein_percent / dryMatterContent) * 100
  const dmFat = (input.fat_percent / dryMatterContent) * 100
  const dmFiber = (input.fiber_percent / dryMatterContent) * 100
  const dmAsh = (input.ash_percent / dryMatterContent) * 100

  // 熱量比計算（使用正確的 ME 係數）
  const proteinCalories = proteinGrams * 3.5  // 蛋白質 3.5 kcal/g
  const fatCalories = fatGrams * 8.5          // 脂肪 8.5 kcal/g
  const carbCalories = carbGrams * 3.5        // 碳水 3.5 kcal/g

  return result
}
```

關鍵點：
- 使用 **Modified Atwater 係數** 計算熱量比，這是 AAFCO 建議的貓狗食品熱量計算方式
- 蛋白質和碳水是 3.5 kcal/g（不是人類的 4 kcal/g）
- 脂肪是 8.5 kcal/g（不是人類的 9 kcal/g）

## 營養評估標準

基於 AAFCO 和各種貓營養學文獻，我設定了以下評估標準：

| 項目 | 合格標準 | 說明 |
|------|---------|------|
| 蛋白質乾物比 | ≥35% | 貓是肉食動物，蛋白質不能少 |
| 脂肪乾物比 | 30-50% | 提供能量，太低或太高都不好 |
| 碳水乾物比 | ≤10% | 貓不需要太多碳水 |
| 纖維乾物比 | ≤2% | 高纖維影響消化 |
| 鈣磷比 | 1.1-1.8:1 | 骨骼健康的關鍵 |
| 磷含量 | <350mg/100kcal | 腎臟友善指標 |

## 多貓管理系統

既然要做，就做完整一點。支援多貓家庭的管理：

- 16 款可愛貓咪頭像選擇
- 自動計算貓咪年齡
- 每隻貓獨立的營養記錄
- 收藏功能標記常用貓糧

## 玻璃質感 UI

這次想嘗試 Glassmorphism 風格，主要透過這段 CSS 實現：

```css
.glass {
  background: rgba(255, 255, 255, 0.65);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.25);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.1);
}
```

配合淡藍色漸層背景和微動效，整體視覺效果還算滿意。

# 踩過的坑

## 1. iOS Safari 的 `-webkit-backdrop-filter`

Safari 對 `backdrop-filter` 的支援需要加上 `-webkit-` 前綴，否則玻璃效果完全不生效。這個坑花了我一個晚上才發現。

## 2. Supabase RLS 政策

一開始設定 RLS 時漏掉了 `food_calculation_cats` 關聯表的 INSERT 權限，導致多貓關聯功能一直報錯。

修復方式：

```sql
CREATE POLICY "Users can create their own food calculation cat associations"
  ON food_calculation_cats
  FOR INSERT
  WITH CHECK (
    EXISTS (
      SELECT 1 FROM food_calculations fc
      WHERE fc.id = food_calculation_cats.food_calculation_id
      AND fc.user_id = auth.uid()
    )
  );
```

## 3. 手機上的按鈕點擊問題

在 iOS 上，如果按鈕太小或太靠近其他元素，點擊經常失靈。解決方案：

```tsx
<button
  style={{
    minHeight: '48px',  // 至少 48px 觸控區域
    touchAction: 'manipulation',  // 禁用雙擊縮放
    WebkitTapHighlightColor: 'transparent'
  }}
>
```

## 4. PWA 安裝提示

iOS 不支援原生的 `beforeinstallprompt` 事件，需要自己做一個提示 UI，引導用戶使用 Safari 的「加入主畫面」功能。

# 資料庫設計

```
┌─────────────┐     ┌─────────────────────┐
│    users    │────<│        cats         │
└─────────────┘     └─────────────────────┘
      │                      │
      │              ┌───────┴───────┐
      │              │               │
      ▼              ▼               ▼
┌─────────────────────────┐  ┌─────────────────────────┐
│   food_calculations     │──│  food_calculation_cats  │
└─────────────────────────┘  └─────────────────────────┘
```

使用多對多關聯表 `food_calculation_cats` 來處理「一筆計算記錄可以關聯多隻貓」的需求，比起在 `food_calculations` 裡存 `cat_ids` JSON 陣列更乾淨。

# 未來規劃

目前 Cat-Raising 完成了 MVP 階段，主要功能都已實作。第二階段規劃包括：

**P1 高優先級：**
- 健康記錄管理（疫苗、醫療、體重追蹤）
- 飲食日記系統

**P2 中優先級：**
- 智能提醒（餵食、用藥、健檢）
- 支出管理
- 庫存追蹤

**P3 低優先級：**
- 數據分析儀表板
- AI 建議系統

# 結語

做 Cat-Raising 最大的收穫是：**解決自己的痛點是最好的專案動力**。

現在每次買新貓糧，我就打開手機輸入成分，幾秒鐘就知道這款貓糧的營養品質如何。不用再手動算乾物質，不用再對照標準表，一切都自動化了。

專案已開源，歡迎參考：

- **線上 Demo**：部署在 Vercel 上
- **技術棧**：Next.js 14 + Supabase + Tailwind CSS
- **完全免費**：在免費額度內可支援約 1,000-2,000 活躍用戶

希望這篇開發筆記對想做寵物科技應用的開發者有所幫助！

---

*🐈 願每一隻貓咪都能吃得健康、活得開心。*
