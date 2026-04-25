---
name: skill-copyright-filter
description: 過濾 skills 版權風險 + 判斷係咪值得分享嘅決策框架
version: 1.0.0
author: Hermes Agent
license: MIT
trigger: "用户要求上傳 skill 去 public repo / 評估 skill 係原創定 adapted / 決定邊個 skills 可以發佈"
---

# Skill Copyright Filter — 版權過濾決策框架

## 核心原則

> **只上傳我哋自己創建嘅 skills，唔上傳下載或 adapted 嘅 third-party skills**
> — Sunny, 2026-04-25

---

## 判斷流程

### Step 1: 檢查 author frontmatter

```
author: Hermes Agent  ✅ 原創
author: Hermes Agent (adapted from X)  ❌ adapted
author: 第三方名  ❌ 第三方
```

有 author field 就要懷疑，唔好假設係原創。

### Step 2: 掃 hardcoded paths

搜尋 SKILL.md + 所有 references/ scripts/ 內嘅：

```
/home/<username>/   ← 用戶個人路徑
\.hermes/           ← Hermes 特定
.sunnycsl           ← 用戶名
```

**有 hardcoded 路徑 ≠ 唔可以上傳**，但需要通用化（見 Step 4）。

### Step 3: 識別 adapted/third-party skills

用關鍵字 scan：

```
(obra|superpowers)     → obra/superpowers 體系
JimLiu|baoyu            → baoyu-skills (JimLiu)
dodo-reach|Synero       → pixel-art-studio
SHL0MS                  → SHL0MS projects
Cocoon AI               → Cocoon AI projects
elder-plinius           → G0DM0D3 / L1B3RT4S / OBLITERATUS
VoltAgent|awesome-design → Teknium design systems
0xbyt4                  → ascii-image-converter
google-labs-code|design.md → Google design.md
MorAlekss               → MorAlekss contributions
```

Match 到任何一項 → ❌ 唔可以自動上傳，需要明確版權確認。

### Step 4: 通用化改動（針對原創但有 hardcoded path 嘅 skills）

把用戶特定路徑替換為環境變量：

| 原本 | 通用版 |
|---|---|
| `.hermes/plans/` | `${AGENT_WORKSPACE}/plans/` |
| `/home/sunnycsl/.hermes/` | `~/.agent/` |
| `Nexi` / `Cow` / 具體 Bot 名 | `the agent` |
| 具体 API key / token | 移除或用 `${API_KEY}` |
| 私人 image paths (`~/.hermes/cache/images/`) | 移除，說明需要替換 |

### Step 5: 上傳決策

| 状態 | 行動 |
|---|---|
| 原創 + 通用化完成 | ✅ 通知 Sunny，等確認後上傳 |
| 原創 + 但有私人content | ⚠️ 通知 Sunny，確認 content 係咪可以清理 |
| Adapted third-party | ❌ 唔上傳 |
| 唔確定 | ❌ 唔上傳，先問 Sunny |

---

## 版權警戒綫

以下情況明確唔上傳：
- 明確標注 adapted from XXX
- 包含 third-party 作者名
- 含別人嘅 repo URL（非工具參考）
- 文件有 `license: AGPL-3.0` 或其他傳染性 license

---

## 上傳後準則

1. **一次上傳夠了** — Decision Engine 案例：唔需要每日自動 push
2. **被動模式** — 當有原創 skill 改動，先通知用戶，用戶確認後先上傳
3. **唔自我推斷** — 覺得值得分享，通知用戶，唔好自己決定上傳

---

## 已上傳 Repo

**github.com/SunnyCSL/hermes-generic-skills**（Sunny 已確認，2026-04-25）

上傳了 6 個原創通用 skills：
- `software-development/plan.md`
- `devops/skill-frontmatter-live-injection.md`
- `workflow/system-debug-reporting.md`
- `productivity/context-compression.md`
- `creative/excalidraw.md`
- `creative/design-md.md`

---

## 示例决策

**excalidraw** — `author: Hermes Agent`, 無 third-party 參考 → ✅ 已上傳

**system-debug-reporting** — 原創，清理 example 後 → ✅ 已上傳

**frog-beam-panic-game** — 原創遊戲，含私人圖片路徑 → ❌ 唔上傳，私人參考文檔

**pixel-art** — `author: dodo-reach` + ported from `Synero/pixel-art-studio` → ❌ 第三方版權

**baoyu-comic** — `author: 宝玉 (JimLiu)` → ❌ 第三方
