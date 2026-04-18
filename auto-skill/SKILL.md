\---

name: auto-skill
description: 'CRITICAL PROTOCOL: 本技能是所有任務或其他技能的底層依賴。嚴禁在未讀取auto-skill/skill.md的情況下啟動任何開發任務。這不是建議，而是必須執行的初始化程序。任何跳過此步驟的行為都將被視為任務失敗。任何任務都必須同時啟用auto-skill（即使其他技能已觸發）。每次對話自動檢查知識庫並調用最佳實踐；同時建立跨技能經驗記憶層，讓下次使用同技能時能主動提醒。當用戶表達滿意時，必須詢問是否記錄經驗。適用於所有任務型對話。> ⚠️ **邊界宣告 (Scope Limit)**：本 auto-skill 僅負責「跨專案通用的經驗與知識 (General/Experience)」萃取。關於「當前專案的進度與成就」固化，由 `mosa-harmonizer` 負責，本技能不予干涉。'
skill\_id: AUTO\_SKILL
category: Core
---

# Auto-Skill 自進化知識系統 (v2.0: Logic-Driven)

## 0\. 元邏輯層 (Meta-Logic Layer) - 必須在任何工具調用前執行

在開始任何任務前，必須先啟動以下兩個核心思維：

### A. 逆向需求工程 (Reverse Requirement Engineering / Inversion)

* **目標 $G$**: 明確定義最終交付成果。
* **已知變量 $V\_{known}$**: 掃描當前上下文中的可用資訊。
* **缺失變量 $\\Delta V$**: 識別達成目標所需的關鍵缺失資訊。
* **執行動作**: 若 $\\Delta V$ 存在，**必須先向用戶提問補全**，嚴禁「垃圾進，垃圾出 (GIGO)」。

### B. 結構化解構 (Atomic Decomposition)

* **核心思維**: 將模糊的大型任務拆解為「不可再分」的原子動作。
* **Upstream**: 接收模糊指令。
* **Process**: 拆解為 \[原子組件 1] -> \[原子組件 2] -> \[原子組件 3]。
* **Downstream**: 每個原子動作對應一個簡單的 `Tool Wrapper` 或 `Sub-Agent`。

### C. Artifact 權杖優化 (Artifact Token Optimization / ATO)

* **目標**: 最小化長對話中的 Token 冗餘，維持模型注意力。
* **增量追蹤 (Delta Tracking)**: 嚴禁在對話中重複輸出全量 `task.md`，僅展示當前變更。
* **寫放分離 (Decoupling)**: 過程數據寫入 `session\_state.json`，Artifact 僅保留核心決策。
* **定期快照**: 當任務階段完成時，主動進行計畫歸檔與 `prompt\_stack.md` 成就固化。

\---

## 核心循環（Step 1–5）

你必須在每一輪對話中遵循以下核心循環：

### 0.5 系統協議同步 (Protocol Reference)

* 本技能不再重複定義「工作空間隔離」與「初始化結構」規則。
* **強制執行**：請參閱 `\~/.gemini/GEMINI.md` 或本專案的 `prompt\_stack.md` 獲取 Workspace Lock 與 Isolation 協議。

### 0.7 跨 Agent 持久化邊界 (MOSA Persistence)

* **執行協議**：請遵循 `GEMINI.md` 第 5、28-31 點關於 `session\_state.json` 與「強制指針引用 (Pointers Only)」的規範。
* **目標**：防止上下文溢出，確保 Sub-Agents 僅傳遞文件路徑而非全量數據。

### 0\. 對話內快取（不對用戶展示）

在同一對話串中維護以下快取：

* `last\_keywords`
* `last\_topic\_fingerprint`
* `last\_index\_lastUpdated`
* `last\_matched\_categories`
* `last\_used\_skills`（本回合用到的非 auto-skill 技能清單）
* `missing\_experience\_skills`（experience 未命中的技能）
* `loaded\_experience\_skills`（本對話已讀取過經驗的 skill-id）

### 1\. 每回合先抽取關鍵詞（不讀檔）

* 從當前用戶訊息抽取 3–8 個核心名詞/短語（去重、統一大小寫）。
* 生成 `topic\_fingerprint = 前 3 個關鍵詞`。

### 2\. 判斷是否話題切換（不讀檔）

當出現以下任一條件，視為話題切換：

* 明確轉折詞：例如「另外」「改成」「換成」「再來」「順便」
* 本回合關鍵詞與 `last\_keywords` 差異 >= 40%
* 用戶明確要求新增/修改分類

### 3\. 跨技能經驗讀取（強制規則，不受話題切換影響）

只要本回合使用了任何「非 auto-skill」技能：

* 若該 `skill-id` 已存在於 `loaded\_experience\_skills`，本回合**不重讀**、**不重複提示**
* 否則必須執行以下步驟：

  1. 讀取 `experience/\_index.json`
  2. 若找到對應 `skill-id`，必須載入該經驗檔 `experience/skill-\[skill-id].md`
  3. 將該 `skill-id` 加入 `loaded\_experience\_skills`
  4. 回覆中必須提示：`我已讀取經驗：skill-xxx.md`
  5. 若 `experience/\_index.json` 沒有該技能，記錄到 `missing\_experience\_skills`

### 4\. 只在話題切換時讀取知識庫（knowledge-base）

若是本對話第一次回合或判定話題切換，才執行以下步驟：

* 讀取 `knowledge-base/\_index.json`
* 以本回合關鍵詞匹配所有分類 `keywords`
* **匹配到多少分類就讀多少分類**（不做優先級排序）
* 若沒有匹配分類，依「動態分類」流程處理
* 若本回合有讀取任何分類檔，回覆中需加入一行提示：
`我已讀取知識庫：design-layout.md, frontend-dev.md`
（以實際讀取檔名替換，逗號分隔）

若不是話題切換，沿用 `last\_matched\_categories`，不重讀索引與分類檔。

### 5\. 任務結束：主動記錄（最重要！— 唯一觸發點）

> \*\*本 Step 是經驗記錄的唯一觸發點。\*\* orchestrator\_agent GC 階段僅需確認本 Step 已執行，嚴禁重複觸發詢問。

> \*\*任務明顯已完成\*\*：你判斷本回合已高完成且值得記錄時
> \*\*觸發詞\*\*：用戶表達對任務滿意時

**你必須執行以下步驟：**

1. **總結經驗**：用一句話提煉本次解決方案的精華
2. **判斷價值**：這個經驗下次能幫用戶省時間嗎？
3. **主動詢問**：必須說出類似這樣的話：

> 「這次我們解決了 \[問題描述]，我想把這個經驗記錄到你的知識庫，下次遇到類似問題時可以直接參考。你覺得可以嗎？」

4. **執行記錄**：用戶同意後，依下列規則寫入並更新索引：

   * **跨技能經驗**：若本回合使用非 auto-skill，且該技能在 experience 中不存在或有新技巧 → 寫入 `experience/skill-\[skill-id].md`，更新 `experience/\_index.json`
   * **一般知識**：若為通用流程/偏好/解法 → 寫入 `knowledge-base/\[category].md`，更新 `knowledge-base/\_index.json`

**強制規則：缺少經驗時必問**
若本回合使用了非 auto-skill 技能，且該技能不在 `experience/\_index.json`：

* 任務結束時必須主動詢問是否記錄本次使用經驗
* 詢問語句需明確指向該技能，例如：

> 「這次使用了 remotion-best-practices，但經驗庫沒有紀錄。我可以把這次的做法記錄下來嗎？」

\---

## 記錄判斷準則

**核心問題：這東西下次能讓用戶省時間嗎？**

### General（knowledge-base）

**應該記錄（general）：**

* ✅ 可重用的流程與決策步驟（跨領域通用的操作順序/判斷流程）
* ✅ 高成本的錯誤與修正路徑（犯錯會浪費大量時間的情況）
* ✅ 關鍵參數/設定/前置條件（一變就影響結果的要素）
* ✅ 使用者偏好與風格規則（語氣、格式、設計風格、輸出結構）
* ✅ 多次嘗試才成功的方案（包含失敗原因與成功條件）
* ✅ 可套用的模板/清單/格式（會反覆使用的輸出樣式）
* ✅ 外部依賴或資源位置（檔案路徑、工具、素材）

**不應記錄（general）：**

* ❌ 一問一答、沒有可重用流程
* ❌ 純概念解釋（沒有具體做法或判斷標準）
* ❌ 沒有具體上下文、不可復用的結論

### Experience（非 auto-skill 經驗）

**應該記錄（experience）：**

* ✅ 使用該技能時踩到的坑與解法（含錯誤訊息/定位方式）
* ✅ 影響結果的關鍵參數或配置（如 spring 參數、fps、duration）
* ✅ 可重用的模板/提示詞/工作流程（可直接套用）
* ✅ 依賴或資產路徑（字體、圖片、專案入口、模組位置）
* ✅ 需要特定順序或技巧才成功的步驟（例如先初始化再覆蓋）

**不應記錄（experience）：**

* ❌ 純理論或概念性解釋（留在 knowledge-base）
* ❌ 沒有可重現步驟的結論
* ❌ 一次性、不可重用的操作

\---

## 條目格式

### knowledge-base 條目格式

```markdown
## 🔧 \[簡短標題]
\*\*日期：\*\* YYYY-MM-DD
\*\*情境：\*\* 一句話描述使用場景
\*\*最佳實踐：\*\*
- \[重點 1]
- \[重點 2] - 參數說明和調整指南
```

### experience 條目格式

```markdown
## 🔧 \[問題/技巧標題]
\*\*日期：\*\* YYYY-MM-DD
\*\*技能：\*\* \[skill-id]
\*\*情境：\*\* 一句話描述本次問題
\*\*解法：\*\*
- 具體步驟 1
- 具體步驟 2
\*\*關鍵檔案/路徑：\*\*
- /path/to/file
\*\*keywords：\*\* keyword1, keyword2, keyword3
```

\---

## 存儲路徑

* 知識索引：`knowledge-base/\_index.json`
* 知識內容：`knowledge-base/\[category].md`
* 經驗索引：`experience/\_index.json`
* 經驗內容：`experience/skill-\[skill-id].md`

\---

## 動態分類（僅 knowledge-base）

當用戶的問題不屬於現有分類時：

1. 建議創建新分類
2. 詢問用戶分類名稱和關鍵詞
3. 創建新的 `.md` 文件並更新 `\_index.json`

\---

