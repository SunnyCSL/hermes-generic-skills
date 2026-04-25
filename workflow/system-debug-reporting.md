---
name: system-debug-reporting
description: 系統問題彙報格式 — Before/After 對比 + 根因鏈分析
version: 1.0.0
author: Hermes Agent
license: MIT
tags: [debug, reporting, root-cause]
---

# System Debug Reporting — Before/After 對比格式

## 核心原則
每次系統問題彙報都要包含：
1. **之前點樣** (Before) — 問題未修復前嘅狀態
2. **發生咗乜** (What Happened) — 具體 error / 症狀
3. **為乜會咁** (Root Cause) — 點解會發生
4. **改咗啲乜** (What Changed) — 修復方案

## 格式模板

```
問題：[簡短描述]

之前點樣：
  → 具體情況

發生咗乜：
  → 錯誤訊息 / 觀察到嘅問題

點解會咁：
  → Root cause chain（層層追問點解）

修復咗啲乜：
  → 具體改動 / PR number
```

## 範例

```
問題：Compression/Session-Search 頻繁 400/401 errors

之前點樣：
  → Auxiliary tasks (auto) 被 routing 去 Gemini Flash
  → Gemini Flash 香港訪問唔到
  → 每次 compression 都 fail

發生咗乜：
  → HTTP 400/401 errors 喺 compression 同 vision tasks

點解會咁：
  → _resolve_auto() 優先級寫錯咗
  → aggregator users 唔用 main model
  → 走去用 provider-side default (Gemini Flash)
  → Gemini Flash 香港 region 封鎖

修復咗啲乜：
  → PR #11900 改咗 _resolve_auto() 優先級
  → "auto" 以後用 main model
```

## 適用場景
- System update 後發現問題
- Cron job 突然 fail
- API errors 頻繁出現
- Provider failover 發生
- 任何非預期嘅系統行為

## 價值
- 結構化彙報方便日後 self-healing
- 見到問題模式就知道係咩事
- 存档到 vault 方便日後查詢
