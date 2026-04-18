\---

name: mosa-harmonizer
description: "MOSA 框架大一統協調員：負責全量審計架構完整性、對齊索引標籤與路徑、並提供系統整合方向。當框架出現邏輯衝突、路徑失效或索引不對稱時啟動。"
skill\_id: MOSA\_HARMONIZER
category: Core
---

# MOSA Harmonizer (框架大一統協調員)
* **功能 1**：全量審計架構完整性，找出硬編碼路徑。
* **功能 2**：比對索引與實際檔案，定位 **Orphan Nodes (孤島技能)**。
* **功能 3**：同步修正框架中的邏輯斷點。
* **功能 4**：**長效記憶維護 (Memory Keeper)**。接收 `--update-stack` 指令，讀取 `task_results.md`，並將當前任務的狀態與成就正確寫入 `00_System/prompt_stack.md`，防止上下文漂移。
## 0\. 元邏輯：全量對齊 (Framework Alignment)

本技能不處理具體業務，僅負責維護 MOSA 框架的「架構純度」與「整合深度」。每次調用時，必須執行以下四層掃描：

### A. 系統協議掃描 (Layer A Protocols)

* 檢查 `\~/.gemini/GEMINI.md` 是否符合「統一啟動序列」。
* 確保無路徑硬編碼（嚴禁出現 `C:\\Users\\...`）。
* **掃描對象（明確）：**

  * `\~/.gemini/GEMINI.md`
  * `\~/.gemini/antigravity/workflows/\*.md`（所有 Agent 定義）
  * `\~/.gemini/antigravity/skills/\*/SKILL.md`（所有技能定義）
  * `{Workspace\_Root}/00\_System/prompt\_stack.md`
  * `{Workspace\_Root}/00\_System/state.json`

### B. 認知索引掃描 (Layer B Memory)

* 比對 `knowledge-base/\_index.json` 與 `experience/\_index.json`。
* 檢查關鍵詞與 `skills\_registry.json` 的對稱性。
* **目標**：確保「想到什麼」就能「載入什麼」。

### C. 執行路徑掃描與拓撲審核 (Layer C Agents)

* 呼叫 `mosa-graph-builder` 對 `workflows/` 與 `skills/` 生成局部的架構圖譜。
* 讀取生成的 `GRAPH\_REPORT.md`，從 Mermaid 節點圖中尋找並列出 **Orphan Nodes** (失去依賴的孤島，代表未註冊或無法被引用的技能)。
* 尋找 **Unexpected Connections** (圖譜中不經過 God Nodes 卻直接跨層連接的異常邊，如 Layer D 檔案直連 Layer A 規則)。
* 檢查 `orchestrator\_agent` 是否正確執行「寫放分離」。
* **驗證責任邊界**：確認經驗記錄觸發點僅在 auto-skill Step 5（不在 orchestrator GC 中重複）。
* **護盾一致性校驗**：檢查所有已部署的 `AGENTS.md` 是否與 `mosa-graph-builder` SKILL.md 中定義的模板內容一致。

### D. 工作空間掃描 (Layer D Workspace)

* 檢查 `00\_System`, `01\_Work`, `02\_Output` 是否初始化。
* 檢查 `session\_state.json` 是否包含非法全文數據（指針檢查）。
* 驗證 `state.json` 包含 `turn\_count` 與 `drift\_threshold`（GEMINI.md §工作空間隔離）。

### E. 負載與歸檔掃描 (Layer E Maintenance) -- 僅在 --maintenance 模式啟動

* **觸發條件**：`state.json` 中 `turn\_count >= drift\_threshold`，由 orchestrator Step 3 轉交。
* **日誌壓縮**：將 `01\_Work/Agent\_Activation\_Log.md` 轉存至 `02\_Output/Archive/Log\_\[Timestamp].md`。
* **狀態重置**：清空 `session\_state.json` 並重置 `state.json` 中的 `turn\_count` 為 0。
* **指針固化**：將當前 `implementation\_plan.md` 的關鍵成就寫入 `00\_System/prompt\_stack.md` 作為長期記憶。

\---

## 整合方向報告規範 (Standard Output)

每次執行後，必須輸出以下結構的報告：

### 1\. 結構完整性 (Architecture Integrity)

* \[ ] Layer A: \[Status]
* \[ ] Layer B: \[Status]
* \[ ] Layer C: \[Status]
* \[ ] Layer D: \[Status]

### 2\. 發現的衝突/Bug (Discovered Misalignments)

* 列表展示所有硬編碼路徑、冗餘指令或斷裂的索引。

### 3\. 未來整合方向 (Future Direction)

* 根據當前專案，建議下一個需要建立的 Skill 或 Agent 角色。
* 建議知識庫的歸檔路徑。

\---

## 執行動作 (Execution Actions)

1. **\[Skill Trigger: mosa-graph-builder]** - 呼叫該技能對工作空間做先導的拓撲圖譜建置。
2. **\[Tool: view\_file]** - 優先讀取產出的 `GRAPH\_REPORT.md` 以鎖定框架中的孤立節點與邏輯異常區塊。
3. **\[Tool: list\_dir]** - 掃描技能庫與工作流程目錄比對實際檔案。
4. **\[Tool: view\_file]** - 讀取三份索引文件 (`knowledge`, `experience`, `registry`)。
5. **\[Tool: grep\_search]** - 檢索全庫中的 `Hardcoded Paths`。
6. **\[Tool: replace\_file\_content]** - 自動修復路徑並同步索引。

