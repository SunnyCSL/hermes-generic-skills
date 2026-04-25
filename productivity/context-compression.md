---
name: context-compression
description: 對話壓縮與 Context 優化技巧
version: 1.0.0
author: Hermes Agent
license: MIT
trigger: 對話超過 30 回合或 context 超過 65% 時觸發
---

# Context Compression 對話壓縮優化

## 核心原則

### 1. 回應要精準 (Be Concise)
- 唔好長篇大論，除非必須
- 一句能夠表達就唔好用三句
- 用短句，唔好用複雜句式

### 2. 避免重複 (No Redundancy)
- 唔重複雙方已知的事實
- 已確認的決定直接執行，唔需要再確認
- 資訊只出現一次，除非特意強調

### 3. 上下文總結 (Summarize Context)
- 長時間任務完成後，用一句話總結
- 保持「現狀快照」而不是完整歷史
- 完成的步驟可以直接移除

### 4. 極限壓縮格式 (Aggressive Compression)
- 用 → 代替 "所以"
- 用 ‖ 代替 "但是"
- 用 ‣ 代替列舉
- 用 ▸ 代替 "我建議"
- 任務狀態用: [done] [in-progress] [pending]

### 5. 記憶 vs Context
- 長期資訊 → 用 mcp_memory 寫入，唔在對話中保留
- 即時狀態 → 留在 context，唔寫入記憶
- 完成嘅任務 → 直接移除，唔保留過程

### 6. 當 context 接近 70% 時
- 主動總結之前的對話
- 用 bullet points 壓縮
- 移除已完成的部分

## 實踐技巧

### 好的例子:
User: "整好咗 X，Y 失敗咗，點算？"
Nexi: "Y 失敗了。選項: [1] retry [2] skip [3] manually fix"

### 唔好:
Nexi: "我注意到你話 X 整好咗，但係 Y 出現咗問題。我建議你可以考慮..."
(太冗長，唔需要這些包裝)

### 極限壓縮範例:
原本: "我睇到系統負荷比較高，CPU 使用率達到 80%，記憶體使用量係 720MB"
壓縮: "負載高: CPU 80% / RAM 720MB"

## 觸發時機
- 對話超過 20 回合
- 回應長度超過 3 段落
- 系統提示即將接近 80% context